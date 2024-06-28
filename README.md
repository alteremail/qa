# QA Checklist

Every client project needs to go through this QA checklist.

## Layouts

Some layout and columns combinations that can be used.

Use [gridcalculator.dk](http://gridcalculator.dk) to check any other custom layouts.

| Container | 2 Columns | 3 Column | 4 Columns | All Even |
|-----------|-----------|----------|-----------|----------|
| 720       | 360       | 240      | 180       | Yes      |
| 708       | 354       | 236      | 177       |          |
| 696       | 348       | 232      | 174       | Yes      |
| 684       | 342       | 228      | 171       |          |
| 672       | 336       | 224      | 168       | Yes      |
| 660       | 330       | 220      | 165       |          |
| 648       | 324       | 216      | 162       | Yes      |
| 636       | 318       | 212      | 159       |          |
| 624       | 312       | 208      | 156       | Yes      |
| 612       | 306       | 204      | 153       |          |
| 600       | 300       | 200      | 150       | Yes      |
| 588       | 294       | 196      | 147       |          |
| 576       | 288       | 192      | 144       | Yes      |
| 564       | 282       | 188      | 141       |          |
| 552       | 276       | 184      | 138       | Yes      |
| 540       | 270       | 180      | 135       |          |
| 528       | 264       | 176      | 132       | Yes      |
| 516       | 258       | 172      | 129       |          |
| 504       | 252       | 168      | 126       | Yes      |

## Code

### Attributes order 

Use a consistent HTML attributes order for all elements:

- `<td|div|etc align?="" class="" style?="">`
- `<img src="" width="" class="" alt="">`
- `<a href="" class="" style?="">`

`?` = optional

Maizzle adds the following attributes by default:

- tables: `cellpadding="0"`, `cellspacing="0"`, `role="presentation"`
- images: `alt=""`

Always try to describe images with alt text, writing it so that it makes sense when images don't load.

For example, a social icon might have `alt="Facebook"` or `alt="Like us on Facebook"` depending on where it's placed and how much space is available.

On the other hand, if there's a button like "Read more &rarr;" and the arrow is an image, it's better to not add any alt text. This way Maizzle just adds `alt=""` to it, instructing screen readers to skip the image.

### Class order

Always write the desktop/to-be-inlined class first, followed by its responsive variants.

The following class order should be used:

1. display
2. width, height 
3. margin, padding
4. text: size, leading, align
5. colors: text, background
6. specials: background image, rounded, shadow, ...

Example:

```html
<div class="hidden sm:block w-[600px] sm:w-full m-0 text-base leading-6 text-right text-gray-600 bg-white bg-image-hero rounded-lg shadow-2xl"></div>
```

As you can see, we group `w-[600px]` with `sm:w-full` so it's immediately clear how the element behaves.

### Styles > Attributes

Always use utility classes instead of HTML attributes. CSS pixel values scale better on high-density screens.

```diff
- <table width="64">
+ <table class="w-16">
```

For larger or one-off values, use arbitrary values as they are more readable:

```diff
- <table class="w-150">
+ <table class="w-[600px]">
```

#### Exceptions

Width and height of images must _always_ be defined through the `width` and `height` attributes, to ensure Outlook compatibility (especially with retina images):

```diff
- <img src="" class="w-6">
+ <img src="" width="24">
```

Maizzle includes a base `img` CSS reset in `src/css/resets.css`, so 99% of the time all you really need for an image are the `src` and the `width`.

`align` on tables or other elements: 

```diff
- <table class="mx-auto">
+ <table align="center" class="mx-auto">
```

Note that depending on the element, you might still need to use some CSS - like is the case with `mx-auto` above.

Also, `align` behaves differently on tables (it floats them) than on table cells (it aligns the text inside).

### Inline styles > Configuration

Prefer writing inline CSS instead of registering new utilities in `tailwind.config.js`. The less config, the better.

For example, when adding a background image, inline CSS makes it clear immediately to anyone viewing the HTML, as opposed to a custom-named utility class:

```diff
- <td class="bg-image-hero">
+ <td style="background-image: url('https://example.com/hero.jpg')">
```

This could also be done with arbitrary values in Tailwind, though maybe still the inline CSS in this case is clearer:

```html
<!-- Arbitrary value -->
<td class="bg-[url(https://example.com/hero.jpg)]">

<!-- Inline CSS -->
<td style="background-image: url('https://example.com/hero.jpg')">
```

### Brevity

The code that we deliver to our clients must be as clean and weigh as less as possible.

There are a few tricks we can use to ensure we generate shorter code.

#### Inheritance and client defaults

Try to use CSS inheritance and browser/email client defaults as much as possible.

For example, the `font-family` can mostly be defined just once, on the outermost root element (but not `<body>`, since that gets removed sometimes). 

And only if we have some element that needs to use a different font, do we specify it, overriding the inherited one. The value for this is usually long and can quickly add up bytes if we're not careful. There are some exceptions to this, like in Gmail web where the font-family is not always properly inherited.

Likewise, things like colors, font sizes, line heights are often also inherited, so no need to always specify them on every single element. This one needs more attention, as it depends on the elements inside (`<a>` do not inherit color for example) and it can also cause issues in Outlook (i.e. with `line-height`).

Also, headings are bold by default in all email clients, no need to use `font-bold` on them.

`<th>` has bold and centered text by default, and so on...

Avoid adding code bloat "for good measure" - that's a code smell. Always aim for the least amount of code, test, and add from there if needed.

#### Shorthand CSS

For margins and paddings, instead of defining just two sides, define them all by setting the other two sides to 0:

```diff
- <div class="my-4">
+ <div class="my-4 mx-0">
```

That will compile to less code in Maizzle thanks to the [shorthand transformer](https://maizzle.com/docs/transformers/shorthand-css), here's a comparison:

```diff
- <div style="margin-top: 16px; margin-bottom: 16px">
+ <div style="margin: 16px 0">
```

A simpler way to do this is to reset to `0` and then override the sides you need. 

Here's a paragraph example:

```diff
- <p class="mx-0 mt-0 mb-4">
+ <p class="m-0 mb-4">
```

In this case we save even more bytes:

```diff
- <div style="margin-top: 0; margin-right: 0; margin-bottom: 16px; margin-left: 0">
+ <div style="margin: 0 0 16px">
```

#### Legibility

Maizzle source code, as well as the compiled, production-ready HTML, need to be legible.

We produce this code for humans to read first, machines come second.

In this sense, we must always make sure that:

- we use `.editorconfig` (install VS Code plugin, add config file at the root of the project)
- we carefully format our source code for legibility:
  
  ```diff
  - <p>Some very long paragraph [...] on a single line...</p>
  + <p>
  +   Some very long paragraph [...] on multiple 
  +   lines (try to stick to less than 100 characters per line for copy)
  + </p>
  ```
  
  When an element has many attributes, write them each on their own line:
  
  ```diff
  - <div role="article" aria-roledescription="email" aria-label="{{{ page.title || '' }}}" lang="{{ page.language || 'en' }}" class="font-sans">
  + <div
  +   role="article"
  +   aria-roledescription="email"
  +   aria-label="{{{ page.title || '' }}}"
  +   lang="{{ page.language || 'en' }}"
  +   class="font-sans"
  + >
  ```

#### Premature component abstractions

Avoid premature abstractions, don't extract to components just for the fun of it.

A subtitle should just be a `<p>` or `<h3>` for example, not a `<subtitle>...</subtitle>` component that doesn't make it obvious what HTML will be rendered.

In general, create and use components if:

- they're globally recognizable (like the `<x-spacer>` or `<x-divider>` components Maizzle provides)
- they're reusable blocks that are complex or could be made "dynamic" by passing them props (as an example, the `<x-button>` or VML components in Maizzle)

## Accessibility

### `alt` attributes

- ensure `alt` attributes on all images that need a description
- decorative images should use an empty `alt` attribute (handled by Maizzle)
- images containing text should be avoided by all means

Optional: styled `alt` text, where applicable. As more and more email clients block images by default, this can help make an email stand out in the inbox.

### `aria` roles

Make sure `aria` roles are properly set, where applicable.

Note: `<hr>` elements do not need a `role="separator"`, they are already semantically correct.

#### Spacers

With `<div>`:

```html
<div class="leading-8" role="separator">&zwj;</div>
```

With `<table>`:

```html
<table class="w-full" role="separator">
  <tr>
    <td class="leading-4">&zwj;</td>
  </tr>
</table>
```

With `<tr>`:

```html
<tr role="separator">
  <td class="leading-4">&zwj;</td>
</tr>
```

#### Dividers

With `<hr>`:

Although they are the best choice semantically speaking, we avoid using `<hr>` dividers in our code as they are impossible to fully control in Outlook on Windows, where they sometimes render wider than their container and there's nothing that can be done about it.

With `<div>`:

This works well in all email clients:

```html
<div class="h-px leading-px bg-slate-300 my-8" role="separator">&zwj;</div>
```

### `<title>` and preview text

Provide a unique `<title>` for each Template, if possible. It can help with accessibility and also SEO if the email is published as a "web version" or an online archive.

Preview text, sometimes referred to as a _preheader_, is the text that appears in the inbox after the subject line. This can use the same value as the `<title>`, but most likely you'll want it to be different so it adapts better to the email client's UI.

### Text

Text should be accessible in terms of sizing and colors, and people need to be able to zoom into an email.

#### Sizes

We generally aim to use even numbers for text sizes, line heights, spacers etc. This helps avoid subpixel rendering issues in some email clients.

Sometimes that's not an option, so it's totally fine to use arbitrary values for one-off text/leading sizes (i.e. `text-[17px]`). Just pay attention to render tests in this context.

#### Colors

If a design does not use the default Tailwind CSS colors, we have two options:

1. Arbitrary values for quick one-offs (i.e. `text-[#FFCC00]`)
1. Define the palette if there are more than 2 shades

When defining a palette in `tailwind.config.js`, always follow the same scale as Tailwind: start from `50` for the lightest and go to `950` for the darkest. Use `DEFAULT` for the default color:

```js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#...', // text-primary-50 (lightest)
          DEFAULT: '#...', // text-primary (default, no shade)
          950: '#...', // text-primary-950 (darkest)
        }
      }
    }
  }
}
```

### Semantic code

- use `<h2>` - `<h4>` for headings (`h1` only if we are really sure it won't be repeated)
- use `<p>` for paragraphs
- use `<em>` if semantics matter, `<i>` otherwise
- use `<strong>` if semantics matter, `<b>` otherwise
- use `ul` and `ol` for lists
- use `<span class="text-sm">`, not `<small>`
- Link buttons (no VML, except for Outlook rounded corners)

### Miscellaneous

- there should be no `title` attributes on any element
- animation, if added, must respect `prefers-reduced-motion`

## Render Testing

At the very minimum, the email template needs to render well in these email clients.

### Mobile

- iPad
- iPhone
- Gmail iOS
- Gmail Android
- Outlook iOS
- Outlook Android
  
### Desktop

- Apple Mail
- Outlook
- Thunderbird
  
### Webmail

- Gmail
- Office 365 / Outlook.com
- Yahoo! / AOL
