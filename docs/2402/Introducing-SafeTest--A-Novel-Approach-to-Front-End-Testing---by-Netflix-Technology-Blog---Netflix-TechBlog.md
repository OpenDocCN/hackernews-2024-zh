<!--yml
category: 未分类
date: 2024-05-27 14:49:07
-->

# Introducing SafeTest: A Novel Approach to Front End Testing | by Netflix Technology Blog | Netflix TechBlog

> 来源：[https://netflixtechblog.com/introducing-safetest-a-novel-approach-to-front-end-testing-37f9f88c152d](https://netflixtechblog.com/introducing-safetest-a-novel-approach-to-front-end-testing-37f9f88c152d)

# Introducing SafeTest: A Novel Approach to Front End Testing

by [Moshe Kolodny](https://medium.com/u/a155da075195?source=post_page-----37f9f88c152d--------------------------------)

In this post, we’re excited to introduce SafeTest, a revolutionary library that offers a fresh perspective on End-To-End (E2E) tests for web-based User Interface (UI) applications.

# The Challenges of Traditional UI Testing

Traditionally, UI tests have been conducted through either unit testing or integration testing (also referred to as End-To-End (E2E) testing). However, each of these methods presents a unique trade-off: you have to choose between controlling the test fixture and setup, or controlling the test driver.

For instance, when using [react-testing-library](https://testing-library.com/docs/react-testing-library/intro/), a unit testing solution, you maintain complete control over what to render and how the underlying services and imports should behave. However, you lose the ability to interact with an actual page, which can lead to a myriad of pain points:

*   Difficulty in interacting with complex UI elements like <Dropdown /> components.
*   Inability to test CORS setup or GraphQL calls.
*   Lack of visibility into z-index issues affecting click-ability of buttons.
*   Complex and unintuitive authoring and debugging of tests.

Conversely, using integration testing tools like Cypress or Playwright provides control over the page, but sacrifices the ability to instrument the bootstrapping code for the app. These tools operate by remotely controlling a browser to visit a URL and interact with the page. This approach has its own set of challenges:

*   Difficulty in making calls to an alternative API endpoint without implementing custom network layer API rewrite rules.
*   Inability to make assertions on spies/mocks or execute code within the app.
*   Testing something like dark mode entails clicking the theme switcher or knowing the localStorage mechanism to override.
*   Inability to test segments of the app, for example if a component is only visible after clicking a button and waiting for a 60 second timer to countdown, the test will need to run those actions and will be at least a minute long.

Recognizing these challenges, solutions like E2E Component Testing have emerged, with offerings from [Cypress](https://docs.cypress.io/guides/component-testing/overview) and [Playwright](https://playwright.dev/docs/test-components). While these tools attempt to rectify the shortcomings of traditional integration testing methods, they have other limitations due to their architecture. They start a dev server with bootstrapping code to load the component and/or setup code you want, which limits their ability to handle complex enterprise applications that might have OAuth or a complex build pipeline. Moreover, updating TypeScript usage could break your tests until the Cypress/Playwright team updates their runner.

# Welcome to SafeTest

SafeTest aims to address these issues with a novel approach to UI testing. The main idea is to have a [snippet of code in our application bootstrapping stage that injects hooks to run our tests](https://www.npmjs.com/package/safetest#bootstrapping-your-application) (see the [How Safetest Works](https://www.npmjs.com/package/safetest#how-safetest-works) sections for more info on what this is doing). **Note that how this works has no measurable impact on the regular usage of your app since SafeTest leverages lazy loading to dynamically load the tests only when running the tests (in the README example, the tests aren’t in the production bundle at all).** Once that’s in place, we can use Playwright to run regular tests, thereby achieving the ideal browser control we want for our tests.

This approach also unlocks some exciting features:

*   Deep linking to a specific test without needing to run a node test server.
*   Two-way communication between the browser and test (node) context.
*   Access to all the DX features that come with Playwright (excluding the ones that come with @playwright/test).
*   Video recording of tests, trace viewing, and pause page functionality for trying out different page selectors/actions.
*   Ability to make assertions on spies in the browser in node, matching snapshot of the call within the browser.

# Test Examples with SafeTest

SafeTest is designed to feel familiar to anyone who has conducted UI tests before, as it leverages the best parts of existing solutions. Here’s an example of how to test an entire application:

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

We can just as easily test a specific component

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

# Leveraging Overrides

SafeTest utilizes React Context to allow for value overrides during tests. For an example of how this works, let’s assume we have a fetchPeople function used in a component:

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

We can modify the People component to use an Override:

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

Now, in our test, we can override the response for this call:

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

The render function also accepts a function that will be passed the initial app component, allowing for the injection of any desired elements anywhere in the app:

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

With overrides, we can write complex test cases such as ensuring a service method which combines API requests from `/foo`, `/bar`, and `/baz`, has the correct retry mechanism for just the failed API requests and still maps the return value correctly. So if `/bar` takes 3 attempts to resolve the method will make a total of 5 API calls.

Overrides aren’t limited to just API calls (since we can use also use `[page.route](https://playwright.dev/docs/api/class-page#page-route)`), we can also override specific app level values like feature flags or changing some static value:

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

Overrides are a powerful feature of SafeTest and the examples here only scratch the surface. For more information and examples, refer to the [Overrides section](https://www.npmjs.com/package/safetest#overrides) on the [README](https://github.com/kolodny/safetest/blob/main/README.md).

# Reporting

SafeTest comes out of the box with powerful reporting capabilities, such as automatic linking of video replays, Playwright trace viewer, and even [deep link directly to the mounted tested component](https://safetest-two.vercel.app/vite-react-ts/?test_path=.%2FAnother.safetest&test_name=Main2+can+do+many+interactions+fast). The SafeTest repo [README](https://github.com/kolodny/safetest/blob/main/README.md) links to all the [example apps](https://safetest-two.vercel.app/) as well as the [reports](https://safetest-two.vercel.app/report.html#results=vite-react-ts/artifacts/results.json&url=vite-react-ts/)

# SafeTest in Corporate Environments

Many large corporations need a form of authentication to use the app. Typically, navigating to localhost:3000 just results in a perpetually loading page. You need to go to a different port, like localhost:8000, which has a proxy server to check and/or inject auth credentials into underlying service calls. This limitation is one of the main reasons that Cypress/Playwright Component Tests aren’t suitable for use at Netflix.

However, there’s usually a service that can generate test users whose credentials we can use to log in and interact with the application. This facilitates creating a light wrapper around SafeTest to automatically generate and assume that test user. For instance, here’s basically how we do it at Netflix:

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

After setting this up, we simply import the above package in place of where we would have used safetest/setup.

# Beyond React

While this post focused on how SafeTest works with React, it’s not limited to just React. SafeTest also works with Vue, Svelte, Angular, and even can run on NextJS or Gatsby. It also runs using either Jest or Vitest based on which test runner your scaffolding started you off with. The [examples folder](https://github.com/kolodny/safetest/tree/main/examples) demonstrates how to use SafeTest with different tooling combinations, and we encourage contributions to add more cases.

At its core, SafeTest is an intelligent glue for a test runner, a UI library, and a browser runner. Though the most common usage at Netflix employs Jest/React/Playwright, it’s easy to add more adapters for other options.

# Conclusion

SafeTest is a powerful testing framework that’s being adopted within Netflix. It allows for easy authoring of tests and provides comprehensive reports when and how any failures occurred, complete with links to view a playback video or manually run the test steps to see what broke. We’re excited to see how it will revolutionize UI testing and look forward to your feedback and contributions.