<!--yml
category: 未分类
date: 2024-05-29 12:35:38
-->

# CSS | 18F Engineering

> 来源：[https://guides.18f.gov/engineering/languages-runtimes/css/](https://guides.18f.gov/engineering/languages-runtimes/css/)

<main class="usa-layout-docs__main desktop:grid-col-9
                  usa-prose measure-5 engineering">

# CSS

The purpose of the CSS coding styleguide is to create consistent CSS or preprocessor CSS code (such as [Sass](http://sass-lang.com)) in your projects. The styleguide should be treated as a guide — rules can be modified according to project needs.

## Architecture[](#architecture)

A site's architecture should be based on its goals and purposes. This means the guidance here should be adapted to different sites and situations.

### Modular or component architecture[](#modular-or-component-architecture)

When using a modular or component architecture, every page is broken into a series of modular components. There are two sets of these components: `components` and `modules`. The architecture starts out with basic HTML element rules: HTML, p, a, form, etc tags that than have components and modules written on top of them. Components are very basic structure such as buttons, blurbs, navs, and positioning structures like insets, island, and enclosure. From here, modules are built with these components. This architecture also attempts to keep the specificity trend in an upwards curve as you move down in the file (more on this to come).

*   Start with an elements file for all tag rules (a, h1-h5, p, *, html, body).
*   Create component files for each structural element, such as buttons, navs, etc. These are mainly class-based and use [BEM](http://getbem.com/introduction/) or another naming scheme.
*   Create more specific structure with modules. For instance, if the logo image and text needs very specific treatment, use a module.
    *   Build modules from components through mixins, extends, and HTML.
    *   Modules can have higher specificity, it’s fine to use deeper nesting.
*   Have an overrides file or folder comprised of global rules that are meant to override components and modules.
    *   These can be generic utilities.
    *   A good thing to put here are breakpoint-specific rules, such as hiding something at small breakpoints.

#### File structure[](#file-structure)

```
_elements.scss
_mixins.scss
_typography.scss
_util.scss
_vars.scss
component/_blurb.scss
component/_button.scss
component/_island.scss
component/_sub_nav.scss
module/_logo.scss
module/_progress_bar.scss
lib/bourbon.scss
lib/neat.scss
_overrides.scss
```

For the `util`, `typography`, `elements`, and `overrides` files, once they grow too large (300 lines or more) in size, split them into their own folder with sub files.

```
elements/_all.scss
elements/_p.scss
elements/_h.scss
typography/_body.scss
typography/_links.scss
overrides/_breakpoints.scss
overrides/_util.scss
util/_center.scss
util/_clearfix.scss
```

### Importing[](#importing)

As you likely know, CSS rules that are later in the file override earlier rules. This means Sass imports can be used to control inheritance and specificity.

*   Start with base elements
*   Move to single nested classes and utils
*   Move next to more specific classes, often with nesting
*   Move next to overrides, possibly with !important rules
*   Import alphabetically
*   Only modify import order for groups of files, not specific files

```
 @import 'module/logo';
@import 'component/mask';
@import 'component/button'; 

@import 'component/button';
@import 'component/mask';
@import 'module/logo';
```

## Documentation[](#documentation)

Be intentional when you use `//` (silent comments) versus `/* */` (which are preserved in the CSS output). When in doubt, use `//`.

### KSS[](#kss)

Use KSS for documentation. More information on KSS can be found on the [official site](http://warpspire.com/kss/).

#### Example[](#example)

```
 .button {
}

.button-modified {
}
```

#### Rationale[](#rationale)

KSS is the most common CSS documentation method to date. While it’s not perfect, the generated documentation can be modified through templates.

## Frameworks[](#frameworks)

TTS recommends using the [U.S. Web Design System (USWDS)](https://github.com/uswds/uswds) as it is specifically designed to help build fast, accessible, mobile-friendly federal government websites.

Sometimes, projects utilize other CSS frameworks such as:

1.  [Bourbon](https://www.bourbon.io/)
2.  [BassCSS](https://basscss.com/)

These frameworks were chosen because they're relatively unopinionated about design decisions while still providing the helpers that make frameworks essential to fast and accurate frontend work, for example, solutions for responsive design, grids, and common design patterns. In addition, both frameworks, through modular design and excellent documentation, make it easy for the designer or developer to only use the parts that they need, rather than including a hefty library.

### Do not use Bootstrap[](#do-not-use-bootstrap)

18F specifically does not recommend using [Bootstrap](http://getbootstrap.com/) for production work because it can be difficult to adapt its opinionated styles to bespoke design work.

## Formatting[](#formatting)

We recommend using [Prettier](https://prettier.io), and enabling it in your editor by default. Prettier is an automatic code formatter that will make your code format consistent. This way we don't have to argue over how to format our code - we just let the tools enforce something! Prettier works with both plain CSS and SCSS.

Do not use shorthand declarations unless you need to explicitly set all the available values.

```
 margin: inherit 3em;

margin-bottom: 3em;
margin-top: 3em;

margin: 3em 4em 2em 1em;
```

Avoid arbitrary numbers that are repeated, or linked, or dependent on other parts of the code, (aka “magic numbers”).

```
 .component {
  top: 0.327em;
}

.component {
  top: 0.327em;
}

$align_top: 100%;
.component {
  top: $align_top;
}
```

### Order[](#order)

*   Use the following ordering:

    1.  variables
    2.  @extend directives
    3.  @include directives
    4.  declaration list (`property: name;`)
    5.  media queries
    6.  pseudo-states (`:checked`, `:target`, etc.) and pseudo-elements (`::after`, `::selection`, etc.)
    7.  nested elements
    8.  nested classes
*   Use alphabetical order or type order for declarations. Pick one to keep the whole project consistent.

*   Place a new line before nested selectors unless they are after the first selector.

*   Treat nested includes, such as Neat's media includes – `@include media($small-screen)` — as a standard media query, rather than a Sass @include. So they would be sorted directly after the declaration list.

*   Place mixin calls with `@content` after nested selectors.

*   You may deviate the sorting order to better suit your project's needs, as long as it's consistent throughout the project.

```
 .module {

  .module-element {
    color: #fff932;
  }
}

.module {
  .module-element {
    color: #fff932;
  }
}

.module {
  $amount = 3;
  @extend .component;
  @include sizing($amount);
  margin-top: $amount * 1em;
  text-align: center;

  @include media($small-screen) {
    margin-top: ($amount + 10em);
  }

  &::before {
    content: "hello";
  }

  .module__ele {
    color: #fff932;
  }
}
```

## Inheritance[](#inheritance)

### Mixins[](#mixins)

*   Use mixins for groups of properties that appear together intentionally and are used multiple times.

    ```
    @mixin clearfix {
      &:after {
        content: '';
        display: table;
        clear: both;
      }
    }
    ```

*   Use mixins for components to change size.

*   Use mixins when something requires parameters.

    ```
    @mixin size($width, $height: $width) {
      width: $width;
      height: $height;
    }
    ```

*   Do not use mixins for browser prefixes. Use [Autoprefixer](https://github.com/postcss/autoprefixer).

    ```
     @mixin transform($value) {
      -webkit-transform: $value;
      -moz-transform: $value;
      transform: $value;
    }
    ```

## Extend[](#extend)

Be very careful with using `@extend`. It's a powerful tool that can have disastrous side-effects. Before using please consider:

*   Where is my current selector going to be appended?
*   Am I likely to be causing undesired side-effects?
*   How large is the CSS generated by this single extend?

If you're unsure of using `@extend`, follow these rules to avoid running into trouble:

*   Use `@extend` from within a module, not across different modules.
*   Use `@extend` on [placeholders](http://thesassway.com/intermediate/understanding-placeholder-selectors) exclusively, not on actual selectors.
*   Make sure the placeholder you extend is present as little as possible in the stylesheet.

You can use [mixins](http://sass-lang.com/guide#mixins) in place of selectors. While mixins will copy more code, the difference will often be negligible once the output file has been gzipped.

## Linting[](#linting)

Using a linter will ensure that CSS code conforms to some consistent rules. A linting tool like [Stylelint](https://stylelint.io/) can issue warnings wherever your code differs from established rules.

### Setting up Stylelint locally[](#setting-up-stylelint-locally)

1.  Run `npm install --save-dev stylelint stylelint-config-standard` to download the package and save it to your package.json
2.  Create a .stylelintrc.json configuration file in the root of your project with the following content:

```
{
  "extends": "stylelint-config-standard"
}
```

3.  Run Stylelint on all the CSS files in your project: `npx stylelint "**/*.css"`

This utilizes stylelint's standard rules configuration for css linting. If your project needs different rules or defaults, or you'd like to extend linting to other types of files, you can utilize custom syntaxes or write your own.

Stylelint has a [user guide](https://stylelint.io/user-guide/get-started) that will give you more details about how to configure and extend your linting rules.

## Naming[](#naming)

*   HTML elements should be in lowercase.

    ```
    body,
    div {
    }
    ```

*   Classes should be lowercase.

*   Avoid camelcase.

*   Name things clearly.

*   Write classes semantically. Name its function not its appearance.

    ```
     .ClassNAME { }

    .commentForm { }

    .c1-xr { }
    ```

*   Avoid presentation- or location-specific words in names, as this will cause problems when you (invariably) need to change the color, width, or feature later.

    ```
     .blue
    .text-gray
    .100width-box

    .warning
    .primary
    .lg-box
    ```

*   Be wary of naming components based on content, as this limits the use of the class.

    ```
     .product_list

    .item_list
    ```

*   Don't abbreviate unless it’s a well-known abbreviation.

    ```
     .bm-rd

    .block--lg
    ```

*   Use quotes in type pseudo selectors.

    ```
     .top_image[type="text"] {
    }
    ```

*   Name CSS components and modules with singular nouns.

    ```
    .button {
    }
    ```

*   Name modifiers and state-based rules with adjectives.

    ```
    .is_hovered {
    }
    ```

*   If your CSS has to interface with other CSS libraries, consider namespacing every class.

    ```
    .f18-component
    ```

### Naming methodologies[](#naming-methodologies)

When it comes to naming, the most important thing is consistency. The recommended way to do this is using an existing methodology like [BEM](#bem), or use a custom one that’s clearly defined.

#### BEM[](#bem)

[BEM](http://getbem.com/introduction/) (**B**lock, **E**lement, **M**odifier) structures CSS such that every entity is composed of (you guessed it) blocks, elements and modifiers. From [Harry Roberts](http://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/):

> The point of BEM is to tell other developers more about what a piece of markup is doing from its name alone. By reading some HTML with some classes in, you can see how – if at all – the chunks are related; something might just be a component, something might be a child, or element, of that component, and something might be a variation or modifier of that component.

TTS generally recommends using a modified BEM methodology outlined in the next subsection. However, you might want to use standard BEM when:

*   You need a naming scheme that general CSS developers will already be familiar with or an existing naming scheme hasn’t been consistent enough.
*   When you want to use JavaScript to modify the BEM class names dynamically.

Here is an example of BEM in SCSS:

```
 .inset {
  margin-left: 15%;

  .inset__content {
    padding: 3em;
  }
}

.inset--sm {
  margin-left: 10%;

  .inset__content {
    padding: 1em;
  }
}

.inset--lg {
  margin-left: 20%;
}
```

#### Suggested custom methodology[](#suggested-custom-methodology)

The TTS recommendation for a naming methodology is a modified version of BEM. It still uses blocks, sections within blocks and modifiers, but with an abbreviated syntax.

```
.accordion
.accordion-item
.accordion-item-selected

.nav_bar
.nav_bar-link
.nav_bar-link-clicked 
```

#### Naming methodology resources[](#naming-methodology-resources)

### js- flagged classes[](#js-flagged-classes)

Don't attach styles to classes with a `js-` flag. These classes are reserved for javascript.

```
// Bad
.js-people {
  color: #ff0;
}
```

#### Rationale[](#rationale-1)

A `js-` flagged class needs to be highly portable. Adding styles to it breaks that portability.

### test- flagged classes[](#test-flagged-classes)

Don't attach styles to classes with a `test-` flag. These classes are reserved for testing hooks such as those used by selenium.

```
// Bad
.test-people {
  color: #ff0;
}
```

## Preprocessors[](#preprocessors)

The most supported CSS preprocessor at TTS is [Sass](http://sass-lang.com). Using this preprocessor means you'll get supported resources such as frameworks, libraries, tutorials, and a comprehensive styleguide as support.

*That being said, any preprocessor is allowed as long as it's a sound project and has community support.*

[Dart Sass](https://sass-lang.com/dart-sass) is the primary implementation of Sass and is recommended for use in your projects.

### Command line installation[](#command-line-installation)

#### With npm[](#with-npm)

#### With homebrew[](#with-homebrew)

*   run `brew install sass/sass/sass`

#### Other installations[](#other-installations)

You can read about other installation methods on [their website](https://sass-lang.com/install).

## Specificity[](#specificity)

*   IDs should be reserved for JavaScript. Don’t use IDs for styles.

    ```
     #component { }

    .component { }
    ```

*   Don't nest more than 3 layers deep.

*   Do not fix problems with `!important`. Use `!important` purposefully.

    ```
     .component {
      width: 37.4% !important;
    }

    .hidden {
      display: none !important
    }
    ```

*   Keep specificity low and trend upwards in specificity as you move further down file. See the [specificity graph](#specificity-graph) section for more info.

*   Don't use unnecessary tag selectors.

    ```
     p.body_text { }

    .body_text
    ```

*   If you have to hack specificity, use a safe hack: the *multi class*.

    ```
     .component.component { }
    ```

### Specificity graph[](#specificity-graph)

An easy rule to use when dealing with specificity is to start from a low specificity and curve to higher specificity as you move towards the bottom of the output file. Since CSS rules get replaced by rules further down in the file, you'll override rules in an expected way.

There’s a tool that can graph your files’ specificity, [CSS specificity graph](http://jonassebastianohlsson.com/specificity-graph/). Run your final output file through this tool and strive for a curve trending upwards.

#### Resources[](#resources)

### Rationale[](#rationale-2)

With specificity comes great responsibility. Broad selectors allow us to be efficient, yet can have adverse consequences if not tested. Location-specific selectors can save us time, but will quickly lead to a cluttered stylesheet. Exercise your best judgment to create selectors that find the right balance between contributing to the overall style and layout of the DOM.

*   When modifying an existing element for a specific use, try to use specific class names. Instead of `.listings-layout.bigger` use rules like `.listings-layout.listings-bigger`. Think about ack/grepping your code in the future.
*   Use lowercase and separate words with hyphens when naming selectors. Avoid camelcase and underscores. Use human-readable selectors that describe what element(s) they style.
*   Attribute selectors should use double quotes around values. Refrain from using over-qualified selectors; `div.container` can simply be stated as `.container`.
*   IDs should be reserved for JavaScript. Unless you have a very good reason, all CSS should be attached to classes rather than IDs. When in doubt, use a class name. This prevents target confusion and allows CSS devs and JS devs to co-exist in the same code in peace. If you must use an id selector (`#id`) make sure that you have no more than one in your rule declaration.

## Units[](#units)

### Measurements[](#measurements)

*   Use **rem** units for font sizes with a px fallback. This can be done with the following mixin:

    ```
    @mixin font-size($sizeValue: 1.6) {
      font-size: ($sizeValue * 10) + px;
      font-size: $sizeValue + rem;
    }
    ```

*   Set the HTML font size to `10px` to ensure that `0.1rem` equals `1px`.

    ```
    html {
      font-size: 10px;
    }
    ```

*   Use **em** units for positioning.

*   Use **percentages** when layout components stay relational to each other (e.g. a main content area that takes up 75% of the screen and a sidebar that takes up 25%).

    ```
     .panel-a {
      width: 25%;
    }

    .panel-b {
      width: 75%;
    }
    ```

*   Use **px** units for when a measurement shouldn't change based on user set font size or browser zooming or for when requiring pixel values below 5.

    ```
     selector {
      border-width: 55px;
    }

    selector {
      border-width: 2px;
    }
    ```

*   Use unitless values for `line-height` as this will inherit values from the `font-size`.

*   Use up to 10 decimal places in em units to ensure accuracy.

    ```
     .body_copy {
      @include rem-font-size(1.4);

      line-height: 1.8;
    }

    .container {
      height: 12em;
      margin-left: 10.6666666667em;
      width: 82.5%;
    }
    ```

*   Do not use a unit with 0.

    ```
     width: 0px;

    width: 0;
    ```

*   Always use a unit for dimensions, margins, borders, padding, and typography.

    ```
     border-width: 12;

    border-width: 12px;
    ```

### Colors[](#colors)

*   Use **hex** notation first, or then **rgb(a)**, or **hsl(a)**.
*   Both three-digit and six-digit hexadecimal notation are acceptable.
*   When denoting color using hexadecimal notation, use all lowercase letters.
*   When using HSL or RGB notation, always add a single space after a comma and no space between parentheses and content.

```
 color: #FFF;
color: rgb( 255, 0, 0 );

$light: #fff;
color: $light;

$primary: #fe9848;
color: $primary;

$secondary: rgba(255, 100, 255, 0.5);
color: $secondary;
```

*   If you use an rgba rule, include a fallback value in hexadecimal.

    ```
     .illustration {
      background-color: #eee; 
      background-color: rgba(221, 221, 221, 0.75);
    }
    ```

## Variables[](#variables)

*   Create new variables in the following circumstances:

    *   The value is repeated twice
    *   The value is likely to be updated at least once
    *   All occurrences of the value are tied to the variable (for example not by coincidence)
*   When building scss that will be used across multiple projects use the `!default` flag to allow overriding.

    ```
    $baseline: 1em !default;
    ```

*   The `!global` flag should only be used when overriding a global variable from a local scope.

*   Variables across the whole scss codebase should be placed in their own file.

*   When declaring color variables, don't base the name on the color content.

    ```
     $light_blue: #18f;
    $dark_green: #383;

    $primary: #18f;
    $secondary: #383;
    $neutral: #ccc;
    ```

*   Be careful when naming variables based on their context.

    ```
     $background_color: #fff;
    ```

*   Don't use the value of dimensional variables in the variable name.

    ```
     $width_100: 100em;

    $width_lg: 100em;
    ```

*   Name all used z-indexes with a variable.

*   Have a z-index variable for each z-index used, and a separate variable, possibly aliased for where the z-index is used.

    ```
    $z_index-neg_1: -100;
    $z_index-neg_2: -200;
    $z_index-1: 100;

    $z_index-hide: $z_index-neg_2;
    $z_index-bg: $z_index-neg_1;
    $z_index-show: $z_index-1;
    ```

### Responsive design & breakpoints[](#responsive-design-and-breakpoints)

*   Set variables for breakpoints at the top of your stylesheet. This functionality is built into Bourbon.

    ```
    $sm: new-breakpoint(min-width 0 max-width 40em $sm_cols);
    ```

*   Use variables to set the queries throughout so they are easy to adapt if necessary.

*   Place media queries nearest to the class they are affecting.

*   Rather than focusing on devices when deciding where to put breakpoints, focus on content; name breakpoint variables relative to each other.

    ```
     $iphone: new-breakpoint(min-width 0 max-width 640px 6);

    $small: new-breakpoint(min-width 0 max-width 40em 6);
    $medium: new-breakpoint(min-width 0 max-width 60em 6);
    ```

</main>