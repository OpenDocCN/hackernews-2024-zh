<!--yml

类别: 未分类

日期: 2024-05-27 14:35:30

-->

# "1195725856" 和其他神秘数字

> 来源：[`chrisdown.name/2020/01/13/1195725856-and-friends-the-origins-of-mysterious-numbers.html`](https://chrisdown.name/2020/01/13/1195725856-and-friends-the-origins-of-mysterious-numbers.html)

# "1195725856" 和其他神秘数字

上周是 Facebook 本半年绩效审查的最后一周，我们在此期间撰写了关于我们和同事在过去半年中的工作和影响的总结。自然而然，这只能意味着一件事：整个公司都趋向于达到拖延的巅峰水平，不惜一切代价地做任何事情来避免不可言喻的恐怖，即写几段文字的不可避免。

在最后期限前几天，我个人的分心选择是看着这样的行，从一些提供 NFS 流量的主机中不断发送：

```
RPC: fragment too large: 1195725856
RPC: fragment too large: 1212498244 
```

让我们看一下负责生成此警告的内核代码。在[`svc_tcp_recv_record`中查找“fragment too large”](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/net/sunrpc/svcsock.c?h=v5.4#n943)显示：

```
if (svc_sock_reclen(svsk) + svsk->sk_datalen >
                            serv->sv_max_mesg) {
    net_notice_ratelimited(
        "RPC: fragment too large: %d\n",
        svc_sock_reclen(svsk));
    goto err_delete;
}
```

所以我们出错了，因为我们收到了一条超出了`sv_max_mesg`的消息，很酷。但这是从哪里来的呢？查看`svc_sock_reclen`显示如下：

```
static inline u32 svc_sock_reclen(struct svc_sock *svsk)
{
    return ntohl(svsk->sk_reclen) & RPC_FRAGMENT_SIZE_MASK;
}
```

`ntohl`将一个 uint 从网络字节顺序转换为主机的字节顺序。与`RPC_FRAGMENT_SIZE_MASK`进行的按位`AND`导致只保留了一些数据，查看定义显示了我们保留了多少位：

```
#define RPC_LAST_STREAM_FRAGMENT (1U << 31)
#define RPC_FRAGMENT_SIZE_MASK   (~RPC_LAST_STREAM_FRAGMENT)
```

好的，我们将只保留前 31 位并清除高位比特，因为`~`是按位`NOT`。

这意味着这些数字来自片段的前四个字节，省略了最高位，该位保留用于记录该片段是否为此记录的最后一个片段（参见[`svc_sock_final_rec`](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/include/linux/sunrpc/svcsock.h?h=v5.4#n47)）。特别是错误发生得如此早的片段解析让我想到，片段可能根本不符合协议，因为我们根本没有进行到任何地方的处理，甚至没有超过前四个字节。那么，那么*这*四个字节是什么呢？查看十六进制数中的数字显示了一些有趣的东西：

```
% python
>>> hex(1195725856)
'0x47455420'
>>> hex(1212498244)
'0x48454144'
```

这些数字都非常紧密地聚集在一起，通常从 0x40 到 0x50，这意味着每字节可能实际上有一些语义含义。而且由于这些是`char`大小的，这里猜测可能编码了什么……

```
>>> '\x47\x45\x54\x20'
'GET '
>>> '\x48\x45\x41\x44'
'HEAD'
```

哦亲爱的。有人正在向 NFS RPC 发送 HTTP 请求，但至少我们直接拒绝了片段，而不是实际分配并使一千兆字节的内存脏。

接下来是找出到底是谁在发送这些请求。`rpcinfo -p`显示 NFS 正在监听默认端口 2049，所以我们可以像这样设置 tcpdump 陷阱：

```
tcpdump -i any -w trap.pcap dst port 2049
# ...wait for logs to appear again, then ^C...
tcpdump -qX -r trap.pcap | less +/HEAD 
```

从这里，追溯到原始主机和服务，利用捕获的 pcap 数据，很容易就能抓住这些请求的来源。之后，你可以与负责的团队协调一下，弄清楚这里到底发生了什么，并且避免这些错误包首次被发送出去。作为一个额外的收获，你还可以了解更多你可能不会与之互动的基础设施部分，这总是很酷的。:-)

有趣的是，如果你在谷歌上搜索这些数字，你会找到很多帖子，里面的人们在实际中遇到了它们。也许我们以后在一些错误路径上打印 ASCII 会更有帮助，当所有字符值都在 0x0 和 0x7F 之间时。我相信这会帮助很多人更快地意识到发生了什么。也许我会发送一个补丁到上游，在 `svc_tcp_recv_record` 和内核中直接解析前几个数据字节作为整数的其他地方，让我们看看。

这是一个可以生成一堆其他 HTTP 整数的简单程序，可能会引起兴趣：

```
#include <assert.h> #include <byteswap.h> #include <inttypes.h> #include <limits.h> #include <stdio.h> #include <string.h>  
static int system_is_little_endian(void) {
    static const int tmp = 1;
    return *(const char *)&tmp == 1;
}

#define print_reinterpreted_inner(type, fmt, bs_func, hdr)      \
    do {                                                        \
        if (strlen(hdr) >= sizeof(type)) {                      \
            type *_hdr_conv = (type *)hdr;                      \
            type _le, _be;                                      \
            if (system_is_little_endian()) {                    \
                _le = *_hdr_conv;                               \
                _be = bs_func(*_hdr_conv);                      \
            } else {                                            \
                _le = bs_func(*_hdr_conv);                      \
                _be = *_hdr_conv;                               \
            }                                                   \
            printf("%.*s,%zu,%" fmt ",%" fmt "\n",              \
                   (int)strlen(hdr) - 2, hdr, sizeof(type),     \
                   _le, _be);                                   \
        }                                                       \
    } while (0) 
#define print_reinterpreted(bits, hdr)                          \
    print_reinterpreted_inner(uint##bits##_t, PRIu##bits,       \
                              bswap_##bits, hdr) 
int main(void) {
    const char *methods[] = {"GET",   "HEAD",   "POST",
                             "PUT",   "DELETE", "OPTIONS",
                             "TRACE", "PATCH",  "CONNECT"};
    size_t i;

    printf("data,bytes,little-endian,big-endian\n");

    for (i = 0; i < sizeof(methods) / sizeof(methods[0]); i++) {
        int ret;
        char hdr[16];
        unsigned const char *check =
            (unsigned const char *)methods[i];

        /* No high bit, so no need to check signed integers */
        assert(!(check[0] & (1U << (CHAR_BIT - 1))));

        ret = snprintf(hdr, sizeof(hdr), "%s /", methods[i]);
        assert(ret > 0 && ret < (int)sizeof(hdr));

        print_reinterpreted(64, hdr);
        print_reinterpreted(32, hdr);
        print_reinterpreted(16, hdr);
    }
}
```

以及结果：

| 数据 | 字节 | 小端 | 大端 |
| --- | --- | --- | --- |
| 获取 | 4 | 542393671 | 1195725856 |
| 获取 | 2 | 17735 | 18245 |
| 头 | 4 | 1145128264 | 1212498244 |
| 头 | 2 | 17736 | 18501 |
| 发送 | 4 | 1414745936 | 1347375956 |
| 发送 | 2 | 20304 | 20559 |
| 放置 | 4 | 542397776 | 1347769376 |
| 放置 | 2 | 21840 | 20565 |
| 删除 | 8 | 3395790347279549764 | 4919422028622405679 |
| 删除 | 4 | 1162626372 | 1145392197 |
| 删除 | 2 | 17732 | 17477 |
| 选项 | 8 | 2329291534720323663 | 5715160600973038368 |
| 选项 | 4 | 1230262351 | 1330664521 |
| 选项 | 2 | 20559 | 20304 |
| 跟踪 | 4 | 1128354388 | 1414676803 |
| 跟踪 | 2 | 21076 | 21586 |
| 补丁 | 4 | 1129595216 | 1346458691 |
| 补丁 | 2 | 16720 | 20545 |
| 连接 | 8 | 2329560872202948419 | 4850181421777769504 |
| 连接 | 4 | 1313754947 | 1129270862 |
| 连接 | 2 | 20291 | 17231 |

这是一个包含所有[IANA 知道的](https://www.iana.org/assignments/http-methods/http-methods.xhtml)方法的可展开完整列表：

<details><summary>| 数据 | 字节 | 小端 | 大端 |

| --- | --- | --- | --- |
| --- | --- | --- | --- |
| 访问控制 | 4 | 541868865 | 1094929440 |
| 访问控制 | 2 | 17217 | 16707 |
| 基线控制 | 8 | 4994009628729884994 | 4774188637087157829 |
| 基线控制 | 4 | 1163084098 | 1111577413 |
| 基线控制 | 2 | 16706 | 16961 |
| 绑定 | 4 | 1145981250 | 1112100420 |
| 绑定 | 2 | 18754 | 16969 |
| 签入 | 8 | 2327878644997113923 | 4848201154192559648 |
| 签入 | 4 | 1128613955 | 1128809795 |
| 签入 | 2 | 18499 | 17224 |
| 结账 | 8 | 6076850456876107843 | 4848201154192954708 |
| 结账 | 4 | 1128613955 | 1128809795 |
| 结账 | 2 | 18499 | 17224 |
| 连接 | 8 | 2329560872202948419 | 4850181421777769504 |
| 连接 | 4 | 1313754947 | 1129270862 |
| 连接 | 2 | 20291 | 17231 |
| 复制 | 4 | 1498435395 | 1129271385 |
| 复制 | 2 | 20291 | 17231 |
| DELETE | 8 | 3395790347279549764 | 4919422028622405679 |
| DELETE | 4 | 1162626372 | 1145392197 |
| DELETE | 2 | 17732 | 17477 |
| GET | 4 | 542393671 | 1195725856 |
| GET | 2 | 17735 | 18245 |
| HEAD | 4 | 1145128264 | 1212498244 |
| HEAD | 2 | 17736 | 18501 |
| LABEL | 4 | 1161969996 | 1279345221 |
| LABEL | 2 | 16716 | 19521 |
| LINK | 4 | 1263421772 | 1279872587 |
| LINK | 2 | 18764 | 19529 |
| LOCK | 4 | 1262702412 | 1280262987 |
| LOCK | 2 | 20300 | 19535 |
| MERGE | 4 | 1196574029 | 1296388679 |
| MERGE | 2 | 17741 | 19781 |
| MKACTIVITY | 8 | 5284491839020288845 | 5569617121606456905 |
| MKACTIVITY | 4 | 1128352589 | 1296777539 |
| MKACTIVITY | 2 | 19277 | 19787 |
| MKCALENDAR | 8 | 4921947636577291085 | 5569619311905295940 |
| MKCALENDAR | 4 | 1094929229 | 1296778049 |
| MKCALENDAR | 2 | 19277 | 19787 |
| MKCOL | 4 | 1329810253 | 1296778063 |
| MKCOL | 2 | 19277 | 19787 |
| MKREDIRECTREF | 8 | 4995135494276926285 | 5569635821625627205 |
| MKREDIRECTREF | 4 | 1163021133 | 1296781893 |
| MKREDIRECTREF | 2 | 19277 | 19787 |
| MKWORKSPACE | 8 | 5788052762991741773 | 5569641362368451408 |
| MKWORKSPACE | 4 | 1331120973 | 1296783183 |
| MKWORKSPACE | 2 | 19277 | 19787 |
| MOVE | 4 | 1163284301 | 1297045061 |
| MOVE | 2 | 20301 | 19791 |
| OPTIONS | 8 | 2329291534720323663 | 5715160600973038368 |
| OPTIONS | 4 | 1230262351 | 1330664521 |
| OPTIONS | 2 | 20559 | 20304 |
| ORDERPATCH | 8 | 6071222086951785039 | 5715705941611004244 |
| ORDERPATCH | 4 | 1162105423 | 1330791493 |
| ORDERPATCH | 2 | 21071 | 20306 |
| PATCH | 4 | 1129595216 | 1346458691 |
| PATCH | 2 | 16720 | 20545 |
| POST | 4 | 1414745936 | 1347375956 |
| POST | 2 | 20304 | 20559 |
| PRI | 4 | 541676112 | 1347569952 |
| PRI | 2 | 21072 | 20562 |
| PROPFIND | 8 | 4921952009106444880 | 5787775677319695940 |
| PROPFIND | 4 | 1347375696 | 1347571536 |
| PROPFIND | 2 | 21072 | 20562 |
| PROPPATCH | 8 | 4851574511785431632 | 5787775677486945347 |
| PROPPATCH | 4 | 1347375696 | 1347571536 |
| PROPPATCH | 2 | 21072 | 20562 |
| PUT | 4 | 542397776 | 1347769376 |
| PUT | 2 | 21840 | 20565 |
| REBIND | 8 | 3395789222064571730 | 5928217367116259375 |
| REBIND | 4 | 1229079890 | 1380270665 |
| REBIND | 2 | 17746 | 21061 |
| REPORT | 8 | 3395806831532066130 | 5928232786117009455 |
| REPORT | 4 | 1330660690 | 1380274255 |
| REPORT | 2 | 17746 | 21061 |
| SEARCH | 8 | 3395793573017371987 | 6000273900112977967 |
| SEARCH | 4 | 1380009299 | 1397047634 |
| SEARCH | 2 | 17747 | 21317 |
| TRACE | 4 | 1128354388 | 1414676803 |
| TRACE | 2 | 21076 | 21586 |
| UNBIND | 8 | 3395789222064574037 | 6146923424020439087 |
| UNBIND | 4 | 1229082197 | 1431192137 |
| UNBIND | 2 | 20053 | 21838 |
| UNCHECKOUT | 8 | 5713734517093781077 | 6146924519086050127 |
| UNCHECKOUT | 4 | 1212370517 | 1431192392 |
| UNCHECKOUT | 2 | 20053 | 21838 |
| UNLINK | 8 | 3395796918646623829 | 6146934419137175599 |
| UNLINK | 4 | 1229737557 | 1431194697 |
| UNLINK | 2 | 20053 | 21838 |
| UNLOCK | 8 | 3395796871502646869 | 6146934444722429999 |
| UNLOCK | 4 | 1330400853 | 1431194703 |
| UNLOCK | 2 | 20053 | 21838 |
| UPDATE | 8 | 3395790347211919445 | 6147488538738106415 |
| UPDATE | 4 | 1094996053 | 1431323713 |
| UPDATE | 2 | 20565 | 21840 |
| UPDATEREDIRECTREF | 8 | 4995131164881866837 | 6147488538738119237 |
| UPDATEREDIRECTREF | 4 | 1094996053 | 1431323713 |
| UPDATEREDIRECTREF | 2 | 20565 | 21840 |
| VERSION-CONTROL | 8 | 3264633956239295830 | 6216465378320535085 |
| VERSION-CONTROL | 4 | 1397900630 | 1447383635 |

| VERSION-CONTROL | 2 | 17750 | 22085 |

顾名思义，如果你在谷歌搜索大部分这些数字，你会发现一个无尽的问题列表，其中提到了它们在错误消息中（我曾回答过的一些先前未识别的例子：[1](https://github.com/inconshreveable/ngrok/issues/545), [2](https://stackoverflow.com/a/59730358/945780), [3](https://github.com/ehang-io/nps/issues/315))。

希望这篇文章能帮助人们更快地找到他们*真正*的问题 - 使用错误的协议 - 未来更快。特别是，如果你的受影响应用由于 ENOMEM/"内存不足"而崩溃，请考虑提交一个将大小限制在一些合理最大值的补丁 :-)
