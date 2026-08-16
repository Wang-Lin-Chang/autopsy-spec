# Autopsy Spec — Agent Autopsy Format

> An open format for structured agent/job death reports: manner of death, primary evidence, verdict, and a **standardized death-code taxonomy (D-01~D-09)**. The aviation industry has the NTSB; software has CVEs; agents had nothing — until the autopsy report.
>
> 一个开放的 agent/任务死亡报告格式：死因、主证据、判决，以及**标准化死因代码分类学（D-01~D-09）**。航空业有 NTSB，软件业有 CVE，agent 行业此前一无所有——直到尸检报告。

[![license](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](./LICENSE)

## 为什么存在 / Why this exists

When an agent fails today, the industry response is: read logs, guess, retry. There is no death-cause taxonomy, no standard report, no cross-tool forensics. The reference implementation ([dsh-witness](https://github.com/Wang-Lin-Chang/dsh-witness)) generates `autopsy.json` for every terminal task — this spec is that format, extracted and versioned.

今天 agent 失败，行业的应对是：看日志、猜原因、重试一遍。没有死因分类学、没有标准报告、没有跨工具法医。参考实现（dsh-witness）为每个终态任务生成 `autopsy.json`——本规范就是那个格式，抽取并版本化。

## autopsy.json Schema

```json
{
  "manner_of_death": "exited non-zero",
  "primary_evidence": ["lock", "exit.txt", "out.log"],
  "verdict": "failed",
  "death_code": "D-02",
  "exit_code": 1,
  "at": 1786880000000
}
```

| 字段 | 类型 | 语义 |
|---|---|---|
| manner_of_death | string | 人类可读死因描述 |
| primary_evidence | string[] | 判定所依据的证据文件列表 |
| verdict | "completed" \| "failed" \| "tampered" | 判决 |
| death_code | string | 死因代码（本规范的分类学）|
| exit_code | number | 退出码 |
| at | number | 终态判定时刻（epoch ms）|

## 死因代码分类学 / Death-code taxonomy

| 代码 | 语义 | 触发条件 |
|---|---|---|
| D-01 | completed normally | 退出码 0 |
| D-02 | exited non-zero | 退出码非 0 |
| D-03 | killed via stop request | 存在 stopping 标记 + 终止 |
| D-04 | 保留 | 未在参考实现中使用 |
| D-05 | 保留 | 未在参考实现中使用 |
| D-06 | evidence tampered by task | 任务篡改锁内容/ACL 被识破（EXIT:-999）|
| D-07 | 保留 | 未在参考实现中使用 |
| D-08 | adopted with no exit file | 崩溃发生在退出码写入之前 |
| D-09 | 保留 | 未在参考实现中使用 |

**扩展规则**：新增代码必须附带一个判决实验（触发条件可复现 + 对照组）；保留代码在被占用前不赋予语义。分类学以语义版本演进。

## 参考实现 / Reference implementation

- 生成方：dsh-witness（12 项验收 C-02 实测）；dsh-cross-platform / dsh-macos 的 12 项验收同格式。
- 语义来源：dsh-witness EXP-5（tampered）、EXP-7（尸检报告断言）。

## 诚实边界 / Honest boundaries

- v0.1.0：字段为已实测子集；新增字段须先有实现产出。
- 分类学是**单机任务**的死因分类；多 agent 协作死因（上游污染链）不在 v0 范围。
- 本规范不收集任何数据——案例库是独立可选设施，与规范分离。
