<!--yml

类别：未分类

日期：2024-05-27 14:51:44

-->

# 如何使用ChatGPT Api和Node.js总结Youtube视频

> 来源：[https://implementing.substack.com/p/how-to-summarize-youtube-video-using](https://implementing.substack.com/p/how-to-summarize-youtube-video-using)

嗨，我是[Marco](https://www.linkedin.com/in/marco-moauro-b49489162/)，欢迎来到我的通讯！

作为一名软件工程师，我创建了这个通讯来分享我在开发领域的第一手知识。我们将探讨的每个主题都将提供宝贵的见解，目标是启发和帮助您在您的旅程中前行。

在本期中，我想为您带来第一个教程，介绍如何在Node.js中制作一个系统，从YouTube视频链接开始，使用OpenAI的[完成API](https://platform.openai.com/docs/guides/text-generation/chat-completions-api)，这与ChatGPT系统所基于的API相同，生成摘要。

> 您可以直接从我的Github存储库下载所有显示的代码：[https://github.com/marcomoauro/youtube-summarizer](https://github.com/marcomoauro/youtube-summarizer)

[订阅Implementing ✨](https://implementing.substack.com/subscribe)

* * *

系统架构主要包括：

1.  从YouTube视频中提取文本

1.  生成文本摘要

此过程涉及从视频中提取文本并利用它进行摘要生成。考虑了各种选项，包括：

所选解决方案涉及爬取，这是各种选项中最具挑战性的。这一决定的动机在于，独立实现所有内容不涉及与文本提取的第三方API相关的费用。此外，作为一个真正的爱好者，我更倾向于这种方法，因为我大部分的个人项目都建立在这种技术基础之上。

> 如果您有兴趣了解有关网络爬取的最佳实践，请订阅我们的新闻通讯，我将很快发布一篇相关文章。

收到字幕后，我们将它们输入到OpenAI中。我面临的第一个挑战是完成API能处理的文本最大大小的限制。此限制取决于所使用的模型；对于3.5 turbo模型，具体设置为4,000个标记。

为了克服这一限制，我采用了递归方法。将文本分成较小的部分，然后将它们合并成组并独立进行总结；此过程重复进行，直到生成单一的输出文本，对应于从中间摘要生成的最终摘要。

[订阅Implementing ✨](https://implementing.substack.com/subscribe)

* * *

对于本教程，您需要安装Yarn和Node.js，具体来说我使用的是LTS版本20.9.0。如果您的计算机上没有安装Node.js，您可以从[官方网站](https://nodejs.org/en)安装。

从我的工作空间文件夹开始，我创建了项目文件夹和npm包：

```
mkdir youtube-summarizer
cd youtube-summarizer
npm init -y 
```

添加“type: module”子句以使用ES6语法：

```
`{
  "name": "youtube-summarizer",
  "version": "1.0.0",` **"type": "module",** `"description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}`
```

我们需要的库包括 **Axios**，用于发起 HTTP 调用，我们需要实现对完成的调用，**he** 和 **striptags** 用于操作 HTML，**p-queue** 用于处理 Promise 的队列，以便同时最多调用 X 次 OpenAI，以及 **yargs** 用于构建交互式命令行工具。

安装它只需键入：

```
yarn add axios
yarn add he
yarn add striptags
yarn add p-queue
yarn add yargs
```

创建名为 **getSubtitleFromVideo.js** 的文件：

```
touch getSubtitleFromVideo.js
```

我们将创建的初始函数旨在使用网页抓取获取 YouTube 视频页面的 HTML 内容。让我们称此函数为 **getHTML**。

```
const getHTML = async (video_id) => {
  const {data: html} = await axios.get(`https://youtube.com/watch?v=${video_id}`);

  return html
}
```

一旦获取到 HTML，我们可以通过这个 **getSubtitle** 函数检索字幕：

```
const getSubtitle = async (html) => {
  if (!html.includes('captionTracks')) {
    throw new Error(`Could not find captions for video`);
  }

  const regex = /https:\/\/www\.youtube\.com\/api\/timedtext[^"]+/;

  const [url] = html.match(regex);
  if (!url) throw new Error(`Could not find captions`);

  const obj = *JSON*.parse(`{"url": "${url}"}`)

  const subtitle_url = obj.url

  const transcriptResponse = await axios.get(subtitle_url);
  const transcript = transcriptResponse.data;

  const lines = transcript
    .replace('<?xml version="1.0" encoding="utf-8" ?><transcript>', '')
    .replace('</transcript>', '')
    .split('</text>')
    .filter(line => line && line.trim())
    .map(line => {
      const startRegex = /start="([\d.]+)"/;
      const durRegex = /dur="([\d.]+)"/;

      const startMatch = startRegex.exec(line);
      const durMatch = durRegex.exec(line);

      const start = startMatch[1];
      const dur = durMatch[1];

      const htmlText = line.replace(/<text.+>/, '').replace(/&amp;/gi, '&').replace(/<\/?[^>]+(>|$)/g, '');
      const decodedText = *he*.decode(htmlText);
      const text = striptags(decodedText);

      return { start, dur, text };
    });

  return lines;
}
```

=> 在视频的 HTML 中，有一个指向其 timedtext API 的 YouTube 链接，可以找到包含自动字幕的 XML 文件（示例在此处提供 [here](https://www.youtube.com/api/timedtext?v=ULDCt13hetw&ei=c5GOZcmyFea26dsP0-ydqAQ&caps=asr&opi=112496729&xoaf=5&hl=it&ip=0.0.0.0&ipbits=0&expire=1703867363&sparams=ip,ipbits,expire,v,ei,caps,opi,xoaf&signature=75063412BCC19F57C0FACAD8F931B04B33607678.3E55E91F52C40D16D66F8420C614830C702CAAB2&key=yt8&kind=asr&lang=en)）。我们提取这些 XML 数据以生成摘要。

**getSubtitleFromVideo.js** 文件将导出 **getSubtitleFromVideo** 函数。此函数可以作为外部调用，以视频链接作为输入，并返回相应的字幕。

这是 **getSubtitleFromVideo.js** 文件的内容：

```
import *axios* from 'axios';
import *he* from "he";
import striptags from "striptags";

export const getSubtitleFromVideo = async (video) => {
  const video_id = await getVideoId(video)
  const html = await getHTML(video_id)
  const subtitle = await getSubtitle(html)
  return subtitle;
}

const getVideoId = async (video) => {
  // video can be an ID or a link like https://www.youtube.com/watch?v=fOBN8OR8YZA&t=10s

  let video_id;
  if (video.startsWith('http')) {
    const url = new *URL*(video);
    // https://www.youtube.com/watch?v=0chZFIZLR_0
    // https://youtu.be/0chZFIZLR_0?si=-Gp9e_RKG3g1SdVG
    video_id = url.searchParams.get('v') || url.pathname.slice(1);
  } else {
    video_id = resource;
  }

  return video_id
}

const getHTML = async (video_id) => {
  const {data: html} = await axios.get(`https://youtube.com/watch?v=${video_id}`);

  return html
}

const getSubtitle = async (html) => {
  if (!html.includes('captionTracks')) {
    throw new Error(`Could not find captions for video`);
  }

  const regex = /https:\/\/www\.youtube\.com\/api\/timedtext[^"]+/;

  const [url] = html.match(regex);
  if (!url) throw new Error(`Could not find captions`);

  const obj = *JSON*.parse(`{"url": "${url}"}`)

  const subtitle_url = obj.url

  const transcriptResponse = await axios.get(subtitle_url);
  const transcript = transcriptResponse.data;

  const lines = transcript
    .replace('<?xml version="1.0" encoding="utf-8" ?><transcript>', '')
    .replace('</transcript>', '')
    .split('</text>')
    .filter(line => line && line.trim())
    .map(line => {
      const startRegex = /start="([\d.]+)"/;
      const durRegex = /dur="([\d.]+)"/;

      const startMatch = startRegex.exec(line);
      const durMatch = durRegex.exec(line);

      const start = startMatch[1];
      const dur = durMatch[1];

      const htmlText = line.replace(/<text.+>/, '').replace(/&amp;/gi, '&').replace(/<\/?[^>]+(>|$)/g, '');
      const decodedText = *he*.decode(htmlText);
      const text = striptags(decodedText);

      return { start, dur, text };
    });

  return lines;
}
```

[订阅 Implementing ✨](https://implementing.substack.com/subscribe)

在获取字幕之后，目标是将它们排列成文本块，然后进行摘要。如前所述，不同的模型具有不同的最大上下文限制；对于 3.5 turbo 模型，它是 4000 个标记，我假设 1 个标记等于 1 个字符。

所以让我们继续创建**splitInChunks.js**文件：

```
touch splitInChunks.js
```

在这个文件中，我们将导出一个同名函数 splitInChunks，负责将视频帧的各个字幕分组成最多 4000 个字的块，这里是实现方法：

```
const CHUNK_SIZE = 4000

export const splitInChunks = (subtitles) => {
  const chunks = []

  let chunk = ''
  for (const subtitle of subtitles) {
    if (chunk.length + subtitle.text.length + 1 <= CHUNK_SIZE) { // +1 for the space
      chunk += subtitle.text + ' '
    } else {
      chunks.push(chunk)
      chunk = ''
    }
  }
  if (chunk) chunks.push(chunk)

  return chunks
}
```

现在我们已经有了这些块，可以继续进行摘要步骤。

创建一个名为 **summarizeChunks.js** 的新文件：

```
`touch summarizeChunks.js`
```

首先，我们定义调用 OpenAI API 的方法：

```
const OPEN_AI_API_KEY = ''
assert(OPEN_AI_API_KEY, 'Please define OPEN_AI_API_KEY, you can create it from https://openai.com/blog/openai-api');

const _computeSummaryByAI = async ({text, language}) => {
    const body = {
      model: "gpt-3.5-turbo",
      messages: [
        {
          role: "system",
          content: `You are a brilliant assistant, and your task is to summarize the provided text in less than 200 words
            in the language ${*LANGUAGE_CODE_TO_LANGUAGE*[language]}.
            Ensure that the sentences are connected to form a continuous discourse.`
        },
        {
          role: "user",
          content: text
        }
      ],
      temperature: 1,
      top_p: 1,
      frequency_penalty: 0,
      presence_penalty: 0
    };

    try {
      const {data} = await axios.post('https://api.openai.com/v1/chat/completions', body, {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${OPEN_AI_API_KEY}`
        },
      });

      const summary = data.choices[0].message.content
      return summary
    } catch (error) { *console*.error('_computeSummaryByAI', error)
      throw error
    }
  }
```

我们向完成 API 提交一个提示，包括一个 "system" 消息，指示它要做什么，以及一个包含要摘要的文本的 "user" 消息。在 OpenAI 开发者门户上创建 API 密钥非常重要，只需访问此 [link](https://openai.com/blog/openai-api)。创建后，只需将密钥插入常量 `OPEN_AI_API_KEY` 中。

我用下划线（_）前缀命名了这个函数，因为我们将定义一个名为 computeSummaryByAI 的新函数，它是调用 OpenAI 的入口点。这个函数利用 p-queue 库来管理同时运行的最大 Promise 数量，防止速率限制问题和响应时间变慢。

```
import PQueue from 'p-queue';
const pq = new PQueue({concurrency: 5});

const computeSummaryByAI = async ({text, language}) => {
  const summary = await pq.add(() => _computeSummaryByAI({text, language}));

  return summary;
}
```

使用 `concurrency: 5` 意味着最多可以同时运行5次对OpenAI的调用，因为我们将使用 **Promise.all** 结构并行启动调用。

现在我们来到总结机制的核心部分，导出文件的函数将是：

```
export const summarizeChunks = async ({chunks, language}) => {
  // summarizes the subtitles by making sense of them
  chunks = await *Promise*.all(chunks.map((chunk) => computeSummaryByAI({text: chunk, language})))

  let summary

  if (chunks.length > 1) {
    summary = await recursiveSummaryByChunks({chunks, language})
  } else {
    summary = chunks[0]
  }

  return summary
}
```

此功能负责生成每个块的初始摘要，以理解内容的含义，考虑到块包含由YouTube生成的自动字幕。计算完摘要后，如果只有一个块，则过程完成，结果准备就绪。否则，我们需要继续使用递归算法。

现在我们定义递归函数：

```
const recursiveSummaryByChunks = async ({chunks, language}) => {
  if (chunks.length <= 5) {
    return computeSummaryByAI({text: chunks.join(' '), language});
  }

  const groups_chunks = chunkArray(chunks, 5)
  const groups_chunks_summary = await *Promise*.all(groups_chunks.map((group_chunk) => computeSummaryByAI({
    text: group_chunk.join(' '),
    language
  })))

  const result = await recursiveSummaryByChunks({chunks: groups_chunks_summary, language});

  return result;
}

const chunkArray = (array, size) => {
  const chunks = []
  for (let i = 0; i < array.length; i += size) {
    const chunk = array.slice(i, i + size);
    chunks.push(chunk)
  }

  return chunks
}
```

行为如下：

1.  如果少于5个块，请继续通过将块合并为单个文本进行摘要。值得注意的是，我们在提示中指定了最多200个字。有5个块时，我们应该最多有1000个字。然而，在实验中，我注意到OpenAI提示中指定的这些限制通常不会达到。因此，我选择了更低的阈值，以确保正常运行而不会因为上下文过大而出现错误。

1.  如果我们有超过5个块，我们将它们分组成每组5个块，对每组进行合并并进行摘要。摘要调用并行执行。

1.  最后，函数通过传递递归步骤的结果来重新启动自身。

> 所有显示的代码都可以在这里找到：[https://github.com/marcomoauro/youtube-summarizer](https://github.com/marcomoauro/youtube-summarizer)

* * *

通过创建一个CLI，您可以通过传递YouTube视频链接和您想要总结的语言代码来总结您想要的视频，以下是如何启动它的示例：

```
yarn cli --video="https://www.youtube.com/watch?v=3l2wh5K_WLI" --language en
```

*要求您位于项目的根目录，并且已正确安装了依赖项：

```
yarn install
```

* * *

我已将此功能集成到我的 [quickview.email](https://quickview.email/try-summary) 网站中。只需提供您希望接收摘要的电子邮件，YouTube视频的链接以及您想阅读的语言。

[订阅 Implementing ✨](https://implementing.substack.com/subscribe)

* * *

这个机制也是我的加密货币简报的推动力量，您对加密货币世界感兴趣但没有时间保持最新吗？立即订阅以接收每日视频摘要！

* * *

就是这样！如果您觉得这份简报有价值，请考虑做以下任何一件事：

1.  🍻 **与朋友一起阅读** — Implementing 的存在得益于口口相传。与喜欢的人分享这篇文章。

    [分享](https://implementing.substack.com/p/how-to-summarize-youtube-video-using?utm_source=substack&utm_medium=email&utm_content=share&action=share)

1.  📣 **提供您的反馈** - 我们欢迎您的想法！请分享您的意见或改进简报的建议，您的意见有助于我们调整内容以迎合您的口味。

[留言评论](https://implementing.substack.com/p/why-i-moved-from-rails-to-nodejs/comments)

我祝你有个美好的一天！☀️

Marco
