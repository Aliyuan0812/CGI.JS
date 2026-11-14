# **CGI.JS**

一个让你不仅能在本地直接运行，而且能通过 FCGI 接口作为服务器后端的项目。

[![Stars](https://img.shields.io/github/stars/Ah-Yuanah/CGI.JS)](https://github.com/Ah-Yuanah/CGI.JS)
[![License](https://img.shields.io/badge/license-MIT-blue)](./LICENSE)
[![Platform](https://img.shields.io/badge/platform-Windows-blue)](https://github.com/Ah-Yuanah/CGI.JS)


> ⭐ 喜欢这个项目欢迎 Star \& Watch，感谢支持！

### **简介**

CGI.JS(CJS)将 FastCGI 与 QuickJS-NG 结合，提供现代化的 JS 执行环境。

### **核心特性**

* 支持 `.cjs` 脚本直接执行
* 支持作为后端接收网络请求处理
* 支持模块化环境
* 支持 JS/C 扩展

### **快速上手**

##### **环境要求**

* 操作系统：Windows 7 及以上版本
* 开发工具：Visual Studio 2022+ 或支持 C/C++ 最新标准与 MSVC 扩展的编译器

##### **安装使用**

1. 使用 Visual Studio 打开 `.sln` 文件
2. 编译生成可执行文件
3. 直接运行 `.cjs` 脚本或作为 FCGI 后端启动

```bash
cjs.exe script.cjs
```

##### **相关文档**

* [API参考文档](./docs/api-reference.md)
* [CAPI头文件](./include/cjsapi.h)

##### **贡献指南**

我们欢迎各种形式的贡献，包括但不限于：

* 提交 bug 报告与修复方案
* 提出新功能建议
* 提交代码 PR
* 完善项目文档/代码等

### **许可证**

本项目采用 MIT 许可证开源，详情参见 [LICENSE](./LICENSE) 文件。

### **更新日志**

* 发布版本v4.3.20261114.01: 加入 `script.export` `script.import`，修改 `navigator.dll` 为 `navigator.ffi`，升级了第三方依赖，修复了部分场景下fcgi模式异常报错。
* 发布版本v4.2.20261108.01: 加入 `navigator.dll.struct`。健壮 FFI层 托管堆内存校验。修复报错语句。
* 发布版本v4.1.20261108.02: 修复 `CJS_SetPrototype` 内存泄露问题。
* 发布版本v4.1.20261108.01: 加入 `--library` 启动参数，修改win32下的平台为 `win-x86`。
* 发布版本v4.0.20261107.01: 升级内核至v0.18.0。重磅加入 FFI 层。新增一系列 `filesystem` 模块接口。全量修复宿主接口函数无法脱离上下文使用的问题。修复 `xhr.send` 函数接收 **FormData** 作为参数时失败的问题。修复 **FormData** 相关历史性问题。修复 `window.setTimeout` 的多线程问题。修复在 `script.execute` 返回新上下文后立即释放崩溃问题。修复 `filesystem.open->close` 函数的查表遗漏解锁问题。修改文件执行模式使其在执行期间不锁定脚本文件。修复 `network.http.open` 状态异常问题。修复一系列问题。
* 发布版本v3.7.20261101.01: 新增 `script.compile`、`script.run`、`script.validate` 用于 编译/执行/验证 字节码，全面加强多线程安全，修复 `system.exit()` 在 `try-catch` 中失效的问题，修改 Ctrl + C 为终止当前操作，Ctrl + Break 为退出程序，优化众多性能问题，加入一堆启动参数。有关新特性的使用，请参阅 [API参考文档](./docs/api-reference.md)。
* 发布版本v3.6.20261029.01: 修复 `system.execute` 的多线程安全问题，新增 `filesystem.input`、`filesystem.scan`、`filesystem.info`，修复 `filesystem.list` 列出占有锁文件的问题。
* 发布版本v3.5.20261026.01: 修复 `window.crypto` 和 `window.bytebuffer` 的多线程安全问题，以及修复一系列多线程安全问题。 
* 发布版本v3.4.20261025.01: 修复 `network.http.open` 和 `window.setTimeout` 的多线程安全问题。 
* 发布版本v3.3.20261024.01: 修复 `window.include` 在非模块化环境下不可用的问题，优化 `network.http.open` 使其更加可靠规范，优化十余并发安全问题。
* 发布版本v3.2.20261017.01: 对 `filesystem.open`、`script.execute`、`network.http.open` 的返回值加入ES显式资源管理的 `using` 支持，修复参考文档的部分注明错误，升级 `console.log` 的输出性能、格式，修改 `filesystem.count`、`filesystem.remove` 的返回值由 **BigInt** 至 **Number**，修复一系列已知Bugs。
* 发布版本v3.1.20261005.01: 升级内核至0.17.0版本。
* 发布版本v3.0.20261004.01: 升级内核至0.16.2版本，重磅支持ES2026 Stage4特性，修复一系列历史性问题，全量重写帮助文档。



