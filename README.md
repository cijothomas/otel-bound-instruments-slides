# OpenTelemetry Metrics Just Got 25× Faster

A 5-minute lightning talk on **bound instruments** in OpenTelemetry.
KCD SF Bay Area 2026 · Cijo Thomas, Microsoft

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

## Branding

KCD SF Bay Area 2026 palette and logos in `style.css` and `public/`.
The template font is Clarity City Next; this deck substitutes Poppins.
