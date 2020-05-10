## Vite

Today we explore [Vite](https://github.com/vuejs/vite), an extremely fast development environment based on ES modules.

We will take a quick look at how ES modules work in a browser, with no build step to speak of, and build a simple clone of [Storybook](https://storybook.js.org/), a UI development environment that Vite is perfectly designed for.

SS:2

## Getting Started

All we will need is `vite` and the latest version of Vue 3:

```sh
yarn add vite vue@next
```

One of the great things about Vite is how easy it is to get started! At this point, normally you'd create a `webpack.config.js`, install a slew of transformers and configure everything. Not with Vite. All you need is an `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title></title>
</head>
<body>
  <div id="app"></div>

  <script type="module">
    import { greet } from '/greet.js'

    greet()
  </script>
</body>
</html>
```

Create a `greet.js` while you are at it:

```js
export const greet = () => console.log('Hello from ES modules!')
```

Before using Vite, run a regular HTTP server with python 3: `python -m http.server`. This is to emphasize what Vite is *not* doing - if you go to `http://localhost:8000` and look at the console. You will see `Hello from ES modules!`. Most modern browsers support ES module syntax without a build step, so this works fine. Vite relies on ES module support, part of why it has basically no configuration required, and why it's so damn fast.

## The Power of Vite

Let's see some of the cool tricks Vite. Kill your python server and start Vite with `yarn vite`. Head to `http://localhost:3000` - your greeting should still be showing in the console. One thing ES modules does not support out of the box is importing from `node_modules` - Vite has you covered. Let's create a simple Vue app:


```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title></title>
</head>
<body>
  <div id="app"></div>

  <script type="module">
    import { createApp, h } from 'vue'
    const App = {
      render() {
        return h('div', 'hello vite')
      }
    }
    const app = createApp(App).mount('#app')
  </script>
</body>
</html>
```

Refresh the page - you should see `hello vite`. Great! Good, but there are other projects that do this, namely [snowpack](https://www.snowpack.dev/). So what else does Vite do?

## Working with Vue components

Create an `App.vue`:

```html
<template>
  <div style="color: blue">
    Count: {{ count }}
    <button @click="inc">Increment</button>
  </div>
</template>

<script>
import { ref, reactive } from 'vue'

export default {
  setup() {
    const count = ref(0)
    const inc = () => count.value++

    return {
      count,
      inc,
    }
  }
}
</script>
```

And update `index.html`

```html
  <script type="module">
    import { createApp, h } from 'vue'
    import App from '/App.vue'

    const app = createApp(App).mount('#app')
  </script>
```

Head to your browser, hit `Increment` a few times, and update the `style="color: blue"` to be something else. Save your code - notice the color changed, but the count remained the same? Hot reload! And it's *fast* - perfect for something like storybook. I have no doubt we will see something like storybook using Vite, but faster and better, and hopefully without the React dependency.

## Bells and Whistles

So we have `vue` files without and configuration - we also have `<script lang="ts">` as well - no configuration needed, Vite is very much batteries included here. For styling, `style lang="sass">`, works too, all you you need to do is a quick `yarn add -D sass`.

## A Simple Storybook Clone

Let's build a simple storybook clone. To really feel just how fast Vite makes this sort of thing, you should either watch the screencast or follow along yourself. We are going to try out the latest beta of Vue Router, so install that with `yarn add vue-router@next`. Update `index.html`:

```html
  <script type="module">
    import { createApp, h } from 'vue'
    import { createRouter, createWebHistory } from 'vue-router'
    import App from '/App.vue'
    import Stories from '/Stories.vue'

    const router = createRouter({
      history: createWebHistory(),
      routes: [
        {
          path: '/:name?',
          component: Stories,
        }
      ]
    })

    const app = createApp(App)
    app.use(router)
    app.mount('#app')
  </script>
```

Pretty simple - we set up a router to render a `<Stories>` component, which will make soon. `App.vue` will now just handle the router-view:

```html
<template>
  <RouterView />
</template>
```

Finally, create `Stories.vue` - see below for a quick explanation of what's going on:

```html
<template>
  <div>
    <h1>Stories</h1>

    <div v-for="story in stories" :key="story">
      <RouterLink :to="`/${story}`">
        Stories of {{ story }}
      </RouterLink>
    </div>
  </div>
</template>

<script>
import { useRoute } from 'vue-router'

export default {
  setup() {
    const storyMap = {
      button: {
        name: 'button',
        component: {}
      },
      card: {
        name: 'card',
        component: {}
      }
    }
    const stories = Object.keys(storyMap)
    const route = useRoute()

    return {
      storyMap,
      stories,
    }
  },
}
</script>
```

There is a bit going on here. If you haven't tried out the new `vue-router`, you should! It has a hook based API, in additions to the classic API. We can get the current route with the `useRoute` hook - we will be using the `/:name` param to decide which story to render. We will store all the stories in an object, so it's easy to access the one we want by doing `storyMap[useRoute().params.name]`. This will save us the effort of looping the array to find the current story.

Refresh your page (Vite does not do hot reload on `index.html`, only JS/TS and Vue files). Now we have this:

SS:1

The router-links should also be working.

## Rendering Stories

I created some basic stories to test things out. Go ahead an import them:

```html
<script>
import { useRoute } from 'vue-router'
import ButtonStories from './Button.stories.vue'
import CardStories from './Card.stories.vue'

export default {
  setup() {
    const storyMap = {
      button: {
        name: 'button',
        component: ButtonStories
      },
      card: {
        name: 'card',
        component: CardStories
      }
    }
    const stories = Object.keys(storyMap)
    const route = useRoute()

    return {
      storyMap,
      stories,
    }
  },
}
</script>
```

The final thing we need to do is render the current story based on the route. We can do this with a `computed` property:

```js
const currentStory = computed(() => {
  const story = storyMap[route.params.name]
  if (story) {
    return story
  }
})
```

Return this from the `setup()` function, and in the `<template>` use a dynamic component to render it:

```html
<div v-if="currentStory">
  <h2>Stories for {{ currentStory.name }}</h2>
  <component :is="currentStory.component" />
</div>
```

It works:

ss2
ss3

Notice how this change is basically reflected in the browser immediately? Perfect for a UI prototyping environment - I'm excited for when someone (maybe me?) releases a storybook like project built on Vite.

Try adding an extra color to the `colors` array in `Button.stories.vue` and see the browser update near instantly. Cool, right?

## Discussion

While we built just touched on what Vite brings to the table, there is a lot more to be excited about. The real power here is because we are not using something like Webpack, that needs to transpile the entire project before updating the browser, Vite **scales**. No matter how many files you have in your project, the start-up and re-render time doesn't change. It only loads the ES modules it needs, when they needed.

For some perspective, one of the projects I work one uses React + TypeScript + storybook, with around 50 components + stories. Start-up takes around 30 seconds, and a few seconds to reflect changes in the browser. It might not seem like a lot, but when you are tweaking a UI ever so slightly, the near-instant feedback is a big deal.

Other than storybook-like, UI driven applications, Evan has a prototype called [Vitepress](https://github.com/vuejs/vitepress), which is a static site generator like [Vuepress](https://vuepress.vuejs.org/), aimed at documentation. The real Vite is a good fit for this is it will only load the page you are currently looking at. Because Vuepress is webpack-based, in needs to watch and recompile everything to show you changes, something which can be very slow when you have a large amount of documentation. My book, the [Vue Testing Handbook](https://vuepress.vuejs.org/), only has around 20 pages, but still takes 5-6 seconds to start up, and 2-3 to reflect changes in the browser. It may not seem like a big difference (only a few seconds), but no-one has ever asked for more complex configuration, or for slower applications, right?

## Conclusion

Vite is a lightning fast development environment, not unlike vue-cli, create-react-app, etc. Unlike those projects, it uses ES modules, which offloads a lot of complexity to the browser, and leads to faster reloads, and less need for configuration.

While we use it with Vue here, it is framework agnostic - it comes with [built-in JSX support](https://github.com/vuejs/vite#jsx), and you can already use it with React and Preact today! The project is young, but looks poised to become a major competitor to webpack based development environments moving forward.
