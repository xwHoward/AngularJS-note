# Angular 学习笔记

## 1. 架构

### 1.1 模块

每个 Angular 应用至少有一个模块（根模块），习惯上命名为AppModule。Angular 模块（无论是根模块还是特性模块）都是一个带有@NgModule装饰器的类。

NgModule是一个装饰器函数（用来装饰所有的Ng模块，Ng模块（带@NgModule装饰器的类）是 Angular 的基础特性之一），它接收一个用来描述模块属性的元数据对象。其中最重要的属性是：

- declarations - 声明本模块中拥有的视图类。Angular 有三种视图类：组件、指令和管道。
- exports - declarations 的子集，可用于其它模块的组件模板。
    > 根模块不需要导出任何东西，因为其它组件不需要导入根模块。
- imports - 本模块声明的组件模板需要的类所在的其它模块。
- providers - 服务的创建者，并加入到全局服务列表中，可用于应用任何部分。
- bootstrap - 指定应用的主视图（称为根组件），它是所有其它视图的宿主。只有根模块才能设置bootstrap属性。

我们通过*引导*根模块来启动应用。在开发期间，通常在一个main.ts文件中引导AppModule:

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

Angular 模块库：Angular 提供了一组 JavaScript 模块。可以把它们看做库模块。每个 Angular 库的名字都带有@angular前缀。

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

