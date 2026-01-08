```js
// BuildSizeReportPlugin.js
class AssetsSizeReportPlugin {
  constructor(options = {}) {
    this.maxSize = options.maxSize || 200 * 1024 // 默认 200KB
    this.filename = options.filename || 'build-size-report.json'
  }

  apply(compiler) {
    compiler.hooks.thisCompilation.tap(
      'AssetsSizeReportPlugin',
      (compilation) => {
        compilation.hooks.processAssets.tap(
          {
            name: 'AssetsSizeReportPlugin',
            stage: compiler.webpack.Compilation.PROCESS_ASSETS_STAGE_SUMMARIZE,
          },
          (assets) => {
            const report = {}
            for (const chunk of compilation.chunks) {
              let chunkSize = 0
              const files = []

              for (const file of chunk.files) {
                const asset = assets[file]
                if (!asset) continue

                const size = asset.size()
                chunkSize += size
                files.push({
                  file,
                  size,
                })
              }

              report[chunk.name || chunk.id] = {
                size: chunkSize,
                files,
                exceed: chunkSize > this.maxSize,
              }

              if (chunkSize > this.maxSize) {
                compilation.warnings.push(
                  new Error(
                    `Chunk "${chunk.name}" size ${Math.round(
                      chunkSize / 1024
                    )}KB exceeds limit ${Math.round(
                      this.maxSize / 1024
                    )}KB`
                  )
                )
              }
            }

            const json = JSON.stringify(report, null, 2)

            compilation.emitAsset(
              this.filename,
              new compiler.webpack.sources.RawSource(json)
            )
          }
        )
      }
    )
  }
}

module.exports = AssetsSizeReportPlugin
```

为什么用 `thisCompilation`
每次构建（包括 watch / HMR）都会生成一个新的 Compilation。这是**插件介入构建过程的正确入口**。
使用了 `thisCompilation` 而不是 `compilation`（更准确）

为什么用 `processAssets`
是 **Webpack 5 新增的核心钩子**，用于在**生成最终资源（assets）后、写入磁盘前**对所有输出文件进行统一处理。
在PROCESS_ASSETS_STAGE_SUMMARIZE阶段，所有 loader 已完成，所有 chunk 已生成，asset 内容稳定

`chunk.files` 是什么？
一个 chunk 最终会生成哪些物理文件（JS / CSS）