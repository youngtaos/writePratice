# React

## React Hooks

`React Hooks 要解决的问题是状态共享`，是继 render-props 和 higher-order components 之后的第三种状态共享方案，`不会产生 JSX 嵌套地狱问题`。

这个状态指的是状态逻辑，所以称为`状态逻辑复用`会更恰当，因为只共享数据处理逻辑，不会共享数据本身。

render-props 代码

```
function App() {
  return (
    <Toggle initial={false}>
      {({ on, toggle }) => (
        <Button type="primary" onClick={toggle}> Open Modal </Button>
        <Modal visible={on} onOk={toggle} onCancel={toggle} />
      )}
    </Toggle>
  )
}
```

React Hooks

```
function App() {
  const [open, setOpen] = useState(false);
  return (
    <>
      <Button type="primary" onClick={() => setOpen(true)}>
        Open Modal
      </Button>
      <Modal
        visible={open}
        onOk={() => setOpen(false)}
        onCancel={() => setOpen(false)}
      />
    </>
  );
}

```

> 可以看到，React Hooks 就像一个内置的打平 renderProps 库，我们可以随时创建一个值，与修改这个值的方法。看上去像 function 形式的 setState，其实这等价于依赖注入，与使用 setState 相比，这个组件是没有状态的。

### React Hooks 的优点

1. 多个状态不会产生嵌套，写法还是平铺的（renderProps 可以通过 compose 解决，可不但使用略为繁琐，而且因为强制封装一个新对象而增加了实体数量）
2. Hooks 可以引用其他 Hooks。
3. 更容易将组件的 UI 与状态分离。

### React Hooks 实现生命周期

#### useEffect

useEffect 拥有两个参数，第一个参数作为回调函数会在浏览器布局和绘制完成后调用，因此它不会阻碍浏览器的渲染进程。

第二个参数是一个数组

- 当数组存在并有值时，如果数组中的任何值发生更改，则每次渲染后都会触发回调。
- 当不传递第二个参数的时候，每一次渲染都会触发回调
- 当其为一个空数组的时候，回调只会被触发一次，类似于 componentDidMount

#### 模拟 componentDidMount

```
function Example() {
  useEffect(() => console.log('mounted'), []);
  return null;
}

```

#### 模拟 shouldComponentUpdate

```
const MyComponent = React.memo(
    _MyComponent,
    (prevProps, nextProps) => nextProps.count !== prevProps.count
)
```

React.memo 包裹一个组件来对它的 props 进行浅比较,但这不是一个 hooks，因为它的写法和 hooks 不同,`其实React.memo 等效于 PureComponent，但它只比较 props`。

#### 模拟 componentDidUpdate

```
const mounted = useRef();
useEffect(() => {
  if (!mounted.current) {
    mounted.current = true;
  } else {
   console.log('I am didUpdate')
  }
});
```

#### 模拟 componentWillUnmount

```
useEffect(() => {
  return () => {
    console.log('will unmount');
  }
}, []);
```

## React Fiber

React Fiber 是 React 库的`核心调度算法`，它负责管理组件的更新、虚拟 DOM 的构建以及最终 DOM 的更新。

Fiber 即纤程，与进程（Process）、线程（Thread）、协程（Coroutine）同为程序执行过程。
在 JS 中，协程的实现是 Generator，纤程的实现是 Fiber，他们也是代数效应在 JS 中的体现。
React Fiber 可以理解为在 React 内部实现的一套状态更新机制。支持任务不同优先级，可中断与恢复，并且恢复后可以复用之前的中间状态。

### React Fiber 结构

React Fiber 树是一种表示 React 组件层次结构和更新过程的数据结构。
它是 React Fiber 架构的核心部分，用于实现增量渲染、优先级调度和可中断恢复等特性

[Fiber 节点的属性定义](https://react.iamkasong.com/process/fiber.html#fiber%E7%9A%84%E5%90%AB%E4%B9%89)

```
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // 作为静态数据结构的属性
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null;

  // 用于连接其他Fiber节点形成Fiber树
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  this.ref = null;

  // 作为动态的工作单元的属性
  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;

  this.mode = mode;

  this.effectTag = NoEffect;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;

  // 调度优先级相关
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  // 指向该fiber在另一次更新时对应的fiber
  this.alternate = null;
}
```

```
<App>
  <Header />
  <Main>
    <Sidebar />
    <Content />
  </Main>
</App>
```

```
  App
  |
  Header
  |
  Main
  /    \
Sidebar  Content
```

#### React Fiber 使用这个树结构来实现以下目标：

> - 增量渲染： React Fiber 可以将渲染工作划分为小的任务单元，可以在每个任务之间进行绘制。这个树结构有助于确定哪些任务应该被执行，以及如何构建和更新虚拟 DOM 树。
> - 优先级调度： 每个 Fiber 节点都包含有关任务优先级的信息，这使得 React Fiber 能够根据任务的重要性来决定执行顺序。
> - Fiber 树的结构和信息允许 React Fiber 在渲染过程中中断任务，并在之后恢复，以支持异步渲染、服务端渲染和 Suspense 等特性。

### 调度策略

React Fiber 使用一种`协作式的任务调度策略`，它将任务拆分成小的单元，可以在不同的优先级下进行调度。这使得 React 能够更好地响应用户交互、优化性能，并确保页面渲染的平滑进行

#### 1. 任务单元和工作单元

在 React Fiber 中，任务被分解为一系列小的单元，每个单元被称为工作单元（Work Unit）。这些工作单元通常对应于组件的更新或渲染任务。React 将这些工作单元组织成一个任务队列。

以下是一个简化的工作单元结构的示例：

```
const workUnit = {
  type: 'updateState',
  payload: { key: 'count', value: 42 },
  callback: null,
  next: null,
};
```

在这个示例中，workUnit 表示一个要执行的任务，类型是更新状态（updateState），要更新的状态键值对是 `{ key: 'count', value: 42 }。callback 是任务完成后要执行的回调函数。

#### 2. 调度器和任务队列

React Fiber 使用一个调度器来管理任务队列。调度器会根据任务的优先级来决定下一个要执行的工作单元。通常，优先级较高的任务（例如用户交互）会被优先执行。

以下是一个简化的调度器示例：

```
const scheduler = {
  queue: [], // 任务队列
  scheduleWork(workUnit, priority) {
    // 将工作单元加入任务队列，根据优先级排序
    // ...
  },
  performWork() {
    // 执行任务队列中的工作单元
    // ...
  },
};

```

调度器的 scheduleWork 方法用于将工作单元加入任务队列，并根据优先级排序。performWork 方法用于执行任务队列中的工作单元。

#### 3. 协作式调度

React Fiber 使用协作式调度，意味着任务执行过程中可以主动让出控制权，以便执行其他任务。这是 react 通过 hack requestIdleCallback 和 requestAnimationFrame 等浏览器 API 来实现的。

以下是一个简化的协作式调度示例：

```
function performWork(workUnit) {
  // 执行工作单元
  // ...

  // 检查是否需要让出控制权
  if (shouldYield()) {
    requestIdleCallback(performWork);
  }
}

```

在这个示例中，shouldYield 函数用于检查是否需要让出控制权。如果浏览器有空闲时间，requestIdleCallback 将调用 performWork 函数以执行下一个工作单元。

> 综合上述示例，React Fiber 的任务调度过程可以被简化为以下步骤：
> 将工作单元加入任务队列，根据优先级排序。
> 执行任务队列中的工作单元，如果需要让出控制权，使用协作式调度机制。检查任务队列是否还有待执行的工作单元，如果有则返回步骤 2，否则结束任务调度。

### DOM 更新

**React Fiber 通过协调阶段（Reconciliation Phase）和提交阶段（Commit Phase）的组合来实现 DOM 更新。**

`协调阶段负责找出需要更新的部分并构建更新队列，而提交阶段负责将这些更新应用到实际的 DOM 上。`

以下是 React Fiber 如何进行 DOM 更新的详细过程：

#### 1.协调阶段（Reconciliation Phase）:

协调阶段是 React Fiber 中更新的第一阶段，`它负责确定哪些组件需要更新，以及如何更新`。协调阶段的核心任务是`比较新旧虚拟 DOM 树，找出需要变更的部分。`

**a. 新旧虚拟 DOM 对比**

React Fiber 会递归遍历新旧虚拟 DOM 树，比较它们之间的差异。这个过程通常被称为 "Diffing"。比较的规则包括：

- 组件类型是否相同。
- 组件的属性是否相同。
- 子节点是否相同。

通过比较，React Fiber 构建了一个更新队列，其中包含了需要进行更新的操作，包括新增、删除、更新等。

**b. Fiber 节点的标记**

在协调阶段，React Fiber 还会在 Fiber 节点上打上标记，表示这个节点需要进行更新。这些标记包括：

- Placement（新增）：表示需要在 DOM 中创建新的元素。
- Update（更新）：表示需要更新现有的元素。
- Deletion（删除）：表示需要从 DOM 中删除元素。

#### 2. 提交阶段（Commit Phase）:

提交阶段是 React Fiber 中更新的第二阶段，它负责将更新队列中的操作应用到实际的 DOM 上，确保用户看到正确的界面。

- **a. 执行 DOM 操作**

  在提交阶段，React Fiber 会遍历更新队列，执行相应的 DOM 操作，包括创建新元素、更新属性、删除元素等。这些操作是同步执行的，确保更新按照正确的顺序被应用。

- **b. 副作用处理**

  在执行 DOM 操作的同时，React Fiber 会处理一些副作用（Side Effects），这些副作用可能包括：

- 执行组件的生命周期方法（componentDidMount、componentDidUpdate、componentWillUnmount）。
- 触发钩子函数，如 useEffect 和 useLayoutEffect。
- 执行更新队列中的回调函数。
  React Fiber 使用一种双缓冲（Double Buffering）的机制来记录这些副作用，以确保在渲染过程中不会产生不一致的结果。

- **c. 清理工作**

  一旦提交阶段完成，React Fiber 会清理工作，包括重置 Fiber 节点的状态、清空更新队列等。
