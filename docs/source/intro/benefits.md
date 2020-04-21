---
title: Why GraphQL?
description: GraphQL and Apollo help you ship features faster
---

Managing data in modern applications is challenging. Most applications require:

* Distinct front-end clients for different platforms (web, iOS, etc.), each with different data requirements
* A backend that serves data to clients from multiple sources (Postgres, Redis, etc.)
* Complex state and cache management for both the frontend and the backend

By adopting GraphQL and Apollo, you can reduce these challenges considerably. GraphQL's declarative model helps you create a consistent, predictable API you can use across all of your clients. As you add, remove, or migrate back-end data stores, that API doesn't change from the client's perspective.

Even with many other advantages, **GraphQL's single greatest benefit is the developer experience it provides.** It's straightforward to add new types and fields to your GraphQL API, and similarly straightforward for your clients to begin using those fields. This helps you design, develop, and deploy features faster.

Combined with [the Apollo platform](./platform/), complex considerations like caching, data normalization, and optimistic UI become straightforward as well.

## Declarative and efficient data fetching

GraphQL's declarative approach to data fetching provides significant performance and quality-of-life improvements over REST.

Consider a webpage that displays a list of adorable pets that are available for adoption at local animal shelters. 🐶

**Using REST**, the page might need to:

1. `GET` a list of shelters and corresponding `pet_id`s from the `/api/shelters` endpoint
2. `GET` each individual pet's details (name, photo URL, etc.) with a separate request to `/api/pets/PET_ID`

This solution requires multiple dependent network requests, which can result in slower page loads and additional battery consumption on mobile devices. This logic is also difficult to reuse on other pages that display slightly different data.

**Using GraphQL**, the page can obtain _all_ of this data with a single query to a single endpoint. That query looks like this:

```graphql
query GetPetsByShelter {
  shelters {
    name
    pets {
      name
      photoURL
    }
  }
}
```

This query describes the shape of the data we want to receive from the GraphQL server. The server takes care of combining and filtering back-end data to return _exactly_ what we ask for. This keeps payload sizes small, especially compared to a REST endpoint that might return hundreds of unnecessary fields.

To execute a query like the one above, the page uses a GraphQL client such as [Apollo Client](https://www.apollographql.com/docs/react/), with code that resembles the following (in the case of a React app):

```jsx:title=mainpage.jsx
// Define the query
const GET_PETS_BY_SHELTER = gql`
  query GetPetsByShelter {
    shelters {
      name
      pets {
        name
        photoURL
      }
    }
  }
`;

function MainPage() {

  // Execute the query within the component that uses its result
  const { loading, error, data } = useQuery(GET_PETS_BY_SHELTER);

  if (error) return <Error />;
  if (loading || !data) return <Fetching />;

  // Populate the component using the query's result
  return <PetList shelters={data.shelters} />;
}
```

Apollo Client's `useQuery` hook takes care of the request lifecycle from start to finish, including tracking loading and error states for you. There’s no middleware to set up or boilerplate to write before making your first request. All you need to do is describe the data your component needs and let Apollo Client do the heavy lifting. 💪

## Powerful tooling

Thanks to GraphQL's [strong typing](https://graphql.org/learn/schema) and built-in support for  [introspection](https://graphql.org/learn/introspection/), developer tools for GraphQL are extremely powerful. These tools let you do things like:

* Explore the full structure of a schema, complete with docstrings
* Compose new queries with live validation and autocomplete
* Register your schema with a management service that tracks and validates changes

### GraphQL Playground

[GraphQL Playground](https://github.com/prismagraphql/graphql-playground) by Prisma is an IDE for exploring a GraphQL schema and writing queries against it. At a glance, you can see all of the data available in your API, without needing to dive into back-end code or know which data stores are being used:

![GraphQL Playground](../assets/graphql-playground.png)

[Apollo Server](https://www.apollographql.com/docs/apollo-server) includes GraphQL Playground out of the box, so as soon as you start up a server on your local machine, you can start exploring your schema and composing queries.

### Apollo Graph Manager

[Apollo Graph Manager](https://engine.apollographql.com/login) is the only tool in the GraphQL ecosystem that can provide monitoring and analytics for your API. Graph Manager displays per-resolver tracing metrics that help you pinpoint bugs, as well as performance distribution for every field in your schema. You can also pipe this data to services you're probably already using like DataDog, and set up Slack alerts if these numbers pass a certain threshold.

![Apollo Graph Manager](../assets/engine.png)

### Apollo Client DevTools

Apollo DevTools is a Chrome extension that allows you to inspect your Apollo Client cache, track active queries, and view mutations. You also have access to GraphiQL within Apollo DevTools which is convenient for testing queries as you're working on front-end code with Apollo Client.

![Apollo DevTools](../assets/dev-tools.png)

## Production readiness

GraphQL's adoption has risen steadily ever since Facebook published the original [specification](https://spec.graphql.org/) in 2015. For more and more organizations, the benefits of GraphQL are taking it from an enthusiastic engineer's hack-week experiment to the heart of the application data layer.

At Apollo, we've received extensive feedback, contributions, and support from enterprise customers who have had both great confidence and great success in their transition to GraphQL. 

## Trade-offs

When deciding whether to adopt a technology, it's just as important to understand the technology's limitations as it is to understand its benefits.

Consider the following trade-offs when using GraphQL:

### Change management

GraphQL introduces a new conceptual model for representing and interacting with data. An organization that's comfortable with this model can design, implement, and ship features quickly. However, the process of _becoming_ comfortable with this model takes time.

* Front-end developers must come up to speed with a new API for fetching and manipulating data. 
* Back-end developers must come up to speed with how to handle incoming requests from the frontend.
* Developers across the organization must collaborate on a single, product-driven GraphQL schema and appoint individuals as the official maintainers of that schema.

The Apollo [docs](https://www.apollographql.com/docs/), [blog](https://www.apollographql.com/blog/), and [Spectrum community](https://spectrum.chat/apollo) are all here to help your organization master GraphQL and take full advantage of it.

### Potential for slow operations

Your [GraphQL schema](https://www.apollographql.com/docs/apollo-server/schema/schema/) defines which types and fields your clients can query. Your GraphQL server's [resolvers](https://www.apollographql.com/docs/apollo-server/data/resolvers/) define how those types and fields are populated from your data stores.

Depending on your schema and your resolver definitions, your server might inadvertently support GraphQL operations that execute very slowly, or even max out your server's resources.

Consequently, it's important to design your schema such that it supports the operations your clients need, _without_ supporting unnecessary operations that affect performance. It's also helpful to set up [trace reporting](https://www.apollographql.com/docs/graph-manager/setup-analytics/) for your GraphQL server, enabling you to identify and improve slow operations.

[Learn about query-driven schema design](https://www.apollographql.com/docs/apollo-server/schema/schema/#query-driven-schema-design)

### Incompatibility with web browser caching

Although [Apollo Client](https://www.apollographql.com/docs/react/) provides useful client-side [caching features](https://www.apollographql.com/docs/react/caching/cache-configuration/), the automatic caching provided by your _web browser_ does not interact well with GraphQL.

Web browsers cache fetched data according to its URL. With GraphQL, you fetch all data from the _same_ URL (the URL of your GraphQL server). Consequently, you can't rely on the cached value for this URL.
