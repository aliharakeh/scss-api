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

### CSS variable naming:

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

### Default usage

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

## Note

Don't use theme state APIs for inner components when creating a complex/wrapper component. Instead, use the related
property API, as you are not changing the component state, but just updating it according to the parent state.

```scss
@use 'scss/theme';

// WRONG [X]
.container .inner-child {
  @include theme.hover(color red);
}

/*
  Output
*/
.container .inner-child {
  --hover-color: red;
}

// ------------------------------------------------

// CORRECT [✓]
.container .inner-child:hover {
  @include theme.style(color red);
}

/*
  Output
*/
.container .inner-child:hover {
  --color: red;
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

# What is the difference between adding a default value in `vars` API & `styles` API

You can add a default value to both the `vars` and `styles` APIs. Usually, you should use a `vars` API default value.
A `styles` API default value is good to use when you have a cascaded variable from a parent element, as in that case you
will not have any variable defined (as it will override the parent variable).

# Tutorial 1 - `vars` & `styles` APIs

In this tutorial, I'll explain how to these APIs to create a custom label component theme and how to modify it.

## 1) Deciding which CSS styles to create CSS variables for

The label component has the following design criteria:

- Change the text color.
- Change the font size, weight, line-height, and family.
- Add a min-width to truncate any overflowing text.

## 2) Creating the variables

- Use the `vars` API to create the variables.
- You can define CSS variables in the same way you define any other JavaScript or TypeScript variables in your
  component. You define them so you can use them later.

**Note 1**: We don't define a **color** variable, because if we do, the vars API will overwrite any **cascaded parent
variable** value.

**Note 2**: Always define your variables inside `:host {...}` so they are available anywhere in the component tree.

```scss
@use 'scss/vars';

:host {
  @include vars.style(line-height, min-width);
  @include vars.font(family, size, weight);
}

/*
  Output
*/
:host {
  --line-height: initial;
  --min-width: initial;
  --font-family: initial;
  --font-size: initial;
  --font-weight: initial;
}
```

## 3) Connecting the created variables to the component styles

- We use the `styles` API to connect the CSS variables to the component styles. You can think of it like using your
  JavaScript or TypeScript variables to connect your HTML template to your data. In this case, we are just connecting
  the CSS instead.
- In the label component case, we need to apply the styles to the inner `span` and the `truncate` class. Connecting
  these variables is as easy as copying the variable definition and replacing the vars keyword with the styles keyword.

**Note**: We also include the `color` property in our styles, even though it doesn't have any variable defined. We do
this because we want to be able to override the cascaded value from any parent element.

```scss
@use 'scss/vars';
@use 'scss/styles';

:host {
  /* variables */
  @include vars.style(line-height, min-width);
  @include vars.font(family, size, weight);

  /* styles */
  &.truncate-enabled {
    @include styles.style(min-width);
  }

  span {
    @include styles.style(color, line-height);
    @include styles.font(family, size, weight);
    @include styles.text(align);
  }
}

/*
Output
*/
:host {
  --line-height: initial;
  --min-width: initial;
  --font-family: initial;
  --font-size: initial;
  --font-weight: initial;

  &.truncate-enabled {
    min-width: var(--min-width);
  }

  span {
    color: var(--color);
    line-height: var(--line-height);
    font-family: var(--font-family);
    font-size: var(--font-size);
    font-weight: var(--font-weight);
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

## 5) Final Result

```scss
@use 'scss/vars';
@use 'scss/styles';

// global
$g-font-family: var(--g-font-family, inherit);
$g-text-color: var(--g-text-color, black);

// label specific
$min-truncate-width: 30ch;
$font-size: 16px;
$line-height: 20px;
$font-weight: 400;

:host {
  @include vars.style(
      line-height $line-height,
      min-width $min-truncate-width
  );
  @include vars.font(
      family $g-font-family,
      size $font-size,
      weight $font-weight
  );

  // other styles ...

  span {
    @include styles.style(color $g-text-color, line-height);
    @include vars.font(family, size, weight);

    // other styles ...
  }

  // other styles ...
}

/*
  Output
*/
:host {
  --min-width: 30ch;
  --font-family: var(--g-font-family, inherit);
  --font-size: 16px;
  --line-height: 20px;
  --font-weight: 400;

  // other styles ...

  span {
    color: var(--color, var(--ds-text-color, black));
    line-height: var(--line-height);
    font-family: var(--font-family);
    font-size: var(--font-size);
    font-weight: var(--font-weight);

    // other styles ...
  }

  // other styles ...
}
```

# Tutorial 2 - CSS `vars` & `styles` States APIs :

State CSS styles differ a bit from what was described above, and I'll use the `ds-button` component to explain them.

## State vars

State vars differ with normal vars in that their they have 1 extra fallback value which is the variable of the affected
property.

They create a state variable that is automatically connected to the related property variable.

This is necessary to prevent a broken UI in case there was no default value for these state variables.

**Note 1:** having `_` before any property name means that we want to use the alias property variable not the actual one
as there might be a name conflict if an inner component require this variable, and it has the same property variable
name. exmaple: button uses a label/icon inside, and both also have a color/font variable with the same name as the
button ones.

**Note 2:** only hover & active states take a property variable fallback, as the other states are not affected by the
property variable and will fall back to the direct value provided when defining their vars.

```scss
@use 'vars';

:host {
  // vars ...

  // state vars
  @include vars.hover(_color, background-color red, border-color, opacity);
}
```

The result will be something like this

```scss
:host {
  // vars ...

  // state vars
  --hover-color: var(--_color);
  --hover-background-color: var(--background-color, red);
  --hover-border-color: var(--border-color);
  --hover-opacity: var(--opacity);
}
```

## State Styles

State styles are the same as normal styles, as they just apply the state variables to the related property.

```scss
@use 'styles';

button {
  &:hover {
    @include styles.hover(color, background-color, border-color, opacity);
  }
}
```

The result will be something like this

```scss
button {
  &:hover {
    color: var(--hover-color);
    background-color: var(--hover-background-color);
    border-color: var(--hover-border-color);
    opacity: var(--hover-opacity);
  }
}
```

## State Default Values

As mentioned above:

- Hover & Active states take a property variable fallback with a default value if that property variable was not
  defined.
- Disabled, Focus, & Error states takes a direct value as other normal vars.
