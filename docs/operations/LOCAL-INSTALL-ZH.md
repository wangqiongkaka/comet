# 从本地代码安装 Comet

本文说明如何用仓库中的代码运行或安装 Comet，用于验证尚未发布的改动。普通用户请直接使用
`npm install -g @rpamis/comet`，不需要本文的任何步骤。

## 前置构建

三种方式都依赖构建产物：`bin/comet.js` 加载 `dist/`，Skill 启动器加载 `assets/skills*/` 下生成的
`.mjs`。仓库刚 clone 或刚切分支时都要先构建一次。

```bash
cd <仓库根目录>
pnpm install
pnpm build
```

只改了某一个 runtime 时可以只跑对应的增量构建，速度更快：

```bash
pnpm build:native-runtime    # 改了 domains/comet-native/
pnpm build:classic-runtime   # 改了 domains/comet-classic/
pnpm build:entry-runtime     # 改了 domains/comet-entry/
```

构建完成后可以立即确认版本：

```bash
node bin/comet.js --version
```

## 方式一：直接运行本地 CLI

不安装、不改动任何全局状态，直接执行仓库中的入口文件。

```bash
node <仓库根目录>/bin/comet.js --version
node <仓库根目录>/bin/comet.js native status --json
```

- 适用场景：验证 CLI 自身的行为，例如命令输出、退出码、JSON 结构。
- 限制：Skill 样板中的命令通过 PATH 解析 `comet`，因此这种方式无法验证完整的 Skill 执行链路。

## 方式二：打包 tarball 后全局安装

与真实用户的安装过程最接近，会同时验证 `package.json` 的 `files` 白名单和 postinstall 脚本。

```bash
cd <仓库根目录>
pnpm build
npm pack --pack-destination /tmp
npm install -g /tmp/rpamis-comet-<版本号>.tgz
comet --version
```

- 适用场景：发布前的最终确认，或排查"本地能跑、装完不能跑"的打包缺失问题。
- 注意：会覆盖当前的全局 Comet。恢复已发布版本：

```bash
npm install -g @rpamis/comet
```

如果只是想验证打包和安装本身是否正确，不希望影响全局环境，直接运行仓库自带的端到端脚本。它在临时目录中
完成 pack、install 和冒烟验证，不改动全局安装：

```bash
pnpm test:package-e2e
```

## 方式三：npm link

把全局 `comet` 软链到当前仓库，适合需要反复修改并立即验证的开发过程。

```bash
cd <仓库根目录>
pnpm build
npm link
comet --version
```

链接建立后，每次改动只需重新构建即可生效，不必重复安装：

```bash
pnpm build:native-runtime && comet native status --json
```

结束后还原为已发布版本：

```bash
npm unlink -g @rpamis/comet
npm install -g @rpamis/comet
```

- 适用场景：开发期间反复验证改动。
- 注意：在 unlink 之前，全局 `comet` 始终指向工作区，切换分支会直接改变全局命令的行为。

## 如何选择

| 目标                           | 推荐方式                |
| ------------------------------ | ----------------------- |
| 只验证 CLI 输出或退出码        | 方式一                  |
| 开发期间反复修改并验证         | 方式三                  |
| 验证完整安装结果与发布资产     | 方式二                  |
| 验证打包正确性但不影响全局环境 | `pnpm test:package-e2e` |
