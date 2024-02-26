+++
authors = ["Jun Wu"]
title = "Apollo GraphQL(NodeJS) Authentication&Authorization Sample - 2024"
date = "2024-02-25"
description = "a guide for Apollo GraphQL(NodeJS) Authentication&Authorization."
tags = [
    "nodejs",
    "typescript",
    "graphql",
    "authentication",
    "authorization"
]
categories = [
    "beckend",
    "middleware"
]
series = ["nodejs","typescript"]
+++

## What you will learn after reading this

1. start a Apollo GraphQL(NodeJS) server with typescript.
2. add auth guard to server shared context.

<!-- more -->

## Step by step to create a nodejs(typescript) proeject.

```shell
mkdir quick-start
cd quick-start
npm init --yes && npm pkg set type="module"
npm install --save-dev typescript @types/node nodemon
npm install @apollo/server graphql dotenv dataloader

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

```shell
touch nodemon.json
```

```json
// content in nodemon.json
{
    "watch": [
        "src",
        ".env"
    ],
    "ext": ".ts",
    "ignore": [
        "src/**/*.spec.ts"
    ],
    "exec": "npm start"
}
```

now all the frameworks thins are done, let's to add the first graphql code.


## Try to add `Article` and `UserDataSource` graphql service code

```shell
mkdir src
touch src/index.ts
```

add the following content to `src/index.ts`

```ts
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';
import { GraphQLError } from 'graphql';
import DataLoader from 'dataloader';

class UserDataSource {

    constructor() {
    }

    demoUsers = [
        { username: 'admin', id: 1, token: 'adpwd123', roles: ['admin', 'editor'] },
        { username: 'alice', id: 2, token: 'apwd123', roles: ['editor'] },
        { username: 'bob', id: 3, token: 'bpwd123', roles: ['reader'] },
    ];

    private batchUsers = new DataLoader(async (token: string[]) => {

        const productIdToProductMap = this.demoUsers.reduce((mapping, user) => {
            mapping[user.token] = user;
            return mapping;
        }, {});
        return token.map((token) => productIdToProductMap[token]);
    });

    async getUserFor(token) {
        return this.batchUsers.load(token);
    }
}

export const typeDefs = `#graphql
  type Article {
    title: String!
    id: Int!
    content: String!
    lastEditedBy: Int!
  }

  type Query {
    listArticles: [Article]!
  }

  type Mutation {
    editArticle(id: Int!, title: String, content: String): Article
  }
`;

export const resolvers = {
    Query: {
        listArticles: async (parent, args, { user, dataSources }, info) => {
            return dataSources.article.listArticles(user);
        },
    },
    Mutation: {
        editArticle: async (parent, args, { user, dataSources }, info) => {

            if (!user) throw new GraphQLError('401 Unauthorized', {
                extensions: {
                    code: 'Unauthorized',
                },
            });

            if (!user.roles.includes('admin') && !user.roles.includes('editor')) {
                throw new GraphQLError('403 Forbidden', {
                    extensions: {
                        code: 'Forbidden',
                    },
                });
            }

            return dataSources.article.editArticle({ id: args.id, title: args.title, content: args.content }, user);
        },
    }
}
```
### Authentication and Authorization

```ts
    Mutation: {
        editArticle: async (parent, args, { user, dataSources }, info) => {

            if (!user) throw new GraphQLError('401 Unauthorized', {
                extensions: {
                    code: 'Unauthorized',
                },
            });

            if (!user.roles.includes('admin') && !user.roles.includes('editor')) {
                throw new GraphQLError('403 Forbidden', {
                    extensions: {
                        code: 'Forbidden',
                    },
                });
            }

            return dataSources.article.editArticle({ id: args.id, title: args.title, content: args.content }, user);
        },
    }
```

- `Authentication`: you must log in and show the valid token to do some operations.
- `Authorization`: you must have permission to do some operations, for example, system admin can delete a user account but a normal user can not delete account.

the sample `Mutation` is a good example to show `Authentication` and `Authorization` both.

- if no user found in `Context`, it return 401.
- if user is not `editor` or `admin`,  it return 403, that means this user can not edit article.

### Add `ACL` and `ArticleDataSource` 

> here a advanced concept [Access-control list](https://en.wikipedia.org/wiki/Access-control_list)

```ts

class ACL {
    hasPermission = (item: { acl }, user: { roles }, operation: 'read' | 'write') => {
        if (!item.acl) return true;
        if (item.acl['*'][operation] == true) {
            return true;
        }
        if (!user.roles) return false;
        user.roles.forEach(role => {
            const roleKey = `role:${role}`;
            if (roleKey in item.acl && item.acl[`role:${role}`][operation] == true) return true;
        });

        return false;
    };

    hasRole = (user: { roles }, roles: []) => {
        return roles.some(item => user.roles.includes(item));
    }
}

const demoArticles = [
    {
        title: 'AAA', id: 1, content: 'aaa', lastEditedBy: 2,
        acl: {
            "*": { read: true },
            "role:editor": {
                read: true,
                write: true
            },
            "role:admin": {
                read: true,
                write: true
            },
        }
    },
    {
        title: 'BBB ', id: 2, content: 'bbb', lastEditedBy: 2,
        acl: {
            "*": { read: true },
            "role:editor": {
                read: true,
                write: true
            },
        }
    },
    {
        title: 'CCC', id: 3, content: 'ccc', lastEditedBy: 2,
        acl: {
            "*": { read: false },
            "role:admin": {
                read: true,
                write: true
            },
        }
    },
];

class ArticleDataSource {

    acl: ACL;
    constructor(acl: ACL) {
        this.acl = acl;
    }

    private batchArticles = new DataLoader(async (ids: number[]) => {

        const productIdToProductMap = demoArticles.reduce((mapping, article) => {
            mapping[article.id] = article;
            return mapping;
        }, {});
        return ids.map((id) => productIdToProductMap[id]);
    });

    async getArticleFor(id) {
        return this.batchArticles.load(id);
    }

    async listArticles(user: { roles }) {
        return demoArticles.filter(article => {
            return this.acl.hasPermission(article, user, 'read');
        });
    }

    async editArticle(article, user) {
        this.batchArticles.clear(article.id);

        const foundIndex = demoArticles.findIndex(x => x.id == article.id);
        if (foundIndex < 0) throw new GraphQLError('404 Not Found', {
            extensions: {
                code: 'Not Found',
            },
        })
        const foundItem = demoArticles[foundIndex];
        foundItem.lastEditedBy = user.id;
        foundItem.content = article.content;
        foundItem.title = article.title;

        this.batchArticles.prime(article.id, foundItem)

        return await this.getArticleFor(foundItem.id);
    }
}
```

### How ACL works

choose an article sample

```json
    {
        title: 'AAA', id: 1, content: 'aaa', lastEditedBy: 2,
        acl: {
            "*": { read: true },
            "role:editor": {
                read: true,
                write: true
            },
            "role:admin": {
                read: true,
                write: true
            },
        }
    },
```
focus on `acl` field

- `"*"` means public access permission, {"read":true} means anyone can read this article, even user is not logged in.
- `"role:editor"` is describing what permissions can users with "editor" have.`  { read: true },{ write: true }` means editors can read and write this article both.
- `"role:admin"` has same meanings with "role:editor".


so we can generate a helper class `ACL` to handle all the `Authorization` cases.

finish our `src/index.ts`

```ts
const server = new ApolloServer<MyContext>({
    typeDefs,
    resolvers,
});

const { url } = await startStandaloneServer(server, {

    // Note: This example uses the `req` argument to access headers,
    // but the arguments received by `context` vary by integration.
    // This means they vary for Express, Fastify, Lambda, etc.

    // For `startStandaloneServer`, the `req` and `res` objects are
    // `http.IncomingMessage` and `http.ServerResponse` types.
    context: async ({ req, res }) => {
        // Get the user token from the headers.
        const token = req.headers.authorization || '';

        const userDataSource = new UserDataSource();
        const acl = new ACL();
        //Try to retrieve a user with the token
        const user = await userDataSource.getUserFor(token);
        // Add the user to the context
        return {
            user,
            dataSources: {
                // Create a new instance of our data source for every request!
                // (We pass in the database connection because we don't need
                // a new connection for every request.)
                user: userDataSource,
                acl: acl,
                article: new ArticleDataSource(acl),
            },
        };
    },
});
```
### Run it to verify

```shell
npm run dev
```

### Edit article with editor token

choose `{ username: 'alice', id: 2, token: 'apwd123', roles: ['editor'] }` to edit article.


![mutation with edited content](/images/mutation-with-edited-content.png)
![mutation with Authorization header](/images/mutation-with-Authorization-header.png)

### Query articles with reader token

choose `{ username: 'bob', id: 3, token: 'bpwd123', roles: ['reader'] }` to query articles.

![query with reader token](/images/query-with-reader-token.png)

> Why return 2 articles but there are 3 articles?

Let check the `article.id=3`

```json
    {
        title: 'CCC', id: 3, content: 'ccc', lastEditedBy: 2,
        acl: {
            "*": { read: false },
            "role:admin": {
                read: true,
                write: true
            },
        }
    },
```

- `"*": { read: false }` means public access is off.
- only `role:admin` has the `read*write` permissions both, but current user is a `reader`, so `article.id=3` will not return to current user.

## Sample Source Code

[Star me @GitHub](https://github.com/wujun4code/apollo-graphql-auth-typescript-sample-2024).


## Further Readings

- [keycloak-connect-graphql](https://github.com/dbateman/keycloak-connect-graphql)
- [Authentication and authorization@apollographql.com](https://www.apollographql.com/docs/apollo-server/security/authentication/)
- [Authorization in the Apollo Router](https://www.apollographql.com/docs/router/configuration/authorization)