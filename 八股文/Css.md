# CSS

## Flex

### flex: 1 含义

flex ：1 即为 flex-grow ：1。
经常用作自适应布局，将父容器的 display：flex，侧边栏大小固定后，将内容区 flex：1，内容区则会自动放大占满剩余空间。

### flex:1 和 flex:auto 的区别

`flex: 1`

```
flex-grow : 1;
flex-shrink : 1;
flex-basis : 0%;
```

`flex: auto`

```
flex-grow : 1;
flex-shrink : 1;
flex-basis : auto;
```

> 0% 即 0 宽度

> auto 对应取原尺寸
