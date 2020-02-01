---
title: Create an Angular app with translated field validations  using Transloco, Server Side Rendering (ssr) and Angular Material
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

- Server Side Rendering (SSR), to reduce first load times so that especially people on mobile would have a very snappy response time
- I18N, which would also need to work when fetching that first page using SSR
- Form Validation using the translation framework
- Angular Material, because I want something ready to use that looks clean :)

## Initial setup
