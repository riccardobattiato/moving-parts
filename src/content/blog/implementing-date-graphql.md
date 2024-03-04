---
pubDatetime: 2024-03-03T17:55:00.677Z
title: Implementing Date type in GraphQL
featured: false
draft: false
tags:
  - development
  - graphql
  - mongodb
  - backend
description: By default, GraphQL doesn't come with a way to work with dates and time. What you can do, however, is to define a custom type representing a ISO DateTime value.
---

## Table of contents

## Scalar types in GraphQL
[GraphQL](https://graphql.org/) is a powerful query language which comes with a full-fledged [type system](https://graphql.org/learn/schema/#type-system), enforcing the precision that distinguishes it from REST and improves on [some of its problems](https://nordicapis.com/what-are-over-fetching-and-under-fetching/). **However, GraphQL doesn't natively support date types.** [Scalar types](https://graphql.org/learn/schema/#scalar-types) -- which we can think of as the primitive types of most programming languages -- only support numbers, strings, booleans and a special ID type.

Instead, developers are invited to define their own **custom scalar types** in order to manage their implementation as they wish. Let's suppose you want to add a simple `Date` type to your schema that works in a similar way to [JavaScript's Date object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) -- which represents a single moment in time. What you'd need to do is to:

1. **Define** the custom type in your GraphQL schema;
2. Tell your GraphQL server how to **resolve** values of the given type.

## Date type implementation
Defining a custom scalar type in GraphQL Schema Language is straightforward:

```graphql
scalar Date
```

From this moment on, you may use the type inside other definitions:

```graphql
scalar Date

type Post {
  title: String!
  description: String
  publishedAt: Date # is now valid
}
```
As for the type's **actual resolution**, the needed steps may vary according to the GraphQL's own implementation you're using, but a value's resolution mainly consists of:

- **Parsing** the value received by the client in order to work with it as intended in your server [(deserialization)](https://developer.mozilla.org/en-US/docs/Glossary/Deserialization)
- **Serializing** the data from a type or structure suitable to your server-side application in order to send it to the client [(also see MDN)](https://developer.mozilla.org/en-US/docs/Glossary/Serialization).

### Working example
For a real example, let's consider [Apollo](https://www.apollographql.com/) as the GraphQL service, a [node.js](https://nodejs.org/en) server-side app, and a [MongoDB](https://www.mongodb.com/en-us) database able to directly work with JavaScript's native `Date` type. The serialization is done in JSON format.

We could do something like this:

```typescript
import { GraphQLScalarType } from "graphql";
import { Kind } from "graphql/language";

const resolvers = {
  // ...other resolvers
  Date: new GraphQLScalarType({
    name: "Date",
    description: "A ISO 8601 compliant datetime value",
    parseValue(value) {
      return new Date(value); // from incoming JSON string to node.js Date object
    },
    parseLiteral(ast) { // abstract syntax tree of query string, see explanation below
      if (ast.kind !== Kind.STRING) {
        return null; // should be an error, handle as you please
      }
      const { value } = ast;
      return new Date(value);
    },
    serialize(value) {
      return value.toISOString(); // from node.js Date object to outgoing JSON string
    },
  }),
};

```
To favor readability, this implementation works with [ISO](https://en.wikipedia.org/wiki/ISO_8601) strings rather than integer values counting from [Unix epoch](https://developer.mozilla.org/en-US/docs/Glossary/Unix_time) as some examples online do instead. This is subject to you or your team's preference, as the only thing that matters is the conversion of the value between one suitable to the serialization data format you're using -- in this case JSON -- and the application's "working" representation. For a node.js app ultimately interacting with MongoDB, using JavaScript's own `Date` type as working format is a valid choice.

You may have noticed that, in the above code snippet, there are *two* functions dealing with parsing: `parseValue` and `parseLiteral`. This is because Apollo calls parse *value* when resolving the custom scalar from an **argument variable**; parse *literal* is called when dealing with a value **hard-coded directly into a query** -- quite common if your app doesn't rely on GraphQL variables and builds queries and mutations internally. See [Apollo Docs](https://www.apollographql.com/docs/apollo-server/schema/custom-scalars/#example-the-date-scalar) for more info.

For an advanced, complete implementation of Date, DateTime etc. see [graphql-scalars](https://github.com/Urigo/graphql-scalars/tree/master/src/scalars/iso-date) by [The Guild](https://the-guild.dev/), which provides advanced parsing and error handling.