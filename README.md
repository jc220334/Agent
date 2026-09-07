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
