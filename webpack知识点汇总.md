Compiler 关注的是「一次完整构建流程」；Compilation 关注的是「一次构建中的具体产物生成过程」。
Compiler hooks 主要覆盖 Webpack 从启动、配置解析、开始构建、创建 compilation 到构建结束的完整生命周期，适合做全局初始化和构建级别的逻辑。
- **Compiler hooks：构建级、全局级**
- **Compilation hooks：模块 / chunk / asset 级**
---
启动阶段：
compiler.hooks.initialize
compiler.hooks.environment 
compiler.hooks.afterEnvironment

配置 & 编译准备
compiler.hooks.entryOption 
compiler.hooks.afterPlugins 
compiler.hooks.afterResolvers

真正开始构建
compiler.hooks.beforeRun 
compiler.hooks.run        // 单次构建 
compiler.hooks.watchRun   // watch 模式

创建 Compilation（极其重要）
compiler.hooks.thisCompilation   // ⭐⭐⭐ **推荐使用**，支持 Webpack 5 新特性
compiler.hooks.compilation

构建结束
compiler.hooks.done 
compiler.hooks.failed 
compiler.hooks.afterDone

---
**Compilation 表示“一次构建的上下文”，包括模块、依赖图、chunk、asset。**
- watch 下每次 rebuild 都会 new 一个 Compilation
- loader、parser、chunk 都在这里发生
- Compilation hooks 主要用于介入模块构建、chunk 生成和 asset 处理过程，适合做代码分析、资源优化、构建报告等细粒度操作。

模块构建阶段
compilation.hooks.buildModule 
compilation.hooks.succeedModule 
compilation.hooks.failedModule

依赖 & 解析
compilation.hooks.finishModules

Chunk 生成阶段（高频）
compilation.hooks.beforeChunks 
compilation.hooks.afterChunks 
compilation.hooks.optimizeChunks 
compilation.hooks.afterOptimizeChunks

Asset 处理（Webpack 5 核心）
compilation.hooks.processAssets  // ⭐⭐⭐
带 stage：
PROCESS_ASSETS_STAGE_ADDITIONS 
PROCESS_ASSETS_STAGE_OPTIMIZE 
PROCESS_ASSETS_STAGE_SUMMARIZE 
PROCESS_ASSETS_STAGE_REPORT

输出阶段
compilation.hooks.afterProcessAssets 
compilation.hooks.afterSeal

| 维度      | Compiler                 | Compilation                              |
| ------- | ------------------------ | ---------------------------------------- |
| 生命周期    | 全局                       | 单次构建                                     |
| 关注点     | 流程                       | 产物                                       |
| 是否复用    | 是                        | 否                                        |
| 常用 hook | thisCompilation、run、done | processAssets、buildModule、optimizeChunks |

### 1. 初始化阶段 (Initialization)

这是 Webpack 启动的准备工作。

- **读取参数**：从你的 `webpack.config.js` 文件以及 CLI 命令行中读取并合并配置参数。
    
- **实例化 Compiler**：用上一步得到的参数初始化 `Compiler` 对象（这是 Webpack 的主引擎，掌控全局生命周期）。
    
- **加载插件**：执行配置文件中 `plugins` 里的 `apply` 方法，让插件监听 Webpack 广播出来的各种事件节点（Hooks）。

### 2. 编译构建阶段 (Compilation)

这是最耗时、也是最核心的阶段，Webpack 在这里完成代码的转换。

- **确定入口 (Entry)**：根据配置找到所有的入口文件。
    
- **编译模块 (Make)**：从入口文件出发，调用所有配置的 **Loaders** 对模块进行转换。
    
- **递归处理**：解析模块内部的依赖关系（`import` 或 `require`），递归地进行编译，直到所有依赖文件都经过了处理。
    
- **生成依赖图**：最终形成一个“模块依赖图”，记录了所有模块及其转换后的代码。
### 3. 输出阶段 (Output)

将内存中的转换结果转化为物理文件。

- **封装 (Seal)**：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 **Chunk**（代码块）。
    
- **优化**：在这个节点，Webpack 会进行 Tree Shaking、代码压缩、分包策略（SplitChunks）等操作。
    
- **输出 (Emit)**：把生成的 Assets（资源）写入到文件系统中，也就是你配置的 `dist` 目录。