# Angular 组件（v19）

Angular团队在开发新代码时推荐使用独立组件（standalone components）来替代 NgModule。

## NgModule
NgModule是一个用@NgModule装饰器标记的类。 @NgModule装饰器使用metadata作为入参，并且告诉Angular如何编译模版组件以及如何配置依赖注入。

```TypeScript
import {NgModule} from '@angular/core';
@NgModule({
  // Metadata goes here
})
export class CustomMenuModule { }
```
一个NgModule的两个主要职责：
- 声明属于NgModule的组件、指令和管道
- 在注入器中为导入NgModule的组件、指令和管道添加提供者（Providers）
1
### 声明（ declarations ）
NgModule元数据的 declarations 属性声明了属于NgModule的组件、指令和管道。

```TypeScript
@NgModule({
  /* ... */
  // CustomMenu and CustomMenuItem are components.
  declarations: [CustomMenu, CustomMenuItem],
})
export class CustomMenuModule { }
```
declarations属性还接受组件、指令和管道的数组。这些数组反过来也可以包含其他数组。

**Angular不允许任何组件、指令和管道在多个NgModule中声明，** 如果你那么做，Angular会报错。

**要想在NgModule中声明**，任何组件、指令或管道都必须显式地标记为 *standalone: false* 

```TypeScript
@Component({
  // Mark this component as `standalone: false` so that it can be declared in an NgModule.
  standalone: false,
  /* ... */
})
export class CustomMenu { /* ... */ }
```

### 导入（ imports ）
在NgModule中声明的组件也许依赖于其他的组件、指令和管道。我们可以使用@NgModule 元数据（metadata）的imports属性将这些依赖引入。

```TypeScript
@NgModule({
  /* ... */
  // CustomMenu and CustomMenuItem depend on the PopupTrigger and SelectorIndicator components.
  imports: [PopupTrigger, SelectionIndicator],
  declarations: [CustomMenu, CustomMenuItem],
})
export class CustomMenuModule { }
```
*imports* 数组可以接受其他的NgModules，同样可以接受独立组件、指令和管道.

### 导出（ exports ）
NgModule可以导出它声明过的组件、指令和管道，这样它们就可以被其他组件和NgModule使用。

```TypeScript
@NgModule({
  imports: [PopupTrigger, SelectionIndicator],
  declarations: [CustomMenu, CustomMenuItem],
  // Make CustomMenu and CustomMenuItem available to
  // components and NgModules that import CustomMenuModule.
  exports: [CustomMenu, CustomMenuItem],
})
export class CustomMenuModule { }
```
但是，exports属性并不局限于声明。NgModule还可以导出它导入的任何其他组件、指令、管道和NgModule。

```TypeScript
@NgModule({
  imports: [PopupTrigger, SelectionIndicator],
  declarations: [CustomMenu, CustomMenuItem],
  // Also make PopupTrigger available to any component or NgModule that imports CustomMenuModule.
  exports: [CustomMenu, CustomMenuItem, PopupTrigger],
})
export class CustomMenuModule { }
```

### NgModule 提供者 （providers）
NgModule可以为注入的依赖指定提供者。这些提供者可用于：
- 任何导入了NgModule的独立组件、指令或管道
- 导入该NgModule的任何其他NgModule的声明和提供者
```TypeScript
@NgModule({
  imports: [PopupTrigger, SelectionIndicator],
  declarations: [CustomMenu, CustomMenuItem],
  // Provide the OverlayManager service
  providers: [OverlayManager],
  /* ... */
})
export class CustomMenuModule { }
@NgModule({
  imports: [CustomMenuModule],
  declarations: [UserProfile],
  providers: [UserDataClient],
})
export class UserProfileModule { }
```

#### forRoot和forChild模式
一些ngmodule定义了一个静态的forRoot方法，它接受一些配置并返回一个提供者数组。名称“forRoot”是一种约定，表明这些提供程序打算在引导期间专门添加到应用程序的根上。
以这种方式包含的任何提供者都是**主动加载**的，从而增加了初始页面加载的JavaScript包大小。
```TypeScript
boorstrapApplication(MyApplicationRoot, {
  providers: [
    CustomMenuModule.forRoot(/* some config */),
  ],
});
```
类似地，一些ngmodule可能会定义一个静态的forChild，表明这些提供者将被添加到应用程序层次结构中的组件中。

```TypeScript
@Component({
  /* ... */
  providers: [
    CustomMenuModule.forChild(/* some config */),
  ],
})
export class UserProfile { /* ... */ }
```

### 引导应用程序
Angular团队建议在所有新代码中使用**bootstrapApplication**而不是**bootstrapModule**。

@NgModule装饰器接受一个可选的bootstrap数组，它可以包含一个或多个组件。
你可以在platformBrowser或platformServer中使用bootstrapModule方法来启动Angular应用。当运行时，该函数使用与所列组件匹配的CSS选择器定位页面上的任何元素，并在页面上呈现这些组件。

```TypeScript
import {platformBrowser} from '@angular/platform-browser';
@NgModule({
  bootstrap: [MyApplication],
})
export class MyApplciationModule { }
platformBrowser().bootstrapModule(MyApplicationModule);
```
在bootstrap中列出的组件会自动包含在NgModule的声明中。当你从NgModule中引导一个应用时，这个模块收集到的提供者和它导入的所有提供者都会被主动加载，并可以注入到整个应用中。

## 独立组件 standalone component
组件必须包括以下：
- 一个TypeScript类，具有处理用户输入和从服务器获取数据等行为
- 一个HTML模板，控制呈现到DOM的内容
- 一个CSS选择器，用于定义组件在HTML中的使用方式
你可以通过在TypeScript类的顶部添加@Component装饰器来为组件提供angular特有的信息

```TypeScript
@Component({
  selector: 'profile-photo',
  template: `<img src="profile-photo.jpg" alt="Your profile photo">`,
})
export class ProfilePhoto { }
```

传递给@Component装饰器的对象被称为组件的元数据。它包括选择器、模板和其他属性。
```TypeScript
@Component({
  selector: 'profile-photo',
  templateUrl: 'profile-photo.html',
  styleUrl: 'profile-photo.css',
})
export class ProfilePhoto { }
```
templateUrl和styleUrl都是相对于组件所在的目录的。

### 使用独立组件
要使用组件、指令或管道，你必须把它们添加到@Component装饰器中的imports数组中
```TypeScript
import {ProfilePhoto} from './profile-photo';
@Component({
  // Import the `ProfilePhoto` component in
  // order to use it in this component's template.
  imports: [ProfilePhoto],
  /* ... */
})
export class UserProfile { }
```

默认情况下，Angular组件是独立组件，这意味着你可以直接将它们添加到其他组件的imports数组中。用早期版本的Angular创建的组件可能会在它们的@Component装饰器中指定standalone: false。对于这些组件，你需要导入定义该组件的NgModule。

**重要**：在19.0.0之前的Angular版本中，standalone选项默认为**false**。

**显示模板中的组件**
每个组件定义一个CSS selector
```TypeScript
@Component({
  selector: 'profile-photo',
  ...
})
export class ProfilePhoto { }
```

你可以通过在其他组件的模板中创建一个匹配的HTML元素来显示组件：
```TypeScript
@Component({
  selector: 'profile-photo',
})
export class ProfilePhoto { }
@Component({
  imports: [ProfilePhoto],
  template: `<profile-photo />`
})
export class UserProfile { }
```
Angular会为遇到的每个匹配的HTML元素创建一个组件实例。与组件的选择器匹配的DOM元素被称为该组件的宿主元素。组件模板的内容在其宿主元素中呈现。

由组件呈现的DOM，对应于该组件的模板，被称为该组件的视图(View)。

### bootstrapApplication
bootstrapApplication 引导Angular应用的一个实例，并将一个独立组件呈现为应用的根组件。

传入这个函数的根组件必须是一个独立的组件（应该在@Component装饰器配置中有standalone: true标志）。
```TypeScript
@Component({
  standalone: true,
  template: 'Hello world!'
})
class RootComponent {}
const appRef: ApplicationRef = await bootstrapApplication(RootComponent);
```
