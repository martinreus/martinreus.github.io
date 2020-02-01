---
title: Create an Angular app with translated field validations  using Transloco, Server-side Rendering (ssr) and Angular Material
date: 2020-02-01 15:35
category: [Software Engineering, Angular]
author: martin
# tags: []
# summary:
image: assets/images/posts/angular.svg
featured: true
hidden: true
---

Angular 9 is almost out! Ivy render engine will now be set as default and we can expect improved i18n and l10n. To translate your website in different languages using the provided I18N capabilities though, you will have to compile your source code in every language your website is going to be presented to the user, and have every different version running behind a proxy that will route your user to the correct version. This might not fit your development workflow and could also add some additional complexity when deploying your website. On top of that, changing the language in runtime is not yet possible, although they are already working on it - [see the issue here](https://github.com/angular/angular/issues/16477).

There are some alternatives to Angular's provided I18N library, like [ngx-translation](https://github.com/ngx-translate/core). However, the owner of this repository is shifting his efforts (see [here](https://github.com/ngx-translate/core/issues/783)) towards helping to improve the official Angular I18N library, as well as being part of the team that is developing a pretty awesome translation library (in my humble opinion) called [Transloco](https://github.com/ngneat/transloco).

Having that in mind, I wanted to start building a new project that I envisioned needed at least to have

- Server-side Rendering (SSR), to reduce first load times, important especially for people on mobile phones
- I18N, which would also need to work when fetching that first page using SSR
- Form Validation with translations using Transloco
- Angular Material, because it just works and looks clean :)

So let's build it!

## Initial setup

#### Creating a new project

While Angular 9 is still not yet released, we can use its RC version (currently at 9.0.0-rc.12). To do this more easily, I've installed @angular/cli@next (which installs the current RC version) to generate a new project called `ssr-translate`.

```bash
# install @angular/cli
npm i -g @angular/cli@next
# generate project
ng new ssr-translate
cd ssr-translate
```

Angular CLI will ask if you want to enable routing (to which I answered yes) and what kind of stylesheet you want to use (I chose SCSS).

#### Enable Server-side Rendering

Enable server-side rendering by adding angular universal (be sure to install the @next version, otherwise you might run into errors)

```bash
ng add @nguniversal/express-engine@next --clientProject ssr-translate
```

where ssr-translate is the name of the project you created. You should expect an output similar to

```cmd
Installing packages for tooling via npm.
Installed packages for tooling via npm.
CREATE src/main.server.ts (298 bytes)
CREATE src/app/app.server.module.ts (318 bytes)
CREATE tsconfig.server.json (325 bytes)
CREATE server.ts (2013 bytes)
UPDATE package.json (1799 bytes)
UPDATE angular.json (5188 bytes)
UPDATE src/main.ts (432 bytes)
UPDATE src/app/app.module.ts (438 bytes)
UPDATE src/app/app-routing.module.ts (284 bytes)
✔ Packages installed successfully.
```

I am not going into much detail here about what files are additionally created after adding SSR support, since the [official documentation](https://next.angular.io/guide/universal) already covers this in great detail, and I can only recommend you reading the details over there :).

By now you should be able to test that SSR works by building and serving your app

```bash
npm run build:ssr && npm run serve:ssr
```

This can take a while, but if everything works you should see something like

```cmd
[...]
chunk {4} styles.0e4338761429b4eb16ac.css (styles) 0 bytes [initial] [rendered]
Date: 2020-02-01T21:56:40.381Z - Hash: 588b308cc4f8db79fdb6 - Time: 31561ms
Hash: 0458e5a6c22e09044830
Time: 17983ms
Built at: 02/01/2020 9:57:08 PM
  Asset      Size  Chunks                    Chunk Names
main.js  2.83 MiB       0  [emitted]  [big]  main
Entrypoint main [big] = main.js
chunk    {0} main.js (main) 5.81 MiB [entry] [rendered]

> ssr-translate@0.0.0 serve:ssr /home/martin/how-tos/ssr-translate
> node dist/ssr-translate/server/main.js

Node Express server listening on http://localhost:4000
```

#### Add Transloco

This was by far the part that took most time, but thankfully now that I am done with it you can reap the benefits =)

First, we'll add transloco, by running

```bash
ng add @ngneat/transloco
```

When asked, choose the languages you'll have on your website (comma separated) and also be sure to answer `y` when asked if working with server side rendering.

Transloco adds a configuration property to both `src/environments/environment.ts` and `src/environments/environment.prod.ts`. The configuration for the production environment needs some special attention: when running the website in SSR mode, Transloco will download translation files dinamically from the URL that is set in `baseUrl` property; for local tests, I had to change mine to

```typescript
export const environment = {
  baseUrl: "http://localhost:4000",
  production: true
};
```

where port is now 4000 (the address express runs on when running the app in SSR mode). Also note that this baseUrl will have to match your domain name once you deploy your app, otherwise Transloco will not be able to retrieve translations!
