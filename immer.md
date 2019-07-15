# immer- JS 不可变数据特性
> 只需修改当前树即可创建下一个不可变状态树

Immer（德语：always）是一个很小的组件，允许您以更方便的方式使用不可变状态。 它基于copy-on-write机制。

基本思想是您将所有更改应用于临时draftState，它是currentState的代理。 一旦完成所有改动，Immer将根据draftState的改动产生nextState。 这意味着您可以通过简单地修改数据来与数据进行交互，同时保留不可变数据的所有好处。

使用Immer就像拥有一个私人助理; 他拿了一封信（current state）并给你一份副本（draft）来记录改变。 一旦你完成，助理将拿你的draft并为你生成真正不可变的最终信件（next state）。

一个有意识的读者可能会注意到这与ImmutableJS的withMutations非常相似。 确实如此，但是通用并应用于普通的原生JavaScript数据结构（数组和对象），而无需进一步添加别的库。

## API
Immer包暴露了一个完成所有工作的默认函数。
```produce（currentState，producer：（draftState）=> void）：nextState```
还有一个curried overload，下面将对此进行解释。

## Example
```
import produce from "immer"

const baseState = [
    {
        todo: "Learn typescript",
        done: true
    },
    {
        todo: "Try immer",
        done: false
    }
]

const nextState = produce(baseState, draftState => {
    draftState.push({todo: "Tweet about it"})
    draftState[1].done = true
})
```
关于Immer的有趣之处在于baseState将不受影响，但nextState将反映对draftState所做的所有更改
```
// the new item is only added to the next state,
// base state is unmodified
expect(baseState.length).toBe(2)
expect(nextState.length).toBe(3)

// same for the changed 'done' prop
expect(baseState[1].done).toBe(false)
expect(nextState[1].done).toBe(true)

// unchanged data is structurally shared
expect(nextState[0]).toBe(baseState[0])
// changed data not (dûh)
expect(nextState[1]).not.toBe(baseState[1])
```

## Benefits
* 使用普通JavaScript对象和数组的不变性。 没有新的API可供学习！
* 强类型，没有基于字符串的路径选择器等。
* 结构共享开箱即用
* 冻结开箱即用
* 深度更新是轻而易举的
* Boilerplate reduction。 更少的噪音，更简洁的代码。

### Reducer Example
这是Immer在实践中可以做出的区别的一个简单例子。

### React.setState example

##Currying
将函数作为生成的第一个参数传递用于currying。 这意味着您将获得一个预先绑定的生成器，该生成器只需要一个状态来生成值。 生成器函数在draft和传入的任何参数中传递和使用。
```
// mapper will be of signature (state, index) => state
const mapper = produce((draft, index) => {
    draft.index = index
})

// example usage
console.dir([{}, {}, {}].map(mapper))
//[{index: 0}, {index: 1}, {index: 2}])
```

也可以很好地利用这种机制来进一步简化我们的示例:

```
import produce from 'immer'

const byId = produce((draft, action) => {
  switch (action.type) {
    case RECEIVE_PRODUCTS:
      action.products.forEach(product => {
        draft[product.id] = product
      })
      return
    })
  }
})
```
请注意，现在已将因子考虑在内（创建的reducer将接受一个状态，并使用它调用绑定的生成器）。

如果要使用此构造初始化未初始化状态，可以通过将初始状态作为第二个参数传递来生成：

```
import produce from "immer"

const byId = produce(
    (draft, action) => {
        switch (action.type) {
            case RECEIVE_PRODUCTS:
                action.products.forEach(product => {
                    draft[product.id] = product
                })
                return
        }
    },
    {
        1: {id: 1, name: "product-1"}
    }
)
```

### Fun with currying
一个随机有趣的例子只是为了灵感：一个巧妙的技巧是将Object.assign变成一个生产者来创建一个比普通的扩展运算符更聪明的“扩展”函数，因为如果结果没有改变，它就不会产生新的state。
```
import produce from "immer"
const spread = produce(Object.assign)

const base = {x: 1, y: 1}

console.log({...base, y: 1} === base) // false
console.log(spread(base, {y: 1}) === base) // true! base is recycled as no actual new value was produced
console.log(spread(base, {y: 2}) === base) // false, produced a new object as it should
```

## Patches
在producer运行期间，Immer可以记录所有可以重放迭代器所做更改的补丁。 如果您想暂时分叉状态并重放原始更改，这是一个非常强大的工具。

补丁在几种情况下很有用：
* 与其他方交换增量更新，例如通过websockets
* 对于调试/跟踪，要准确了解状态如何随时间变化
* 作为撤消/重做的基础或作为在稍微不同的状态树上重放更改的方法

为了帮助重放补丁，applyPatches派上用场。 以下是如何使用补丁记录增量更新和（inverse）应用它们的示例：
```
import produce, {applyPatches} from "immer"

let state = {
    name: "Micheal",
    age: 32
}

// 我们假设用户在向导中，我们不知道是否他的改动应该处于最终状态.....
let fork = state
// all the changes the user made in the wizard
let changes = []
// 逆变动
let inverseChanges = []

fork = produce(
    fork,
    draft => {
        draft.age = 33
    },
    // 第三个参数是patch的返回
    (patches, inversePatches) => {
        changes.push(...patches)
        inverseChanges.push(...inversePatches)
    }
)

// 与此同时，我们的原始状态被替换，例如，从服务器收到了一些更改
state = produce(state, draft => {
    draft.name = "Michel"
})

// 当向导完成（成功）后，我们可以将fork中的更改重放到* new * state！
state = applyPatches(state, changes)

// state now contains the changes from both code paths!
expect(state).toEqual({
    name: "Michel", // changed by the server
    age: 33 // changed by the wizard
})

// 最后，即使在完成向导之后，用户也可能改变主意并撤消他的更改
state = applyPatches(state, inverseChanges)
expect(state).toEqual({
    name: "Michel", // Not reverted
    age: 32 // Reverted
})
```

生成的补丁与RFC-6902 JSON补丁标准类似（但不相同），但path属性是数组，而不是字符串。 这使处理补丁更容易。 如果要规范化到官方规范，patch.path = patch.path.join（“/”）应该可以解决问题。 无论如何，这是一堆补丁和它们的反转看起来像:
```
[
    {
        "op": "replace",
        "path": ["profile"],
        "value": {"name": "Veria", "age": 5}
    },
    {"op": "remove", "path": ["tags", 3]}
]
```

## Async producers
允许返回Promise对象。 或者，换句话说，使用async / await。 这对于长时间运行的进程非常有用，只有在promise链解析resolve后才生成新对象。 请注意，如果生成器是异步的，则produce本身（即使是以curried形式）将返回promise。 例
```
import produce from "immer"

const user = {
    name: "michel",
    todos: []
}

const loadedUser = await produce(user, async function(draft) {
    draft.todos = await (await window.fetch("http://host/" + draft.name)).json()
})
```
警告：请注意，draft不应该从异步过程“泄露”并存储在其他位置。 异步过程完成后，draft将被删除。

## createDraft and finishDraft
createDraft和finishDraft是两个低级函数，主要用于在immer之上构建抽象的库。 它避免了为了使用draft而总是创建函数的需要。 相反，可以创建draft，修改它，并在将来的某个时间完成draft，在这种情况下，将生成下一个不可变状态。 例如，我们可以将上面的示例重写为：
```
import {createDraft, finishDraft} from "immer"

const user = {
    name: "michel",
    todos: []
}

const draft = createDraft(user)
draft.todos = await (await window.fetch("http://host/" + draft.name)).json()
const loadedUser = finishDraft(draft)
```
注意：finishDraft将patchListener作为第二个参数，可用于记录补丁，类似于produce。

警告：一般来说，我们建议使用产品而不是createDraft / finishDraft组合，产品使用时不易出错，并且更清楚地区分代码库中的可变性和不变性概念。

## Returning data from producers
不需要从producer返回任何内容，因为Immer将返回draft的（最终版）版本。 但是，可以返回draft。

它还允许从producer任意返回其他数据。 但是不能修改draft。 这对于产生一个全新的state很有用。 一些例子：
```
const userReducer = produce((draft, action) => {
    switch (action.type) {
        case "renameUser":
            // OK: we modify the current state
            draft.users[action.payload.id].name = action.payload.name
            return draft // same as just 'return'
        case "loadUsers":
            // OK: we return an entirely new state
            return action.payload
        case "adduser-1":
            // NOT OK: This doesn't do change the draft nor return a new state!
            // It doesn't modify the draft (it just redeclares it)
            // In fact, this just doesn't do anything at all
            draft = {users: [...draft.users, action.payload]}
            return
        case "adduser-2":
            // NOT OK: modifying draft *and* returning a new state
            draft.userCount += 1
            return {users: [...draft.users, action.payload]}
        case "adduser-3":
            // OK: returning a new state. But, unnecessary complex and expensive
            return {
                userCount: draft.userCount + 1,
                users: [...draft.users, action.payload]
            }
        case "adduser-4":
            // OK: the immer way
            draft.userCount += 1
            draft.users.push(action.payload)
            return
    }
})
```
注意：不可能以这种方式返回undefined，因为它与不更新草稿无法区分！ 继续阅读..

## Producing undefined using nothing
因此，通常，可以通过从return新值来替换当前状态，而不是修改draft。 但是有一个微妙的边缘情况：如果你试图编写一个想要用undefined替换当前state的producer：
```
produce({}, draft => {
    // don't do anything
})
or
produce({}, draft => {
    // Try to return undefined from the producer
    return undefined
})
```
问题是在JavaScript中，一个不返回任何东西的函数也返回undefined！ 因此，immer无法区分这些不同的情况。 因此，默认情况下，Immer将假设任何返回undefined的producer只是尝试修改draft。

但是，为了向Immer明确表示您有意想要生成undefined的值，您可以返回内置标记 nothing：
```
import produce, {nothing} from "immer"

const state = {
    hello: "world"
}

produce(state, draft => {})
produce(state, draft => undefined)
// Both return the original state: { hello: "world"}

produce(state, draft => nothing)
// Produces a new state, 'undefined'
```
注： 请注意，此问题特定于未定义的值，任何其他值（包括null）都不会受此问题的影响。

## Inline shortcuts using void
Immer中draft的变动通常覆盖代码块，因为返回表示覆盖。 有时，可以更舒服地扩展代码。

在这种情况下，您可以使用javascripts void operator，它可以计算表达式并返回undefined。
```
// Single mutation
produce(draft => void (draft.user.age += 1))

// Multiple mutations
produce(draft => void ((draft.user.age += 1), (draft.user.height = 186)))
```
代码风格是高度个性化的，但对于许多人需要理解的代码库，我们建议坚持使用标准 draft=> {draft.user.age + = 1}以避免学习成本。

## 从代理实例中提取原始对象
Immer公开了一个export original，它将从produce中的代理实例中获取原始对象（或者对于未经过代理的值返回undefined）。 这可能有用的一个很好的例子是使用严格相等搜索树状结构state的节点
```
const baseState = {users: [{name: "Richie"}]}
const nextState = produce(baseState, draftState => {
    original(draftState.users) // is === baseState.users
})
```
只想知道某个值是否是代理实例？ 使用isDraft功能！
```
import {isDraft} from "immer"

const baseState = {users: [{name: "Bobby"}]}
const nextState = produce(baseState, draft => {
    isDraft(draft) // => true
    isDraft(draft.users) // => true
    isDraft(draft.users[0]) // => true
})
isDraft(nextState) // => false
```
## Auto freezing
Immer自动冻结使用produce修改的任何状态树。 这可以防止producer之外的状态树的意外修改。 但是这会带来性能影响，因此建议在生产中禁用此选项。 它默认启用。 默认情况下，它在本地开发期间打开，在生产中关闭。 使用setAutoFreeze（true / false）可以显式打开或关闭此功能。

## Importing immer
产品作为默认导出公开，但也可以选择将其用作名称导入，因为这有利于在较旧的项目中使用。 所以以下导入都是正确的，建议第一个：
```
import produce from "immer"
import {produce} from "immer"

const {produce} = require("immer")
const produce = require("immer").produce
const produce = require("immer").default

import unleashTheMagic from "immer"
import {produce as unleashTheMagic} from "immer"
```
## Supported object types
普通对象和数组总是由Immer draft管理

其他对象必须使用 immerable 符号来标记自己与Immer兼容。 当其中一个对象在produce中发生变动时，其原型将保留在副本之间。
```
import {immerable} from "immer"

class Foo {
    [immerable] = true // Option 1

    constructor() {
        this[immerable] = true // Option 2
    }
}

Foo[immerable] = true // Option 3
```
对于数组，只能改变数字属性和length属性。 数组上不保留自定义属性。

使用Date对象时，应始终创建新的Date实例，而不是改变现有的Date对象。

不支持内置类，如Map和Set。 作为一种解决方法，您应该在producer中对它们进行变更之前克隆它们：
```
const state = {
    set: new Set(),
    map: new Map()
}
const nextState = produce(state, draft => {
    // Don't use any Set methods, as that mutates the instance!
    draft.set.add("foo") // ❌

    // 1. Instead, clone the set (just once)
    const newSet = new Set(draft.set) // ✅

    // 2. Mutate the clone (just in this producer)
    newSet.add("foo")

    // 3. Update the draft with the new set
    draft.set = newSet

    // Similarly, don't use any Map methods.
    draft.map.set("foo", "bar") // ❌

    // 1. Instead, clone the map (just once)
    const newMap = new Map(draft.map) // ✅

    // 2. Mutate it
    newMap.set("foo", "bar")

    // 3. Update the draft
    draft.map = newMap
})
```
TypeScript or Flow
Immer包内自带类型定义，应该由TypeScript和Flow拾取，无需进一步配置。

TypeScript类型会自动从 draft 类型中删除 readonly 修饰符，并返回与原始类型匹配的值。 看到这个实际例子：
```
import produce from "immer"

interface State {
    readonly x: number
}

// `x` cannot be modified here
const state: State = {
    x: 0
}

const newState = produce(state, draft => {
    // `x` can be modified here
    draft.x++
})
// `newState.x` cannot be modified here
```
这可以确保您可以修改状态的唯一位置是您的produce回调。 它甚至可以递归地使用ReadonlyArrays！

对于curried Reducer，类型是从recipe函数的第一个参数推断出来的，所以一定要传入。 如果state参数类型是不可变的，则可以使用Draft实用程序类型：
```
import produce, {Draft} from "immer"

interface State {
    readonly x: number
}

// `x` cannot be modified here
const state: State = {
    x: 0
}

const increment = produce((draft: Draft<State>, inc: number) => {
    // `x` can be modified here
    draft.x += inc
})

const newState = increment(state, 2)
// `newState.x` cannot be modified here
```
## Pitfalls(陷阱)
不要用类似 draft = myCoolNewState 这种方式重新定义draft。相反，要么修改draft，要么返回新状态。请参阅 Returning data from producers。
Immer假设您的状态是单向树。也就是说，没有对象应该在树中出现两次，并且不应该有循环引用。
由于Immer使用代理，从状态读取大量数据会带来开销（特别是在ES5实现中）。如果这成为一个问题（在优化之前进行性能测试），请在进入producer函数之前进行当前状态分析，或者从currentState而不是draftState进行state读取。另外，要意识到immer在所有地方都是选择性的，因此可以手动编写性能严格要求的reducer，并使用immer来完成普通工作。另请注意，original 可用于获取对象的原始状态，这样可以更利于访问。
总是试图拉动 produce '向上'，for (let x of y) produce(base, d => d.push(x)) 效率比produce(base, d => { for (let x of y) d.push(x)})更低
可以从producer返回值，但是，不能以这种方式返回undefined，因为它与根本未更新draft无法区分！如果要使用undefined替换draft的返回结果，只需从reducer返回nothing即可。
Example patterns.
```
import produce from "immer"

// object mutations
const todosObj = {
    id1: {done: false, body: "Take out the trash"},
    id2: {done: false, body: "Check Email"}
}

// add
const addedTodosObj = produce(todosObj, draft => {
    draft["id3"] = {done: false, body: "Buy bananas"}
})

// delete
const deletedTodosObj = produce(todosObj, draft => {
    delete draft["id1"]
})

// update
const updatedTodosObj = produce(todosObj, draft => {
    draft["id1"].done = true
})

// array mutations
const todosArray = [
    {id: "id1", done: false, body: "Take out the trash"},
    {id: "id2", done: false, body: "Check Email"}
]

// add
const addedTodosArray = produce(todosArray, draft => {
    draft.push({id: "id3", done: false, body: "Buy bananas"})
})

// delete
const deletedTodosArray = produce(todosArray, draft => {
    draft.splice(draft.findIndex(todo => todo.id === "id1"), 1)
    // or (slower):
    // return draft.filter(todo => todo.id !== "id1")
})

// update
const updatedTodosArray = produce(todosArray, draft => {
    draft[draft.findIndex(todo => todo.id === "id1")].done = true
})
```