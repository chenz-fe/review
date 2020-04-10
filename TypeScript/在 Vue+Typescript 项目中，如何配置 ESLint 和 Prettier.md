# 在 Vue+Typescript 项目中，如何配置 ESLint 和 Prettier

在接手一些老项目的时候，最让人头疼的就是代码格式化不统一的问题，控制台满屏 `eslint ` 警告，简直是要逼死强迫症的节奏。

如果是开启一个新的Vue项目，我一定会选用 `Vue Cli + TypeScript + ESLint + Prettier` 的组合，这个配置有以下好处：

- TypeScript 使我们的代码更规范
- ESLint + Prettier 可以统一团队代码格式化，并且保存时自动进行格式修正



### 代码地址

[vue-ts](https://github.com/dora-zc/vue-ts)

### 创建新项目

```bash
vue create vue-ts
```

<img src="/Users/zengchen/Desktop/2020/review/TS/assets/image-20200410153055024.png" alt="image-20200410153055024" style="width: 50%;" />

<img src="/Users/zengchen/Desktop/2020/review/TS/assets/image-20200410153156764.png" alt="image-20200410153156764" style="width:50%;" />

<img src="/Users/zengchen/Desktop/2020/review/TS/assets/image-20200410153304860.png" alt="image-20200410153304860" style="width:50%;" />

注意这里需要勾选 ESLint + Prettier，我试过勾选 TSLint，但是发现 TSLint 无法实现代码自动格式化，只能提供很多很多的警告信息。

后面要选择 Lint on save 和 In decicated config files

<img src="/Users/zengchen/Desktop/2020/review/TS/assets/image-20200410153817662.png" alt="image-20200410153817662" style="width:50%;" />

### 修改.eslintrc.js

规则可以自己配置，我主要修改了extends，将`@vue/prettier`位置提前了

- `@typescript-eslint/parser`：`ESLint`的解析器，用于解析`TypeScript`，从而检查和规范`TypeScript`代码
- `@vue/prettier/@typescript-eslint`：使得@typescript-eslint中的样式规范失效，遵循prettier中的样式规范

```js
module.exports = {
  root: true,
  env: {
    node: true
  },
  extends: [
    'plugin:vue/essential',
    '@vue/prettier',
    'eslint:recommended',
    '@vue/typescript/recommended',
    '@vue/prettier/@typescript-eslint'
  ],
  parserOptions: {
    ecmaVersion: 2020,
    parser: "@typescript-eslint/parser"
  },
  rules: {
    'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    "no-unused-vars": "off",
    "@typescript-eslint/no-unused-vars": "off",
    "@typescript-eslint/no-explicit-any": "off",
    "prefer-const": 'off'
  }
};
```

### 在根目录添加 .prettierrc.js

```js
module.exports = {
  printWidth: 80,
  tabWidth: 2,
  useTabs: false,
  singleQuote: true,
  semi: true,
  trailingComma: 'none',
  bracketSpacing: true,
  jsxBracketSameLine: true
};
```

然后启动/重启项目，就可以在保存代码时按照你项目里配置的格式化规则进行自动格式化了。

### 附上我的 vscode 配置

```js
{
	"workbench.iconTheme": "material-icon-theme",
	"workbench.colorTheme": "Arc Dark",
	"files.associations": {
		"*.cjson": "jsonc",
		"*.wxss": "css",
		"*.wxs": "javascript"
	},
	"emmet.includeLanguages": {
		"wxml": "html"
	},
	"minapp-vscode.disableAutoConfig": true,
	"editor.fontSize": 16,
	"editor.minimap.enabled": false,
	"[html]": {
		"editor.defaultFormatter": "esbenp.prettier-vscode"
	},
	"git.autofetch": true,
	"[json]": {
		"editor.defaultFormatter": "esbenp.prettier-vscode"
	},
	"prettier.semi": false,
	"prettier.jsxSingleQuote": true,
	"prettier.singleQuote": true,
	"prettier.proseWrap": "never",
	"prettier.printWidth": 180,
	"html.format.maxPreserveNewLines": 1,
	"html.format.wrapAttributes": "force",
	"prettier.eslintIntegration": true,
	"prettier.jsxBracketSameLine": true,
	"[javascript]": {
		"editor.defaultFormatter": "vscode.typescript-language-features"
	},
	"editor.tabSize": 2,
	"prettier.useTabs": true,
	"eslint.options": {
		"extensions": [".js", ".vue", ".ts", ".tsx"]
	},
	"[jsonc]": {
		"editor.defaultFormatter": "esbenp.prettier-vscode"
	},
	"eslint.run": "onSave",
	"javascript.validate.enable": false,
	"editor.wordWrap": "wordWrapColumn",
	"files.autoSave": "afterDelay",
	"terminal.external.osxExec": "/bin/zsh",
	"terminal.integrated.shell.osx": "/bin/zsh",
	"terminal.integrated.fontFamily": "Meslo LG M for Powerline",
	"[vue]": {
		"editor.defaultFormatter": "esbenp.prettier-vscode"
	},
	"git.enableSmartCommit": true,
	"leetcode.endpoint": "leetcode-cn",
	"leetcode.hint.configWebviewMarkdown": false,
	"leetcode.workspaceFolder": "/Users/zengchen/Desktop/Github/leetcode-practice",
	"leetcode.defaultLanguage": "javascript",
	"editor.codeActionsOnSave": {
		"source.fixAll.eslint": true
	},
	"leetcode.hint.commandShortcut": false,
	"leetcode.hint.commentDescription": false,
	"stylusSupremacy.insertColons": false,
	"stylusSupremacy.insertSemicolons": false,
	"stylusSupremacy.insertBraces": false,
	"stylusSupremacy.insertNewLineAroundBlocks": false,
	"[typescript]": {
		"editor.defaultFormatter": "vscode.typescript-language-features"
	},
	"tslint.run": "onSave",
	"eslint.format.enable": true,
	"vetur.format.defaultFormatter.html": "prettyhtml",
	"vetur.format.defaultFormatter.js": "prettier",
	"vetur.format.defaultFormatterOptions": {
		"prettier": {
			"singleQuote": true,
			"semi": false,
			"eslintIntegration": true
		}
	},
	"javascript.implicitProjectConfig.checkJs": true,
}
```

