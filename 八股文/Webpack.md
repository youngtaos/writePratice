# Webpack

## Webpack 和 vite 的区别

### 编译速度

- Webpack: Webpack 的构建速度相对较慢，尤其在大型项目中，因为`它需要分析整个依赖图，进行多次文件扫描和转译`。
- Vite: Vite 以开发模式下的极速构建著称。`它利用 ES 模块的特性，只构建正在编辑的文件，而不是整个项目`。这使得它在开发环境下几乎是即时的。

### 开发模式

- Webpack: Webpack 通常使用`热模块替换（HMR）`来实现快速开发模式，`但配置相对复杂`。
- Vite: Vite 的开发模式非常轻量且快速，`支持 HMR，但无需额外配置`，因为它默认支持。

### 配置复杂度

- Webpack: Webpack 的`配置相对复杂，特别是在处理不同类型的资源和加载器时`。
- Vite: Vite `鼓励零配置，使得项目起步非常简单`，但同时也支持自定义配置，使其适用于复杂项目。

### 插件生态

- Webpack: Webpack 拥有庞大的插件生态系统，适用于各种不同的需求。
- Vite: Vite 也有相当数量的插件，但相对较小，因为它的开发模式和构建方式减少了对一些传统插件的需求。

### 编译方式

- Webpack: Webpack `使用了多种加载器和插件来处理不同类型的资源`，如 JavaScript、CSS、图片等。
- Vite: Vite `利用 ES 模块原生支持，使用原生浏览器导入来处理模块`，不需要大规模的编译和打包。

### 应用场景

- Webpack: 适用于复杂的大型项目，特别是需要大量自定义配置和复杂构建管道的项目。
- Vite: 更适用于小到中型项目，或者需要快速开发原型和小型应用的场景。

### 打包原理

- Webpack: `Webpack 的打包原理是将所有资源打包成一个或多个 bundle 文件，通常是一个 JavaScript 文件`。
- Vite: `Vite 的打包原理是保持开发时的模块化结构，使用浏览器原生的导入机制，在生产环境中进行代码分割和优化`
