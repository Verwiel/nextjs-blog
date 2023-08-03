# Packages
For styling with Sass
```
npm install -D sass
```

Each markdown file has a metadata section at the top containing title and date. This is called YAML Front Matter, which can be parsed using a library called gray-matter.
```
npm install gray-matter
```

To render markdown content
```
npm install remark remark-html
```

Format dates
```
npm install date-fns
```

# Basic Routing
## Client-Side Navigation
The Link component enables client-side navigation between two pages in the same Next.js app.

Client-side navigation means that the page transition happens using JavaScript, which is faster than the default navigation done by the browser.

Here’s a simple way you can verify it:

Use the browser’s developer tools to change the background CSS property of ```<html>``` to yellow.
Click on the links to go back and forth between the two pages.
You’ll see that the yellow background persists between page transitions.
This shows that the browser does not load the full page and client-side navigation is working.

If you’ve used ```<a href="…">``` instead of ```<Link href="…">``` and did this, the background color will be cleared on link clicks because the browser does a full refresh.

## Code splitting and prefetching
Next.js does code splitting automatically, so each page only loads what’s necessary for that page. That means when the homepage is rendered, the code for other pages is not served initially.

This ensures that the homepage loads quickly even if you have hundreds of pages.

Only loading the code for the page you request also means that pages become isolated. If a certain page throws an error, the rest of the application would still work.

Furthermore, in a production build of Next.js, whenever Link components appear in the browser’s viewport, Next.js automatically prefetches the code for the linked page in the background. By the time you click the link, the code for the destination page will already be loaded in the background, and the page transition will be near-instant!

## Summary
Next.js automatically optimizes your application for the best performance by code splitting, client-side navigation, and prefetching (in production).

You create routes as files under pages and use the built-in Link component. No routing libraries are required.

You can learn more about the Link component in the API reference for next/link and routing in general in the routing documentation.

# Dynamic Routes
Pages that begin with [ and end with ] are dynamic routes in Next.js.
![](https://nextjs.org/static/images/learn/dynamic-routes/page-path-external-data.png)
![](https://nextjs.org/static/images/learn/dynamic-routes/how-to-dynamic-routes.png)

Important: The returned list is not just an array of strings — it must be an array of objects that look like the comment above. Each object must have the params key and contain an object with the id key (because we’re using [id] in the file name). Otherwise, getStaticPaths will fail.

## Fetch External API or Query Database
Like getStaticProps, getStaticPaths can fetch data from any data source. In our example, getAllPostIds (which is used by getStaticPaths) may fetch from an external API endpoint:

```
export async function getAllPostIds() {
  // Instead of the file system,
  // fetch post data from an external API endpoint
  const res = await fetch('..');
  const posts = await res.json();
  return posts.map((post) => {
    return {
      params: {
        id: post.id,
      },
    };
  });
}
```
## Fallback
Recall that we returned fallback: false from getStaticPaths. What does this mean?

If fallback is false, then any paths not returned by getStaticPaths will result in a 404 page.

If fallback is true, then the behavior of getStaticProps changes:

- The paths returned from getStaticPaths will be rendered to HTML at build time.
- The paths that have not been generated at build time will not result in a 404 page. Instead, Next.js will serve a “fallback” version of the page on the first request to such a path.
- In the background, Next.js will statically generate the requested path. Subsequent requests to the same path will serve the generated page, just like other pages pre-rendered at build time.

If fallback is blocking, then new paths will be server-side rendered with getStaticProps, and cached for future requests so it only happens once per path.

This is beyond the scope of our lessons, but you can learn more about fallback: true and fallback: 'blocking' in the fallback documentation.

## Catch-all Routes
Dynamic routes can be extended to catch all paths by adding three dots (...) inside the brackets. For example:

pages/posts/[...id].js matches /posts/a, but also /posts/a/b, /posts/a/b/c and so on.
If you do this, in getStaticPaths, you must return an array as the value of the id key like so:

```
return [
  {
    params: {
      // Statically Generates /posts/a/b/c
      id: ['a', 'b', 'c'],
    },
  },
  //...
];
```

And params.id will be an array in getStaticProps:

```
export async function getStaticProps({ params }) {
  // params.id will be like ['a', 'b', 'c']
}
```

## Router
If you want to access the Next.js router, you can do so by importing the useRouter hook from next/router.

## 404 Pages
To create a custom 404 page, create pages/404.js. This file is statically generated at build time.

# Assets
Next.js can serve static assets, like images, under the top-level public directory. Files inside public can be referenced from the root of the application similar to pages.

The public directory is also useful for robots.txt, Google Site Verification, and any other static assets. Check out the documentation for Static File Serving to learn more.

## Image Component and Image Optimization
```next/image``` is an extension of the HTML <img> element, evolved for the modern web.

Next.js also has support for Image Optimization by default. This allows for resizing, optimizing, and serving images in modern formats like WebP when the browser supports it. This avoids shipping large images to devices with a smaller viewport. It also allows Next.js to automatically adopt future image formats and serve them to browsers that support those formats.

Automatic Image Optimization works with any image source. Even if the image is hosted by an external data source, like a CMS, it can still be optimized.

### Using the Image Component
Instead of optimizing images at build time, Next.js optimizes images on-demand, as users request them. Unlike static site generators and static-only solutions, your build times aren't increased, whether shipping 10 images or 10 million images.

Images are lazy loaded by default. That means your page speed isn't penalized for images outside the viewport. Images load as they are scrolled into viewport.

Images are always rendered in such a way as to avoid Cumulative Layout Shift, a Core Web Vital that Google is going to use in search ranking.

## Metadata
What if we wanted to modify the metadata of the page, such as the ```<title>``` HTML tag?

```<title>``` is part of the ```<head>``` HTML tag, so let's dive into how we can modify the ```<head>``` tag in a Next.js page.

```<Head>``` is used instead of the lowercase ```<head>```. ```<Head>``` is a React Component that is built into Next.js. It allows you to modify the ```<head>``` of a page.

You can import the Head component from the ```next/head``` module.

## Third-Party JavaScript
Third-party JavaScript refers to any scripts that are added from a third-party source. Usually, third-party scripts are included in order to introduce newer functionality into a site that does not need to be written from scratch, such as analytics, ads, and customer support widgets.

```next/script``` is an extension of the HTML ```<script>``` element and optimizes when additional scripts are fetched and executed.

- ```strategy``` controls when the third-party script should load. A value of lazyOnload tells Next.js to load this particular script lazily during browser idle time
- ```onLoad``` is used to run any JavaScript code immediately after the script has finished loading. In this example, we log a message to the console that mentions that the script has loaded correctly

## Global Styles
CSS Modules are useful for component-level styles. But if you want some CSS to be loaded by every page, Next.js has support for that as well.

To load global CSS to your application, create a file called pages/_app.js with the following content:

The default export of _app.js is a top-level React component that wraps all the pages in your application. You can use this component to keep state when navigating between pages, or to add global styles as we're doing here. Learn more about _app.js file.

### Adding Global CSS
In Next.js, you can add global CSS files by importing them from pages/_app.js. You cannot import global CSS anywhere else.

The reason that global CSS can't be imported outside of pages/_app.js is that global CSS affects all elements on the page.

If you were to navigate from the homepage to the /posts/first-post page, global styles from the homepage would affect /posts/first-post unintentionally.

You can place the global CSS file anywhere and use any name. So let’s do the following:
- Create a top-level styles directory and a global.css file.
- Add the following CSS inside styles/global.css. This code resets some styles and changes the color of the a tag:

``` 
html,
body {
  padding: 0;
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen, Ubuntu,
    Cantarell, Fira Sans, Droid Sans, Helvetica Neue, sans-serif;
  line-height: 1.6;
  font-size: 18px;
}

* {
  box-sizing: border-box;
}

a {
  color: #0070f3;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

img {
  max-width: 100%;
  display: block;
} 
```

# Styling Tips
## Using clsx library to toggle classes
```clsx``` is a simple library that lets you toggle class names easily. You can install it using ```npm install clsx```.

Please take a look at its documentation for more details, but here's the basic usage:

Suppose that you want to create an Alert component which accepts type, which can be 'success' or 'error'.
If it's 'success', you want the text color to be green. If it's 'error', you want the text color to be red.
You can first write a CSS module (e.g. alert.module.css) like this:
```
.success {
  color: green;
}
.error {
  color: red;
}
```
And use clsx like this:
```
import styles from './alert.module.css';
import { clsx } from 'clsx';

export default function Alert({ children, type }) {
  return (
    <div
      className={clsx({
        [styles.success]: type === 'success',
        [styles.error]: type === 'error',
      })}
    >
      {children}
    </div>
  );
}
```

## Using Sass
Out of the box, Next.js allows you to import Sass using both the .scss and .sass extensions. You can use component-level Sass via CSS Modules and the .module.scss or .module.sass extension.

Before you can use Next.js' built-in Sass support, be sure to install sass:
``` npm install -D sass ```

# Pre-rendering and Data Fetching
We’ll store the blog content in the file system, but it’ll work if the content is stored elsewhere (e.g. database or Headless CMS).

## Pre-rendering
By default, Next.js pre-renders every page. This means that Next.js generates HTML for each page in advance, instead of having it all done by client-side JavaScript. Pre-rendering can result in better performance and SEO.

Each generated HTML is associated with minimal JavaScript code necessary for that page. When a page is loaded by the browser, its JavaScript code runs and makes the page fully interactive. (This process is called hydration.)

You can check that pre-rendering is happening by taking the following steps:
- Disable JavaScript in your browser. (Here’s how in Chrome).
- Try accessing this page (the final result of this tutorial).

You should see that your app is rendered without JavaScript. That’s because Next.js has pre-rendered the app into static HTML, allowing you to see the app UI without running JavaScript.

``` Note: You can also try the above steps on localhost, but CSS won’t be loaded if you disable JavaScript. ```

### Two Forms of Pre-rendering
Next.js has two forms of pre-rendering: Static Generation and Server-side Rendering. The difference is in when it generates the HTML for a page.
- Static Generation is the pre-rendering method that generates the HTML at build time. The pre-rendered HTML is then reused on each request.
- Server-side Rendering is the pre-rendering method that generates the HTML on each request.

``` In development mode (when you run npm run dev or yarn dev), pages are pre-rendered on every request. This also applies to Static Generation to make it easier to develop. When going to production, Static Generation will happen once, at build time, and not on every request. ``` 

Importantly, Next.js lets you choose which pre-rendering form to use for each page. You can create a "hybrid" Next.js app by using Static Generation for most pages and using Server-side Rendering for others.

### When to Use Static Generation v.s. Server-side Rendering
We recommend using Static Generation (with and without data) whenever possible because your page can be built once and served by CDN, which makes it much faster than having a server render the page on every request.

You can use Static Generation for many types of pages, including:
- Marketing pages
- Blog posts
- E-commerce product listings
- Help and documentation

You should ask yourself: "Can I pre-render this page ahead of a user's request?" If the answer is yes, then you should choose Static Generation.

On the other hand, Static Generation is not a good idea if you cannot pre-render a page ahead of a user's request. Maybe your page shows frequently updated data, and the page content changes on every request.

In that case, you can use Server-side Rendering. It will be slower, but the pre-rendered page will always be up-to-date. Or you can skip pre-rendering and use client-side JavaScript to populate frequently updated data.

## Static Generation with and without Data
Static Generation can be done with and without data.

For some pages, you might not be able to render the HTML without first fetching some external data. Maybe you need to access the file system, fetch external API, or query your database at build time. Next.js supports this case — Static Generation with data — out of the box.

### Static Generation with Data using getStaticProps
How does it work? Well, in Next.js, when you export a page component, you can also export an async function called getStaticProps. If you do this, then:

- getStaticProps runs at build time in production, and…
- Inside the function, you can fetch external data and send it as props to the page.

```
export default function Home(props) { ... }

export async function getStaticProps() {
  // Get external data from the file system, API, DB, etc.
  const data = ...

  // The value of the `props` key will be
  //  passed to the `Home` component
  return {
    props: ...
  }
}
```

Essentially, getStaticProps allows you to tell Next.js: *“Hey, this page has some data dependencies — so when you pre-render this page at build time, make sure to resolve them first!”*

``` Note: In development mode, getStaticProps runs on each request instead. ```

### Fetch External API or Query Database
In lib/posts.js, we’ve implemented getSortedPostsData which fetches data from the file system. But you can fetch the data from other sources, like an external API endpoint or database, and it’ll work just fine.

``` Note: Next.js polyfills fetch() on both the client and server. You don't need to import it. ```

This is possible because getStaticProps only runs on the server-side. It will never run on the client-side. It won’t even be included in the JS bundle for the browser. That means you can write code such as direct database queries without them being sent to browsers.

getStaticProps can only be exported from a page. You can’t export it from non-page files.

One of the reasons for this restriction is that React needs to have all the required data before the page is rendered.

Since Static Generation happens once at build time, it's not suitable for data that updates frequently or changes on every user request.

In cases like this, where your data is likely to change, you can use Server-side Rendering. Let's learn more about server-side rendering in the next section.

## Client-side Rendering
If you do not need to pre-render the data, you can also use the following strategy (called Client-side Rendering):
- Statically generate (pre-render) parts of the page that do not require external data.
- When the page loads, fetch external data from the client using JavaScript and populate the remaining parts.

This approach works well for user dashboard pages, for example. Because a dashboard is a private, user-specific page, SEO is not relevant, and the page doesn’t need to be pre-rendered. The data is frequently updated, which requires request-time data fetching.

## SWR
The team behind Next.js has created a React hook for data fetching called SWR. We highly recommend it if you’re fetching data on the client side. It handles caching, revalidation, focus tracking, refetching on interval, and more. We won’t cover the details here, but here’s an example usage:
```
import useSWR from 'swr';

function Profile() {
  const { data, error } = useSWR('/api/user', fetch);

  if (error) return <div>failed to load</div>;
  if (!data) return <div>loading...</div>;
  return <div>hello {data.name}!</div>;
}
```

# API Routes

## Creating API Routes

API Routes let you create an API endpoint inside a Next.js app. You can do so by creating a function inside the pages/api directory that has the following formaT:

```
// req = HTTP incoming message, res = HTTP server response
export default function handler(req, res) {
  // ...
}
```

They can be deployed as Serverless Functions (also known as Lambdas).

## Do Not Fetch an API Route from getStaticProps or getStaticPaths
You should not fetch an API Route from getStaticProps or getStaticPaths. Instead, write your server-side code directly in getStaticProps or getStaticPaths (or call a helper function).

Here’s why: getStaticProps and getStaticPaths run only on the server-side and will never run on the client-side. Moreover, these functions will not be included in the JS bundle for the browser. That means you can write code such as direct database queries without sending them to browsers. Read the Writing Server-Side code documentation to learn more.

## A Good Use Case: Handling Form Input
A good use case for API Routes is handling form input. For example, you can create a form on your page and have it send a POST request to your API Route. You can then write code to directly save it to your database. The API Route code will not be part of your client bundle, so you can safely write server-side code.

## Preview Mode
Static Generation is useful when your pages fetch data from a headless CMS. However, it’s not ideal when you’re writing a draft on your headless CMS and want to preview the draft immediately on your page. You’d want Next.js to render these pages at request time instead of build time and fetch the draft content instead of the published content. You’d want Next.js to bypass Static Generation only for this specific case.

Next.js has a feature called Preview Mode to solve the problem above, and it utilizes API Routes. To learn more about it take a look at our Preview Mode documentation.

## Dynamic API Routes
API Routes can be dynamic, just like regular pages. Take a look at our Dynamic API Routes documentation to learn more.

# Deployment

