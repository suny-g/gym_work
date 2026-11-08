# gym_work
a gym work for the final test.

> C++ 期末课程设计（四人小组）· 单机控制台程序 · SQLite 存储

把**训练记录**变成**训练决策**：不只是记住你练了什么，还会告诉你下次该练多重、为什么是这个重量。

## 这是什么

一个单机程序：管理每周训练计划、逐组记录训练、按规则给出下次重量建议，并把历史数据汇总成容量、PR、肌群分布等统计。数据存在本地 SQLite 文件里，批量导入与统计计算在后台线程执行。

界面刻意用控制台——**把难度集中在数据库、并发与算法上**，而不是图形编程。取舍理由见 [TECH.md](TECH.md) 第 15 节。

## 当前状态

| 模块 | 状态 | 说明 |
|---|---|---|
| `core/` 领域逻辑 | 🟡 部分完成 | 日期、动作、1RM、进阶规则四个模块；**尚未编译验证**（本机还没有编译器） |
| `db/` SQLite | ⬜ 未开始 | 表结构已在 TECH.md 第 5 节定稿 |
| `async/` 并发 | ⬜ 未开始 | 任务队列 + 单写者 DB 线程 |
| `app/` 用例 | ⬜ 未开始 | |
| `ui/` 控制台 | ⬜ 未开始 | |
| 种子数据 | ✅ 完成 | 32 个动作，`data/exercises.csv` |


## 核心设计

| 信念 | 落地方式 |
|---|---|
| 唯一数据源 | PR、容量、1RM 全部实时计算，不写入数据库 |
| 建议必须能解释 | 每条建议重量都附带一句人话理由 |
| 规则可替换 | 进阶规则用策略模式，换规则不动界面与存储 |

## 功能范围

**做**：动作库、周计划、逐组记录、进阶建议、1RM 与 PR、统计、SQLite 持久化、并发批量导入、CSV 导出。

**不做**：图形界面、账号体系、多设备同步、手机端、AI 教练。

完整清单与验收标准见 [TECH.md](TECH.md) 第 1 节。

## 技术栈

| 层 | 选择 |
|---|---|
| 语言 | C++23（低风险子集，见 TECH.md 第 11 节） |
| 构建 | CMake ≥ 3.20 + Ninja |
| 编译器 | MinGW-w64 GCC 13 及以上，或 MSVC 2022 |
| 存储 | SQLite3 |
| 界面 | 控制台（Windows Console API） |
| 数据交换 | CSV |
| 测试 | 自写断言 + CTest |

除 SQLite 外不引入第三方库。

## 目录结构

```
src/
  core/           纯逻辑，零依赖：model / date / onerm / progression / stats / csv
  db/             SQLite 连接、DDL、DAO、事务
  async/          任务队列、DB 线程、线程池
  app/            用例编排
  ui/             控制台渲染与各页面
data/
  exercises.csv   种子动作库（32 条，主动肌/协同肌/器械/类型）
tests/            单元测试与并发压力测试
TECH.md           技术方案：架构、数据库、并发模型、技术清单
```

`db/`、`async/`、`app/`、`ui/`、`tests/` 目录尚未创建。

## 构建

```powershell
$env:PATH = "C:\msys64\ucrt64\bin;$env:PATH"
cmake -S . -B build -G Ninja
cmake --build build
```

一次装齐依赖（MSYS2）：

```powershell
C:\msys64\usr\bin\bash.exe -lc "pacman -S --needed mingw-w64-ucrt-x86_64-gcc mingw-w64-ucrt-x86_64-cmake mingw-w64-ucrt-x86_64-ninja mingw-w64-ucrt-x86_64-sqlite3"
```

当前产物：`build/libgymlog_core.a`。可执行程序在 W1 末期接入。
