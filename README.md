# Carousel

This carousel is made with Nuxt3, Composition API (script setup) and SCSS. Images are fetched from API (https://api.nuxtjs.dev/). The carousel can be swiped with both mouse click and mouse movement/touch movement. It is looped (so swipe away 😉) and utilizes cookies to store the last displayed image and start from it after refresh.

## Setup

Make sure to install the dependencies:

```bash
# yarn
yarn install

# npm
npm install

# pnpm
pnpm install --shamefully-hoist
```

## Development Server

Start the development server on http://localhost:3000

```bash
npm run dev
```

## Production

Build the application for production:

```bash
npm run build
```

Locally preview production build:

```bash
npm run preview
```

Check out the [deployment documentation](https://nuxt.com/docs/getting-started/deployment) for more information.
