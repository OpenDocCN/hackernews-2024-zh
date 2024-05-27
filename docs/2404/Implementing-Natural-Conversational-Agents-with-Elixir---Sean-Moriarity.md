<!--yml
category: 未分类
date: 2024-05-27 13:19:22
-->

# Implementing Natural Conversational Agents with Elixir – Sean Moriarity

> 来源：[https://seanmoriarity.com/2024/02/25/implementing-natural-conversational-agents-with-elixir/](https://seanmoriarity.com/2024/02/25/implementing-natural-conversational-agents-with-elixir/)

In [my last post](https://seanmoriarity.com/2024/02/25/nero-part-1-home-automations/), I discussed some work I had done building Nero, the assistant of the future that I’ve always wanted. I ended up creating an end-to-end example which used Nx, OpenAI APIs, and ElevenLabs to create an in-browser home automation assistant. For a first product, it’s decent. Nero is a neat little party trick that I can use to impress my non-tech friends. I am, however, not in this business to impress my friends. I want Nero to *actually help me* and *actually feel like an assistant*. My previous version is not that.

One missing piece is the ability to converse naturally without browser interaction. The first implementation of Nero’s “conversational” abilities relied on user interaction with the screen every time we wanted to initiate a response or action. Nero also did not retain any conversational history. In short, Nero was not a great conversational assistant. It was one of the things I wanted to fix; however, I was motivated to do it sooner rather than later after watching [an impressive demo from Retell](https://beta.retellai.com/home-agent).

The Retell demo implements a conversational agent backed by their WebSocket API in a browser. The demonstration has:

*   “Always on” recording
*   Low latency
*   Support for interrupts
*   Impressive filtering (e.g. snapping and other non-voice activity doesn’t seem to throw off the agent)

Their documentation suggests they also have support for [backchanneling](https://www.cs.utep.edu/nigel/bc/#:~:text=What%20is%20a%20Backchannel%3F,utterances%20such%20as%20uh%2Dhuh.) and intelligent end of turn detection—two things that are essential to natural conversational feel but which are very difficult to express programmatically.

I had previously convinced myself that I could implement a passable conversational agent experience in a short amount of time. So that is what I set out to do.

## Always On Recording

The first thing that needed to change about Nero’s design was the speech to text pipeline. My original demonstration relied on an [example from Bumblebee](https://github.com/elixir-nx/bumblebee/blob/main/examples/phoenix/speech_to_text.exs) which implemented a speech to text pipeline using Whisper. The pipeline uses mouse events in a [Phoenix LiveView Hook](https://hexdocs.pm/phoenix_live_view/js-interop.html#client-hooks-via-phx-hook) to start and stop recordings before sending them to the server to initiate transcription. If you’re not familiar, [Phoenix LiveView](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html) is a server-side rendering framework built on top of Elixir. LiveView has support for client-side JavaScript hooks which support bidirectional communication between client and server.

The original speech to text implementation used a hook with an event listener attached to `mousedown` and `mouseup` on a button to start and stop recording. After recording stops, the hook decodes the recorded buffer into a PCM buffer, converts the endianness, and then pushes the buffer to the server with an upload. The original hook implements most of the functionality we want; however, we need to make some minor tweaks. Rather than trigger recordings to stop and start on mouse events, we want to trigger recordings to start and stop exactly when a person starts and stops speaking. Simple, right?

My first idea in implementing what I called “always on recording” was to monitor the microphone’s volume, and trigger a recording when the volume reached a certain threshold. The recording would stop when the volume dipped below that threshold. At this point, I learned about [getUserMedia](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia). `getUserMedia` prompts the user for permission to access media devices such as a microphone and/or webcam, and then produces a `MediaStream`. A `MediaStream` is a stream of media content containing information about audio and video tracks in the stream. We can use data from the `MediaStream` to determine speaker activity and thus trigger recordings.

To determine the volume for a given sample, we can use an [AnalyserNode](https://developer.mozilla.org/en-US/docs/Web/API/AnalyserNode). Per the documentation `AnyalyserNode` is designed for processing generated audio data for visualization purposes, but we can use it to determine spikes in audio:

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

This uses an analyser and repeatedly checks if the volume of the microphone at a given frame exceeds the given `VOLUME_THRESHOLD`. If it does, it checks to see if we are recording and if not, starts the recording.

After testing a bit, I realized this implementation sucked. Of the many issues with this approach, the biggest is that there are many natural dips in a person’s volume. Checking a single frame doesn’t account for these natural dips. To fix this, I thought it would be a good idea to introduce a timeout which only stopped recording after the volume was below a threshold for a certain amount of time:

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

This actually ended up working decent, but required tuning hyperparameters for both `VOLUME_THRESHOLD` and `SILENCE_TIMEOUT`. The challenge here is that higher `SILENCE_TIMEOUT` introduces additionally latency in transition time between a speaker and Nero; however, lower timeouts might be too sensitive to speakers with slower and quieter speaking rhythms. Additionally, a static `VOLUME_THRESHOLD` does not account for ambient noise. Now, despite these shortcomings, I found I was able to passably detect a single speaker in a quiet room.

After hooking this up to my existing LiveView and trying some end-to-end conversations, I realized something was significantly off. The transcriptions I was getting were off. I soon realized that they were always off at the beginning of a transcription. Shorter audio sequences were especially affected. It turns out that the detection algorithm always resulted in some amount of truncation at the beginning of an audio clip. When a speaker starts talking, their volume ramps up – it’s not an instantaneous spike. To account for this, I introduced a pre-recording buffer which always tracked the previous 150ms of audio. After recording started, I would stop the pre-recording buffer and start the actual recording, and then eventually splice these 2 together to send to the server for transcription.

Overall, this *actually* worked okay. While there are some obvious failure modes, it worked well enough to get a passable demonstration. If you can’t tell by now, I am not an audio engineer. I learned later that this is a very naive attempt at [voice activity detection](https://en.wikipedia.org/wiki/Voice_activity_detection). Later on in this post, I’ll run through some of the improvements I made based on my research into the field of VAD.

## End-to-End Implementation

The demonstration I built for Nero in my [first post](https://seanmoriarity.com/2024/02/25/nero-part-1-home-automations/) already contained the scaffolding for an end-to-end transcription -> response -> speech pipeline. I only needed to make some slight modifications to get the phone call demo to work. The end-to-end the pipeline looks like this:

When our algorithm detects that speech has stopped, it invokes the `stopRecording` method. `stopRecording` takes the recorded audio, does some client-side pre-processing, and uploads it to the server. The server consumes the uploaded entry as a part of [LiveView’s normal uploads lifecycle](https://hexdocs.pm/phoenix_live_view/uploads.html) and then invokes an [async task](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#module-arbitrary-async-operations) to start transcription:

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

Note that because we did most of the pre-processing client-side, we can just consume the audio binary as an `Nx.Tensor`, without any additional work. The `SpeechToText` module implements transcription using [Nx.Serving](https://hexdocs.pm/nx/Nx.Serving.html):

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

`Nx.Serving` is an abstraction in the Elixir Nx ecosystem for serving machine learning models directly in an Elixir application. It implements dynamic batching, encapsulates pre-processing, inference, and post-processing, supports distribution and load-balancing between multiple GPUs natively, and in general is an extremely easy way to serve machine learning models.

After transcription completes, we get an async result we can handle to initiate a response:

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

Here `Nero.Agent.respond/1` returns an Elixir `Stream` of text. For my original demonstration I just used the Elixir OpenAI library to produce a stream from a GPT-3.5 response:

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

The response stream is consumed by `speak/2`. `speak/2` implements the text to speech pipeline:

```
  defp speak(socket, text) do
    start_async(socket, :speak, fn ->
      Nero.TextToSpeech.stream(text)
    end)
  end

```

Where `Nero.TextToSpeech.stream/1` uses the [ElevenLabs WebSocket API](https://elevenlabs.io/docs/api-reference/websockets) to stream text in and speech out. You can read a bit more about the implementation in my previous post.

`Nero.TextToSpeech.stream/1` returns the consumed response as text so we can append that to the chat history after the `:speak` task finishes:

```
def handle_async(:speak, {:ok, response}, socket) do
  chat = socket.assigns.chat ++ [%{role: "assistant", content: response}]
  {:noreply, assign(socket, :chat, chat)}
end

```

This is basically all of the scaffolding needed for an end-to-end demo, but I wanted to add a few more features. First, I wanted to support “intelligent” hang-ups. Basically, I wanted to be able to detect when a conversation was finished, and stop the recording. To do that, I used [Instructor](https://github.com/thmsmlr/instructor_ex):

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

Please ignore my wonderfully engineered prompt. This uses GPT-3.5 to determine whether or not a given conversation has ended. After every one of Nero’s turns, we check the transcript to possibly end the call:

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

This pushes a `hang_up` event to the socket:

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

Which stops the recording, and then pushes an event to `toggle_conversation` back to the server. `toggle_conversation` implements the start/stop logic from the server:

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

Finally, I wanted to implement information extraction from the transcript. Again, I used instructor and defined an extraction schema:

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

And used GPT-3.5 with a rough prompt to get the necessary information from the transcript:

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

And then anytime a conversation ends, we attempt to retrieve appointment information:

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

Now this is essentially the exact implementation that produced [this demonstration](https://twitter.com/sean_moriarity/status/1760435005119934862). End-to-end this amounted to a couple of hours of work; however, I already had most of the basic scaffold implemented from my previous work on Nero. In my biased opinion, I think my demo is pretty good, but as others have pointed out Retell’s demo kicks my ass in:

*   Latency
*   Reliability
*   Natural sounding voice

And so, I set out to improve my implementation – starting with latency.

## Reducing Latency

Human conversations have extremely tight “time-to-turn.” In-person conversations are especially rapid because we rely on visual as well as audio signals to determine when it’s our time to participate in a conversation. The “average” time-to-turn in a conversation can be as quick as 200ms. That means for a conversational agent to feel realistic, it needs an extremely quick turn around time for “time to first spoken word.”

After posting my original demonstration, I already knew there were some very easy optimizations I could make, so I set out to improve the average latency of my implementation as much as possible in a short amount of time. First, I needed at least some method for determining whether an optimization worked. My rudimentary approach was to use [JavaScript Performance Timers](https://developer.mozilla.org/en-US/docs/Web/API/Performance/now) and logging. Basically, I computed a `startTime` from the exact moment an audio recording stopped and an `endTime` from the exact moment an audio output started, and then I logged that time to the console.

This is a very unscientific way of doing business. In the future, I’d like to implement a much more involved profiling and benchmarking methodology. For this process though, it worked well enough.

Next, I considered all of the areas that could introduce latency into the pipeline. From the moment a recording stops, these are all of the steps we take:

1.  Pre-process recording by converting to PCM buffer, and then converting endianness to match server (if necessary)
2.  Upload buffer to server
3.  Perform speech to text on buffer to produce text
4.  Send text to LLM
5.  Send streamed text to ElevenLabs
6.  Receive streamed audio from ElevenLabs
7.  Broadcast audio to client
8.  Decode audio on client and play

That’s a lot of steps that can introduce latency, including potentially 3 (in our case 2 because we own the STT pipeline) network calls.

Next, I wanted to esablish a “baseline” of performance. To demonstrate this iterative process, I did a baseline example on my M3 Mac CPU. Note that this is going to be slow relative to my previous demo because the previous demo runs on a GPU. The baseline performance I got from the original demo running on my mac was `4537 ms`. 4.5 seconds turn around time. Yikes. Lots of work to do.

To start, I knew that the `SILENCE_TIMEOUT` used to wait for speech to end was rather long. For the original demo, I used 1000 ms, which basically means a speaker has to stop talking for a full second before we’ll even start the long response process. After some trial and error, I figured 500 ms was a “passable” hyperparameter. After adjusting this down, the latency change was almost exactly correlated to the dip: `4079 ms`.

I had a hunch that my text to speech pipeline was not efficient. Fortunately, ElevenLabs gives us a nice [Latency Guide](https://elevenlabs.io/docs/api-reference/reducing-latency). The first suggestion is to use their turbo model by specifying `eleven_turbo_v2`. I set that and we got a slight performance boost: `4014 ms`.

Next, they suggest adding `optimize_streaming_latency`. I set the value to `3` and we get: `3791 ms`. Their next suggestion is to use a pre-made voice. I actually didn’t realize until much later that I was not using a pre-made voice so I don’t have a comparison for how that change impacted latency.

Now it says to limit closing WebSocket connections. my current implementation opens a connection everytime it speaks – which is not good. Basically every “turn” has to establish a new websocket connection. Additionally, ElevenLabs has a timeout of 20s from when you connect. So you need to send a message at least every 20s. I considered 2 options at this point:

1.  Open a global WebSocket connection, or maybe even a pool of connections, and try to keep the connection alive. But that seems really wasteful, and I don’t think is the intended use of their API
2.  Open a WebSocket connection when convo starts. We don’t have to worry about 20s pauses

I decided to go with option 2, but I still think there are some drawbacks and considerations for a production system. The implementation I used opens a websocket connection on first “speak” and stores the connection PID as an assign in the LiveView socket. If you have a system with potentially many concurrent users speaking, you run the risk of creating a potentially unbounded number of connections. A more robust solution would probably use connection pools; however, I’m not really worried about traffic or scaling here.

While adding this optimization, I struggled a bit because ElevenLabs would send the first frame back, then cut off. Then I realized that it was waiting to generate becuase it thought I was going to send more frames. So I needed to “flush” the generation after I finished sending my tokens. This also seemed to fix unnatural audio problems I was having. After applying this optimization, our time to first spoken word was slightly lower in the `3700 ms` range.

After perusing their docs a bit more, I learned that ElevenLabs will send PCM buffers instead of MP3\. Web Browser’s have to decode MP3 to PCM, which potentially introduces some overhead. One drawback is that you need to be on the independent creator tier to receive PCM instead of MP3\. Now, if you’re wondering if I spent $99 to save some milliseconds for a meaningless demo, the answer is absolutely yes I did.

At this point, I believe I’ve exhausted a lot of the “easy” optimizations for TTS latency. One thing that does bother me about the ElevenLabs Websocket API is that there’s no way to receive binary payloads instead of JSON payloads. This is probably because they send alignment data, but I’m not using the alignment data here. When handling an incoming frame from their API we have to first decode the JSON, and then decode the Base64 encoded audio buffer. I’m not sure what the latency impact is, but I’m sure we could shave *some* time by avoiding both of these conversions. I also think the Base64 representation results in slightly larger buffers which could impact network latency.

The next area I looked to improve was the speech-to-text pipeline. I am using `Nx.Serving` specifically for Speech-to-Text. The benefit of this approach is that we can avoid an additional network call just for transcription. Of course, that assumes our transcription pipeline can run fast enough on our own hardware. XLA is notoriously slow on CPUs (it’s getting better). The first “optimization” I did was to switch to my GPU: `2050 ms`

And that right there is a bitter lesson, because it’s the largest performance boost we’re going to get.

Next, I realized the model isn’t using F16, which can introduce some solid speed-ups: `1800 ms`. Now, there are probably some additional optimizations we could add to Nx and EXLA specifically. For example, we don’t have a flash attention implementation. Of course, XLA does a great job of applying similar optimizations as a baseline, so I’m not sure how much it would help. There’s also [fast JAX implementations of Whisper](https://github.com/sanchit-gandhi/whisper-jax) that claim up to 70x speed ups. One issue with a lof of these claimed speed-ups; however, is that they are almost **always** for long audio sequences. GPUs and TPUs do well with large batch sizes and sequence lengths, but not for batch size 1 and short sequence lengths like we care about in this implementation. One day I may go down the performance hole of fast batch size 1 transcription, but today is not that day.

At this point, I had moved on to improving some of the failure modes of my demo. While doing so, I learned much more about audio than I had previously known, and realized that the configuration I used to record audio can significantly improve whisper performance as well. Turns out there’s a [nice guide](https://dev.to/mxro/optimise-openai-whisper-api-audio-format-sampling-rate-and-quality-29fj) of somebody discussing parameters that work. Specifically, you should use 16 kHz sample rate for transcriptions. Reducing the sample rate also should reduce network overhead because we have less data, but it could reduce quality of the transcription. Oh well. Additionally, I realized I wasn’t using a pre-made ElevenLabs voice. After introducing both of these optimizations, I was able to achieve `1520 ms` turn time.

Finally, I realized I was doing all of my benchmarks on a development server. I switched my phoenix environment from `dev` to `prod` and got: `1375 ms`. So, with all of these optimizations we’re sitting at about 1.3s turn around time in a conversation. When conversing, it starts to feel somewhat close to natural. I should also point out that this is also running over Tailscale, so there is about 100 ms ping between my Mac and the server running on my GPU. When I run this locally on my GPU, I can consistently get about `1000 ms` and sometimes `900 ms` turn around time. Still, unfortunately, this does not match Retell’s latency. According to them, they are able to achieve `800 ms` consistently. I have some musings at the end about how this is possible.

I believe the biggest area I could improve the implementation is to use a better VAD implementation that relies on small rolling windows of activity rather than frames. We could probably get away with using 20-30 ms windows, which could theoretically offer a 480 ms latency improvement. I would like to eventually explore this.

In all honesty though, I think that is a significant improvement, and I could *probably* stop right here and be done with it.

If I were to keep going, I would explore using a local LLM with Nx and Bumblebee. Nx and Bumblebee support LLMs like Mistral and Llama out-of-the box. And our text generation servings support streaming. That means we can possibly eliminate any network latency to OpenAI, and instead run 2 of the 3 models locally. One issue with this is that Nx currently does not have any quantized inference support (it’s coming I promise), so my single 4090 is not sufficient to deploy both Whisper and Mistral. Fortunately, the folks at [Fly.io](https://fly.io/gpu) were kind enough to give me access to some 80GB A100s. I will post a demo when I get one deployed 🙂

Maybe one day I will implement StyleTTS2 and see how efficient we can get with an entirely local inference pipeline.

## Improving the Conversational Experience

Some people pointed out that my original demo did not have the same conversational experience as Retell’s, and they are absolutely right. Aside from latency, mine was prone to failure, picks up system sounds, picks up random noises like keyboard and mouse clicks, and doesn’t do well with ambient noise. They also have support for backchanneling, fillers and interruptions which introduces some element of “realness” when interacting with their agent.

Now I didn’t get around to adding backchannels or fillers, but I was able to make some slight improvements to the VAD algorithm I used, and I added support for interruptions.

### Fixing Some Failure Modes with VAD

The first failure mode that seems to happen is echo from the system sounds. Nero always records and will start transcribing after audio spikes over a certain threshold. After some digging into the `getUserMedia` API, I found options for `echoCancellation`, `noiseSuppression`, and `autoGainControl`. This is the same point I realized that I could specify the microphone sample rate for the optimization I could added from the last section. Most of these options are on by default depending on your browser, but I added them explicitly anyway:

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

Now that somewhat helped, but Nero still picks up it’s own audio. This probably requires a more sophisticated solution, but I moved on to the next problem.

The second obvious failure mode is the fact that it picks up keyboard clicks, and the silence timeout is hard to tune. My first attempt to fix this was to “ignore” large spikes in audio by “smoothing” the volume at each frame:

```
let sum = 0;
for (let i = 0; i < bufferLength; i++) {
  sum += (dataArray[i] - 128) * (dataArray[i] - 128);
}
const volume = Math.sqrt(sum / bufferLength);
const smoothedVolume = SMOOTHING_ALPHA * volume + (1 - SMOOTHING_ALPHA) * lastSmoothedVolume;

lastSmoothedVolume = smoothedVolume;

```

Then, with some advice from [Paulo Valente](https://twitter.com/polvalente), I implemented a biquad filter to with a low and high-pass in order to filter audio to the range of human speech:

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

In practice, both of these solutions actually seemed to work decent, but they could absolutely be better. I know it’s possible to improve the client-side filtering using a rolling-window that looks energy of the speaking frequences relative to energy of an entire sample. But, there are also machine learning models that perform VAD, and have `1ms` inference times. I realized that it’s probably quicker to just send all of the data over the websocket in chunks, and perform VAD on the server. I’ll discuss that implementation a little later.

### Supporting Interruptions

Next I wanted to add support for interruptions. In the Retell example, the speaker will cut off mid-speech if it detects that you are speaking. To implement this feature in Nero, I added a `pushEvent` to the Microphone hook which would push an `interrupt` event to the server anytime speech is detectected:

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

The server handles this event and broadcasts an event to the TTS channel to stop speaking:

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

And the channel handles the event by clearing out the output audio stream and queue:

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

Unfortunately, this does create a race condition. There’s a potential situation where a speaker interrupts and the speaking queue gets cleared on the client, but ElevenLabs is still streaming audio back to the server. The server is always going to just broadcast this info to the client, and as is the client will process it. This potentially creates a situation with weird continutations in the audio. To get around this, I refactored the TTS implementation so that each audio broadcast appends a 6 digit token to the payload. Then, all we need to do is keep the token in sync with the client and server. On the client, when processing the audio queue, it simply checks whether or not the token at the beginning of the payload matches, and if it doesn’t it ignores that sample.

The limitation with this implementation is it does not update the chat transcript. It’s entirely possible because we have access to the alignment information from ElevenLabs, but I just didn’t implement it at this time.

### Time-based Hang Ups

Another thing the Retell demo has support for is cues and hang ups after a duration of silence. If you are silent for too long, you’ll get a cue from the AI speaker asking you if you’re still there. After another duration of silence, it will hang up. This is something that’s pretty easy to do with LiveView and `Process.send_after/4`:

```
  defp nudge(socket) do
    nudge_pid = Process.send_after(self(), :nudge, @nudge_ms)
    assign(socket, :nudge_pid, nudge_pid)    
  end

```

And then we can cancel the timer anytime we receive a transcription, and restart it after every turn speaking. Note that we can’t depend on the Phoenix `speak` async task ending as the trigger to send nudges. Instead, we need to push an event from the speaker hook that the audio has ended. This avoids a case where the speaker initiates a really long speech, which overlaps with the `nudge_ms` duration. Now, we can control the number of nudges with an assign. In my case, I just used a boolean:

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

## Re-doing the Entire Thing

Somewhere along the line I realized that my attempts at engineering solid VAD client-side were never going to deliver the experience that I wanted. I discussed with [Andres Alejos](https://twitter.com/ac_alejos) a bit, and he found a [Silero VAD](https://github.com/snakers4/silero-vad) model which is capable of performing VAD in `1ms` on a single CPU thread. They also had an ONNX model—and we have a library in the Elixir ecosystem called [Ortex](https://github.com/elixir-nx/ortex) which allows us to execute ONNX models.

To accomodate for the new VAD model, I ended up re-implementing the original LiveView I had as a WebSocket. This actually works out well because the WebSocket server is generic, and can be consumed by any language with a WebSocket client. The implementation is also relatively simple, and easily expanded to accomodate for other LLMs, TTS, and STT models. The WebSocket implementation has low latency (when running on a GPU), and supports interrupts.

You can find the [project on my GitHub](https://github.com/seanmor5/echo) as well as an [example using the server](https://github.com/seanmor5/echo_example).

## Musings

The final implementation I ended up with still does not match the quality of the Retell demo. That said, I think it’s a solid start for future work. I believe I acted with some hubris when first posting about this project, and I would like to say that Retell’s work should not be understated. I can appreciate the attention to detail that goes into making an effective conversational agent, and Retell’s demo shows they paid a lot of attention to the details. Kudos to them and their team.

I will also admit that my demo is playing to one benchmark. I’m optimizing the hell out of latency to support a single user—me. I think this solution would change if it needed to accomodate for multiple concurrent users.

Retell’s website claims they have a conversation orchestration model under the hood to manage the complexities of conversation. I had my doubts about that going into this, but I believe it now. Whether or not this model is actually a single model or a series of models for VAD, adding backchannels, etc. I’m not sure. I think eventually it *will* be a single model, but I’m not sure if it is now, which leads me to my next point.

### Another Bitter Lesson

While doing all of these optimizations, I could not help but think that it will eventually be all for naught. Not because I don’t think people will find it useful, but because large models trained on lots of data simply seem to always beat engineering effort. I believe the future of this area of work is in joint models. I think the only way to achieve real-time conversations is to merge parts of the stack. I predict in less than a year we will see an incredibly capable joint speech/text model. I recently saw a large audio model called [Qwen-Audio](https://github.com/QwenLM/Qwen-Audio) that I believe is similar to what I envision.

Specifically, if somebody were kind enough to give me some money to throw at this problem, here is exactly what I would do:

1.  Generate an [Alpaca-style](https://github.com/tatsu-lab/stanford_alpaca/blob/main/alpaca_data.json) and/or LLaVA-style dataset of synthetic speech. Note that it would require a bit of pre-processing to change Alpaca inputs to mirror a style compatible with spoken-word. I would use ElevenLabs to generate the dataset in mulitple voices. Of course this dataset would be a bit too “clean,” so we’d need to apply some augmentations which add ambient noise, change speaking pitch and speed, etc. Bonus points: adding samples of “noise” which require no response to merge the VAD part of the pipeline in as well. You can even throw in text prompts that dictate when and when not to respond to support things like [wake word detection](https://picovoice.ai/platform/porcupine/) without needing to train a separate model.
2.  Create a LLaVA-style model with a Whisper or equivalent base, an LLM, and a projection layer.
3.  Secure H100s, train model, and “turn H100s into $100s” (thank you [@thmsmlr](https://twitter.com/thmsmlr?lang=en))

If you want to give me some $$$, my e-mail is smoriarity.5@gmail.com 🙂

I believe we are also close to just having full-on speech-to-speech models. A specific challenge I can see when creating these models is coming up with a high-quality dataset. I think if you make a deliberate attempt at “recording conversations” for the purposes of training, you will actually probably end up with a lower-quality dataset. People tend to change their behavior under observation. Additionally, conversations from movies and TV shows aren’t actually very natural. Even some podcasts have an unnatural converastional rhythm.

While watching [Love is Blind](https://en.wikipedia.org/wiki/Love_Is_Blind_(TV_series)) with my fiancé, I realized you could probably get a decent amount of quality data from reality tv shows. The conversations in reality TV are overly dramatic and chaotic, but are (I think) closer to realistic than anything else.

### Conversational Knowledge Base?

I do wonder what a solid RAG implementation looks like on top of a conversational agent. RAG and complex CoT pipelines will introduce latency which could deteriorate the conversational experience. However, there are clever ways you can hide this. In conversations that require “search” between humans, e.g. like scheduling an appointment, you’ll often have one party saying “one moment please” before performing a system search. Building something like that in is entirely possible. Additionally, if your agent requires information up front about an individual, it’s possible to include that in the initial prompt.

### You Should Use Elixir

I was very excited for this problem in particular because it’s literally the perfect application of Elixir and Phoenix. If you are building conversational agents, you should seriously consider giving Elixir a try. A large part of how quick this demo was to put together is because of how productive Elixir is.

## Conclusion

This was a fun technical challenge. I am pleased with the performance of the final demonstration. I’m also happy I was able to OSS a small library for others to build off of. If you are interested in conversational agents, I encourage you to check it out, give feedback, and contribute! I know it’s very rough right now, but it will get better with time.

Additionally, I plan to periodically build out the rest of the Nero project, so please follow me on [Twitter](https://twitter.com/sean_moriarity) if you’d like to stay up to date.