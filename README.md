## 代码风格

团队开发中，多人开发同一个项目，由于个人编码习惯不同，一个项目中最终的代码风格可能差别很大，所以需要通过工具进行约束来保证代码风格的统一。同时也希望通过工具尽可能的减少低级错误出现，并且能帮助修正，所以有了各种各样的 lint 和 formatter。

Prettier 可以作为[ESLint的插件](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fprettier%2Feslint-plugin-prettier)运行，它允许你用单个命令对代码进行 lint 和格式化。你对自己开发过程所做的任何优化都是本文的一个胜利。Prettier + ESLint 是开发者的天堂。



## 各种工具的实现方式



#### vscode 开箱即用的 code formatter 功能

vscode 提供开箱即用的代码样式化功能（没有 css 格式化功能）

在文件中右键，选择**格式化文档**或者**格式化文档，方法是使用…**，然后选择HTML 语言功能、JavaScript 和 TypeScript 语言功能



#### 用 prettier 对代码进行格式化

首先安装 prettier 插件

prettier 的官方解释是：

- An opinionated code formatter
- Supports many languages
- Integrates with most editors
- Has few options

它能和多种编辑器结合，对多种语言进行 format，所以 css 也不是话下。

由于 vscode 默认有格式化的功能，安装了 prettier 插件后，prettier 也有格式化的功能以会造成冲突（对于html，js），这里编辑器会提示你，可以进行配置。

> HTML 文件有多个格式化程序。选择一个默认格式化程序以继续操作。
>
> JavaScript 文件有多个格式化程序。选择一个默认格式化程序以继续操作。

需要注意的是，vscode 和 prettier 会有很多默认配置，可以通过 `cmd + ,` 快捷键进入配置界面进行管理，所有修改后的结果会保存在 `settings.json` 文件里。

vscode 默认的格式化程序和 prettier 冲突，经过选择后形成配置文件并写入 `settings.json`，如下：

```json
{
    "[html]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[javascript]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[jsonc]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    }
}
```

上面是指针不同类型的文件，分别指定 formatter，当然你也可以一次性指定**所有类型**文件的 formatter，修改后的配置文件 `settings.json` 如下：

```json
{
    "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

经过如上配置， `css` 及其他类型的文件，拥有了通过 prettier 进行格式化的能力。



#### 用 eslint 对 javascript 代码进行校验

经过如上配置，可以对代码进行格式化了，但是如果要想去代码风格进行校验和修复，就要用到 eslint 了，下面分两步将 eslint 功能集成了项目中：

1. 在项目内安装 eslint 及相关的包
2. 给 vscode 安装 eslint 插件

下面分别来说

###### 在项目内安装 eslint 及相关的包

```bash
npm i -D eslint
npx eslint --init
? How would you like to use ESLint? (Use arrow keys)
  To check syntax only 
❯ To check syntax and find problems 
  To check syntax, find problems, and enforce code style 
// 选择To check syntax and find problems
? What type of modules does your project use? (Use arrow keys)
❯ JavaScript modules (import/export) 
  CommonJS (require/exports) 
  None of these 
// 选择JavaScript modules (import/export) 
? Which framework does your project use? (Use arrow keys)
❯ React 
  Vue.js 
  None of these 
// 选择React
? Where does your code run? (Press <space> to select, <a> to toggle all, <i> to invert selection)
❯◉ Browser
 ◯ Node
// 选择Browser
? What format do you want your config file to be in? (Use arrow keys)
❯ JavaScript 
  YAML 
  JSON 
// 选择JavaScript
eslint-plugin-react@latest
? Would you like to install them now with npm? (Y/n) 
// 输入Y 回车
```

经过上面的操作，将 eslint 及相关的包安装到项目里了 `package.json`如下：

```json
{
  ...
  "devDependencies": {
    "eslint": "^5.16.0",
    "eslint-plugin-react": "^7.13.0"
  }
  ...
}

```

项目目录下多了一个 eslint 的配置文件 `.eslintrc.js` :

```js
module.exports = {
    "env": {
        "browser": true,
        "es6": true
    },
    "extends": "eslint:recommended",
    "globals": {
        "Atomics": "readonly",
        "SharedArrayBuffer": "readonly"
    },
    "parserOptions": {
        "ecmaFeatures": {
            "jsx": true
        },
        "ecmaVersion": 2018,
        "sourceType": "module"
    },
    "plugins": [
        "react"
    ],
    "rules": {
    }
};
```

这个配置文件的内容，是通过 `npx eslint --init` 自动生成的，当然你也可以手动配置，所有的选项这里都有中文说明：[Configuring ESLint](http://eslint.cn/docs/user-guide/configuring)

接下来就可以手动执行校验了：

```bash
npx eslint ./src/index.js
```

在执行的时候可能会有包未安装的提示

> Failed to load plugin vue: Cannot find module 'eslint-plugin-react'

手动安装一下就好了，从执行结果来看，index.js 文件有一个错误提示，说明校验程序已经能正常跑进来了。

```bash
1:10  error  'getUserInfo' is defined but never used  no-unused-vars
  2:41  error  Missing semicolon                        semi
  3:25  error  Missing semicolon                        semi

✖ 3 problems (3 errors, 0 warnings)
  2 errors and 0 warnings potentially fixable with the `--fix` option.
```

现在采用的规则是 `eslint:recommended` ，我们的目标是采用 'eslint-config-airbnb-base'，所以再安装相应的包：

```bash
npm i -D eslint-config-airbnb-base eslint-plugin-import
```

然后对 `.eslintrc.js` 进行配置：

```js
module.exports = {
    "env": {
        "browser": true,
        "es6": true
    },
    "extends": "airbnb-base",
    "globals": {
        "Atomics": "readonly",
        "SharedArrayBuffer": "readonly"
    },
    "parserOptions": {
        "ecmaFeatures": {
            "jsx": true
        },
        "ecmaVersion": 2018,
        "sourceType": "module"
    },
    "plugins": [
        "react"
    ],
    "rules": {
        'linebreak-style': ["error", "windows"]
    }
};
```

再进行校验：

```bash
1:10  error  'getUserInfo' is defined but never used              no-unused-vars
  2:7   error  'HelloStr' is never reassigned. Use 'const' instead  prefer-const
  2:41  error  Missing semicolon                                    semi
  3:25  error  Missing semicolon                                    semi

✖ 4 problems (4 errors, 0 warnings)
  3 errors and 0 warnings potentially fixable with the `--fix` option.
```

可以看到明显比之前的错误要多，Aribnb 的规则相对较为严格，可以规避很多低级错误。

这里要重点说一下的是，我们在 `.eslintrc.js` 的 `rules` 里加了 `'linebreak-style': ["error", "windows"]`，是由于不同系统间对换行的处理不同导致的，加这个规则来处理这个问题。[点击查看该配置的更多说明](https://cloud.tencent.com/developer/section/1135633)



###### 给 vscode 安装 eslint 插件，修改vscode配置

走到这里我们已经可以校验 js 文件了，通过校验也发现了很多问题，但在 vscode里并没有错误提示，这就用到了 **vscode** 的另一个插件 `eslint`，安装完插件以后，在 vscode 里可以看到错误提示了。

****

走到这里，我们离成功已经很近啦！

**注意**

**此处vscode的已有配置可能会干扰到eslint的错误提示，出现编辑器中不显示eslint的错误提示的现象**

**解决**

去除standard eslint的**validate**

```json
{
  ...
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    {
      "language": "html",
      "autoFix": true
    }
  ],
  "standard.validate": [
    {
      "language": "javascript",
      "autoFix": true
    },
    {
      "language": "javascriptreact",
      "autoFix": true
    },
    {
      "language": "vue",
      "autoFix": true
    },
    "html"
  ],
  ...
}
```



#### 让 pretter 根据 eslint 校验结果，对代码进行样式化

到目前为上，已经可以对 js 文件进行校验，甚至可以对 js 文件按规则进行修复了：

```bash
1:10  error  'getUserInfo' is defined but never used              no-unused-vars
  2:7   error  'HelloStr' is never reassigned. Use 'const' instead  prefer-const
  2:41  error  Missing semicolon                                    semi
  3:25  error  Missing semicolon                                    semi

✖ 4 problems (4 errors, 0 warnings)
  3 errors and 0 warnings potentially fixable with the `--fix` option.
```

但是如果你用 vscode（如前述，vscode 使用 prettier） 进行修复，发现并没有应用 Airbnb 的规则，这里需要手动配置一下：

1. `CTRL + ,` 打开配置界面

2. 扩展 -> Prettier 里打到 Eslint Integration 并勾选 得到 vscode 的配置文件 `settings.json` 如下

   ```json
   {
       "editor.defaultFormatter": "esbenp.prettier-vscode",
       "prettier.eslintIntegration": true
   }
   ```

   这时再对 js 文件进行格式化，就能按照指定的规则执行了。

###### 如何对Prettier进行配置

一共有三种方式支持对Prettier进行配置：

- 根目录创建`.prettierrc`文件，能够写入YML、JSON的配置格式，并且支持`.yaml/.yml/.json/.js`后缀

- 根目录创建`.prettier.config.js`文件，并对外export一个对象

- 在`package.json`中新建`prettier`属性

下面我们使用`.prettierrc.js`的方式对prettier进行配置，同时讲解下各个配置的作用。

```js
module.exports = {
  "printWidth": 80, //一行的字符数，如果超过会进行换行，默认为80
  "tabWidth": 2, //一个tab代表几个空格数，默认为80
  "useTabs": false, //是否使用tab进行缩进，默认为false，表示用空格进行缩减
  "singleQuote": false, //字符串是否使用单引号，默认为false，使用双引号
  "semi": false, //行位是否使用分号，默认为true
  "trailingComma": "none", //是否使用尾逗号，有三个可选值"<none|es5|all>"
  "bracketSpacing": true, //对象大括号直接是否有空格，默认为true，效果：{ foo: bar }
  "parser": "babylon" //代码的解析引擎，默认为babylon，与babel相同。
}
```



#### VS Code 中实现 Prettier 自动格式化

看这里：[Visual Studio Code中的插件prettier-vscode](https://prettier.io/docs/en/editors.html#visual-studio-code)

装个 [`prettier-vscode`](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fprettier%2Fprettier-vscode) 插件就完事了，非常简单。

实现自动格式化的关键，VS Code 的配置中，`editor.formatOnSave` 改为 `true`。项目中统一使用 `.prettierrc` 配置。从而了统一规范、云端插拔。

###### 问题来了：如果与已存在的插件冲突怎么办

自动格式化的时候，Prettier 和 ESLint的规则冲突了。

![](/Users/jean/SeenrootDocs/tool/prettier/imgs/pic1.png)

**说明**

ESLin t的规则是需要分号，编辑器中此时显示的错误提示也是需要分号。

而 Prettier 的规则配置是不需要分号。此时自动格式化是去除了分号，但是编辑器还是提示错误，难受了~~~

此时我们更改 vscode 的配置项，切换使用规则：**Use 'prettier-eslint' instead of 'prettier'.** 

```js
"prettier.eslintIntegration": true
```

当为true时，会使用 ESLint 的规则来格式化代码，编辑器也不会有分号的错误提示了。但是我们的目标是使用 Prettier 来格式化，解决方法如下：

如果你曾经尝试过将 Prettier 和 ESLint 放在一起运行，那么可能会遇到规则冲突。别担心！你不是在孤军奋战。[eslint-config-prettier插件](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fprettier%2Feslint-config-prettier)可以帮助你解决这个问题。

```bash
npm i -D eslint-config-prettier
```

通过使用eslint-config-prettier配置，能够关闭一些不必要的或者是与prettier冲突的lint选项。这样我们就不会看到一些error同时出现两次。使用的时候需要确保，这个配置在extends的最后一项。

```js
//.eslintrc.js
{
  extends: [
    'airbnb-base', //使用airbnb-base做代码规范
    "prettier",
  ],
}

// vscode的配置项 setting.json
// "prettier.eslintIntegration": true // 不需要该配置
```

###### 配合ESLint检测代码风格

eslint-plugin-prettier插件会调用prettier对你的代码风格进行检查，其原理是先使用prettier对你的代码进行格式化，然后与格式化之前的代码进行对比，如果过出现了不一致，这个地方就会被prettier进行标记。

```bash
npm i -D eslint-plugin-prettier
```

接下来，我们需要在rules中添加，`"prettier/prettier": "error"`，表示被prettier标记的地方抛出错误信息。

```js
//.eslintrc.js
{
  "plugins": ["prettier"],
  "rules": {
    // 此处不要设置其他的eslint的配置了，已prettier的为准
    "prettier/prettier": "error"
  }
}

// setting.json
prettier.eslintIntegration": false
```



## 参考文章

[一步一步，统一项目中的编码规范（vue, vscode, vetur, prettier, eslint）](https://juejin.im/post/5cbfde7c5188250a7d6ddcd1)

[使用ESLint+Prettier来统一前端代码风格](https://juejin.im/post/5b27a326e51d45588a7dac57#heading-5)

[使用 ESLint + Prettier 简化代码 Review 过程](https://juejin.im/post/5cb063bd6fb9a0689b6ed52a)

