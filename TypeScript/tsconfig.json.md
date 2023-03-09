# compileOnSave
该选项会在保存时向IDE申请自动编译，以便在保存时就能使给定的tsconfig.json生成所有文件
```json
{
  "compileOnSave": true
}
```
# exclude
指定编译器需要排除的文件或文件夹，支持glob通配符写法
```json
{
  "exclude": ["node_modules", "build", "dist"]
}
```
:::info
glob通配符写法

- *匹配零个或多个字符（不包括目录分隔符）
- ?匹配任何一个字符（不包括目录分隔符）
- **/匹配嵌套到任何级别的任何目录

如果glob模式不包含文件扩展名，则只包含支持扩展名的文件(例如.ts、.tsx和.d)。如果allowJs设置为true，则默认为.js和.jsx)。
:::
# extends
引入其他配置文件，继承配置，默认包含当前目录和子目录下所有 TypeScript 文件
目前唯一被排除在继承之外的顶级属性是references
```json
{
  "extends": "./base.json"
}
```
# files
指定需要编译的单个文件列表，默认包含当前目录和子目录下所有 TypeScript 文件
这个选项只有少量文件并且不需要使用 glob 来引用许多文件时才会使用，如果不满足请尽量使用include选项
```json
{
  "files": ["src/utils"]
}
```
# include
指定编译需要编译的文件或目录，支持glob通配符写法
```json
{
  "include": [
    "mock/**/*",
    "src/**/*",
    "test/**/*",
    "typings/**/*",
    "config/**/*",
    ".eslintrc.js",
    ".stylelintrc.js",
    ".prettierrc.js",
    "jest.config.js",
    "mock/*"
  ],
}
```
# watchOptions
编译器的实现--watch依赖于node 提供的fs.watch和fs.watchFile，这两种方式各有利弊
> 本节主要用于处理在 Linux 上fs.watch有fs.watchFile额外约束的情况
> - fs.watch使用文件系统事件来通知文件/目录中的更改
> - fs.watchFile使用轮询，因此涉及 CPU 周期
> 
详细watch类型的区别介绍可以查看[typescript配置手册](https://www.typescriptlang.org/docs/handbook/configuring-watch.html)

## watchFile 
如何监视单个文件的策略。

- fixedPollingInterval，以固定的时间间隔每秒多次检查每个文件的更改
- priorityPollingInterval，每秒检查每个文件几次更改，但使用试探法检查某些类型的文件的频率低于其他文件
- dynamicPriorityPolling，使用动态队列，不经常检查修改频率较低的文件
- useFsEvents（the default），尝试使用操作系统/文件系统的本机事件进行文件更改
- useFsEventsOnParentDirectory，尝试使用操作系统/文件系统的本机事件来侦听文件父目录的更改
## watchDirectory
在缺乏递归文件监视功能的系统下如何监视整个目录树的策略。

- fixedPollingInterval，以固定的时间间隔每秒多次检查每个目录的更改
- dynamicPriorityPolling，使用动态队列，其中不经常修改的目录将被较少地检查
- useFsEvents（the default），尝试使用操作系统/文件系统的本机事件进行目录更改
## fallbackPolling
使用文件系统事件时，此选项指定当系统用完本机文件观察器和/或不支持本机文件观察器时使用的轮询策略。

- fixedPollingInterval，以固定的时间间隔每秒多次检查每个文件的更改
- priorityPollingInterval，每秒检查每个文件几次更改，但使用试探法检查某些类型的文件的频率低于其他文件
- dynamicPriorityPolling，使用动态队列，不经常检查修改频率较低的文件
- synchronousWatchDirectory，禁用对目录的延迟监视。node_modules当可能同时发生大量文件更改时（例如从 running更改npm install），延迟监视很有用，但对于一些不太常见的设置，您可能希望使用此标志禁用它
## synchronousWatchDirectory
在本机不支持递归监视的平台上同步调用回调并更新目录监视程序的状态。而不是给出一个小的超时来允许对一个文件进行潜在的多次编辑
## excludeDirectories
可以使用excludeDirectories来大幅减少在watch时进行监视的文件.。这可能是减少 TypeScript 在 Linux 上跟踪的打开文件数量的更有用的方法。
## excludeFiles
可以使用excludeFiles从监视的文件中删除一组特定文件。
```json
{
  "watchOptions": {
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents",
    "fallbackPolling": "dynamicPriority",
    "synchronousWatchDirectory": true,
    "excludeDirectories": ["**/node_modules", "_build"],
    "excludeFiles": ["build/fileWhichChangesOften.ts"]
  }
}
```
# references
大大缩短构建和编辑器交互时间，强制组件之间的逻辑分离，并以新的和改进的方式组织代码。
:::info
引用的项目必须启用新的composite设置
每个引用的属性path可以指向包含文件的目录tsconfig.json，或指向配置文件本身（可以有任何名称）
prepend选项为是否启用前置依赖项的输出
引用的项目必须启用新compilerOptions.composite设置。需要此设置以确保 TypeScript 可以快速确定在哪里可以找到引用项目的输出
打开composite设置后将有以下设置需要注意

- 该rootDir设置，如果未明确设置，则默认为包含该tsconfig文件的目录
- 所有实现文件必须通过模式匹配include或列在files数组中。如果违反此约束，tsc将通知您未指定哪些文件
- compilerOptions.declaration选项必须打开
:::
```json
{
  "references": [
    { "path": "./src/web", "prepend": true },
    { "path": "./src/api" }
  ]
}
```
# typeAcquisition
设置自动引入库类型定义文件(.d.ts)相关配置，该项设置对 JavaScript 项目很重要

- enable，是否禁用自动类型获取
- exclude，额外指定全局依赖类型
- include，禁用 JavaScript 项目中特定模块的类型获取的配置
- disableFilenameBasedTypeAcquisition，是否禁用从DefinitelyTyped 自动下载exclude指定的类型文件（4.1版本以上已发布）
```json
{
  "typeAcquisition": {
    "enable": false,
    "exclude": ["jquery"],
    "include": ["jest", "mocha"],
    "disableFilenameBasedTypeAcquisition": false
  }
}
```
# compilerOptions
配置编译选项
:::warning
⚠下面代码示例仅提供部分参数的写法示例，并不代表该配置合理
:::
```json
{
  "compilerOptions": {
    "declaration": true,
    "diagnostics": true,
    "emitDeclarationOnly": true,
    "incremental": true,
    "tsBuildInfoFile": "./buildFile",
    "inlineSourceMap": true,
    "jsx": "preserve",
    "listFiles": true,
    "module": "ES2015",
    "moduleResolution": "node",
    "newLine": "lf",
    "noEmit": false,
    "noEmitHelpers": false,
    "noEmitOnError": false,
    "noImplicitAny": true,
    "noImplicitThis": false,
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "noLib": false,
    "noResolve": false,
    "noStrictGenericChecks": false,
    "skipDefaultLibCheck": false,
    "skipLibCheck": false,
    "outFile": "./main.js",
    "outDir": "./dist",
    "preserveConstEnums": false,
    "preserveSymlinks": false,
    "preserveValueImports": false,
    "pretty": true,
    "removeComments": false,
    "rootDir": "./",
    "isolatedModules": false,
    "sourceMap": false,
    "suppressExcessPropertyErrors": false,
    "suppressImplicitAnyIndexErrors": false,
    "target": "ES2020",
    "useUnknownInCatchVariables": false,
    "watchDirectory": "useFsEvents",
    "noImplicitReturns": false,
    "noFallthroughCasesInSwitch": false,
    "noImplicitOverride": false,
    "forceConsistentCasingInFileNames": false,
    "generateCpuProfile": "profile.cpuprofile",
    "baseUrl": "./",
    "paths": {
      "jquery": ["node_modules/jquery/dist/jquery.min.js"]
    },
    "plugins": [
      {
        "name": "ts-sql-plugin",
        "command": "psql -c", // optionnal
        "tags": { // optionnal
          "sql": "sql",
          "raw": "raw",
          "cond": "cond",
          "and": "and",
          "or": "or",
          "ins": "ins",
          "upd": "upd",
          "mock": "mock"
        },
        "mock": "0",
        "cost_pattern": "/\\(cost=\\d+\\.?\\d*\\.\\.(\\d+\\.?\\d*)/",
        "error_cost": null, // 100,
        "warn_cost": null, // 50,
        "info_cost": null, // -1,
        "schema_command": "pg" // pg | mysql | custom - read the source
      }
    ],
    "rootDirs": ["src","out"],
    "typeRoots": ["node_modules/@types"],
    "types": [],
    "traceResolution": false,
    "allowJs": true,
    "noErrorTruncation": false,
    "noImplicitUseStrict": false,
    "listEmittedFiles": false,
    "disableSizeLimit": false,
    "lib": ["ES2016", "DOM"],
    "strictNullChecks": false,
    "maxNodeModuleJsDepth": 0,
    "importHelpers": false,
    "importsNotUsedAsValues": "remove",
    "alwaysStrict": true,
    "strict": false,
    "strictBindCallApply": false,
    "downlevelIteration": true,
    "checkJs": true,
    "strictFunctionTypes": false,
    "strictPropertyInitialization": true,
    "esModuleInterop": false,
    "allowUmdGlobalAccess": false,
    "keyofStringsOnly": false,
    "useDefineForClassFields": false,
    "declarationMap": false,
    "resolveJsonModule": false,
    "resolvePackageJsonExports": false,
    "resolvePackageJsonImports": false,
    "assumeChangesOnlyAffectDirectDependencies": false,
    "extendedDiagnostics": false,
    "listFilesOnly": false,
    "disableSourceOfProjectReferenceRedirect": false,
    "disableSolutionSearching": false,
    "verbatimModuleSyntax": false
  }
}
```
| **properties/特性** | **type/类型** | **description/描述** | **default/默认值** | **enum/枚举值** | **other/其他说明** |
| --- | --- | --- | --- | --- | --- |
| allowArbitraryExtensions | boolean | 启用导入具有任何扩展名的文件，前提是存在声明文件 | / | / | / |
| allowImportingTsExtensions | boolean | 允许导入包含TypeScript文件扩展名。需要设置“——modulerresolution bundler”和“——noEmit”或“——emitDeclarationOnly” | / | / | / |
| charset | string | 不再支持 | / | / | / |
| composite | boolean | 启用约束，允许TypeScript项目与项目引用一起使用。 | true | / | / |
| customConditions | array | 在解析导入时，除了特定于解析器的默认值之外还要设置的条件。 | / | / | uniqueItems: true
itemsType: true |
| declaration | boolean | 生成.d.ts文件中的TypeScript和JavaScript文件在你的项目。 | false | / | / |
| declarationDir | ["string", "null"] | 为生成的声明文件指定输出目录。 | / | / | / |
| diagnostics | boolean | 编译后输出编译器性能信息。 | / | / | / |
| disableReferencedProjectLoad | boolean | 减少TypeScript自动加载的项目数量。 | / | / | / |
| noPropertyAccessFromIndexSignature | boolean | 强制对使用索引类型声明的键使用索引访问器 | / | / | / |
| emitBOM | boolean | 在输出文件的开头发出一个UTF-8字节顺序标记(BOM)。 | false | / | / |
| emitDeclarationOnly | boolean | 只输出d.ts文件，不输出JavaScript文件。 | false | / | / |
| exactOptionalPropertyTypes | boolean | 在类型检查时区分未定义和不存在 | false | / | / |
| incremental | boolean | 启用增量编译。需要TypeScript 3.4或更高版本。 | / | / | / |
| tsBuildInfoFile | string | 指定.tsbuildinfo增量编译文件的文件夹。 | .tsbuildinfo | / | / |
| inlineSourceMap | boolean | 在发出的JavaScript中包含sourcemap文件。 | false | / | / |
| inlineSources | boolean | 在发出的JavaScript的sourcemaps文件中包含源代码。 | false | / | / |
| jsx | enum | 指定生成什么JSX代码。 | / | preserve，react，react-jsx，react-jsxdev，react-native | / |
| reactNamespace | string | 指定' createElement '调用的对象。这只适用于针对“react”JSX emit时 | React | / | / |
| jsxFactory | string | 指定针对React JSX emit时使用的JSX工厂函数，例如:的反应。createElement'或'h' | React.createElement | / | / |
| jsxFragmentFactory |  | 当目标是React JSX发射时，指定用于片段的JSX片段e.g. 'React. Fragment'或'Fragment'。 | React.Fragment | / | / |
| jsxImportSource | string | 当使用' JSX: react-jsx '时，指定用于导入JSX工厂函数的模块说明符。 | react | / | / |
| listFiles | boolean | 打印编译期间读取的所有文件。 | false | / | / |
| mapRoot | string | 指定调试器应该定位映射文件的位置，而不是生成的位置。 | / | / | / |
| module | string | 指定生成什么模块代码。 | / | CommonJS,AMD,System,UMD,ES6,ES2015,ES2020,ESNext,None,ES2022,Node16,NodeNext | 不区分大小写 |
| moduleResolution | string | 指定TypeScript如何从给定的模块说明符查找文件。 | classic | classic，node，node16，nodenext，bundler | 不区分大小写 |
| newLine | string | 设置发出文件的换行符。 | / | crlf，lf | 不区分大小写 |
| noEmit | boolean | 从编译中禁用发射文件。 | false | / | / |
| noEmitHelpers | boolean | 在编译输出中禁用生成自定义帮助函数，如' __extends '。 | false | / | / |
| noEmitOnError | boolean | 如果报告任何类型检查错误，禁用发出文件。 | false | / | / |
| noImplicitAny | boolean | 为隐含的' any '类型的表达式和声明启用错误报告。 | / | / | / |
| noImplicitThis | boolean | 当' this '被赋予类型' any '时，启用错误报告。 | / | / | / |
| noUnusedLocals | boolean | 在未读取局部变量时启用错误报告。 | false | / | / |
| noUnusedParameters | boolean | 未读取函数形参时引发错误 | false | / | / |
| noLib | boolean | 禁用包括任何库文件，包括默认的lib.d.ts。 | false | / | / |
| noResolve | boolean | 禁止使用import、require或<reference>扩展TypeScript应该添加到项目中的文件数量。 | false | / | / |
| noStrictGenericChecks | boolean | 禁用严格检查函数类型中的泛型签名。 | false | / | / |
| skipDefaultLibCheck | boolean | 在TypeScript中跳过类型d.ts文件。 | false | / | / |
| skipLibCheck | boolean | 跳过所有d.ts文件。 | false | / | / |
| outFile | string | 指定一个将所有输出捆绑到一个JavaScript文件的文件。如果' declaration '为true，也指定一个捆绑所有.d.ts输出。 | / | / | / |
| outDir | string | 为所有发出的文件指定一个输出文件夹。 | / | / | / |
| preserveConstEnums | boolean | 禁用删除生成代码中的“const enum”声明。 | false | / | / |
| preserveSymlinks | boolean | 禁用将符号链接解析到它们的真实路径。这与node中的相同标志相关联。 | false | / | / |
| preserveValueImports | boolean | 在JavaScript输出中保留未使用的导入值，否则这些值将被删除 | false | / | / |
| preserveWatchOutput | boolean | 禁用在watch模式下清除控制台 | / | / | / |
| pretty | boolean | 在输出中启用颜色和格式，使编译器错误更容易阅读 | true | / | / |
| removeComments | boolean | 禁用保留注释 | false | / | / |
| rootDir | string | 在源文件中指定根文件夹 | / | / | / |
| isolatedModules | boolean | 确保每个文件都可以安全地转译，而不依赖于其他导入。 | false | / | / |
| sourceMap | boolean | 创建JavaScript文件排放源映射文件。 | false | / | / |
| sourceRoot | string | 指定调试器查找参考源代码的根路径。 | / | / | / |
| suppressExcessPropertyErrors | boolean | 在创建对象文字期间禁用报告过多的属性错误。 | false | / | / |
| suppressImplicitAnyIndexErrors | boolean | 索引缺少索引签名的对象时，禁止' noImplicitAny '错误。 | false | / | / |
| stripInternal | boolean | 禁用在JSDoc注释中含有“@internal”的发出声明。 | / | / | / |
| target | string | 为发出的JavaScript设置JavaScript语言版本，并包含兼容的库声明。 | ES3 | ES3,ES5,ES6,ES2015,ES2016,ES2017,ES2018,ES2019,ES2020,ES2021,ES2022,ES2023,ESNext | 不区分大小写 |
| useUnknownInCatchVariables | boolean | 默认catch子句变量为“unknown”而不是“any”。 | false | / | / |
| watch | boolean | 观察输入文件 | / | / | / |
| fallbackPolling | enum | 指定在缺乏递归文件监视功能的系统下监视目录的策略。需要TypeScript 3.8或更高版本。 | / | fixedPollingInterval,priorityPollingInterval,dynamicPriorityPolling,fixedInterval,priorityInterval,dynamicPriority,fixedChunkSize | / |
| watchDirectory | enum | 指定监视单个文件的策略。需要TypeScript 3.8或更高版本。 | useFsEvents | useFsEvents,fixedPollingInterval,dynamicPriorityPolling,fixedChunkSizePolling | / |
| watchFile | enum | 指定监视单个文件的策略。需要TypeScript 3.8或更高版本。 | useFsEvents | fixedPollingInterval,priorityPollingInterval,dynamicPriorityPolling,useFsEvents,useFsEventsOnParentDirectory,fixedChunkSizePolling |  |
| experimentalDecorators | boolean | 为TC39阶段2草稿装饰器提供实验支持。 | / | / | / |
| emitDecoratorMetadata | boolean | 为源文件中的装饰声明发出设计类型的元数据。 | / | / | / |
| allowUnusedLabels | boolean | 禁用未使用标签的错误报告。 | / | / | / |
| noImplicitReturns | boolean | 为函数中未显式返回的代码路径启用错误报告。 | false | / | / |
| noUncheckedIndexedAccess | boolean | 当使用索引访问一个类型时，添加' undefined '。 | / | / | / |
| noFallthroughCasesInSwitch | boolean | 为switch语句中的fallthrough情况启用错误报告。 | false | / | / |
| noImplicitOverride | boolean | 确保派生类中的覆盖成员用覆盖修饰符标记。 | false | / | / |
| allowUnreachableCode | boolean | 禁用不可达代码的错误报告。 | / | / | / |
| forceConsistentCasingInFileNames | boolean | 在导入时确保包装正确。 | false | / | / |
| generateCpuProfile | string | 发出编译器运行的v8 CPU配置文件以进行调试。 | profile.cpuprofile | / | / |
| baseUrl | string | 指定基目录以解析非相对模块名。 | / | / | / |
| paths | object | 指定一组将导入重新映射到其他查找位置的条目。 | / | / | uniqueItems: true
具体写法请查看下方示例配置文件 |
| plugins | array | 指定要包含的语言服务插件列表。 | / | / | 示例 |
| rootDirs | array | 在解析模块时，允许将多个文件夹视为一个文件夹。 | / | / | uniqueItems: true
itemsType: string |
| typeRoots | array | 指定多个文件夹，类似于'./node_modules/@types '。 | / | / | uniqueItems: true
itemsType: string |
| types | array | 指定要包含的类型包名，而不需要在源文件中引用。 | / | / | uniqueItems: true
itemsType: string |
| traceResolution | boolean | 启用名称解析过程的跟踪。要求TypeScript 2.0或更高版本。 | false | / | / |
| allowJs | boolean | 允许JavaScript文件成为程序的一部分。使用' checkJS '选项从这些文件中获取错误。 | false | / | / |
| noErrorTruncation | boolean | 禁用错误消息中的截断类型。 | false | / | / |
| allowSyntheticDefaultImports | boolean | 当模块没有默认导出时，允许“import x from y”。 | / | / | / |
| noImplicitUseStrict | boolean | 禁用在发出的JavaScript文件中添加“use strict”指令。 | false | / | / |
| listEmittedFiles | boolean | 在编译后打印发出的文件的名称。 | false | / | / |
| disableSizeLimit | boolean | 移除TypeScript语言服务器中JavaScript文件源代码大小的20mb上限。 | false | / | / |
| lib | array | 指定一组描述目标运行时环境的绑定库声明文件。 | / | libEnum请在下方查看 | uniqueItems: trueitemsType: string
不区分大小写 |
| moduleDetection | enum | 指定TypeScript如何将文件确定为模块。 | / | auto,legacy,force | / |
| strictNullChecks | boolean | 在类型检查时，考虑“null”和“undefined” | false | / | / |
| maxNodeModuleJsDepth | number | 指定用于从' node_modules '检查JavaScript文件的最大文件夹深度。只适用于' allowJs '。 | 0 | / | / |
| importHelpers | boolean | 允许每个项目从tslib导入一次helper函数，而不是每个文件都包含它们。 | false | / | / |
| importsNotUsedAsValues | enum | 为仅用于类型的导入指定发出/检查行为。 | remove | remove,preserve,error | / |
| alwaysStrict | boolean | 确保'use strict'总是被触发。 | / | / | / |
| strict | boolean | 启用所有严格的类型检查选项。 | false | / | / |
| strictBindCallApply | boolean | 检查' bind '， ' call '和' apply '方法的参数是否与原始函数匹配。 | false | / | / |
| downlevelIteration | boolean | 为迭代提供更兼容，但冗长且性能较差的JavaScript。 | false | / | / |
| checkJs | boolean | 在类型检查的JavaScript文件中启用错误报告。 | false | / | / |
| strictFunctionTypes | boolean | 在分配函数时，检查以确保参数和返回值是子类型兼容的。 | false | / | / |
| strictPropertyInitialization | boolean | 检查在构造函数中声明但未设置的类属性。 | false | / | / |
| esModuleInterop | boolean | 生成额外的JavaScript以简化对导入CommonJS模块的支持。这使' allowsyntheticdefaulultimports '类型兼容。 | false | / | / |
| allowUmdGlobalAccess | boolean | 允许从模块中访问UMD全局变量。 | false | / | / |
| keyofStringsOnly | boolean | 使keyof只返回字符串，而不是字符串、数字或symbols.Legacy选择。 | false | / | / |
| useDefineForClassFields | boolean | 发出符合ECMAScript-standard-compliant的类字段。 | false | / | / |
| declarationMap | boolean | 为d.ts文件创建源map文件。 | false | / | / |
| resolveJsonModule | boolean | 启用importing.json文件 | false | / | / |
| resolvePackageJsonExports | boolean | 使用package. json的“exports”字段。 | false | / | / |
| resolvePackageJsonImports | boolean | 使用package. json的“imports”字段。 | false | / | / |
| assumeChangesOnlyAffectDirectDependencies | boolean | 在“--incremental”和“--watch”中重新编译假设文件中的更改只会直接影响依赖于它的文件。需要TypeScript 3.8或更高版本。 | / | / | / |
| extendedDiagnostics | boolean | 在构建后输出更详细的编译器性能信息。 | false | / | / |
| listFilesOnly | boolean | 打印作为编译一部分的文件的名称，然后停止处理。 | / | / | / |
| disableSourceOfProjectReferenceRedirect | boolean | 在引用复合项目时禁用首选源文件而不是声明文件 | / | / | / |
| disableSolutionSearching | boolean | 在编辑时，选择一个项目脱离多项目参考检查。 | / | / | / |
| verbatimModuleSyntax | boolean | 不要转换或省略任何未标记为纯类型的导入或导出，确保它们以基于'module'设置的输出文件格式写入。 | / | / | / |

:::info
uniqueItems: 子元素的唯一性
itemsType: type为array的属性每一个item的属性，例itemsType: string，该属性为一个string[]
libEnum: [
                      "ES5",
                      "ES6",
                      "ES2015",
                      "ES2015.Collection",
                      "ES2015.Core",
                      "ES2015.Generator",
                      "ES2015.Iterable",
                      "ES2015.Promise",
                      "ES2015.Proxy",
                      "ES2015.Reflect",
                      "ES2015.Symbol.WellKnown",
                      "ES2015.Symbol",
                      "ES2016",
                      "ES2016.Array.Include",
                      "ES2017",
                      "ES2017.Intl",
                      "ES2017.Object",
                      "ES2017.SharedMemory",
                      "ES2017.String",
                      "ES2017.TypedArrays",
                      "ES2018",
                      "ES2018.AsyncGenerator",
                      "ES2018.AsyncIterable",
                      "ES2018.Intl",
                      "ES2018.Promise",
                      "ES2018.Regexp",
                      "ES2019",
                      "ES2019.Array",
                      "ES2019.Intl",
                      "ES2019.Object",
                      "ES2019.String",
                      "ES2019.Symbol",
                      "ES2020",
                      "ES2020.BigInt",
                      "ES2020.Promise",
                      "ES2020.String",
                      "ES2020.Symbol.WellKnown",
                      "ESNext",
                      "ESNext.Array",
                      "ESNext.AsyncIterable",
                      "ESNext.BigInt",
                      "ESNext.Intl",
                      "ESNext.Promise",
                      "ESNext.String",
                      "ESNext.Symbol",
                      "DOM",
                      "DOM.Iterable",
                      "ScriptHost",
                      "WebWorker",
                      "WebWorker.ImportScripts",
                      "Webworker.Iterable",
                      "ES7",
                      "ES2021",
                      "ES2020.SharedMemory",
                      "ES2020.Intl",
                      "ES2021.Promise",
                      "ES2021.String",
                      "ES2021.WeakRef",
                      "ESNext.WeakRef",
                      "es2021.intl",
                      "ES2022",
                      "ES2022.Array",
                      "ES2022.Error",
                      "ES2022.Intl",
                      "ES2022.Object",
                      "ES2022.String"
                    ]
:::
# 参考资料
参考1：[https://www.typescriptlang.org/docs/](https://www.typescriptlang.org/docs/)
参考2：[http://json.schemastore.org/tsconfig](http://json.schemastore.org/tsconfig)
参考3：[https://zhuanlan.zhihu.com/p/285270177](https://zhuanlan.zhihu.com/p/285270177)
