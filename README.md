# commit-lint

[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)](http://commitizen.github.io/cz-cli/)

## 从完全空项目开始一步步搭建

<img src="./assets/gif/git-normalized-commit-demo.gif" alt="git-normalized-commit-demo" width="800" />

### 安装 bun 包管理器

Linux&MacOS

```bash
curl -fsSL https://bun.sh/install | bash
```

Windows

```bash
powershell -c "irm bun.sh/install.ps1 | iex"
```

之所以不使用 npm 反而用 bun 是因为 npm 安装依赖太慢了，而 bun 安装依赖非常快。另外 bun 不只是一个包管理器，它是一个现代的 JavaScript 和 TypeScript 运行时，兼容 Node.js 的 API。

### 项目初始化工作

```bash
# 1. 创建一个新的项目目录
mkdir commit-lint

# 2. 进入项目目录
cd commit-lint

# 3. git 初始化
git init

# 4. 添加远程仓库
git remote add origin https://github.com/your-repo/commit-lint.git

# 5. 初始化一个新的 bun 项目
bun init
```

### 安装 husky 用作 git 钩子

```bash
# 1. 安装 husky 包
bun add -D husky

# 2. 初始化 husky，
#（1）会生成 .husky 目录，在目录内部有 _ 文件夹和 pre-commit 案例脚本文件；
#（2）_ 文件夹里存放了 husky 可以提供的一些钩子的默认脚本， husky-cli 使用文件等；
#（3）自动在 package.json 中添加了
#     "scripts": {
#       "prepare": "husky"
#     }；
bunx husky init
```

### 安装 prettier 用作代码美化

1. 安装 prettier 包

```bash
bun add -D prettier
```

2. 在项目根目录下创建 .prettierrc 文件用于配置 prettier 的规则，在线 playground 地址：https://prettier.io/playground/ ，可以直接点击网站左下角 Copy config Json按钮复制规则并粘贴到 .prettierrc 文件中

如下是默认规则 json 文件内容：

```json
{
  "arrowParens": "always",
  "bracketSameLine": false,
  "bracketSpacing": true,
  "semi": true,
  "experimentalTernaries": false,
  "singleQuote": false,
  "jsxSingleQuote": false,
  "quoteProps": "as-needed",
  "trailingComma": "all",
  "singleAttributePerLine": false,
  "htmlWhitespaceSensitivity": "css",
  "vueIndentScriptAndStyle": false,
  "proseWrap": "preserve",
  "insertPragma": false,
  "printWidth": 80,
  "requirePragma": false,
  "tabWidth": 2,
  "useTabs": false,
  "embeddedLanguageFormatting": "auto"
}
```

3. 在项目根目录接着创建 .prettierignore 文件用于配置 prettier 忽略的文件，我的配置如下：

```json
# prettier doesn't respect newlines between chained methods
# https://github.com/prettier/prettier/issues/7884
**/*.spec.js
**/*.spec.ts
**/dist
# https://github.com/prettier/prettier/issues/5246
**/*.html
```

【提示】

之后你可以使用 `bunx prettier --write .` 命令来格式化代码，使用 `bunx prettier --check .` 命令来检查代码是否符合 prettier 的规则。

### 安装 eslint 用作代码检查

```bash
bun create @eslint/config@latest
```

使用上述命令安装后，会生成 eslint.config.js 配置文件，然后你就可以使用 `bunx eslint --fix` 命令来检查代码并自动修复语法错误。

<img src="./assets/gif/eslint-install.gif" alt="eslint-install" width="600" />

### 安装 lint-staged 用作 git 提交前检查

1. 安装 lint-staged 包

```bash
bun add -D lint-staged
```

2. 在项目根目录下创建 .lintstagedrc 文件用于配置 lint-staged 的规则，我的配置如下：

```json
{
  "*.{js,cjs,mjs,md,ts,vue,json,css}": [
    "bunx prettier --write",
    "bunx eslint --fix"
  ]
}
```

3. 新建 .husky/pre-commit 文件使其在 git 提交之前触发，内容如下：

- 运行 lint-staged 检查代码格式和语法
- 获取暂存的文件列表
- 重新添加文件（如果有修改）
- 允许提交继续

```sh
#!/usr/bin/env sh

# 只运行 lint-staged
bunx lint-staged

# 获取暂存的文件列表
files=$(git diff --cached --name-only --diff-filter=ACM | tr '\n' ' ')

# 重新添加文件（如果有修改）
if [ -n "$files" ]; then
  git add $files
fi

# 允许提交继续
exit 0
```

【提示】

以防无权执行，在 Linux 和 MacOS 系统中建议使用 `chmod +x .husky/pre-commit` 命令赋予执行权限。

**到目前为止，你成功实现了 git 提交前检查代码格式和语法，并自动修复了语法错误。**

### 安装 commitizen 以及相关 Adapter

1. 安装 commitizen 包和相关依赖

```bash
bun add -D commitizen cz-conventional-changelog
```

2. 在 package.json 中添加 json 配置，
   （1）添加 commit-cz 命令可随时使用 `bun run commit-cz` 命令来交互式提交代码
   （2）添加用于配置 commitizen 的规则

```json
{
  "scripts": {
    "commit-cz": "cz"
  },
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
}
```

3. 新建 .husky/prepare-commit-msg 文件使其在 git 提交信息前触发，它在 pre-commit 之后执行，内容如下：

```sh
#!/usr/bin/env sh

# 如果是 merge 或者 rebase 等操作，跳过 commitizen
if [ -n "$2" ]; then
  echo "跳过 commitizen，因为存在 $1 操作"
  exit 0
fi

# 使用 commitizen 进行交互式提交
exec < /dev/tty && bunx cz --hook || true
```

【提示】

以防无权执行，在 Linux 和 MacOS 系统中建议使用 `chmod +x .husky/prepare-commit-msg` 命令赋予执行权限。

### 安装 commitlint 用作 git 提交信息验证

1. 安装 commitlint 包和相关依赖

```bash
bun add -D @commitlint/config-conventional @commitlint/cli
```

2. 在项目根目录下创建 commitlint.config.js 文件用于配置 commitlint 的规则，我的配置如下：

```js
export default {
  extends: ["@commitlint/config-conventional"],

  rules: {
    "type-enum": [
      2,
      "always",
      [
        "feat",
        "fix",
        "docs",
        "style",
        "refactor",
        "perf",
        "test",
        "chore",
        "revert",
        "build",
      ],
    ],
    "type-case": [2, "always", "lower-case"],
    "type-empty": [2, "never"],
    "scope-empty": [0, "never"],
    "subject-empty": [2, "never"],
    "subject-full-stop": [2, "never", "."],
    "header-max-length": [2, "always", 100],
  },
};
```

3. 在 package.json 中添加 commitlint 命令，可随时使用 `bun run commitlint` 命令来检查你上次的 git 提交信息是否符合规范

```json
{
  "scripts": {
    "commitlint": "commitlint --edit"
  }
}
```

4. 新建 .husky/commit-msg 文件使其在 git 提交信息时触发，内容如下：

```sh
#!/usr/bin/env sh

# 使用 commitlint 做检查
bunx --bun commitlint --edit "$1"
```

【提示】

以防无权执行，在 Linux 和 MacOS 系统中建议使用 `chmod +x .husky/commit-msg` 命令赋予执行权限。

## 注意事项

2025年2月19日再次搭建跑通这一过程，所有包均使用当日最新版本，重点的几个包版本号如下：

System: MacOS 15.2

- husky: ^8.0.0
- prettier: ^3.5.1
- eslint: ^9.20.1
- lint-staged: ^15.4.3
- commitizen: ^4.3.1
- cz-conventional-changelog: ^3.3.0
- @commitlint/config-conventional: ^18.4.3
- @commitlint/cli: ^18.4.3

如果你发现这种方法跑不通，可能是系统原因或包版本问题，请自行调整。
