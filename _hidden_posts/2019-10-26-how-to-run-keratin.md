---
title: How to run Keratin Authn server
date: 2019-10-26 15:47
category:
author:
tags: []
summary:
hidden: true
---

First, compile your server, using the provided Makefile. When not on a Mac, this could involve some specific task of the Makefile (since when not providing any target it will try to compile for all platforms).

For running in dev locally and assuming you're only going to use SQLite for test, you will first need to run the migrations.

```bash
APP_DOMAINS=localhost AUTHN_URL=localhost SECRET_KEY_BASE=1qazxsw2 DATABASE_URL=sqlite3://local/db ./dist/authn-linux64 migrate
```

And then you will be able to run the project, with

```bash
USERNAME_IS_EMAIL=true GITHUB_OAUTH_CREDENTIALS=githubClientId:githubClientPass FACEBOOK_OAUTH_CREDENTIALS=fbAppId:fbSecret APP_DOMAINS=localhost HTTP_AUTH_USERNAME=secure HTTP_AUTH_PASSWORD=secure AUTHN_URL=http://localhost:8081 PUBLIC_PORT=8081 PORT=8091 SECRET_KEY_BASE=256BitStrongPasswordHere DATABASE_URL=sqlite3://local/db ./dist/authn-linux64
```
