<!--yml

category: 未分类

date: 2024-05-29 12:44:41

-->

# 队列并不是正确的抽象化方式 - Inngest 博客

> 来源：[https://www.inngest.com/blog/queues-are-no-longer-the-right-abstraction](https://www.inngest.com/blog/queues-are-no-longer-the-right-abstraction)

亚马逊 SQS 将在今年晚些时候迎来[20周年](https://aws.amazon.com/about-aws/whats-new/2004/11/03/introducing-the-amazon-simple-queue-service/)。它仍然提供它所宣传的——[简单队列服务](https://aws.amazon.com/sqs/)。并且依旧表现出色。在 2022 年 Prime Day，SQS 在高峰时期每秒处理[超过7000万条消息](https://aws.amazon.com/blogs/aws/amazon-prime-day-2022-aws-for-the-win/)。如果你需要队列，像 RabbitMQ 或 ActiveMQ 这样的经过时间验证的解决方案也很有效。

但你不只是需要一个队列。你需要更好的解决方案。

## 消息队列的目标

过去几十年来，为什么消息队列在开发人员中被广泛使用于网络系统和环境中？

应用程序中许多工作负载是长时间运行的：通知、数据处理、计费等。这些操作可能耗时或消耗资源，并且你不希望它们阻塞应用程序的其他部分。消息队列将这些功能从关键路径（*或线程*）中卸载，并允许你异步执行它们。队列将使应用程序保持响应性，同时在后台处理任务。

因此，队列运行关键工作负载，只是不在关键路径上。你需要队列有三个核心原因：

1.  通过保证传递、持久性和死信队列，它们提供可靠性，因此开发人员知道它们并非将工作负载发送到黑洞中。

1.  它们允许开发人员运行关键的长时间运行过程。这些过程通常涉及复杂的业务逻辑、数据转换或与外部系统的集成。通过将这些任务卸载到队列中，开发人员可以确保它们可靠地执行，而不会阻塞主应用程序流程。

1.  队列还通过允许多个消费者并行处理消息来提供水平扩展能力。这意味着应用程序可以处理高吞吐量，并在工作负载增加时进行扩展。队列还可以很好地处理工作负载的突发情况，平衡负载并使应用程序保持稳定。

从更高层面来看，队列是构建更高效、功能性和可靠的应用程序的一种方式。当你将队列添加到应用程序中时，你从必须将每个操作适应请求-响应框架的情况转变为能够解耦和优先处理工作负载的能力。这使你能够设计围绕最佳用户体验的应用程序，而不受同步处理限制的约束。通过利用队列，你可以创建更响应、可扩展和对故障更具韧性的应用程序。

今天为什么开发人员需要队列？

恰好是同样的原因。这就是为什么开发人员仍然选择它们的原因。上述所有情况仍然适用。你想要解耦吗？队列适用。你想要可扩展性吗？队列适用。你想要灵活性吗？队列适用。

但从实现的角度来看，队列有些不尽如人意。底层思想很好，但任何使用消息队列构建基础设施的人都知道，队列只是一个通道。而你需要大量的架构来支持这个通道。

## 队列不是正确的抽象

消息队列是一个抽象概念。它们抽象出了需要入队和传递消息所需的通信协议和数据存储。

但在2024年，队列已不再是构建现代系统的正确抽象。队列的基本概念仍然适用，但构建具有队列的系统需要的不仅仅是队列本身。一旦开发人员实现了主要的队列，他们会发现他们需要：

+   **链式处理**，按顺序运行任务。

+   **并发控制**，控制同时执行的作业数量。

+   **退避策略**，优雅地处理速率限制的API调用。

+   **去抖动**，防止函数多次执行。

+   **状态持久性和管理**，因为你需要在不同的工作者和队列之间共享状态。

+   **优先级排序**，以确定哪些消息最为关键。

+   **错误处理**，包括失败和超时的重试。

+   **幂等性**，以确保不重复执行并防止意外副作用。

+   **取消操作**，在必要时中止消息和正在进行的作业。

+   **可观察性**，监视和优化系统。

+   **恢复工具**，帮助理解错误并重新处理失败事件。

假设你需要并发处理。现在你要考虑消费消息的工作者。你需要增加更多工作者和/或调整工作者代码中的消息轮询逻辑。现在，你需要考虑每条消息的可见超时时间等因素，并考虑消息被处理多次的下游影响。

对基础设施的每个组件都要重复这个决策树。这不仅仅是获取一个SQS URL并让SQS处理所有事务的问题。所有这些都需要围绕消息队列进行构建。即使你现在不需要这个基础设施，随着时间的推移，你也必须完全构建它来构建一个强大的消息系统。

这是具有挑战性和乏味的工作。有减轻负担的方法 - [Celery](https://docs.celeryq.dev/en/stable/getting-started/introduction.html)，[Resque](https://github.com/resque/resque)，[BullMQ](https://bullmq.io/) - 但使用这些半抽象存在问题：缺乏多语言选项，略有不同的实现，以及对这些工具不熟悉的开发人员的学习曲线，增加了复杂性和维护开销。

那么，为什么开发人员仍然使用消息队列，即使它们不是正确的抽象？有两个原因：

1.  他们了解消息队列。即使他们知道他们将不得不实现队列周围的基础设施，开发人员也已经使用SQS、RabbitMQ等工具有20年了。这是一件麻烦事，但却是*众所周知的*麻烦。

1.  他们不知道有更好的抽象化。

随着时间的推移，抽象化已经发展并向上移动到堆栈。

## 通过耐久性实现可靠性

耐久执行是正确的抽象化，而不是直接处理队列。耐久执行结合两个元素，为应用工作流程提供可靠性和保证。

首先是**耐久性**。耐久执行是容错执行，保证代码即使在失败或重新启动时也能完成执行。这解决了上述问题的一部分。有了耐久执行，开发人员不必构建确保执行的流程，比如实现重试逻辑、处理失败或持久化状态，因为这些能力由耐久执行运行时提供。

但单单耐久性是不够的。通过耐久执行实现可靠性需要**流控制**。流控制涵盖了您必须为可靠的队列工作流程构建的所有内容。虽然耐久执行侧重于保证任务的执行和处理失败，但流控制则处理如何或何时执行代码。

流控制涵盖了各种方面，如并发控制、速率限制、资源共享和优先级设置。假设您有一个用于转录视频的处理工作流程。这意味着什么？

+   您需要为转录、总结和上传设置多个队列。由于这些必须按顺序运行，您必须在它们之间管理状态。

+   您必须通过一个事件触发初始转录。然后，接下来的两个步骤由前一个步骤触发。

+   您必须控制并发性，以管理单个用户同时处理的视频数量。

+   您希望有一定程度的优先级设置，以便控制哪些用户的工作流程先运行。

+   您必须实现重试逻辑，以防任何步骤失败，后退逻辑如果任何API受到速率限制，去抖动以阻止多次调用，和可观察性以确保您可以理解操作中发生了什么。

在队列级别实现这一点是一项严肃的工作。您会将所有这些封装在一个单一函数内部，还是将其拆分为针对特定队列的不同任务？您将为每个步骤在数据库中保存状态吗？还是放任不管，只是创建一个调用链？

通过耐久函数，这些问题被抽象化处理，为您提供了以下功能：

```
typescript

export  const  processVideo  =  inngest.createFunction(

 {

 name:  "Process video upload", id:  "process-video",

 concurrency: {

 limit:  1,

 key:  `event.data.userId`,  // You can use any piece of data from the event payload

 },

 priority: {

 run:  "event.data.billingPlan != 'free' ? 120 : 0",

 },

 },

 { event:  "video.uploaded" },

  async ({ event, step }) => {

  const  transcript  =  await  step.run('transcribe-video',  async () => {

  return  deepgram.transcribe(event.data.videoUrl);

 });

  const  summary  =  await  step.run('summarize-transcript',  async () => {

  return  llm.createCompletion({

 model:  "gpt-3.5-turbo",

 prompt:  createSummaryPrompt(transcript),

 });

 });

  await  step.run('write-to-db',  async () => {

  await  db.videoSummaries.upsert({

 videoId:  event.data.videoId,

 transcript,

 summary,

 });

 });

 }

)

```

你看不到所有的基础设施，因为它被抽象化处理，但这处理并发性、重试、状态和底层队列。它由`video.uploaded`事件触发，然后将继续执行直到完成或显示错误（并允许您在必要时重新播放）。

如果使用像 Celery 和 Python 中的部分抽象，它可能会是什么样子呢？首先，我们需要一个消息代理服务，因为 Celery 并没有抽象这部分。然后，我们将其添加到我们的 Celery 工作进程中作为代理 URL。接下来，我们需要考虑流程控制。我们如何管理优先级和并发性，以及基于每个用户的情况？在 Celery 中，可以通过动态更新配置来实现这一点：

```
python

# celery_app.py

def  user_task_router(name,  args,  kwargs,  options,  task=None,  **kw):

 user_id = kwargs.get('user_id')

  if user_id:

  return  {

  'queue':  f'user_{user_id}',

  'priority': kwargs.get('priority', 10),  # Get the priority from kwargs, default to 10

  }

  return  None

app =  Celery('tasks', broker='amqp://guest:guest@localhost:5672//')

app.conf.update(

 worker_concurrency=4,

 worker_prefetch_multiplier=1,

 task_routes = (user_task_router,),

 task_default_queue =  'default',

 task_default_priority =  0,

 task_queue_max_priority =  10,

 task_queues = [Queue('default', routing_key='default')],

)

```

然后我们可以在我们的 `tasks.py` 文件中使用它：

```
python

# tasks.py

from celery import shared_task

from utils import transcribe_video, summarize_transcript, write_to_db

@shared_task(name='process_video')

def  process_video(event,  user_id,  priority=10):

 video_path = event['data']['videoPath']

  # Transcribe the video

 transcript =  transcribe_video(video_path, user_id, priority)

  # Summarize the transcript

 summary =  summarize_transcript(transcript, user_id, priority)

  # Write to the database

  write_to_db(video_path, transcript, summary, user_id, priority)

@shared_task

def  process_video_event(event,  priority=10):

 user_id = event['data']['userId']

 process_video.apply_async(args=[event], kwargs={'user_id': user_id, 'priority': priority})

```

每个 `transcribe_video`、`summarize_transcript` 和 `write_to_db` 将在 `utils.py` 中，并且我们可以从实际应用程序中调用 `process_video_event`。

这有什么问题？嗯，它只是更复杂了。关于 Celery 是部分抽象的观点，我们仍在直接处理低级消息队列，然后不得不设置大量的配置。我们想要的流程控制（优先级、并发性）不容易嵌入到我们的代码中。相反，我们必须使用一种解决方案为每个用户更新 Celery 配置。状态和逻辑也被分离开来，使得工作流更难理解。

将此与执行更复杂任务并暂停队列一段时间的示例进行比较。

```
typescript

export  const  handlePayments  =  inngest.createFunction(

 {

 name:  "Handle payments", id:  "handle-payments"

 },

 { event:  "api/invoice.created" },

  async ({ event, step }) => {

  // Wait until the next billing date

  await  step.sleepUntil("wait-for-billing-date",  event.data.invoiceDate);

  // Steps automatically retry on error, and only run

  // once on success - automatically, with no work.

  const  charge  =  await  step.run("charge",  async () => {

  return  await  stripe.charges.create({

 amount:  event.data.amount,

 });

 });

  await  step.run("update-db",  async () => {

  return  await  db.payments.upsert(charge);

 });

  await  step.run("send-receipt",  async () => {

  return  await  resend.emails.send({

 to:  event.user.email,

 subject:  "Your receipt for Inngest",

 });

 });

 }

);

```

即使是一个长时间运行的工作流，也更容易跟踪。比如说，你正在向客户进行结算，如果结算失败需要重试。通常情况下，追索过程会持续几天。你如何使用队列处理这种情况？

需要大量的布洛芬。有些队列允许你添加延迟，但通常是在秒和分钟的时间范围内，而不是你在这里所需的几天。因此，你必须保存状态，并在一天后启动另一个队列。为此，你可能最终会使用一堆 cron 作业和队列，管理它们之间的状态和函数。

使用持久化执行，所有这些都可以封装在一个单独的工作流函数中：

整个流程在暂停一天后继续。开发人员在此期间无需管理任何状态——一切都自动完成。跨语言支持是一流的，因为管理队列的复杂部分已完全抽象化。

持久化执行实现了队列的原始目标：可靠地执行在异步 API 请求之外运行的代码。

## 不要使用原始队列构建

你能自己构建这个系统吗？当然可以。如果你从一开始就在使用队列，你可能已经理解了将它们置于实际功能状态所需的各种变通方法。

你应该这样做吗？不。实施所有这些只是为了拥有一个现成可用的持久化执行引擎，会消耗大量时间和金钱，还需要大量的基础设施、逻辑和状态管理——这实在太烦琐了。如果你对消息队列的具体细节感兴趣，可以深入研究。对于其他人来说，使用现有的高级抽象层即可——持久化执行。

要了解团队如何在生产中使用持久执行，请查看[Soundcloud简化动态视频生成](/customers/soundcloud?ref=blog)，[Resend使用无服务器工作流发送电子邮件](/customers/resend?ref=blog)，以及[Aomni推广AI驱动的销售流程](/customers/aomni?ref=blog)。当您想要自己开始持久执行时，您可以今天就[注册Inngest](https://app.inngest.com/sign-up)，或者[联系销售工程部门](/contact?ref=blog)。
