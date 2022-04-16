# GraphQL Cache Control

This package offers you a simple decorator to set cache control on your resolvers.

## Installation

On Yarn:

```shell
yarn add nestjs-gql-cache-control
```

On NPM:

```shell
npm install nestjs-gql-cache-control
```

## Usage

To use caching, you are gonna need these packages too:

```
@nestjs/graphql
apollo-server-core
apollo-server-plugin-response-cache
```

First, register graphql module and cache plugins in your app module:

```ts
import responseCachePlugin from 'apollo-server-plugin-response-cache';
import { ApolloServerPluginCacheControl } from 'apollo-server-core/dist/plugin/cacheControl';

GraphQLModule.forRoot({
  // ...
  plugins: [
    ApolloServerPluginCacheControl({ defaultMaxAge: 5 }), // optional
    responseCachePlugin(),
  ],
}),
```

> To add Redis or other caching stores, check [Apollo's docs](https://www.apollographql.com/docs/apollo-server/performance/caching/#in-memory-cache-setup)

Then, you can use the decorator on your queries and field resolvers:

```ts
import { CacheControl } from 'nestjs-gql-cache-control';

@Resolver((type) => Post)
export class PostResolver {
  @Query(() => [Post])
  @CacheControl({ maxAge: 10 })
  posts() {
    // database calls
    return posts;
  }

  @ResolveField(() => User)
  @CacheControl({ inheritMaxAge: true })
  owner() {
    // database calls
    return owner;
  }

  @ResolveField(() => boolean)
  @CacheControl({ maxAge: 5, scope: 'PRIVATE' })
  hasLiked() {
    // database calls
    return hasLiked;
  }
}
```

## How The Apollo Cache Works

Please carefully read [Apollo's docs about caching](https://www.apollographql.com/docs/apollo-server/performance/caching/) to understand how caching works, since it has a set of rules for cache calculation. In a brief:

> a response should only be considered cacheable if every part of that response opts in to being cacheable. At the same time, we don't think developers should have to specify cache hints for every single field in their schema.
> So, we follow these heuristics:
> Root field resolvers are extremely likely to fetch data (because these fields have no parent), so we set their default maxAge to 0 to avoid automatically caching data that shouldn't be cached.
> Resolvers for other non-scalar fields (objects, interfaces, and unions) also commonly fetch data because they contain arbitrarily many fields. Consequently, we also set their default maxAge to 0.
> Resolvers for scalar, non-root fields rarely fetch data and instead usually populate data via the parent argument. Consequently, these fields inherit their default maxAge from their parent to reduce schema clutter.

## Connections (Pagination)

If you happen to use [nestjs-gql-connections](https://github.com/overnested/nestjs-gql-connections), `edges` and `node` will automatically inherit cache control from their parents. but otherwise you should set `inheritMaxAge` on your connection fields to prevent connections from cancelling your cache.

Why you should do that? because you probably don't want your connections to cancel your cache control. ([learn more](https://www.apollographql.com/docs/apollo-server/performance/caching/#default-maxage))
