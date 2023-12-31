## Setup

### Env variables 
Create a .env file in root directory and set your base url
eg. PUBLIC_BASE_URL="http://localhost:5173/"

### Backend
https://github.com/crabstars/CallbackServerPromoCode

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.


## Deploy

I use caddy with following config

```
your-domain {
   log

   root * /root/promotion-fe/build
   file_server

   # I use localhost:8001 because my api and fe are on the same server
   route /api/* {	
      reverse_proxy localhost:8001
   }
 
   redir / /promo 302
}
```
