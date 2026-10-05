# OpenTelemetry Metrics Just Got 25× Faster

A 10-minute lightning talk on **bound instruments** in OpenTelemetry.
Observability Summit Europe 2026 · October 5 · Prague
Cijo Thomas, Microsoft

[Official session](https://events.linuxfoundation.org/observability-summit-europe/program/schedule/?id=1263867) · 12:00–12:10 · South Hall 3 A (Floor 3)

### ▶ [View the slides](https://cijothomas.github.io/otel-bound-instruments-slides/)

*(arrow keys to navigate)*

## Run

```sh
npm install
npm run dev      # present
npm run export   # export to PDF
```

## The talk

Most of OpenTelemetry's metric cost is **attribute processing**, not the update.
The whole talk is about where that processing happens:

| | |
|---|---|
| on every call | the problem |
| at startup | the fix — bound instruments |
| in your code | the mistake |

## Numbers

Bound instruments landed in `opentelemetry-rust` behind an experimental feature
flag in [#3421](https://github.com/open-telemetry/opentelemetry-rust/pull/3421).

Benchmark figures come from `docs/metrics.md`
([#3495](https://github.com/open-telemetry/opentelemetry-rust/pull/3495)), with
stress tests in
[#3516](https://github.com/open-telemetry/opentelemetry-rust/pull/3516) —
measured on an Apple M4 Max with 3 attributes.

> `~5 ms` (HTTP request) and `~100 ns` (packet routing) on slide 2 are
> illustrative orders of magnitude, not measurements.

## Presentation material

`slides.md` is the active deck. `slides-with-notes.md` includes presenter notes;
`SCRIPT.md` has click cues, and `TALK-PLAN.md` has the 10-minute pacing plan.

The current deck adapts the official Observability Summit Europe 2026 PowerPoint
template: navy backgrounds, original circle artwork and event logos, Arial text,
and locally bundled JetBrains Mono for code. Original KCD assets remain in the repository as source
material for the earlier delivery.
