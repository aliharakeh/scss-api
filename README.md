<!-- TOC -->
* [Intro](#intro)
* [Why are these APIs useful?](#why-are-these-apis-useful)
* [Vars API](#vars-api)
  * [When to use?](#when-to-use)
  * [Where to use it?](#where-to-use-it)
  * [How to use it?](#how-to-use-it)
    * [CSS variable naming](#css-variable-naming)
    * [Single CSS property function](#single-css-property-function)
    * [Custom variable function](#custom-variable-function)
* [Styles API](#styles-api)
  * [When to use?](#when-to-use-1)
  * [Where to use it?](#where-to-use-it-1)
  * [How to use it?](#how-to-use-it-1)
    * [Usage](#usage)
    * [Connecting a style to a custom variable](#connecting-a-style-to-a-custom-variable)
* [Theme API](#theme-api)
  * [When to use?](#when-to-use-2)
  * [Where to use it?](#where-to-use-it-2)
  * [How to use it?](#how-to-use-it-2)
* [Api args](#api-args)
* [What is the difference between adding a default value in `vars` API & `styles` API](#what-is-the-difference-between-adding-a-default-value-in-vars-api--styles-api)
* [Tutorial](#tutorial)
  * [1) Specifying what we need](#1-specifying-what-we-need)
  * [2) Adding variables](#2-adding-variables)
  * [3) Connecting the variables to the component styles](#3-connecting-the-variables-to-the-component-styles)
  * [4) Adding default values](#4-adding-default-values)
    * [Cases:](#cases)
  * [5) Adding Dark theme](#5-adding-dark-theme)
  * [6) Adding State](#6-adding-state)
    * [Option 1: Static hover styles - `theme` api.](#option-1-static-hover-styles---theme-api)
    * [Option 2: Dynamic State Styles - State APIs](#option-2-dynamic-state-styles---state-apis)
<!-- TOC -->

# Intro

Design libraries typically have a theme system with local variables for styling components and global variables for
styling the theme itself, such as dark themes. However, I found that there was some inconsistency in how we defined and
used these local variables. To address this, I created a sub-API system to manage styling.

The Styling-API system consists of three small APIs:

The first API helps us define local variables and set their initial values.
The second API helps us connect these variables to the style of the component.
The third API is for changing the theme of the components through the variables.
To use the Styling-API system, simply define your local variables, connect them to the styles of the components, and
then use the third API to change the theme as needed.

The Styling-API system provides a consistent way to manage styling, so you don't need to know what is happening
internally. You can simply use normal CSS property names to generate the related variables that are needed inside the
component or the theme.

# Why are these APIs useful?

These APIs make it easier to manage CSS variables without having to remember them and provide code hints and help when
needed.

# Vars API

## When to use?

When you need to use a CSS variable to configure your component theme.

## Where to use it?

Inside your component's top most container block. ex: `:host{ ... }`

## How to use it?

1. Decide on the CSS property you want to update through a variable.
2. Use the related `vars.<style-api>(...)` function to define your variable, if it is not cascaded. Otherwise, do not
   add it, as it will overwrite the parent inherited value.
3. Add a default value to your variable, in case it needs to be initialized. Otherwise, it will take the default value
   of `initial`.
    - css properties should not take a value if they are needed to be changed later. (ex: like border-color that needs
      to be changed later on hover state)

### CSS variable naming

Each Vars API function provides a property prefix for related CSS properties. Your CSS variable will be named as your
CSS property, with the property prefix prepended.

```scss
@use 'scss/vars';

.container {
    // define vars
    @include vars.border(color, width 1px);
    @include vars.background(color blue);
    @include vars.padding(paddin 1rem);
}

/*
  Output
*/
.container {
    --border-color: initial;
    --border-width: 1px;
    --background-color: blue;
    --padding: 1rem;
}
```

### Single CSS property function

The `vars.styles(...)` function is used for any single CSS property.

```scss
@use 'scss/vars';

.container {
    // define vars
    @include vars.style(color red, width 100px);
}

/*
  Output
*/
.container {
    --color: red;
    --width: 100px;
}
```

### Custom variable function

The `vars.custom(...)` API function is used for custom variables.

```scss
@use 'scss/vars';

.container {
    // define vars
    @include vars.custom(image-size 50%);
}

/*
  Output
*/
.container {
    --image-size: 50%;
}
```

# Styles API

## When to use?

when you need to connect a css style to your defined css variables.

## Where to use it?

inside you component classes.

## How to use it?

1. Choose the CSS style you want to update with a variable.
2. Use the related `styles.<func>(...)` function to define your style. The API will match the style with the variable
   created through the `vars.<func>(...)` function.
3. Add a default value to your style if you did not specify one in the Vars definition.

### Usage

```scss
@use 'scss/vars';
@use 'scss/styles';

.container {
    // define vars
    @include vars.style(color red, width 100px);
}

.container .child {
    // define styles
    @include styles.style(color, width);
}

/*
  Output
*/
.container {
    --color: red;
    --width: 100px;
}

.container .child {
    color: var(--color);
    width: var(--width);
}
```

### Connecting a style to a custom variable

```scss
@use 'scss/vars';
@use 'scss/styles';

.container {
    // define vars
    @include vars.custom(child-width 100px);
}

.container .child {
    // define styles
    @include styles.connect-css-to-var(width child-width);
}

/*
  Output
*/
.container {
    --child-width: 100px;
}

.container .child {
    width: var(--child-width);
}
```

# Theme API

## When to use?

When you need to modify the styles or theme of your component.

## Where to use it?

- External CSS to modify the component styles.
- When you want to reuse a component in another, and you need to modify the styles a bit.

## How to use it?

1. Choose the CSS style you want to update.
2. Use the related `theme.<func>(...)` to modify your styles.

```scss
@use 'scss/vars';
@use 'scss/styles';
@use 'scss/theme';

.container {
    // define vars
    @include vars.style(color black);
    @include vars.background(color white);

    // define styles
    @include styles.style(color black);
    @include styles.background(color white);
}

.container.dark {
    // switch theme
    @include theme.style(color white);
    @include theme.background(color black);
}

/*
  Output
*/
.container {
    --color: white;
    --background-color: black;

    color: var(--color);
    background-color: var(--background-color);
}

.container.dark {
    --color: black;
    --background-color: white;
}
```

# Api args

Each API function takes from 1 to 3 arguments:

- **CSS Property Name**: `color`, `width`, ...
- **Default Value**
    - CSS Value: `red`, `blue`, .....
    - SCSS Variable: `global.$ds-color`, `$some-value`, `button.$padding`, ....
    - Direct Value: `0px`, `100%`
    - Complex Direct Value:
        - Use (...) to set the value as a single term
        - Ex: `(8px 16px)`, `(0px 0px 0px 1px)`, ...
- **Modifier**: `!important`

**Note**: Having a `_` before any property name means that we want to use the alias property variable, not the actual
one, as there might be a name conflict if an inner component requires this variable where it has the same property
variable name.

# What is the difference between adding a default value in `vars` API & `styles` API

You can add a default value to both the `vars` and `styles` APIs. Usually, you should use a `vars` API default value.
A `styles` API default value is good to use when you have a cascaded variable from a parent element, as in that case you
will not have any variable defined (as it will override the parent variable).

# Tutorial

In this tutorial, I'll explain how to these APIs to create a custom card component theme and how to modify its theme.

## 1) Specifying what we need

The card component consists of 3 sections:

- Image Content
- Text Content
- Actions

Both the Content and Action sections will change their styles to match the current theme.

**Here is a basic sample card we will start with:**

```html

<div class="card">
    <img
        src="https://www.w3schools.com/howto/img_avatar.png"
        alt="Avatar"
        class="card-image"
    >
    <div class="card-content">
        <div>
            <h3>John Doe</h3>
            <p>Architect & Engineer</p>
        </div>
        <button class="card-action">View More</button>
    </div>
</div>
```

```scss
.card {
    overflow: hidden;
    border-style: solid;
    border-width: 1px;
    border-radius: 8px;

    &-image {
        width: 100%;
    }

    &-content {
        padding: 8px 1rem;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }

    &-action {
        align-self: flex-end;
        background: transparent;
        padding: 8px 1rem;
        cursor: pointer;
        font-size: 20px;
        border-radius: 10px;
    }
}
```

## 2) Adding variables

- Use the `vars` API to add a variable.
- You define them so that you can use them later.
- variables name will use the api function name as a prefix and append the property you chose to define the variable for
  tp it.

```
<api>.<function>(<property>) = --<funvtion>-<property>

Example:
-------
vars.background(color) = --background-color
```

**Note**: Always define your variables inside your top host element, so they will be available anywhere in the component
tree.

```scss
@use 'scss/vars';

.card {
    // define your css variables with css property-like names & default values
    // through the vars api
    @include vars.style(color);
    @include vars.background(color);

    // ...
}
```

## 3) Connecting the variables to the component styles

- We use the `styles` API to connect the CSS variables to the component styles. You can think of it like using your
  JavaScript or TypeScript variables to connect your HTML template to your data. In this case, we are just connecting
  the CSS instead.
- In the card component case, we need to apply the styles to both content & action sections.
- We have 2 ways to connect these variables to our styles:
    - connect the style to a variable with the same css property.
    - connect the style to a custom variable or one related to another property.

```scss
@use 'scss/vars';
@use 'scss/styles';
@use 'scss/theme';

.card {
    // ...

    // set the background-color css property by the background-color variable
    @include styles.background(color);

    &-image {
        // ...
    }

    &-content {
        // ...

        // set the color css property by the color variable
        @include styles.style(color);
    }

    &-action {
        // ...

        // add css
        border-width: 1px;
        border-style: solid;

        // connect different css properties to any variable you want
        @include styles.style(color);
        @include styles.connect-css-to-var(border-color color);
    }
}
```

## 4) Adding default values

To finish theming our component, we need to add some default style values that will be applied by default. We have two
options for adding a default value:

- `vars` API
- `styles` API

The API you choose depends on your needs.

### Cases:

a) We have a variable defined, and it **cannot be undefined** (undefined = initial CSS value). In this case, we can add
a
default value directly to the variable. For example:

```scss
@use 'scss/vars';

:host {
    @include vars.style(color red);
}
```

b) We have **parent cascaded variables**. In this case, we do not have any variable defined (as it will overwrite the
parent
value), so we use the styles API default value. For example:

```scss
@use 'scss/styles';

:host {
    @include styles.style(color red);
}
```

c) We need to **update the component style externally** (from a parent component for example). In this case, we use the
styles API to add new styles from external sources. We give it a default value since we do not have a variable inside
the component for this custom-added style.

**In out case, we will add them to the `vars` api:**

```scss
@use 'scss/vars';
@use 'scss/styles';
@use 'scss/theme';

.card {
    // define your css variables with css property-like names & default values
    // through the vars api
    @include vars.style(color black); // default value
    @include vars.background(color lightgray); // default value

    // ...
}
```

## 5) Adding Dark theme

We use the `themes` api to modify the component theme

```scss
@use 'scss/vars';
@use 'scss/styles';
@use 'scss/theme';

.card {
    // ...

    // add dark theme
    &.dark {
        // update the css vars though the theme api
        @include theme.style(color white);
        @include theme.background(color #333232);
    }
}
```

## 6) Adding State

We have **2 options** to add a state to our action button.

### Option 1: Static hover styles - `theme` api.

```scss
@use 'scss/vars';
@use 'scss/styles';
@use 'scss/theme';

.card {
    // ...

    &-action {
        // ...

        // update the color variable value on hover to a specific value
        &:hover {
            @include theme.style(color blue, border-color blue);
        }
    }

    // ...
}
```

### Option 2: Dynamic State Styles - State APIs

- `State variables` differ from normal variables in that they have an extra fallback value, which is the variable of the
  affected property. State variables create a variable that is automatically connected to the related property variable.
  This is necessary to prevent a broken UI in case there is no default value for these state variables.
  - Only hover and active states take a property variable fallback, as the other states are not affected by the property
    variable and will fall back to the direct value provided when defining their variables.
- `State Styles` are the same as normal style, they will connect the state var to the css property.
- `State Theme` is also the same as any normal theme api function, it will set the variable's value directly.

```scss
@use 'scss/vars';
@use 'scss/styles';
@use 'scss/theme';

.card {
    // ...
    
    @include vars.hover(
        color, // --hover-color: var(--color);
        border-color // --hover-border-color: var(--border-color);
    );

    // ...
    
    &-action {
        // ...

        // update color style on hover through external theme
        &:hover {
            @include styles.hover(
                color, // color: var(--hover-color);
                border-color // border-color: var(--border-color);
            );
        }
    }

    // add light theme config
    &.light {
        // update hover color theme
        @include theme.hover(
            color blue, // --hover-color: blue;
            border-color blue // --hover-border-color: blue;
        );
    }

    &.dark {
        // ...

        // update hover color theme
        @include theme.hover(
            color yellow, // --hover-color: yellow;
            border-color yellow // --hover-border-color: yellow;
        );
    }
}
```
