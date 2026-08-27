---
name: unity-profiler
description: Analyze Unity Profiler .data files with PerfData.Cli, producing CPU and Render Thread reports plus source-aware AI findings. Trigger with /UnityProfiler, a .data path, or Unity performance analysis requests.
license: MIT
compatibility: Windows; requires PerfData.Cli.exe from this distribution.
metadata:
  author: dev-team
  version: "1.1"
  requires:
    bins:
      - PerfData.Cli.exe
---

# UnityProfiler

## Goal

Turn Unity Profiler `.data` captures into a detailed offline CPU and Render Thread report, then correlate only relevant hotspots to project source. Do not upload capture data or modify the input `.data` file.

## Input and Output

Support these forms:

```text
/UnityProfiler G:/temp/profile.data
/UnityProfiler G:/temp
/UnityProfiler latest G:/temp
/UnityProfiler G:/temp/profile.data --focus UI
/UnityProfiler G:/temp/profile.data --include-foundation
```

- A `.data` path is analyzed directly.
- For a directory or `latest <directory>`, enumerate direct `.data` files by modification time. Select automatically only when there is exactly one candidate; otherwise ask the user to select one. Never scan an entire drive.
- Invoke `PerfData.Cli.exe` from the command name on PATH. If it cannot be found, check the directory beside this skill and then ask for the executable path. Do not write configuration during a normal analysis.
- By default, pass `--report "<DATA_PATH>.report.md" --no-pause`; the base report and AI report remain beside the capture for easy handoff. If the user supplies an output path, honor it.
- Reuse a matching existing `.report.md` only when the report timestamp is newer than the `.data` timestamp; otherwise regenerate it.
- The AI report is `<DATA_PATH>.ai-report.md` and must not overwrite the base report.

## Analyzer Invocation

```powershell
& "<PerfData.Cli.exe>" "<DATA_PATH>" --report "<DATA_PATH>.report.md" --no-pause
```

The CLI can install this global skill explicitly:

```powershell
& "<PerfData.Cli.exe>" --install-skill --no-pause
```

Use `--force` only when the user explicitly requests an upgrade of an existing global skill. This writes to `%USERPROFILE%\.config\opencode\skills\unity-profiler\SKILL.md` and does not alter project files.

## Analysis Scope

- Treat CPU Usage as Main Thread frame timings and marker hotspots. Report average, max, P50, P95, P99, and budget overruns.
- Treat Rendering as Render Thread timings and rendering-related markers. Do not declare a GPU bottleneck based only on Render Thread waits.
- The report must state that Memory Profiler snapshots, GC Alloc byte counts, DrawCalls, memory curves, and GPU timings are unavailable unless the analyzer directly parsed them. Do not invent values.
- Deep Profile inflates absolute method and allocation costs. Use it to find call directions, then request a matching non-Deep recording for final timing decisions.
- Do not treat a recording's final frame as a game stall without checking for Editor Flush work.

## Source Correlation and Findings

Classify every candidate finding into exactly one group:

1. **New project code candidates**: hand-written source under the current project or declared source roots, with Profiler evidence and a concrete code location. These are the only findings eligible for the default priority list.
2. **Project foundation / legacy code**: shared framework, protocol infrastructure, resource framework, or broad existing platform code. Keep these in a separate section with ownership and scope notes; do not present them as newly introduced developer defects.
3. **Generated / third-party / engine code**: `_Gen`, generated protocol/config code, Unity, .NET, plugins, or vendor libraries. Preserve facts only; explain that they require a call-site correlation before actionable project work exists.

For every finding, include profiler evidence, code evidence where available, causal explanation, confidence, minimal direction, and a normal-profile verification step. Never label a marker-only guess as a confirmed defect.

When a line of investigation needs business context, end the response with an opt-in prompt such as:

```text
可指定 `--focus UI|Battle|Network|Resource|模块名` 缩小源码关联范围；
可指定 `--include-foundation` 纳入项目底层/遗留代码；
可指定 `--include-generated` 仅查看生成或第三方调用链事实。
```

## Final Response

Use Chinese and separate facts, inferences, and verification actions. State the base-report and AI-report paths, the count of new project code candidates, foundation/legacy findings, and generated/third-party findings.
