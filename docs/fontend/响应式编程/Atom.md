# 响应式的基础——Atom

原子响应式是一种特殊的响应式模式，其基础底层为 Signal。在 Signal 之上建立起一组完备的响应式工具与开发关系。

> 下面的所有代码皆基于 SolidJS @cn-ui/reactive 实现的组件。

## Atom——基石

Atom，是定义在响应式作用域的响应式变量。其内部的值不可直接更改、直接读取。你可以认为 Atom 是一个函数，里面包裹着数据。当函数无入参时，将会返回值；当入餐为函数时，将可以改变内部的值。当响应式变量的值发生改变时，将会触发具有依赖关系的响应式变量的变更，从而将视图进行极细的颗粒度更新。Atom 类似于 Signal，但是其可以承载非常多便利的函数，为你的数据控制手段增光添彩，这个将在后面的文章中有所讲解。

```tsx
import { atom } from '@cn-ui/reactive'
export const App = () => {
    /** 创建一个Atom，并初始化其值 **/
    const count = atom(10)
    return (
        <section>
            <div>{count()}</div>
            <button onclick={() => count((i) => i + 1)}>+</button>
            <button onclick={() => count((i) => i - 1)}>-</button>
        </section>
    )
}
```

## Atom——变更

在前端项目中，对于响应式变量进行变更的场景是非常多的，每一次不同的场景都导致了这些响应式变量越来越难以被我们掌控。换句话说，我们越来越不清楚这个视图中的变量的转变带来的后果，产生的副作用常常让我们非常头疼。

Atom 的变更有两种，一种是值变更，一种是函数变更。值变更将会直接覆盖上一个值，并触发响应式变化。但值得注意的是，值变更不能是一个函数，因为值如果是函数，那么其将会与下面的函数变更混淆。

Atom 的函数变更是通过输入一个函数，获取其 return 的值来进行变更的。变更函数创建了一个新的作用域，通过在这个新作用域中操作上一个数据，返回不同的数据，这样你的逻辑就不会过度与其他变更的代码混淆在一起。

> 这样设计的好处是可以通过 VSCode 或者其他编辑器的重构功能，将这个变更过程提取出来，方便你将其重构为更纯净的过程。同时，对于简短的数值变更，你可以直接将其写在 JSX 中，它能和箭头函数友好相处。

```tsx
const App = () => {
    const count = atom(100) // 初始化
    const change = () => count(200) // 值变更
    // 函数变更
    return <div onclick={() => count((lastVal) => lastVal - 1)}>减一</div>
}
```

## Atom——衍生（Reflect）

在 Vue 中，通过 `computed` 可以衍生出一个新的响应式变量，这个响应式变量可以自动计算内部函数，并在每一次依赖的响应式变量发生变化时进行重新计算。

而在原子的世界中，原子即是一切响应式变量的起点，你可以通过衍生（Reflect）来构建一个原子。这个衍生出来的原子能够像 `computed` 一样自动根据依赖计算自身的值，同时其也是原子，也可以进行上一小结所演示的变更操作。

```tsx
const App = () => {
    const count = atom(100)
    const isEnough = reflect(() => count() > 98)
    // isEnough(true) // 临时变更数值
    return (
        <>
            <div onclick={() => count((lastVal) => lastVal - 1)}>减一</div>
            <button>{isEnough() ? '小于98' : '太大了'}</button>
        </>
    )
}
```

衍生这一功能对于构建响应式数据流是非常有效的。衍生让数据流在 JS 作用域中生成一个节点，这个节点代表了这个地方的数据需要被展示，需要产生更新视图的副作用，而后在 JSX 中将会用它来进行视图的变更。或者，这个节点可以被其他的数据所依赖，从而实现一个便利的、自动计算的、像静脉血管那样精细的响应式代码逻辑。

### 突变（mutation）

突变（mutation）是当 Atom 具有自动依赖时，由外部主动触发该节点的数据更新，从而直接影响该节点以后数据的计算结果的编程现象。

突变的好处是，你可以直接影响这条响应式数据流的中间及后续部分，而不用担心污染上游数据，下游的数据也可以方便地遵从你的逻辑去进行自动计算。而且，下一次上游发生变化时，自动会将突变取消，而不用担心产生额外的错误信息。

```tsx
const App = () => {
    const count = atom(Math.random() * 100)
    const isCountAbove50 = reflect(() => form().count > 50)
    const message = reflact(() => (isCountAbove50() ? 'large' : 'small'))
    return (
        <>
            <div>
                {message()} - {count()}
            </div>
            <button onclick={() => count(Math.random() * 100)}>重置</button>
            <button onclick={() => isCountAbove50((i) => !i)}>突变操作</button>
        </>
    )
}
```
