<!--yml

类别：未分类

日期：2024-05-27 15:01:42

-->

# 使用 LLaVA 和 pgvector 对 iCloud 照片进行索引 | 作者 Mustafa Akin | Medium

> 来源：[`medium.com/@mustafaakin/indexing-icloud-photos-with-ai-using-llava-and-pgvector-fd58182febf6`](https://medium.com/@mustafaakin/indexing-icloud-photos-with-ai-using-llava-and-pgvector-fd58182febf6)

# 使用 LLaVA 和 pgvector 对 iCloud 照片进行索引

## 一个直接的想法，将东西粘在一起直到它能够工作，但这只是在未来可能借助本地人工智能的一瞥

我一直对人工智能的崛起感到着迷。然而，大多数时间，它感觉像魔术一样。我尽可能地想深入了解事物的底层。我在计算机科学本科最喜欢的课程是数字设计、计算机体系结构、操作系统和网络，我在其中排名前 1%（除了网络，该死的电子学生）。我非常喜欢它们。尽管我父亲在我大约 10 岁时教我 Visual Basic 和数据库，但是参加这些课程最终让我豁然开朗。这感觉就像 Neo 最终看到了矩阵只是垂直的 `<marquee>` 中的绿色代码，但我的启示版本要逊色得多：“这就是 64 位 CPU 的意思”和“哦，原来超线程一直都意味着 2 倍的 CPU”，以及“我知道 for 循环的顺序对于某些架构的最佳缓存性能是重要的”，“哦，互联网是 1500 字节的消息交换。奇迹般地，它在这个规模上能够工作”等等。

但是，对我来说，人工智能改变了一切。坦率地说，我几乎不知道最新发展的情况。这来自于一个计算机科学博士辍学者（*部分是因为军事义务延期，部分是自我发现过程*），他别无选择，只能选修几门经典（换句话说，枯燥）的人工智能和机器学习课程，因为吹得很厉害，几乎没有认真的教授愿意研究操作系统、分布式系统和网络（这是我最喜欢的），而只有像生物信息学这样的闪亮东西，一篇论文里有 50 个人和老旧的机器学习。

现代人工智能的工作方式，你可以从一个单一文件中下载一堆它们，并在 M1 机器中的不到 8GB RAM 中运行，真是令人惊叹。但是，由于大多数人不需要整天处理 CPU 架构，我们实际上不需要真正理解矩阵乘法是如何最终导致某种形式的人工智能的。

有一段时间，我想尝试一下 LLMs 和 RAG 概念，为 [Resmo](https://mustafaakin.dev/posts/2024-01-08-indexing-icloud-photos-with-ai-using-llava/www.resmo.com) 的内部使用案例，但我没有强大的用例。让我这次尝试一下。在您的网站上添加一个由 AI/LLM 提供支持的聊天机器人进行客户支持，对文档进行摘要或将自然语言转换为 SQL 只是略微有用；但是，是的，这可能让您说您的产品中有“人工智能”。

为了避免再多废话，简而言之，我的业余项目是利用一个可以理解图像的多模式 LLM，并改进 iCloud 照片档案中的语义搜索。苹果照片已经可以识别图像上的物体并对图像进行全文搜索。然而，它只能检测到物体和颜色。我发现谷歌照片搜索效果要好得多。

# 描述 iCloud 照片使用 LLM

正如我所说，在我的硕士和博士期间，我不得不修过几门 ML/AI 课程，并更加了解到有最先进的标注和图像分割算法，其性能非常好，比 LLM 工作效率要高得多。但我想尝试这个想法。

我们可以问一个 LLM 它在图像中看到了什么，并使用一种流行的算法将响应嵌入为一个向量，然后让用户在上面搜索？并不是最先进的，但这是一个有趣的案例。显然，性能将取决于 LLM 的识别，但一个开源模型能否足够好地搜索我的照片呢？如果足够好，它能否演变成其他东西？让我们深入探讨。

起步时，如果你正在使用 iCloud 照片，你所有的照片缩略图都可以在一个本地目录中进行搜索，即使所有的照片由于存储问题而不在你的 Mac 上。你的照片还有一个 SQLite 数据库，我在 2019 年的描述性博客中有介绍，但是模式似乎已经改变了，你在[Simon Willison 的博客](https://simonwillison.net/2020/May/21/dogsheep-photos/)上可能会有更好的运气。

为了保持简单，我编写了一个代码，递归列出了 iCloud 缩略图文件夹中的 JPEG 文件，这些文件可以在**`~/Pictures/Photos Library.photoslibrary/resources/derivatives**/找到。它们是原始照片的缩小版本，但对于我们的用例来说已经足够了。作为 LLM 模型，我选择了带有 Q4 的 LLaVA，并使用了只需执行的[llamafile](https://github.com/Mozilla-Ocho/llamafile)，这样部署就变得非常容易了。它具有一个我可以调用的 REST API，我所需要做的就是对图像进行编码。

# 提示

我不喜欢术语提示工程。在我看来，这不是一个足够深入的主题，有好几个分支都可以称为真正的工程，这感觉像是对真正的工程的侮辱。如果我们称每次试错而没有真正理解根本原因的东西为工程，那么就会贬低真正工程的价值。

我尝试了几个提示，以了解哪一个能为图片提供更好的描述。考虑到每一代在我的 M1 Max 64GB 机器上使用 LLaVA 7B Q4 模型大约需要 10 秒（我使用了 REST API，我相信它可能会更好），我没有太多的选择。由于我不再是博士研究生，我不会发布任何违背基准的东西，但我将分享一些使用不同提示和不同温度生成的 LLaVA 示例，并比较 7B 和 13B 参数模型。

## 什么关于 ChatGPT？

当然，GPT4V（带有视觉模型的 ChatGPT）能够以极大的细节生成我的图片的完美描述。但是以什么代价呢？所有博客和 YouTube 上的 LLM 和 RAG 示例**默认使用 OpenAI**，这让我**伤心**。不要误会，我很感激它们的存在，但是能够在你的计算机上运行带有视觉的 LLM 是很惊人的，无论他们做了什么，我们都不应该依赖于一家公司。我是一位老 Linux 用户，买不起 Macbook，没有 Linux 的话，一个免费的操作系统和一个包管理器，我的编程技能不会进步得这么快（我觉得 Windows 阻碍了很多细节）。所以不要成为一个对估值 1000 亿美元的公司唯命是从的人，支持开源 LLM，不要默认为专有。我感激 OpenAI 为推动这个领域所做的工作，但是甚至有一些特殊的 LLM 甚至可以在边缘工作。我也不想把整个相册上传到一个隐私做法可疑的公司。

+   **提示 1:** “用户与人工智能助手之间的聊天。助手对人类的问题给出详细的答复。

    用户：[img-10]详细描述这张图片

    **助手:**”

+   **提示 2:** “简洁的图像摘要请求。

    用户：[img-10]请简要、简洁地描述这张图片，只突出显示最重要的元素，用几个词。

    **助手:**”

+   **提示 3:** “详细图像分析对话。

    用户：[img-10]我需要对这张图片进行彻底的分析，包括所有元素、颜色和任何显著的特征。

    **助手:**”

+   **提示 4:** “图像分析和描述的交互式会话。

    用户：[img-10]请提供这张图片的全面描述，重点介绍所有可见的细节。

    **助手:**”

+   **ChatGPT**: 详细描述这张图片

## 图片 1：飞越球体

这是一个令人惊叹的结构。在重新:Invent 之后的 2023 年 12 月拍摄并感染流感

+   **提示 1**：这张图片捕捉到了一个独特的机场高空视角，机场跑道上停放着多架大型客机。除了这些飞机外，周围还可以看到几辆汽车和卡车，很可能是为了服务于各种目的，比如运输或飞机维护。场景还包括了两个靠近机场的高尔夫球场，一个位于图片中心附近，另一个则更靠左侧。这些球场为繁忙的机场环境提供了对比的元素，并为该地区的游客提供了娱乐机会。飞机、汽车、卡车和绿色高尔夫球场的结合创造了一个有趣的城市生活快照，航空基础设施与之交织在一起。

+   **提示 2**：一架飞机飞过高尔夫球场。

+   **提示 3**：这张图片展示了一个城市景观，一架飞机在头顶飞过，为城市景观提供了独特的视角。一个大型高尔夫球场在下方可见，为场景增添了绿意。整个区域散布着许多汽车和卡车，表明这很可能是白天，人们在通勤或在城市周围旅行。除了这些元素外，城市各处还有几个交通灯，帮助规范下方街道上的车辆流量。一架大型蓝色飞机在上空翱翔，为这张城市景观照片增添了一种有趣的元素。总的来说，这是一张引人入胜的图片，展示了自然与人造结构共存于同一环境中的情景。

+   **提示 4**：这张图片是一个高空视角的高尔夫球场，有许多树木和绿草。附近停着几辆车，还有一些建筑物环绕着这个区域。除了陆地上的车辆外，还可以看到一架飞机飞过高尔夫球场。这个独特的视角既提供了人们在地面上玩耍或参观球场的活动的见解，又提供了从空中俯瞰的视角。这个场景是自然与城市元素的融合，为那些喜欢户外活动如高尔夫的人们创造了一种风景如画的环境。

+   **ChatGPT**：图片展示了从飞机上俯瞰的风景，展示了一个被温暖阳光照射的广阔城市景观。在前景中，我们看到飞机的机翼，给人一种高度和运动的感觉。在下方，城市密集地布满各种建筑物，包括似乎是大型酒店综合体的建筑，这表明这是一个以旅游业闻名的大都市地区。有一个引人注目的建筑物，有一个弯曲的蓝色玻璃屋顶，与周围的建筑物形成鲜明对比。郁郁葱葱的绿地，可能是高尔夫球场或公园，与城市环境形成对比，有修剪整齐的草坪和点缀其中的树木。道路和高速公路穿过城市，车辆的微小点缀表明了城市生活的喧嚣。城市以外的地形变得更加贫瘠，表明城市周围是一种类似沙漠的环境。阳光从建筑物投下长长的阴影，表明是日出或日落，增添了场景的宁静氛围。

## 图片 2：贝尔格莱德圣萨瓦大教堂

你可以看到我妻子和女儿在摆姿势。这是我们可以带着一个两岁孩子最少麻烦直飞安卡拉而不需要签证的最好的地方。

+   **提示 1**：图片展示了一座大型、华丽的教堂，有两座突出的塔楼和一个令人印象深刻的圆顶。教堂外有几个人在周围走动，一些人提着手提包或背包，探索周围环境。一些人可以看到站在入口附近，而其他人则沿着附近的人行道走动。整体氛围显得宁静祥和，游客们享受着在这个历史地标上的时光。教堂的建筑增添了场景的壮丽气息，使其成为那些欣赏其美丽和历史意义的人们吸引的目的地。”

+   **提示 2**：图片显示了一座大教堂，前面有人们在走动。一个男人提着一个粉色的包，而其他人则在该区域周围走动，可能是在参观或参加教堂的活动。附近还有一张长椅，供那些希望在访问期间坐下来放松的人使用。

+   **提示 3**：“图片描述了一座宏伟的白色教堂，有两个突出的圆顶和一个华丽的钟楼。人群聚集在建筑物外面，走动或站在附近。在场景中至少可以看到 14 个人以不同的姿势。其中一些人肩上背着背包，这表明他们可能是在参观教堂的游客。整体氛围似乎是繁忙的一天，人们在壮观建筑物外从事不同的活动。”

+   **提示 4**：图像显示了一座大型的白色教堂，拥有装饰华丽的圆顶屋顶和几座高大的，狭窄的塔楼。周围绿树成荫，为景色增添了美感。一群人可以在教堂外面漫步，享受这个风景如画的场景。总共有 13 个人在整个图像中可见；有些人靠近前景，而其他人则更远。一个人在教堂附近的区域走动时拿着一个手提包。这个场景捕捉到了在这座壮观建筑前的社区活动和悠闲氛围。

+   **ChatGPT**：这张图像描绘了一座宏伟教堂的正面视图，天空晴朗，只有几朵薄云。教堂有一个带有十字架的大圆顶，并被较小的圆顶环绕，所有的圆顶都有绿色的铜锈。立面是白色的，有装饰性细节和多个拱形入口，入口上方是金色的马赛克或绘画。教堂前有一个宽敞的广场，上面散落着几个人。一位穿着黑色 T 恤和短裤的男子走在前景，拿着一个粉红色的文件或文件夹，而在右边，一对夫妇并肩行走。在左边，一群人似乎在排队，可能是为了进入教堂。广场区域还可见各种树木和带有铭文的纪念碑。这是一个阳光明媚的日子，整体氛围平静而悠闲。

## 图像 3：酒店房间里的两个行李箱。

这是来自 re:Invent 的再次提醒，[Venetian Vegas](https://www.venetianlasvegas.com/)，具有非常独特的装饰。

+   **提示 1**：图像显示了两个箱子放在地板上并排放置。其中一个较大，占据了大部分前方空间，而较小的一个则站在它旁边。这两个箱子都是带有手柄的轮胎箱，方便携带。它们似乎是靠在墙壁或角落旁边并排坐着，可能在等待被拿走或放置到其他地方。整个场景表明，可能有人最近到达了目的地或正在准备旅行。

+   **提示 2**：两件行李箱并排坐着。

+   **提示 3**：图像显示了一个酒店房间，地板上放着两个箱子。其中一个较大，带有一个手柄，而另一个则看起来较小，可能是破损或分解成多个部分。这两个箱子都放在房间里靠近墙壁的位置。房间本身包含各种元素，如位于图像右侧的床，左侧的另一件较小的家具以及在房间远端靠近一起的两把椅子。此外，场景中还放置了一把伞。这些物品的存在表明，这个酒店房间可能用于临时住宿或旅行目的。

+   **提示 4**：图像显示了一个走廊，两个行李箱并排放置在一起。一个行李箱较大，是灰色的，而第二个看起来较小。两个行李箱都靠在一起，方便携带，配有轮子。附近还可以看到一个手提包。除了行李之外，场景左侧还有一个背包，进一步说明这可能是旅客或通勤者使用的区域。走廊本身的墙壁上有各种图案和颜色，为过路人营造了视觉上令人愉悦的环境。

+   **ChatGPT**：图像显示了两个行李箱放置在瓷砖地板上，靠墙摆放，墙上有条纹壁纸图案。在左边，有一个较小的、带图案的随身箱，设计为黑白交错的千鸟格。它有一个伸缩手柄，似乎有两个前袋，一个比另一个大。箱子上还挂有一个小黑色标签。在它旁边，右边是一个较大的硬壳行李箱，具有凹凸设计。它是浅色的，可能是灰色或米白色，也配有伸缩手柄和四个万向轮，这是为了便于移动的常见特点。地板由拼花的石砖制成，混合了米色、米白色和黑色大理石图案。行李箱后面的墙壁上装饰着双色条纹壁纸，交替着米色和较深的棕褐色。条纹是垂直的，墙壁上的壁板下有一个木制椅子扶手。行李箱在墙壁和地板上投下了轻微的阴影，表明房间里有环境光。

## 图像 4：我的女儿在沙地上玩耍，带着玩具。

她真是一个户外爱好者。

+   **提示 1**：图像展示了一个年幼的女孩在土路上玩耍。她被各种玩具围绕着，包括散落在她周围地面上的几辆车和卡车。场景右侧可以看到两个瓶子，而另一辆车和一把勺子则放在中心位置附近。女孩似乎在这个有趣的环境中享受着自己和她周围的沙土创作。

+   **提示 2**：一个小女孩在沙地上玩耍。

+   **提示 3**：图像描述了一个年幼的女孩在公园的沙地上玩耍。她跪坐在地上，周围散落着各种玩具，如汽车和卡车。附近还有两只泰迪熊，一只靠近场景的左侧，另一只靠近右侧。这个孩子似乎在享受这些玩具，同时也在沙地上玩得很开心。

+   **提示 4**：图像描绘了一个小女孩在沙地上玩耍，周围散落着各种玩具。 她穿着粉色衣服，似乎专注于玩耍。 她周围有几辆散落的汽车，以及附近的几辆卡车。 此外，有一个手提包放在其中一辆车旁边。 场景表明，这个小女孩可能在沙地上玩耍，周围有各种各样的玩具和娱乐物品。

+   **ChatGPT**：这张图显示了一个小孩子在沙箱里玩耍。 这个孩子，看起来是个蹒跚学步的孩子，穿着粉色的衣服，可能是扎着辫子的头发。 周围散落着各种彩色的塑料玩具，包括一个黄色的卡车、一个红色的铲子、一个蓝色的桶和其他各种各样的沙玩具。 沙地区域一侧被草边界线环绕着，可以看到一小部分白色的边界标志或线条。 这看起来像是一个户外游乐区，可能是在后院或公园里。 沙箱外的地面上长满了草，背景中可以看到几棵年轻的树，用支撑棍支撑着。 这是一个阳光明媚的日子，可以看到孩子的影子投在沙地上，表明是早上或下午。

# 嵌入和 pgvector

我记得 word2vec 在我大学时代的出现，那时感觉就像魔法一样，现在依然如此。 正如我所说，我不需要理解它在底层如何工作的所有细节（但如果了解会非常有帮助），只需要了解它的基本原理。 给定一个字符串，嵌入模型将其转换为一个向量，您可以运行一个简单的距离算法来查找相关的或与您的查询相似的字符串。 即使我不在乎模型是如何训练的和嵌入是如何工作的，通过抽象出这一部分，我也可以继续我的生活，并进行多维向量相似度。 我将我的应用程序状态存储在我最喜欢的数据库 Postgres 中，使用 pgvector 扩展，查询相似性非常容易。

# 生成描述、嵌入和查询它们的 Python 代码

这只是在我的 iCloud 照片库中迭代文件，使用 HTTP API 提示本地 LLaVA。

```
import os
import psycopg2

from pgvector.psycopg2 import register_vector
conn = psycopg2.connect(user="mustafa", password="", database="postgres")
register_vector(conn)

prompt = "Detailed image analysis dialogue.\nUSER:[img-10] I need a thorough analysis of this image, including all elements, colors, and any noticeable features.\nASSISTANT:"
model = SentenceTransformer('sentence-transformers/all-MiniLM-L6-v2')

for root, dirs, files in os.walk("/Users/mustafa/Pictures/Photos Library.photoslibrary/resources/derivatives/"):
    for name in files:
        x = os.path.join(root, name)
        filename, ext = os.path.splitext(x)
        if ext == ".jpeg":
          response = llm(prompt, x)
          description = response['content']
          embeddings = model.encode(response['description'])
          cur.execute("INSERT INTO results (filename, prompt, model, description, embedding) VALUES(%s, %s, %s, %s)",
                      (x, prompt, json.dumps(response), description), embeddings)
          conn.commit()
```

对于查询数据，我们使用相同的嵌入模型，只需询问 pgvector 带来最接近的向量。 我没有使用任何索引，因为数据已经很小，但是 pgvector 支持像 [IVFFlat](https://github.com/pgvector/pgvector#ivfflat) 和 [HNSW](https://github.com/pgvector/pgvector#hnsw) 这样的索引，以加快检索速度，并对正确性造成小小的影响。 但由于这些只是图像描述，我相信这不会有太大影响。

```
import ipyplot
from PIL import Image

query = "inside of a shopping mall"
embeddings = model.encode(query)
cur.execute("SELECT filename, description, embedding <-> %s as score FROM results ORDER BY embedding <-> %s LIMIT 3", (embeddings, embeddings))
rows = cur.fetchall()
images = []
labels = []
for row in rows:
    img = Image.open(row[0])
    images.append(img)
    desc = row[1]
    score = row[2]
    labels.append(f'{score:.4f}\n{desc}')

ipyplot.plot_images(images, labels, max_images=30, img_width=350, zoom_scale=1)
```

最后，我使用了 [ipyplot](https://github.com/karolzak/ipyplot) 库来显示一个图像网格，带有它们的标签和与我的查询的距离。 我已经用尽了我的 Python 知识，迫不及待地想回到写一些 Kotlin。

# 结果

嗯，在我的约 4000 张图像数据集中，结果出奇地好。我尝试了不同的量化，从 LLaVA 7B-Q4 到 LLaVA 13B-FP16，对于我的用例来说并没有太大关系。当然，如果这是一个官方基准测试，并且我们有一个实际的数据集进行比较，我相信更大的模型会导致稍微更高的分数，但它的性能要低得多，对于我的数据集来说不值得去折腾。而且这只是一个开始。搜索非常天真。

## **查询 1：“**黑色汽车”和“白色汽车”

我想先测试简单的查询。汽车是我相册中最常见的主题之一，我想测试它是否可以通过一个非常简短的查询来区分黑色和白色。

“黑色汽车” — 嗯，所有的汽车都是黑色的。第一张是一辆非常酷的汽车，进行漂移。在最后一张图片中，它甚至从反射中检测到了一个橙色交通锥，这令人印象深刻。

“白色汽车” — 你可以看到在第一张图片中它的幻觉很多，因为没有太多可说的。如果 AI 能够自信而不虚构，那将会非常棒。

## 查询 2：“雨夜”

没有一张照片是在夜晚拍摄的。但是有一些雨因素和一点黑暗，所以还不错。

第一张是雪而没有雨。第二和第三张是雨天。

## 查询 3：“运输货物”

我选择了这个因为它可以被视为一个抽象的主题。令人惊讶的是，所有的图片都以独特的方式“运输”一些“东西”。然而第二张图片让人感到困扰，我已经向有关部门报告了。

是的，这是一个孩子坐在一辆非常危险的车里。

## 查询 4：“幼儿在外面玩耍，阳光明媚，青草茵茵”

在第一张照片中，看不到太阳和阴影，但它是一片繁茂的绿色，所以 LLm 将其总结为夏天，这是匹配的。

## 查询 5：“丰富多彩的夜晚”与“黑夜”

我试过这个查询，知道它会匹配拉斯维加斯。如果我在寻找更具体的事物，我认为它会更有效，因为你可以看到通过最少的幻觉生成的广泛描述。如果有很多对象，LLaVA 可以生成漂亮的

“丰富多彩的夜晚” — 当然就是拉斯维加斯了。

“黑夜” — 黑暗而可怕的天空，充满不确定性。

## 查询 6：“网络交换机”

当然，我的照片集中包含各种网络设备。 LLaVA 认为中间图片中的管道是彩色电缆。对于第三张图片，我感到抱歉。

对于第三张照片造成的视觉不适，我真的很抱歉。

# 接下来呢？

我喜欢写博客。我喜欢分享我的见解和经验，这样可以激励其他人或者给出一个想法。如果我看到任何想法在这里被实际建成产品，我将会非常高兴。这是我的目标。我没有时间或相关经验来进一步追求这个想法。否则，它不会是一个博客，而会是一个演示文稿。

到目前为止，我分享的内容仍然有可能改进。该产品的用户可能会搜索非常具体的东西，比如“圣诞节上的蓝色婴儿 T 恤”，而不是一大段数据。然而，LLM 的输出比典型的用户查询要长得多。我不太确定是否明智地使用向量嵌入来处理非常不同长度的字符串。

**可能的改进**

+   将图像的元数据（如位置、拍摄日期）整合进来

+   使用人脸识别和（如果可用）人名

+   使用简短的提示词对图像进行分类，聚类共同主题

+   使用多个提示来独立描述对象、场景、情感、颜色和其他具体主题，而不是一种通用的描述，以保持一致性，并确保查询可以利用这种分段

+   对结果进行重新排名，结合经典的信息检索技术

+   使用不同的量化方法进行测试，加速过程
