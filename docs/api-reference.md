# CGI.JS-Windows API 参考文档
> 本文档仅对解释器宿主扩展 API 进行详细说明；其余原生语言接口均遵循 ECMAScript 规范。
>
> 解释器版本: v3.0.20261004.01。接口行为会受运行环境影响，实际表现请以本地测试结果为准。
>
> 文档最后更新: 2026 年 10 月 04 日。

## 说明

> 本解释器既可以使用浏览器风格
> ```js
> window
> ```
> 访问全局对象，也可以使用Node.js风格
> ```js
> global
> ```
> 或者是原生语言支持
> ```js
> globalThis
> ```
> 它们最终都指向同一个Global全局对象，在本文档中将以 `window` 作为统一指代。
>
> 本解释器具有三种运行模式: 
>
> 1.文件(file)模式，通常是通过命令行启动解释器并且执行指定.cjs文件，例如
> ```bash
> cjs.exe script.cjs 
> ```
> 2.交互式(repl)模式，通常是直接运行解释器并且在shell中执行代码，例如
> ```bash
> cjs.exe
> ```
> 3.FCGI模式(fcgi)，通常是由浏览器发起请求经过服务器的FCGI模块将请求转发给本解释器。
>
> 值得注意的是，在FCGI模式下在 `window.network.request` 中会有相关的网络请求数据以及 `window.network.response` 变得可用，这将在下文详细说明。
>
> 本解释器具有模块化设计，部分功能需要使用 `window.include` 函数引入模块后才可使用，这将在下文详细说明。
>
> 本解释器的宿主API具有强入参校验，传入参数类型或数量异常可能会抛出异常。
>

---

# window 对象

  ## include 函数
  在开启模块化功能下用于导入指定模块/扩展；在未开启模块化功能下此函数将被自动调用。

  ### 语法
  ```js
  include(...moduleName: String) : Number
  ```

  ### 参数
  `...moduleName`

  类型: **String**
  
  需要导入的模块或者扩展的名称，支持一次导入多个模块/扩展。内置模块名格式为: `cjs:${模块自身名称}`，扩展名格式为: `library:${扩展相较于Library目录的相对路径}`(C扩展)，`extension:${扩展相较于Library目录的相对路径}`(JS扩展)。

  ### 返回值
  类型: **Number**

  如果函数成功，则返回值为成功导入的模块/扩展数量。
  
  如果函数失败，则返回值为 **0**。

  此函数通常因以下原因之一而失败: 

  &bull; 传入参数的类型错误。

  &bull; 指定模块/扩展不存在。

  ### 备注
  **include** 函数在导入模块时从可执行文件内部查找，导入扩展时将从`可执行文件所在目录\Library\*.dll`查找C扩展和`可执行文件所在目录\Extension\*.js`查找JS扩展。注意，未开启模块化功能时，**include** 函数会优先加载内置模块，其次是C扩展，最后是JS扩展。有关支持的内置模块以及函数所属模块，请参阅本文档的任意模块的要求部分。

  ### 例
  以下示例代码演示如何使用 **include**。
  ```js
  include('cjs:bytebuffer', 'cjs:filesystem', 'library:./myCExt.dll', 'extension:./myJSExt.js');
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 全局可用 |
  | 位置 | window.include |

  ---

  ## use 函数
  将指定对象的所有属性复写到指定位置。

  ### 语法
  ```js
  use(namespace: Object[, where: Object]) : Boolean
  ```

  ### 参数
  `namespace`

  类型: **Object**

  被复写属性的来源对象。

  `[where]`

  类型: **Object**

  需要被复写的目标对象，默认值为 **window**。

  ### 返回值
  类型: **Boolean**

  如果函数成功，则返回值为 **true**。

  如果函数失败，则返回值为 **false**。

  此函数通常因以下原因之一而失败: 

  &bull; 传入参数的类型错误。

  &bull; 来源对象不存在可历遍的属性。

  ### 备注
  **use** 函数会遍历来源对象的自身的所有属性(不包括原型链上的属性)并且尝试将其设置到目标对象上相同键的位置。

  ### 例
  以下示例代码演示如何使用 **use**。
  ```js
  console.log(Math.E);                   //输出 2.7182818284590451
  console.log(Object.E);                 //输出 undefined
  console.log(use(Math, Object));      //输出 true
  console.log(Object.E);                 //输出 2.7182818284590451
  console.log(Object.min === Math.min);  //输出 true
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 全局可用 |
  | 位置 | window.use |

  ---

  ## wait 函数
  阻塞等待指定的时长。

  ### 语法
  ```js
  wait(milliseconds: Number) : void
  ```

  ### 参数
  `milliseconds`

  类型: **Number**

  需要等待的时长，单位为毫秒，只能为正整数或 **0**。

  ### 返回值
  无

  此函数通常因以下原因之一而失败: 

  &bull; 传入参数的类型错误。

  &bull; 来源对象不存在可历遍的属性。

  ### 备注
  **wait** 函数会阻塞程序并且等待指定的时长后继续执行代码，受到系统时钟精度的影响，实际等待时长可能存在10毫秒左右的波动。

  ### 例
  以下示例代码演示如何使用 **wait**。
  ```js
  console.log(Date.now());  //0000
  wait(1000);
  console.log(Date.now());  //1000
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 全局可用 |
  | 位置 | window.wait |

  ---

  ## this_close 函数
  关闭当前上下文。

  ### 语法
  ```js
  this_close() : void
  ```

  ### 参数
  无

  ### 返回值
  无

  此函数通常因以下原因之一而失败: 

  &bull; 当前上下文无效。

  ### 备注
  **this_close** 函数会删除当前上下文的所有数据并且与夫上下文解除关联。

  ### 例
  以下示例代码演示如何使用 **this_close**。
  ```js
  const ctx = script.execute();
  ctx.this_close();
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 仅在子上下文可用 |
  | 位置 | window.this_close |

  ---

  ## URL 类
  解析URL。

  ### 语法
  ```js
  new this_close(url: String) : URL
  ```

  ### 构造器参数
  `url`

  类型: **String**

  需要解析的URL字符串。

  ### 实例
  ```js
  URL {
      href: String,
      protocol: "https:",
      username: "",
      password: "",
      host: "aliyuan.cpolar.cn",
      hostname: "aliyuan.cpolar.cn",
      port: "443",
      pathname: "/",
      search: "",
      hash: ""
  }
  ```
  `href`

  类型: **String**

  完整URL。

  `protocol`

  类型: **String**

  协议。

  `username`

  类型: **String**

  username字段。

  `password`

  类型: **String**

  password字段。

  `host`

  类型: **String**

  主机。

  `hostname`

  类型: **String**

  主机名。
  
  `port`

  类型: **String**

  端口。

  `pathname`

  类型: **String**

  路径名。

  `search`

  类型: **String**

  查询字符串。

  `hash`

  类型: **String**

  哈希。


  此类的构造器通常因以下原因之一而失败: 

  &bull; 传入参数的类型错误。

  &bull; 传入的URL解析失败。

  ### 例
  以下示例代码演示如何使用 **URL**。
  ```js
  const url = new URL('https://example.com');
  console.log(url);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | URL |
  | 可用性 | 全局可用 |
  | 位置 | window.URL |

  ---

# system 对象

  ## help 函数
  输出帮助消息。

  ### 语法
  ```js
  help() : void
  ```

  ### 参数
  无

  ### 返回值
  无

  ### 例
  以下示例代码演示如何使用 **help**。
  ```js
  system.help();
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 全局可用 |
  | 位置 | system.help |

  ---
  
  ## exit 函数
  结束当前上下文。

  ### 语法
  ```js
  exit() : void
  ```

  ### 参数
  无

  ### 返回值
  无

  此函数通常因以下原因之一而失败: 

  &bull; 当前上下文无效。

  ### 备注
  **exit** 函数会删除当前上下文的所有数据并且释放所有资源，且后续的代码将不再执行。在REPL模式的顶层上下文调用此函数将同时退出程序。

  ### 例
  以下示例代码演示如何使用 **exit**。
  ```js
  system.exit();
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 全局可用 |
  | 位置 | system.exit |

  ---
  
  ## updateConfig 函数
  更新并应用程序配置。

  ### 语法
  ```js
  updateConfig(config: Object) : void
  ```

  ### 参数
  `config`

  类型: **Object**

  待更新的程序配置对象，先前的配置通常位于 `system.config`。

  ### 返回值
  无

  ### 备注
  **updateConfig** 函数会将程序配置对象更新到整个程序，使其在运行期间以新的配置执行，但不会在下一次启动时生效(除非调用了 **saveConfig** 函数)，且不会同步更新 `system.config` 中的配置。有关如何永久保存程序配置，请参阅 [system.saveConfig](#saveconfig-函数)。

  ### 例
  以下示例代码演示如何使用 **updateConfig**。
  ```js
  system.config.myConfig = 'myValue';
  system.updateConfig(system.config);
  system.saveConfig();
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | system |
  | 可用性 | 全局可用 |
  | 位置 | system.updateConfig |

  ---
  
  ## saveConfig 函数
  保存应用程序配置。

  ### 语法
  ```js
  saveConfig() : Boolean
  ```

  ### 参数
  无

  ### 返回值
  类型: **Boolean**

  如果函数成功，则返回值为 **true**。

  如果函数失败，则返回值为 **false**。
  
  此函数通常因以下原因之一而失败: 

  &bull; 无权限写入配置文件。

  ### 备注
  **saveConfig** 函数会将程序配置写入到配置文件，使其在下一次启动时仍然能够生效。如果需要更新配置，请结合 **updateConfig** 函数实现。有关如何更新程序配置，请参阅 [system.updateConfig](#updateconfig-函数)。有关程序配置对象，请参阅 [system.config](#config-属性)。

  ### 例
  以下示例代码演示如何使用 **saveConfig**。
  ```js
  system.config.myConfig = 'myValue';
  system.updateConfig(system.config);
  system.saveConfig();
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 全局可用 |
  | 位置 | system.saveConfig |

  ---
  
  ## execute 函数
  执行一段命令。

  ### 语法
  ```js
  execute(cmd: String) : Promise<Object>
  ```

  ### 参数
  `cmd`

  类型: **String**

  需要执行的命令。

  ### 返回值
  类型: **Promise&lt;Object&gt;**

  返回值的决议值。
  ```js
  {
      isSuccess: Boolean,
      exitCode: Number,
      output: String
  }
  ```
  `isSuccess`

  类型: **Boolean**

  命令是否执行成功。

  `exitCode`

  类型: **Number**

  命令行程序的退出码。
  
  `output`

  类型: **String**

  命令行程序的完整输出内容。

  此函数通常因以下原因之一而失败: 

  &bull; 传入参数的类型错误。

  &bull; 无权限调用命令行程序。

  &bull; 找不到命令行程序。

  ### 备注
  **execute** 函数会在本地电脑查找默认的命令行程序(例如在Windows中默认为命令提示符)并执行传入的命令，且会保持Promise挂起直至命令行程序退出。

  ### 例
  以下示例代码演示如何使用 **execute**。
  ```js
  const command = 'echo hello';
  console.log(await system.execute(command));
  /* 输出: 
  {
      isSuccess: true,
      exitCode: 0,
      output: "hello
  "
  }
  */
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | system |
  | 可用性 | 全局可用 |
  | 位置 | system.execute |

  ---

  ## cmd 函数
  获取当前进程的命令行。

  ### 语法
  ```js
  cmd() : String
  ```

  ### 参数
  无

  ### 返回值
  类型: **String**

  当前进程的完整命令行。

  ### 备注
  **cmd** 函数会返回启动当前进程时使用的完整命令行字符串，包含可执行文件路径以及所有启动参数。

  ### 例
  以下示例代码演示如何使用 **cmd**。
  ```js
  console.log(system.cmd());  //输出 ""C:\Program Files\MyApp\app.exe" --flag"
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | system |
  | 可用性 | 全局可用 |
  | 位置 | system.cmd |

  ---

  ## cwd 函数
  获取当前工作目录。

  ### 语法
  ```js
  cwd() : String
  ```

  ### 参数
  无

  ### 返回值
  类型: **String**

  当前上下文的当前工作目录的格式化路径。

  ### 备注
  **cwd** 函数会返回当前上下文的工作目录，并使用格式化后的路径表示，其中路径分隔统一用正斜杠，末尾统一带正斜杠。

  ### 例
  以下示例代码演示如何使用 **cwd**。
  ```js
  console.log(system.cwd());  //输出 "C:/MyProgram/"
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | system |
  | 可用性 | 全局可用 |
  | 位置 | system.cwd |

  ---

  ## ecwd 函数
  获取可执行文件所在目录。

  ### 语法
  ```js
  ecwd() : String
  ```

  ### 参数
  无

  ### 返回值
  类型: **String**

  可执行文件所在目录的格式化路径。

  ### 备注
  **ecwd** 函数会返回当前可执行文件所在的目录，并使用格式化后的路径表示，其中路径分隔统一用正斜杠，末尾统一带正斜杠。

  ### 例
  以下示例代码演示如何使用 **ecwd**。
  ```js
  console.log(system.ecwd());  //输出 "C:/MyProgram/"
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | system |
  | 可用性 | 全局可用 |
  | 位置 | system.ecwd |

  ---

  ## platform 属性
  当前运行的平台。

  ### 语法
  ```js
  platform: String
  ```

  ### 格式
  `${system}-${architecture}`

  `system`

  当前平台系统简写，例如Windows简写为win。

  `architecture`

  当前平台架构，例如x64、x86。

  ### 例
  以下示例代码演示如何获取 **platform**。
  ```js
  console.log(system.platform);  //输出 "win-x64"
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 全局可用 |
  | 位置 | system.platform |

  ---

  ## path 属性
  执行脚本的位置。

  ### 语法
  ```js
  path: String
  ```

  ### 格式
  在交互式模式中，顶层上下文固定为 **"typein"**，空子上下文固定为 **"buildIn"**，正常子上下文为 `${文件名称(带扩展名)}`。
  在文件模式中，顶层上下文为 `${当前执行文件的完整路径}`，空子上下文固定为 **"buildIn"**，正常子上下文为 `${文件名称(带扩展名)}`。
  在FCGI模式中，为 `${文件名称(带扩展名)}`。

  ### 例
  以下示例代码演示如何获取 **path**。
  ```js
  console.log(system.path);  //输出 "typein"
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 全局可用 |
  | 位置 | system.path |

  ---

  ## scriptPath 属性
  执行脚本所在的位置。

  ### 语法
  ```js
  scriptPath: String
  ```

  ### 格式
  在交互式模式中，固定为 **""**。
  在文件模式中，为完整的被执行脚本所在的路径。
  在FCGI模式中，固定为 **""**。

  ### 例
  以下示例代码演示如何获取 **scriptPath**。
  ```js
  console.log(system.scriptPath);  //输出 "C:\script.cjs"
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 仅在顶层上下文可用 |
  | 位置 | system.scriptPath |

  ---

  ## runMode 属性
  程序运行模式。

  ### 语法
  ```js
  runMode: String
  ```

  ### 格式
  在交互式模式中，固定为 **"repl"**。
  在文件模式中，固定为 **"file"**。
  在FCGI模式中，固定为 **"fcgi"**。

  ### 例
  以下示例代码演示如何获取 **scriptPath**。
  ```js
  console.log(system.scriptPath);  //输出 "C:\script.cjs"
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | system |
  | 可用性 | 全局可用 |
  | 位置 | system.runMode |

  ---

  ## cmdLine 属性
  当前进程的命令行。

  ### 语法
  ```js
  cmdLine: String
  ```

  ### 格式
  等价于 `system.cmd()` 的返回值。有关 **cmd** 函数的说明，请参阅 [system.cmd](#cmd-函数)

  ### 例
  以下示例代码演示如何获取 **cmdLine**。
  ```js
  console.log(system.cmdLine);  //输出 ""C:\Program Files\MyApp\app.exe" --flag"
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 全局可用 |
  | 位置 | system.cmdLine |

  ---

  ## cmdLineArgs 属性
  当前进程的命令行参数。

  ### 语法
  ```js
  cmdLineArgs: Object
  ```

  ### 格式
  ```js
  {
      String: String,
      String: String,
      ...
  }
  ```
  其中每一项的键为参数名，值为参数值，不包含前缀横杠，也不区分单横杠或双横杠。

  ### 例
  以下示例代码演示如何获取 **cmdLineArgs**。
  ```js
  console.log(system.cmdLineArgs);
  /* 输出: 
  {
      "flag": "",
      "path": "C:/MyProgram"
  }
  */
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 全局可用 |
  | 位置 | system.cmdLineArgs |

  ---
  
  ## isOnLine 属性
  程序运行模式。

  ### 语法
  ```js
  isOnLine: Boolean
  ```

  ### 格式
  在本地电脑已连接到互联网时，固定为 **true**。
  在本地电脑未连接到互联网时，固定为 **false**。

  ### 例
  以下示例代码演示如何获取 **isOnLine**。
  ```js
  console.log(system.isOnLine);  //输出 true
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | system |
  | 可用性 | 全局可用 |
  | 位置 | system.isOnLine |

  ---

  ## version 属性
  解释器版本。

  ### 语法
  ```js
  version: Boolean
  ```

  ### 格式
  固定为 `${majorVersion}.${subVersion}.{YYYYMMDD}.{releaseNumber}`

  ### 例
  以下示例代码演示如何获取 **version**。
  ```js
  console.log(system.version);  //输出 "3.0.20261004.01"
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 全局可用 |
  | 位置 | system.isOnLine |

  ---

  ## config 属性
  程序配置对象。

  ### 语法
  ```js
  config: ConfigSystem
  ```

  ### 格式
  ```js
  {
    fastcgi: {
      isFlushNamedPipe: Boolean,
      isModernMode: Boolean,
      isOutputError: Boolean,
      isStrictStandard: Boolean,
      timeout: Number
    },
    shell: {
      isAlwaysPauseWhenQuit: Boolean,
      isShowReturnDetail: Boolean,
      isShowReturnValue: Boolean
    },
    file: {
      isShowConsole: Boolean,
      isTotalOutput: Boolean
    },
    general: {
      isModuleMode: Boolean
    }
  }
  ```
  `fastcgi`
  >
  针对FCGI模式的配置。
  >
  > `isFlushNamedPipe`
  >
  > 类型: **Boolean**
  >
  > 是否在请求响应前强制刷新管道。
  >
  > `isModernMode`
  >
  > 类型: **Boolean**
  >
  > 是否让请求限制更现代，如果开启，则TRACE请求将不被允许带有响应体。
  >
  > `isOutputError`
  >
  > 类型: **Boolean**
  >
  > 是否在存在未捕获的错误时显示详细的错误页面，如果关闭，则只会响应500。
  >
  > `isStrictStandard`
  >
  > 类型: **Boolean**
  >
  > 是否严格遵循HTTP规范，如果开启，则例如不容忍GET请求带请求体等，以及相关响应码将会根据请求类型进行符合标准的限制。
  >
  > `timeout`
  >
  > 类型: **Number**
  >
  > 请求的超时时长，单位为秒。

  `shell`
  >
  针对交互式模式的配置。
  >
  > `isAlwaysPauseWhenQuit`
  >
  > 类型: **Boolean**
  >
  > 是否让解释器在出现致命异常时暂停而不直接退出，这有助于发现异常原因。
  >
  > `isShowReturnDetail`
  >
  > 类型: **Boolean**
  >
  > 是否让解释器输出更现代更详细更哇塞美观的执行结果，如果关闭，非普通类型的代码执行结果将会调用 `toString()` 输出。
  >
  > `isShowReturnValue`
  >
  > 类型: **Boolean**
  >
  > 是否输出代码执行结果，如果关闭，`isShowReturnDetail` 将不可用。
  
  `file`
  >
  针对文件模式的配置。
  >
  > `isShowConsole`
  >
  > 类型: **Boolean**
  >
  > 是否在脚本执行时显示控制台。
  >
  > `isTotalOutput`
  >
  > 类型: **Boolean**
  >
  > 是否让解释器在执行完脚本后输出总结性语句。
    
  `general`
  >
  针对文件模式的配置。
  >
  > `isModuleMode`
  >
  > 类型: **Boolean**
  >
  > 是否开启模块化功能。


  ### 例
  以下示例代码演示如何获取 **config**。
  ```js
  console.log(system.config);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | system |
  | 可用性 | 全局可用 |
  | 位置 | system.config |

  ---

# filesystem 对象

  ## list 函数
  列出指定目录中的条目。

  ### 语法
  ```js
  filesystem.list(path: String) : Object
  ```

  ### 参数
  `path`

  类型: **String**

  需要列出的目录路径。相对路径基于当前上下文的工作目录解析。

  ### 返回值
  类型: **Object**

  如果函数成功，则返回一个对象，其键为目录内条目的路径，值为条目名称。

  此函数通常因以下原因之一而抛出异常:

  &bull; 传入参数的数量不为 **1**。

  &bull; `path` 不是字符串。

  &bull; 指定路径不是目录。

  ### 备注
  **list** 函数会读取指定目录下的条目，并构造一个普通对象返回。返回对象的属性名为条目路径，属性值为条目名称。目录为空时返回空对象。

  ### 例
  以下示例代码演示如何使用 **list**。
  ```js
  include('cjs:filesystem');

  const entries = filesystem.list('./');
  for (const path in entries) {
      console.log(path, entries[path]);
  }
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | filesystem |
  | 可用性 | 全局可用 |
  | 位置 | filesystem.list |

  ---

  ## count 函数
  统计指定目录中的条目数量。

  ### 语法
  ```js
  filesystem.count(path: String) : BigInt
  ```

  ### 参数
  `path`

  类型: **String**

  需要统计的目录路径。相对路径基于当前上下文的工作目录解析。

  ### 返回值
  类型: **BigInt**

  如果函数成功，则返回指定目录中的条目数量。

  此函数通常因以下原因之一而抛出异常:

  &bull; 传入参数的数量不为 **1**。

  &bull; `path` 不是字符串。

  &bull; 指定路径不是目录。

  ### 备注
  **count** 函数仅统计目录中的条目数量，不会递归统计子目录内部的内容。

  ### 例
  以下示例代码演示如何使用 **count**。
  ```js
  include('cjs:filesystem');

  console.log(filesystem.count('./'));
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | filesystem |
  | 可用性 | 全局可用 |
  | 位置 | filesystem.count |

  ---

  ## remove 函数
  删除指定路径。

  ### 语法
  ```js
  filesystem.remove(path: String) : BigInt
  ```

  ### 参数
  `path`

  类型: **String**

  需要删除的文件或目录路径。相对路径基于当前上下文的工作目录解析。

  ### 返回值
  类型: **BigInt**

  如果函数成功，则返回成功删除的条目数量。如果指定路径不存在，通常返回 **0**。

  此函数通常因以下原因之一而抛出异常:

  &bull; 传入参数的数量不为 **1**。

  &bull; `path` 不是字符串。

  ### 备注
  **remove** 函数会尝试删除指定路径。若指定路径为目录，是否递归删除子条目由底层 `FileController` 实现决定。参数类型错误时会抛出异常。

  ### 例
  以下示例代码演示如何使用 **remove**。
  ```js
  include('cjs:filesystem');

  console.log(filesystem.remove('./demo.txt'));
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | filesystem |
  | 可用性 | 全局可用 |
  | 位置 | filesystem.remove |

  ---

  ## exists 函数
  判断指定路径是否存在。

  ### 语法
  ```js
  filesystem.exists(path: String) : Boolean
  ```

  ### 参数
  `path`

  类型: **String**

  需要判断的路径。相对路径基于当前上下文的工作目录解析。

  ### 返回值
  类型: **Boolean**

  如果路径存在，则返回 **true**。

  如果路径不存在，则返回 **false**。

  此函数通常因以下原因之一而抛出异常:

  &bull; 传入参数的数量不为 **1**。

  &bull; `path` 不是字符串。

  ### 备注
  **exists** 函数不会区分文件与目录，只要指定路径存在即返回 **true**。

  ### 例
  以下示例代码演示如何使用 **exists**。
  ```js
  include('cjs:filesystem');

  console.log(filesystem.exists('./demo.txt'));
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | filesystem |
  | 可用性 | 全局可用 |
  | 位置 | filesystem.exists |

  ---

  ## open 函数
  打开文件并返回文件控制器对象。

  ### 语法
  ```js
  filesystem.open(path: String[, mode: String]) : Object
  ```

  ### 参数
  `path`

  类型: **String**

  需要打开的文件路径。相对路径基于当前上下文的工作目录解析。该参数不能为空字符串。

  `[mode]`

  类型: **String**

  可选。文件打开模式，默认值为 `"r"`。支持读、写、读写、追加以及二进制模式。常见模式包括 `"r"`、`"w"`、`"a"`、`"r+"`、`"w+"`、`"a+"`、`"rb"`、`"wb"`、`"ab"`、`"rb+"`、`"wb+"`、`"ab+"` 等。若模式无效，则抛出异常。

  ### 返回值
  类型: **Object**

  如果函数成功，则返回一个文件控制器对象。返回值的结构如下：
  ```js
  {
      id: String,
      mode: String,
      name: String,
      closed: Boolean,
      seekPtr: Number,
      buffer: Uint8Array,
      read: Function,
      write: Function,
      close: Function,
      tell: Function,
      size: Function,
      seek: Function
  }
  ```
  `id`

  类型: **String**

  文件控制器的内部标识。

  `mode`

  类型: **String**

  当前文件的打开模式。

  `name`

  类型: **String**

  当前文件的路径。

  `closed`

  类型: **Boolean**

  文件是否已关闭。

  `seekPtr`

  类型: **Number**

  当前读写位置。

  `buffer`

  类型: **Uint8Array**

  最近一次调用 **read** 后读取到的原始字节数据。

  `read`

  类型: **Function**

  从当前读写位置读取文件内容。

  语法：
  ```js
  read([size: Number]) : String | Uint8Array
  ```

  参数：
  `[size]`

  类型: **Number**

  可选。需要读取的字节数。省略或传入 **-1** 时表示读取到文件末尾。只能为正整数、**0** 或 **-1**。

  返回值：
  类型: **String** 或 **Uint8Array**

  文本模式下返回字符串，二进制模式下返回 **Uint8Array**。读取后，文件对象的 `seekPtr` 会向后移动，`buffer` 属性会被设置为本次读取到的原始字节数据。

  备注：
  该方法只能用于以读模式或读写模式打开的文件对象。

  `write`

  类型: **Function**

  向文件写入数据。

  语法：
  ```js
  write(data: String | Uint8Array) : Number
  ```

  参数：
  `data`

  类型: **String** 或 **Uint8Array**

  需要写入的数据。文本模式下必须为字符串，二进制模式下必须为 **Uint8Array**。

  返回值：
  类型: **Number**

  返回实际写入的字节数。

  备注：
  非追加模式下从当前 `seekPtr` 位置开始写入；追加模式下始终写入文件末尾，并忽略当前 `seekPtr`。写入后 `seekPtr` 会更新到新的读写位置。

  `close`

  类型: **Function**

  关闭文件并释放资源。

  语法：
  ```js
  close() : void
  ```

  参数：
  无

  返回值：
  无

  备注：
  关闭后文件对象失效。重复调用 **close** 会抛出异常。

  `tell`

  类型: **Function**

  返回当前读写位置。

  语法：
  ```js
  tell() : Number
  ```

  参数：
  无

  返回值：
  类型: **Number**

  返回当前文件对象的 `seekPtr` 值。

  备注：
  该方法不会改变文件读写位置。

  `size`

  类型: **Function**

  返回文件大小。

  语法：
  ```js
  size() : Number
  ```

  参数：
  无

  返回值：
  类型: **Number**

  返回当前文件的总字节数。

  备注：
  该方法不会改变文件读写位置。

  `seek`

  类型: **Function**

  移动文件读写位置。

  语法：
  ```js
  seek(offset: Number[, whence: Number]) : void
  ```

  参数：
  `offset`

  类型: **Number**

  偏移量，必须为整数。

  `[whence]`

  类型: **Number**

  可选。参考位置，默认值为 **0**。支持以下值：

  &bull; **0**：从文件开头计算。

  &bull; **1**：从当前位置计算。

  &bull; **2**：从文件末尾计算。

  返回值：
  无

  备注：
  文本模式下不支持从当前位置计算偏移，也不支持从文件末尾计算非零偏移。二进制模式下支持所有参考位置。移动后的位置不能为负数。调用成功后，文件对象的 `seekPtr` 会被更新。

  此函数通常因以下原因之一而抛出异常:

  &bull; 传入参数的数量不为 **1** 或 **2**。

  &bull; `path` 不是非空字符串。

  &bull; `mode` 不是字符串，或模式无效。

  &bull; 以读模式打开时文件不存在。

  ### 备注
  **open** 函数在写模式且非追加模式下会清空文件内容；在读模式下要求文件必须存在。返回的文件控制器对象中，`read` 方法仅在读模式或读写模式下存在，`write` 方法仅在写模式、读写模式或追加模式下存在，`close`、`tell`、`size`、`seek` 始终存在。

  ### 例
  以下示例代码演示如何使用 **open**。
  ```js
  include('cjs:filesystem');

  let file = filesystem.open('./demo.txt', 'w');
  file.write('Hello, world!');
  file.close();

  file = filesystem.open('./demo.txt', 'r');
  console.log(file.read());
  file.close();
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | filesystem |
  | 可用性 | 全局可用 |
  | 位置 | filesystem.open |

  ---

  # console 对象

  ## restore 函数
  创建或恢复控制台窗口。

  ### 语法
  ```js
  console.restore([title: String]) : void
  ```

  ### 参数
  `[title]`

  类型: **String**

  可选。控制台窗口的标题。省略时使用默认标题。

  ### 返回值
  无

  ### 备注
  **restore** 函数仅在文件模式（`system.runMode === "file"`）且控制台尚未创建时创建控制台窗口，并设置窗口标题。若控制台已存在，则不会重复创建。有关如何关闭控制台，请参阅 [console.kill](#kill-函数)。

  ### 例
  以下示例代码演示如何使用 **restore**。
  ```js
  console.restore('My Console');
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | console |
  | 可用性 | 全局可用 |
  | 位置 | console.restore |

  ---

  ## kill 函数
  关闭控制台窗口。

  ### 语法
  ```js
  console.kill() : void
  ```

  ### 参数
  无

  ### 返回值
  无

  ### 备注
  **kill** 函数仅在文件模式（`system.runMode === "file"`）下关闭控制台窗口。若控制台未创建或非文件模式，则不做任何操作。有关如何恢复被关闭的控制台，请参阅 [console.restore](#restore-函数)。

  ### 例
  以下示例代码演示如何使用 **kill**。
  ```js
  console.kill();
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | console |
  | 可用性 | 全局可用 |
  | 位置 | console.kill |

  ---

  ## hide 函数
  隐藏控制台窗口。

  ### 语法
  ```js
  console.hide() : void
  ```

  ### 参数
  无

  ### 返回值
  无

  ### 备注
  **hide** 函数将控制台窗口设置为工具窗口样式并隐藏，使其不在任务栏中显示。若控制台未创建，则不做任何操作。有关如何恢复控制台显示，请参阅 [console.show](#show-函数)。

  ### 例
  以下示例代码演示如何使用 **hide**。
  ```js
  console.hide();
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | console |
  | 可用性 | 全局可用 |
  | 位置 | console.hide |

  ---

  ## show 函数
  显示控制台窗口。

  ### 语法
  ```js
  console.show() : void
  ```

  ### 参数
  无

  ### 返回值
  无

  ### 备注
  **show** 函数取消工具窗口样式，恢复并显示控制台窗口，同时将其置于前台。若控制台未创建，则不做任何操作。有关如何隐藏控制台，请参阅 [console.hide](#hide-函数)。

  ### 例
  以下示例代码演示如何使用 **show**。
  ```js
  console.show();
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | console |
  | 可用性 | 全局可用 |
  | 位置 | console.show |

  ---

  ## pause 函数
  暂停控制台输出。

  ### 语法
  ```js
  console.pause() : void
  ```

  ### 参数
  无

  ### 返回值
  无

  ### 备注
  **pause** 函数设置暂停标志，后续控制台输出将被暂停，直到调用 **resume** 恢复。有关 **resume** 函数的说明，请参阅 [console.resume](#resume-函数)。

  ### 例
  以下示例代码演示如何使用 **pause**。
  ```js
  console.pause();
  // 此时输出被暂停
  console.resume();
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | console |
  | 可用性 | 全局可用 |
  | 位置 | console.pause |

  ---

  ## resume 函数
  恢复控制台输出。

  ### 语法
  ```js
  console.resume() : void
  ```

  ### 参数
  无

  ### 返回值
  无

  ### 备注
  **resume** 函数清除暂停标志，恢复控制台输出。有关 **pause** 函数的说明，请参阅 [console.pause](#pause-函数)。

  ### 例
  以下示例代码演示如何使用 **resume**。
  ```js
  console.pause();
  // 输出被暂停
  console.resume();
  // 输出恢复
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | console |
  | 可用性 | 全局可用 |
  | 位置 | console.resume |

  ---

  ## log 函数
  输出日志信息到控制台。

  ### 语法
  ```js
  console.log(...data: any) : void
  ```

  ### 参数
  `...data`

  类型: **any**

  需要输出的一个或多个值，支持任意类型。

  ### 返回值
  无

  ### 备注
  **log** 当开启返回详细信息的配置时，将自动调用此函数。函数会将传入的每个值格式化后输出到控制台，值之间以空格分隔，末尾换行。输出支持丰富的类型识别与彩色显示，包括：

  &bull; 字符串、数字、BigInt、布尔、null、undefined、Symbol。

  &bull; 函数（普通函数、内置函数、类、方法）。

  &bull; 对象、数组、TypedArray、ArrayBuffer、Promise、Date、RegExp、Module。

  &bull; 自动处理循环引用，显示为 `[Circular]`。

  &bull; 对 Promise 会显示其状态与结果。

  &bull; 对对象会递归展示自身属性，并过滤内部私有属性。

  有关程序配置对象，请参阅 [system.config](#config-属性)。

  ### 例
  以下示例代码演示如何使用 **log**。
  ```js
  console.log('hello', 123, { a: 1 }, [1, 2, 3]);
  console.log(Promise.resolve('done'));
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 全局可用 |
  | 位置 | console.log |

  ---

  ## clear 函数
  清空控制台输出。

  ### 语法
  ```js
  console.clear() : void
  ```

  ### 参数
  无

  ### 返回值
  无

  ### 备注
  **clear** 函数会清除控制台中的所有输出内容。

  ### 例
  以下示例代码演示如何使用 **clear**。
  ```js
  console.clear();
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 全局可用 |
  | 位置 | console.clear |

  ---

  ## input 函数
  在控制台中提示并获取用户输入。

  ### 语法
  ```js
  console.input([forwarder: String[, defaultValue: String[, isHidden: Boolean]]]) : String
  ```

  ### 参数
  `[forwarder]`

  类型: **String**

  可选。提示信息，会在输入前输出到控制台。

  `[defaultValue]`

  类型: **String**

  可选。输入框中的默认值。

  `[isHidden]`

  类型: **Boolean**

  可选。是否隐藏输入内容（例如密码输入）。默认为 **false**。

  ### 返回值
  类型: **String**

  返回用户输入的字符串。

  此函数通常因以下原因之一而失败:

  &bull; 第一个参数不是字符串。

  &bull; 第二个参数不是字符串。

  &bull; 第三个参数不是布尔值。

  ### 备注
  **input** 函数会在控制台中显示提示信息，等待用户输入，并返回输入的内容。若指定了默认值，输入框会预填该值。若 `isHidden` 为 **true**，输入内容不会显示。

  ### 例
  以下示例代码演示如何使用 **input**。
  ```js
  const name = console.input('请输入姓名：', '匿名', false);
  console.log('你好，' + name);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | console |
  | 可用性 | 全局可用 |
  | 位置 | console.input |

  ---

# script 对象

  ## execute 函数
  创建一个新的上下文并可选执行指定脚本文件。

  ### 语法
  ```js
  script.execute([path: String]) : Object
  ```

  ### 参数
  `[path]`

  类型: **String**

  可选。需要执行的脚本文件路径。相对路径基于当前上下文的工作目录解析。省略时仅创建新上下文，不执行任何文件。

  ### 返回值
  类型: **Object**

  如果函数成功，则返回新上下文的全局对象。该对象具有以下方法：
  ```js
  {
      this_close: Function
  }
  ```
  `this_close`

  类型: **Function**

  有关 **this_close** 的说明，请参阅 [window.this_close](#this_close-函数)。

  此函数通常因以下原因之一而抛出异常:

  &bull; 传入参数的数量超过 **1**。

  &bull; `path` 不是字符串。

  &bull; 指定文件不存在或读取失败。

  &bull; 执行脚本文件时发生错误。

  ### 备注
  **execute** 函数会创建一个新的 JavaScript 上下文，其父上下文为当前上下文。若提供了 `path`，则读取该文件并在新上下文中执行。返回的全局对象可用于在新上下文中求值或调用 **this_close** 关闭上下文。未提供 `path` 时，仅创建新上下文并返回其全局对象。

  ### 例
  以下示例代码演示如何使用 **execute**。
  ```js
  // 创建新上下文并执行脚本文件
  const ctx = script.execute('./demo.js');
  // 关闭该上下文
  ctx.this_close();

  // 仅创建新上下文
  const emptyCtx = script.execute();
  emptyCtx.this_close();
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | script |
  | 可用性 | 全局可用 |
  | 位置 | script.execute |

  ---

  ## include 函数
  在当前上下文中包含并执行一个或多个脚本文件。

  ### 语法
  ```js
  script.include(...path: String) : void
  ```

  ### 参数
  `...path`

  类型: **String**

  需要包含的脚本文件路径，支持一次传入多个文件。相对路径基于当前上下文的工作目录解析。

  ### 返回值
  无

  此函数通常因以下原因之一而抛出异常:

  &bull; 未传入任何参数。

  &bull; 任意一个参数不是字符串。

  &bull; 指定文件不存在或读取失败。

  &bull; 执行脚本文件时发生错误。

  ### 备注
  **include** 函数会在当前上下文中依次读取并执行指定文件中的代码，效果类似于将文件内容直接插入当前上下文执行。与全局 **include** 函数不同，**script.include** 仅执行脚本文件，不涉及模块或扩展的导入。若某个文件不存在或执行出错，将立即抛出异常并终止后续文件执行。

  ### 例
  以下示例代码演示如何使用 **include**。
  ```js
  script.include('./a.js', './b.js');
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | script |
  | 可用性 | 全局可用 |
  | 位置 | script.include |

  ---

  # bytebuffer 对象

  ## readAsJson 函数
  将数据解析为 JSON 对象。

  ### 语法
  ```js
  bytebuffer.readAsJson(data: any) : Promise<Object>
  ```

  ### 参数
  `data`

  类型: **any**

  需要解析的数据，可以是字符串、Uint8Array 或其他可转换为字节缓冲的类型。

  ### 返回值
  类型: **Promise&lt;Object&gt;**

  返回值的决议值为解析后的 JSON 对象。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  &bull; 数据无法解析为有效的 JSON。

  ### 备注
  **readAsJson** 函数会将传入的数据转换为文本，然后使用 JSON 解析器解析为对象。解析过程在后台线程中执行，不会阻塞主线程。若解析失败，Promise 会被拒绝并携带 SyntaxError。

  ### 例
  以下示例代码演示如何使用 **readAsJson**。
  ```js
  include('cjs:bytebuffer');

  const obj = await bytebuffer.readAsJson('{"a":1}');
  console.log(obj.a); // 1
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.readAsJson |

  ---

  ## readAsFormData 函数
  将数据构造为 FormData 对象。

  ### 语法
  ```js
  bytebuffer.readAsFormData(data: any) : Promise<FormData>
  ```

  ### 参数
  `data`

  类型: **any**

  需要构造 FormData 的数据。

  ### 返回值
  类型: **Promise&lt;FormData&gt;**

  返回值的决议值为构造出的 FormData 对象。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  &bull; 构造 FormData 失败。

  ### 备注
  **readAsFormData** 函数会在后台线程中调用全局 FormData 构造函数，并将传入的数据作为参数传入。构造成功后 Promise 决议为 FormData 实例。

  ### 例
  以下示例代码演示如何使用 **readAsFormData**。
  ```js
  include('cjs:bytebuffer');

  const fd = await bytebuffer.readAsFormData('a=1&b=2');
  console.log(fd.get('a')); // "1"
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.readAsFormData |

  ---

  ## readAsString 函数
  将数据转换为字符串。

  ### 语法
  ```js
  bytebuffer.readAsString(data: any) : Promise<String>
  ```

  ### 参数
  `data`

  类型: **any**

  需要转换为字符串的数据。

  ### 返回值
  类型: **Promise&lt;String&gt;**

  返回值的决议值为转换后的字符串。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  ### 备注
  **readAsString** 函数会将传入的数据转换为字节缓冲，再以文本形式读取为字符串。转换在后台线程中执行。

  ### 例
  以下示例代码演示如何使用 **readAsString**。
  ```js
  include('cjs:bytebuffer');

  const str = await bytebuffer.readAsString([72, 101, 108, 108, 111]);
  console.log(str); // "Hello"
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.readAsString |

  ---

  ## decodeBase91 函数
  将 Base91 编码的数据解码为二进制。

  ### 语法
  ```js
  bytebuffer.decodeBase91(data: any[, isUrlEncoding: Boolean]) : Promise<Uint8Array>
  ```

  ### 参数
  `data`

  类型: **any**

  需要解码的 Base91 数据。

  `[isUrlEncoding]`

  类型: **Boolean**

  可选。是否使用 URL 安全字符集。默认为 **false**。

  ### 返回值
  类型: **Promise&lt;Uint8Array&gt;**

  返回值的决议值为解码后的二进制数据。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量超过 **2**。

  &bull; 第二个参数不是布尔值。

  &bull; 解码失败。

  ### 备注
  **decodeBase91** 函数在后台线程中将 Base91 字符串解码为二进制数据。若指定了 URL 安全编码，则使用对应的字符集。

  ### 例
  以下示例代码演示如何使用 **decodeBase91**。
  ```js
  include('cjs:bytebuffer');

  const bin = await bytebuffer.decodeBase91('Hello');
  console.log(bin);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.decodeBase91 |

  ---

  ## decodeBase85 函数
  将 Base85 编码的数据解码为二进制。

  ### 语法
  ```js
  bytebuffer.decodeBase85(data: any) : Promise<Uint8Array>
  ```

  ### 参数
  `data`

  类型: **any**

  需要解码的 Base85 数据。

  ### 返回值
  类型: **Promise&lt;Uint8Array&gt;**

  返回值的决议值为解码后的二进制数据。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  &bull; 解码失败。

  ### 备注
  **decodeBase85** 函数在后台线程中将 Base85 字符串解码为二进制数据。

  ### 例
  以下示例代码演示如何使用 **decodeBase85**。
  ```js
  include('cjs:bytebuffer');

  const bin = await bytebuffer.decodeBase85('Hello');
  console.log(bin);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.decodeBase85 |

  ---

  ## decodeBase64 函数
  将 Base64 编码的数据解码为二进制。

  ### 语法
  ```js
  bytebuffer.decodeBase64(data: any[, isUrlEncoding: Boolean]) : Promise<Uint8Array>
  ```

  ### 参数
  `data`

  类型: **any**

  需要解码的 Base64 数据。

  `[isUrlEncoding]`

  类型: **Boolean**

  可选。是否使用 URL 安全字符集。默认为 **false**。

  ### 返回值
  类型: **Promise&lt;Uint8Array&gt;**

  返回值的决议值为解码后的二进制数据。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量超过 **2**。

  &bull; 第二个参数不是布尔值。

  &bull; 解码失败。

  ### 备注
  **decodeBase64** 函数在后台线程中将 Base64 字符串解码为二进制数据。若指定了 URL 安全编码，则使用对应的字符集。

  ### 例
  以下示例代码演示如何使用 **decodeBase64**。
  ```js
  include('cjs:bytebuffer');

  const bin = await bytebuffer.decodeBase64('SGVsbG8=');
  console.log(bin);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.decodeBase64 |

  ---

  ## decodeBase62 函数
  将 Base62 编码的数据解码为二进制。

  ### 语法
  ```js
  bytebuffer.decodeBase62(data: any) : Promise<Uint8Array>
  ```

  ### 参数
  `data`

  类型: **any**

  需要解码的 Base62 数据。

  ### 返回值
  类型: **Promise&lt;Uint8Array&gt;**

  返回值的决议值为解码后的二进制数据。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  &bull; 解码失败。

  ### 备注
  **decodeBase62** 函数在后台线程中将 Base62 字符串解码为二进制数据。

  ### 例
  以下示例代码演示如何使用 **decodeBase62**。
  ```js
  include('cjs:bytebuffer');

  const bin = await bytebuffer.decodeBase62('Hello');
  console.log(bin);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.decodeBase62 |

  ---

  ## decodeBase58 函数
  将 Base58 编码的数据解码为二进制。

  ### 语法
  ```js
  bytebuffer.decodeBase58(data: any) : Promise<Uint8Array>
  ```

  ### 参数
  `data`

  类型: **any**

  需要解码的 Base58 数据。

  ### 返回值
  类型: **Promise&lt;Uint8Array&gt;**

  返回值的决议值为解码后的二进制数据。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  &bull; 解码失败。

  ### 备注
  **decodeBase58** 函数在后台线程中将 Base58 字符串解码为二进制数据。

  ### 例
  以下示例代码演示如何使用 **decodeBase58**。
  ```js
  include('cjs:bytebuffer');

  const bin = await bytebuffer.decodeBase58('Hello');
  console.log(bin);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.decodeBase58 |

  ---

  ## decodeBase32 函数
  将 Base32 编码的数据解码为二进制。

  ### 语法
  ```js
  bytebuffer.decodeBase32(data: any) : Promise<Uint8Array>
  ```

  ### 参数
  `data`

  类型: **any**

  需要解码的 Base32 数据。

  ### 返回值
  类型: **Promise&lt;Uint8Array&gt;**

  返回值的决议值为解码后的二进制数据。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  &bull; 解码失败。

  ### 备注
  **decodeBase32** 函数在后台线程中将 Base32 字符串解码为二进制数据。

  ### 例
  以下示例代码演示如何使用 **decodeBase32**。
  ```js
  include('cjs:bytebuffer');

  const bin = await bytebuffer.decodeBase32('JBSWY3DP');
  console.log(bin);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.decodeBase32 |

  ---

  ## decodeBase16 函数
  将 Base16（十六进制）编码的数据解码为二进制。

  ### 语法
  ```js
  bytebuffer.decodeBase16(data: any) : Promise<Uint8Array>
  ```

  ### 参数
  `data`

  类型: **any**

  需要解码的 Base16 数据。

  ### 返回值
  类型: **Promise&lt;Uint8Array&gt;**

  返回值的决议值为解码后的二进制数据。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  &bull; 解码失败。

  ### 备注
  **decodeBase16** 函数在后台线程中将十六进制字符串解码为二进制数据。

  ### 例
  以下示例代码演示如何使用 **decodeBase16**。
  ```js
  include('cjs:bytebuffer');

  const bin = await bytebuffer.decodeBase16('48656c6c6f');
  console.log(bin);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.decodeBase16 |

  ---

  ## encodeBase91 函数
  将二进制数据编码为 Base91 字符串。

  ### 语法
  ```js
  bytebuffer.encodeBase91(data: any[, isUrlEncoding: Boolean]) : Promise<String>
  ```

  ### 参数
  `data`

  类型: **any**

  需要编码的二进制数据。

  `[isUrlEncoding]`

  类型: **Boolean**

  可选。是否使用 URL 安全字符集。默认为 **false**。

  ### 返回值
  类型: **Promise&lt;String&gt;**

  返回值的决议值为编码后的 Base91 字符串。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量超过 **2**。

  &bull; 第二个参数不是布尔值。

  &bull; 编码失败。

  ### 备注
  **encodeBase91** 函数在后台线程中将二进制数据编码为 Base91 字符串。若指定了 URL 安全编码，则使用对应的字符集。

  ### 例
  以下示例代码演示如何使用 **encodeBase91**。
  ```js
  include('cjs:bytebuffer');

  const str = await bytebuffer.encodeBase91([72, 101, 108, 108, 111]);
  console.log(str);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.encodeBase91 |

  ---

  ## encodeBase85 函数
  将二进制数据编码为 Base85 字符串。

  ### 语法
  ```js
  bytebuffer.encodeBase85(data: any) : Promise<String>
  ```

  ### 参数
  `data`

  类型: **any**

  需要编码的二进制数据。

  ### 返回值
  类型: **Promise&lt;String&gt;**

  返回值的决议值为编码后的 Base85 字符串。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  &bull; 编码失败。

  ### 备注
  **encodeBase85** 函数在后台线程中将二进制数据编码为 Base85 字符串。

  ### 例
  以下示例代码演示如何使用 **encodeBase85**。
  ```js
  include('cjs:bytebuffer');

  const str = await bytebuffer.encodeBase85([72, 101, 108, 108, 111]);
  console.log(str);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.encodeBase85 |

  ---

  ## encodeBase64 函数
  将二进制数据编码为 Base64 字符串。

  ### 语法
  ```js
  bytebuffer.encodeBase64(data: any[, isUrlEncoding: Boolean]) : Promise<String>
  ```

  ### 参数
  `data`

  类型: **any**

  需要编码的二进制数据。

  `[isUrlEncoding]`

  类型: **Boolean**

  可选。是否使用 URL 安全字符集。默认为 **false**。

  ### 返回值
  类型: **Promise&lt;String&gt;**

  返回值的决议值为编码后的 Base64 字符串。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量超过 **2**。

  &bull; 第二个参数不是布尔值。

  &bull; 编码失败。

  ### 备注
  **encodeBase64** 函数在后台线程中将二进制数据编码为 Base64 字符串。若指定了 URL 安全编码，则使用对应的字符集。

  ### 例
  以下示例代码演示如何使用 **encodeBase64**。
  ```js
  include('cjs:bytebuffer');

  const str = await bytebuffer.encodeBase64([72, 101, 108, 108, 111]);
  console.log(str); // "SGVsbG8="
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.encodeBase64 |

  ---

  ## encodeBase62 函数
  将二进制数据编码为 Base62 字符串。

  ### 语法
  ```js
  bytebuffer.encodeBase62(data: any) : Promise<String>
  ```

  ### 参数
  `data`

  类型: **any**

  需要编码的二进制数据。

  ### 返回值
  类型: **Promise&lt;String&gt;**

  返回值的决议值为编码后的 Base62 字符串。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  &bull; 编码失败。

  ### 备注
  **encodeBase62** 函数在后台线程中将二进制数据编码为 Base62 字符串。

  ### 例
  以下示例代码演示如何使用 **encodeBase62**。
  ```js
  include('cjs:bytebuffer');

  const str = await bytebuffer.encodeBase62([72, 101, 108, 108, 111]);
  console.log(str);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.encodeBase62 |

  ---

  ## encodeBase58 函数
  将二进制数据编码为 Base58 字符串。

  ### 语法
  ```js
  bytebuffer.encodeBase58(data: any) : Promise<String>
  ```

  ### 参数
  `data`

  类型: **any**

  需要编码的二进制数据。

  ### 返回值
  类型: **Promise&lt;String&gt;**

  返回值的决议值为编码后的 Base58 字符串。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  &bull; 编码失败。

  ### 备注
  **encodeBase58** 函数在后台线程中将二进制数据编码为 Base58 字符串。

  ### 例
  以下示例代码演示如何使用 **encodeBase58**。
  ```js
  include('cjs:bytebuffer');

  const str = await bytebuffer.encodeBase58([72, 101, 108, 108, 111]);
  console.log(str);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.encodeBase58 |

  ---

  ## encodeBase32 函数
  将二进制数据编码为 Base32 字符串。

  ### 语法
  ```js
  bytebuffer.encodeBase32(data: any) : Promise<String>
  ```

  ### 参数
  `data`

  类型: **any**

  需要编码的二进制数据。

  ### 返回值
  类型: **Promise&lt;String&gt;**

  返回值的决议值为编码后的 Base32 字符串。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  &bull; 编码失败。

  ### 备注
  **encodeBase32** 函数在后台线程中将二进制数据编码为 Base32 字符串。

  ### 例
  以下示例代码演示如何使用 **encodeBase32**。
  ```js
  include('cjs:bytebuffer');

  const str = await bytebuffer.encodeBase32([72, 101, 108, 108, 111]);
  console.log(str); // "JBSWY3DP"
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.encodeBase32 |

  ---

  ## encodeBase16 函数
  将二进制数据编码为 Base16（十六进制）字符串。

  ### 语法
  ```js
  bytebuffer.encodeBase16(data: any) : Promise<String>
  ```

  ### 参数
  `data`

  类型: **any**

  需要编码的二进制数据。

  ### 返回值
  类型: **Promise&lt;String&gt;**

  返回值的决议值为编码后的十六进制字符串。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  &bull; 编码失败。

  ### 备注
  **encodeBase16** 函数在后台线程中将二进制数据编码为十六进制字符串。

  ### 例
  以下示例代码演示如何使用 **encodeBase16**。
  ```js
  include('cjs:bytebuffer');

  const str = await bytebuffer.encodeBase16([72, 101, 108, 108, 111]);
  console.log(str); // "48656c6c6f"
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.encodeBase16 |

  ---

  ## toBinary 函数
  将数据转换为二进制 Uint8Array。

  ### 语法
  ```js
  bytebuffer.toBinary(data: any) : Promise<Uint8Array>
  ```

  ### 参数
  `data`

  类型: **any**

  需要转换的数据。

  ### 返回值
  类型: **Promise&lt;Uint8Array&gt;**

  返回值的决议值为转换后的二进制数据。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  ### 备注
  **toBinary** 函数在后台线程中将传入的数据转换为 Uint8Array 形式的二进制数据。

  ### 例
  以下示例代码演示如何使用 **toBinary**。
  ```js
  include('cjs:bytebuffer');

  const bin = await bytebuffer.toBinary('Hello');
  console.log(bin);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.toBinary |

  ---

  ## toString 函数
  将数据转换为字符串。

  ### 语法
  ```js
  bytebuffer.toString(data: any) : Promise<String>
  ```

  ### 参数
  `data`

  类型: **any**

  需要转换为字符串的数据。

  ### 返回值
  类型: **Promise&lt;String&gt;**

  返回值的决议值为转换后的字符串。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  ### 备注
  **toString** 函数在后台线程中将传入的数据转换为字符串。

  ### 例
  以下示例代码演示如何使用 **toString**。
  ```js
  include('cjs:bytebuffer');

  const str = await bytebuffer.toString([72, 101, 108, 108, 111]);
  console.log(str); // "Hello"
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | bytebuffer |
  | 可用性 | 全局可用 |
  | 位置 | bytebuffer.toString |

  ---

# network 对象

  ## http.open 函数
  创建一个 XMLHttpRequest 对象并打开连接。

  ### 语法
  ```js
  network.http.open(method: String, url: String[, async: Boolean[, username: String[, password: String]]]) : Object
  ```

  ### 参数
  `method`

  类型: **String**

  需要使用的 HTTP 请求方法，例如 `"GET"`、`"POST"` 等。

  `url`

  类型: **String**

  需要请求的目标地址。

  `[async]`

  类型: **Boolean**

  可选。是否使用异步请求。默认为 **true**。

  `[username]`

  类型: **String**

  可选。用于认证的用户名。默认为空字符串。

  `[password]`

  类型: **String**

  可选。用于认证的密码。默认为空字符串。

  ### 返回值
  类型: **Object**

  如果函数成功，则返回一个 XMLHttpRequest 对象。返回值的结构如下：
  ```js
  {
      id: BigInt,
      isAsync: Boolean,
      UNSENT: Number,
      OPENED: Number,
      HEADERS_RECEIVED: Number,
      LOADING: Number,
      DONE: Number,
      readyState: Number,
      response: String,
      responseType: String,
      status: Number,
      statusText: String,
      onreadystatechange: Function | null,
      onloadstart: Function | null,
      onload: Function | null,
      onloadend: Function | null,
      onprogress: Function | null,
      onerror: Function | null,
      onabort: Function | null,
      ontimeout: Function | null,
      onheadersreceived: Function | null,
      upload: Object,
      open: Function,
      send: Function,
      overrideMimeType: Function,
      setRequestHeader: Function,
      getResponseHeader: Function,
      getAllResponseHeaders: Function,
      abort: Function,
      close: Function
  }
  ```
  `id`

  类型: **BigInt**

  当前 XMLHttpRequest 实例的内部标识。

  `isAsync`

  类型: **Boolean**

  当前实例是否以异步方式请求。

  `UNSENT`

  类型: **Number**

  值为 **0**，表示未初始化状态。

  `OPENED`

  类型: **Number**

  值为 **1**，表示已打开状态。

  `HEADERS_RECEIVED`

  类型: **Number**

  值为 **2**，表示已接收到响应头。

  `LOADING`

  类型: **Number**

  值为 **3**，表示正在接收响应体。

  `DONE`

  类型: **Number**

  值为 **4**，表示请求已完成。

  `readyState`

  类型: **Number**

  当前请求的进行状态。

  `response`

  类型: **String**

  响应内容，其类型取决于 `responseType` 的设置。

  `responseType`

  类型: **String**

  响应类型，支持 `""`、`"text"`、`"json"`、`"arrayBuffer"` 以及二进制类型。

  `status`

  类型: **Number**

  HTTP 状态码。

  `statusText`

  类型: **String**

  HTTP 状态文本。

  `onreadystatechange`

  类型: **Function** 或 **null**

  当 `readyState` 发生变化时触发的回调。

  `onloadstart`

  类型: **Function** 或 **null**

  请求开始加载时触发的回调。

  `onload`

  类型: **Function** 或 **null**

  请求成功完成时触发的回调。

  `onloadend`

  类型: **Function** 或 **null**

  请求完成（无论成功与否）时触发的回调。

  `onprogress`

  类型: **Function** 或 **null**

  请求进行中周期性触发的回调。

  `onerror`

  类型: **Function** 或 **null**

  请求发生错误时触发的回调。

  `onabort`

  类型: **Function** 或 **null**

  请求被中止时触发的回调。

  `ontimeout`

  类型: **Function** 或 **null**

  请求超时时触发的回调。

  `onheadersreceived`

  类型: **Function** 或 **null**

  接收到响应头时触发的回调。

  `upload`

  类型: **Object**

  上传事件对象，结构与主对象类似，包含 `onloadstart`、`onload`、`onloadend`、`onprogress`、`onerror`、`onabort`、`ontimeout` 等回调。

  有关各个回调的说明，请参阅 [ProgressEvent 对象](#progressevent-对象)。

  此函数通常因以下原因之一而抛出异常:

  &bull; 传入参数的数量不在 **2** 至 **5** 之间。

  &bull; 参数类型错误（如 `method`、`url` 不是字符串）。

  &bull; 创建 XMLHttpRequest 失败。

  ### 备注
  **open** 函数会创建一个 XMLHttpRequest 对象并立即调用其内部 `open` 方法，同时设置默认的请求状态、响应属性以及各事件的默认回调。返回的对象支持常见的 XHR 属性与事件回调，可对其进行进一步配置并调用 **send** 发起请求。

  ### 例
  以下示例代码演示如何使用 **http.open**。
  ```js
  include('cjs:network');

  const xhr = network.http.open('GET', 'https://example.com/');
  xhr.onload = function () {
      console.log(xhr.status, xhr.response);
  };
  xhr.send();
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | network |
  | 可用性 | 全局可用 |
  | 位置 | network.http.open |

  ---

  ## ProgressEvent 对象
  由 **network.http.open** 返回对象的回调接收的事件对象。

  ### 属性
  | 属性 | 类型 | 说明 |
  | :- | :- | :- |
  | `type` | **String** | 事件类型，如 `"load"`、`"progress"`、`"error"` 等。 |
  | `loaded` | **Number** | 已加载的字节数。 |
  | `total` | **Number** | 总字节数。 |
  | `lengthComputable` | **Boolean** | 总字节数是否可计算。 |
  | `target` | **Object** | 触发事件的目标对象。 |

  ---

  ## XMLHttpRequest 对象

  
  由 **network.http.open** 返回对象的原型对象。

  ### open 方法
  重新配置当前 XMLHttpRequest 对象。

  #### 语法
  ```js
  open(method: String, url: String[, async: Boolean[, username: String[, password: String]]]) : void
  ```

  #### 参数
  `method`

  类型: **String**

  需要使用的 HTTP 请求方法。

  `url`

  类型: **String**

  需要请求的目标地址。

  `[async]`

  类型: **Boolean**

  可选。是否使用异步请求。默认为 **true**。

  `[username]`

  类型: **String**

  可选。用于认证的用户名。默认为空字符串。

  `[password]`

  类型: **String**

  可选。用于认证的密码。默认为空字符串。

  #### 返回值
  无

  #### 备注
  调用后会将 `readyState`、`response`、`responseType`、`status`、`statusText` 重置为初始值。

  ---

  ### send 方法
  发送当前请求。

  #### 语法
  ```js
  send([body: String | Uint8Array | ArrayBuffer | FormData | Blob]) : void
  ```

  #### 参数
  `[body]`

  类型: **String**、**Uint8Array**、**ArrayBuffer**、**FormData** 或 **Blob**

  可选。请求体内容。省略或传入 **null** / **undefined** 时不携带请求体。文本类型数据会被转换为字节缓冲后发送。

  #### 返回值
  无

  #### 备注
  调用前必须确保 `readyState` 为 **OPENED** 且尚未调用过 **send**。异步模式下会在后台线程中执行请求，不阻塞主线程。

  ---

  ### overrideMimeType 方法
  覆盖响应的 MIME 类型。

  #### 语法
  ```js
  overrideMimeType(mimeString: String) : void
  ```

  #### 参数
  `mimeString`

  类型: **String**

  需要设置的 MIME 类型字符串。

  #### 返回值
  无

  ---

  ### setRequestHeader 方法
  设置请求头。

  #### 语法
  ```js
  setRequestHeader(name: String, value: String) : void
  ```

  #### 参数
  `name`

  类型: **String**

  请求头名称。

  `value`

  类型: **String**

  请求头值。

  #### 返回值
  无

  ---

  ### getResponseHeader 方法
  获取指定的响应头。

  #### 语法
  ```js
  getResponseHeader(name: String) : String
  ```

  #### 参数
  `name`

  类型: **String**

  响应头名称。

  #### 返回值
  类型: **String**

  返回对应响应头的值。

  ---

  ### getAllResponseHeaders 方法
  获取所有响应头。

  #### 语法
  ```js
  getAllResponseHeaders() : String | null
  ```

  #### 参数
  无

  #### 返回值
  类型: **String** 或 **null**

  返回以 `\r\n` 分隔的所有响应头文本；若不存在响应头，则返回 **null**。

  ---

  ### abort 方法
  中止当前请求。

  #### 语法
  ```js
  abort() : void
  ```

  #### 参数
  无

  #### 返回值
  无

  #### 备注
  若当前实例已被关闭，则抛出异常。

  ---

  ### close 方法
  关闭当前 XMLHttpRequest 实例并释放资源。

  #### 语法
  ```js
  close() : void
  ```

  #### 参数
  无

  #### 返回值
  无

  #### 备注
  关闭后该实例失效。重复调用 **close** 会抛出异常。

  ---

  ## request 属性
  当前请求的请求对象。

  ### 语法
  ```js
  network.request: RequestNetwork
  ```

  ### 格式
  ```js
  {
      workDirectory: String,
      url: String,
      method: String,
      scriptPath: String,
      path: String,
      header: Object,
      advHeader: Object,
      body: Uint8Array | null
  }
  ```
  `workDirectory`

  类型: **String**

  当前请求脚本所在目录的绝对路径。

  `url`

  类型: **String**

  当前请求的 URL，等价于 CGI 环境变量 `REQUEST_URI`。

  `method`

  类型: **String**

  当前请求的 HTTP 方法，例如 `"GET"`、`"POST"`。

  `scriptPath`

  类型: **String**

  当前请求的脚本路径，等价于 CGI 环境变量 `SCRIPT_FILENAME`。

  `path`

  类型: **String**

  当前请求所使用的脚本的完整路径。

  `header`

  类型: **Object**

  当前请求的 HTTP 头，键为格式化后的头部名称，值为头部内容。

  `advHeader`

  类型: **Object**

  当前请求的附加信息对象。有关 **advHeader** 的说明，请参阅 [network.request.advHeader](#advheader-属性)。

  `body`

  类型: **Uint8Array** 或 **null**

  当前请求的请求体。当请求方法不需要读取请求体（如在现代模式下为 `GET`、`HEAD`、`DELETE`、`OPTIONS`、`TRACE`、`CONNECT`），或请求体为空时为 **null**。

  ### 例
  以下示例代码演示如何获取 **request**。
  ```js
  include('cjs:network');

  console.log(network.request.method);
  console.log(network.request.url);
  console.log(network.request.header);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 仅在 FCGI 模式下可用 |
  | 位置 | network.request |

  ---

  ### advHeader 属性
  当前请求的附加信息。

  #### 语法
  ```js
  network.request.advHeader: Object
  ```

  #### 格式
  ```js
  {
      query: String,
      contentLength: String,
      remoteIp: String,
      remotePort: String,
      host: String,
      userAgent: String,
      protocol: String,
      realIp: String,
      scheme: String,
      referer: String,
      contentType: String,
      documentRoot: String
  }
  ```
  `query`

  类型: **String**

  当前请求的查询字符串，等价于 CGI 环境变量 `QUERY_STRING`。

  `contentLength`

  类型: **String**

  当前请求的请求体长度，等价于 CGI 环境变量 `CONTENT_LENGTH`。

  `remoteIp`

  类型: **String**

  当前请求的客户端 IP 地址，等价于 CGI 环境变量 `REMOTE_ADDR`。

  `remotePort`

  类型: **String**

  当前请求的客户端端口，等价于 CGI 环境变量 `REMOTE_PORT`。

  `host`

  类型: **String**

  当前请求的主机名，等价于 CGI 环境变量 `HTTP_HOST`。

  `userAgent`

  类型: **String**

  当前请求的用户代理，等价于 CGI 环境变量 `HTTP_USER_AGENT`。

  `protocol`

  类型: **String**

  当前请求的协议版本，等价于 CGI 环境变量 `SERVER_PROTOCOL`。

  `realIp`

  类型: **String**

  当前请求的真实客户端 IP 地址。若请求头中包含 `HTTP_X_FORWARDED_FOR`，则取该值；否则取 `REMOTE_ADDR`。

  `scheme`

  类型: **String**

  当前请求所使用的协议方案。若请求头中包含 `HTTP_X_FORWARDED_PROTO`，则取该值；否则根据 `SERVER_PORT` 是否为 `443` 返回 `"https"` 或 `"http"`。

  `referer`

  类型: **String**

  当前请求的来源地址，等价于 CGI 环境变量 `HTTP_REFERER`。

  `contentType`

  类型: **String**

  当前请求的内容类型，等价于 CGI 环境变量 `CONTENT_TYPE`。

  `documentRoot`

  类型: **String**

  当前请求的文档根目录，等价于 CGI 环境变量 `DOCUMENT_ROOT`。

  #### 例
  以下示例代码演示如何获取 **advHeader**。
  ```js
  include('cjs:network');

  console.log(network.request.advHeader.query);
  console.log(network.request.advHeader.realIp);
  console.log(network.request.advHeader.scheme);
  ```

  #### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 仅在 FCGI 模式下可用 |
  | 位置 | network.request.advHeader |

  ---

  ## response 属性
  当前请求的响应对象。

  ### 语法
  ```js
  network.response: ResponseNetwork
  ```

  ### 格式
  ```js
  {
      header: Object,
      body: any,
      setResponseCode: Function
  }
  ```
  `header`

  类型: **Object**

  当前响应的响应头。默认包含以下字段：

  &bull; `Status`：值为 `"200 OK"`。

  &bull; `Allow`：值为 `"GET, HEAD, OPTIONS"`。

  &bull; `Content-Type`：值为 `"text/plain; charset=UTF-8"`。

  可以通过修改 `header` 中的属性来定制响应头。

  `body`

  类型: **any**

  当前响应的响应体，支持多种类型，具体行为如下：

  &bull; **ArrayBufferView**：作为二进制响应体发送，并自动设置 `Content-Type` 为 `application/octet-stream`，设置 `Content-Length` 为字节长度。

  &bull; **Number**：转换为十进制字符串后作为响应体。

  &bull; **Boolean**：转换为 `"true"` 或 `"false"` 后作为响应体。

  &bull; **String**：作为文本响应体。

  &bull; **null**：发送 `"null"`。

  &bull; **undefined**：发送空内容。

  &bull; 其他对象：发送 `"[object 类型名]"` 形式的文本。

  `setResponseCode`

  类型: **Function**

  设置响应状态码。

  语法：
  ```js
  setResponseCode([code: Number]) : void
  ```

  参数：
  `[code]`

  类型: **Number**

  可选。HTTP 状态码，默认为 **200**。

  返回值：
  无

  备注：
  **setResponseCode** 会根据传入的状态码自动查找对应的状态文本，并将其写入 `header.Status`。

  有关 **setResponseCode** 的说明，请参阅 [network.response.setResponseCode](#setresponsecode-函数)。

  ### 例
  以下示例代码演示如何获取 **response**。
  ```js
  include('cjs:network');

  network.response.header['Content-Type'] = 'application/json; charset=utf-8';
  network.response.body = '{"ok": true}';
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 仅在 FCGI 模式下可用 |
  | 位置 | network.response |

  ---

  ### setResponseCode 函数
  设置当前响应的状态码。

  #### 语法
  ```js
  network.response.setResponseCode([code: Number]) : void
  ```

  #### 参数
  `[code]`

  类型: **Number**

  可选。需要设置的 HTTP 状态码，默认为 **200**。

  #### 返回值
  无

  #### 备注
  **setResponseCode** 函数会根据传入的状态码从内置的状态码表中查找对应的状态文本，并将结果（如 `"200 OK"`）写入 `network.response.header.Status`。

  #### 例
  以下示例代码演示如何使用 **setResponseCode**。
  ```js
  include('cjs:network');

  network.response.setResponseCode(404);
  console.log(network.response.header.Status); //输出 "404 Not Found"
  ```

  #### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | 无 |
  | 可用性 | 仅在 FCGI 模式下可用 |
  | 位置 | network.response.setResponseCode |

  ---

# crypto 对象

  ## getRandomValues 函数
  使用加密安全的随机数填充指定的 ArrayBufferView。

  ### 语法
  ```js
  crypto.getRandomValues(array: ArrayBufferView) : ArrayBufferView
  ```

  ### 参数
  `array`

  类型: **ArrayBufferView**

  需要填充的数组视图，支持 **Uint8Array**、**Uint16Array**、**Uint32Array**、**Int8Array**、**Int16Array**、**Int32Array**。

  ### 返回值
  类型: **Uint8Array | Uint16Array | Uint32Array | Int8Array | Int16Array | Int32Array**

  返回填充随机数后的数组视图。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **1**。

  &bull; 参数不是受支持的 ArrayBufferView 类型。

  &bull; 请求的字节长度超过 **65536**。

  &bull; 随机数生成操作失败。

  ### 备注
  **getRandomValues** 函数会使用加密安全的随机数生成器填充传入的数组视图，并返回该视图。请求长度不能超过 **65536** 字节，否则会抛出 `QuotaExceededError`。

  ### 例
  以下示例代码演示如何使用 **getRandomValues**。
  ```js
  include('cjs:crypto');

  const arr = new Uint8Array(16);
  crypto.getRandomValues(arr);
  console.log(arr);
  ```

  ### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | crypto |
  | 可用性 | 全局可用 |
  | 位置 | crypto.getRandomValues |

  ---

  ## subtle 对象
  提供加密算法相关功能。

  ### deriveBits 函数
  从基础密钥派生指定长度的位。

  #### 语法
  ```js
  crypto.subtle.deriveBits(algorithm: Object, baseKey: CryptoKey[, length: Number]) : Promise<ArrayBuffer>
  ```

  #### 参数
  `algorithm`

  类型: **Object**

  派生算法对象，必须包含 `name` 属性。支持 `PBKDF2`、`ECDH`、`HKDF`。

  `baseKey`

  类型: **CryptoKey**

  基础密钥，其 `usages` 必须包含 `deriveBits` 或 `deriveKey`。

  `[length]`

  类型: **Number**

  可选。需要派生的位数，必须为正整数且为 **8** 的倍数。`PBKDF2` 和 `HKDF` 必须提供该参数。

  #### 返回值
  类型: **Promise&lt;ArrayBuffer&gt;**

  返回值的决议值为派生出的位数据。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不在 **2** 至 **3** 之间。

  &bull; `algorithm` 不是对象，或缺少 `name` 属性。

  &bull; `baseKey` 不是 CryptoKey，或缺少有效用途。

  &bull; `length` 无效。

  &bull; 算法不支持，或缺少必要属性。

  &bull; 派生操作失败。

  #### 备注
  **deriveBits** 函数支持 `PBKDF2`、`ECDH`、`HKDF` 三种派生算法。派生过程在后台线程中执行，不会阻塞主线程。

  #### 例
  ```js
  include('cjs:crypto');

  // 假设 baseKey 为已生成的 CryptoKey
  const bits = await crypto.subtle.deriveBits(
      { name: 'PBKDF2', salt: new Uint8Array(16), iterations: 100000, hash: 'SHA-256' },
      baseKey,
      256
  );
  console.log(bits);
  ```

  #### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | crypto |
  | 可用性 | 全局可用 |
  | 位置 | crypto.subtle.deriveBits |

  ---

  ### deriveKey 函数
  从基础密钥派生新的 CryptoKey。

  #### 语法
  ```js
  crypto.subtle.deriveKey(algorithm: Object, baseKey: CryptoKey, derivedKeyAlgorithm: Object, extractable: Boolean, keyUsages: Array) : Promise<CryptoKey>
  ```

  #### 参数
  `algorithm`

  类型: **Object**

  派生算法对象，支持 `PBKDF2`、`ECDH`、`HKDF`。

  `baseKey`

  类型: **CryptoKey**

  基础密钥，其 `usages` 必须包含 `deriveBits` 或 `deriveKey`。

  `derivedKeyAlgorithm`

  类型: **Object**

  派生密钥的算法对象，必须包含 `name` 属性。

  `extractable`

  类型: **Boolean**

  派生密钥是否可导出。

  `keyUsages`

  类型: **Array**

  派生密钥的用途数组，不能为空。

  #### 返回值
  类型: **Promise&lt;CryptoKey&gt;**

  返回值的决议值为派生出的 CryptoKey。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **5**。

  &bull; 参数类型错误。

  &bull; 基础密钥缺少有效用途。

  &bull; 派生密钥算法不支持，或用途无效。

  &bull; 派生操作失败。

  #### 备注
  **deriveKey** 函数支持 `PBKDF2`、`ECDH`、`HKDF` 三种派生算法，派生出的密钥算法可为 AES、HMAC 等。派生过程在后台线程中执行。

  #### 例
  ```js
  include('cjs:crypto');

  const derivedKey = await crypto.subtle.deriveKey(
      { name: 'PBKDF2', salt: new Uint8Array(16), iterations: 100000, hash: 'SHA-256' },
      baseKey,
      { name: 'AES-GCM', length: 256 },
      true,
      ['encrypt', 'decrypt']
  );
  console.log(derivedKey);
  ```

  #### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | crypto |
  | 可用性 | 全局可用 |
  | 位置 | crypto.subtle.deriveKey |

  ---

  ### sign 函数
  使用指定密钥对数据进行签名。

  #### 语法
  ```js
  crypto.subtle.sign(algorithm: Object, key: CryptoKey, data: ArrayBufferView) : Promise<ArrayBuffer>
  ```

  #### 参数
  `algorithm`

  类型: **Object**

  签名算法对象，支持 `RSA-PSS`、`RSASSA-PKCS1-v1_5`、`ECDSA`、`HMAC`。

  `key`

  类型: **CryptoKey**

  签名密钥，其 `usages` 必须包含 `sign`。

  `data`

  类型: **ArrayBufferView**

  需要签名的数据。

  #### 返回值
  类型: **Promise&lt;ArrayBuffer&gt;**

  返回值的决议值为签名结果。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **3**。

  &bull; 参数类型错误。

  &bull; 密钥用途不包含 `sign`。

  &bull; 算法不支持，或缺少必要属性。

  &bull; 签名操作失败。

  #### 备注
  **sign** 函数支持 `RSA-PSS`、`RSASSA-PKCS1-v1_5`、`ECDSA`、`HMAC`。签名过程在后台线程中执行。

  #### 例
  ```js
  include('cjs:crypto');

  const signature = await crypto.subtle.sign(
      { name: 'HMAC', hash: 'SHA-256' },
      key,
      new TextEncoder().encode('hello')
  );
  console.log(signature);
  ```

  #### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | crypto |
  | 可用性 | 全局可用 |
  | 位置 | crypto.subtle.sign |

  ---

  ### verify 函数
  使用指定密钥验证签名。

  #### 语法
  ```js
  crypto.subtle.verify(algorithm: Object, key: CryptoKey, signature: ArrayBufferView, data: ArrayBufferView) : Promise<Boolean>
  ```

  #### 参数
  `algorithm`

  类型: **Object**

  验证算法对象，支持 `RSA-PSS`、`RSASSA-PKCS1-v1_5`、`ECDSA`、`HMAC`。

  `key`

  类型: **CryptoKey**

  验证密钥，其 `usages` 必须包含 `verify`。

  `signature`

  类型: **ArrayBufferView**

  签名数据。

  `data`

  类型: **ArrayBufferView**

  原始数据。

  #### 返回值
  类型: **Promise&lt;Boolean&gt;**

  返回值的决议值为验证是否通过。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **4**。

  &bull; 参数类型错误。

  &bull; 密钥用途不包含 `verify`。

  &bull; 算法不支持，或缺少必要属性。

  #### 备注
  **verify** 函数支持与 **sign** 相同的算法。验证过程在后台线程中执行。

  #### 例
  ```js
  include('cjs:crypto');

  const ok = await crypto.subtle.verify(
      { name: 'HMAC', hash: 'SHA-256' },
      key,
      signature,
      new TextEncoder().encode('hello')
  );
  console.log(ok);
  ```

  #### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | crypto |
  | 可用性 | 全局可用 |
  | 位置 | crypto.subtle.verify |

  ---

  ### encrypt 函数
  使用指定密钥加密数据。

  #### 语法
  ```js
  crypto.subtle.encrypt(algorithm: Object, key: CryptoKey, data: ArrayBufferView) : Promise<ArrayBuffer>
  ```

  #### 参数
  `algorithm`

  类型: **Object**

  加密算法对象，支持 `AES-GCM`、`AES-CBC`、`AES-CTR`、`ChaCha20-Poly1305`、`RSA-OAEP`。

  `key`

  类型: **CryptoKey**

  加密密钥，其 `usages` 必须包含 `encrypt` 或 `wrapKey`。

  `data`

  类型: **ArrayBufferView**

  需要加密的明文数据。

  #### 返回值
  类型: **Promise&lt;ArrayBuffer&gt;**

  返回值的决议值为密文数据。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **3**。

  &bull; 参数类型错误。

  &bull; 密钥用途无效。

  &bull; 算法不支持，或缺少必要属性。

  &bull; 加密操作失败。

  #### 备注
  **encrypt** 函数支持对称加密与非对称加密算法。加密过程在后台线程中执行。

  #### 例
  ```js
  include('cjs:crypto');

  const cipher = await crypto.subtle.encrypt(
      { name: 'AES-GCM', iv: new Uint8Array(12) },
      key,
      new TextEncoder().encode('hello')
  );
  console.log(cipher);
  ```

  #### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | crypto |
  | 可用性 | 全局可用 |
  | 位置 | crypto.subtle.encrypt |

  ---

  ### decrypt 函数
  使用指定密钥解密数据。

  #### 语法
  ```js
  crypto.subtle.decrypt(algorithm: Object, key: CryptoKey, data: ArrayBufferView) : Promise<ArrayBuffer>
  ```

  #### 参数
  `algorithm`

  类型: **Object**

  解密算法对象，支持 `AES-GCM`、`AES-CBC`、`AES-CTR`、`ChaCha20-Poly1305`、`RSA-OAEP`。

  `key`

  类型: **CryptoKey**

  解密密钥，其 `usages` 必须包含 `decrypt` 或 `unwrapKey`。

  `data`

  类型: **ArrayBufferView**

  需要解密的密文数据。

  #### 返回值
  类型: **Promise&lt;ArrayBuffer&gt;**

  返回值的决议值为明文数据。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **3**。

  &bull; 参数类型错误。

  &bull; 密钥用途无效。

  &bull; 算法不支持，或缺少必要属性。

  &bull; 解密操作失败。

  #### 备注
  **decrypt** 函数支持与 **encrypt** 相同的算法。解密过程在后台线程中执行。

  #### 例
  ```js
  include('cjs:crypto');

  const plain = await crypto.subtle.decrypt(
      { name: 'AES-GCM', iv: new Uint8Array(12) },
      key,
      cipher
  );
  console.log(new TextDecoder().decode(plain));
  ```

  #### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | crypto |
  | 可用性 | 全局可用 |
  | 位置 | crypto.subtle.decrypt |

  ---

  ### digest 函数
  计算数据的摘要。

  #### 语法
  ```js
  crypto.subtle.digest(algorithm: Object, data: ArrayBufferView) : Promise<ArrayBuffer>
  ```

  #### 参数
  `algorithm`

  类型: **Object**

  摘要算法对象，必须包含 `name` 属性，支持 SHA 系列哈希算法。

  `data`

  类型: **ArrayBufferView**

  需要计算摘要的数据。

  #### 返回值
  类型: **Promise&lt;ArrayBuffer&gt;**

  返回值的决议值为摘要结果。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **2**。

  &bull; `algorithm` 缺少 `name` 属性。

  &bull; `data` 不是 ArrayBufferView。

  &bull; 哈希算法不支持。

  &bull; 摘要计算失败。

  #### 备注
  **digest** 函数在后台线程中计算数据的哈希摘要。

  #### 例
  ```js
  include('cjs:crypto');

  const hash = await crypto.subtle.digest(
      { name: 'SHA-256' },
      new TextEncoder().encode('hello')
  );
  console.log(hash);
  ```

  #### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | crypto |
  | 可用性 | 全局可用 |
  | 位置 | crypto.subtle.digest |

  ---

  ### exportKey 函数
  导出指定密钥。

  #### 语法
  ```js
  crypto.subtle.exportKey(format: String, key: CryptoKey) : Promise<ArrayBuffer | Object>
  ```

  #### 参数
  `format`

  类型: **String**

  导出格式，支持 `raw`、`spki`、`pkcs8`、`jwk`。

  `key`

  类型: **CryptoKey**

  需要导出的密钥，必须可导出（`extractable` 为 **true**）。

  #### 返回值
  类型: **Promise&lt;ArrayBuffer | Object&gt;**

  当 `format` 为 `jwk` 时，决议值为 JWK 对象；否则决议值为 ArrayBuffer。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **2**。

  &bull; `format` 不是字符串。

  &bull; `key` 不是 CryptoKey。

  &bull; 密钥不可导出。

  &bull; 格式或算法不支持。

  #### 备注
  **exportKey** 函数支持导出对称密钥、RSA 密钥、EC 密钥以及 Ed25519、X25519 密钥。导出过程在后台线程中执行。

  #### 例
  ```js
  include('cjs:crypto');

  const raw = await crypto.subtle.exportKey('raw', key);
  console.log(raw);
  ```

  #### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | crypto |
  | 可用性 | 全局可用 |
  | 位置 | crypto.subtle.exportKey |

  ---

  ### importKey 函数
  导入密钥并生成 CryptoKey。

  #### 语法
  ```js
  crypto.subtle.importKey(format: String, keyData: ArrayBufferView | Object, algorithm: Object, extractable: Boolean, keyUsages: Array) : Promise<CryptoKey>
  ```

  #### 参数
  `format`

  类型: **String**

  导入格式，支持 `raw`、`spki`、`pkcs8`、`jwk`。

  `keyData`

  类型: **ArrayBufferView | Object**

  密钥数据。`jwk` 格式时为对象，其他格式时为 ArrayBufferView。

  `algorithm`

  类型: **Object**

  密钥算法对象，必须包含 `name` 属性。

  `extractable`

  类型: **Boolean**

  导入的密钥是否可导出。

  `keyUsages`

  类型: **Array**

  密钥用途数组。

  #### 返回值
  类型: **Promise&lt;CryptoKey&gt;**

  返回值的决议值为导入的 CryptoKey。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **5**。

  &bull; 参数类型错误。

  &bull; 格式或算法不支持。

  &bull; 密钥数据无效。

  &bull; 密钥用途无效。

  #### 备注
  **importKey** 函数支持从原始字节、SPKI、PKCS8 以及 JWK 导入密钥。导入过程在后台线程中执行。

  #### 例
  ```js
  include('cjs:crypto');

  const key = await crypto.subtle.importKey(
      'raw',
      new Uint8Array(32),
      { name: 'HMAC', hash: 'SHA-256' },
      true,
      ['sign', 'verify']
  );
  console.log(key);
  ```

  #### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | crypto |
  | 可用性 | 全局可用 |
  | 位置 | crypto.subtle.importKey |

  ---

  ### generateKey 函数
  生成新的密钥。

  #### 语法
  ```js
  crypto.subtle.generateKey(algorithm: Object, extractable: Boolean, keyUsages: Array) : Promise<CryptoKey | Object>
  ```

  #### 参数
  `algorithm`

  类型: **Object**

  密钥算法对象，必须包含 `name` 属性。支持 `HMAC`、`AES-GCM`、`AES-CBC`、`AES-CTR`、`AES-KW`、`ChaCha20-Poly1305`、`RSA-PSS`、`RSA-OAEP`、`RSASSA-PKCS1-v1_5`、`ECDSA`、`ECDH`、`Ed25519`、`X25519`。

  `extractable`

  类型: **Boolean**

  生成的密钥是否可导出。

  `keyUsages`

  类型: **Array**

  密钥用途数组，不能为空。

  #### 返回值
  类型: **Promise&lt;CryptoKey | Object&gt;**

  对称密钥决议值为 CryptoKey；非对称密钥决议值为包含 `publicKey` 与 `privateKey` 的对象。

  此函数通常因以下原因之一而失败:

  &bull; 传入参数的数量不为 **3**。

  &bull; 参数类型错误。

  &bull; 算法不支持，或缺少必要属性。

  &bull; 密钥用途无效。

  &bull; 密钥生成失败。

  #### 备注
  **generateKey** 函数在后台线程中生成密钥。对于 RSA、ECDSA、ECDH、Ed25519、X25519 等非对称算法，返回包含公钥与私钥的对象。

  #### 例
  ```js
  include('cjs:crypto');

  const key = await crypto.subtle.generateKey(
      { name: 'AES-GCM', length: 256 },
      true,
      ['encrypt', 'decrypt']
  );
  console.log(key);
  ```

  #### 要求
  | 要求 | 参数值 |
  | :- | :- |
  | 最低支持的运行时 | Cgi.js v1.0.20260328.01 |
  | 目标平台 | Windows |
  | 所属模块 | crypto |
  | 可用性 | 全局可用 |
  | 位置 | crypto.subtle.generateKey |

  ---

  
