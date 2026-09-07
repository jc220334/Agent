# Agent 使用记录

> 本文记录我在完成 Make & CMake Recruit 任务时使用 AI 编程助手（Agent）的过程。

## 你使用了什么 Agent

我使用了 OpenAI 的 **Codex**（桌面版应用里的 AI 编程助手，可以读写文件、执行命令、操作终端）。由于任务需要 Linux 环境，我让 Codex 通过 **WSL（Ubuntu）** 访问 `~/make-cmake-recurit` 仓库来查看和修改文件、运行构建与 git 命令。

## 安装和配置过程

- 工具本身无需额外安装：Codex 桌面应用和 WSL 环境之前已经配置好，仓库里的 `gcc`、`make`、`cmake`（`build-essential` + `cmake`）也都已安装。
- 过程中只做了一项"授权"配置：在 Codex 里批准它调用 WSL 命令（`wsl -e bash -lc ...`），以便它能进入 Linux 环境操作文件、跑 `make`/`cmake`/`git`。
- 我没有把任何代码一次性交给 Agent 让它"代写"，而是在每一步自己先思考，再由 Agent 执行或解释。

## 你交给了它什么任务

1. 修复 `./build/calculator` 报 `No such file or directory` 的问题。
2. 解释并把修改提交、推送到 GitHub 仓库。
3. 清理仓库里的临时文件（`hello*`、拼错的 `CMakeLisits.txt`）。
4. 回答 Task 4 的三道思考题，并写入 `answers/task4.md`。

## Agent 做了哪些修改

1. **定位问题**：发现我写好的完整 CMake 内容被保存成了拼错的文件 `cmake-task/CMakeLisits.txt`，而真正的 `CMakeLists.txt` 里仍然是 TODO 注释，没有定义 `calculator` 目标，所以 CMake 配置出的项目是空的，自然没有 `build/calculator`。
2. **修复 cmake-task**：把 `CMakeLisits.txt` 改名为 `CMakeLists.txt`，然后 `cmake -S . -B build` 重新配置、`cmake --build build` 编译，运行 `./build/calculator` 输出正确。
3. **自检**：在仓库根目录运行 `./check.sh`，Make 和 CMake 两个任务均 PASS。
4. **Git 提交与推送**：
   - 提交 `make-task/Makefile`、`cmake-task/CMakeLists.txt`、`check.sh`（可执行权限位），commit message：`Complete Makefile and CMakeLists.txt tasks`。
   - push 第一次被拒（远程已有 `Update task1.md`），用 `git pull --rebase origin main` 同步后重新 `git push origin main` 成功。
5. **清理**：删除 `hello`、`hello.c`、`hello.i`、`hello.s`、`hello.o` 和 `make-task/CMakeLisits.txt` 等临时文件，工作区恢复干净。
6. **回答思考题**：把 Task 4 三题的解释写入 `answers/task4.md`，提交 `Answer task 4 questions` 并推送。

## git diff 中你看到了什么

- `make-task/Makefile`：把 4 处 TODO 替换成实际的编译/链接命令（`gcc -c src/main.c -o main.o` 等），+5 行。
- `cmake-task/CMakeLists.txt`：加入 `add_executable(calculator ...)` 和 `target_include_directories(calculator ...)`，删掉 TODO 注释，+8 -3。
- `check.sh`：文件模式 `100644 => 100755`（加上可执行权限，0 行内容变化）。
- `answers/task4.md`：把"在这里作答"占位符替换为三题答案，+15 -5。

## 最终结果是否符合你的预期

符合预期：

- `./build/calculator` 能正常运行，输出 `10 + 5 = 15` 和 `10 - 5 = 5`。
- `./check.sh` 自检 2/2 全部通过。
- 所有改动都已提交并推送到 GitHub，`git status` 显示工作区干净、与 `origin/main` 同步。
- 整个过程我也理解了每一处修改的原因（尤其是"CMake 只认 `CMakeLists.txt` 这个文件名"和"增量构建 vs 全量脚本"的区别），不是盲目让 Agent 代做。

---

# 关于 AI Agent 的五问五答

## 1. Agent 为什么能够读取文件、修改代码，而普通聊天 AI 通常不能？

核心差别不是模型更聪明，而是架构不同。两者底层都是大语言模型，但：

- 普通聊天 AI 只有"文本进 -> 文本出"一个通道，没有绑定文件系统或执行环境，代码写得再好也只能输出文字，碰不到磁盘。
- Agent = 模型 + 工具调用（function calling）+ 执行环境，外面套着一个循环：
  1. 模型"看"当前状态（问题、工作目录、之前读到的内容）
  2. 模型决定下一步，输出一个结构化的工具调用，如读取文件、写文件、执行命令
  3. 程序里的运行时真的去执行调用（有操作系统权限），把结果作为文本塞回对话
  4. 模型看到结果再决定下一步，循环直到任务完成

所以 Agent 能改文件，是因为它"长着手"（工具）+ 有人替它"动手"（运行时），而不是模型本身会魔法。

## 2. Tool 在 Agent 中起到了什么作用？

Tool 是 Agent 与真实世界之间的桥梁：

- 扩展能力边界：模型只会生成文本，通过工具才能真正读写文件、跑命令、上网、操作应用。
- 提供"地面真相"、减少幻觉：不"猜"文件内容或编译结果，而是真的去查、去跑，用真实输出校正判断（例如这次直接读文件才发现 `CMakeLisits.txt` 拼写问题）。
- 让 Agent 可执行、可验证：写完代码立刻 `make` / `cmake --build` 验证，错了当场修。
- 权限可控：每个工具可以单独设权限（哪些目录可写、哪些命令需审批），这是安全的基础。

## 3. 为什么项目需要给 Agent 配置一份类似"员工手册"的规则？

因为 Agent 相当于"新员工"，不了解项目潜规则。这类规则文件（如 AGENTS.md、README 约定）相当于入职手册：

- 告诉它本地约定：怎么构建、怎么测试、目录结构、代码风格。
- 划定行为边界：不许删哪些文件、不许提交密钥、改动前先读说明，避免重复踩坑。
- 减少来回解释、节省额度：约定写清楚，Agent 不用每次问。
- 让多次会话/多个 Agent 行为一致。

打个比方：系统提示词是"公司制度"，项目规则文件是"部门手册"，每次具体请求是"工单"。

## 4. Agent 为什么可能"忘记"之前说过的内容？额度怎么计算？

忘记的原因：

- 模型没有真正的记忆，每次回复都把系统提示 + 对话历史 + 读过的文件 + 工具输出重新塞进一个有限的上下文窗口处理。
- 超过窗口上限时，最早/不重要的内容会被截断或压缩丢弃，于是看起来像"失忆"。
- 所以重要信息要落到文件里、提交到 git——那是给 Agent 的"外部记忆"。

额度计算：

- 计费/限流单位是 token（约 1 个汉字约 1~2 个 token）。
- 一次请求消耗约等于输入 token（提示 + 历史 + 文件 + 工具输出）+ 输出 token（生成的回答）。
- 工具输出在下一轮会变成"输入"，所以长日志很烧额度。
- 控制方法：问题问精确、少整段塞大文件、及时把结论写进文件、减少无谓来回。

## 5. 如果一个 Agent 可以随便执行任何终端命令，会有什么风险？

风险很大且真实存在：

- 数据丢失：`rm -rf`、`git reset --hard`、覆盖文件，一步毁掉几天工作。
- 泄露机密：读取 `~/.ssh`、`.env`、`.git-credentials` 并把内容外发；安装恶意依赖。
- 破坏环境：改系统配置、杀进程、装奇怪软件。
- 危害远程仓库：`push --force` 覆盖远端、把密钥提交上去、推到错误仓库。
- 成本失控：命令死循环、无限重试，烧光额度。
- 合规问题：违反许可证、外传私有数据。

所以真实产品都有防护，例如：沙箱（默认只能写工作区）、破坏性操作需人工审批、最小权限、审计日志。会话中批准执行 `wsl` 命令、删除文件前先征求同意，这些"麻烦"正是安全设计。
