---
layout: post
title: Angular async validator on reactive forms with keystroke throttling and real time password strength meter
date: 2020-02-10 14:35
category: [Software Engineering, Angular]
author: martin
tags: [Angular, Async, Validation, Throttling]
# summary:
image: assets/images/posts/angular.svg
featured: false
hidden: false
---

In this tutorial we are going to learn how to implement an [Angular async](https://angular.io/guide/form-validation#async-validation) validator to validate a password field with a call to a backend service, while also throttling user keystrokes and showing on a progress bar how good the password is.

Some familiarity with npm and Angular is assumed here :)

## Setup

Create a new angular project by running

```bash
# if you don't have angular cli, install it first
# npm install -g @angular/cli
ng new async-thrott-val
cd async-thrott-val
ng add @angular/material
```

For the questions that will pop up during project generation, default should be fine.

## Creating the form

Lets add a password input field, a progress bar for showing how good the chosen password is and a button to our `app.component.hml`.
Erase everything you find in said file and paste this

{% assign escapedError = '{{ error }}' %}

```html
<div class="container">
  <form [formGroup]="form">
    <mat-form-field class="form-field">
      <input matInput formControlName="password" type="password" />
      <mat-error *ngIf="form.controls.password.errors">Weak password</mat-error>
    </mat-form-field>
    <mat-progress-bar
      [value]="passScoreBar$ | async"
      [color]="form.controls.password.errors && form.controls.password.touched ? 'warn': 'primary'"
    ></mat-progress-bar>
    <button mat-raised-button type="submit" color="primary">Submit</button>
  </form>
</div>
```

Add some styling to `app.component.css` just to see things better

```css
.container {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

form {
  width: 500px;
}

.form-field {
  width: 100%;
}
```

In `app.module.ts` you'll also need to add the proper material and reactive form modules

```typescript
@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    ReactiveFormsModule,
    MatProgressBarModule,
    MatFormFieldModule,
    MatInputModule,
    MatButtonModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

In `AppComponent`, paste the base for our reactive form:

```typescript
import { Component, ChangeDetectionStrategy, OnInit } from "@angular/core";
import { FormGroup, FormBuilder } from "@angular/forms";
import { Observable } from "rxjs";

@Component({
  selector: "app-root",
  templateUrl: "./app.component.html",
  styleUrls: ["./app.component.css"],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class AppComponent implements OnInit {
  form: FormGroup;
  passScoreBar$: Observable<number>;

  constructor(private fb: FormBuilder) {}

  ngOnInit(): void {
    this.form = this.fb.group({
      password: [""]
    });
  }
}
```

Everything should be runnable with `ng serve`. If it does not work, check all the steps again.

## Creating our backend service and validator

Our backend service will be used to validate a score for a given password. To simplify this tutorial, we are not really going to call a real back end. Instead, we are going to implement a mock service that will just evaluate a password based on its length. Our validator will then call this service.

Create a new service by running `ng g s password`. The code for it will be

```typescript
import { Injectable } from "@angular/core";
import { Observable, of } from "rxjs";
import { map, delay, tap } from "rxjs/operators";

@Injectable({
  providedIn: "root"
})
export class PasswordService {
  // define the minimum safe score for a password to be used
  static MIN_PASSWORD_SCORE = 8;

  constructor() {}

  // return a password score
  getPasswordScore(password: string | undefined): Observable<number> {
    password = password || "";

    return of(password.length).pipe(
      // output to console to show that our 'request' is running
      tap(_ => console.log("going to request backend for ", password))
    );
  }
}
```

Now let's create our validator. Create a file inside the app folder called `async-pass.validator.ts` and paste this inside:

```typescript
import { Injectable } from "@angular/core";
import {
  AbstractControl,
  AsyncValidator,
  ValidationErrors
} from "@angular/forms";
import { Observable, of, Subject, timer } from "rxjs";
import { map, switchMap, tap } from "rxjs/operators";
import { PasswordService } from "./password.service";

@Injectable({ providedIn: "root" })
export class AsyncPassValidator implements AsyncValidator {
  constructor(private pwService: PasswordService) {}

  validate(
    control: AbstractControl
  ): Promise<ValidationErrors | null> | Observable<ValidationErrors | null> {
    return this.pwService.getPasswordScore(control.value).pipe(
      map(score => {
        // if password score is below threshold, validation will fail
        if (score < PasswordService.MIN_PASSWORD_SCORE) {
          return { unsafe: true };
        }
        // otherwise, no errors
        return null;
      })
    );
  }
}
```

and update your `app.component.ts` by adding the async password validator to the form builder code:

```typescript
[...]
  constructor(private fb: FormBuilder, private pwValidator: AsyncPassValidator) {}

  ngOnInit(): void {
    this.form = this.fb.group({
      password: ['', [], [this.pwValidator.validate.bind(this.pwValidator)]]
    });
  }
[...]
```

Now if you run this example with `ng s`, you should see that when inputting a password and then 'submitting' the form, an error will appear under the input box. If you open the browser inspection tools, you will also see that for every key stroke, a new 'request' is being made to the backend (as a console log output), which would spam our server (if we were communicating with one) with requests. Also, the user has no active feedback on how good the password he is typing is. So let's add some throttling and feedback with that progress bar we already have in our html file.

## Adding throttling and password score feedback

Angular async validators will run either immediately for every keystroke, on form submit, or when the input field loses focus, depending on [how you configure the validator](https://angular.io/guide/form-validation#note-on-performance). We are going to reach a middle point here, were we react to every keypress, but throttle user input so that we don't spam the server, while still allowing feedback about what was input into the field without having to submit or lose input focus.

Change the validator's `validate` implementation to only start fetching from service after 500ms

```typescript
  validate(
    control: AbstractControl
  ): Promise<ValidationErrors | null> | Observable<ValidationErrors | null> {
    return timer(500).pipe(
      switchMap(_ => this.pwService.getPasswordScore(control.value)),
      map(score => {
        // if password score is below threshold, validation will fail
        if (score < PasswordService.MIN_PASSWORD_SCORE) {
          return { unsafe: true };
        }
        // otherwise, no errors
        return null;
      })
    );
  }
```

What will happen here is that with a new control value change, our FormControl will unsubscribe from the previous Observable that was awaiting completion. Therefore, the old Observable will not emit a value, hence no backend call will be made. This logic can be better understood with an image:

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="651px" viewBox="-0.5 -0.5 651 491" style="max-width:100%;max-height:491px;"><defs/><g><rect x="10" y="110" width="640" height="240" fill="#ffffff" stroke="#000000" pointer-events="all"/><ellipse cx="40" cy="40" rx="40" ry="40" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 78px; height: 1px; padding-top: 40px; margin-left: 1px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: normal; word-wrap: normal; ">keystroke</div></div></div></foreignObject><text x="40" y="44" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">keystroke</text></switch></g><path d="M 40 170 L 173.63 170" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 178.88 170 L 171.88 173.5 L 173.63 170 L 171.88 166.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 170px; margin-left: 110px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 11px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; background-color: #ffffff; white-space: nowrap; ">timer(500)</div></div></div></foreignObject><text x="110" y="173" fill="#000000" font-family="Helvetica" font-size="11px" text-anchor="middle">timer(500)</text></switch></g><path d="M 40 80 L 40 163.63" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 40 168.88 L 36.5 161.88 L 40 163.63 L 43.5 161.88 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><rect x="270" y="90" width="40" height="20" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 100px; margin-left: 271px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: normal; word-wrap: normal; ">AsyncValidator</div></div></div></foreignObject><text x="290" y="104" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">AsyncV...</text></switch></g><rect x="10" y="400" width="640" height="90" fill="#ffffff" stroke="#000000" pointer-events="all"/><rect x="12" y="402" width="636" height="86" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 634px; height: 1px; padding-top: 445px; margin-left: 13px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: normal; word-wrap: normal; ">FormControl (app.component.ts)</div></div></div></foreignObject><text x="330" y="449" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">FormControl (app.component.ts)</text></switch></g><path d="M 42.64 400.36 L 40.07 176.37" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 40.01 171.12 L 43.59 178.08 L 40.07 176.37 L 36.59 178.16 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 336px; margin-left: 53px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 11px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; background-color: #ffffff; white-space: nowrap; ">subscribe</div></div></div></foreignObject><text x="53" y="340" fill="#000000" font-family="Helvetica" font-size="11px" text-anchor="middle">subscribe</text></switch></g><path d="M 500 310 L 500 395.63" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 500 400.88 L 496.5 393.88 L 500 395.63 L 503.5 393.88 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 370px; margin-left: 500px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 11px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; background-color: #ffffff; white-space: nowrap; "><div>password score (with unsubscription)</div></div></div></div></foreignObject><text x="500" y="373" fill="#000000" font-family="Helvetica" font-size="11px" text-anchor="middle">password s...</text></switch></g><path d="M 390 270 L 453.63 270" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 458.88 270 L 451.88 273.5 L 453.63 270 L 451.88 266.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 270px; margin-left: 426px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 11px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; background-color: #ffffff; white-space: nowrap; ">pipe</div></div></div></foreignObject><text x="426" y="273" fill="#000000" font-family="Helvetica" font-size="11px" text-anchor="middle">pipe</text></switch></g><ellipse cx="350" cy="270" rx="40" ry="40" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 78px; height: 1px; padding-top: 270px; margin-left: 311px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: normal; word-wrap: normal; ">emit</div></div></div></foreignObject><text x="350" y="274" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">emit</text></switch></g><ellipse cx="500" cy="270" rx="40" ry="40" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 78px; height: 1px; padding-top: 270px; margin-left: 461px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: normal; word-wrap: normal; "><div>password</div><div>service</div><div>call<br /></div></div></div></div></foreignObject><text x="500" y="274" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">password...</text></switch></g><path d="M 155 195 L 205 145" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 205 193 L 155 143" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><ellipse cx="150" cy="40" rx="40" ry="40" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 78px; height: 1px; padding-top: 40px; margin-left: 111px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: normal; word-wrap: normal; "><div>new keystroke</div><div>(less than 500ms)<br /></div></div></div></div></foreignObject><text x="150" y="44" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">new keystroke...</text></switch></g><path d="M 150 80 L 150.15 263.71" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 150.16 268.96 L 146.65 261.96 L 150.15 263.71 L 153.65 261.96 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><path d="M 150 270 L 303.63 270" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 308.88 270 L 301.88 273.5 L 303.63 270 L 301.88 266.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 270px; margin-left: 230px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 11px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; background-color: #ffffff; white-space: nowrap; ">timer(500)</div></div></div></foreignObject><text x="230" y="273" fill="#000000" font-family="Helvetica" font-size="11px" text-anchor="middle">timer(500)</text></switch></g><path d="M 182.8 398.65 L 180.08 176.37" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 180.01 171.12 L 183.6 178.07 L 180.08 176.37 L 176.6 178.16 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 300px; margin-left: 200px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 11px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; background-color: #ffffff; white-space: nowrap; ">unsubscribe</div></div></div></foreignObject><text x="200" y="304" fill="#000000" font-family="Helvetica" font-size="11px" text-anchor="middle">unsubscribe</text></switch></g><path d="M 150.16 400 L 150.01 276.37" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 150 271.12 L 153.51 278.11 L 150.01 276.37 L 146.51 278.12 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 334px; margin-left: 150px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 11px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; background-color: #ffffff; white-space: nowrap; ">subscribe</div></div></div></foreignObject><text x="150" y="337" fill="#000000" font-family="Helvetica" font-size="11px" text-anchor="middle">subscribe</text></switch></g></g><switch><g requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"/><a transform="translate(0,-5)" xlink:href="https://desk.draw.io/support/solutions/articles/16000042487" target="_blank"><text text-anchor="middle" font-size="10px" x="50%" y="100%">Viewer does not support full SVG 1.1</text></a></switch></svg>

Now we just need to show how good the password is while the user types his password. For that, we are going to emit a score value as soon as a call is made to our "backend", which can then be used as a value in the progress bar. We need to export a `Subject` which will emit the score to be read from `app.component.ts`.

Let's change the `validate` function again to emit a value every time a request is successfully sent to the backend. For that, add a private variable to `AsyncPassValidator`:

```typescript
  private scoreEmitter$ = new Subject<number>();
```

and then also create a getter for this subject:

```typescript

  get score(): Observable<number> {
    return this.scoreEmitter$;
  }
```

Finally, every time a score is returned from our backend, we will emit it using `rxjs`s `tap` operator. Right after `switchMap(_ => this.pwService.getPasswordScore(control.value)),` introduce a new line with the `tap` operator:

```typescript
      switchMap(_ => this.pwService.getPasswordScore(control.value)),
      tap(score => this.scoreEmitter$.next(score)),
```

Now in our `app.component.ts`, inside `ngOnInit` and after the initialization of our form code block, we will insert some code to use the value we emitted from the validator as input for our `passScoreBar$`. We will need to normalize the password score (which ranges from 0 to 20) to a value that can be used for `MatProgressBar`, which ranges from 0 to 100.

```typescript
this.passScoreBar$ = this.pwValidator.score.pipe(
  map(score => {
    // first, normalize value for minimum needed password strength.
    score =
      score > PasswordService.MIN_PASSWORD_SCORE
        ? PasswordService.MIN_PASSWORD_SCORE
        : score;

    // then, normalize for progress bar value
    return score * (100 / PasswordService.MIN_PASSWORD_SCORE);
  })
);
```

And that should do the trick! If you head over to [http://localhost:4200](http://localhost:4200) you should see the progress bar updating after the keystrokes stop for longer than 500ms.

## Wrapping up

It can be very frustrating having to enter a lot of data into a form just to find out later on submit that something is missing or wrong.
Asynchronous validations are great for giving real time feedback for users when they are typing something into a form that is going to be submitted. Hopefully this can help you implement a similar validation while also sparing your precious server resources to not be fully spammed by the user :)
If you want to check the code for this tutorial, head over to [the Github repository](https://github.com/martinreus/async-throttled-validation).
