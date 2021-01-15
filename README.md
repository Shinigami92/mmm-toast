# mmm-toast [![npm version](https://badge.fury.io/js/mmm-toast.svg)](https://badge.fury.io/js/mmm-toast) [![npm monthly downloads](https://img.shields.io/npm/dm/mmm-toast.svg?style=flat-square)](https://www.npmjs.com/package/mmm-toast)

An Angular toast component that shows growl-style alerts and messages for your application.
This is a continuation of the legacy previously championed by [ngx-toasta](https://github.com/emonney/ngx-toasta)
and before that [ng2-toasty](https://github.com/akserg/ng2-toasty) with the latest package versions, complete refactoring from the ground-up, and additional enhancements.

## Installation

```sh
npm i mmm-toast
```

## Demo

Online demo available [here](https://stevewhitmore.github.io/mmm-toast)

## Usage

### Table of Contents

- [Setup](#setup)
- [Models](#models)
- [Methods](#methods)
- [Examples](#examples)

### Setup

#### 1. Update the styles

- Import style into your web page. Choose one of the following files;
  - `style-default.css` - Contains DEFAULT theme
  - `style-bootstrap.css` - Contains Bootstrap 3 theme
  - `style-material.css` - Contains Material Design theme

  ```css
  // styles.css/styles.scss - you really only need to import the one you'll use
  @import "~mmm-toast/lib/styles/style-default.css";
  @import "~mmm-toast/lib/styles/style-bootstrap.css";
  @import "~mmm-toast/lib/styles/style-material.css";
  ```

- Assign the selected theme name [`default`, `bootstrap`, `material`] to the `theme` property of the instance of ToastaConfig.
- Add `<mmm-toast></mmm-toast>` tag in template of your application component.

#### 2. Import the `MmmToastModule`

Import `MmmToastModule` in the NgModule of your application.
The `forRoot` method is a convention for modules that provide a singleton service.

```ts
import {BrowserModule} from "@angular/platform-browser";
import {NgModule} from '@angular/core';
import {MmmToastModule} from 'mmm-toast';
import {AppComponent} from './app.component';

@NgModule({
    imports: [
        BrowserModule,
        MmmToastModule.forRoot()
    ],
    bootstrap: [AppComponent]
})
export class AppModule {
}
```

If you have multiple NgModules and you use one as a shared NgModule (that you import in all of your other NgModules),
don't forget that you can use it to export the `MmmToastModule` that you imported in order to avoid having to import it multiple times.

```ts
@NgModule({
    imports: [
        BrowserModule,
        MmmToastModule.forRoot()
    ],
    exports: [BrowserModule, MmmToastModule],
})
export class SharedModule {
}
```

#### 3. Use the `MmmToastService` for your application

### Models

```js
export interface GlobalConfigModel {
  id?: number;
  title?: string;
  showClose?: boolean;
  showDuration?: boolean;
  theme?: string;
  timeout?: number;
  position?: string;
  limit?: number;
  isCountdown?: boolean;
}
```

```js
export interface ToastModel extends GlobalConfigModel {
  type: string;
  message: string;
}
```

### Methods

#### MmmToastService

`receiveGlobalConfigs(configs: GlobalConfigModel): void {}`

Allows you to set global properties for your toasts so you don't have to set the same properties
on toasts passed in over and over. These properties are overwritten by whatever you pass into 
`addToast()`.

`removeToast(toastId: number): void {}`

Will remove specific toast message you click the closing x of.

`clearAll(): void {}`

Clears out all toast messages.

`clearLast(): void {}`

Clears the last toast message to appear (LIFO so it goes from the bottom up).

`addToast(toast: ToastModel): void {}`

At minimum the `type` and `message` properties are required. Takes the ToastModel object
passed in, overwrites whatever was set as a global property. It then checks for any properties
not set and sets them to default values. After that it makes sure the limit is not surpassed and 
then the toast is added to the DOM.

### Examples

- Import `MmmToastService` from `mmm-toast` in your application code:

```js
import {Component} from '@angular/core';
import {MmmToastService, ToastaConfig, ToastOptions, ToastData} from 'mmm-toast';

@Component({
    selector: 'app-root',
    template: `
        <div>Hello world</div>
        <button (click)="addToast()">Add Toast</button>
        <mmm-toast></mmm-toast>
    `
})
export class AppComponent {

    constructor(private MmmToastService: MmmToastService, private ToastaConfig: ToastaConfig) {
        // Assign the selected theme name to the `theme` property of the instance of ToastaConfig.
        // Possible values: default, bootstrap, material
        this.ToastaConfig.theme = 'material';
    }

    addToast() {
        // Just add default Toast with title only
        this.MmmToastService.default('Hi there');
        // Or create the instance of ToastOptions
        var toastOptions: ToastOptions = {
            title: "My title",
            msg: "The message",
            showClose: true,
            timeout: 5000,
            theme: 'default',
            onAdd: (toast: ToastData) => {
                console.log('Toast ' + toast.id + ' has been added!');
            },
            onRemove: function(toast: ToastData) {
                console.log('Toast ' + toast.id + ' has been removed!');
            }
        };
        // Add see all possible types in one shot
        this.MmmToastService.info(toastOptions);
        this.MmmToastService.success(toastOptions);
        this.MmmToastService.wait(toastOptions);
        this.MmmToastService.error(toastOptions);
        this.MmmToastService.warning(toastOptions);
    }
}
```

### 4. How to dynamically update title and message of a toast

Here is an example of how to dynamically update message and title of individual toast:

```js
import {Component} from '@angular/core';
import {MmmToastService, ToastaConfig, ToastComponent, ToastOptions, ToastData} from 'mmm-toast';
import {Subject, Observable, Subscription} from 'rxjs/Rx';

@Component({
    selector: 'app',
    template: `
        <div>Hello world</div>
        <button (click)="addToast()">Add Toast</button>
        <mmm-toast></mmm-toast>
    `
})
export class AppComponent {

    getTitle(num: number): string {
        return 'Countdown: ' + num;
    }

    getMessage(num: number): string {
        return 'Seconds left: ' + num;
    }

    constructor(private MmmToastService: MmmToastService) { }

    addToast() {
        let interval = 1000;
        let timeout = 5000;
        let seconds = timeout / 1000;
        let subscription: Subscription;

        let toastOptions: ToastOptions = {
            title: this.getTitle(seconds),
            msg: this.getMessage(seconds),
            showClose: true,
            timeout: timeout,
            onAdd: (toast: ToastData) => {
                console.log('Toast ' + toast.id + ' has been added!');
                // Run the timer with 1 second iterval
                let observable = Observable.interval(interval).take(seconds);
                // Start listen seconds beat
                subscription = observable.subscribe((count: number) => {
                    // Update title of toast
                    toast.title = this.getTitle(seconds - count - 1);
                    // Update message of toast
                    toast.msg = this.getMessage(seconds - count - 1);
                });

            },
            onRemove: function(toast: ToastData) {
                console.log('Toast ' + toast.id + ' has been removed!');
                // Stop listenning
                subscription.unsubscribe();
            }
        };

        switch (this.options.type) {
            case 'default': this.MmmToastService.default(toastOptions); break;
            case 'info': this.MmmToastService.info(toastOptions); break;
            case 'success': this.MmmToastService.success(toastOptions); break;
            case 'wait': this.MmmToastService.wait(toastOptions); break;
            case 'error': this.MmmToastService.error(toastOptions); break;
            case 'warning': this.MmmToastService.warning(toastOptions); break;
        }
    }
}
```

### 5. How to close specific toast

Here is an example of how to close an individual toast:

```js
import {Component} from '@angular/core';
import {MmmToastService, ToastaConfig, ToastComponent, ToastOptions, ToastData} from 'mmm-toast';
import {Subject, Observable, Subscription} from 'rxjs/Rx';

@Component({
    selector: 'app',
    template: `
        <div>Hello world</div>
        <button (click)="addToast()">Add Toast</button>
        <mmm-toast></mmm-toast>
    `
})
export class AppComponent {

    getTitle(num: number): string {
        return 'Countdown: ' + num;
    }

    getMessage(num: number): string {
        return 'Seconds left: ' + num;
    }

    constructor(private MmmToastService:MmmToastService) { }

    addToast() {
        let interval = 1000;
        let subscription: Subscription;

        let toastOptions: ToastOptions = {
            title: this.getTitle(0),
            msg: this.getMessage(0),
            showClose: true,
            onAdd: (toast: ToastData) => {
                console.log('Toast ' + toast.id + ' has been added!');
                // Run the timer with 1 second iterval
                let observable = Observable.interval(interval);
                // Start listen seconds beat
                subscription = observable.subscribe((count: number) => {
                    // Update title of toast
                    toast.title = this.getTitle(count);
                    // Update message of toast
                    toast.msg = this.getMessage(count);
                    // Extra condition to hide Toast after 10 sec
                    if (count > 10) {
                        // We use toast id to identify and hide it
                        this.MmmToastService.clear(toast.id);
                    }
                });

            },
            onRemove: function(toast: ToastData) {
                console.log('Toast ' + toast.id + ' has been removed!');
                // Stop listenning
                subscription.unsubscribe();
            }
        };

        switch (this.options.type) {
            case 'default': this.MmmToastService.default(toastOptions); break;
            case 'info': this.MmmToastService.info(toastOptions); break;
            case 'success': this.MmmToastService.success(toastOptions); break;
            case 'wait': this.MmmToastService.wait(toastOptions); break;
            case 'error': this.MmmToastService.error(toastOptions); break;
            case 'warning': this.MmmToastService.warning(toastOptions); break;
        }
    }
}
```

### 6. Customize the `mmm-toast` for your application in template

You can use the following properties to customize the mmm-toast component in your template:

- `position` - The window position where the toast pops up. Default value is `bottom-right`. Possible values: `bottom-right`, `bottom-left`, `bottom-fullwidth` `top-right`, `top-left`, `top-fullwidth`,`top-center`, `bottom-center`, `center-center`
Example:

```html
<mmm-toast [position]="'top-center'"></mmm-toast>
```

### 7. Options

Use these options to configure individual or global toasts

Options specific to an individual toast:

```js
ToastOptions
{
    "title": string,      //A string or html for the title
    "msg": string,        //A string or html for the message
    "showClose": true,    //Whether to show a close button
    "showDuration": true, //Whether to show a progress bar
    "theme": "default",   //The theme to apply to this toast
    "timeout": 5000,      //Time to live until toast is removed. 0 is unlimited
    "onAdd": Function,    //Function that gets called after this toast is added
    "onRemove": Function  //Function that gets called after this toast is removed
}
```

Configurations that affects all toasts:

```js
ToastaConfig
{
    "limit": 5,                 //Maximum toasts that can be shown at once. Older toasts will be removed. 0 is unlimited
    "showClose": true,          //Whether to show the 'x' icon to close the toast
    "showDuration": true,       //Whether to show a progress bar at the bottom of the notification
    "position": "bottom-right", //The window position where the toast pops up
    "timeout": 5000,            //Time to live in milliseconds. 0 is unlimited
    "theme": "default"          //What theme to use
}
```

## Credits

Original work by [ng2-toasta](https://github.com/akserg/ng2-toasta)

## License

 [MIT](https://github.com/stevewhitmore/mmm-toast/blob/master/LICENSE)
