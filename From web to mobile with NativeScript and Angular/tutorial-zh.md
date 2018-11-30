<h1> 英雄项目的 NativeScript 迁移之旅 </h1>

> 欢迎来到英雄项目 NativeScript 化迁移之旅的教程！你可以学到如何将你的 Angular web 项目扩展成能运行在 Android 和 iOS 上的应用。

# 准备

## NativeScript 配置

你需要 NativeScript CLI 和一台 Android 或 iOS 物理设备来完成这个教程。

1. 运行下面的命令来安装  NativeScript CLI:

```bash
npm install -g nativescript@latest
```

2. 在等待 `npm install` 的过程中，你可以拿起手机安装两个配合运行的 app：

| app | Android | iOS |
|-----|---------|-----|
| NativeScript Preview    | [Google Play](https://play.google.com/store/apps/details?id=org.nativescript.preview)   | [App Store](https://itunes.apple.com/us/app/nativescript-preview/id1264484702) |
| NativeScript Playground | [Google Play](https://play.google.com/store/apps/details?id=org.nativescript.play) | [App Store](https://itunes.apple.com/us/app/nativescript-playground/id1263543946) |

## Angular 配置

Angular CLI 提供了一些方便迁移和管理 Angular 项目的命令。 NativeScript 用自身的 `schematics` 集合扩展了 Angular CLI 的内置命令。通过执行下面的命令来安装 Angular CLI 和 NativeScript 扩展。
```bash
npm install -g @angular/cli @nativescript/schematics
```

## 应用

今天你将会对 [Angular 官方文档教程](https://angular.io/tutorial) 中的英雄之旅项目进行迁移。

首先克隆这个项目的 web 完全版。
```bash
git clone https://github.com/sebawita/tour-of-heroes/
cd tour-of-heroes
npm install
```

# 将项目转换成代可以进行代码共享的结构

Angular CLI 提供的 **ng add** 命令让你可以为 Angular 项目添加不同的功能，比如 Angular material, ngRx, Firebase, Angular Apollo，当然也包括 NativeScript。
你可以使用 **ng add** 命令将 web 项目转换成供 Web 和 NativeScript 共享代码的项目结构。在项目中运行下面的命令即可：
```bash
ng add @nativescript/schematics
```

## 验证迁移

确保转换顺利最简单的方式就是运行一下 web 和移动项目，亲眼看一下它们是否正常工作。
1. Web 端

运行下面的命令来启动 web 端应用:
```bash
ng serve --open
```

当构建完成时，你的浏览器会自动打开一个新的标签页，页面中运行着之前的英雄之旅应用。

1. 移动端

运行下面的命令来启动移动端应用:
```
tns preview --bundle
```

此命令会在命令行终端输出一个二维码。打开你手机中的 **NativeScript Playground** 应用，用 **Scan QR code** 功能扫描命令行中的二维码。

扫描之后，**NativeScript Preview** 应用即会启动。一开始应用可能会白屏。先不要失望哦，等几秒（WiFi 信号差时可能慢至一分钟）你就会看到一个大按钮，告诉你 **AUTO-GENERATED WORKS!**。这样你就得到了你第一个 NativeScript Angular。爽不爽？
> 提示：如果无法扫描二维码，可以试着更换命令行的颜色。

## 进行第一次修改
你不必在每次更新应用时都去扫描二维码。NativeScript CLI 会自动推送所有已保存的变更。

让我们来试一下。
1. 打开 **src/app/auto-generated/auto-generated.component.tns.html**。
2. 将 `text` 属性改为 `text="Hello World"`。
3. 保存文件。

现在你应该会看到 app 中的大按钮写着 **Hello World** 💭🌍。

## 导航
在迁移后的项目中你会发现联众不同的路由 `NgModule`：

* **app-routing.module.tns.ts** - 一个 NativeScript 端路由 NgModule。

> 代码共享的项目使用命名约定来区分 web 端和移动端文件。NativeScript 特定的文件后缀为 `.tns`。 在 NativeScript 文档的 [代码分割](https://docs.nativescript.org/angular/code-sharing/code-splitting) 章节你可以了解更多。

打开文件 **app-routing.module.tns.ts**，你会注意到 NativeScript 引入了 **NativeScriptRouterModule**。这个模块封装了 Angular 提供的 **RouterModule**。NativeScript 扩展了 Angular 的默认路由，添加了一些移动设备的功能，比如屏幕之间的过渡动画。

你还会看到一个 **routes** 数组。它包含了一个指向 **HomeComponent** 的路由。在后面的教程中，我们会将 **routes** 移至一个普通的文件，以在 web 端和移动端的 `NgModule` 中共享。

<h1> 迁移项目内容 </h1>

**ng add @nativescript/schematics** 这个命令将项目转换为代码共享的结构。然而它并没有转换应用的逻辑。关于自动生成的部分先讲到这里，是时候自己亲自动手写一写代码了！


接下来的几步：

* 迁移 **AppModule**
* 迁移其它的 **modules**
* *(可选)* 统一 **navigation**

# 迁移 AppModule

在这部分中，你将会编辑：
- **app.module.ts** - web 端的根模块;
- **app.module.tns.ts** - 移动端的根模块.

根模块 `NgModule`（通常叫做 `AppModule`）的迁移包括寻找它引入的 `NgModule` 的等价模块以及迁移它声明的组件。

## 迁移 AppModule 的引入模块

让我们看一下 web 端 `NgModule` 的元信息中引入了哪些模块。

**app.module.ts**

```TypeScript
imports: [
  BrowserModule,
  FormsModule,
  AppRoutingModule,
  HttpClientModule,

  HttpClientInMemoryWebApiModule.forRoot(
    InMemoryDataService, {dataEncapsulation: false}
  )
],
```

对 **imports** 部分的迁移工作指的是将 web 端文件 (`app.module.ts`) 改为移动端文件(`app.module.tns.ts`)。对每个引入的模块你都需要检查：

* 如果这是 **平台无关的引入** - 意味着它在 web 和 {N}（注：{N} 指 NativeScript 环境）两种环境中均可工作 - 把它添加至 **app.module.tns.ts** 即可；
* 如果是平台相关的引入 - 我们需要找到 **{N} 等价模块** 并把等价模块添加至 **app.module.tns.ts**。

### **BrowserModule**

**BrowserModule** 是 **web 特定** 的模块。
它在 {N} 中的等价模块是 **NativeScriptModule**。 你的 **app.module.tns.ts** 已经引入了**NativeScriptModule**，这里不需再做额外工作。

### **FormsModule**

**FormsModule** 也是 **web 特定** 的模块。
它在 {N} 中的等价模块是 **NativeScriptFormsModule**。你的 **app.module.tns.ts** 中有一条被注释掉的对它的引入，取消注释即可把它引入至**NgModule imports**。

### **AppRoutingModule**

这个阶段我们有两个版本的 **AppRoutingModule**：

* Web - **app-routing.module.ts** - 包含 **routes** 来对你原来的 web 组件进行导航,
* {N} - **app-routing.module.tns.ts** - 包含 **routes** 来导航至 **HomeComponent** 样例组件.

稍后我们将详细探讨导航，现在我们先讨论如何将两种配置集合在一起。
不过，在迁移工作的这个阶段，还是将它们分开比较好。

你的 **app.module.tns.ts** 早已引入了 **{N} AppRoutingModule**，所以不需再做额外工作。

### **HttpClientModule**

**HttpClientModule** 实际上不是 **web 特定**的。你也可以在移动应用中使用。但是，NativeScript 提供了相似的模块 - `NativeScriptHttpClientModule`。它扩展了默认的 Angular HTTP client，对移动端更加友好。我们推荐你优先使用 {N} module。

你的 **app.module.tns.ts** 已经有一条被注释了的对**NativeScriptHttpClientModule**的引入，取消注释即可把它引入至**NgModule imports**。

### **HttpClientInMemoryWebApiModule** 和 **InMemoryDataService**


**HttpClientInMemoryWebApiModule** 和 **InMemoryDataService** 是平台无关的模块。把他们的引入从 **app.module.ts** 复制到 **app.module.tns.ts** 后我们就能顺利进行下去了。

## Module 引入总结

这是你 NativeScript 的 **imports** 应该的样子:

**app.module.tns.ts**

```TypeScript
import {NativeScriptModule} from 'nativescript-angular/nativescript.module';
import {NativeScriptFormsModule} from 'nativescript-angular/forms';
import {NativeScriptHttpClientModule} from 'nativescript-angular/http-client';

import {AppRoutingModule} from './app-routing.module.tns';

import {HttpClientInMemoryWebApiModule} from 'angular-in-memory-web-api';
import {InMemoryDataService} from './in-memory-data.service';
...

@NgModule({
  ...
  imports: [
    NativeScriptModule,
    NativeScriptFormsModule,
    AppRoutingModule,
    NativeScriptHttpClientModule,

    HttpClientInMemoryWebApiModule.forRoot(
      InMemoryDataService, {dataEncapsulation: false}
    ),
  ],
  ...
})
export class AppModule { }
```

这是一张所有 `NgModule` imports 表:

| Name | Same Module | {N} Name | import from  |
|---------------|---|---|---|
| BrowserModule | No | NativeScriptModule | 'nativescript-angular/nativescript.module' |
| FormsModule | No | NativeScriptFormsModule | 'nativescript-angular/forms' |
| AppRoutingModule | No | AppRoutingModule | './app-routing.module.tns' |
| HttpClientModule | No | NativeScriptHttpClientModule | 'nativescript-angular/http-client' |
| HttpClientInMemoryWebApiModule | Yes | --- | --- |
| InMemoryDataService | Yes | --- | --- |

## 迁移 AppModule 组件

下一步的任务是将 **AppModule Components** 迁移成代码共享的结构。

通常，组件的迁移由三个步骤组成：

1. 创建一个 **{N}** 版本的模板文件（例如 **name.component.tns.html**）
   * 为它加上 **{N} UI Code**
2. （可选）创建一个 **{N}** 版本的样式文件（例如：**name.component.tns.css**）
   * 为它加上 **{N} UI Code**
3. 在 **{N}** AppModule (`app.module.tns.ts`)的 **Declarations** 中加入该组件   
（可选）将该组件添加至 {N} 导航。

这步任务可以在 [migrate-component schematic](https://docs.nativescript.org/angular/code-sharing/migrating-a-web-project#schematic-migrate-component) 的帮助下进行。它会自动完成前三步。通过下面的命令执行它：

```bash
ng g migrate-component --name=component-name
```

### 迁移 HeroesComponent

首先进行 **HeroesComponent** 的组件迁移：

```bash
ng g migrate-component --name=heroes
```

这个命令通过如下方式更改你的项目：

* 使用 **heroes.component.html** 中注释掉的 html 生成 **heroes.component.tns.html**；
* 生成 **heroes.component.tns.css** 空文件；
* 将 **HeroesComponent** 添加至 **app.module.tns.ts** 的 **declarations** 数组中。

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

#### 将组件加到移动端的导航中

你需要更新 NativeScript 的导航 **routes**，让它可以在应用启动时导航至新创建的 **HeroesComponent** 。

**app-routing.module.tns.ts**

```TypeScript
import {HeroesComponent} from './heroes/heroes.component';

export const routes: Routes = [
  {path: '', redirectTo:'/heroes', pathMatch:'full'},
  {path: 'heroes', component: HeroesComponent},
];
```

你会看到屏幕中有一条简单的消息： *Heroes Component works*。

> 如果你没有看到变化，试着重新构建应用。停止命令行的当前进程并再次执行预览命令：
> ```
> tns preview --bundle
> ```

#### 更新模板

是时候创建你第一个移动端页面了！

> NativeScript UI 组件是一个广泛的话题，今天的时间比较宝贵，我们不会深入探讨它。如果你想了解更多，可以看看这些资料：
>  * [The interactive tutorial for {N} layouts](https://www.nslayouts.com/);
>  * [UI Widgets](https://docs.nativescript.org/angular/ui/ng-ui-widgets/action-bar).

好了，让我们开始吧。html 模板看起来像这样：

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

一开始，是:

**heroes.component.html**

```html
<h2>My Heroes</h2>
```

这显然是页面的标题。你可以用 **ActionBar** 来替换。

**heroes.component.tns.html**

```html
<ActionBar title="My Heroes" class="action-bar">
</ActionBar>
```

接下来是一个包含着一个 label，一个 input 和一个 button 的 `div` 容器。

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

你需要进行如下替换：

* 将 `<div>` 替换为 `<GridLayout>`，排成三列；
* 将 `<label>` 标签替换为 `<Label>` 标签（首字母大写），将**Hero name:**的值赋给标签的 `[text]` 属性；
* 将 `<input>` 替换为 `<TextField>`；
* 将 `<button>` 替换为 `<Button>` - 在这里你传入 `heroName.text`（而不是 `heroName.value`）；
* 将 `(click)` 事件属性替换为 `(tap)` 事件属性。

像这样:

**heroes.component.tns.html**

```html
<GridLayout rows="auto" columns="auto, *, auto">
  <Label text="Hero name:" class="h2 m-l-5"></Label>
  <TextField col="1" hint="enter hero name..." #heroName class="m-x-5"></TextField>
  <Button col="2" text="add" (tap)="add(heroName.text); heroName.text=''" class="btn btn-primary"></Button>
</GridLayout>
```

下面是英雄列表:

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

你需要进行如下替换：

* 将无序列表 `<ul>` 替换为 `<ListView>`；
* 将每一个 `<li>` 替换为 `<ng-template>`；
* 将导航链接 `<a>` 替换为 `GridLayout`。同时将 `routerLink` 指令换为 `nsRouterLink`；
* 将 `<span>` 替换为`<Label>`；
* 将 `<button>` 替换为 `<Button>`。

像这样：

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

最后，你需要用 `StackLayout` 将 `GridLayout` 和 `ListView` 包起来。

> 请注意，包括 **ActionBar** 在内的 NativeScript 模板最多仅能包含一个组件。这就是你需要一个布局容器把其他组件囊括起来的原因。

整个模板会像这样:

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

#### 更新样式

你可以用 CSS 来为 NativeScript 编写样式。为了让上面的组件更有吸引力，把下面的样式复制到 **heroes.component.tns.css**。

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

## 迁移 `HeroDetailComponent`

下一步你需要迁移 **HeroDetailComponent**。

步骤与前面相同.

1. 运行迁移命令.

	```bash
	ng g migrate-component --name=hero-detail
	```

2. 在移动端导航配置中注册新的组件。

	**app-routing.module.tns.ts**

	```bash
	import {HeroDetailComponent} from './hero-detail/hero-detail.component';

	export const routes: Routes = [
	  {path: '', redirectTo:'/heroes', pathMatch:'full'},
	  {path: 'heroes', component: HeroesComponent},
	  {path: 'detail/:id', component: HeroDetailComponent},
	];
	```

3. 更新 NativeScript 模板。

**hero-detail.component.tns.html**

```html
<ActionBar title="{{hero?.name | uppercase}} Details" icon=""class="action-bar">
</ActionBar>

<StackLayout *ngIf="hero">
  <Label text="ID: {{hero.id}}" class="h1 text-center"></Label>
  <TextField [(ngModel)]="hero.name" hint="name" class="input input-border"></TextField>

  <Button text="Go Back" (tap)="goBack()" class="btn btn-primary"></Button>
  <Button text="Save" (tap)="save()" class="btn btn-primary"></Button>
</StackLayout>
```

4. 更新样式。

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

5. 试一下新页面

运行 app 并导航到一个英雄页面再返回。重复几次以确定程序正常运行。

## 迁移 DashboardComponent 和 HeroSearchComponent

**DashboardComponent** 使用了 **HeroSearchComponent**，这意味着你需要同时迁移这两个组件。

1. 为这两个组件运行迁移命令:

```bash
  ng g migrate-component --name=dashboard
  ng g migrate-component --name=hero-search
```

2. 更新导航

在路由中为 **DashboardComponent** 添加一个新的路径。
但是，你不需要为 **HeroSearchComponent** 再添加路径，因为可以通过 **<app-hero-search>** 选择器直接使用它。


**app-routing.module.tns.ts**

```bash
import {DashboardComponent} from './dashboard/dashboard.component';

export const routes: Routes = [
  {path: '', redirectTo:'/heroes', pathMatch:'full'},
  {path: 'heroes', component: HeroesComponent},
  {path: 'detail/:id', component: HeroDetailComponent},
  {path: 'dashboard', component: DashboardComponent},
];
```

1. 更新 the NativeScript 模板

**dashboard.component.tns.html**

```html
<ActionBar title="Top Heroes" icon=""class="action-bar">
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
    (loaded)="heroName.text =''"
    (textChange)="search($event.value)">
  </TextField>
  <ListView [items]="heroes$ | async" class="list-group">
    <ng-template let-hero="item">
      <Label [text]="hero.name" nsRouterLink="/detail/{{hero.id}}" class="list-group-item"></Label>
    </ng-template>
  </ListView>
</StackLayout>
```

> 有趣的是，`<app-hero-search>` 选择器在 web 端和移动端都可使用。因为 **HeroSearchComponent** 是平台无关的。

4. 更新 NativeScript 样式:

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

5. 试一下组件。

此时，你的移动端应用还无法导航至 Dashboard。 导航到它最简单的方法就是修改默认 **redirectTo** 路径为`'/dashboard'`。像这样：

**app-routing.module.tns.ts**

```TypeScript
export const routes: Routes = [
  {path: '', redirectTo:'/dashboard', pathMatch:'full'},
  {path: 'heroes', component: HeroesComponent},
  {path: 'detail/:id', component: HeroDetailComponent},
  {path: 'dashboard', component: DashboardComponent},
];
```

但一定要记得测试完改回去。

## SideDrawer 导航

你现在还缺一个好方法实现在英雄页和 Dashboard 页之间导航。通常可以用 **TabView** 和 **SideDrawer**。

在本教程中，我们选择有一系列按钮的 **SideDrawer** 导航方式。

### 添加 SideDrawer 插件

运行下面的命令以安装  **NativeScript SideDrawer**

```bash
tns plugin add nativescript-ui-sidedrawer
```

### 更新 {N} AppModule

然后你需要在 **NativeScript AppModule** 中引入 **NativeScriptUISideDrawerModule**。

**app.module.tns.ts**

```TypeScript
import {NativeScriptUISideDrawerModule} from 'nativescript-ui-sidedrawer/angular';

@NgModule({
  imports: [
    ...
    NativeScriptUISideDrawerModule,
  ]
  ...
})
export class AppModule { }
```

### 添加 SideDrawer

在项目中添加 SideDrawer 非常容易。 就像 web 项目 **app.component.html** 中导航定义的那样，你可以将  SideDrawer 添加到 **app.compoment.tns.html**：

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

注意，`<GridLayout>` 包含着 SideDrawer 的内容（你需要在这添加所有的导航链接）。但是页面的内容会在 `<page-router-outlet>` 处加载。

此外， 每个按钮都同时使用了 `nsRouterLink='/path'` 和 `clearHistory="true"`。这是为了防止 iOS 在 ActionBar 上添加后退按钮，并防止 Android 在用户使用设备的返回功能时回退。

最后，每个按钮调用 `closeDrawer()` 函数，为了在应用导航后 SideDrawer 自己进行优雅的关闭，以提供良好的用户体院。


```html
<Button
	text="heroes"
	nsRouterLink="/heroes"
	clearHistory="true"
	(tap)="closeDrawer()"
	class="btn btn-primary">
</Button>
```

你需要在 **app.component.tns.ts** 中添加 `closeDrawer()` 函数，像这样：

```TypeScript
import {Component} from '@angular/core';
import * as app from 'tns-core-modules/application';
import {RadSideDrawer} from 'nativescript-ui-sidedrawer';

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

### 测试 SideDrawer

现在你可以在屏幕左侧滑动手指，SideDrawer 会出现。当你点击任何一个按钮时，SideDrawer 会关闭并导航至对应页面。

# 共享导航

## 分割路由

在这个阶段路由会被分割为两个不同的配置，分别应用在 web 端和移动端。

这是你启动一个 NativeScript 项目时，它会先展示 **HomeComponent** 的原因。

> 在你将你的 web 组件迁移成代码共享结构时，路由分割通常是有用的。一旦迁移完成，你可以为双端选择一份单一的配置。

> 并且，如果你希望你的 web app 相比 mobile app 有不同的页面，你也可以使用两套不同的路由。比如是，你的 web app 有一个管理页面，但 mobile app 并不需要。这种情况下使用两套导航配置比较合理。

这个阶段在你迁移每个页面组件时，你需要在 **app-routing.module.tns.ts** 的路由数组里添加对应的导航路径。

## 共享路由

这个项目你可以简单地使用同一套 **routes** 配置，只需要将 **routes** 配置移到一个共享的文件。

### 步骤 1 - 创建一个共享的路由文件。

创建一个名为 **app.routes.ts** 的新文件 并将 **app-routing.module.ts** 中的 **routes** 数组复制过来。

确保**导出**了路由，确保引入了所有组件。

**app.routes.ts**

```
import {Routes} from '@angular/router';
import {HeroesComponent} from './heroes/heroes.component';
import {DashboardComponent} from './dashboard/dashboard.component';
import {HeroDetailComponent} from './hero-detail/hero-detail.component';

export const routes: Routes = [
  {path: '', redirectTo:'/heroes', pathMatch:'full'},
  {path: 'heroes', component: HeroesComponent},
  {path: 'dashboard', component: DashboardComponent},
  {path: 'detail/:id', component: HeroDetailComponent},
];
```

### Step 2 - Update both **app-routing** files with the shared **routes** property
### 步骤 2 - 使用共享的 **routes** 属性更新两个 **app-routing** 文件

用从 **'./app.routes'** 引入的 **routes** 更新两个 **app-routing** 文件中 **routes** 的值。然后清理掉未使用的引入。

**app-routing.module.ts**

```
import {NgModule} from '@angular/core';
import {RouterModule} from '@angular/router';
import {routes} from './app.routes';

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
import {NgModule} from '@angular/core';
import {NativeScriptRouterModule} from 'nativescript-angular/router';
import {routes} from './app.routes';

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

现在你有了一个双端共享的导航配置。

# 圆满完成

Congratulations! 你成功地将英雄之旅 app 转化为跨平台的项目，它可以运行在 web，Android 和 iOS 上。
如果你想阅读更多关于与 NativeScript 共享代码和迁移 web 应用的资料，可以看一下官方文档：

> [Migrating a Web Project](https://docs.nativescript.org/angular/code-sharing/migrating-a-web-project)

> Credits: This tutorial is created by Sebastian Witalec and partly edited by Stanimira Vlaeva.
