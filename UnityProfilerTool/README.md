# Unity Profiler Tool

Offline analyzer and DodWork/OpenCode skill for Unity Editor Profiler `.data` files.

## Contents

- `analyzer/publish/`: Windows analyzer runtime. Keep every file in this directory together.
- `skill/unity-profiler/`: Standard Skill folder. Its root contains `SKILL.md`.

## Requirements

- Windows x64
- .NET 10 runtime
- DodWork or OpenCode for AI source correlation

## Quick Start

1. Download or clone this repository.
2. Open PowerShell in `analyzer/publish/`.
3. Install the global Skill:

```powershell
.\PerfData.Cli.exe --install-skill --no-pause
```

4. Restart or refresh DodWork/OpenCode so it discovers the new Skill.
5. Analyze a capture:

```text
/UnityProfiler D:/ProfilerCaptures/profile.data
```

You can also provide a directory. When it contains exactly one `.data` file, the Skill uses it automatically:

```text
/UnityProfiler D:/ProfilerCaptures
```

## Analyzer Usage

Run directly when only the base report is needed:

```powershell
.\PerfData.Cli.exe "D:\ProfilerCaptures\profile.data" --no-pause
```

The analyzer writes `profile.report.md` beside the `.data` file. The Skill reads this base report and writes `profile.ai-report.md` beside it.

Useful options:

```text
--frames
--hotspots main
--hotspots render
--hotspots <frame-number>
--top <count>
--report <path.md>
```

Use `--force` with `--install-skill` only when intentionally replacing an existing global installation.

## Analysis Boundaries

- CPU: Main Thread frame times, P50/P95/P99, budget overruns, and marker hotspots.
- Rendering: Render Thread frame times and rendering-related markers. Render Thread waits alone do not prove a GPU bottleneck.
- Memory: the current `.data` parser does not provide Memory Profiler snapshots, GC Alloc byte counts, Native/Managed memory, DrawCalls, or GPU timings.
- Deep Profile: use it to find call directions, then verify absolute costs with a matching non-Deep capture.

## Source Correlation Controls

Use these optional terms with `/UnityProfiler`:

```text
--focus UI|Battle|Network|Resource|<module>
--include-foundation
--include-generated
```

The AI report separates hand-written project candidates from foundation/legacy code and generated, third-party, or engine markers.

## Distribution

For an offline release, zip the repository contents while preserving the directory structure. Do not distribute `PerfData.Cli.exe` by itself because it requires the DLL and runtime configuration files beside it.

## License

MIT
