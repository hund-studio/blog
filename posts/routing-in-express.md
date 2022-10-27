---
title: Dominate routing in Express
description: Brief thoughts about routing in express
tags: ''
cover_image: ''
canonical_url: null
published: true
id: 1227809
date: '2022-10-23T19:40:59Z'
---

When a project starts growing in complexity, you might end up with different routes for API handling, page-serving, webhooks etc...

To avoid your index.ts to grow indefinitely one smart approach would be to divide all the different routes into groups (ae. by path, functionality, auth type etc...) and load them into your file. It can be done using different approaches to fit different levels of complexity.

I'm not a fan of overcomplicating the project with sophisticated dynamic loading if not strictly required.

## Step 1: Create groups of routes

Let's say you want to group all API routes under the same `/api` URL path. The easiest solution would be to create a new `express.Router`;

```ts
const apiRouter = express.Router();
```

Append all the needed `POST` and `GET` routes to the newly created `apiRouter`...

```ts
apiRouter.get("/hello", async (request, response, next) => {
	try {
		return response.status(200).json({ foo: "bar" });
	} catch (e) {
		const error = createHttpError(500);
		return response.status(error.statusCode).json(error);
	}
});

apiRouter.get("/world", async (request, response, next) => {
	try {
		return response.status(200).json({ hello: "world" });
	} catch (e) {
		const error = createHttpError(500);
		return response.status(error.statusCode).json(error);
	}
});
```

...and finally, add your `apiRouter` to the express app

```ts
app.use("/api", apiRouter);
```

In my opinion, if your app has two or three `/api` routes grouping them using this system would be enough.
This configuration will allow you to apply specific middleware and custom rules to your `/api` routes with ease.

```ts
app.use("/api", apiRouter, passwordAuthenticationMiddleware);
```

## Step 2: Store groups of routes inside different files

By now we grouped all the different routes but everything it is still inside the main `index.ts` file. As stated before, this is ok for a rather simple API with two or three small routes. As soon as the project grows in size and complexity grouping routes together wouldn't be enough and moving those routes to a different file will be necessary.

I would probably use a file structure similar to this one:

```tree
.
|--routes
|  |--api
|  |  |--hello
|  |  |  |--index.ts
|  |  |  |--get.ts
|  |  |--world
|  |  |  |--index.ts
|  |  |  |--get.ts
|  |  |  `--post.ts
|  |  `--index.ts
|  `--index.ts
|--public
`--index.ts
```

First of all, we have to 'pack' everything inside a `./routes` folder and then `import` using a structure which mimics the routes paths structure:

```
GET  /api/hello

GET  /api/world
POST /api/world
```

Starting from the 'deeper' file which identifies the request type (GET, POST, PUT, PATCH...)and will contain the actual response `return`:

```ts
// 📄 ./routes/api/hello/get.ts
import { NextFunction, Request, Response } from "express";
import createHttpError from "http-errors";

const get = async (
	request: Request,
	response: Response,
	next: NextFunction
) => {
	try {
		return response.status(200).json({ foo: "bar" });
	} catch (e) {
		const error = createHttpError(500);
		return response.status(error.statusCode).json(error);
	}
};

export { get };
```

```ts
// 📄 ./routes/api/hello/index.ts
export { get } from "./get";
```

It is long and boring but always tries to create an `index.ts` file to export your function (you can eventually use a vsCode plugin).

Going up to `./routes/api` we will need to assemble all our API routes using an `/api` router...

```ts
// 📄 ./routes/api/index.ts
import express from "express";
import { get as helloGet } from "./hello";
import { get as worldGet } from "./world";
import { post as worldPost } from "./world";

const router = express.Router();

router.get("hello", helloGet);
router.get("world", worldGet);
router.post("world", worldPost);

export { router };
```

```ts
// 📄 ./routes/index.ts
export { router as apiRouter } from "./api";
```

...and we are now ready to consume it...

```ts
// 📄 ./index.ts
import { apiRouter } from "./routes";

/* ...some in between code here... */

app.use("/api", apiRouter);

/* ...some in between code here... */
```

Now things are getting quite complex... but hey! your `index.ts` file is much clearer now and going through it in the distant future will be painless than using a single huge file.

> I'm not a fan of overcomplicating the project with sophisticated dynamic loading if not strictly required.

Even though it might seem a huge Matrioska of code at some point it will be inevitable (unless you are planning to use some different framework, which is a smart choice too).

This brings us straight to...

## Step 3: Create an automation to load all those routes automatically

Now. I know that many of you directly jump to this point because usually, the last point is the one which assembles everything in a working piece of code...BUT

> I'm not a fan of overcomplicating the project with sophisticated dynamic loading if not strictly required.

For this reason, unless you are crying in pain because adding new routes has become impossible yet... you should give it a shot at [Step 1](#step-1-create-groups-of-routes) or [Step 2](#step-2-store-groups-of-routes-inside-different-files) and check out if it is not worth stopping at that stage.

That said. Let's move forward.

> ⚠️ **This solution might not fit your case**: it has been written assuming that you are working on a plain and basic express application bundled with webpack >5.

Starting from here...

```ts
app.use("/api", apiRouter);
```

...our goal is to achieve something like that:

```ts
await loadRoutes(path.join(__dirname, "routes"), app);
```

A function that, given one entry path, load all our routes on our `app`.

First and foremost we need to define two functions which will allow us to get files and folders in a specific path, something like this perhaps:

```ts
const getSubfolders = (path: string) =>
	fs
		.readdirSync(path, { withFileTypes: true })
		.filter((file) => file.isDirectory());
```

```ts
const getFiles = (path: string) =>
	fs
		.readdirSync(path, { withFileTypes: true })
		.filter((file) => !file.isDirectory());
```

To start scraping recursively from top to bottom we need to wrap everything in a function which will lately call itself, starting from the `routesRoot` folder and will append all the found **middlewares** to the generated routes. Let's do it one step at a time.

```ts
const loadRoutes = async (routesRoot: string, router: Router) => {};
```

Inside our `loadRoutes` folder we will start looking for subfolders (we are not interested in files here since we are inside the root and we do not expect to find any middleware here).

To import all `routers` and `middlewares` from the `routes` tree we will use a **dynamic import**, this way **webpack** will be aware of what kind of chunks has to bundle separately.

Since **dynamic imports** are async functions we need to use a lot of `await` and `Promise.all()`.

Let's start scraping our root folder and create **relative paths** which we will use inside **dynamic imports** (disclaimer: this relative path is requested to use dynamic imports. It won't work with absolute paths):

```ts
const loadRoutes = async (routesRoot: string, router: Router) => {
	await Promise.all(
		getSubfolders(routesRoot).map(async ({ name }) => {
			const subFolder = `${routesRoot}/${name}`;
			const relativePath = path.relative(__dirname, subFolder);
		})
	);
};
```

Given this structure:

```tree
.
|--routes
|  |--api
|  |  |--hello
|  |     `-get.ts
|  `--welcome
|     |--index.ts
|     |--get.ts
|     `--post.ts
|--public
`--index.ts
```

We are now mapping `api` and `welcome` folders and their relative path will be `routes/api` and `routes/welcome`.

Next step is to create/import an `express.Router()` our rule will be:

- If the subfolder contains an `index.ts` file we will try to export an `express.Router()` out of it;
- If the `index.ts` file does not exist we will create a brand new router to handle the routes;

We can do something like this:

```ts
const loadRoutes = async (routesRoot: string, router: Router) => {
	await Promise.all(
		getSubfolders(routesRoot).map(async ({ name }) => {
			const subFolder = `${routesRoot}/${name}`;
			const relativePath = path.relative(__dirname, subFolder);

			let subRouter: Router;

			try {
				const { default: customRouter } = await import(
					`./${relativePath}/index.ts`
				);
				subRouter = customRouter;
			} catch (e) {
				subRouter = express.Router();
			}

			router.use(`/${name}`, subRouter);
		})
	);
};
```

You might have noticed that our `loadRoutes` accepts two parameters:

- `routesRoot`: is the path of our routes root folder form which we will start importing files and folders;
- `router`: is the initial router that we want to attach our routes (ad example the `app` router created with the express app).

Ok, let's recap a bit. Now we are scraping each subfolder and creating a sub router which will handle all routes contained in the such folder;

```
routers:
app.use(apiRouter)
  `--apiRouter.use(helloRouter)

app.use(welcomeRouter)
```

Now we must implement our scraping folder to look not only for subfolders and `index.ts` files but for `get`, `post`, `put` etc... files

Something like this:

```ts
type ExpressRouterRequestMethod =
	| "all"
	| "get"
	| "post"
	| "put"
	| "delete"
	| "patch";

const loadRoutes = async (routesRoot: string, router: Router) => {
	const expressRouterRequestMethod = [
		"all",
		"get",
		"post",
		"put",
		"delete",
		"patch",
	];

	await Promise.all(
		getSubfolders(routesRoot).map(async ({ name }) => {
			const subFolder = `${routesRoot}/${name}`;
			const relativePath = path.relative(__dirname, subFolder);

			let subRouter: Router;

			try {
				const { default: customRouter } = await import(
					`./${relativePath}/index.ts`
				);
				subRouter = customRouter;
			} catch (e) {
				subRouter = express.Router();
			}

			await Promise.all(
				getFiles(subFolder).map(async ({ name }) => {
					const requestMethodName = name.replace(".ts", "");

					try {
						if (expressRouterRequestMethod.includes(requestMethodName)) {
							const { default: customMiddleware } = await import(
								`./${relativePath}/${requestMethodName}.ts`
							);

							subRouter[requestMethodName as ExpressRouterRequestMethod](
								"/",
								customMiddleware
							);
						}
					} catch (e) {
						console.log(e);
					}
				})
			);

			await loadRoutes(path.join(__dirname, subFolder), subRouter);

			router.use(`/${name}`, subRouter);
		})
	);
};
```

As you can see, at the time of writing, I Had to manually specify the accepted request method and check them manually otherwise typescript will complain a lot. This is ugly and dirty so comment/writes to me if you find a more elegant way. [Check out the express docs here](https://expressjs.com/en/4x/api.html#app.METHOD).

This way we are basically importing every valid router request type and attaching it to the right router in order to have a structure like this one:

```
routers:
app.use(apiRouter)
  `--apiRouter.use(helloRouter)
    `--helloRouter.get(getHelloMiddleware)

app.use(welcomeRouter)
  |--welcomeRouter.get(getWelcomeMiddleware)
  `--welcomeRouter.get(postWelcomeMiddleware)
```

The last step is to export everything in the right format. In step 2 we grouped everything using routers to apply middlewares to multiple routes. We can keep that structure by reworking it a bit to simply our **dynamic import**

```diff
// 📄 ./routes/api/hello/get.ts
import { NextFunction, Request, Response } from "express";
import createHttpError from "http-errors";

- const get = async (
+ const middleware = async(
	request: Request,
	response: Response,
	next: NextFunction
) => {
	try {
		return response.status(200).json({ foo: "bar" });
	} catch (e) {
		const error = createHttpError(500);
		return response.status(error.statusCode).json(error);
	}
};

- export { get };
+ export default middleware;
```

```diff
// 📄 ./routes/api/index.ts
import express from "express";
- import { get as helloGet } from "./hello";
- import { get as worldGet } from "./world";
- import { post as worldPost } from "./world";

const router = express.Router();

+ router.use(authMiddleware); // this is useful to apply a middleware to all subroutes

- router.get("hello", helloGet);
- router.get("world", worldGet);
- router.post("world", worldPost);

- export { router };
+ export default router; // export default instead of named export
```

The next step will probably create a separate file and import this function from there or whatever you like the most.

This is my five cents about express routing handling on small applications.
The entire article could be resumed as follows:

- Don't overcomplicate your application ahead of time, with 2 routes you don't need anything overcomplicated as it will cause stupid problems during the prototyping phase;
- Beware of copy/pasting because once you start adding ts + webpack + custom loaders + whatever you will have to adjust your setup accordingly.

### Stupid problems I faced while writing this

- To read, write and scan files and directories freely on **webpack** remember to enable this option inside your `webpack.config.js`;

```js
node: {
			__dirname: true,
		}
```

- When using dynamic import **webpack** automatically manage the bundle without specifying any particular `entry` option to create custom output bundles;
- When using **dynamic imports** try to keep as much as possible plain string paths. That's why I'm using `./${relativePath}/${requestMethodName}.ts` instead of `${name}`, when using variables webpack will try to match all possible escaped paths warning you for each not existing file.

That's it, I guess. Bye 👋
