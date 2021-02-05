+++
title = "Extra robustness in your Koa projects with Typescript"
date = 2021-02-05
slug = "koa-typings"
[extra]
hn = "26041047"
lobsters = "wzxbh6/extra_robustness_your_koa_projects_with"
+++

With a bit of Typescript magic, it is possible to add strict typing to the body of Koa requests and responses.

<!-- more -->

Typescript and Koa users know how loose the Typescript typing is for the requests and responses of the Koa contexts. Indeed, Koa (and [Koa-Body](https://github.com/dlau/koa-body)) default types use `any` for the request and response body type. 

This short blog post will show how you can add more robustness to your controllers by typing your bodies thanks to a simple generic interface.

# Sparkling Koa with Genericity 

We want to be able to provide the types of the request and the response body, whilst keeping the built-in flexibility. That's easy to achieve with Typescript generics as demonstrated below:

```ts
import { Context, Request } from 'koa';

interface KoaRequest<RequestBody = any> extends Request {
    body?: RequestBody;
}

export interface KoaContext<RequestBody = any, ResponseBody = any> extends Context {
    request: KoaRequest<RequestBody>;
    body: ResponseBody;
}

export interface KoaResponseContext<ResponseBody> extends KoaContext<any, ResponseBody> {}
```

I am actually surprised that those are not the original type definitions in `@types/koa` and `@types/koa-body` but I am sure they had their reasons.

## Examples

Let's put our new interfaces in practice with a couple of examples.

```ts
import Router from 'koa-router';
import koaBody from 'koa-body';

interface HelloWorldRequestDto {
    name: string;
}

interface HelloWorldResponseDto {
    message: string;
}

const helloRouter = new Router();

helloRouter.post('/', koaBody(), async (ctx: KoaContext<HelloWorldRequestDto, HelloWorldResponseDto>) => {
    // Request body is now typed.
    const { name } = ctx.res.body;
    ctx.res.statusCode = 200;
    // Response body is also typed.
    ctx.body = {
        message: `Hello ${name}!`,
    };
});
```

The compiler will error if we try to access properties not defined in the body types. Using the same DTO types, the code below would not compile:

```ts
helloRouter.post('/', koaBody(), async (ctx: KoaContext<HelloWorldRequestDto, HelloWorldResponseDto>) => {
    // The compiler will fail as lastName is not defined in `HelloWorldRequestDto`.
    const { name, lastName } = ctx.res.body;
    ctx.res.statusCode = 200;
    // The compiler would also fail here because we are defining a property absent from `HelloWorldResponseDto`, and `message` is absent.
    ctx.body = {
        msg: `Hello ${name}!`,
    };
});
```
The `KoaContext` interface uses default value for the generics; this allows specifying the request body type without the response's: `KoaContext<RequestBodyType>` will assume that the response body type is `any`. The `KoaResponseContext` allows you to specify the response body type without the request's for extra convenience.

## Conclusion

Leveraging Typescript and generics, we were able to add stricter typing to Koa. That's it for today!