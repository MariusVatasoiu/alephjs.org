---
title: SSR & SSG
authors:
  - ije
  - Serdar Sever
  - razermoon
---

# SSR & SSG

By default, Aleph.js **pre-renders** every page. This means that Aleph.js generates HTML for each page in advance, instead of rendering it with client-side JavaScript. Pre-rendering can result in better performance and SEO.

Each generated HTML page only needs a small amount of JavaScript. When a page is loaded by the browser, its JavaScript code runs and makes the page fully interactive. (This process is called _hydration_.)

You can disable **SSR** functionality in `aleph.config.ts`:

```ts
export default {
  ssr: false, // SPA mode
  ...
}
```

or specify exclude paths:

```ts
export default {
  ssr: {
    exclude: [
      /^\/admin\//,
      /^\/dashboard\//
    ]
  },
  ...
}
```

## SSR Options

If you export an object called `ssr` with a `props` function from a page, Aleph.js will pre-render this page using the computed props by the `props` function at build time. The `RouterURL` object is passed as the first parameter of the function.

The `paths` in the `ssr` options returns a static paths if the page is a dynamic route. (This is equal to `getStaticPaths` of Next.js)

```tsx
import React from 'https://esm.sh/react'
import type { SSROptions } from 'https://deno.land/x/aleph/types.d.ts'

export const ssr: SSROptions = {
  props: async router => {
    return {
      $revalidate: 1, // revalidate props after 1 second
      serverTime: Date.now()
    }
  },
  paths: async () => {
    return []
  }
}

export default function Page(props) {
  return (
    <p>Now: {props.serverTime}</p>
  )
}
```

## `useDeno` hook

Aleph.js also provides an `useDeno` hook to mix the **Deno** runtime in your component:

```tsx
import React from 'https://esm.sh/react'
import { useDeno } from 'https://deno.land/x/aleph/framework/react/mod.ts'

export default function Page() {
  const version = useDeno(() => {
    return Deno.version
  })

  return (
    <p>Powered by Deno v{version.deno}</p>
  )
}
```

> To learn more `useDeno`, check the [`useDeno` Hook](/docs/advanced-features/usedeno-hook) documentation.

## Static Site Generation (SSG)

Aleph.js can export your app to a **static site** in html, which can be served on any server or CDN.

```bash
$ aleph build
```

For **dynamic routes**, your can define the **static paths** in `ssr` options:

```tsx
// pages/post/[id].tsx

import type { SSROptions } from 'https://deno.land/x/aleph/types.d.ts'

export const ssr: SSROptions = {
  paths: async () => {
    const posts = await (await fetch('https://.../api/posts')).json()
    return posts.map(({ id }) => `/post/${id}`)
  }
}

export default function Post() {
  return (
    <div>...</div>
  )
}
```

> See the [hello-world](https://alephjs-hello-world.vercel.app/) example on [Vercel](https://vercel.com).
