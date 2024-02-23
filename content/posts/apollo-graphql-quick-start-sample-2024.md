+++
authors = ["Jun Wu"]
title = "Apollo GraphQL(NodeJS) Quick Start Sample - 2024"
date = "2024-02-23"
description = "a guide for Apollo GraphQL(NodeJS)."
tags = [
    "nodejs",
    "typescript",
    "graphql",
    "api",
]
categories = [
    "beckend",
    "middleware"
]
series = ["nodejs","typescript"]
+++

## What you will learn after reading this

1. start a nodejs server with typescript.
2. add cache control, rest datasource sample to your project.
3. docker package your nodejs project.

<!-- more -->

## Step by step to create a nodejs(typescript) proeject.

```shell
mkdir quick-start
cd quick-start
npm init --yes && npm pkg set type="module"
npm install --save-dev typescript @types/node
npm install @apollo/server graphql

touch tsconfig.json
```

paste the following content to `tsconfig.json`

```json
{
  "compilerOptions": {
    "rootDirs": ["src"],
    "outDir": "dist",
    "lib": ["es2020"],
    "target": "es2020",
    "module": "esnext",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "types": ["node"]
  }
}
```

now all the frameworks thins are done, let's to add the fisrt graphql code.

### create index.ts to serve a web api

```shell
mkdir src
touch src/hello-world.ts
```

add the following code to `src/hello-world.ts`

```typescript
export const typeDefs = `#graphql
  type Demo {
    foo: String!
    bar: Int!
  }

  type Query {
    demos: [Demo]!
  }
`;

const demos = [
    { foo: "hello world!", bar: 1 },
    { foo: "hello apollo!", bar: 2 },
    { foo: "hello graphql!", bar: 3 },
];

export const resolvers = {
    Query: {
        demos: () => demos,
    },
}
```

then add the following code to `index.ts`

```typescript
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';

import { typeDefs, resolvers } from './hello-world.js';

const server = new ApolloServer({
    typeDefs,
    resolvers,
});

const { url } = await startStandaloneServer(server, {
    listen: { port: 4000 },
});

console.log(`🚀  Server ready at: ${url}`);
```

one more step, tell `package.json` how to start the server, edit your `package.json` with the following content

```json
{
  // ...etc.
  "scripts": {
    "compile": "tsc",
    "start": "npm run compile && node ./dist/index.js"
  },
  // other content
}
```

ok, let try our fisrt grapgql query. open `localhost:4000` in your browser.

put the following graphql sentence to `Operation` window

```graphql
query Demos {
  demos {
    foo
    bar
  }
}
```

then clieck the blue button right to the `Operation` window. 

the check the response at the right panel of browser.


![try graphql](/images/apollo-graphql-quick-start-sample-2024-1.png)

Congratulations. your fist milestone is done well.

## Wrap the upstream datasource

> let talk a new topic: your legacy beckend service is an REST api web service, we can set graphql as a datasource angent, it can provide an unify entrance for clients(web/iOS/Android/Flutter, etc), 
> that means we should call REST api in graphql server, then re-group/re-org the response to clients. graphql can play a middleware role to handle multiple datasources.

{{<mermaid>}}
sequenceDiagram
    Clients->>+GraphQL: Hello GraphQL, give something
    GraphQL->>+DataSource: Hi Upstream, show me the data.
    DataSource-->>-GraphQL: Hi GraphQL, here is the data
    GraphQL-->>-Clients: Hey Clients, here some data

{{</mermaid>}}

we can imagine that, upstream datasource can be REST apis, MySQL, Redis or s3, clients side will keep GraphQL as the only unify contracts.


now let try it.

### Wrap REST api as the upstream datasource

here is free REST api sample -> https://reqres.in/.

one case from it -> https://reqres.in/api/users?page=2


now go back to `index.ts`, remove the imports from `hello-world.js` 

~~import { typeDefs, resolvers } from './hello-world.js';~~

and remove the refers

const server = new ApolloServer({
    ~~typeDefs,~~
    ~~resolvers,~~
});

it should looks like this

```ts
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';

const server = new ApolloServer({
});

const { url } = await startStandaloneServer(server, {
    listen: { port: 4000 },
});

console.log(`🚀  Server ready at: ${url}`);
```

```shell
npm install @apollo/datasource-rest
touch src/reqres.ts
```

then we are going to create a new graphql service, edit `src/reqres.ts`, past the whole content to it

```ts
import { RESTDataSource } from '@apollo/datasource-rest';

export class ReqresDataSource extends RESTDataSource {
    override baseURL = 'https://reqres.in';

    listUsers = async (page: number) => {
        if (!page) page = 1;
        const response = await this.get(`/api/users`, {
            params: {
                page: page.toString(),
            },
        });
        return response.data;
    }
}

export interface ContextValue {
    dataSources: {
        reqres: ReqresDataSource;
    };
}

export const typeDefs = `#graphql
  type User {
    id: Int!
    email: String!
    first_name: String!
    last_name: String!
    avatar: String!
  }

  type Query {
    users(page: Int!): [User!]!
  }
`;


export const resolvers = {
    Query: {
        users: async (parent, args, { dataSources }, info) => {
            return dataSources.reqres.listUsers(args.page);
        },
    },
}
```

last step, edit `index.ts`, paste the following content to it

```ts
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';

import { typeDefs, resolvers, ContextValue, ReqresDataSource } from './reqres.js';

const server = new ApolloServer<ContextValue>({
    typeDefs,
    resolvers,
});

const { url } = await startStandaloneServer(server, {
    context: async () => {
        const { cache } = server;
        return {
            dataSources: {
                reqres: new ReqresDataSource({ cache }),
            },
        };
    },
    listen: { port: 4000 },
});

console.log(`🚀  Server ready at: ${url}`);
```

now let try it

```shell
npm start
```

here is the graphql query

```graphql
query Users($page: Int!) {
  users(page: $page) {
    id
    email
    first_name
    last_name
    avatar
  }
}
```

and do NOT forget add the `variables`

```json
{
  "page": 2
}
```
you can get some response with the following screen shot.

![try graphql](/images/apollo-graphql-quick-start-sample-2024-2.png)

## Add cache layer to make query faster

```shell
npm install @apollo/server-plugin-response-cache
```

edit `index.ts`

add three imports

```ts
import { InMemoryLRUCache } from '@apollo/utils.keyvaluecache';
import responseCachePlugin from '@apollo/server-plugin-response-cache';
import { ApolloServerPluginCacheControl } from '@apollo/server/plugin/cacheControl';
```

then edit the code for `server` in `index.ts`

```ts
const server = new ApolloServer<ContextValue>({
    typeDefs,
    resolvers,
    cache: new InMemoryLRUCache(),
    plugins: [ApolloServerPluginCacheControl({ defaultMaxAge: 30 }), responseCachePlugin()],
});
```

add the cache schema to `typeDefs`

```ts
  enum CacheControlScope {
    PUBLIC
    PRIVATE
  }

  directive @cacheControl(
    maxAge: Int
    scope: CacheControlScope
    inheritMaxAge: Boolean
  ) on FIELD_DEFINITION | OBJECT | INTERFACE | UNION
```

so the final typeDefs in `src/reqres.ts` should be 

```ts
export const typeDefs = `#graphql
  enum CacheControlScope {
    PUBLIC
    PRIVATE
  }

  directive @cacheControl(
    maxAge: Int
    scope: CacheControlScope
    inheritMaxAge: Boolean
  ) on FIELD_DEFINITION | OBJECT | INTERFACE | UNION

  type User {
    id: Int!
    email: String!
    first_name: String!
    last_name: String!
    avatar: String!
  }

  type Query {
    users(page: Int!): [User!]!
  }
`;
```

let try the cache

```shell
npm start
```

when you do the same query twice, you can check the response time for the second time, it should be less than 30ms, but the first time will cost more then 500ms.
That means the first time, it called the rest api, but for the second time, it reponded from cache. the cache time is *30* seconds. you can edit the config in this line

```ts
plugins: [ApolloServerPluginCacheControl({ defaultMaxAge: 30 }), responseCachePlugin()],
```

you can edit the `defaultMaxAge` to 60, try it.

## Run it in docker

```shell
touch Dockerfile
```

add the following content to `Dockerfile`

```dockerfile
# path: ./Dockerfile
FROM node:18-alpine
ARG NODE_ENV=development
ENV NODE_ENV=${NODE_ENV}

WORKDIR /opt/
COPY package.json package-lock.json ./
ENV PATH /opt/node_modules/.bin:$PATH

WORKDIR /opt/app
COPY . .
RUN chown -R node:node /opt/app
USER node
RUN npm install
EXPOSE 4000
CMD ["npm", "start"]
```

one small thing

```shell
touch .dockerignore
```

add the following to it

```dockerignore
node_modules
dist
Dockerfile
```

then run 

```shell
docker build -t graphql-demo .
```

when build completed, run it

```shell
docker images

REPOSITORY                                 TAG       IMAGE ID       CREATED              SIZE
graphql-demo                                    latest    bff8b356e7af   About a minute ago   231MB

docker run -p 4000:4000 bff8b356e7af
#or
docker run -p 4000:4000 graphql-demo:latest
```

you will see the logs

```shell
> apollo-graphql-quick-start-sample-typescript-2024@1.0.0 start
> npm run compile && node ./dist/index.js


> apollo-graphql-quick-start-sample-typescript-2024@1.0.0 compile
> tsc

🚀  Server ready at: http://localhost:4000/
```

Congratulations!  🤩🤩🤩


## Sample Source Code

[Star me @GitHub](https://github.com/wujun4code/apollo-graphql-quick-start-sample-typescript-2024).


Thanks for reading this.
