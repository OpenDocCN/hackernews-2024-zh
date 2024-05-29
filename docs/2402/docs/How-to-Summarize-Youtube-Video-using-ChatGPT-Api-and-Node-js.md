<!--yml
category: 未分类
date: 2024-05-27 14:51:44
-->

# How to Summarize Youtube Video using ChatGPT Api and Node.js

> 来源：[https://implementing.substack.com/p/how-to-summarize-youtube-video-using](https://implementing.substack.com/p/how-to-summarize-youtube-video-using)

Hey, I'm [Marco](https://www.linkedin.com/in/marco-moauro-b49489162/) and welcome to my newsletter!

As a software engineer, I created this newsletter to share my first-hand knowledge of the development world. Each topic we will explore will provide valuable insights, with the goal of inspiring and helping all of you on your journey.

In this episode I want to bring you the first tutorial, on how to make a system in Node.js that starting from a youtube video link generates a summary using OpenAI's [completions api](https://platform.openai.com/docs/guides/text-generation/chat-completions-api), the same api on which the ChatGPT system is based.

> You can download all the code shown directly from my Github repository: [https://github.com/marcomoauro/youtube-summarizer](https://github.com/marcomoauro/youtube-summarizer)

[Subscribe to Implementing ✨](https://implementing.substack.com/subscribe)

* * *

The system architecture primarily comprises:

1.  Extracting text from YouTube videos

2.  Generating text summaries

This process involves extracting text from the video and utilizing it for summary generation. Various options were considered, including:

The chosen solution involves scraping, which is the most challenging among the options. This decision is motivated by the fact that implementing everything independently incurs no costs associated with third-party APIs for text extraction. Additionally, as a genuine enthusiast, I prefer this approach, given that most of my personal projects are founded on this technique.

> If you are interested in finding out what are the best practices for web scraping sign up for the newsletter, I will be publishing a post about it soon.

After getting the captions, we put them into OpenAI. The first challenge I faced was the limit on the maximum size of the text that the completions API can handle. This limit depends on the model used; for the 3.5 turbo model, it's specifically set at 4.000 tokens.

To overcome this limitation, I adopted a recursive approach. The text is divided into smaller parts, which are merged into groups and summarized independently; this process is repeated until a single output text is generated, corresponding to the final summary generated from the intermediate summaries.

[Subscribe to Implementing ✨](https://implementing.substack.com/subscribe)

* * *

For this tutorial you need to have Yarn and Node.js installed, specifically I used the LTS version 20.9.0\. If you don't have Node.js on your machine you can install it from the [official website](https://nodejs.org/en).

Starting from my workspaces folder I have created the project folder and the npm package:

```
mkdir youtube-summarizer
cd youtube-summarizer
npm init -y 
```

add the clause “type: module” to use ES6 syntax:

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

The libraries we need are **Axios**, a library for making HTTP calls, we will need to implement the call to the completion, **he** and **striptags** to manipulate the HTML, **p-queue** for handle a queue for the promises in order to call OpenAI with max X calls in the same moment and **yargs** to build interactive command line tools.

You can install it typing:

```
yarn add axios
yarn add he
yarn add striptags
yarn add p-queue
yarn add yargs
```

Create the file **getSubtitleFromVideo.js**:

```
touch getSubtitleFromVideo.js
```

The initial function we'll create is designed to use web scraping to fetch the HTML content of a YouTube video page. Let's name this function **getHTML**.

```
const getHTML = async (video_id) => {
  const {data: html} = await axios.get(`https://youtube.com/watch?v=${video_id}`);

  return html
}
```

Once we get the html we can retrieve the subtitles via this **getSubtitle** function:

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

=> Inside the video's HTML, there's a YouTube link directing to their timedtext API, where an XML file containing automatic captioning can be found (an example is provided [here](https://www.youtube.com/api/timedtext?v=ULDCt13hetw&ei=c5GOZcmyFea26dsP0-ydqAQ&caps=asr&opi=112496729&xoaf=5&hl=it&ip=0.0.0.0&ipbits=0&expire=1703867363&sparams=ip,ipbits,expire,v,ei,caps,opi,xoaf&signature=75063412BCC19F57C0FACAD8F931B04B33607678.3E55E91F52C40D16D66F8420C614830C702CAAB2&key=yt8&kind=asr&lang=en)). This XML data is what we extract to generate the summary.

**getSubtitleFromVideo.js** file will export the **getSubtitleFromVideo** function. This function can be called externally, taking the video link as input and returning the corresponding subtitles.

This is the contents of the file **getSubtitleFromVideo.js:**

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

[Subscribe to Implementing ✨](https://implementing.substack.com/subscribe)

After getting the subtitles, the goal is to arrange them in chunks of text and then summarize. As mentioned earlier, different models have various maximum context limits; for the 3.5 turbo model, it's 4,000 tokens, I made the assumption that 1 token equals 1 character.

So let's go ahead and create the **splitInChunks.js** file:

```
touch splitInChunks.js
```

In this file we are going to export a function with the same name splitInChunks that takes care of grouping the individual subtitles of the video frames into chunks of up to 4000 words, here is the implementation:

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

At this point we have the chunks and we can proceed with the summary step.

Create a new file **summarizeChunks.js:**

```
`touch summarizeChunks.js`
```

we first define the method for calling the OpenAI api:

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

We submit a prompt to the completions API, comprising a "system" message instructing it on what to do, and a "user" message containing the text to be summarized. It's important to create an API key on the OpenAI developer portal, just follow this [link](https://openai.com/blog/openai-api). Once created, simply insert the key into the constant `OPEN_AI_API_KEY`.

I've named this function with the underscore (_) prefix because we'll define a new function called computeSummaryByAI, which serves as the entry point to OpenAI. This function utilizes the p-queue library to manage a maximum number of Promises running concurrently, preventing rate-limiting issues and slower response times.

```
import PQueue from 'p-queue';
const pq = new PQueue({concurrency: 5});

const computeSummaryByAI = async ({text, language}) => {
  const summary = await pq.add(() => _computeSummaryByAI({text, language}));

  return summary;
}
```

With `concurrency: 5` it means that there can be at most 5 calls running to OpenAI, all because we will launch calls in "parallel" via the **Promise.all** construct.

We now come to the core part of the summary mechanism, the function that will export the file will be:

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

This function is responsible for generating an initial summary for each chunk to make sense of the content, considering that the chunks contain automatic subtitles generated by YouTube. After calculating the summary, if we have only one chunk, the process is complete, and the result is ready. Otherwise, we need to proceed with the recursive algorithm.

We now define the recursive function:

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

The behavior is as follows:

1.  If there are fewer than 5 pieces, proceed to the summary by combining the pieces into a single text. It is worth noting that we specified a maximum of 200 words in the prompt. With 5 pieces, we should have a maximum of 1000 words. However, in experimenting, I have noticed that these limits specified in the OpenAI prompt are often not met. As a result, I opted for lower thresholds to ensure proper operation without running into errors due to too large context.

2.  If we have more than 5 chunks, we group them into groups of 5, for each we merge them into a single text and summarize them. Calls for summaries are executed in parallel.

3.  Finally, the function relaunches itself by passing the result of the recursive step.

> All the code shown can be found here: [https://github.com/marcomoauro/youtube-summarizer](https://github.com/marcomoauro/youtube-summarizer)

* * *

I created a CLI through which you can summarize the videos you want by passing the YouTube video link and the language code in which you want the summary, here is an example of how to launch it:

```
yarn cli --video="https://www.youtube.com/watch?v=3l2wh5K_WLI" --language en
```

*requires that you are in the root of the project and have properly installed the dependencies with:

```
yarn install
```

* * *

I have integrated this functionality into my [quickview.email](https://quickview.email/try-summary) website. Simply provide the email with which you want to receive the summary, the link to the youtube video and the language in which you want to read it.

[Subscribe to Implementing ✨](https://implementing.substack.com/subscribe)

* * *

This same mechanism is the driving force behind my crypto newsletter, are you interested in the world of cryptocurrencies but don't have time to stay up-to-date? Sign up now to receive daily video summaries!

* * *

And that’s it for today! If you are finding this newsletter valuable, consider doing any of these:

1.  🍻 **Read with your friends** — Implementing lives thanks to word of mouth. Share the article with someone who would like it.

    [Share](https://implementing.substack.com/p/how-to-summarize-youtube-video-using?utm_source=substack&utm_medium=email&utm_content=share&action=share)

2.  📣 **Provide your feedback** - We welcome your thoughts! Please share your opinions or suggestions for improving the newsletter, your input helps us adapt the content to your tastes.

[Leave a comment](https://implementing.substack.com/p/why-i-moved-from-rails-to-nodejs/comments)

I wish you a great day! ☀️

Marco