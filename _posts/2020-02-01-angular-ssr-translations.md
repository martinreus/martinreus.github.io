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

#### TL;DR;

This is a very large post :)

If you just want a working example of these integrations, head over to the [git repository](https://github.com/martinreus/ssr-translate)

## Initial setup

### Creating a new project

While Angular 9 is still not yet released, we can use its RC version (currently at 9.0.0-rc.12). To do this more easily, I've installed @angular/cli@next (which installs the current RC version) to generate a new project called `ssr-translate`.

```bash
# install @angular/cli
npm i -g @angular/cli@next
# generate project
ng new ssr-translate
cd ssr-translate
```

Angular CLI will ask if you want to enable routing (to which I answered yes) and what kind of stylesheet you want to use (I chose SCSS).

## Enable Server-side Rendering

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

Node Express server listening on [http://localhost:4000](http://localhost:4000)
```

## Add Transloco

This was by far the part that took most time, but thankfully now that I am done with it you can reap the benefits =)

First we'll add transloco by running

```bash
ng add @ngneat/transloco
```

When asked, choose the languages you'll have on your website (comma separated) and also be sure to answer `yes` when asked if working with server side rendering.

Transloco will add some translations inside `assets/i18n` folder. For a simple test, let's add a test translation. In my case, I configured Transloco to support _pt_ and _en_ languages. So I added some content in en.json

```json
{
  "test": "this is a test"
}
```

and pt.json

```json
{
  "test": "isto é um teste"
}
```

Now erase whatever you find inside app.component.html and paste this snippet:

{% assign escapedCurly = '{{ t("test") }}' %}

```html
<ng-container *transloco="let t">
  <p>{{ escapedCurly }}</p>
</ng-container>
```

You will find a more detailed explanation about how to use transloco in the [official documentation](https://netbasal.gitbook.io/transloco/translation-in-the-template/structural-directive).

Now if you run the app (without using SSR yet)

```bash
ng serve
```

and head over to [http://localhost:4200](http://localhost:4200), you will see that the message will always be presented in your preferred language, which will be the first you listed when configuring the available languages. If you open `transloco-root.module.ts` you will see that a default language is set there. What we actually want though is to choose it dinamically, depending on which language is set in the user's browser configuration.

For that, we will need to get the browser's language configuration and use it to decide in which language to present the page. Since we are also going to do SSR, we will need to get the language from the request headers, since we don't have access to the user's browser configuration.

We will use Angular's dependency injection to properly fetch the preferred user's language, for the browser part and for the server side.

##### Fetching Language from browser config

Create a file called `locale-lang-config.ts` inside `src/app`. We are going to use this file to list all our supported locales and languages, which we will also reference from `transloco-root.module.ts`.

```typescript
// this configuration will be injectable by app.component.ts
export class LocaleConfig {
  constructor(public language: string, public locale: string) {}
}

// some locale and language configuration for all the available languages our website supports
export const DEFAULT_LANG = "pt";
export const DEFAULT_LOCALE = "pt-BR";
export const SUPPORTED_LANGUAGES = [
  { language: "pt", locales: ["pt-BR", "pt-PT"] },
  { language: "en", locales: ["en-US", "en-GB"] }
];

// This factory is what will create the LocaleConfig to be used when
// the app is running in our browser (so not in our Server-side rendered page)
// Notice the reference to window object, which is only available when
// we are running in a browser
export const browserLocaleFactory: () => LocaleConfig = () => {
  if (
    typeof window === "undefined" ||
    typeof window.navigator === "undefined"
  ) {
    throw new Error("Fetching locale failed. Are you really in a browser??");
  }
  const wn = window.navigator as any;
  let locale = wn.languages ? `${wn.languages[0]}` : DEFAULT_LOCALE;
  locale = locale || wn.language || wn.browserLanguage || wn.userLanguage;
  const language = locale.split("-")[0];

  return new LocaleConfig(language, locale);
};
```

In `transloco-root.module.ts`, change `availableLangs` and `defaultLangs` values to point to the ones created in `locale-lang-config.ts`, like so:

```typescript
[...]
@NgModule({
  exports: [TranslocoModule],
  providers: [
    {
      provide: TRANSLOCO_CONFIG,
      useValue: translocoConfig({
        availableLangs: SUPPORTED_LANGUAGES.map(lang => lang.language),
        defaultLang: DEFAULT_LANG,
        reRenderOnLangChange: true,
        prodMode: environment.production
      })
    },
    { provide: TRANSLOCO_LOADER, useClass: TranslocoHttpLoader }
  ]
})
export class TranslocoRootModule {}
```

Now in you `app.module.ts` file, add a provider configuration, which will provide our `LocaleConfig` using the `browserLocaleFactory` we defined in `locale-lang-config.ts`:

```typescript
  [...]
  bootstrap: [AppComponent],
  providers: [
    {
      provide: LocaleConfig,
      useFactory: browserLocaleFactory
    }
  ]
})
export class AppModule {}
```

Finally, to have the app presented using the browser language config, add this constructor to `app.component.ts`:

```typescript
  constructor(transloco: TranslocoService, localeConf: LocaleConfig) {
    transloco.setActiveLang(localeConf.language);
  }
```

If everything went according to plan, when opening the page at [http://localhost:4200](http://localhost:4200) you should see the page in the language configured in your browser. If you want to test changing the language in firefox:

![](/assets/images/posts/angular-ssr-transloco/change-lang-sm.gif)

##### Fetching Language from request headers (SSR part)

Just as an experiment, let's try to run this using Server-side Rendering.

```bash
npm run build:ssr && npm run serve:ssr
```

If we now head over to [http://localhost:4000](http://localhost:4000), we get an internal server error. Looking at the logs, we will see that we get

```cmd
Node Express server listening on [http://localhost:4000](http://localhost:4000)
ERROR Error: Fetching locale failed. Are you really in a browser??
```

This happens because browserLocaleFactory is being used to provide the `LocaleConfig`. This won't work because we need to provide a serverLocalFactory for the SSR part. We will also need to change a bit the way we provide these factories.

Let's add a new function called `serverLocalFactory` to our `locale-lang-config.ts` file:

```typescript
[...]
export const serverLocaleFactory: (locale?: string) => () => LocaleConfig = (reqLocales?: string) => () => {
  if (!reqLocales) {
    return new LocaleConfig(DEFAULT_LANG, DEFAULT_LOCALE);
  }

  // try setting locale according to list of locales sent to us. Try finding the first one of the list - since
  // this will probably the preferred client's language
  const localeFound: string | undefined = reqLocales
    .split(new RegExp(',|;'))
    .find(reqLocale => SUPPORTED_LANGUAGES.find(lang => reqLocale.includes(lang.language)));

  if (localeFound) {
    // From the iteration above we only arrive here if a language was found (that's why the bang and tslint disable)
    // tslint:disable-next-line: no-non-null-assertion
    const foundLangauge = SUPPORTED_LANGUAGES.find(lang => localeFound.includes(lang.language))!;
    const supportedLocale = foundLangauge.locales.find(locale => locale === localeFound);
    return new LocaleConfig(foundLangauge.language, supportedLocale || foundLangauge.locales[0]);
  } else {
    return new LocaleConfig(DEFAULT_LANG, DEFAULT_LOCALE);
  }
};
```

This new factory will have to be provided in the `/server.ts` express server file - found in the root folder of our project - which was created for us when we added support for SSR. The reason to add it in this file is that we will be able to extract the language from the request headers sent by the user's browser. Find this code block:

```typescript
[...]
  // All regular routes use the Universal engine
  server.get('*', (req, res) => {
    res.render(indexHtml, { req, providers: [{ provide: APP_BASE_HREF, useValue: req.baseUrl }] });
  });
[...]
```

and substitute it by adding the `serverLocaleFactory` as a provider for `LocaleConfig`, like so:

```typescript
  [...]
  // All regular routes use the Universal engine
  server.get('*', (req, res) => {
    res.render(indexHtml, {
      req,
      providers: [
        { provide: APP_BASE_HREF, useValue: req.baseUrl },
        { provide: LocaleConfig, useFactory: serverLocaleFactory(req.headers['accept-language']) }
      ]
    });
  });
  [...]
```

This still won't do the trick. When we try to access the website using SSR, we will now have 2 distinct providers for `LocaleConfig` since `AppModule` also already defines a provider for `LocaleConfig` - look at `AppServerModule`: it references `AppModule`.

So the last step is to separate client and server modules so that each declares a single locale factory. To do so, create an `app.client.module.ts` file under `src/app` with the following content:

```typescript
import { NgModule } from "@angular/core";
import { AppComponent } from "./app.component";
import { AppModule } from "./app.module";
import { browserLocaleFactory, LocaleConfig } from "./locale-lang-config";

@NgModule({
  imports: [AppModule],
  providers: [
    {
      provide: LocaleConfig,
      useFactory: browserLocaleFactory
    }
  ],
  bootstrap: [AppComponent]
})
export class AppClientModule {}
```

Now in `src/main.ts`, instead of bootstrapping the `AppModule`, we will bootstrap `AppClientModule`, like so:

```typescript
[...]
document.addEventListener('DOMContentLoaded', () => {
  platformBrowserDynamic()
    .bootstrapModule(AppClientModule)
    .catch(err => console.error(err));
});
```

We may now remove the `browserLocaleFactory` from our `AppModule`. It should look like this:

```typescript
@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule.withServerTransition({ appId: "serverApp" }),
    AppRoutingModule,
    HttpClientModule,
    TranslocoRootModule
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

As a last step, we need to change `src/environments/environment.prod.ts`. When we added Transloco, the `ng add` included a `baseURL` configuration property to both `src/environments/environment.ts` and `src/environments/environment.prod.ts`. The configuration for the production environment needs some special attention: when running the website in SSR mode, Transloco will download translation files dinamically from the URL that is set in `baseUrl` property; for local tests, I had to change mine to

```typescript
export const environment = {
  baseUrl: "http://localhost:4000",
  production: true
};
```

where express server port is now 4000 (when running the app in SSR mode). Also **note that this baseUrl will have to match your domain name once you deploy your app**, otherwise Transloco will not be able to retrieve translations!

FINALLY! After much sweat and tears, and if everything went well, you can start the server with `npm run build:ssr && npm run serve:ssr` and test that all works by navigating to [http://localhost:4000](http://localhost:4000). The page should render instantly with the correct language configuration.

## Add Angular Material

Add Angular material by running

```bash
ng add @angular/material@next
```

I've also used `next` version here, just to be consistent with versioning. Progress through all questions with your choices (I also chose to have global fonts enabled, since this applies roboto font to the whole page). If you don't want roboto applied to the body of your page, just stick with the defaults.

Now let's add a new reactive form in our app.component.html, with some translated fields:

{% assign buttonTranslate = '{{ t("app.form.submit") }}' %}

```html
<form class="example-form" [formGroup]="form" *transloco="let t">
  <mat-form-field class="full-width">
    <textarea
      matInput
      [placeholder]="t('app.form.address')"
      formControlName="address"
    ></textarea>
  </mat-form-field>

  <table class="full-width" cellspacing="0">
    <tr>
      <td>
        <mat-form-field class="full-width">
          <input
            matInput
            [placeholder]="t('app.form.city')"
            formControlName="city"
          />
        </mat-form-field>
      </td>
      <td>
        <mat-form-field class="full-width">
          <input
            matInput
            [placeholder]="t('app.form.state')"
            formControlName="state"
          />
        </mat-form-field>
      </td>
    </tr>
  </table>

  <button type="submit" mat-button>{{ buttonTranslate }}</button>
</form>
```

For this to work, we need to add some imports to our `AppModule`,

```typescript
  imports: [
    [...]
    ReactiveFormsModule,
    MatFormFieldModule,
    MatButtonModule,
    MatInputModule,
  ]
```

and we also create the translations for pt and en

```json
{
  "app": {
    "form": {
      "address": "Endereço",
      "city": "Cidade",
      "state": "Estado",
      "submit": "Enviar"
    }
  }
}
```

```json
{
  "app": {
    "form": {
      "address": "Address",
      "city": "City",
      "state": "State",
      "submit": "Submit"
    }
  }
}
```

We must not forget to also initialize our FormGroup in app.component.ts:

```typescript
export class AppComponent implements OnInit {
  form: FormGroup;

  constructor(
    transloco: TranslocoService,
    localeConf: LocaleConfig,
    private fb: FormBuilder
  ) {
    transloco.setActiveLang(localeConf.language);
  }
  ngOnInit(): void {
    this.form = this.fb.group({
      address: [""],
      city: [""],
      state: [""]
    });
  }
}
```

And that's it, we now have Material fields with translated names.

## Translate form field validation errors with Transloco

Now to the final piece of the puzzle: translating field validation errors. To make this work, we will need a pipe operator, to transform the `ValidationErrors` that are returned from each validator into something that can be used as input for Transloco. Create a new pipe operator by using angular cli:

```bash
ng g p pipes/translate-error --module app
```

with the following content

```typescript
import { Pipe, PipeTransform } from "@angular/core";
import { ValidationErrors } from "@angular/forms";

@Pipe({
  name: "i18nErr"
})
export class TranslateErrorPipe implements PipeTransform {
  transform(
    errors: ValidationErrors | null,
    i18nFunc: (i18nKey: string, obj?: { [key: string]: any }) => string
  ): string {
    if (!errors) {
      return "";
    }
    const errorKeys = Object.keys(errors);
    if (errorKeys.length === 0) {
      return "";
    }
    const errorKey = errorKeys[0];
    const errorDetails = errors[errorKey];

    if (typeof errorDetails === "object") {
      return i18nFunc(errorKey, errorDetails);
    }
    return i18nFunc(errorKey);
  }
}
```

Our new pipe will take two arguments; the first argument takes all `ValidationErrors` a field will output when in error. If we look at the definition of Angular's ValidationError interface:

```typescript
export interface ValidationError {
  [key: string]: any;
}
```

We see that there might be multiple errors, represented as keys in this interface. The `any` type on the right side might contain an arbitrary object detailing the error. For instance, if we define a `Validators.maxLength(3)` for a field and the user inputs more than 3 chars, the field will have following error:

```json
{
  "maxlength": {
    "requiredLength": 3,
    "actualLength": 6
  }
}
```

The second pipe argument is Transloco's `translate` function (named `i18nFunc` in this pipe). This function accepts a translation key and an optional additional object used for interpolating the string we are translating; we can use the additional information we receive in the error object to complete our translation [with additional information](https://netbasal.gitbook.io/transloco/translation-in-the-template/structural-directive).

Ultimately, what our pipe will do is to find the first error (if any) from the `ValidationErrors` argument and extract the first key and the value it finds for that key. The key will be passed as an argument to `i18nFunc` function, and if there is also a value object for the error key, it will be passed as the optional argument for the `i18nFunc` function

This sounds confusing right now but it'll make sense in a minute when we add translations for our validations.

Let's add some validations in app.component.ts then:

```typescript
  ngOnInit(): void {
    this.form = this.fb.group({
      address: [''],
      city: ['', [Validators.required]],
      state: ['', [Validators.required, Validators.maxLength(20), Validators.minLength(3)]]
    });
  }
```

And in the template, display the error messages by adding a mat-error right under each input tag
{% assign errorPipe = '{{ errors | i18nErr: t }}' %}

```html
    [...]
        <mat-form-field class="full-width">
          <input matInput [placeholder]="t('app.form.city')" formControlName="city" />
          <mat-error *ngIf="form.controls.city.errors as errors">{{ errorPipe }}</mat-error>
        </mat-form-field>
      </td>
      <td>
        <mat-form-field class="full-width">
          <input matInput [placeholder]="t('app.form.state')" formControlName="state" />
          <mat-error *ngIf="form.controls.state.errors as errors">{{ errorPipe }}</mat-error>
        </mat-form-field>
      </td>
    </tr>
  </table>
  [...]
```

Lets add translations for `required`, `minlength` and `maxlength`, which are the keys of `ValidationErrors` we might get when we have these errors present in our form to pt.json and en.json.

```json
  [...]
  "required": "Campo mandatŕio",
  "minlength": "Deve ter no mínimo {{ requiredLength }} caracteres",
  "maxlength": "Deve ter no máximo {{ requiredLength }} caracteres"
```

```json
  [...]
  "required": "Value required",
  "minlength": "Must have at least {{ requiredLength }} characters",
  "maxlength": "Cannot have more than {{ requiredLength }} characters"
```

In the translation files, you can see we are using string interpolation for the `requiredLength`, which is part of `ValidationError` details when we have a `minlength` or `maxlength` validation error.

And that's it, we are finally done with this huge tutorial!

## Wrapping up

Hopefully this tutorial didn't melt your brain. Having translations and SSR configured properly right at the beginning of a new project is extremely beneficial down the road since having to change all this later can be very frustrating and time consuming!

If you want to have a look at the fully working example, please check it out at [https://github.com/martinreus/ssr-translate](https://github.com/martinreus/ssr-translate).

If you found an issue or typos in this tutorial, please kindly let me know by opening an issue here -> [https://github.com/martinreus/martinreus.github.io/issues](https://github.com/martinreus/martinreus.github.io/issues)

Thanks for reading and see you in the next one! Cheers!
