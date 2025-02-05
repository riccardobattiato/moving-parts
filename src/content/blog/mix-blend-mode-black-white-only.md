---
pubDatetime: 2025-01-30T19:56:27.764Z
title: How to make mix-blend-mode only black or white
featured: false
draft: false
tags:
  - css
  - tutorial
  - frontend
  - svg
description: The CSS property mix-blend-mode is a classic visual effect for logos, making them contrast nicely with the background of a web page...
---

One of the coolest CSS effects I know, used by many websites since when it started being supported by modern browsers, is the `mix-blend-mode: difference` property. It's generally used on elements of sticky headers such as logos and text, since it creates a nice contrasting visual effect with the background below it as the user scrolls.

However, one of the issues developers [experience](https://www.reddit.com/r/css/comments/1542qf7/is_there_a_way_to_round_a_color_value_with/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button) [the](https://stackoverflow.com/q/67544643/29254483) [most](https://discourse.webflow.com/t/mix-blend-mode-black-white/116734) with the property is that you can't really control the colors that the `difference` or `exclusion` between the sticky elements and the background will produce. **Specifically, how to make it so that `mix-blend-mode` will display only black or white colors**.

Today we'll see a **pure CSS solution** to this known issue that uses a combination of backdrop and normal filters. _Note: the solution is currently only supported by Chromium-based browsers (Chrome, Edge, etc.)._

<iframe height="500" style="width: 100%;" scrolling="no" title="mix-blend-mode black or white only" src="https://codepen.io/riccardobattiato/embed/PwYLGWd?default-tab=result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/riccardobattiato/pen/PwYLGWd">
  mix-blend-mode black or white only</a> by Riccardo Mario Battiato (<a href="https://codepen.io/riccardobattiato">@riccardobattiato</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

## CSS filters in action

The key to the solution is simple: **the same element which blends with the background, must set a grayscale and contrast filter**. The code responsible for the effect shown in the Pen above is as simple as the following:

```css
.blending-element {
  mix-blend-mode: difference;
  backdrop-filter: grayscale(1) contrast(100); // be careful: the order matters!
  background-color: black;
}
```

Setting a background color isn't always required, but remember that **in order for `mix-blend-mode` to work, both the blending element and the background must have an actual color**. If your body looks `white` but is really `transparent`, there will be no colors to blend.

Once `mix-blend-mode` is set, you'll have the typical inverted-colors look: unless you have a perfectly black or white background, the blending element may show unwanted colors. This is where CSS filters come into action: **grayscale** will force any color into a tint of black or white, while setting a really high **contrast** value will "round" the grayscale tints to pure black or white.

You may have noticed that we're using a [**backdrop-filter**](https://developer.mozilla.org/en-US/docs/Web/CSS/backdrop-filter) rather than a [normal one](https://developer.mozilla.org/en-US/docs/Web/CSS/filter). There are two reasons for this:

1. First, it's becauase moving the `grayscale(1)` parameter to a normal, non-backdrop filter simply won't work. The solution is slightly hacky, and it needs for the backdrop to filter its colors to grayscale.
2. The `contrast(/* really high value here */)` on direct elements makes them look way worse. It's better to rely on the backdrop and let the `mix-blend-mode` do its job on the actual element anyway.

Relying on `backdrop-filter` has one issue: even if you have a transparent background behind your blending element, **the rectangle behind it will look black or white too**. This is fine for rectangular elements such as the simple scroller inside the CodePen example, but not for complex shapes such as the [planet logo](https://thenounproject.com/icon/planet-7302521/) from the [Noun Project](https://thenounproject.com/) shown in the same example.

## What about complex shapes?

There's another really useful CSS property that will solve our problem: [clip-path](https://developer.mozilla.org/en-US/docs/Web/CSS/clip-path). Thanks to it, we're able to clip our visual page elements to simple shapes, custom polygons and **custom SVG paths**. The latter is the most flexible approach and the one I like the most.

1. **Get your complex shape and/or text as SVG code.** This could be an external file or an inline SVG such as in the example below.
2. Move the paths and other elements inside a `<clipPath>` definition element, as done below.
3. Set an `id="..."` for the `<clipPath>` element: the CSS code will need to reach it.

You should get something like this:

```xml
  <svg xmlns="http://www.w3.org/2000/svg" version="1.1" viewBox="-5.0 -10.0 110.0 135.0">
    <defs>
      <clipPath id="complex-shape">
        <path d="m36.438 79.031c-2 1.5-3.9375 2.8438-5.75 4.0312-2.6289-1.5703-5.0547-3.4609-7.2188-5.625-9.4805-9.4805-13.18-23.293-9.7109-36.242 3.4688-12.949 13.582-23.062 26.531-26.531 12.949-3.4688 26.762 0.23047 36.242 9.7109 2.1719 2.1719 4.0625 4.6055 5.625 7.25-1.2188 1.8125-2.5312 3.7188-4.0312 5.7188-5.9023 7.9023-12.391 15.348-19.406 22.281-6.9336 7.0156-14.379 13.504-22.281 19.406zm26.625-15.062c-6.1797 6.207-12.723 12.039-19.594 17.469-2.125 1.6875-4.2188 3.25-6.2812 4.6875 8.9844 3.2617 18.875 2.9688 27.652-0.82031 8.7734-3.7891 15.77-10.785 19.559-19.562 3.7891-8.7734 4.082-18.664 0.82031-27.648-1.4688 2.0625-3.0312 4.1562-4.6562 6.25-5.4414 6.8828-11.285 13.434-17.5 19.625zm-38.25 22.719c-8.1875 4.6562-13.625 5.9062-15.062 4.4688s-0.1875-6.8438 4.4688-15.031c-1.3711-1.9453-2.5703-4.0078-3.5938-6.1562-6.9688 11.219-9.9062 20.844-5.2188 25.531 1.6641 1.5742 3.8984 2.3984 6.1875 2.2812 5.0938 0 11.906-2.8438 19.375-7.4688 2.0312-1.25 4.0938-2.6562 6.2188-4.1875-2.2695-0.78906-4.4453-1.8164-6.5-3.0625-2.0938 1.375-4.0625 2.5938-5.875 3.625zm69.781-80.375c-4.0938-4.0938-12.125-2.6562-23.844 4.25-0.53125 0.3125-1.0938 0.65625-1.625 1 2.1172 1.0117 4.1484 2.2031 6.0625 3.5625 3.9414-2.6719 8.4609-4.3672 13.188-4.9375 0.66016-0.058594 1.3203 0.10547 1.875 0.46875 1.4375 1.4375 0.1875 6.875-4.4688 15.062-1.0312 1.8438-2.25 3.8125-3.625 5.9062 1.2461 2.043 2.2734 4.2109 3.0625 6.4688 1.5-2.125 2.9062-4.2188 4.1562-6.25 6.9688-11.219 9.9062-20.844 5.2188-25.531z" />
        <text x="0.0" y="117.5" font-size="5.0" font-weight="bold" font-family="Arbeit Regular, Helvetica, Arial-Unicode, Arial, Sans-serif" fill="#000000">Created
          by Ricons</text>
        <text x="0.0" y="122.5" font-size="5.0" font-weight="bold" font-family="Arbeit Regular, Helvetica, Arial-Unicode, Arial, Sans-serif" fill="#000000">from
          Noun Project</text>
      </clipPath>
    </defs>
  </svg>
```

Afterwards, you need to prepare a `<div />` or other element to be the one that will be clipped by your custom path. Personally, I like to reuse my SVG as an inline element in the page, both for providing the `<clipPath>` definition _and_ for providing a visual element to clip.

If you use an empty element or an SVG which doesn't display actual elements by itself (as in my example), remember to specify a width or height you're about to clip:

```css
.complex-shape {
  width: 6.875rem; // Warning: this has to be really precise!
}
```

Once the element is in page and has some dimensions, set the `clip-path` property so that it references your SVG definition:

```css
.complex-shape {
  width: 6.875rem;
  clip-path: url(#clipping-shape);
}
```

Now you can make your black and white backdrop appear, and it will have the same shape or text from your SVG:

```css
.complex-shape {
  width: 6.875rem;
  clip-path: url(#clipping-shape);
  mix-blend-mode: difference;
  backdrop-filter: grayscale(1) contrast(100); // be careful: the order matters!
}
```

Feel free to be creative with `clip-path` — there are many ways to use it, and many other ways to provide a valid SVG `<clipPath>` definition.

Let me know if you have any improvements to this solution or find even easier ways to accomplish the effect!
