---
pubDatetime: 2024-05-04T09:27:40.306Z
title: Design Patterns for Vue.js Book Review
featured: false
draft: false
tags:
  - development
  - design-patterns
  - testing
  - vue
  - book-review
description: Design Patterns for Vue.js is a book about first principles, separation of concerns and quality frontend written by the author of Vue Test Utils, Lachlan Miller.
---

## Table of contents

## Introduction
When considering books on software engineering, there are two different requirements they could satisfy in order to become an instant favorite of mine.

The first is a focus on **first principles**: in modern web development we are _spoiled for options_, tutorials and books showing how to use this specific framework or that one API; rather, a better and longer-lasting approach is the one **treating tools as just the tools they are**, however good, and uses them to **showcase the underlying concepts**.

The second is having **practical value**: I love reading about ideas and the philosophy behind solutions, but we live in the real world and have to solve real problems using real tools; I recognize a good book because I'll be putting into practice what I learned maybe even the day after reading it.

In _Design Patterns for Vue.js - a Test Driven Approach to Maintainable Applications_, [Lachlan Miller](https://lachlan-miller.me) -- who's also the [creator](https://github.com/lmiller1990) of [Vue Test Utils](https://github.com/vuejs/test-utils) -- achieves exactly both goals.

## Drawing lines
> Thinking in design patterns is not about memorizing a lot of fancy names and diagrams. Knowing how to test is not really about learning a test runner or reading documentation.

From the guidelines to validate forms to seeing the [Strategy pattern](https://refactoring.guru/design-patterns/strategy) in practice, every concept and every API is presented as a declination of the same founding principle: **separation of concerns**.

According to the author, sensible separation -- **drawing the appropriate lines** between the parts of a system, whatever its complexity -- is the _line_ separating junior and senior developers. The more decoupled your applications, the easier to debug and scale.

In modern frontend, the major concerns needing careful separation are the same from _"one of the big ideas, or even the biggest idea"_ behind Vue and React: your **user interface** is a function of your **data**. Not only your UI should be deterministic and predictable: **rendering elements** on the user's screen and **managing the data** that power them are entirely different things, and must be dealt with -- _and tested_ -- separately.

## Think before you code, test so that you think
> Thinking in patterns, consider how data flows between different parts of a system and writing for testability starts before writing any code.

The book places a strong emphasis on testing and, of course, the rigor and confidence they promote that just make developers' life easier. However, even to the creator of Vue Test Utils **testing is, too, just another tool**: what really matters is _testability_.

**Writing for testability** forces you to think more _before_ and _after_ you write code, to ask more questions and, ultimately, **to apply systems thinking in your applications** (anyone else recognized [Donella Meadows' words](https://en.wikipedia.org/wiki/Thinking_In_Systems:_A_Primer) in the above quote?).

Tools such as [Testing Library](https://testing-library.com/docs/vue-testing-library/intro/) and [Cypress](https://www.cypress.io/) are shown in action not for their APIs but to promote a methodical, inquisitive approach: identify edge cases, error cases, the _"happy path"_, **ask yourself how you'll test all of them**, decide if you'll write the implementation before or after the tests.

## Real world value
Lachlan Miller is an experienced developer who's able to provide a real-world example behind every lesson he's learned.

Consider the problem of mocking HTTP requests: a popular approach consists of mocking the requests from the client, but could there be a better one? Did you know about Cypress' [intercept](https://docs.cypress.io/api/commands/intercept), or [Mock Service Worker](https://mswjs.io/)? What does it mean to _"mock the lowest dependency in the chain"_, why and how would you do that?

Did you know that [Headless UI](https://github.com/tailwindlabs/headlessui) from Tailwind achieved decoupling thanks to the [Renderless components pattern](https://www.patterns.dev/vue/renderless-components) that makes it easier to maintain both Vue and React versions?

If you're more experienced, have you ever [explored the actual JavaScript](https://play.vuejs.org/) your Vue SFCs get compiled to? Have you ever considered studying how Vue works under the hood... by taking a look at its tests? I, for instance, have started to do so, inspired by this book.


In the end, _Design Patterns for Vue.js_ has some value for most developers, even if they don't have to use Vue or don't need to improve their testing skills. If you want, you can purchase the book on [Gumroad](https://lachlanmiller.gumroad.com/l/vuejs-design-patterns).