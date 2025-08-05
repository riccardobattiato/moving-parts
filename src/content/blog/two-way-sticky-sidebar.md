---
pubDatetime: 2025-08-05T17:39:01.632Z
title: Dynamically sticking to opposite sides of the screen, with minimal JavaScript
featured: false
draft: false
tags:
  - frontend
  - css
  - layout
description: "Working with position: sticky can be tricky (pun intended). Luckily, we can use margins or similar properties to have better control of our sticky content's walks."
---

This week I worked on a sticky sidebar. It's one of the fundamental components of the UI, and its visual behavior is relatively complex, because the content almost always exceeds the viewport in height.

So the designers got creative. The sidebar is rendered statically in a deep area of the page and does nothing, until the user scrolls past certain thresholds:

- If the bottom of the viewport meets the bottom edge of the sidebar, the sidebar starts following the user while sticking to the bottom; the excess top content stays outside the viewport;
- If the user stops scrolling down and the viewport doesn’t meet either edge, the sidebar stops moving and stays where the user left it;
- If the user scrolls back to the top and keeps going, the sidebar will follow the user again, this time sticking to top.

I think a ~video~ working implementation speaks more than a thousand words (open the preview in a new tab for a better experience):

<iframe height="500" style="width: 100%;" scrolling="no" title="mix-blend-mode black or white only" src="https://stackblitz.com/edit/2-way-sticky-sidebar?ctl=1&embed=1&file=src%2FApp.tsx&hideExplorer=1&theme=dark&view=preview" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
</iframe>

This has been a really fun challenge, one that could be solved in many ways but never really satisfied me in terms of _smoothness_. That's right: I want my sticky content to be _smooth_, even when anchors point meet and the sidebar leaves its state of rest to stick to the screen.

## Getting rid of anchor points

The **naive implementation** was about making the concept of _anchor point_ literal. Anchor points could be actual elements, like when using [IntersectionObserver](https://developer.mozilla.org/en-us/docs/web/api/intersection_observer_api) sentinels, or particular values of `window.scrollY` if listening to the `scroll` event. When one of those was met by the appropriate viewport edge, the switch from `position: sticky` to `position: absolute` (or vice versa) would happen.

This would however make the transition _jumpy_, because even when using Observers it's impossible to get instantaneous, precise readings of the coordinates, especially with longer, non-smooth scrolls. This is why the end, smooth implementation, delegates most of the work to the browser and CSS.

Let's walk through the most important steps of the implementation.

## Implementation walkthrough

Given that we have a _container_ representing a track for the sidebar to slide on, and some main _content_ exceeding the viewport height, in 2025 we can't not use `position: sticky` for doing the user-following work.

The issue here is that the sidebar doesn't stick always to one side. For now, let's just setup a dynamic top/bottom sticky behavior without worring about the idle state, when the sidebar doesn't follow the user anymore.

```tsx
const [stickyDirection, setStickyDirection] = useState<"top" | "bottom">(
  "bottom"
);
<aside
  className={clsx("sidebar relative flex flex-col", {
    "justify-end": stickyDirection === "bottom",
  })}
>
  <div
    ref={sidebarRef}
    className={clsx("sidebar__content sticky", {
      "bottom-auto top-0": stickyDirection === "top",
      "bottom-0 top-auto": stickyDirection === "bottom",
    })}
  >
    <SidebarContent />
  </div>
</aside>;
```

It's worth noting that, for the sidebar to stick to the bottom of the screen with `top: auto; bottom: 0;`, the content must be "pushed down" to the end of the slide track: this is why you see the `justify-content: flex-end` class when the `stickyDirection === 'bottom'`; otherwise, the content would behave as if there were no more space left for the bottom stickiness to make the sidebar slide.

Now that we have top or bottom stickiness, we must alternate between them. If you're wondering how can we manage the idle state, don't worry: we'll deal with it later. Let's extract a custom hook to signal a change in scroll direction: the stickiness will update accordingly.

```ts
// Don't forget to throttle your scroll event handlers!
import { useCallback, useEffect, useRef, useState } from "react";

export const useScrollDirection = () => {
  const lastY = useRef(0);
  const ticking = useRef(false); // used for requestAnimationFrame throttling below
  const [direction, setDirection] = useState<"top" | "bottom">();

  const handleScroll = () => {
    const isScrollingBottom = window.scrollY > lastY.current;
    lastY.current = window.scrollY;
    setDirection(!isScrollingBottom ? "top" : "bottom");
  };

  const onScroll = useCallback(() => {
    if (!ticking.current) {
      requestAnimationFrame(() => {
        handleScroll();
        ticking.current = false;
      });

      ticking.current = true;
    }
  }, []);

  useEffect(() => {
    window.addEventListener("scroll", onScroll);

    return () => {
      window.removeEventListener("scroll", onScroll);
    };
  });

  return { direction };
};
```

Now the sidebar can stick to the same direction as the user is scrolling to.

```ts
// const [stickyDirection, setStickyDirection] = useState<"top" | "bottom">(
// "bottom"
// );
const { direction: stickyDirection } = useScrollDirection();

// ...
// same JSX as before
```

Now our content sticks to top when the user starts scrolling top, and vice versa. But we currently have no idle state! The sidebar jumps in a really ugly way, because the `position: sticky` will instantly bring the sidebar where we told it should stick to (in this case, the top or the bottom of the screen). No other states allowed.

So, should we introduce a third state, perhaps using `position: absolute`? No! As said before, this implementation can never be _super-smooth_. We will never be able to always precisely transition between the `absolute` and one of the `sticky` states without jumps or imprecision in coordinates. The browser should do all of that work!

### Track size rather than anchor points

The secret is: we should store the information of how much length the sidebar has already walked _in the size of the track_. Yep! We can keep the `position: sticky` always on, and make the track expand and contract to freeze the sidebar in place right when the user changes scroll direction.

We need to handle both sticky cases: top and bottom; let's start with the `bottom` one. As said before, `position: sticky` can't move the sidebar if we don't somehow create the sliding track, that's why we had to give `justify-content: flex-end` in a flexbox column.

We can use the same feature to our advantage, thanks to a little bit of `margin-top`. By restricting the length available to the top, the sidebar can save its position and stay prepared for when the bottom edges -- the former "anchor points" - will match.

```tsx
const [offset, setOffset] = useState(0); // stores current position relatively to top OR bottom
const { direction: scrollDirection } = useScrollDirection();
const [stickyDirection, setStickyDirection] = useState<"top" | "bottom">(
  "bottom"
);

useEffect(() => {
  const sidebar = sidebarRef.current;

  if (sidebar) {
    const distanceWalked = sidebar.offsetTop;

    if (scrollDirection === "bottom") {
      // Before sticking to bottom, let's freeze the distance from top
      setOffset(distanceWalked);
    } else if (scrollDirection === "top") {
      // TODO
    }

    // *Now* we can switch sticky directions
    setStickyDirection(prev => scrollDirection || prev);
  }
}, [scrollDirection]);

// ...

return (
  // ...
  <div
    ref={sidebarRef}
    className={clsx("sidebar__content sticky", {
      "bottom-auto top-0": stickyDirection === "top",
      "bottom-0 top-auto": stickyDirection === "bottom",
    })}
    style={{
      marginTop: stickyDirection === "bottom" ? `${offset}px` : 0,
    }}
  >
    <SidebarContent />
  </div>
  // ...
);
```

To recap:

1. Scroll direction changes to `bottom`;
2. We save the distance from top, `sidebar.offsetTop`, we'll use it as **top margin**;
3. Afterwards, we can let the direction change reach the sidebar itself (thanks to the additional `useState`)

We'll do the same thing when the sidebar sticks to top. In this case, we'll use `margin-bottom`, so we need to look at the distance _in the bottom_ -- the bottom space left for the sidebar to walk, before the user made the sticky switch to top:

```tsx
useEffect(() => {
  const sidebar = sidebarRef.current;
  const container = sidebarRef.current?.offsetParent;

  if (sidebar && container) {
    const distanceWalked = sidebar.offsetTop;

    if (scrollDirection === "bottom") {
      // Before sticking to bottom, let's freeze the distance from top
      setOffset(distanceWalked);
    } else if (scrollDirection === "top") {
      // Before sticking to top, let's freeze the distance from bottom
      const totalWalkingSpace = container.clientHeight - sidebar.clientHeight;
      const spaceLeftToWalk = totalWalkingSpace - distanceWalked;

      setOffset(spaceLeftToWalk);
    }

    // *Now* we can switch sticky directions
    setStickyDirection(prev => scrollDirection || prev);
  }
}, [scrollDirection]);

// ...

return (
  // ...
  <div
    ref={sidebarRef}
    className={clsx("sidebar__content sticky", {
      "bottom-auto top-0": stickyDirection === "top",
      "bottom-0 top-auto": stickyDirection === "bottom",
    })}
    style={{
      marginBottom: stickyDirection === "top" ? `${offset}px` : 0,
      marginTop: stickyDirection === "bottom" ? `${offset}px` : 0,
    }}
  >
    <SidebarContent />
  </div>
  // ...
);
```

Now, the anchoring work is **completely managed by the browser** at a **pure CSS level**. We use JavaScript only to detect scroll direction change and to assign margins. This way, anchoring and un-anchoring couldn't be smoother!

You can find the complete implementation [here](https://stackblitz.com/edit/2-way-sticky-sidebar?file=src%2FApp.tsx).
