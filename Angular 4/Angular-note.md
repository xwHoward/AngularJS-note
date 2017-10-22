# Angular 学习笔记

## 1 架构

### 1.1 模块

每个 Angular 应用至少有一个模块（根模块），习惯上命名为`AppModule`。Angular 模块（无论是根模块还是特性模块）都是一个带有`@NgModule`装饰器的类。

`NgModule`是一个装饰器函数（用来装饰所有的Ng模块，Ng模块（带@NgModule装饰器的类）是 Angular 的基础特性之一），它接收一个用来描述模块属性的元数据对象。其中最重要的属性是：

- declarations - 声明本模块中拥有的视图类。Angular 有三种视图类：组件、指令和管道。
- exports - declarations 的子集，可用于其它模块的组件模板。
    > 根模块不需要导出任何东西，因为其它组件不需要导入根模块。
- imports - 本模块声明的组件模板需要的类所在的其它模块。
- providers - 服务的创建者，并加入到全局服务列表中，可用于应用任何部分。
- bootstrap - 指定应用的主视图（称为根组件），它是所有其它视图的宿主。只有根模块才能设置bootstrap属性。

我们通过*引导*根模块来启动应用。在开发期间，通常在一个main.ts文件中引导`AppModule`:

```ts
import { enableProdMode } from '@angular/core';
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

import { AppModule } from './app/app.module';
import { environment } from './environments/environment';

if (environment.production) {
  enableProdMode();
}

platformBrowserDynamic().bootstrapModule(AppModule);
```

Angular 模块库：Angular 提供了一组 JavaScript 模块。可以把它们看做库模块。每个 Angular 库的名字都带有`@angular`前缀。

### 1.2 组件

组件控制视图。我们在类中定义组件的应用逻辑，为视图提供支持。 组件通过一些由属性和方法组成的 API 与视图交互。

```ts
// 应用运行过程中，Angular 会创建、更新和销毁组件。 应用可以通过生命周期钩子在组件生命周期的各个时间点上插入自己的操作，比如这里的OnInit
export class HeroListComponent implements OnInit {
  heroes: Hero[];// HeroListComponent有一个heroes属性，它返回一个英雄数组，这个数组从一个服务获得。
  selectedHero: Hero;

  constructor(private service: HeroService) { }

  ngOnInit() {
    this.heroes = this.service.getHeroes();
  }

  // 还有一个当用户从列表中点选一个英雄时设置selectedHero属性的selectHero()方法。
  selectHero(hero: Hero) { this.selectedHero = hero; }
}
```

### 1.3 模板

模板的定义和写法和AngularJS的模板大同小异，只是循环、条件等模板语法做了一些调整。

### 1.4 元数据

用装饰器 (decorator) 来附加元数据。

```ts
@Component({
  selector:    'app-hero-list',
  templateUrl: './hero-list.component.html',
  providers:  [ HeroService ]
})
export class HeroListComponent implements OnInit {
/* . . . */
}
```

这里的`@Component`装饰器，它把紧随其后的类标记成了组件类。

`@Component`装饰器能接受一个*配置对象*， Angular 会基于这些信息创建和展示组件及其视图。

`@Component`的配置项包括：

- selector： CSS 选择器，它告诉 Angular 在父级 HTML 中查找`<hero-list>`标签，创建并插入该组件。
- templateUrl：组件 HTML 模板的模块*相对地址*。
- providers - 组件所需服务的依赖注入提供者数组。

元数据将模板、组件联系起来，构建了视图。

### 1.5 数据绑定

数据绑定的语法有四种形式。每种形式都有一个方向 —— 绑定到 DOM 、绑定自 DOM 以及双向绑定。

```html
<li>{{hero.name}}</li>
<app-hero-detail [hero]="selectedHero"></app-hero-detail>
<li (click)="selectHero(hero)"></li>
<input [(ngModel)]="hero.name">
```

- {{hero.name}}*插值表达式*在`<li>`标签中显示组件的`hero.name`属性的值。
- [hero]*属性绑定*把父组件的selectedHero的值传到子组件的hero属性中。
- (click) *事件绑定*在用户点击英雄的名字时调用组件的selectHero方法。
- *双向数据绑定*是重要的第四种绑定形式，它使用ngModel指令组合了属性绑定和事件绑定的功能。在双向绑定中，数据属性值通过属性绑定从组件流到输入框。用户的修改通过事件绑定流回组件，把属性值设置为最新的值。

Angular 在每个 JavaScript 事件循环中处理*所有*的数据绑定，它会从组件树的根部开始，递归处理全部子组件。

### 1.6 指令

组件是一个带模板的指令；`@Component`装饰器实际上就是一个`@Directive`装饰器，只是扩展了一些面向模板的特性。

### 1.7 服务

组件类应保持精简。组件本身不从服务器获得数据、不进行验证输入，也不直接往控制台写日志。 它们把这些任务委托给服务。

组件的任务就是提供用户体验，仅此而已。它介于视图（由模板渲染）和应用逻辑（通常包括模型的某些概念）之间。 设计良好的组件为数据绑定提供属性和方法，把其它琐事都委托给服务。

### 1.8 依赖注入

Angular 使用依赖注入来提供新组件以及组件所需的服务。Angular 通过查看*构造函数*的参数类型得知组件需要哪些服务。

当 Angular 创建组件时，会首先为组件所需的服务请求一个injector。

injector维护了一个服务实例的容器，存放着以前创建的实例。 如果所请求的服务实例不在容器中，injector就会创建一个服务实例，并且添加到容器中，然后把这个服务返回给 Angular。 当所有请求的服务都被解析完并返回时，Angular 会以这些服务为参数去调用组件的构造函数，这就是依赖注入 。

如果injector还没有`HeroService`，它怎么知道该如何创建一个呢？

简单点说，我们必须先用injector为`HeroService`注册一个provider。 provider用来创建或返回服务，通常就是这个服务类本身（相当于`new HeroService()`）。

我们可以在模块中或组件中注册 provider 。但通常会把 provider 添加到根模块上，以便在任何地方都使用服务的同一个实例。或者，也可以在`@Component`元数据中的 providers 属性中把它注册在组件层。把它注册在组件级表示该组件的每一个新实例都会有一个服务的新实例。

需要记住的关于依赖注入的要点是：

- 依赖注入渗透在整个 Angular 框架中，被到处使用，injector 是本机制的核心。
- injector 负责维护一个*容器*，用于存放它创建过的服务实例。
- 把 provider 注册到 injector。
- provider 是一个用于创建服务的配方(recipe)。
- injector 能使用 provider 创建一个新的服务实例。

## 2 模板与数据绑定

### 2.1 显示数据

使用插值表达式（包裹在双花括号里的 JavaScript 表达式）来显示组件中的属性。一般来说，括号间的内容是一个**模板表达式**，Angular 先对它**求值**，再把它**转换**成字符串。这个表达式还可以调用宿主组件的**方法**。

使用`NgFor`指令来进行列表渲染，用法：`*ngFor`

为数据创建一个类：

```ts
export class Hero {
  constructor(
    public id: number,
    public name: string) { }
}
```

它可能看上去不像是有属性的类，但它确实有，利用的是 TypeScript 提供的简写形式 —— 用构造函数的参数直接定义属性。

这个简写语法做了很多：

- 声明了两个构造函数参数及其类型
- 声明了两个同名的公共属性
- 当我们new出该类的一个实例时，把该属性初始化为相应的参数值

通过 NgIf 指令进行条件显示，用法：*ngIf

> Angular 并不是在显示和隐藏这条消息，它是在从 DOM 中添加和移除这个段落元素。 这会提高性能，特别是在一些大的项目中有条件地包含或排除一大堆带着很多数据绑定的 HTML 时。

### 2.2 模板语法

在 Angular 中，组件扮演着Model-View-Controller里的控制器(C)或Model-View-ViewModel里的视图模型(VM)的角色，模板则扮演视图(V)的角色。

#### 2.2.1 模板表达式

编写模板表达式所用的语言看起来很像 JavaScript。 很多 JavaScript 表达式也是合法的模板表达式，但不是全部。

JavaScript 中那些具有或可能引发副作用的表达式是被禁止的，包括：

- 赋值 (=, +=, -=, ...)
- new运算符
- 使用;或,的链式表达式
- 自增或自减操作符 (++和--)

和 JavaScript语 法的其它显著不同包括：

- 不支持位运算|和&
- 具有新的模板表达式运算符，比如|、?.和!。（后文会介绍）

典型的表达式上下文就是这个组件实例。表达式的上下文还可以包括组件之外的对象。 比如模板输入变量 (let hero)和模板引用变量(#heroInput)就是备选的上下文对象之一。

```html
<div *ngFor="let hero of heroes">{{hero.name}}</div>
<input #heroInput> {{heroInput.value}}
```

表达式中的上下文变量是由模板输入变量(let hero)、指令的上下文变量（如果有）和组件的成员叠加而成的。 如果我们要引用的变量名存在于一个以上的命名空间中，那么，模板变量是最优先的，其次是指令的上下文变量，最后是组件的成员。

模板表达式不能引用全局命名空间中的任何东西，比如window或document。它们也不能调用console.log或Math.max。 它们只能引用表达式上下文中的成员。

#### 2.2.2 模板语句

模板语句用来响应由绑定目标（如 HTML 元素、组件或指令）触发的**事件**。语法为：`(event)="statement"`。

```html
<button (click)="deleteHero()">Delete hero</button>
```

和模板表达式一样，模板语句使用的语言也像 JavaScript。 模板语句解析器和模板表达式解析器有所不同，特别之处在于它支持基本赋值 (=) 和表达式链 (;和,)。

典型的语句上下文就是当前组件的实例。模板语句上下文还可以引用模板自身上下文中的属性。 在下面的例子中，就把模板的$event对象、模板输入变量 (let hero)和模板引用变量 (#heroForm)传给了组件中的一个事件处理器方法。

```html
<button (click)="onSave($event)">Save</button>
<button *ngFor="let hero of heroes" (click)="deleteHero(hero)">{{hero.name}}</button>
<form #heroForm (ngSubmit)="onSubmit(heroForm)"> ... </form>
```

模板上下文中的变量名的优先级高于组件上下文中的变量名。

#### 2.2.3 数据绑定

绑定的类型可以根据数据流的方向分成三类： 从数据源到视图、从视图到数据源以及双向的从视图到数据源再到视图。