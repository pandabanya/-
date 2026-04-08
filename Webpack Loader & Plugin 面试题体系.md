# Webpack Loader & Plugin 面试题体系

## 面试题 1：Loader 和 Plugin 的本质区别是什么？

- Loader 是文件转换器, Plugin 是功能扩展器
- Loader 在模块加载时工作,Plugin 在整个构建生命周期工作
- Loader 是单一职责（处理特定文件）,Plugin 可以做任何事


一般常见的Loader 比如像
```plaintxt
// 样式处理
style-loader    // 把 CSS 插入到 DOM
css-loader      // 解析 @import 和 url()
sass-loader     // 编译 Sass/SCSS
postcss-loader  // CSS 后处理（自动加前缀等）

// 脚本处理
babel-loader    // ES6+ 转 ES5
ts-loader       // TypeScript 编译

// 资源处理
file-loader     // 文件输出到目录
url-loader      // 小文件转 base64
image-webpack-loader // 图片压缩

// 其他
vue-loader      // 解析 .vue 单文件组件

```

常见的 Plugin 有哪些？分别解决什么问题？
```plaintxt
// HTML 处理
HtmlWebpackPlugin          // 自动生成 HTML 并注入资源

// 代码优化
TerserPlugin              // JS 压缩混淆
MiniCssExtractPlugin      // CSS 提取成独立文件
OptimizeCSSAssetsPlugin   // CSS 压缩

// 开发体验
HotModuleReplacementPlugin // 热更新
FriendlyErrorsPlugin      // 友好的错误提示

// 分析工具
BundleAnalyzerPlugin      // 打包体积分析
SpeedMeasurePlugin        // 构建速度分析

// 其他
CleanWebpackPlugin        // 清理输出目录
CopyWebpackPlugin         // 复制静态资源
DefinePlugin              // 定义全局常量
```


