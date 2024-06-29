<!--yml

category: 未分类

date: 2024-05-27 13:19:22

-->

# 用Elixir实现自然对话代理 - Sean Moriarity

> 来源：[https://seanmoriarity.com/2024/02/25/implementing-natural-conversational-agents-with-elixir/](https://seanmoriarity.com/2024/02/25/implementing-natural-conversational-agents-with-elixir/)

在[我上一篇文章](https://seanmoriarity.com/2024/02/25/nero-part-1-home-automations/)中，我讨论了我构建的一些工作，构建了我一直想要的未来助手Nero。最终，我创建了一个端到端示例，使用了Nx、OpenAI APIs和ElevenLabs来创建一个浏览器内的家庭自动化助手。作为第一个产品，它还不错。Nero是一个小巧的聚会把戏，我可以用来给非技术朋友留下深刻印象。然而，我做这个并不是为了让朋友们印象深刻。我希望Nero能够*真正帮助我*，*真正感觉像一个助手*。我的上一个版本并不是那样。

缺少的一环是在没有浏览器交互的情况下自然对话的能力。Nero的第一个“对话”能力实现依赖于用户每次想要启动响应或动作时与屏幕的交互。Nero也没有保留任何对话历史记录。简而言之，Nero不是一个很好的对话助手。看完[Retell的一个令人印象深刻的演示之后](https://beta.retellai.com/home-agent)，我立即想要尽快修复这些问题。

Retell演示在浏览器中通过他们的WebSocket API实现了一个支持对话代理。演示包括：

+   “始终开启”录音

+   低延迟

+   支持中断

+   令人印象深刻的过滤（例如，捕捉和其他非语音活动似乎不会干扰代理）

他们的文档表明他们还支持[后援反馈](https://www.cs.utep.edu/nigel/bc/#:~:text=What%20is%20a%20Backchannel%3F,utterances%20such%20as%20uh%2Dhuh.)和智能的对话结束检测，这两点对于实现自然对话感觉非常重要，但在程序上非常难以表达。

我之前确信自己可以在很短的时间内实现一个过得去的对话体验助手。所以我就着手去做了。

## 始终开启录音

关于Nero的设计需要改变的第一件事是语音到文本流水线。我的初始演示依赖于[Bumblebee的一个示例](https://github.com/elixir-nx/bumblebee/blob/main/examples/phoenix/speech_to_text.exs)，该示例使用Whisper实现了一个使用Phoenix LiveView Hook在发送到服务器进行转录之前开始和停止录音的语音到文本流水线。如果您不熟悉，[Phoenix LiveView](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html)是建立在Elixir之上的服务器端渲染框架。LiveView支持客户端JavaScript钩子，支持客户端和服务器之间的双向通信。

初始的语音转文字实现使用了一个钩子，附加到按钮的`mousedown`和`mouseup`事件监听器上，用于开始和停止录音。在录音停止后，钩子将记录的缓冲区解码为PCM缓冲区，转换字节顺序，然后将缓冲区推送到服务器进行上传。原始钩子实现了大部分我们想要的功能；然而，我们需要进行一些小的调整。与其在鼠标事件上触发录音开始和停止，我们希望在人开始说话和停止说话时准确地触发录音的开始和停止。简单吧？

我在实现我称之为“始终录音”的想法时，第一个想到的是监视麦克风的音量，并且当音量达到一定的阈值时触发录音。当音量低于该阈值时停止录音。在这一点上，我了解了[getUserMedia](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia)。`getUserMedia`提示用户允许访问媒体设备，比如麦克风和/或摄像头，然后生成一个`MediaStream`。`MediaStream`是一个包含有关流中音频和视频轨道信息的媒体内容流。我们可以使用`MediaStream`的数据来确定说话者的活动，从而触发录音。

要确定特定样本的音量，我们可以使用[AnalyserNode](https://developer.mozilla.org/en-US/docs/Web/API/AnalyserNode)。根据文档，`AnyalyserNode`被设计用于处理生成的音频数据以进行可视化，但我们可以使用它来确定音频中的峰值：

```
navigator.mediaDevices.getUserMedia({ audio: true }).then((stream) => {
  this.stream = stream;
  const analyser = this.audioContext.createAnalyser();
  const microphone = this.audioContext.createMediaStreamSource(this.stream);

  microphone.connect(analyser);

  analyser.fftSize = 512;
  const bufferLength = analyser.frequencyBinCount;
  const dataArray = new Uint8Array(bufferLength);

  const checkAudio = () => {
    analyser.getByteTimeDomainData(dataArray);

    let sum = 0;
    for (let i = 0; i < bufferLength; i++) {
      sum += (dataArray[i] - 128) * (dataArray[i] - 128);
    }
    const volume = Math.sqrt(sum / bufferLength);

    if (volume > VOLUME_THRESHOLD && !this.isRecording) {
      this.startRecording();
    } else if (this.isRecording()) {
      this.stopRecording();
    }

    if (this.isMonitoring) {
      requestAnimationFrame(checkAudio);
    }
  }

  checkAudio();
})

```

这使用一个分析器并重复检查给定帧的麦克风音量是否超过给定的`VOLUME_THRESHOLD`。如果是，则检查我们是否正在录制，如果不是，则开始录制。

测试了一下之后，我意识到这个实现并不好用。这种方法有很多问题，其中最大的问题是人的音量会自然地有很多波动。仅检查单个帧无法解决这些自然波动。为了解决这个问题，我想引入一个超时机制，只有在音量低于某个阈值一段时间后才停止录制：

```
navigator.mediaDevices.getUserMedia({ audio: true }).then((stream) => {
  this.stream = stream;
  const analyser = this.audioContext.createAnalyser();
  const microphone = this.audioContext.createMediaStreamSource(this.stream);

  microphone.connect(analyser);

  analyser.fftSize = 512;
  const bufferLength = analyser.frequencyBinCount;
  const dataArray = new Uint8Array(bufferLength);

  this.silenceTimeout = null;

  const checkAudio = () => {
    analyser.getByteTimeDomainData(dataArray);

    let sum = 0;
    for (let i = 0; i < bufferLength; i++) {
      sum += (dataArray[i] - 128) * (dataArray[i] - 128);
    }
    const volume = Math.sqrt(sum / bufferLength);

    if (volume > VOLUME_THRESHOLD) {
      if (!this.isRecording()) {
        this.startRecording();
      }

      clearTimeout(this.silenceTimeout);

      this.silenceTimeout = setTimeout(() => {
        if (this.isRecording()) {
          this.stopRecording();
        }
      }, SILENCE_TIMEOUT);
    }

    if (this.isMonitoring) {
      requestAnimationFrame(checkAudio);
    }
  }

  checkAudio();
})

```

这实际上效果还不错，但需要调整 `VOLUME_THRESHOLD` 和 `SILENCE_TIMEOUT` 的超参数。挑战在于，更高的 `SILENCE_TIMEOUT` 会增加说话者与 Nero 之间转换时间的延迟，但较低的超时时间可能对说话节奏较慢和音量较低的说话者过于敏感。此外，静态的 `VOLUME_THRESHOLD` 无法考虑环境噪声。尽管存在这些缺点，我发现我能够基本上在安静的房间里检测出单个说话者。

将这个连接到我的现有 LiveView 并尝试了一些端到端对话后，我意识到有些地方明显不对劲。我得到的转录结果不正确。很快我意识到问题总是出现在转录开始的时候。较短的音频序列尤其受影响。原来检测算法在音频剪辑的开头总是会有一定的截断。当说话者开始讲话时，他们的音量会逐渐增加，而不是瞬间的峰值。为了解决这个问题，我引入了一个预录音缓冲区，始终跟踪前 150 毫秒的音频。录制开始后，我会停止预录音缓冲并开始实际录制，然后最终将这两者拼接在一起发送到服务器进行转录。

总体来说，这 *实际上* 运行得还可以。虽然存在一些明显的失败模式，但足够演示了一下。如果到目前为止你还没发现，我不是一个音频工程师。后来我了解到，这是对 [语音活动检测](https://en.wikipedia.org/wiki/Voice_activity_detection) 的一个非常幼稚的尝试。在本文的后面部分，我将介绍一些基于我对 VAD 领域研究的改进。

## 端到端实现

我在我的 [第一篇文章](https://seanmoriarity.com/2024/02/25/nero-part-1-home-automations/) 中为 Nero 构建的演示已经包含了端到端转录 -> 响应 -> 语音管道的脚手架。我只需要进行一些微小的修改就能让电话演示工作起来。端到端的管道如下所示：

当我们的算法检测到语音已停止时，它调用`stopRecording`方法。`stopRecording`获取录制的音频，执行一些客户端预处理，并将其上传到服务器。服务器作为[LiveView的正常上传生命周期](https://hexdocs.pm/phoenix_live_view/uploads.html)的一部分消耗上传的条目，并调用一个[异步任务](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#module-arbitrary-async-operations)来启动转录：

```
  defp handle_progress(:audio, entry, socket) when entry.done? do
    binary =
      consume_uploaded_entry(socket, entry, fn %{path: path} ->
        {:ok, File.read!(path)}
      end)

    audio = Nx.from_binary(binary, :f32)

    socket =
      start_async(socket, :transcription, fn ->
        Nero.SpeechToText.transcribe(audio)
      end)

    {:noreply, socket}
  end

```

注意，因为我们大部分预处理是在客户端完成的，所以我们可以直接将音频二进制作为`Nx.Tensor`消耗，无需额外工作。`SpeechToText`模块使用[Nx.Serving](https://hexdocs.pm/nx/Nx.Serving.html)实现转录：

```
defmodule Nero.SpeechToText do
  @repo "distil-whisper/distil-medium.en"

  def serving() do
    {:ok, model_info} = Bumblebee.load_model({:hf, @repo})

    {:ok, featurizer} = Bumblebee.load_featurizer({:hf, @repo})
    {:ok, tokenizer} = Bumblebee.load_tokenizer({:hf, @repo})
    {:ok, generation_config} = Bumblebee.load_generation_config({:hf, @repo})

    Bumblebee.Audio.speech_to_text_whisper(model_info, featurizer, tokenizer, generation_config,
      task: nil,
      compile: [batch_size: 1],
      defn_options: [compiler: EXLA]
    )
  end

  def transcribe(audio) do
    output = Nx.Serving.batched_run(__MODULE__, audio)
    output.chunks |> Enum.map_join(& &1.text) |> String.trim()
  end
end

```

`Nx.Serving`是Elixir Nx生态系统中用于直接为Elixir应用程序提供机器学习模型的抽象。它实现动态批处理，封装预处理、推理和后处理，原生支持多个GPU之间的分布和负载平衡，并且通常是一种极其简单的方法来提供机器学习模型。

在转录完成后，我们获得了一个异步结果，可以处理以启动响应：

```
  def handle_async(:transcription, {:ok, transcription}, socket) do
    chat = socket.assigns.chat ++ [%{role: "user", content: transcription}]
    response = Nero.Agent.respond(chat)

    {:noreply,
     socket
     |> assign(chat: chat)
     |> speak(response)}
  end

```

这里，`Nero.Agent.respond/1`返回一个Elixir文本流。对于我的原始演示，我只是使用Elixir OpenAI库从GPT-3.5响应中生成流：

```
  def respond(chat) do
    prompt = Nero.Prompts.response()

    response_stream =
      OpenAI.chat_completion(
        model: "gpt-3.5-turbo",
        messages: [%{role: "system", content: prompt} | chat],
        max_tokens: 400,
        stream: true
      )

    response_stream
    |> Stream.map(&get_in(&1, ["choices", Access.at(0), "delta", "content"]))
    |> Stream.reject(&is_nil/1)
  end

```

响应流由`speak/2`消耗。`speak/2`实现了文本到语音的流水线：

```
  defp speak(socket, text) do
    start_async(socket, :speak, fn ->
      Nero.TextToSpeech.stream(text)
    end)
  end

```

`Nero.TextToSpeech.stream/1`使用[ElevenLabs WebSocket API](https://elevenlabs.io/docs/api-reference/websockets)来流入文本并流出语音。您可以在我的上一篇文章中进一步了解其实现。

`Nero.TextToSpeech.stream/1`将消耗的响应返回为文本，以便我们可以在`:speak`任务完成后将其附加到聊天历史记录中：

```
def handle_async(:speak, {:ok, response}, socket) do
  chat = socket.assigns.chat ++ [%{role: "assistant", content: response}]
  {:noreply, assign(socket, :chat, chat)}
end

```

这基本上是进行端到端演示所需的所有脚手架，但我想添加更多功能。首先，我希望支持“智能”挂断电话。基本上，我希望能够检测到对话何时结束，并停止录音。为此，我使用了[Instructor](https://github.com/thmsmlr/instructor_ex)：

```
  def hang_up?(chat) do
    {:ok, %{hang_up: hang_up}} =
      Instructor.chat_completion(
        model: "gpt-3.5-turbo",
        messages: [
          %{
            role: "system",
            content:
              "Decide whether or not to hang up the phone given this transcript. You should hang up after the user says goodbye or that there's nothing else you can help them with. DO NOT HANG UP ON THE USER UNLESS THEY SAY GOODBYE."
          }
          | chat
        ],
        response_model: %{hang_up: :boolean}
      )

    hang_up
  end

```

请忽略我精心设计的提示。这使用GPT-3.5来确定给定对话是否已经结束。在Nero的每次对话之后，我们检查转录以可能结束通话：

```
  def handle_async(:speak, {:ok, response}, socket) do
    chat = socket.assigns.chat ++ [%{role: "assistant", content: response}]

    socket =
      if Nero.Agent.hang_up?(chat) do
        push_event(socket, "hang_up", %{})
      else
        socket
      end

    {:noreply, assign(socket, :chat, chat)}
  end

```

这将向套接字推送一个`hang_up`事件：

```
    this.handleEvent('hang_up', () => {
      hook.pushEvent("toggle_conversation");

      if (this.audioContext) {
        this.audioContext.close();
        this.audioContext = null;
      }

      if (this.isMonitoring) {
        this.stopMonitoring();
      }
    });

```

这将停止录音，并将一个事件推送到`toggle_conversation`返回到服务器。`toggle_conversation`实现了服务器上的启动/停止逻辑：

```
  def handle_event("toggle_conversation", _params, socket) do
    socket =
      if socket.assigns.conversing do
        stop_conversation(socket)
      else
        start_conversation(socket)
      end

    {:noreply, socket}
  end

```

最后，我想实现从转录中提取信息。同样，我使用了instructor并定义了一个提取模式：

```
  defmodule Appointment do
    use Ecto.Schema

    embedded_schema do
      field :booked, :boolean
      field :date, :string
      field :time, :string
      field :name, :string
      field :phone_number, :string
      field :reason_for_visit, :string
    end
  end

```

并使用GPT-3.5与一个粗略的提示从转录中获取必要的信息：

```
  def extract_appointment(chat) do
    Instructor.chat_completion(
      model: "gpt-3.5-turbo",
      messages: [
        %{
          role: "system",
          content:
            "Extract appointemnt information from the transcript. If info is missing, leave it blank. If it seems like no appointment was booked, mark booked as false and leave everything else blank. An appointment is not booked if there's no established date."
        }
        | chat
      ],
      response_model: Appointment
    )
  end

```

每当对话结束时，我们尝试检索约会信息：

```
defp stop_conversation(socket) do
  case Nero.Agent.extract_appointment(socket.assigns.chat) do
    {:ok, %{booked: true} = appointment} ->
      assign(socket,
        message: "You made an appointment! Details:",
        appointment: appointment,
        conversing: false
      )

    _ ->
      assign(socket,
        message: "Looks like you didn't actually book an appointment. Try again",
        conversing: false
      )
  end
end

```

现在这基本上就是生成了[这个演示](https://twitter.com/sean_moriarity/status/1760435005119934862)的确切实现。从头到尾这大约花了几个小时的工作；然而，我已经在我之前在 Nero 上的工作中实现了大部分基本框架。在我主观的看法中，我认为我的演示相当不错，但正如其他人指出的，Retell 的演示比我的好得多：

+   延迟

+   可靠性

+   自然的声音

因此，我开始改善我的实现——从延迟开始。

## 减少延迟

人类对话有极其紧密的“响应时间”。面对面的对话尤其迅速，因为我们依赖视觉和听觉信号来确定我们何时参与对话。对话中的“平均”响应时间可以非常快，只有 200 毫秒。这意味着为了使对话代理程序感觉真实，它需要极快的“第一个发言词的时间”。

在发布我的原始演示之后，我已经知道有一些非常简单的优化方法可以实现，因此我着手尽可能在短时间内改善实现的平均延迟。首先，我需要至少一种方法来确定优化是否奏效。我的初步方法是使用[JavaScript 性能计时器](https://developer.mozilla.org/en-US/docs/Web/API/Performance/now)和日志记录。基本上，我从音频录制停止的确切时刻计算一个 `startTime`，从音频输出开始的确切时刻计算一个 `endTime`，然后将该时间记录到控制台。

这是一种非常不科学的做法。将来，我希望实现一个更加深入的分析和基准测试方法。尽管如此，在这个过程中，它表现得足够好。

接下来，我考虑了可能在流水线中引入延迟的所有领域。从录音停止的那一刻起，这些都是我们所采取的所有步骤：

1.  通过转换为 PCM 缓冲区预处理录音，然后转换字节序以匹配服务器（如果需要）

1.  将缓冲区上传到服务器

1.  对缓冲区执行语音转文字以生成文本

1.  发送文本到 LLM

1.  发送流式文本到 ElevenLabs

1.  接收来自 ElevenLabs 的流式音频

1.  向客户端广播音频

1.  在客户端解码音频并播放

这是一个可能引入延迟的很多步骤，包括潜在的 3 个（在我们的情况下是 2 个，因为我们拥有 STT 流水线）网络调用。

接下来，我想建立一个性能的“基线”。为了演示这个迭代过程，我在我的 M3 Mac CPU 上做了一个基准示例。请注意，与之前在 GPU 上运行的演示相比，这将会比较慢。在我的 Mac 上运行原始演示的基准性能为 `4537 ms`。4.5 秒的反馈时间。天啊。还有很多工作要做。

起初，我知道等待语音结束的 `SILENCE_TIMEOUT` 时间相当长。在最初的演示中，我使用了 1000 毫秒，这基本上意味着说话者必须停止说话整整一秒钟，然后我们才会开始长时间的响应过程。经过一些试验和错误，我发现 500 毫秒是一个“可以接受”的超参数。在将这个值调整下来后，延迟变化几乎完全与下降的相关性成正比：`4079 ms`。

我有一种预感我的文本到语音管道效率不高。幸运的是，ElevenLabs 给了我们一个很好的 [延迟指南](https://elevenlabs.io/docs/api-reference/reducing-latency)。第一个建议是通过指定 `eleven_turbo_v2` 来使用他们的 turbo 模型。我设置了这个值，我们得到了轻微的性能提升：`4014 ms`。

接下来，他们建议添加 `optimize_streaming_latency`。我将值设置为 `3`，我们得到了：`3791 ms`。他们的下一个建议是使用预制的语音。直到很久以后我才意识到我没有使用预制的语音，所以我没有比较这种变化对延迟的影响。

现在建议限制关闭 WebSocket 连接。我的当前实现每次讲话都会打开一个连接，这不是一个好的做法。基本上每一个“回合”都需要建立一个新的 WebSocket 连接。此外，ElevenLabs 从连接开始有 20 秒的超时时间。因此，你需要至少每 20 秒发送一条消息。在这一点上，我考虑了两个选项：

1.  打开一个全局 WebSocket 连接，或者甚至是一个连接池，并尝试保持连接活动。但这似乎非常浪费，并且我认为这不是他们 API 的预期使用方式。

1.  开始对话时打开一个 WebSocket 连接。我们不必担心 20 秒的暂停。

我决定选择第二个选项，但我仍然认为在生产系统中有一些缺点和需要考虑的地方。我使用的实现在第一次“讲话”时打开 WebSocket 连接，并将连接 PID 存储为 LiveView socket 中的一个分配。如果你的系统可能有许多并发用户在讲话，你就有可能创建一个潜在无限数量的连接的风险。一个更健壮的解决方案可能会使用连接池；然而，在这里，我并不真的担心流量或扩展问题。

在添加这个优化时，我遇到了一些困难，因为 ElevenLabs 会发送第一帧回来，然后就中断了。后来我意识到它是在等待生成，因为它以为我会发送更多的帧。所以我需要在发送完我的令牌后“刷新”生成。这似乎也解决了我遇到的不自然的音频问题。应用这一优化后，我们的首次说话时间在 `3700 ms` 左右。

在进一步研究他们的文档后，我了解到ElevenLabs将发送PCM缓冲而不是MP3。Web浏览器必须将MP3解码为PCM，这可能会引入一些额外开销。一个缺点是，你需要处于独立创作者层级才能接收PCM而不是MP3。现在，如果你想知道我是否花了99美元来节省一些毫秒的无意义演示，答案是绝对是的。

在这一点上，我相信我已经用完了很多关于TTS延迟的“简单”优化。有一件事让我不太高兴的是，ElevenLabs Websocket API没有办法接收二进制载荷而不是JSON载荷。这可能是因为他们发送了对齐数据，但我在这里没有使用对齐数据。当处理来自他们API的传入帧时，我们必须首先解码JSON，然后解码Base64编码的音频缓冲区。我不确定延迟的影响是什么，但我确信如果避免这两种转换，我们可以节省一些时间。我还认为Base64表示会导致稍大的缓冲区，这可能会影响网络延迟。

我接下来想要改进的是语音转文本管道。我特别使用`Nx.Serving`来进行语音转文本。这种方法的好处是我们可以避免因为转录而额外进行网络调用。当然，这假设我们的转录管道在我们自己的硬件上运行足够快。XLA在CPU上是出了名的慢（正在变得更好）。我做的第一个“优化”是切换到我的GPU：`2050 ms`

而这正是一个痛苦的教训，因为这是我们能得到的最大的性能提升。

接下来，我意识到模型没有使用F16，这可能会引入一些明显的加速：`1800 ms`。现在，我们可能还可以为Nx和EXLA添加一些额外的优化。例如，我们没有快速的注意力实现。当然，XLA在应用类似优化作为基线时表现得很出色，所以我不确定它会帮助多少。还有[Whisper的快速JAX实现](https://github.com/sanchit-gandhi/whisper-jax)，声称可以加速达到70倍。然而，这些声称的加速几乎总是针对长音频序列。GPU和TPU在大批量和序列长度方面表现良好，但不适合我们在这个实现中关心的单批量和短序列长度。有一天我可能会深入研究批量大小为1的快速转录性能，但今天不是那一天。

在这一点上，我已经开始改进我的演示的一些失败模式。在这样做的过程中，我学到了比以前更多的关于音频的知识，并意识到我用来录制音频的配置可以显著提高Whisper的性能。结果发现有一个[不错的指南](https://dev.to/mxro/optimise-openai-whisper-api-audio-format-sampling-rate-and-quality-29fj)介绍了一些可以工作的参数。具体来说，应该使用16kHz的采样率进行转录。减少采样率还应该减少网络开销，因为数据量减少了，但可能会降低转录的质量。哦，好吧。此外，我意识到我没有使用预制的ElevenLabs语音。在引入了这两个优化措施后，我能够实现`1520 ms`的转换时间。

最后，我意识到我所有的基准测试都是在开发服务器上进行的。我将我的Phoenix环境从`dev`切换到`prod`后得到了：`1375 ms`。因此，通过所有这些优化，我们在对话中的响应时间约为1.3秒。在对话中，开始感觉接近自然了。我还应该指出，这也是在Tailscale上运行的，所以我的Mac和运行在GPU上的服务器之间大约有100ms的ping。当我在我的GPU上本地运行时，我可以稳定地得到约`1000 ms`，有时`900 ms`的转换时间。不幸的是，这仍然无法与Retell的延迟匹配。据他们称，他们能够稳定地达到`800 ms`。关于这如何可能的问题，我最后有些感想。

我认为我可以改进实现的最大领域是使用一个更好的VAD实现，它依赖于小的活动滚动窗口而不是帧。我们可能可以使用20-30毫秒的窗口，这理论上可以提供480毫秒的延迟改进。我最终想探索这一点。

说实话，我认为这是一个显著的改进，我*可能*可以在这里停下来并完成它。

如果我继续下去，我将探索使用本地LLM与Nx和Bumblebee。Nx和Bumblebee支持像Mistral和Llama这样的LLM开箱即用。我们的文本生成服务支持流式处理。这意味着我们可能可以消除到OpenAI的任何网络延迟，而是在本地运行其中的3个模型中的2个。其中一个问题是，Nx目前没有任何量化推理支持（我承诺这是在进行中），所以我的单个4090不足以部署Whisper和Mistral。幸运的是，[Fly.io](https://fly.io/gpu)的人们很友好地给了我一些80GB的A100。我将在部署后发布演示 🙂

也许有一天我会实现StyleTTS2，并看看我们能通过完全本地推理流水线有多高效。

## 改善对话体验

有些人指出，我的原始演示与 Retell 的对话体验不同，他们完全正确。除了延迟之外，我的系统容易出现故障，会捕捉到系统声音、随机的键盘和鼠标点击声，并且对环境噪音不敏感。他们还支持回声、填充语音和中断，这在与他们的代理进行交互时增加了一些“真实感”元素。

现在我还没来得及添加回声或填充语音，但我能够对我使用的 VAD 算法进行一些微小的改进，并添加了对中断的支持。

### 使用 VAD 修复一些故障模式

看起来第一个失败模式是系统声音的回声。Nero 总是录制并在音频超过某个阈值后开始转录。在深入研究 `getUserMedia` API 后，我发现了 `echoCancellation`、`noiseSuppression` 和 `autoGainControl` 的选项。我意识到，为了优化，我可以指定麦克风的采样率。这是我从上一节添加的优化点。这些选项大多数情况下是浏览器默认开启的，但我还是显式地添加了它们：

```
const audioOptions = {
  sampleRate: SAMPLING_RATE,
  echoCancellation: true,
  noiseSuppression: true,
  autoGainControl: true,
  channelCount: 1,
};

navigator.mediaDevices.getUserMedia({ audio: audioOptions }).then((stream) => {
  ...
}

```

尽管有些帮助，但 Nero 仍然会捕捉到自己的音频。这可能需要更复杂的解决方案，但我继续处理下一个问题。

第二个明显的失败模式是它会捕捉键盘点击声，并且沉默超时很难调节。我第一次尝试修复这个问题是通过在每一帧中“平滑”音量来“忽略”音频中的大波动：

```
let sum = 0;
for (let i = 0; i < bufferLength; i++) {
  sum += (dataArray[i] - 128) * (dataArray[i] - 128);
}
const volume = Math.sqrt(sum / bufferLength);
const smoothedVolume = SMOOTHING_ALPHA * volume + (1 - SMOOTHING_ALPHA) * lastSmoothedVolume;

lastSmoothedVolume = smoothedVolume;

```

然后，在 [Paulo Valente](https://twitter.com/polvalente) 的建议下，我实施了一个双二阶滤波器，使用低通和高通滤波将音频过滤到人类语音的范围内：

```
this.stream = stream;
const analyser = this.audioContext.createAnalyser();
const microphone = this.audioContext.createMediaStreamSource(this.stream);

var highPassFilter = this.audioContext.createBiquadFilter();
highPassFilter.type = 'highpass';
highPassFilter.frequency.value = FILTER_LOWER_BOUND;

var lowPassFilter = this.audioContext.createBiquadFilter();
lowPassFilter.type = 'lowpass';
lowPassFilter.frequency.value = FILTER_UPPER_BOUND;

microphone.connect(highPassFilter);
highPassFilter.connect(lowPassFilter);
lowPassFilter.connect(analyser);

```

实际上，这两种解决方案似乎都效果不错，但肯定还有改进的空间。我知道可以通过使用一个滚动窗口来改进客户端过滤，该窗口会根据发言频率的能量相对于整个样本的能量进行判断。此外，还有一些机器学习模型可以执行 VAD，并且推断时间为 `1ms`。我意识到，通过分块方式将所有数据发送到 WebSocket，并在服务器端执行 VAD 可能更快。稍后我会详细讨论这个实现。

### 支持中断

接下来，我想要添加中断支持。在 Retell 的例子中，如果检测到你在讲话，说话人会在中途中断。为了在 Nero 中实现这一功能，我在麦克风钩子中添加了一个 `pushEvent`，每次检测到语音时会向服务器推送一个 `interrupt` 事件：

```
if (smoothedVolume > VOLUME_THRESHOLD) {
  if (!this.isRecording()) {
    // To handle interrupts, push an event to the LV which will
    // then push an event to the TTS channel. Not sure how much
    // these round trips will lag. Alternatively we could create
    // a global audio context and stop that, but we would need
    // a different way to push alignment info to the server
    this.pushEvent("interrupt");
    this.startRecording();
  }
  ...
}

```

服务器处理此事件，并向 TTS 通道广播事件以停止说话：

```
  def handle_event("interrupt", _params, socket) do
    NeroWeb.Endpoint.broadcast_from(
      self(),
      socket.assigns.audio_channel,
      "phx:audio-stop",
      %{}
    )

    {:noreply, socket}
  end

```

通道通过清除输出音频流和队列来处理事件：

```
  this.channel.on("phx:audio-stop", () => {
    if (hook.audioContext.state === 'running') {
      hook.audioContext.suspend().then(() => {
        if (hook.source) {
          hook.source.onended = null;
          hook.source.stop();
          hook.source = null;
        }
        hook.isPlaying = false;
        hook.audioQueue = [];
      });
    }
  });

```

不幸的是，这确实会造成竞态条件。可能出现的情况是，发言者打断并且客户端上的发言队列被清除，但 ElevenLabs 仍在向服务器流式传输音频。服务器总是会将这些信息广播到客户端，并且客户端会处理它。这可能会导致音频中出现奇怪的连续性问题。为了解决这个问题，我重构了 TTS 实现，使得每个音频广播都附加一个6位数的令牌到负载中。然后，我们只需要保持客户端和服务器的令牌同步即可。在客户端处理音频队列时，只需检查负载开头的令牌是否匹配，如果不匹配则忽略该样本。

这种实现的限制是它不会更新聊天记录。这完全可能是因为我们可以从 ElevenLabs 获取对齐信息，但我目前没有实现它。

### 时间基础的挂起

Retell 演示还支持在静默一段时间后的提示和挂断。如果你保持沉默太久，你会从 AI 扬声器那里得到一个提示，问你是否还在那里。在另一个静默期后，它会挂断。这在 LiveView 和 `Process.send_after/4` 中非常容易实现：

```
  defp nudge(socket) do
    nudge_pid = Process.send_after(self(), :nudge, @nudge_ms)
    assign(socket, :nudge_pid, nudge_pid)    
  end

```

每当我们收到转录时，我们可以随时取消计时器，并在每次说话后重新启动它。请注意，我们不能依赖 Phoenix `speak` 异步任务结束来触发发送提示。相反，我们需要从扬声器钩子推送一个事件，表明音频已结束。这避免了发言者启动一个非常长的讲话与 `nudge_ms` 持续时间重叠的情况。现在，我们可以通过分配来控制提示的数量。在我的情况下，我只是使用了一个布尔值：

```
  def handle_info(:nudge, socket) do
    socket =
      if socket.assigns.nudged? do
        stop_conversation(socket)
      else
        socket
        |> speak(["Are ", "you ", "still ", "there? "])
        |> assign(nudged?: true)
      end

    {:noreply, socket}
  end

```

## 重新做整个事情

某个时候，我意识到我在客户端进行可靠的 VAD 工程尝试永远无法提供我想要的体验。我与 [Andres Alejos](https://twitter.com/ac_alejos) 讨论了一下，他找到了一个 [Silero VAD](https://github.com/snakers4/silero-vad) 模型，能够在单个 CPU 线程上以 `1ms` 执行 VAD。他们还有一个 ONNX 模型—我们在 Elixir 生态系统中有一个叫做 [Ortex](https://github.com/elixir-nx/ortex) 的库，可以执行 ONNX 模型。

为了适应新的 VAD 模型，我最终重新实现了最初的 LiveView 作为 WebSocket。这实际上效果很好，因为 WebSocket 服务器是通用的，可以被任何具有 WebSocket 客户端的语言消费。实现也相对简单，可以轻松扩展以适应其他 LLMs、TTS 和 STT 模型。WebSocket 实现具有低延迟（在 GPU 上运行时），并支持中断。

您可以在[我的GitHub项目](https://github.com/seanmor5/echo)找到，以及一个[使用服务器的示例](https://github.com/seanmor5/echo_example)。

## 思索

最终实现的版本仍然无法与Retell演示的质量匹敌。尽管如此，我认为这是未来工作的一个良好起点。我认为在最初发布关于这个项目时有些傲慢，我想说Retell的工作不应被低估。我能理解制作有效对话代理所需的细节关注，Retell的演示显示他们对细节非常重视。向他们及其团队致敬。

我还要承认，我的演示正在针对一个基准进行优化。我在极力优化延迟，以支持单个用户——我自己。我认为如果需要支持多个并发用户，这个解决方案将会改变。

Retell的网站声称他们在后台有一个对话编排模型来管理对话的复杂性。在开始时，我对此表示怀疑，但现在我相信了。无论这个模型实际上是单一模型还是一系列用于VAD、添加反馈通道等的模型，我都不确定。我认为最终它将会成为一个单一模型，但我不确定现在是否已经是这样，这也带我到我的下一个观点。

### 另一个苦涩的教训

在进行所有这些优化的同时，我不禁在想，最终这些努力可能会付诸东流。不是因为我认为人们不会觉得它有用，而是因为大量数据训练的大模型似乎总是能够击败工程努力。我相信这个工作领域的未来在于联合模型。我认为实现实时对话的唯一方法是合并堆栈的部分。我预测不到一年内我们将看到一个非常有能力的联合语音/文本模型。我最近看到了一个称为[Qwen-Audio](https://github.com/QwenLM/Qwen-Audio)的大型音频模型，我认为这与我所设想的类似。

具体来说，如果有人愿意为我解决这个问题提供资金，这里是我将要做的事情：

1.  创建一个[Alpaca风格](https://github.com/tatsu-lab/stanford_alpaca/blob/main/alpaca_data.json)和/或LLaVA风格的合成语音数据集。请注意，需要进行一些预处理来改变Alpaca输入，以反映与口语兼容的风格。我将使用ElevenLabs在多个声音中生成数据集。当然，这个数据集可能过于“干净”，所以我们需要应用一些增强技术，如增加环境噪声、改变说话音调和速度等。额外加分点：添加一些“噪声”样本，无需响应以合并管道中的VAD部分。甚至可以添加文本提示，指导何时以及何时不需要响应，支持诸如[wake word detection](https://picovoice.ai/platform/porcupine/)之类的功能，而无需训练单独的模型。

1.  创建一个LLaVA风格的模型，使用Whisper或等效基础、LLM和投影层。

1.  安全的H100s，训练模型，并将“H100s变成100美元”（谢谢 [@thmsmlr](https://twitter.com/thmsmlr?lang=en)）

如果你想给我一些 $$$，我的电子邮件是 smoriarity.5@gmail.com 🙂

我认为我们也接近完全的语音对话模型。在创建这些模型时，我能看到的一个具体挑战是如何获得高质量的数据集。我认为，如果您试图“记录对话”以进行训练，实际上可能会得到一个质量较低的数据集。人们往往在观察下改变他们的行为。此外，电影和电视节目中的对话实际上并不非常自然。甚至一些播客也有不自然的对话节奏。

在和我的未婚夫一起观看 [《爱情是盲的》](https://en.wikipedia.org/wiki/Love_Is_Blind_(TV_series)) 时，我意识到你可能可以从真人秀中获得相当数量的高质量数据。真人秀中的对话过于夸张和混乱，但（我认为）比其他任何东西更接近现实。

### 对话知识库？

我确实很想知道在对话代理之上，一个稳固的RAG实现看起来是什么样子。RAG和复杂的CoT管道将引入潜伏期，这可能会损害对话体验。但是，你可以采用一些巧妙的方法来隐藏这一点。在需要人与人之间“搜索”的对话中，例如安排预约，通常会有一方在执行系统搜索之前说“请稍等片刻”。完全可以构建类似的功能。此外，如果您的代理需要关于个人的信息，可以在初始提示中包含这些信息。

### 你应该使用Elixir

我对这个问题特别兴奋，因为它实际上是Elixir和Phoenix的完美应用。如果您正在构建对话代理，应该认真考虑尝试Elixir。这个演示能够如此快速地完成很大程度上是因为Elixir的高效率。

## 结论

这是一个有趣的技术挑战。我对最终演示的表现感到满意。我也很高兴能够为其他人开发一个小型库。如果您对对话代理感兴趣，我鼓励您查看它，提供反馈并贡献！我知道现在还很粗糙，但随着时间的推移，它会变得更好。

此外，我计划定期扩展Nero项目的其余部分，请通过 [Twitter](https://twitter.com/sean_moriarity) 关注我以获取最新信息。
