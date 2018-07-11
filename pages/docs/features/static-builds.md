import withDoc, { components } from '../../../lib/with-doc'
import { arunoda } from '../../../lib/data/team'

export const meta = {
  title: 'Static Builds',
  description: 'All you need to know about assigning domains to your Now deployments',
  date: '12 July 2018',
  authors: [arunoda],
  editUrl: 'pages/docs/features/static-builds.md'
}

Deploying static apps with Now is very interesting because you don't need to pay for the computing resources. You just pay for the bandwidth, just like a CDN.
Still, you can do pretty cool stuff like custom public directories, URL rewrites, and many more thanks to [serve handlers](https://zeit.co/blog/new-static-deployments#new-features).

Usually, you'll use a static site generator like Jekyll or even Next.js to build static HTML apps.
In order to deploy your app with Now, you need to build it locally and deploy it.

This is not bad, but we can do better.

## Now Static Builds

Now static builds are based on Docker.
You can create a `Dockerfile` where you asked to build the static app and put it in a directory called `/public`.
Then Now's build setup can extract that static app and deploy it for you.

This works really well with our [GitHub integration](https://zeit.co/github). See this [pull request](https://github.com/zeit/now-static-build-starter/pull/1). For each and every commit, you can access the built static app.
This was not possible before, since we don't know how to build your static app. But now, we can do it.

## Getting Started

Here we are using a [simple Next.js app](https://github.com/zeit/now-static-build-starter) which is configured to export HTML pages.
Basically, in order to generate the app, we need to run the following commands:

```
npm run build
npm run export -- -o ./out
```

> After that, you have a static HTML app inside the `./out` directory.

This is how you can deploy this app with Now's static build support.

### Dockerfile

Create a file called `Dockerfile` and add the following content:

```
FROM mhart/alpine-node

# set the default working directory
RUN mkdir /app
WORKDIR /app

# copy local files
COPY pages /app/pages
COPY package.json /app

# build and export the app
RUN npm install
RUN npm run build
RUN npm run export -- -o /public
```

It will simply build your app and put the static HTML into the /public directory.


> It is very important to put your static app inside the **/public** directory. That's the directory Now will consider as the root for your static app.

### Now.json

Next, you need to mark this as a static app. To do that, simply add the following content into a file called `now.json`.

```
{
   "type": "static"
}
```

> You can have any other fields inside the `now.json` file.<br/>
> But having `type: "static"` is a must.



### Deploy time

Now you can simply deploy your app by running the following command:

```
now
```

To do this, you need to have the latest version of the Now CLI.<br/>
You can get it from https://zeit.co/download.

### Installing Docker is optional.

In order to deploy this app, you don't need to have Docker installed on your local machine. Everything will be run inside Now. Now CLI will simply upload the Dockerfile and start the build process.

 You can of course build the Dockerfile locally and try it out.

But why don't you try it out with Now itself? It'll be faster.


## Limitless Possibilities

Here we just showed you how to deploy a static Next.js app, but you can build any kind of static app thanks to Docker.
Simply write a Dockerfile which suits your app's build setup.

Let's have a look at some examples:


**Jekyll with custom plugins**

[Here](https://github.com/now-examples/now-jekyll-example)'s a simple Jekyll-based static app which uses a custom plugin. You cannot deploy it with GitHub pages since that plugin is not whitelisted inside GitHub pages.

However, you can deploy this app inside Now without any issues. Here's the Dockerfile which builds the Jekyll app inside Now:

```
FROM ruby:2.5-alpine

# install some useful packages
RUN apk --no-cache add
  zlib-dev
  build-base
  libxml2-dev
  libxslt-dev
  readline-dev
  libffi-dev
  yaml-dev
  zlib-dev
  libffi-dev
  cmake

# set the default working directory
RUN mkdir /app
WORKDIR /app

# copy local files
COPY . /app

# build and export the app
RUN bundle install
RUN bundle exec jekyll build --destination /public
```

**Your own CI server**

This is a really interesting way to use libraries and plugin authors. Even though your repository doesn't have anything to deploy, you can use both static builds and Now GitHub integration in an interesting way.

Have a look at this pull request(PR): https://github.com/importpw/querystring/pull/1.

When someone pushes a commit for that PR, Now will run tests inside that app and host the test report as a static app.

Here's the Dockerfile used inside that repo:

```
FROM alpine:3.5
RUN apk add --no-cache curl bash
WORKDIR /public
COPY . .
RUN bash ./test.sh
RUN echo "All tests passed!" > index.txt
```

These are just two interesting things you can do with static builds. Since you can do anything with Docker, you can build and deploy inside Now.

export default withDoc({...meta})(({children}) => <>{children}</>)
