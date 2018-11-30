<h1>NativeScript Tour of Heroes migration</h1>

> Welcome to the NativeScript Tour of Heroes migration tutorial! You will learn how to extend your web Angular projects with mobile applications for Android and iOS. 

# Preparation

## NativeScript setup

For completing the tutorial you will need the NativeScript CLI and a physical Android or iOS device.

1. To install the NativeScript CLI, run the following command:

```bash
npm install -g nativescript@latest
```

2. While waiting for the `npm install`, grab your phone and install the two companion apps:

| app | Android | iOS |
|-----|---------|-----|
| NativeScript Preview    | [Google Play](https://play.google.com/store/apps/details?id=org.nativescript.preview)   | [App Store](https://itunes.apple.com/us/app/nativescript-preview/id1264484702) |
| NativeScript Playground | [Google Play](https://play.google.com/store/apps/details?id=org.nativescript.play) | [App Store](https://itunes.apple.com/us/app/nativescript-playground/id1263543946) |

## Angular setup

The Angular CLI provides us with a bunch of useful commands for migrating and managing Angular projects. NativeScript extends the built-in set of commands with its own `schematics` collection. To install both packages, run the following command:

```bash
npm install -g @angular/cli @nativescript/schematics
```

## The application

Today you will migrate the Tour of Heroes application from the [official Angular docs tutorial](https://angular.io/tutorial).

You should start by cloning the finished web version of the project.

```bash
git clone https://github.com/sebawita/tour-of-heroes/
cd tour-of-heroes
npm install
```

# Converting the project to a code-sharing structure

The **ng add** command, provided by the Angular CLI, allows you to add different capabilities to your Angular projects. A few examples of capabilities available through **ng add** are - Angular material, ngRx, Firebase, Angular Apollo, and, of course - NativeScript.
You can use **ng add** to convert the web project into a Web+NativeScript Code-Sharing project structure.  Just run the following command from inside the project:

```bash
ng add @nativescript/schematics
```

## Validating the migration
The easiest way to make sure everything went well, is to run the web and the mobile applications and see with your own eyes that they work.

1. Web

To build the web application, run the following command:
```bash
ng serve --open
```

When the build finishes, a new tab in your browser opens. It contains the same old web Tour of Heroes application.

2. Mobile

To build the mobile application, run the following command:
```
tns preview --bundle
```

The command prints out a QR code in your terminal. Open the **NativeScript Playground** app on your phone and press the **Scan QR code** button. Obviously, the next step is to scan the QR code by pointing your phone's camera to your laptop's screen.
After you scan the code, the **NativeScript Preview** app launches. At this point, you probably see a blank white screen. Don't get disappointed yet! Wait for a few seconds (up to a minute on a slow wifi connection) and you will be presented with a big button saying **AUTO-GENERATED WORKS!**. You have your first NativeScript Angular application! Exciting, isn't it?

> Troubleshooting: If you cannot scan the QR code, try changing the color theme of your terminal.

## Make your first change
You don't need to scan the QR code each time you want to update the app. The NativeScript CLI will push all saved changes automatically.

Let's put that to a test.
1. Open **src/app/auto-generated/auto-generated.component.tns.html**.
2. Change the `text` attribute to `text="Hello World"`.
3. Save the file.

Now you should see an app with a big button saying **Hello World** 💭🌍.

## Navigation

In your migrated project you can find two different routing `NgModule`s:

* **app-routing.module.ts** - a web routing NgModule;
* **app-routing.module.tns.ts** - a NativeScript routing NgModule.

> The code-sharing project uses a naming convention to differentiate between the files for web and mobile. The NativeScript-specific files have a `.tns` suffix. You can learn more about it in the NativeScript docs on [code-splitting](https://docs.nativescript.org/angular/code-sharing/code-splitting).

If you open **app-routing.module.tns.ts**, you will notice that NativeScript imports **NativeScriptRouterModule**. This module is a wrapper around the **RouterModule** provided by Angular. NativeScript extends the default Angular router with a few mobile-specific features, such as animated transitions between screens.

You will also see a **routes** array. It contains a single path, leading to the **HomeComponent**. Later in the tutorial, we will move the **routes** to a common file, shared between the web and the mobile `NgModule`s.

<h1>Migrating the Project Content</h1>

The **ng add @nativescript/schematics** command converts the project to a code-sharing structure. However, it doesn't convert your application logic. Enough auto generation! It's time to get your hands dirty and write some real code!

The next steps from here are to:

* migrate the **AppModule**
* migrate the other **modules**
* *(optional)* unify **navigation**

# Migrate the AppModule

In this section, you will be editing:
- **app.module.ts** - the root module for web;
- **app.module.tns.ts** - the root module for mobile.

The migration of the root `NgModule` (usually named `AppModule`) involves finding equivalents for its imported `NgModule`s and migrating its declared components. 

## Migrating AppModule imports

Let's have a look at the imports in the metadata of our web `NgModule`.

**app.module.ts**

```TypeScript
imports: [
  BrowserModule,
  FormsModule,
  AppRoutingModule,
  HttpClientModule,

  HttpClientInMemoryWebApiModule.forRoot(
    InMemoryDataService, { dataEncapsulation: false }
  )
],
```

Transforming module **imports** means that you need to move the imports from the web file (`app.module.ts`) to the mobile file (`app.module.tns.ts`). For each import you need to check:

* if this is a **platform-agnostic import** - meaning it works for both web and {N} - then just add it to the **app.module.tns.ts**;
* if this is a web-specific import - then we need to find a **{N} equivalent** and add it to the **app.module.tns.ts**.

### **BrowserModule**

The **BrowserModule** is **web specific**.
The equivalent module in {N} is the **NativeScriptModule**. Your **app.module.tns.ts** already contains an import of **NativeScriptModule**. No need to do anything here.

### **FormsModule**

The **FormsModule** is also **web specific**.
The equivalent module in {N} is **NativeScriptFormsModule**. Your **app.module.tns.ts** already contains a commented out import of **NativeScriptFormsModule**. Just uncomment it and add it to the **NgModule imports**.

### **AppRoutingModule**

At this stage you have two versions of the **AppRoutingModule**:

* Web - **app-routing.module.ts** - contains the **routes** to navigate to all your web components,
* {N} - **app-routing.module.tns.ts** - contains the **routes** to navigate to the sample **HomeComponent**.

We will cover the navigation in more details later on and we will discuss the options to bring both configurations in one place.
However, at this stage of the migration process it is better to keep them separately.

Your **app.module.tns.ts** already contains an import of the **{N} AppRoutingModule**, so no need to do anything.

### **HttpClientModule**

The **HttpClientModule** is not really **web specific**. You can use it in your mobile application as well. However, NativeScript has a similar `NgModule` - the `NativeScriptHttpClientModule`. It extends the default Angular HTTP client with a few mobile features. We recommend you to always use the {N} module in your mobile apps.

Your **app.module.tns.ts** already contains a commented out import of **NativeScriptHttpClientModule**. Just uncomment it and add it to the **NgModule imports**.

### **HttpClientInMemoryWebApiModule** and **InMemoryDataService**

The **HttpClientInMemoryWebApiModule** and **InMemoryDataService** are platform-agnostic. Copy their imports from **app.module.ts** to **app.module.tns.ts** and we are good to go.

## Module Imports Summary

This is how your NativeScript **imports** should look like:

**app.module.tns.ts**

```TypeScript
import { NativeScriptModule } from 'nativescript-angular/nativescript.module';
import { NativeScriptFormsModule } from 'nativescript-angular/forms';
import { NativeScriptHttpClientModule } from 'nativescript-angular/http-client';

import { AppRoutingModule } from './app-routing.module.tns';

import { HttpClientInMemoryWebApiModule } from 'angular-in-memory-web-api';
import { InMemoryDataService } from './in-memory-data.service';
...

@NgModule({
  ...
  imports: [
    NativeScriptModule,
    NativeScriptFormsModule,
    AppRoutingModule,
    NativeScriptHttpClientModule,

    HttpClientInMemoryWebApiModule.forRoot(
      InMemoryDataService, { dataEncapsulation: false }
    ),
  ],
  ...
})
export class AppModule { }
```

Here is a table with all `NgModule` imports:

| Name | Same Module | {N} Name | import from  |
|---------------|---|---|---|
| BrowserModule | No | NativeScriptModule | 'nativescript-angular/nativescript.module' |
| FormsModule | No | NativeScriptFormsModule | 'nativescript-angular/forms' |
| AppRoutingModule | No | AppRoutingModule | './app-routing.module.tns' |
| HttpClientModule | No | NativeScriptHttpClientModule | 'nativescript-angular/http-client' |
| HttpClientInMemoryWebApiModule | Yes | --- | --- |
| InMemoryDataService | Yes | --- | --- |

## Migrating AppModule Components

Your next task is to migrate the **AppModule Components** into a code-sharing structure.

Often the migration of each component consists of three steps:

1. create a **{N}** version of the template file (i.e. **name.component.tns.html**)
	* add the **{N} UI Code** to it
1. (optional) create a **{N}** version of the stylesheet file (i.e. **name.component.tns.css**)
	* add the **{N} Styling** to it
1. add the component to the **Declarations** of the **{N}** AppModule (`app.module.tns.ts`)
1. (optional) add the component to the {N} navigation

This task can be helped with the [migrate-component schematic](https://docs.nativescript.org/angular/code-sharing/migrating-a-web-project#schematic-migrate-component). It automates the first three steps. You can use it by running the following command:

```bash
ng g migrate-component --name=component-name
```

### Migrate HeroesComponent

First run the component migration for the **HeroesComponent**:

```bash
ng g migrate-component --name=heroes
```

The command modifies your project by:

* generating **heroes.component.tns.html** with the commented out html from **heroes.component.html**;
* generating an empty **heroes.component.tns.css**;
* adding the **HeroesComponent** to the **declarations** array in **app.module.tns.ts**.

```
src
└── app
    ├── heroes
    |   ├── heroes.component.html
    |   ├── heroes.component.tns.html <= create
    |   └── heroes.component.ts
    |   └── heroes.component.tns.css  <= create
    |   └── heroes.component.css
    ├── app.module.tns.ts             <= update
    └── app.module.ts           
```

#### Add the component to the mobile navigation

You need to update the NativeScript navigation **routes**, to make it navigate to the newly created **HeroesComponent** when the app starts.

**app-routing.module.tns.ts**

```TypeScript
import { HeroesComponent } from './heroes/heroes.component';

export const routes: Routes = [
  { path: '', redirectTo: '/heroes', pathMatch: 'full' },
  { path: 'heroes', component: HeroesComponent },
];
```

You should see a simple screen with a message: *Heroes Component works*.
> If you don't see the changes, try rebuilding the app. Stop the current process in the terminal and run the preview command again:
> ```
> tns preview --bundle
> ```

#### Update the Template

It's time to build your first mobile UI screen!

> The NativeScript UI components are a broad topic and we won't waste our precious tutorial time diving deep into them. If you are eager to learn more, check out these materials:
>  * [The interactive tutorial for {N} layouts](https://www.nslayouts.com/);
>  * [UI Widgets](https://docs.nativescript.org/angular/ui/ng-ui-widgets/action-bar).

Alright, let's get started! The html template looks like this:

**heroes.component.html**

```html
<h2>My Heroes</h2>

<div>
  <label>Hero name:
    <input #heroName />
  </label>
  <button (click)="add(heroName.value); heroName.value=''">
    add
  </button>
</div>

<ul class="heroes">
  <li *ngFor="let hero of heroes">
    <a routerLink="/detail/{{hero.id}}">
      <span class="badge">{{hero.id}}</span> {{hero.name}}
    </a>
    <button class="delete" title="delete hero"
      (click)="delete(hero)">x</button>
  </li>
</ul>
```

First, you have:

**heroes.component.html**

```html
<h2>My Heroes</h2>
```

Which is clearly the title of the page. You can replace it with an **ActionBar**.

**heroes.component.tns.html**

```html
<ActionBar title="My Heroes" class="action-bar">
</ActionBar>
```

Then you have a `div` container with a label, an input field and a button.

**heroes.component.html**

```html
<div>
  <label>Hero name:
    <input #heroName />
  </label>
  <button (click)="add(heroName.value); heroName.value=''">
    add
  </button>
</div>
```

You can convert it to {N} by replacing:

* the `<div>` container with a `<GridLayout>`, arranged in three columns;
* the `<label>` tag with a `<Label>` tag (capital `L`) with **Hero name:** provided as a `[text]` attribute;
* the `<input>` with a `<TextField>`;
* the `<button>` with a `<Button>` - where you pass `heroName.text` (instead of `heroName.value`);
  * the `(click)` event attribute with a `(tap)` event attribute.

Like this:

**heroes.component.tns.html**

```html
<GridLayout rows="auto" columns="auto, *, auto">
  <Label text="Hero name:" class="h2 m-l-5"></Label>
  <TextField col="1" hint="enter hero name..." #heroName class="m-x-5"></TextField>
  <Button col="2" text="add" (tap)="add(heroName.text); heroName.text=''" class="btn btn-primary"></Button>
</GridLayout>
```

Next, you have the list of heroes:

```html
<ul class="heroes">
  <li *ngFor="let hero of heroes">
    <a routerLink="/detail/{{hero.id}}">
      <span class="badge">{{hero.id}}</span> {{hero.name}}
    </a>
    <button class="delete" title="delete hero"
      (click)="delete(hero)">x</button>
  </li>
</ul>
```

You can convert it by replacing:

* the unordered list `<ul>` with a `<ListView>`;
* each item `<li>` with an `<ng-template>`;
* the navigation link `<a>` with a `GridLayout`. Also swap the `routerLink` directive with `nsRouterLink`;
* the `<span>` with a `<Label>`;
* the `<button>` with a `<Button>`.

Like this:

**heroes.component.tns.html**

```html
<ListView [items]="heroes" class="list-group" height="100%">
  <ng-template let-hero="item">
    <GridLayout columns="auto, *, auto" nsRouterLink="/detail/{{hero.id}}">
      <Label col="0" [text]="hero.id" class="list-group-item"></Label>
      <Label col="1" [text]="hero.name" class="list-group-item"></Label>
      <Button col="2" text="X" (tap)="delete(hero)" class="btn btn-primary"></Button>
    </GridLayout>
  </ng-template>
</ListView>
```

Finally, you need to wrap the `GridLayout` and the `ListView` in a `StackLayout`.

> Please note that besides the **ActionBar**, a NativeScript template can only contain one component. This is why you need a Layout container to wrap all the other components.

The full template should look like this:

**heroes.component.tns.html**

```TypeScript
<ActionBar title="My Heroes" class="action-bar">
</ActionBar>

<StackLayout>
  <GridLayout rows="auto" columns="auto, *, auto">
    <Label text="Hero name:" class="h2 m-l-5"></Label>
    <TextField col="1" hint="enter hero name..." #heroName class="m-x-5"></TextField>
    <Button col="2" text="add" (tap)="add(heroName.text); heroName.text=''" class="btn btn-primary"></Button>
  </GridLayout>

  <ListView [items]="heroes" class="list-group" height="100%">
    <ng-template let-hero="item">
      <GridLayout columns="auto, *, auto" nsRouterLink="/detail/{{hero.id}}">
        <Label col="0" [text]="hero.id" class="list-group-item"></Label>
        <Label col="1" [text]="hero.name" class="list-group-item"></Label>
        <Button col="2" text="X" (tap)="delete(hero)" class="btn btn-primary"></Button>
      </GridLayout>
    </ng-template>
  </ListView>
</StackLayout>
```

#### Update the Styling

You can style NativeScript UI with CSS. To make the above component more appealing, copy the CSS below to **heroes.component.tns.css**.

```CSS
.container {
  background-color: #EFEFF4;
}

.search-container {
  padding: 15;
}

.search-container TextField {
  color: #757575;
  padding-left: 10;
  font-size: 20;
}

.search-container Button {
  background-color: #30CE91;
  color: white;
  font-size: 18;
  padding: 10 30;
  border-radius: 20;
}

ListView {
  background-color: transparent;
  separator-color: transparent;
}

.hero-container {
  background-color: white;
  border-radius: 10;
  margin: 10 15;
  padding: 15;

}
.circle {
  background-color: #C6D7FE;
  color: #2A48CD;
}

.hero-label {
  font-weight: bold;
  font-size: 18;
  margin-left: 10;
}

.delete-button {
  font-size: 24;
  font-weight: normal;
}
```

## Migrate `HeroDetailComponent`

Next up you should migrate the **HeroDetailComponent**.

The steps are the same as before.

1. Run the migration command.
	
	```bash
	ng g migrate-component --name=hero-detail
	```
	
1. Register the new component in the mobile routing configuration.

	**app-routing.module.tns.ts**

	```bash
	import { HeroDetailComponent } from './hero-detail/hero-detail.component';
	
	export const routes: Routes = [
	  { path: '', redirectTo: '/heroes', pathMatch: 'full' },
	  { path: 'heroes', component: HeroesComponent },
	  { path: 'detail/:id', component: HeroDetailComponent },
	];
	```

1. Update the NativeScript template

**hero-detail.component.tns.html**

```html
<ActionBar title="{{hero?.name | uppercase}} Details" icon="" class="action-bar">
</ActionBar>

<StackLayout *ngIf="hero">
  <Label text="ID: {{hero.id}}" class="h1 text-center"></Label>
  <TextField [(ngModel)]="hero.name" hint="name" class="input input-border"></TextField>
  
  <Button text="Go Back" (tap)="goBack()" class="btn btn-primary"></Button>
  <Button text="Save" (tap)="save()" class="btn btn-primary"></Button>
</StackLayout>
```

1. Update the stylesheet

**hero-detail.component.tns.css**

```css
.container {
  background-color: #EDECF2;
}

.main-name {
  background-color: #2A48CD;
  font-size: 36;
  font-weight: bold;
  text-align: center;
  color: white;
  padding: 0 0 15 0;
}

.grid {
  margin: 30;
}

.circle {
  background-color: #C6D7FE;
  color: #2A48CD;
}

.name {
  font-size: 22;
  font-weight: bold;
  margin-left: 20;
}

.id-label {
  text-align: center;
}

.name-label {
  margin-left: 20;
}

.id-label, .name-label {
  margin-top: 10;
  color: #ABABAB;
}

.btn {
  background-color: #2A48CD;
  color: white;
  font-weight: bold;
  font-size: 18;
  padding: 15 0;
  border-radius: 25;
}
```

1. Try out the new page

Run the app and navigate to a hero and then navigate back. Repeat the first two steps several times to make sure it really works.

## Migrate DashboardComponent and HeroSearchComponent

The **DashboardComponent** uses the the **HeroSearchComponent**, which means that you should migrate both of the components at the same time.

1. Run the migration command for the two components:

```bash
  ng g migrate-component --name=dashboard
  ng g migrate-component --name=hero-search
```
	
1. Update the navigation

Add a path for the **DashboardComponent** to the routes.
However, you don't need a path for the **HeroSearchComponent**, as it is used directly via the **<app-hero-search>** selector.

**app-routing.module.tns.ts**

```bash
import { DashboardComponent } from './dashboard/dashboard.component';

export const routes: Routes = [
  { path: '', redirectTo: '/heroes', pathMatch: 'full' },
  { path: 'heroes', component: HeroesComponent },
  { path: 'detail/:id', component: HeroDetailComponent },
  { path: 'dashboard', component: DashboardComponent },
];
```

1. Update the NativeScript templates

**dashboard.component.tns.html**

```html
<ActionBar title="Top Heroes" icon="" class="action-bar">
</ActionBar>

<GridLayout rows="*, *">
  <ListView row="0" [items]="heroes" class="list-group">
    <ng-template let-hero="item">
      <Button [text]="hero.name" nsRouterLink="/detail/{{hero.id}}" class="btn btn-primary"></Button>
    </ng-template>
  </ListView>
  <StackLayout row="1">
    <app-hero-search row="1"></app-hero-search>
  </StackLayout>
</GridLayout>
```

**hero-search.component.tns.css**

```html
<StackLayout>
  <Label text="Hero Search" class="h1 text-center"></Label>

  <TextField #heroName
    hint="enter hero-name"
    autocorrect="false"
    class="m-x-5"
    (loaded)="heroName.text = ''"
    (textChange)="search($event.value)">
  </TextField>
  <ListView [items]="heroes$ | async" class="list-group">
    <ng-template let-hero="item">
      <Label [text]="hero.name" nsRouterLink="/detail/{{hero.id}}" class="list-group-item"></Label>
    </ng-template>
  </ListView>
</StackLayout>
```
	
> Here is the interesting part. You can use the `<app-hero-search>` selector for both the web and mobile apps. As the **HeroSearchComponent** is platform agnostic.
	
1. Update the NativeScript stylesheets:

**dashboard.component.tns.css**

```CSS
.header {
    font-size: 42;
    font-weight: bold;
    color: white;
    text-align: center;
    margin: 0 0 20 0;
}

FlexboxLayout {
    flex-wrap: wrap;
    justify-content: center;
    align-items: center;
    align-content: flex-start;
    margin-top: 30;
}
.hero-container {
    width: 49%;
    border-width: 1;
    border-color: #395ad9;
}

.name {
    color: white;
    font-size: 20;
    text-align: center;
    margin: 10 0 20 0;
}

.circle {
    margin-top: 20;
}
```

**hero-search.component.tns.css**

```CSS
.search-result li {
  border-bottom: 1px solid gray;
  border-left: 1px solid gray;
  border-right: 1px solid gray;
  width:195px;
  height: 16px;
  padding: 5px;
  background-color: white;
  cursor: pointer;
  list-style-type: none;
}

.search-result li:hover {
  background-color: #607D8B;
}

.search-result li a {
  color: #888;
  display: block;
  text-decoration: none;
}

.search-result li a:hover {
  color: white;
}
.search-result li a:active {
  color: white;
}
#search-box {
  width: 200px;
  height: 20px;
}


ul.search-result {
  margin-top: 0;
  padding-left: 0;
}
```

1. Try out the components

At this point, your mobile app doesn't have a way to navigate to the Dashboard. The easiest way to test it, is to change the default **redirectTo** path, to `'/dashboard'`. Like this:

**app-routing.module.tns.ts**

```TypeScript
export const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'heroes', component: HeroesComponent },
  { path: 'detail/:id', component: HeroDetailComponent },
  { path: 'dashboard', component: DashboardComponent },
];
```

But, make sure to put it back, once you finished testing the app.

## SideDrawer Navigation

What you are missing now is a nice way to navigate between the Heroes page and the Dashboard. This usually comes down to the choice between **TabView** and **SideDrawer**.

For the purpose of this tutorial, you will use a **SideDrawer** with a set of buttons to navigate to the Heroes page and to the Dashboard page.

### Adding the SideDrawer plugin

To install the **NativeScript SideDrawer**, run the following command:

```bash
tns plugin add nativescript-ui-sidedrawer
```

### Update the {N} AppModule

Then, you need to import the **NativeScriptUISideDrawerModule** in the **NativeScript AppModule*.

**app.module.tns.ts**

```TypeScript
import { NativeScriptUISideDrawerModule } from 'nativescript-ui-sidedrawer/angular';

@NgModule({
  imports: [
    ...
    NativeScriptUISideDrawerModule,
  ]
  ...
})
export class AppModule { }
```

### Add the SideDrawer

Adding the SideDrawer to your project is really easy. Just like the navigation for the web project is defined in **app.component.html**, you can add the SideDrawer to **app.compoment.tns.html**:

**app.component.tns.html**

```html
<RadSideDrawer>
  <GridLayout tkDrawerContent rows="*" class="sidedrawer sidedrawer-left">
    <StackLayout class="sidedrawer-content">
      <Button text="heroes" nsRouterLink="/heroes" (tap)="closeDrawer()" clearHistory="true" class="btn btn-primary"></Button>
      <Button text="dashboard" nsRouterLink="/dashboard" (tap)="closeDrawer()" clearHistory="true" class="btn btn-primary"></Button>
    </StackLayout>
  </GridLayout>

  <page-router-outlet tkMainContent class="page page-content"></page-router-outlet>
</RadSideDrawer>
```

Note, that the `<GridLayout>` contains the SideDrawer content (this is where you need to add all your navigation links). While, the `<page-router-outlet>` is where the page content is loaded.

Additionally, each button uses the combination of `nsRouterLink='/path'` and `clearHistory="true"`. This is to stop iOS from adding a back button in the ActionBar, or to stop Android from navigating back, when the user hits the back button on the device.

Finally, each button calls the `closeDrawer()` function, so that after the app navigates, the SideDrawer will close itself gracefully, thus giving a nice UX experience for your users.

```html
<Button
	text="heroes"
	nsRouterLink="/heroes"
	clearHistory="true"
	(tap)="closeDrawer()"
	class="btn btn-primary">
</Button>
```

However, you need to add the `closeDrawer()` function to the **app.component.tns.ts**, like this:

```TypeScript
import { Component } from '@angular/core';
import * as app from 'tns-core-modules/application';
import { RadSideDrawer } from 'nativescript-ui-sidedrawer';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
})

export class AppComponent {
  closeDrawer() {
    const sideDrawer = <RadSideDrawer>app.getRootView();
    sideDrawer.closeDrawer();
  }
}
```

### Test the SideDrawer

Now you should be able to slide with your finger from the left of the screen, which will show the SideDrawer. When you tap on any of the buttons, it should close the drawer and navigate to that page.

# Shared Navigation

## Split Routes

In this state the routes are split into two separate configurations. Which allows you to work with different navigation **routes** for web and mobile.

This is why when you run a NativeScript project now, it will start by showing the **HomeComponent**.

> This is usually useful, while you are in the process of migrating your web components into a code-sharing structure. Once this is complete, you could switch to a singe routes configuration for both web and mobile.

> Also, if you expect your web app to have a different set of pages to your mobile app, then you could keep the routes separate. For example, your web app could have an admin screen, which might not be required in the mobile app. In this case it makes sense to have two sets of navigation configurations.

The process here is that you should add navigation paths to routes array in **app-routing.module.tns.ts**, as you migrate each of your page components.

## Shared Routes

However, for this project you can easily work with a single **routes** configuration.

To achieve that, move the **routes** configuration to a shared file.

### Step 1 - Create a shared routes file.

Create a new file called **app.routes.ts** and copy the **routes** array from **app-routing.module.ts**.

Make sure to **export** routes, and also import all the components.

**app.routes.ts**

```
import { Routes } from '@angular/router';
import { HeroesComponent } from './heroes/heroes.component';
import { DashboardComponent } from './dashboard/dashboard.component';
import { HeroDetailComponent } from './hero-detail/hero-detail.component';

export const routes: Routes = [
  { path: '', redirectTo: '/heroes', pathMatch: 'full' },
  { path: 'heroes', component: HeroesComponent },
  { path: 'dashboard', component: DashboardComponent },
  { path: 'detail/:id', component: HeroDetailComponent },
];
```

### Step 2 - Update both **app-routing** files with the shared **routes** property

Replace the **routes** property definition with the imported one from **'./app.routes'** in both **app-routing** files. Then clean up any unused imports.

**app-routing.module.ts**

```
import { NgModule } from '@angular/core';
import { RouterModule } from '@angular/router';
import { routes } from './app.routes';

@NgModule({
  imports: [
    RouterModule.forRoot(routes)
  ],
  exports: [
    RouterModule
  ]
})
export class AppRoutingModule { }
```

**app-routing.module.tns.ts**

```
import { NgModule } from '@angular/core';
import { NativeScriptRouterModule } from 'nativescript-angular/router';
import { routes } from './app.routes';

@NgModule({
  imports: [
    NativeScriptRouterModule.forRoot(routes)
  ],
  exports: [
    NativeScriptRouterModule
  ]
})
export class AppRoutingModule { }
```

Now you have a single shared navigation configuration for both your web and mobile apps.

# Wrap up
Congratulations! You successfully converted the Tour of Heroes app to a multi-platform project, running on the web, Android and iOS.
If you want to read more about sharing code with NativeScript and migrating web applications, check out the official documentation:

> [Migrating a Web Project](https://docs.nativescript.org/angular/code-sharing/migrating-a-web-project)

> Credits: This tutorial is created by Sebastian Witalec and partly edited by Stanimira Vlaeva.
