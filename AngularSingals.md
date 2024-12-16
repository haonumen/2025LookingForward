# Angular 19 的 Singals (信号)
Angular Signals是一个系统，它可以细粒度地跟踪你的状态在整个应用程序中的使用方式和位置，从而允许框架优化渲染更新。
## Singals 是什么
信号是一个值的包装器，当该值发生变化时通知感兴趣的消费者。信号可以包含任何值，从原始类型到复杂的数据结构。
你可以通过调用信号的getter函数来读取信号的值，该函数允许Angular跟踪信号的使用位置。
信号可以是可写的，也可以是只读的。

### 可写信号 （Writable signals）
可写信号类型是 WritableSignal.
可写信号提供了一个API来直接更新它们的值。通过使用信号的初始值调用signal函数来创建可写信号。
```TypeScript
const count = signal(0);
// Signals are getter functions - calling them reads their value.
console.log('The count is: ' + count());
```
要更改可写信号的值，可以直接使用.set()。
```TypeScript
count.set(3);
```
或者使用.update（）操作从前一个值计算一个新值。
```TypeScript
// Increment the count by 1.
count.update(value => value + 1);
```

### 计算信号 （Computed signals）
计算信号是只读信号，其值来源于其他信号。你可以使用computed函数定义计算信号并指定一个来源。
```TypeScript
const count: WritableSignal<number> = signal(0);
const doubleCount: Signal<number> = computed(() => count() * 2);
```
doubleCount信号依赖于count信号。当count更新时，Angular知道doubleCount也需要更新。
**计算信号是惰性计算并且被缓存（记忆）的**
在您第一次读取doubleCount之前，不会运行doubleCount的派生函数来计算它的值。然后缓存计算的值，如果再次读取doubleCount，它将返回缓存的值，而无需重新计算。
如果你改变了count， Angular就知道doubleCount的缓存值不再有效，下次读取doubleCount时，它的新值就会被计算出来。
因此，您可以安全地在计算信号中执行计算代价高昂的推导，例如过滤数组。
**计算信号不是可写信号**
不能直接为计算信号赋值。也就是说,
```TypeScript
doubleCount.set(3);
```
产生编译错误，因为doubleCount不是WritableSignal。
**计算信号的依赖关系是动态的**
只有在推导过程中实际读取的信号才会被跟踪。例如，在这个计算中，count信号只有在showCount信号为ture时才会被读取：
```TypeScript
const showCount = signal(false);
const count = signal(0);
const conditionalCount = computed(() => {
  if (showCount()) {
    return `The count is ${count()}.`;
  } else {
    return 'Nothing to see here!';
  }
});
```
当读取conditionalCount时，如果showCount为假，则在不读取计数信号的情况下返回“Nothing to see here!”消息。这意味着，如果稍后更新count，将不会导致重新计算conditionalCount。
如果将showCount设置为true，然后再次读取conditionalCount，则推导过程将重新执行并获取showCount为true的分支，返回显示count值的消息。更改count将使conditionalCount的缓存值失效。
**请注意**在推倒过程中可以删除依赖项，也可以添加依赖项。如果稍后将showCount设置回false，则count将不再被视为conditionalCount的依赖项。

### 读取OnPush组件中的信号
当你在OnPush组件的模板中读取信号时，Angular会将该信号作为该组件的依赖项来跟踪。当该信号的值发生变化时，Angular会自动标记该组件，以确保它在下次运行变更检测时得到更新。

### 效果器 （Effects）
信号很有用，因为当它们发生变化时，它们会通知感兴趣的消费者。效果器是当一个或多个信号值发生变化时运行的操作。你可以用effect函数创建一个效果器：
```TypeScript
effect(() => {
  console.log(`The current count is: ${count()}`);
});
```
效果器总是至少运行一次。当一个效果器运行时，它会跟踪读取的任何信号值。每当这些信号值中的任何一个改变时，效果就会再次运行。与计算信号类似，效果器动态地跟踪它们的依赖关系，并且只跟踪在最近执行中读取的信号。
在变更检测过程中，效果器总是异步执行的。
#### 效果器用例
在大多数应用程序代码中很少需要效果器，但在特定情况下可能很有用。以下是一些效果器可能是一个很好的解决方案的例子：
- 记录正在显示的数据以及数据更改的时间，用于分析或作为调试工具。
- 与window.localStorage保持数据同步。
- 添加无法用模板语法表达的自定义DOM行为。
- 对\<canvas>、图表库或其他第三方UI库执行自定义呈现。
***什么时候不用效果器***
避免使用效果器来传播状态变化。这可能导致ExpressionChangedAfterItHasBeenChecked错误、无限循环更新或不必要的更改检测周期。
相反，使用计算信号来模拟依赖于其他状态的状态。
#### 注入上下文 （Injection context）
默认情况下，你只能在注入上下文中创建effect()（在这个上下文中你可以访问inject函数）。要满足这个要求，最简单的方法是在组件、指令或服务构造函数中调用effect：
```TypeScript
@Component({...})
export class EffectiveCounterComponent {
  readonly count = signal(0);
  constructor() {
    // Register a new effect.
    effect(() => {
      console.log(`The count is: ${this.count()}`);
    });
  }
}
```
或者，您可以将效果器分配给一个字段（这也给了它一个描述性的名称）。
```TypeScript
@Component({...})
export class EffectiveCounterComponent {
  readonly count = signal(0);
  private loggingEffect = effect(() => {
    console.log(`The count is: ${this.count()}`);
  });
}
```
要在构造函数之外创建一个效果器，你可以通过它的选项将注入器传递给effect
```TypeScript
@Component({...})
export class EffectiveCounterComponent {
  readonly count = signal(0);
  constructor(private injector: Injector) {}
  initializeLogging(): void {
    effect(() => {
      console.log(`The count is: ${this.count()}`);
    }, {injector: this.injector});
  }
}
```
#### 销毁效果器
当您创建一个效果器时，当它的封闭上下文被销毁时，它将自动被销毁。这意味着当组件被销毁时，在组件中创建的效果器也会被销毁。指令、服务等内部的效果也是如此。
效果器返回一个EffectRef，你可以通过调用.destroy（）方法来手动销毁它们。您可以将此选项与manualCleanup选项结合使用，以创建一个持续到手动销毁为止的效果器。当不再需要这些效果时，要小心清理它们。

### 高级主题
#### 信号相等函数
在创建信号时，您可以选择提供一个相等函数，该函数将用于检查新值是否实际上与前一个值不同。
```TypeScript
import _ from 'lodash';
const data = signal(['test'], {equal: _.isEqual});
// Even though this is a different array instance, the deep equality
// function will consider the values to be equal, and the signal won't
// trigger any updates.
data.set(['test']);
```
等式函数可以提供给可写信号和可计算信号。
默认情况下，信号使用引用相等（Object.is（）比较）。
#### 不跟踪依赖项的读取
很少情况下，您可能希望在不创建依赖项的情况下执行可能读取响应函数（如computed或effect）中的信号的代码。例如，假设当currentUser发生变化时，应该记录计数器的值。你可以创建一个读取两个信号的效果器：
```TypeScript
effect(() => {
  console.log(`User set to ${currentUser()} and the counter is ${counter()}`);
});
```
此示例将在currentUser或计数器更改时记录一条消息。然而，如果效果应该只在currentUser更改时运行，那么读取计数器只是偶然的，对计数器的更改不应该记录新消息。你可以通过使用untracked来调用它的getter来防止信号被跟踪：
```TypeScript
effect(() => {
  console.log(`User set to ${currentUser()} and the counter is ${untracked(counter)}`);
});
```
当效果器需要调用一些不应被视为依赖的外部代码时，Untracked也很有用：
```TypeScript
effect(() => {
  const user = currentUser();
  untracked(() => {
    // If the `loggingService` reads signals, they won't be counted as
    // dependencies of this effect.
    this.loggingService.log(`User set to ${user}`);
  });
});
```
#### 效果器清理函数
效果器可能会启动长时间运行的操作，如果效果器被破坏或在第一个操作完成之前再次运行，则应该取消这些操作。当你创建一个效果器时，你的函数可以选择接受一个onCleanup函数作为它的第一个参数。这个onCleanup函数允许您注册一个回调函数，该回调函数将在效果的下一次运行开始之前或在效果器被销毁时调用。
```TypeScript
effect((onCleanup) => {
  const user = currentUser();
  const timer = setTimeout(() => {
    console.log(`1 second ago, the user became ${user}`);
  }, 1000);
  onCleanup(() => {
    clearTimeout(timer);
  });
});
```