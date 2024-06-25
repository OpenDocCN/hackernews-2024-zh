<!--yml

类别：未分类

日期：2024-05-27 14:47:21

-->

# 音频库的挑战 | Tesselode Labs

> 来源：[`tesselode.github.io/articles/audio-libraries-considered-challenging/`](https://tesselode.github.io/articles/audio-libraries-considered-challenging/)

# 音频库的挑战

发布于 2022 年 5 月 16 日 | 更新于 2022 年 5 月 17 日

阅读时间：16 分钟

我开发了一个名为[Kira](https://github.com/tesselode/kira/)的游戏音频库。以下是我已经摸索出来的一些难点。如果你因某种原因决定制作一个音频库，可以从我的实验中学习！

## 为什么音频很难

在图形方面，不同的游戏有不同的可接受帧率。大多数人会认为 60 帧每秒看起来“流畅”。但即使帧率稍微低于 60 帧每秒，也不是世界末日。游戏将继续显示先前渲染的帧，直到下一个帧准备好。如果帧率下降得足够小，玩家甚至可能都不会注意到。

但是如果你的游戏无法像操作系统期望的那样快速生成音频（这被称为缓冲区不足），操作系统别无选择，只能用沉默填补这些间隙。而玩家*会*注意到的。

一小段音乐正常播放

一小段音乐回放时出现了缓冲区不足

如果你无法听取示例音频，请想象当游戏帧率下降时，显示器将为游戏无法快速产生的所有帧显示黑屏。音频卡顿就相当于那个。

写音频代码时，你要尽一切可能避免缓冲区不足。这意味着**你必须在一个独立的线程上进行音频处理。** 如果你试图在与图形和输入相同的线程上进行音频处理，当图形渲染变得过于要求时，音频会出现卡顿。

**你也不能阻塞音频线程。** 如果有什么事情会导致音频线程暂停未知的时间，那就不应该这样做。如果音频线程暂停时间过长，它将无法快速处理音频，导致缓冲区不足。

值得注意的是，这意味着**你不能在音频线程上分配或释放内存**。当你要求操作系统分配或释放内存时，你必须暂停线程，直到操作系统准备好处理你的请求。通常它会很快完成，但如果系统负担过重，操作系统可能会降低音频线程的优先级，导致长时间内无法处理任何音频。

在牢记这两个约束条件的情况下，让我们看一些在创建游戏音频库时会遇到的问题。

## 在线程间通信

一个游戏音频库提供了可以从游戏代码中调用的播放和修改音频的函数。它的结构大致如下：

```
let mut sound_handle = audio.play("sound.mp3");
sound_handle.set_volume(0.5); 
```

当我们播放新声音时，我们需要在游戏线程上从文件中加载音频数据并将其发送到音频线程。（我们不能在音频线程上加载音频数据，因为那太耗时了，可能会导致缓冲区欠载。）为了将音频数据发送到音频线程，我们可以使用一个环形缓冲区，比如[`ringbuf`](https://crates.io/crates/ringbuf)包。

```
audio_producer.push(audio_data);
while let Some(audio_data) = audio_consumer.pop() {
  play_audio(audio_data); } 
```

但是我们如何告诉音频线程修改现有声音的状态（例如，设置正在播放的声音的音量或设置声音的播放状态）？

### 通过`Mutex`共享所有权

我们可以通过使用`Mutex`直接让游戏线程控制音频线程上的数据来实现共享所有权。

在音频线程上，我们将声音状态存储为`Arc<Mutex<Sound>>`。我们在游戏线程上的声音句柄将具有该`Arc<Mutex<Sound>>`的克隆：

```
struct SoundHandle {
  playback_state: PlaybackState,
  sound: Arc<Mutex<Sound>>, } 
```

要在游戏线程上访问音频数据，我们只需锁定`Mutex`：

```
impl SoundHandle {
  pub fn set_playback_state(&self, playback_state: PlaybackState)  {
    sound.lock().unwrap().playback_state = playback_state; }

  pub fn set_volume(&self, volume: f32)  {
    sound.lock().unwrap().volume = volume; } } 
```

当然，音频线程在访问数据之前也必须锁定数据。事实上，等待其他线程解锁数据会阻塞音频线程，这绝对是我们绝不能做的事情之一。所以`Mutex`是不行的。

### 通过原子方式共享所有权

有一种方法可以在多个线程之间共享数据而无需锁定它：原子。原子是 CPU 知道如何在线程之间保持同步的基本类型的特殊版本。

我们可以使声音的每个可修改字段原子化：

```
struct Sound {
  playback_state: Arc<AtomicU8>,
  volume: Arc<AtomicU32>, } 
```

声音句柄将获得每个字段的克隆：

```
struct SoundHandle {
  playback_state: Arc<AtomicU8>,
  volume: Arc<AtomicU32>, } 
```

并且要从游戏线程设置这些字段：

```
impl SoundHandle {
  pub fn set_playback_state(&self, playback_state: PlaybackState)  {
    self.playback_state.store(playback_state as u8, Ordering::SeqCst); }

  pub fn set_volume(&self, volume: f32)  {
    self.volume.store(volume.to_bits(), Ordering::SeqCst); } } 
```

使用原子不会阻塞音频线程，但它确实有一些限制。最大的原子是 64 位。这足以存储音量级别或播放状态，但是如果我们想要向音频线程发送更复杂的命令怎么办？例如，如果我们想要在一段时间内平滑调整声音的音量？甚至带有用户指定的缓和曲线？

如果我们将该命令所需的所有信息表示为结构体，它将类似于这样：

```
struct VolumeChange {
  volume: f32,   duration: Duration,   easing: Easing, } 
```

那已经超过了我们可以放入一个原子中的内容。我们可以将命令存储在多个原子中，但那么我们就必须保持它们同步。如果我们限制最长持续时间，也许我们可以将其存储在 16 位中。我确信这是一个可以解决的问题，但解决方案可能不太符合人体工程学。那我们还能做什么呢？

### 使用更多的环形缓冲区

为什么不直接通过环形缓冲区发送命令呢？我们已经在用它们发送音频数据了。

我们可以用枚举描述所有可能的命令：

```
enum SoundCommand {
  SetPlaybackState(PlaybackState),
  SetVolume(f32), } 
```

声音句柄将拥有一个可以推送命令的命令生成器：

```
struct SoundHandle {
  command_producer: Producer<SoundCommand>, }

impl SoundHandle {
  pub fn set_playback_state(&self, playback_state: PlaybackState)  {
    self.command_producer.push(
      SoundCommand::SetPlaybackState(playback_state), ); }

  pub fn set_volume(&self, volume: f32)  {
    self.command_producer.push(SoundCommand::SetVolume(volume)); } } 
```

而音频线程将拥有一个可以从中弹出命令的命令消费者：

```
struct Sound {
  playback_rate: PlaybackRate,
  volume: f32,
  command_consumer: Consumer<SoundCommand>, }

impl Sound {
    pub fn update(&mut self)  {
    while let Some(command) = self.command_consumer.pop() {
      match command {
        SoundCommand::SetPlaybackRate(playback_rate) => {
          self.playback_rate = playback_rate; }
        SoundCommand::SetVolume(volume) => {
          self.volume = volume; } } } } } 
```

这种方法也有一个缺点：每个声音都必须定期轮询新命令。大多数声音在任何一个时间点都不会被改变，所以它们都必须轮询命令似乎是一种浪费。（而且在我的非科学基准测试中，所有的轮询确实会对性能产生明显影响。）

或许我们可以只使用一个环形缓冲区来收集每个声音的命令？我们已经有一个用于发送音频数据的环形缓冲区，所以让我们将其扩展为发送一个命令枚举：

```
Command {
  PlaySound(AudioData),
  SetSoundPlaybackRate(PlaybackRate),
  SetSoundVolume(f32), } 
```

当然，我们需要一种方法告诉音频线程我们想要更改哪个声音，所以让我们给这些命令添加一些唯一的标识符。

```
Command {
  PlaySound(AudioData),
  SetSoundPlaybackRate(SoundId, PlaybackRate),
  SetSoundVolume(SoundId, f32), } 
```

每个声音句柄都需要声音的 ID 来控制。此外，每个声音句柄还需要向相同的命令生成器推送命令，所以我们需要将其包装在 `Mutex` 中：

```
struct SoundHandle {
  sound_id: SoundId,
  command_producer: Arc<Mutex<Producer<Command>>>, } 
```

与上次尝试使用 `Mutex` 不同，这个 `Mutex` 只在游戏线程上共享，因此不会阻塞音频线程。

在音频线程上，我们会有类似以下代码：

```
while let Some(command) = command_consumer.pop() {
  match command {
    Command::PlaySound(audio_data) => play_sound(audio_data),
    Command::SetSoundPlaybackState(sound_id, playback_state) => {
      if let Some(sound) = get_sound_by_id(sound_id) {
        sound.playback_state = playback_state; } }
    Command::SetSoundVolume(sound_id, volume) => {
      if let Some(sound) = get_sound_by_id(sound_id) {
        sound.volume = volume; } } } } 
```

这个方法有效！而且相当高效，我们可以向音频线程发送任意复杂的命令而不会阻塞任何东西。

`SoundId` 是从哪里来的呢？

## 在音频线程上存储资源

我们需要以一种提供以下功能的方式在音频线程上存储资源（声音、混音轨道等）：

+   快速迭代

+   通过游戏线程也能快速随机访问资源的 ID

### 解决方案 1：Arenas

竞技场数据结构是一个自然的选择。竞技场本质上是一个可以被占用或为空的 `Vec` 槽的集合。当我们将一个项目插入到竞技场中时，竞技场会选择一个空槽来插入项目，并返回一个包含该槽索引的密钥。访问单个项目就像索引到一个 `Vec` 中一样快速。如果你遍历每个槽并过滤出空的槽，那么遍历项目就会很慢，但是你可以使用链表来使迭代更快。

这就是将资源发送到音频线程的流程：

1.  用户调用库中的一个函数将资源发送到音频线程

1.  库向音频线程发送一个带有要添加的资源的命令，并等待音频线程发送密钥回来

1.  下次音频线程开始处理时，它会接收到添加资源的命令，将其添加到竞技场中，并通过单独的环形缓冲区发送密钥回来

1.  下次游戏线程检查密钥时，它会接收到并将其返回给用户

因此，我们将不得不等待一段时间让音频线程返回密钥，但是不会太久，对吗？

…对吗？

#### 为什么要花这么长时间

当你听到扬声器传来的声音时，你听到的是随时间均匀分布的数字样本的模拟表示。但是应用程序不会以恒定的速率产生这些数字样本。如果是这样，那么如果任何样本花费太长时间来计算，应用程序就会落后，并且无法以足够快的速度产生音频，导致欠采样。相反，操作系统周期性地要求应用程序一次生成一批样本。

假设操作系统希望以 48,000 Hz（或每秒样本数）输出音频，并请求一次输出 512 个样本。音频线程将生成 512 个音频样本，然后休眠，直到操作系统唤醒它以获取下一批样本。由于操作系统可能不需要再次唤醒它以获取另外 10 毫秒的音频，因为它已经排队了足够的音频。

如果游戏线程在音频线程入睡后立即发送播放声音的命令，则音频线程将不会收到该命令，并在 10 毫秒后发送回声音 ID。以此为例，在一个 60 FPS 的游戏中，10 毫秒已经超过半帧。因此，如果我们连续播放两个声音，每次都阻塞游戏线程以等待音频线程发送回声音 ID，我们可能会出现帧率下降。对于音频库来说，这是不可接受的性能。

所以 arena 不再适用。

### 解决方案 2：`IndexMap`

如果我们将资源存储在哈希映射中，我们可以在游戏线程上创建键，并将它们与添加资源的命令一起发送到音频线程。标准库的`HashMap`遍历速度不够快，但[`indexmap`](https://crates.io/crates/indexmap) crate 解决了这个问题。

这是新的流程：

1.  用户调用库中的函数将资源发送到音频线程

1.  库内部递增一个 ID 以用作资源的键

1.  库向音频线程发送命令以添加带有 ID 的资源

1.  库立即将 ID 返回给调用者

问题解决了！我们不必等待音频线程发送回一个 ID。

不过，这种方法也有一些缺点：

+   从哈希映射中获取项比从 arena 中获取项要慢，因为它们必须对键进行哈希以获取内存中项的位置

+   `IndexMap` 随时间失去容量……等等，什么？

如果您和我一样，您会惊讶地了解后一种事实。但我会向您证明它！

```
use indexmap::IndexMap;

fn main()  {
  let mut map = IndexMap::with_capacity(10);
  for i in 0..1000 {
    map.insert(i, i);
    println!("{} / {}", map.len(), map.capacity());
    if i % 5 == 0 {
      map.swap_remove_index(i / 2);
      map.swap_remove_index(i / 3);
      map.swap_remove_index(i / 4); } } } 
```

这里有一个小例子，我向`IndexMap`添加项。每添加 5 个项，我会在任意索引处移除 3 个项。每次添加一个项时，我会打印地图的长度和总容量。您可以在[这里](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=a163cd12ab67bf774d0a9adef9187419)运行此代码片段。

我们最终得到类似于这样的结果：

```
28 / 28
26 / 26 // capacity decreases
27 / 27
28 / 28
29 / 56
30 / 56 
```

注意从容量为 28 下降到容量为 26？这不是 bug，[这只是 hashbrown 算法的工作方式](https://github.com/bluss/indexmap/issues/183)（`IndexMap` 和标准库的 `HashMap` 都使用了这个算法）。

原来有一个解决方法：如果不超过容量的 50%，则容量永远不会减少。所以我们可以分配比我们需要的两倍空间来避免这个问题。Kira v0.5 使用了这种方法，但我不太愿意依赖库的一个未明示的实现细节。

### 解决方案 3：重新审视 arena

或许竞技场可以起作用；它只需要让我们从游戏线程生成新的密钥。Rust 音频 Discord 的建议让我想到了原子如何用于实现这个目的。我最终想出了一个具有两个组件的竞技场：

+   竞技场本身，它保存资源数据并位于音频线程上。

+   一个竞技场控制器，它跟踪空闲和空槽，并且可以生成密钥。控制器可以克隆并发送到其他线程。

当我们想要向竞技场添加一个项目时，我们首先从竞技场控制器中保留一个密钥。如果保留了太多密钥，控制器会告诉我们竞技场已满并且不会给我们密钥。如果有可用的插槽，我们可以将密钥与添加项目的命令一起发送，并立即将密钥返回给调用者。

我发现这是一个非常优雅的解决方案，因为它一次解决了多个问题：

+   后备数据存储是一个`Vec`，因此容量不会有任何意外。

+   查找资源很快，因为密钥只是`Vec`中的索引。

+   我们可以直接从游戏线程生成新的密钥。

+   从游戏线程很容易获取分配资源和剩余容量的信息。

这是我在 Kira v0.6 中使用的解决方案。您可以在[这里](https://crates.io/crates/atomic-arena)查看我的竞技场的实现。

## 结论

制作音频库很困难。我不知道最佳方法是什么。这只是我尝试过的方式以及它对我产生的影响。
