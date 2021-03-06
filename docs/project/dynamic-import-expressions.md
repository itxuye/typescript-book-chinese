# 动态导入表达式

动态导入表达式是 ECMASript 的一个新功能，它允许你在你程序的任意地方异步加载一个模块，TC39 JavaScript 委员会有这样一个处于 stage 3 的一个提案，它被称为 [import() proposal for JavaScript](https://github.com/tc39/proposal-dynamic-import)。

此外，**webpack** bunder 有一个被称为 [`Code Splitting`](https://webpack.js.org/guides/code-splitting/) 的功能，它能允许你将代码拆分为许多块，这些块在将来可被异步下载。这意味着，你可以在程序中首先提供一个最小的程序启动包，并在将来异步加载其他模块。

这很自然的想到（如果我们工作在 dev 的工作流程）[TypeScript 2.4 dynamic import expressions](https://github.com/Microsoft/TypeScript/wiki/What%27s-new-in-TypeScript#dynamic-import-expressions) 将会把你最终生成的 JS 代码自动分割成很多代码块。但是这似乎并不容易实现，因为它依赖于我们正在使用的 `tsconfig.json` 配置文件。

webpack 实现代码分割的方式有两种：使用 `import()` （首选，ECMAScript 的提案）和 `require.ensure()` （最后考虑，webpack 具体实现）。因此，我们期望 TypeScript 的输出是保留 `import()` 语句，而不是将其转化为其他任何代码。

让我们来看一个例子，在这个例子中，我们演示了如何配置 webpack 和 TypeScript 2.4 +。

在下面的代码，我希望懒加载 `moment` 库，但是我也对代码分割感兴趣，这意味 `moment` 会被分割到一个单独的 JavaScript 文件，当它被使用时，会被异步加载。

```typescript
import(/* webpackChunkName: "momentjs" */ 'moment')
  .then((moment) => {
    // 懒加载的模块拥有所有的类型，并且能够按期工作
    // 类型检查会工作，代码引用也会工作  :100:
    const time = moment().format()
    console.log('TypeScript >= 2.4.0 Dynamic Import Expression:')
    console.log(time);
  })
  .catch((err) => {
    console.log('Failed to load moment', err)
  })
```

这是 `tsconfig.json` 的配置文件：

```javascript
{
  "compilerOptions": {
    "target": "es5",
    "module": "esnext",
    "lib": [
      "dom",
      "es5",
      "scripthost",
      "es2015.promise"
    ],
    "jsx": "react",
    "declaration": false,
    "sourceMap": true,
    "outDir": "./dist/js",
    "strict": true,
    "moduleResolution": "node",
    "typeRoots": [
      "./node_modules/@types"
    ],
    "types": [
      "node",
      "react",
      "react-dom"
    ]
  }
}
```

:::danger 重要的提示

- 使用 "module": "esnext" 选项：TypeScript 为 Webpack Code Splitting 生成 `import()` 语句。
- 进一步了解有关信息，推荐阅读这篇文章：[Dynamic Import Expressions and webpack 2 Code Splitting integration with TypeScript 2.4.](https://blog.josequinto.com/2017/06/29/dynamic-import-expressions-and-webpack-code-splitting-integration-with-typescript-2-4/)

:::
