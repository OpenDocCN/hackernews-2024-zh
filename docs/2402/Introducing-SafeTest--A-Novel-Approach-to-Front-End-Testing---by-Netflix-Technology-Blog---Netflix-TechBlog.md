<!--yml

category: 未分类

date: 2024-05-27 14:49:07

-->

# 引入 SafeTest：一种新颖的前端测试方法 | Netflix 技术博客 | Netflix TechBlog

> 来源：[https://netflixtechblog.com/introducing-safetest-a-novel-approach-to-front-end-testing-37f9f88c152d](https://netflixtechblog.com/introducing-safetest-a-novel-approach-to-front-end-testing-37f9f88c152d)

# 引入 SafeTest：一种新颖的前端测试方法

作者：[莫什·科洛德尼](https://medium.com/u/a155da075195?source=post_page-----37f9f88c152d--------------------------------)

在本文中，我们很高兴介绍 SafeTest，这是一个为基于 Web 的用户界面（UI）应用程序提供全新端到端（E2E）测试视角的革命性库。

# 传统 UI 测试的挑战

传统上，UI 测试通常通过单元测试或集成测试（也称为端到端（E2E）测试）来进行。然而，每种方法都有其独特的权衡：您必须在控制测试夹具和设置之间做出选择，或者在控制测试驱动程序之间做出选择。

例如，当使用 [react-testing-library](https://testing-library.com/docs/react-testing-library/intro/) 这样的单元测试解决方案时，您可以完全控制要渲染的内容以及底层服务和导入的行为。然而，您失去了与实际页面交互的能力，这可能导致许多痛点：

+   难以与复杂 UI 元素交互，例如 <Dropdown /> 组件。

+   无法测试 CORS 设置或 GraphQL 调用。

+   缺乏对影响按钮可点击性的 z-index 问题的可见性。

+   复杂和不直观的测试编写和调试。

相反，使用像 Cypress 或 Playwright 这样的集成测试工具可以控制页面，但会牺牲对应用程序引导代码进行仪表化的能力。这些工具通过远程控制浏览器访问 URL 并与页面交互来操作。这种方法有其自身的一系列挑战：

+   在不实现自定义网络层 API 重写规则的情况下难以调用替代 API 端点。

+   无法对间谍/模拟对象进行断言或在应用程序内执行代码。

+   测试诸如暗模式之类的内容需要点击主题切换器或了解覆盖 localStorage 机制。

+   无法测试应用程序的部分内容，例如如果一个组件只在点击按钮并等待 60 秒倒计时后才可见，则测试将需要执行这些操作，至少需要一分钟的时间。

识别这些挑战后，像E2E组件测试这样的解决方案应运而生，Cypress和[Playwright](https://playwright.dev/docs/test-components)提供了这些解决方案。虽然这些工具试图纠正传统集成测试方法的缺点，但由于它们的架构，它们也存在其他限制。它们启动一个带有引导代码的开发服务器来加载组件和/或设置您希望的代码，这限制了它们处理可能具有OAuth或复杂构建流水线的复杂企业应用程序的能力。此外，更新TypeScript使用可能会导致您的测试失败，直到Cypress/Playwright团队更新其运行器。

# 欢迎使用SafeTest

SafeTest旨在通过一种新颖的UI测试方法来解决这些问题。其主要思想是在我们应用程序的引导阶段[插入代码片段以注入运行我们的测试的钩子](https://www.npmjs.com/package/safetest#bootstrapping-your-application)（有关此操作的更多信息，请参见[SafeTest工作原理](https://www.npmjs.com/package/safetest#how-safetest-works)部分）。**请注意，这种方式对您应用程序的常规使用没有任何可测量的影响，因为SafeTest利用延迟加载仅在运行测试时动态加载测试（在README示例中，测试根本不包含在生产捆绑包中）。**设置完成后，我们可以使用Playwright来运行常规测试，从而实现我们想要的理想浏览器控制。

此方法还解锁了一些令人兴奋的功能：

+   深层链接到特定测试，而无需运行节点测试服务器。

+   浏览器和测试（节点）上下文之间的双向通信。

+   访问所有随Playwright一起提供的DX功能（不包括@playwright/test提供的功能）。

+   测试录像，跟踪查看和暂停页面功能以尝试不同的页面选择器/操作。

+   能够对浏览器中的间谍进行断言，在节点中匹配调用的快照。

# 使用SafeTest的测试示例

SafeTest旨在让以前进行过UI测试的任何人都感到熟悉，因为它利用现有解决方案的最佳部分。以下是如何测试整个应用程序的示例：

```
import { describe, it, expect } from 'safetest/jest';
import { render } from 'safetest/react';

describe('my app', () => {
  it('loads the main page', async () => {
    const { page } = await render();

    await expect(page.getByText('Welcome to the app')).toBeVisible();
    expect(await page.screenshot()).toMatchImageSnapshot();
  });
});
```

我们可以轻松测试特定组件

```
import { describe, it, expect, browserMock } from 'safetest/jest';
import { render } from 'safetest/react';

describe('Header component', () => {
  it('has a normal mode', async () => {
    const { page } = await render(<Header />);

    await expect(page.getByText('Admin')).not.toBeVisible();
   });

  it('has an admin mode', async () => {
    const { page } = await render(<Header admin={true} />);

    await expect(page.getByText('Admin')).toBeVisible();
  });

  it('calls the logout handler when signing out', async () => {
    const spy = browserMock.fn();
    const { page } = await render(<Header handleLogout={spy} />);

    await page.getByText('logout').click();
    expect(await spy).toHaveBeenCalledWith();
  });
});
```

# 利用覆盖功能

SafeTest利用React Context允许在测试期间进行值覆盖。例如，假设我们在组件中使用fetchPeople函数的示例：

```
import { useAsync } from 'react-use';
import { fetchPerson } from './api/person';

export const People: React.FC = () => {
  const { data: people, loading, error } = useAsync(fetchPeople);

  if (loading) return <Loader />;
  if (error) return <ErrorPage error={error} />;
  return <Table data={data} rows=[...] />;
}
```

我们可以修改People组件以使用覆盖：

```
 import { fetchPerson } from './api/person';
+import { createOverride } from 'safetest/react';

+const FetchPerson = createOverride(fetchPerson);

 export const People: React.FC = () => {
+  const fetchPeople = FetchPerson.useValue();
   const { data: people, loading, error } = useAsync(fetchPeople);

   if (loading) return <Loader />;
   if (error) return <ErrorPage error={error} />;
   return <Table data={data} rows=[...] />;
 }
```

现在，在我们的测试中，我们可以覆盖此调用的响应：

```
const pending = new Promise(r => { /* Do nothing */ });
const resolved = [{name: 'Foo', age: 23], {name: 'Bar', age: 32]}];
const error = new Error('Whoops');

describe('People', () => {
  it('has a loading state', async () => {
    const { page } = await render(
      <FetchPerson.Override with={() => () => pending}>
        <People />
      </FetchPerson.Override>
    );

    await expect(page.getByText('Loading')).toBeVisible();
  });

  it('has a loaded state', async () => {
    const { page } = await render(
      <FetchPerson.Override with={() => async () => resolved}>
        <People />
      </FetchPerson.Override>
    );

    await expect(page.getByText('User: Foo, name: 23')).toBeVisible();
  });

  it('has an error state', async () => {
    const { page } = await render(
      <FetchPerson.Override with={() => async () => { throw error }}>
        <People />
      </FetchPerson.Override>
    );

    await expect(page.getByText('Error getting users: "Whoops"')).toBeVisible();
  });
});
```

渲染函数还接受一个函数，该函数将传递初始应用程序组件，允许在应用程序的任何位置注入任何所需的元素：

```
it('has a people loaded state', async () => {
  const { page } = await render(app =>
    <FetchPerson.Override with={() => async () => resolved}>
      {app}
    </FetchPerson.Override>
  );
   await expect(page.getByText('User: Foo, name: 23')).toBeVisible();
});
```

通过覆盖重写，我们可以编写复杂的测试案例，例如确保一个服务方法，该方法结合来自 `/foo`、`/bar` 和 `/baz` 的 API 请求，并对仅失败的 API 请求进行正确的重试机制，并且仍然正确映射返回值。因此，如果 `/bar` 需要尝试 3 次才能解析，则该方法将总共进行 5 次 API 调用。

覆盖重写不仅限于 API 调用（因为我们也可以使用 `[page.route](https://playwright.dev/docs/api/class-page#page-route)`），我们还可以覆盖特定的应用级值，例如功能标志或更改某些静态值：

```
+const UseFlags = createOverride(useFlags);
 export const Admin = () => {
+  const useFlags = UseFlags.useValue();
   const { isAdmin } = useFlags();
   if (!isAdmin) return <div>Permission error</div>;
   // ...
 }

+const Language = createOverride(navigator.language);
 export const LanguageChanger = () => {
-  const language = navigator.language;
+  const language = Language.useValue();
   return <div>Current language is { language } </div>;
 }

 describe('Admin', () => {
   it('works with admin flag', async () => {
     const { page } = await render(
       <UseIsAdmin.Override with={oldHook => {
         const oldFlags = oldHook();
         return { ...oldFlags, isAdmin: true };
       }}>
         <MyComponent />
       </UseIsAdmin.Override>
     );

     await expect(page.getByText('Permission error')).not.toBeVisible();
   });
 });

 describe('Language', () => {
   it('displays', async () => {
     const { page } = await render(
       <Language.Override with={old => 'abc'}>
         <MyComponent />
       </Language.Override>
     );

     await expect(page.getByText('Current language is abc')).toBeVisible();
   });
 });
```

覆盖重写是 SafeTest 的一个强大功能，这里的示例只是冰山一角。更多信息和示例，请参阅 [README](https://github.com/kolodny/safetest/blob/main/README.md) 上的[覆盖重写部分](https://www.npmjs.com/package/safetest#overrides)。

# 报告

SafeTest 自带强大的报告功能，例如自动链接视频回放，Playwright 追踪查看器，甚至可以[直接深入到已挂载的测试组件](https://safetest-two.vercel.app/vite-react-ts/?test_path=.%2FAnother.safetest&test_name=Main2+can+do+many+interactions+fast)。SafeTest 仓库的 [README](https://github.com/kolodny/safetest/blob/main/README.md) 链接到所有[示例应用](https://safetest-two.vercel.app/)以及[报告](https://safetest-two.vercel.app/report.html#results=vite-react-ts/artifacts/results.json&url=vite-react-ts/)。

# SafeTest 在企业环境中

许多大型公司需要一种认证形式来使用应用程序。通常，导航到 localhost:3000 只会导致页面永久加载。您需要转到另一个端口，如 localhost:8000，它有一个代理服务器来检查和/或将认证凭据注入到底层服务调用中。这种限制是 Cypress/Playwright 组件测试在 Netflix 不适合使用的主要原因之一。

然而，通常有一种服务可以生成测试用户的凭证，我们可以用来登录并与应用程序进行交互。这有助于创建一个轻量级的包装器，自动生成并假定该测试用户。例如，在 Netflix 的基本做法如下：

```
import { setup } from 'safetest/setup';
import { createTestUser, addCookies } from 'netflix-test-helper';

type Setup = Parameters<typeof setup>[0] & {
  extraUserOptions?: UserOptions;
};

export const setupNetflix = (options: Setup) => {
  setup({
    ...options,
    hooks: { beforeNavigate: [async page => addCookies(page)] },
  });

  beforeAll(async () => {
    createTestUser(options.extraUserOptions)
  });
};
```

设置完成后，我们只需在原本使用 safetest/setup 的地方导入上述包即可。

# 超越 React

虽然这篇文章侧重于 SafeTest 如何与 React 协作，但它并不仅限于 React。SafeTest 也适用于 Vue、Svelte、Angular，甚至可以在 NextJS 或 Gatsby 上运行。它还可以使用 Jest 或 Vitest，具体取决于你的脚手架使用的测试运行器。[示例文件夹](https://github.com/kolodny/safetest/tree/main/examples)展示了如何在不同的工具组合中使用 SafeTest，我们鼓励添加更多案例。

在其核心，SafeTest 是一个智能的测试运行器粘合剂，一个 UI 库和一个浏览器运行器。尽管 Netflix 最常见的使用方式是 Jest/React/Playwright，但很容易为其他选项添加更多的适配器。

# 结论

SafeTest 是一个强大的测试框架，在 Netflix 内部得到了广泛应用。它支持轻松编写测试，并提供全面的报告，详细说明任何失败发生的时间和方式，包括链接以查看回放视频或手动运行测试步骤以了解出了什么问题。我们对它将如何革新 UI 测试感到兴奋，并期待您的反馈和贡献。
