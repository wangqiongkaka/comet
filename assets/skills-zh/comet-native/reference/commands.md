# Native 命令与异常参考

正常流程直接执行 Runtime 在 `continuation` 中给出的命令。本文件只解释返回字段和被拒绝的输入。`continuation.disposition` 说明现在应继续、等待用户、处理阻塞还是结束。只有用户明确确认后，才执行含 `--confirmed` 的后续命令。

命令签名和当前参数始终以 CLI 为准：

```text
comet native --help
comet native <command> --help
comet native <group> <command> --help
```

## Runtime 返回的下一步

- `disposition`：说明现在应该继续、等待用户、处理阻塞还是结束；
- `commandArgs` / `commandAlternatives`：Runtime 要求执行的完整命令参数；
- `inputOptions`：这次命令需要填写的字段和 JSON 模板；
- `workspace` / `preparation`：实际工作目录和 change 创建结果；
- `stateVersion` / `loop`：当前状态版本和执行进度；
- `acceptance` / `children` / `readyChildren` / `nextPageArgs`：验收摘要、Supervisor Change 的子项投影、当前可启动子项和下一页查询命令；
- `workspaceFinishResult` / `recoveryArgs`：归档后的工作区收尾结果和恢复命令。

模板中的尖括号表示需要填写的值。`await-user` 表示先等待用户决定，此时不执行推进命令。若 `commandArgs` 为 `null` 且返回了 `commandAlternatives`，先确认用户决定，再执行对应 alternative 的完整参数，并保留 `--expected-state-version` 和 `--expected-action`。命令因状态过期或动作不匹配失败时，重新读取最新 `continuation`。

## 填写 Build 与 Verify 输入

把 `inputOptions.template` 复制到系统临时 JSON 文件，只替换模板要求填写的内容，执行 `continuation.commandArgs` 后删除临时文件。

- `builder-handoff`：保留模板中的 `validation_mode: model`，并提交实现摘要、已处理的验收 ID、开发期检查和已知限制；Runtime 接受后进入由当前模型执行的 Verify。
- `verification-result`：当前模型完成语义核对后，提交验证摘要、逐项验收结论（`acceptance` 数组中每项填写 `id`、`result` 与可追溯的 `reason`）、Runtime 检查计划和已知限制。每项检查填写 `id`、`name`、`executable`、`argv`、`cwdRef`、`timeoutMs` 和 `repeatable`；没有适用命令时使用空数组并在摘要中说明。

Runtime 执行并记录这些检查。缺少验收项、任一验收结论未通过或任一检查未通过时，不得进入 Archive。全部通过后 Runtime 记录 `self-validated` 结果并等待用户决定，用户接受后使用 `--accept-result` 进入 Archive。

已有 change 可能仍在等待独立 Verifier 响应；Runtime 保留旧输入和恢复动作，只用于完成或退回这类尝试。新 Verify 不再启动独立 Verifier。

## 诊断

先运行只读 `doctor`。只有 `doctor` 明确给出修复命令时才执行；锁、跨设备状态和事务仍由 Runtime 管理。
