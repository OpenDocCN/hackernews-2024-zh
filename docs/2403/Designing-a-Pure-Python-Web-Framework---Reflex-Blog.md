<!--yml

category: 未分类

date: 2024-05-29 12:34:30

-->

# 设计纯 Python Web 框架 · Reflex 博客

> 来源：[https://reflex.dev/blog/2024-03-21-reflex-architecture/](https://reflex.dev/blog/2024-03-21-reflex-architecture/)

我们一年前开始开发 Reflex，以便任何懂 Python 的人都可以轻松构建 Web 应用并与世界分享，而无需学习新语言并拼凑一堆不同的工具。

在这篇文章中，我将分享更多关于 Reflex 的特点，以及其内部工作原理。

我们将使用以下基本应用程序作为示例，显示 Github 个人资料图像，以解释体系结构的不同部分。

```
import requests
import reflex as rx

class GithubState(rx.State):
    url: str = "https://github.com/reflex-dev"
    profile_image: str = (
        "https://avatars.githubusercontent.com/u/104714959"
    )

    def set_profile(self, username: str):
        if username == "":
            return
        github_data = requests.get(
            f"https://api.github.com/users/{username}"
        ).json()
        self.url = github_data["url"]
        self.profile_image = github_data["avatar_url"]

def index():
    return rx.hstack(
        rx.link(
            rx.avatar(src=GithubState.profile_image),
            href=GithubState.url,
        ),
        rx.input(
            placeholder="Your Github username",
            on_blur=GithubState.set_profile,
        ),
    )
```

Web 开发是编程中最受欢迎的应用之一。Python 是全球最流行的编程语言之一。那么为什么我们不能用 Python 构建 Web 应用呢？

在开始 Reflex 之前，我曾在一家初创公司和一家大型科技公司的 AI 项目上工作。在这些团队中，我们从数据分析到机器学习再到后端服务都使用 Python。但是，当涉及到构建用户界面或应用以便其他人使用我们的工作时，没有一个好的选择能保持在 Python 中。突然间，我们不得不切换到 JavaScript 并学习一个全新的生态系统。

制作 UI 应该很简单，但即使我们的团队有很棒的工程师，学习新语言和工具的开销仍是一个巨大的障碍。通常，制作 UI 比我们实际工作更难！

以前已经有几种方法可以在 Python 中构建应用程序，但没有一种符合我们的需求。

另一方面，像[Django](https://www.djangoproject.com/)和[Flask](https://flask.palletsprojects.com/)这样的框架非常适合构建生产级 Web 应用程序。但它们仅处理后端 - 您仍然需要使用 JavaScript 和前端框架，并编写大量样板代码来连接前端和后端。

另一方面，像[Dash](https://dash.plotly.com/)和[Streamlit](https://streamlit.io/)这样的纯 Python 库可能非常适合小型项目，但它们仅限于特定用例，并且缺乏构建完整 Web 应用所需的功能和性能。随着您的应用程序功能和复杂性的增长，您可能会发现自己受到框架的限制，到达此时，您要么限制自己的想法以适应框架，要么放弃您的项目，并使用“真正的 Web 框架”重建它。

我们希望通过创建一个易于上手且直观的框架来填补这一差距，同时保持灵活性和强大性，以支持任何应用程序。

+   **纯 Python**：一种语言处理一切。

+   **轻松上手**：轻松构建您的想法，无需网页开发经验。

+   **完全灵活性**：Web 应用程序应该匹配传统 Web 框架的可定制性和性能。

+   **电池包含**：从前端到后端再到部署，一站式解决。

现在让我们深入了解我们如何构建 Reflex 以实现这些目标。

全栈Web应用程序由前端和后端组成。前端是用户界面，作为运行在用户浏览器上的Web页面提供服务。后端处理逻辑和状态管理（如数据库和API），并在服务器上运行。

在传统的Web开发中，这些通常是两个单独的应用程序，并且通常使用不同的框架或语言编写。例如，您可以将Flask后端与React前端结合使用。使用这种方法，您必须维护两个独立的应用程序，并且最终编写大量的样板代码来连接前端和后端。

我们希望在 Reflex 中简化这个过程，通过在单个代码库中定义前端和后端，同时全部使用 Python。开发者只需关注其应用的逻辑，而不必担心底层实现细节。

在幕后，Reflex 应用程序编译成一个 [React](https://react.dev) 前端应用程序和一个 [FastAPI](https://github.com/tiangolo/fastapi) 后端应用程序。只有UI编译为Javascript；所有应用程序逻辑和状态管理都保留在Python中，并在服务器上运行。Reflex 使用 [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) 从前端向后端发送事件，并从后端向前端发送状态更新。

下图详细说明了 Reflex 应用程序的工作原理。我们将在接下来的章节中更详细地讨论每个部分。

我们希望 Reflex 应用程序在最终用户看来看起来和感觉像传统的Web应用程序，同时对于开发者来说易于构建和维护。为此，我们构建在成熟和流行的Web技术之上。

当您运行 `reflex run` 您的应用程序时，Reflex 将前端编译为一个单页 [Next.js](https://nextjs.org) 应用程序，并在端口（默认为 `3000`）上提供服务，您可以在浏览器中访问它。

前端的任务是反映应用程序的状态，并在用户与UI交互时向后端发送事件。前端不会运行实际的逻辑。

Reflex 前端使用可以组合在一起创建复杂UI的组件构建。我们不使用将HTML和Python混合的模板语言，而只是使用Python函数定义UI。

```

```

def index():

    return rx.hstack(

        rx.link(

            rx.avatar(src=GithubState.profile_image),

            href=GithubState.url,

        ),

        rx.input(

            placeholder="你的Github用户名",

            on_blur=GithubState.set_profile,

        ),

    )

```

```

在我们的示例应用程序中，我们有诸如`rx.hstack`、`rx.avatar`和`rx.input`等组件。这些组件可以具有不同的**props**，影响它们的外观和功能 - 例如，`rx.input`组件具有一个`placeholder` prop来显示默认文本。

我们可以通过事件（如`on_blur`）使我们的组件响应用户交互，我们将在下面更详细地讨论这个问题。

在幕后，这些组件编译成React组件。例如，上述代码编译成以下React代码：

```

```

<HStack>

    <Link href={GithubState.url}>

        <Avatar src={GithubState.profile_image}/>

    </Link>

    <Input

        placeholder="您的Github用户名"

        // 实际上这将是一个向后端发起的WebSocket调用。

        onBlur={GithubState.set_profile}

</HStack>

```

```

我们许多核心组件基于[Radix](https://radix-ui.com/)，这是一个流行的React组件库。我们还有许多其他组件用于图表、数据表格等。

我们选择React，因为它是一个拥有庞大生态系统的流行库。我们的目标不是重新创建Web生态系统，而是使其对Python开发者更加友好。

这也使我们的用户可以在我们没有需要的组件时使用自己的组件。用户可以[包装他们自己的React组件](/docs/wrapping-react/overview)，然后[发布它们](/docs/custom-components/overview)供他人使用。随着时间的推移，我们将建立起我们的[第三方组件生态系统](/docs/custom-components)，使用户可以轻松地找到和使用其他人构建的组件。

我们希望确保Reflex应用程序在开箱即用时看起来良好，同时仍然给开发人员完全控制其应用程序外观的能力。

我们拥有一个核心[主题系统](/docs/styling/theming)，可以让您设置高级别的样式选项，如深色模式和强调色，以在整个应用程序中提供统一的外观和感觉。

此外，Reflex组件可以利用CSS的全部功能进行样式化。我们利用[Emotion](https://emotion.sh/docs/introduction)库允许"CSS-in-Python"样式化，因此您可以将任何CSS属性作为组件的关键字参数传递。这包括通过传递值列表来使用[响应式属性](/docs/styling/responsive)。

现在让我们看看如何为我们的应用程序添加互动性。

在Reflex中，只有前端编译成JavaScript并在用户浏览器上运行，而所有的状态和逻辑都保留在Python中，并在服务器上运行。当您`reflex run`时，我们会启动一个FastAPI服务器（默认端口为`8000`），前端通过WebSocket连接到该服务器。

所有的状态和逻辑都定义在一个`State`类中。

```

```

class GithubState(rx.State):

    url: str = "https://github.com/reflex-dev"

    profile_image: str = (

        "https://avatars.githubusercontent.com/u/104714959"

    )

    def set_profile(self, username: str):

        如果用户名为空

            返回

        github_data = requests.get(

            f"https://api.github.com/users/{username}"

        ).json()

        self.url = github_data["url"]

        self.profile_image = github_data["avatar_url"]

```

```

状态由**变量（vars）**和**事件处理程序（event handlers）**组成。

变量是应用程序中随时间可以更改的任何值。它们被定义为`State`类的类属性，并且可以是可以序列化为JSON的任何Python类型。在我们的示例中，`url`和`profile_image`是变量。

事件处理程序是您的`State`类中的方法，当用户与UI交互时会调用它们。它们是我们可以修改Reflex中变量的唯一方式，并且可以响应用户操作调用，例如点击按钮或在文本框中输入。在我们的例子中，`set_profile`是一个事件处理程序，用于更新`url`和`profile_image`变量。

由于事件处理程序在后端运行，您可以在其中使用任何Python库。在我们的例子中，我们使用`requests`库向Github发起API调用以获取用户的个人资料图片。

现在让我们进入有趣的部分 - 如何处理事件和状态更新。

通常在编写Web应用程序时，您需要编写大量样板代码来连接前端和后端。使用Reflex，您不必担心这些问题 - 我们为您处理前端和后端之间的通信。开发人员只需编写他们的事件处理逻辑，并在变量更新时，UI会自动更新。

您可以参考上面的图表，了解这个过程的视觉表示。让我们用我们的Github个人资料图片示例来详细了解一下。

用户可以以多种方式与UI交互，例如点击按钮、在文本框中输入或悬停在元素上。在Reflex中，我们称这些为**事件触发器**。

```

```

rx.input(

    placeholder="您的Github用户名",

    on_blur=GithubState.set_profile，

)

```

```

在我们的例子中，我们将`on_blur`事件触发器绑定到`set_profile`事件处理程序。这意味着当用户在输入字段中键入然后单击离开时，将调用`set_profile`事件处理程序。

在前端，我们维护一个包含所有待处理事件的事件队列。一个事件包括三个主要数据部分：

+   **客户端令牌**：每个客户端（浏览器标签）都有一个唯一的令牌用于识别它。这让后端知道要更新哪个状态。

+   **事件处理程序**：在状态上运行的事件处理程序。

+   **参数**：传递给事件处理程序的参数。

假设我在输入框中输入我的用户名“picklelo”。在这个示例中，我们的事件看起来会是这样的：

```

```

{

    客户端令牌："abc123"

    事件处理程序："GithubState.set_profile"

    参数：["picklelo"]

}

```

```

在前端，我们维护一个包含所有待处理事件的事件队列。

当事件被触发时，它会被添加到队列中。我们有一个`processing`标志，以确保一次只处理一个事件。这确保了状态始终一致，并且没有两个事件处理程序在同一时间修改状态的竞争条件。

一旦事件准备好处理，它会通过WebSocket连接发送到后端。

一旦接收到事件，它将在后端上进行处理。

Reflex使用一个**状态管理器**，它维护客户端令牌和它们的状态之间的映射关系。默认情况下，状态管理器只是一个内存中的字典，但可以扩展为使用数据库或缓存。在生产环境中，我们使用Redis作为我们的状态管理器。

一旦我们获取到用户的状态，下一步就是使用参数运行事件处理程序。

```

```

def set_profile(self, username: str):

    if username == "":

        return

    github_data = requests.get(

        f"https://api.github.com/users/{username}"

    ).json()

    self.url = github_data["url"]

    self.profile_image = github_data["avatar_url"]

```

```

在我们的示例中，`set_profile` 事件处理程序在用户状态上运行。这会向 Github 发送 API 调用以获取用户的个人资料图片，然后更新状态的 `url` 和 `profile_image` 变量。

每当事件处理程序返回（或 [yield](/docs/events/yield-events)），我们就会将状态保存在状态管理器中，并发送 **状态更新** 到前端来更新 UI。

为了在状态增长时保持性能，内部 Reflex 会跟踪在事件处理程序中更新的变量（**dirty vars**）。当事件处理程序处理完毕时，我们会找到所有的 dirty vars 并创建一个状态更新，以发送到前端。

在我们的情况下，状态更新可能看起来像这样：

```

```

{

    "url": "https://github.com/picklelo",

    "profile_image": "https://avatars.githubusercontent.com/u/104714959"

}

```

```

我们将新状态存储在我们的状态管理器中，然后将状态更新发送到前端。然后前端更新 UI 来反映新的状态。在我们的示例中，会显示新的 Github 用户头像。

希望这为您提供了 Reflex 在幕后运作的良好概述。我们将有更多的帖子发布，分享如何通过诸如状态分片和编译器优化等功能使 Reflex 可扩展和高性能化。

您可以随时关注我们获取更多信息：
