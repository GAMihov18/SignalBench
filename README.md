# SignalBench

A general-purpose workbench for recording, replaying, and comparing telemetry.

> Planning repository — documentation and Git initialization only. No application, package, solution, database, tests, CI workflow, or runnable demo has been implemented yet. Everything below describes the proposed implementation, unless explicitly marked as verified.

## Goals

Provide a reusable bench for inspecting numeric signals from sensors, experiments, software counters, and recorded datasets. Make timing, units, quality flags, and transformations visible so users can reproduce an analysis.

## Target users

Developers, students, hardware hobbyists, and experimenters working with non-vehicle-specific telemetry streams.

## MVP scope

- Import timestamped CSV files with explicit column, timezone, unit, and missing-value mappings.
- Replay recorded sessions at selectable speed and connect to a simulated live source.
- Plot multiple channels with cursor inspection, range statistics, and visible gaps.
- Apply documented resampling, smoothing, and threshold annotations without overwriting raw samples.
- Export selected raw or transformed ranges with session metadata and transformation provenance.

## Stretch goals

- Serial and MQTT adapters with explicit framing schemas.
- Frequency-domain inspection after sampling assumptions are validated.
- Saved dashboards, source plugins, and side-by-side experiment comparisons.

## Proposed tech stack

C# / .NET 10, Avalonia desktop UI, SQLite session metadata and chunked sample storage, a charting library selected after a rendering/performance spike, xUnit and BenchmarkDotNet.

Select compatible stable dependency versions during milestone 1, record them in project files, and pin the SDK/toolchain after the compatibility spike. No packages or development workloads are installed by this documentation task.

## Architecture

Source adapters emit batches into bounded channels. A session recorder stores raw samples; a transformation pipeline produces derived series with provenance. Query services downsample for the visible chart range, keeping acquisition off the UI thread. The UI depends on application interfaces rather than transport implementations. TimeSeriesKit supplies numeric operations; domain-specific decoders belong outside this repository.

## Proposed folder structure

Only README.md and .gitignore exist initially, alongside Git metadata. The following tree is a plan; create directories when their implementation begins.

```text
SignalBench/
  README.md
  .gitignore
  SignalBench.slnx
  src/SignalBench.Core/
  src/SignalBench.Acquisition/
  src/SignalBench.Storage/
  src/SignalBench.Desktop/
  tests/SignalBench.UnitTests/
  tests/SignalBench.IntegrationTests/
  benchmarks/SignalBench.Benchmarks/
  samples/telemetry/
  docs/formats/
```

## Data models and invariants

Session(Id, Name, StartedAtUtc, Metadata); Channel(Id, SessionId, Name, Unit, ValueKind); Sample(ChannelId, TimestampUtcTicks, Sequence, Value, Quality); Transformation(Id, SourceChannelIds, Kind, Parameters, Version). Quality distinguishes valid, missing, invalid, and disconnected samples. Preserve source timestamps and receipt timestamps when both are available.

## Proposed API and contracts

ITelemetrySource.ReadAsync(CancellationToken) returns IAsyncEnumerable<SampleBatch>; ISessionWriter.AppendAsync(batch, cancellationToken) persists bounded batches; ISeriesQuery.QueryAsync(channelId, timeRange, maxPoints, cancellationToken) returns display-ready data. An import profile specifies timestamp format and unit mapping; never infer ambiguous local times silently.

Version persisted/exported formats from their first implementation. Validate at boundaries and return structured errors rather than leaking internal exceptions. The contracts above remain proposals until backed by code and contract tests.

## Implementation milestones

1. Specify canonical sample schema and import profiles; ship irregular and gapped fixture datasets.
2. Implement CSV import, simulated source, storage, and bounded recording.
3. Build replay and responsive multi-channel charts with cancellation.
4. Add transformation provenance, export round-trip checks, and measured dataset limits.

Each milestone should end with a small demonstrable result, a written acceptance check, and tests for its important failure modes. Work through a vertical slice before expanding the stretch scope.

## Testing and CI plan

Test malformed CSV, duplicate timestamps, timezone ambiguity, out-of-order data, disconnects, backpressure, unit mismatch, and export/import round trips. Benchmark million-sample queries with declared hardware and memory bounds. CI builds core/tests on Windows and Linux; add a desktop launch smoke test and manual visual checks before releases.

CI is planned, not configured. Once a solution exists, add .github/workflows/ci.yml with read-only default permissions, explicit toolchain setup, dependency restore, build, and the checks above. Keep ordinary CI offline from real services and hardware. Store test reports as artifacts; do not publish packages, create releases, or deploy from the first workflow. Record hardware/manual results separately from automated passes.

## Setup and development

### Current checkout

Open this README in a Markdown viewer. Git is initialized locally; no build or run step exists yet. Check the working tree with `git status --short --branch`. Documentation files start untracked; no initial commit or remote is created.

### Prerequisites and first implementation

After scaffolding, install an Avalonia template compatible with the selected SDK, create the desktop project and headless core projects, and add representative telemetry fixtures. Open a simulated session first; hardware adapters remain optional. Keep import examples free of personal or device-identifying information.

Install only the dependencies needed for the selected milestone. Add a checked-in sample configuration with harmless defaults; keep local databases, recordings, generated datasets, credentials, and device identifiers outside version control. Pick a license before public distribution; no license is implied by this plan.

### Future commands

These commands are examples for after the referenced projects/files have been created; they will not build this documentation-only repository.

```powershell
dotnet restore SignalBench.slnx
dotnet test SignalBench.slnx
dotnet run --project src/SignalBench.Desktop
```

## Supporting libraries and boundaries

TimeSeriesKit for resampling/statistics, DataSynth for telemetry fixtures, ConfigLens for source profiles. AutoDiag can share these concepts and libraries but owns vehicle transports, PID decoding, and diagnostic meaning.

Consume stable library versions through local package artifacts initially. Avoid absolute ProjectReference paths into sibling repositories and avoid Git submodules for the MVP. Keep a consumer's first vertical slice runnable using a small local interface or fixture while a supporting package is under development.

## Realistic limitations

No OBD-II, diagnostic trouble code, VIN, ECU, or vehicle connection logic belongs in the MVP. Chart refresh rate is independent of recording rate; rendering every sample is not a scalability strategy. This is an exploratory workbench, not certified measurement or hard real-time software.

Scope and performance targets are hypotheses until measured. Document reproducible dataset sizes, hardware, dependency versions, and failure cases alongside any benchmark claim. Do not present planned capabilities as shipped features.

## Documentation provenance

Prepared 2026-10-23. The previous chat supplied the project names and the AutoDiag platform change, but its detailed assistant responses were unavailable content references. The user confirmed the seven proposed application themes in this session; individual designs here are newly proposed plans, not a recovered transcript.

