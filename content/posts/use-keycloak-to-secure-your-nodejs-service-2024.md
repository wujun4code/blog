+++
authors = ["Jun Wu"]
title = "Use Keycloak to secure your NodeJS service - 2024"
date = "2024-02-24"
description = "a guide for Keycloak&NodeJS."
tags = [
    "nodejs",
    "typescript",
    "keycloak",
    "authentication",
]
categories = [
    "beckend",
    "middleware"
]
series = ["keycloak","nodejs"]
+++


## What you will learn after reading this

1. start a nodejs server with express.
2. docker run a Keycloak locally.
2. add a authenticated api endpoint powered by Keycloak.


<!-- more -->

## Step by step to create a nodejs(typescript) proeject.

```shell
mkdir express-keycloak-typescript-quick-start-2024
cd express-keycloak-typescript-quick-start-2024
npm init --yes && npm pkg set type="module"
npm install --save-dev typescript @types/node @types/express nodemon
npm install express
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

add the following content to it:

```json
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

add some commands to `scripts` part in `package.json`

```json
{
  // ...etc.
  "scripts": {
    "compile": "tsc",
    "dev": "nodemon",
    "start": "npm run compile && node ./dist/index.js"
  },
  // other content
}
```

now let create `index.ts` the main entrance of our express web server


```shell
touch src/index.ts
```

append content to it

```ts
import express, { Request, Response } from 'express';
const app = express();

// routes
app.get('/', (req: Request, res: Response) => {
    res.send('Hello World!');
});

// some more stuff

const APP_PORT = 3000;

app.listen(APP_PORT, () => {
    console.log(`🍗 Server started on port ${APP_PORT}`);
});

// end
```

try to run it

```shell
npm run dev
```


```shell
npm run dev

> express-keycloak-typescript-quick-start-2024@1.0.0 dev
> nodemon

[nodemon] 3.1.0
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): src/**/*
[nodemon] watching extensions: ts
[nodemon] starting `npm start`

> express-keycloak-typescript-quick-start-2024@1.0.0 start
> npm run compile && node ./dist/index.js


> express-keycloak-typescript-quick-start-2024@1.0.0 compile
> tsc

🍗 Server started on port 3000
```

### Docker run Keycloak locally

> detailed guide can be found in [keycloak - getting-started-docker](https://www.keycloak.org/getting-started/getting-started-docker)

```shell
docker run -p 8080:8080 -d -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:23.0.7 start-dev
```

if you want to keep the config even stop the Keycloak container, please use a persistent storage linked. try the following docker compose file

```yml
version: '3'

services:
  postgres:
    image: postgres
    container_name: keycloak-postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: dbpw1234
    ports:
      - '5432:5432'
    networks:
      - keycloak

  keycloak:
    image: quay.io/keycloak/keycloak:latest
    container_name: keycloak
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: dbpw1234
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin1234
    command: start-dev
      # Uncomment the line below if you want to specify JDBC parameters. The parameter below is just an example, and it shouldn't be used in production without knowledge. It is highly recommended that you read the PostgreSQL JDBC driver documentation in order to use it.
      #JDBC_PARAMS: "ssl=true"
    ports:
      - 8080:8080
    depends_on:
      - postgres
    networks:
      - keycloak

networks:
  keycloak:
    name: keycloak
    driver: bridge
volumes:
  postgres_data:
    driver: local

```

### Set and config Keycloak

1. create a new Realm, you can You can think of it as a concept similar to a company/team.
2. create two Roles, `admin` and `user`.
3. ceeate two Users, `admin` is a user with `admin` role, but `foo` is a user with `user` role.


#### create a new Realm named `express-demo`

![create new realm](/images/keycloak-create-realm.png)


#### create a new client named `dev-auth` in Realm `express-demo`

just input client name with `dev-auth` only, all the left configs can been ignored for current demo.

![create new client](/images/keycloak-create-client-in-realm.png)

#### create two new Role named `admin` and `user` in Realm `express-demo`

![create two roles](/images/create-two-roles.png)

#### Do NOT forget to set passwords for these two users

![create passwords](/images/create-passwords.png)

try to set password like this, demo only, do NOT set this type of password in your production.

- user: foo 
- password: foo123

- user: admin
- passowrd: admin123

all things is Keycloak side are done.

### Set Keycloak middleware for Express

install related packages

```shell
npm i cors dotenv jsonwebtoken keycloak-connect 
npm install --save-dev @types/cors
```

edit your `index.ts`

```ts
import 'dotenv/config'
import express, { Request, Response } from 'express';
import cors from 'cors';

const app = express();

app.use(express.json());
app.use(cors());

// routes
app.get('/', (req: Request, res: Response) => {
    res.send('Hello World!');
});

app.get('/secured', (req: Request, res: Response) => {
    res.json({
        message: "secured connection established.",
        status: "success"
    }).status(200);
});

app.get('/admin', (req: Request, res: Response) => {
    res.json({
        message: "admin connection established.",
        status: "success"
    }).status(200);
});

// some more stuff

const APP_PORT = 3000;

app.listen(APP_PORT, () => {
    console.log(`🍗 Server started on port ${APP_PORT}`);
});

// end
```

try the new router

```shell
npm run dev
curl -X GET localhost:3000/secured
curl -X GET localhost:3000/admin
```

you will get the sample result

```json
{"message":"secured connection established.","status":"success"}
{"message":"admin connection established.","status":"success"}
```

as we can see, these two endpoints can be accessed without authentication currently, so let's set two features

1. the `secured` can be accessed when `Bearer {access token}` in headers, and the access token should be a valid token with a role name `User`.
2. but `admin` can be accessd only when access token belongs to a user in `admin` role.

edit your `index.ts` to this sample

```ts
import 'dotenv/config'
import express, { Request, Response } from 'express';
import Keycloak from 'keycloak-connect';
import cors from 'cors';

const app = express();

const keycloakConfig = {
    "confidential-port": 0,
    "realm": process.env.KEYCLOAK_REALM,
    "auth-server-url": `${process.env.KEYCLOAK_URL}`,
    "ssl-required": "external",
    "resource": process.env.KEYCLOAK_CLIENT,
    "bearer-only": true
}

const keycloak = new Keycloak({}, keycloakConfig);
app.use(keycloak.middleware());
app.use(express.json());
app.use(cors());

// routes
app.get('/', (req: Request, res: Response) => {
    res.send('Hello World!');
});

app.get('/secured', keycloak.protect('realm:user'), (req: Request, res: Response) => {
    res.json({
        message: "secured connection established.",
        status: "success"
    }).status(200);
});

app.get('/admin', keycloak.protect('realm:admin'), (req: Request, res: Response) => {
    res.json({
        message: "admin connection established.",
        status: "success"
    }).status(200);
});

// some more stuff

const APP_PORT = 3000;

app.listen(APP_PORT, () => {
    console.log(`🍗 Server started on port ${APP_PORT}`);
});

// end
```

and add file named `.env` in root directory

```.env
KEYCLOAK_REALM=express-demo
KEYCLOAK_CLIENT=dev-auth
KEYCLOAK_URL=http://localhost:8080
```

then you can try to call these two endpoints

```shell
npm run dev
curl -I -X GET localhost:3000/secured
curl -I -X GET localhost:3000/admin
```

you will get 

```shell
HTTP/1.1 403 Forbidden
Content-Length: 13
Access-Control-Allow-Origin: *
Connection: keep-alive
Date: Sat, 24 Feb 2024 10:56:49 GMT
Keep-Alive: timeout=4
Proxy-Connection: keep-alive
X-Powered-By: Express
```

nice, this means client can not access these two endpoints without valid token.


### Get token from Keycloak

```shell
curl --location 'http://localhost:8080/realms/express-demo/protocol/openid-connect/token' \
--header 'content-type: application/x-www-form-urlencoded' \
--data-urlencode 'client_id=dev-auth' \
--data-urlencode 'username=foo' \
--data-urlencode 'password=foo123' \
--data-urlencode 'grant_type=password'
```

please copy the value with key `access_token` from the json response.

try to access `secured`

```shell
curl --location http://localhost:3000/secured \
  --header 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJ4MjBmcUEzMkN0ckpHdWtJQjVVMVloQ0Rvb2thYVh0bElFRGh0QndqNnNzIn0.eyJleHAiOjE3MDg4MzI0NDIsImlhdCI6MTcwODgzMDY0MiwianRpIjoiNWY5Nzk5ZmMtZTJlZC00ZTNkLWJiNjMtYmUxMTA2ZjJjNmM2IiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL3JlYWxtcy9leHByZXNzLWRlbW8iLCJhdWQiOiJhY2NvdW50Iiwic3ViIjoiMDhmY2JlNmQtZGJhNC00NDg3LTljMzAtOGYzODdkOGQ4M2VjIiwidHlwIjoiQmVhcmVyIiwiYXpwIjoiZGV2LWF1dGgiLCJzZXNzaW9uX3N0YXRlIjoiMjMxNWQzMzYtMzdjMi00NmQ2LWJhOWQtNjczZDQ2ZWQwZjI4IiwiYWNyIjoiMSIsImFsbG93ZWQtb3JpZ2lucyI6WyIvKiJdLCJyZWFsbV9hY2Nlc3MiOnsicm9sZXMiOlsiZGVmYXVsdC1yb2xlcy1leHByZXNzLWRlbW8iLCJvZmZsaW5lX2FjY2VzcyIsInVtYV9hdXRob3JpemF0aW9uIiwidXNlciJdfSwicmVzb3VyY2VfYWNjZXNzIjp7ImFjY291bnQiOnsicm9sZXMiOlsibWFuYWdlLWFjY291bnQiLCJtYW5hZ2UtYWNjb3VudC1saW5rcyIsInZpZXctcHJvZmlsZSJdfX0sInNjb3BlIjoicHJvZmlsZSBlbWFpbCIsInNpZCI6IjIzMTVkMzM2LTM3YzItNDZkNi1iYTlkLTY3M2Q0NmVkMGYyOCIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwibmFtZSI6IkZvbyBCYXIiLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJmb28iLCJnaXZlbl9uYW1lIjoiRm9vIiwiZmFtaWx5X25hbWUiOiJCYXIifQ.A8OytfZkogcWJXoU7rVzQPNLw08rBMYlcT-BO_77_ogA3IAMxUE0lRTVR0gbYrFgaXnfzqbjMYm34sGIaQtJ6IMyRFGO1uNtjbXkaTX0eVnijk_eHoJJg3B8FK5Yaq-mTJwejJhQEcl7TOp24tjVNZGZS2wm6Cq7KGpjrXdaRxgOn4SID4gUWqkkVW7OasPh6GBjyEvv4sUK0H00NCjiJHSo1rgBA4khpIUkzCpceuIw7p1HGrIdKhLnOmw8LNgo0U9AbxD4aVFTR1J2WftBl4m8yDaNoPS3WuB48qlzohlL21xDXkC-pFMvaDJRem1ScJSA6d0dTJWw0N61YZ5ekw'
```

you will see

```json
{
    "message": "secured connection established.",
    "status": "success"
}
```

but when you try to access `admin` with the same token, you will get a 403

```shell
curl -I --location http://localhost:3000/admin \
  --header 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJFbFlqelNKcENaa0RaNVl6Q243dF9lU3FSc1oxblZCRFc5NjVLYkhyOFJrIn0.eyJleHAiOjE3MDg3NzM0MzYsImlhdCI6MTcwODc3MzEzNiwianRpIjoiZDMyOGU3MWYtZmViZi00YWIwLWIxMGEtNGY4MTY5OWU2MzE3IiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL3JlYWxtcy9leHByZXNzLWRlbW8iLCJhdWQiOiJhY2NvdW50Iiwic3ViIjoiZmYyYjJiMzYtMzU4ZS00OTcwLWFlZjgtYmI2ZjNmMjEzOTQ4IiwidHlwIjoiQmVhcmVyIiwiYXpwIjoiZGV2LWF1dGgiLCJzZXNzaW9uX3N0YXRlIjoiYjgxNDY4MjEtNGJkZi00ZTg1LTgzNmQtOWNlYWIxZTI3Zjk3IiwiYWNyIjoiMSIsImFsbG93ZWQtb3JpZ2lucyI6WyIvKiJdLCJyZWFsbV9hY2Nlc3MiOnsicm9sZXMiOlsib2ZmbGluZV9hY2Nlc3MiLCJkZWZhdWx0LXJvbGVzLWV4cHJlc3MtZGVtbyIsInVtYV9hdXRob3JpemF0aW9uIiwidXNlciJdfSwicmVzb3VyY2VfYWNjZXNzIjp7ImFjY291bnQiOnsicm9sZXMiOlsibWFuYWdlLWFjY291bnQiLCJtYW5hZ2UtYWNjb3VudC1saW5rcyIsInZpZXctcHJvZmlsZSJdfX0sInNjb3BlIjoicHJvZmlsZSBlbWFpbCIsInNpZCI6ImI4MTQ2ODIxLTRiZGYtNGU4NS04MzZkLTljZWFiMWUyN2Y5NyIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwibmFtZSI6IkZvbyIsInByZWZlcnJlZF91c2VybmFtZSI6ImZvbyIsImdpdmVuX25hbWUiOiJGb28ifQ.p7dfx-LZsPVRokiAarMx_nt-jnFbyWQlf8EnzX9SwjydL840ePG1DVfl4i9yIoaLcC8OyonW5F1RmAETW_HvR9jHLG_EpGZJHFj6K-zdohtsLsCinJ-fm2s07sQo8H9DxlWufEE7SkzKkbrIhZLzQLQ27Hi0UT-Qy7AN0cOqHcGP49xrEqr1KsB3EjW9zLjfwrk2cSpVDLsegEKerhJdEVnsIxVqxK9GD0Cv3PCcNqFOJlaK6GBm1uiWyDXKH5YUZ8MkwQ8qcgl727UiFZP_vKM8dFxTTCwGAcPQAPiL23sM3Ij4k_5wJFKtKYyn_QX2p27OYgP9ZPNL3K7KIAqSWw'
```

```shell
HTTP/1.1 403 Forbidden
Connection: close
Access-Control-Allow-Origin: *
Connection: keep-alive
Date: Sat, 24 Feb 2024 11:15:14 GMT
Keep-Alive: timeout=4
Proxy-Connection: keep-alive
X-Powered-By: Express
```

now you can switch to admin user

```shell
curl --location 'http://localhost:8080/realms/express-demo/protocol/openid-connect/token' \
--header 'content-type: application/x-www-form-urlencoded' \
--data-urlencode 'client_id=dev-auth' \
--data-urlencode 'username=admin' \
--data-urlencode 'password=admin123' \
--data-urlencode 'grant_type=password'
```

try to call `admin`

```shell
curl -I --location http://localhost:3000/admin \
  --header 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJFbFlqelNKcENaa0RaNVl6Q243dF9lU3FSc1oxblZCRFc5NjVLYkhyOFJrIn0.eyJleHAiOjE3MDg3NzM2NjUsImlhdCI6MTcwODc3MzM2NSwianRpIjoiMzlkNzdlZmYtODEyNC00ZTAxLWI0NDktZTAzMmE3OWUwMGNjIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL3JlYWxtcy9leHByZXNzLWRlbW8iLCJhdWQiOiJhY2NvdW50Iiwic3ViIjoiZjE3MjJhNTYtYzgyNC00MDc3LTg1ZDItMjk5MTcxMDdiNjhmIiwidHlwIjoiQmVhcmVyIiwiYXpwIjoiZGV2LWF1dGgiLCJzZXNzaW9uX3N0YXRlIjoiY2Q5Njk1OTEtNjdmMy00YjE0LWI1ZjktNTE5MDFiMWIxMjk5IiwiYWNyIjoiMSIsImFsbG93ZWQtb3JpZ2lucyI6WyIvKiJdLCJyZWFsbV9hY2Nlc3MiOnsicm9sZXMiOlsib2ZmbGluZV9hY2Nlc3MiLCJkZWZhdWx0LXJvbGVzLWV4cHJlc3MtZGVtbyIsImFkbWluIiwidW1hX2F1dGhvcml6YXRpb24iXX0sInJlc291cmNlX2FjY2VzcyI6eyJhY2NvdW50Ijp7InJvbGVzIjpbIm1hbmFnZS1hY2NvdW50IiwibWFuYWdlLWFjY291bnQtbGlua3MiLCJ2aWV3LXByb2ZpbGUiXX19LCJzY29wZSI6InByb2ZpbGUgZW1haWwiLCJzaWQiOiJjZDk2OTU5MS02N2YzLTRiMTQtYjVmOS01MTkwMWIxYjEyOTkiLCJlbWFpbF92ZXJpZmllZCI6ZmFsc2UsIm5hbWUiOiJBZG1pbiIsInByZWZlcnJlZF91c2VybmFtZSI6ImFkbWluIiwiZ2l2ZW5fbmFtZSI6IkFkbWluIn0.Id5xtzGMcT5YegaiFNOQ72dz5UyDVqc0lyAAyY-BLdBYOomnT1iVo0Ef9cARNIqiBSP_IYAEu5O7Fs0USiOxFaSnG1CokdRhWgUQgQTJHpK8-9VQVM7KgNTuY8_Ve2t5iEYHwIHPB4mpoXqjR86U-COwp5OEvPAt_2hW9EGNP91WRuabFReOftK9NOpBU5EykVQT-dqSkLiBKvBcLxWIhkxmj6onrLkYui6OA51K3ySsjPSOhFsaxQCxjKEx1HiUacUa514VzbhptXZuPT6-vGmFf5vaWdkoziPdGr64QN60AWEQTkDOG7o_7wgGa5kwoffkwaVZHjg58IpH8S1q7w'
```

yes, it works.

```shell
HTTP/1.1 200 OK
Content-Length: 62
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Sat, 24 Feb 2024 11:16:47 GMT
Etag: W/"3e-c1bXCmx1ld/9Hu8gcReh8tu8CCg"
Keep-Alive: timeout=4
Proxy-Connection: keep-alive
X-Powered-By: Express
```

all done.

## How keycloak-connect middleware work

{{<mermaid>}}
sequenceDiagram
    Clients->>+keycloak-connect: Hello Express, i want access /secured endpoint,here is my token
    keycloak-connect->>+Keycloak Server: Hi Keycloak, show me public key/certs
    Keycloak Server-->>-keycloak-connect: Hi Express, here is public key/certs
    keycloak-connect-->>-keycloak-connect: verify signature with public key,if token is valid then go next middleware,if invalid return 403
    keycloak-connect->>+Clients: Hello Clients, your token is invalid, return 403

{{</mermaid>}}

- [keycloak-connect](https://www.npmjs.com/package/keycloak-connect) will get public key/certs from [http://localhost:8080/realms/express-demo/protocol/openid-connect/certs](http://localhost:8080/realms/express-demo/protocol/openid-connect/certs)

please go to [http://localhost:8080/realms/express-demo/protocol/openid-connect/certs](http://localhost:8080/realms/express-demo/protocol/openid-connect/certs) to get your public key/certs.

```json
{
  "keys": [
    {
      "kid": "x20fqA32CtrJGukIB5U1YhCDookaaXtlIEDhtBwj6ss",
      "kty": "RSA",
      "alg": "RS256",
      "use": "sig",
      "n": "lRUEZzahCKh2QFz0LpiZyaoIGXksqZQ9g8aLtlVvTBg9_gXExz4gjTZeZN5H_Xx6_5Ze3ZRWAKg9u4IN1xvdWOa5Eq6QeXc-csMjrTiGZfL5Z1Ov9KvuSl3W8tgVyytw2-zB9yI905tjGc5W8MIKSKBLkA_-K2opSze37sxRo70oqv-JR34sPcbEo1deI8Qq869qIHiRrUApmxuj3nDsALaIKrzjwDF0Hr1Z1U4UrxYYYoYnVDDbyQeCTh7fRfHpfRr_18e82by5sOckXcudabjCbqz7t84iCLa2OwNz3mR5F3mh6kqpG-OVmTHDNSFfnvK6pvDtXhlVLzZbqpSmnw",
      "e": "AQAB",
      "x5c": [
        "MIICpzCCAY8CBgGN3dERszANBgkqhkiG9w0BAQsFADAXMRUwEwYDVQQDDAxleHByZXNzLWRlbW8wHhcNMjQwMjI1MDEwOTQyWhcNMzQwMjI1MDExMTIyWjAXMRUwEwYDVQQDDAxleHByZXNzLWRlbW8wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCVFQRnNqEIqHZAXPQumJnJqggZeSyplD2Dxou2VW9MGD3+BcTHPiCNNl5k3kf9fHr/ll7dlFYAqD27gg3XG91Y5rkSrpB5dz5ywyOtOIZl8vlnU6/0q+5KXdby2BXLK3Db7MH3Ij3Tm2MZzlbwwgpIoEuQD/4railLN7fuzFGjvSiq/4lHfiw9xsSjV14jxCrzr2ogeJGtQCmbG6PecOwAtogqvOPAMXQevVnVThSvFhhihidUMNvJB4JOHt9F8el9Gv/Xx7zZvLmw5yRdy51puMJurPu3ziIItrY7A3PeZHkXeaHqSqkb45WZMcM1IV+e8rqm8O1eGVUvNluqlKafAgMBAAEwDQYJKoZIhvcNAQELBQADggEBAFxSFu+rE8nwF5BWRyesb3YER9IpCBgtVhEy0mtM2+BE4cM9Z2NUJ14tMXPofFk2SipefPImX+R9U2L9hIaAENQFlyMw58NHwKlhclkCB74k3S3OzzJRmTz3O7SSlViJUoV/ODBOdNTPYi8ZS09JwtyFeNu1Ber5Tf3ucjrpYCvOdCCAF/E1IrDMLxMWuv5qGicIOd1x+nHJD3pFN6z+8L7WwMdi+UdCmqaytXHtl6oiyuoa07uAIEKKcagRPR+CMU1zDa5OyZ8+lkxZfU8IfwiVe6PfYt6xh3nxF7vNrhcQdrGwwN3kOn+xQJQVWidl6rpOUTIr5H1eFWRV/T4a5Uk="
      ],
      "x5t": "uOixjC3EZPHJ-O8gfcjMLyRaiuI",
      "x5t#S256": "wFsYZihCnx7CnRXQ_ZQK38EzbpWjA9SkwlUBfOQR2PY"
    },
    {
      "kid": "04CCtah5XxLzBB8Rvuin6vSBIA4jICozElhTeiwMmac",
      "kty": "RSA",
      "alg": "RSA-OAEP",
      "use": "enc",
      "n": "uj9VjPD2eomE8VMLvNlswUzCzGesh1STIKeTWNoDqIqLTHXyqG7C_q50Vegq5hJeN9ijLVSB5DQECRc9BrCX4Qj2JL50nayO1FMA_QWO-Gta3M-VDgV2ikxP3mMSjXNP0Msq3atbsQRz4PL28JtFQtLQp-Qs3Z5hSJhit5OvbsnUoemKu1fwW-4BiArr7PSKAeRqCnWwQgOxX7N4trTqt1XsOyo8W6p2ncOusnVTHqvvG0lbpblZ84PvUCCSJ0Ykq9I5D9-aCmR-_9PKmeWQo-1BJqk2BTyOm7escKrpIO1mlvwvEwFyqjxpkluR0-nfsusE2MOCWt96Z-64sJaUOQ",
      "e": "AQAB",
      "x5c": [
        "MIICpzCCAY8CBgGN3dESrjANBgkqhkiG9w0BAQsFADAXMRUwEwYDVQQDDAxleHByZXNzLWRlbW8wHhcNMjQwMjI1MDEwOTQzWhcNMzQwMjI1MDExMTIzWjAXMRUwEwYDVQQDDAxleHByZXNzLWRlbW8wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC6P1WM8PZ6iYTxUwu82WzBTMLMZ6yHVJMgp5NY2gOoiotMdfKobsL+rnRV6CrmEl432KMtVIHkNAQJFz0GsJfhCPYkvnSdrI7UUwD9BY74a1rcz5UOBXaKTE/eYxKNc0/Qyyrdq1uxBHPg8vbwm0VC0tCn5CzdnmFImGK3k69uydSh6Yq7V/Bb7gGICuvs9IoB5GoKdbBCA7Ffs3i2tOq3Vew7Kjxbqnadw66ydVMeq+8bSVuluVnzg+9QIJInRiSr0jkP35oKZH7/08qZ5ZCj7UEmqTYFPI6bt6xwqukg7WaW/C8TAXKqPGmSW5HT6d+y6wTYw4Ja33pn7riwlpQ5AgMBAAEwDQYJKoZIhvcNAQELBQADggEBAHV4TC129PdBx34JgSWMhpp1ytmECvrM1IdnDOrC9KuM7kmKzEiDRlEG6ElDbU1duzrE/CkZmVNwDUDaHtAvPGCixeQfHyRd+KRWTnNqKhlvLhngLPOHljybp4THLMYAesmyFDqySnJiQglyiY4KlbBMoWt0rvZ7+3FbrCR6XdMxZU27YCdANopXE0RaRLEKT4KEWGTQ/6PZ8yjfdkGTBQ6Mxxltky1d7T7SuEufnJAOo1SinXv+ypXIke2ug0+SXekAuBCG5sBb91739gI5ATGuObzlkbnHAd16f6ddmb49JU//Y2ICy6/5V+f8CT+e+YiQBs9b54XUpmyjnfXK5EQ="
      ],
      "x5t": "7gZ4100kJvz2QGO765XZ-jCAVzw",
      "x5t#S256": "2kcRZp8RoCiPQjLBOMhuvz5z79MDyzvFOSHmZgNlPlY"
    }
  ]
}
```
pick the `x5c` value from `alg:RS256`.

then you can go to [https://jwt.io/](https://jwt.io/) to verify your token manually.

1. paste your token on the left input box
2. insert your `x5c` value between `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----`

something like this

```cert
-----BEGIN CERTIFICATE-----
MIICpzCCAY8CBgGN3dERszANBgkqhkiG9w0BAQsFADAXMRUwEwYDVQQDDAxleHByZXNzLWRlbW8wHhcNMjQwMjI1MDEwOTQyWhcNMzQwMjI1MDExMTIyWjAXMRUwEwYDVQQDDAxleHByZXNzLWRlbW8wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCVFQRnNqEIqHZAXPQumJnJqggZeSyplD2Dxou2VW9MGD3+BcTHPiCNNl5k3kf9fHr/ll7dlFYAqD27gg3XG91Y5rkSrpB5dz5ywyOtOIZl8vlnU6/0q+5KXdby2BXLK3Db7MH3Ij3Tm2MZzlbwwgpIoEuQD/4railLN7fuzFGjvSiq/4lHfiw9xsSjV14jxCrzr2ogeJGtQCmbG6PecOwAtogqvOPAMXQevVnVThSvFhhihidUMNvJB4JOHt9F8el9Gv/Xx7zZvLmw5yRdy51puMJurPu3ziIItrY7A3PeZHkXeaHqSqkb45WZMcM1IV+e8rqm8O1eGVUvNluqlKafAgMBAAEwDQYJKoZIhvcNAQELBQADggEBAFxSFu+rE8nwF5BWRyesb3YER9IpCBgtVhEy0mtM2+BE4cM9Z2NUJ14tMXPofFk2SipefPImX+R9U2L9hIaAENQFlyMw58NHwKlhclkCB74k3S3OzzJRmTz3O7SSlViJUoV/ODBOdNTPYi8ZS09JwtyFeNu1Ber5Tf3ucjrpYCvOdCCAF/E1IrDMLxMWuv5qGicIOd1x+nHJD3pFN6z+8L7WwMdi+UdCmqaytXHtl6oiyuoa07uAIEKKcagRPR+CMU1zDa5OyZ8+lkxZfU8IfwiVe6PfYt6xh3nxF7vNrhcQdrGwwN3kOn+xQJQVWidl6rpOUTIr5H1eFWRV/T4a5Uk=
-----END CERTIFICATE----- 
```

then paste the content to the right botton input box, you will see the sample result with the screen shot

![jwt-token-verifying](/images/jwt-token-verifying.png)

## Sample Source Code

[Star me @GitHub](https://github.com/wujun4code/express-keycloak-typescript-quick-start-2024).


## Further Features

1. Keycloak is a beckend service and do NOT provide a UI to login/signup, so you can integrate some front-end UI components based Express, or you can redirect by client side(React, etc.)
2. Secure all your beckend services with a proxy, this proxy is armed by Keycloak, one solution is [oauth2-proxy](https://github.com/oauth2-proxy/oauth2-proxy).