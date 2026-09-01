---
theme: seriph
title: "OpenTelemetry Metrics Just Got 25× Faster"
info: |
  KCD SF Bay Area 2026 — Lightning Talk (5 min)
  by Cijo Thomas, Microsoft
layout: cover
class: text-center kcd-brand
fonts:
  provider: none
drawings:
  persist: false
transition: slide-left
mdc: true
---

<img src="/kcd-sfbay-logo.png" class="mx-auto" style="width: 300px; margin-bottom: 2.5rem;" />

# OpenTelemetry Metrics<br/>Just Got **25× Faster**

<div class="mt-8 text-xl">
<strong>Cijo Thomas</strong>, Microsoft
</div>

---
layout: center
class: text-center
---

# The Problem

<div class="text-7xl font-bold mt-8">~50 ns</div>

<div class="text-xl opacity-55 mt-3">the hot path latency to increment a simple OTel Counter<br/>with 3 attributes/labels</div>

<div class="mt-10" style="max-width: 50rem; margin-left:auto; margin-right:auto;">

<div class="flex items-baseline text-sm opacity-40 mb-2" style="white-space: nowrap;">
  <div class="text-left shrink-0" style="width: 15rem;"></div>
  <div class="shrink-0" style="width: 8rem;">the work</div>
  <div class="shrink-0" style="width: 10rem;">OTel adds</div>
</div>

<v-clicks>

<div class="flex items-baseline text-2xl mb-6" style="white-space: nowrap;">
  <div class="text-left shrink-0" style="width: 15rem;">an HTTP request</div>
  <div class="shrink-0 opacity-70" style="width: 8rem;">~5 ms</div>
  <div class="shrink-0 opacity-70" style="width: 10rem;">+0.001%</div>
  <div class="opacity-50">invisible</div>
</div>

<div class="flex items-baseline text-2xl" style="white-space: nowrap;">
  <div class="text-left shrink-0" style="width: 15rem;">routing a packet</div>
  <div class="shrink-0 opacity-70" style="width: 8rem;">~100 ns</div>
  <div class="shrink-0 font-bold" style="width: 10rem;">+50%</div>
  <div class="font-bold">a blocker</div>
</div>

</v-clicks>

</div>

<v-click>

<div class="text-3xl mt-14">I want OTel <em>everywhere</em>.</div>

</v-click>

<v-click>

<div class="text-2xl opacity-70 mt-3">Just not in a flame graph.</div>

</v-click>

---

# Where is the 50 ns spent?

<div class="text-xl opacity-65 mt-1">
The Metric API accepts attributes on <em>every</em> call &mdash; so processing them is mandatory.
</div>

<div class="mt-8" style="max-width: 48rem;">

<div class="text-2xl">process the attributes to find the metric point</div>
<div class="flex items-center gap-5 mt-2">
  <div style="height: 34px; width: 92%; background: #94A3B8; border-radius: 5px;"></div>
  <div class="text-2xl opacity-70 shrink-0">~48 ns</div>
</div>

<v-click>

<div class="text-2xl mt-10">update it</div>
<div class="flex items-center gap-5 mt-2">
  <div style="height: 34px; width: 4%; background: #1C7FD9; border-radius: 5px;"></div>
  <div class="text-2xl font-bold shrink-0">~2 ns</div>
</div>

</v-click>

</div>

<v-click>

<div class="text-5xl mt-12">
on <strong>every call</strong>
</div>

</v-click>

---

# What is the fix?

<div class="grid grid-cols-2 gap-8 mt-8">

<div v-click>

<div class="text-xl opacity-60 mb-2">bound</div>

```rust
// once, at startup
let tcp = counter.bind(&[
    KeyValue::new("protocol", "tcp"),
]);

// hot path
tcp.add(1);
```

<div class="text-lg mt-3">no attributes at the call site</div>

</div>

<div v-click>

<div class="text-xl opacity-60 mb-2">unbound &mdash; what you write today</div>

```rust
// hot path
counter.add(1, &[
    KeyValue::new("protocol", "tcp"),
]);
```

<div class="text-lg opacity-60 mt-3">attributes processed <strong>every call</strong></div>

</div>

</div>

<v-click>

<div class="text-3xl mt-8 text-center">
the expensive attribute processing <span class="opacity-40">&rarr;</span> <strong>once, at startup</strong>
</div>

</v-click>

---

# The Result

<div class="text-xs opacity-50 mb-6">
Rust SDK &middot; Apple M4 Max &middot; 3 attributes
</div>

<div class="text-3xl" style="max-width: 46rem;">

| | before | bound | |
|---|---|---|---|
| `Counter::add` | ~50 ns | **~1.9 ns** | **~26×** |
| `Histogram::record` | ~60 ns | **~6.6 ns** | **~9×** |

</div>

---
layout: center
class: text-center
---

# So is it a magic bullet?

<v-click>

<div class="text-8xl font-bold mt-6">No.</div>

</v-click>

<div class="mt-12 text-3xl" style="max-width: 40rem; margin-left:auto; margin-right:auto;">

<v-clicks>

<div class="mb-5" style="white-space: nowrap;">every attribute value <strong>known upfront</strong></div>
<div class="mb-5" style="white-space: nowrap;">somewhere handy to <strong>keep the bound instrument</strong></div>

</v-clicks>

</div>

<v-click>

<div class="text-2xl opacity-70 mt-8">miss either one &mdash; and it's a no-go</div>

</v-click>

---

# Can I just bind everything?

```rust
// tempting: bind every combination at startup
let handles: HashMap<Attributes, BoundCounter> = build_them_all();

// hot path
handles.get(&attrs)?.add(1);
```

<v-click>

<div class="text-3xl mt-14">
Don't move the problem from OTel <span class="opacity-40">&rarr;</span> <strong>to your app</strong>.
</div>

</v-click>

---
layout: center
class: text-center
---

# Status

<div class="text-5xl mt-8">
Java &nbsp;·&nbsp; C++ &nbsp;·&nbsp; Rust
</div>

<div class="text-xl opacity-60 mt-3">
preview
</div>

<div class="text-2xl opacity-40 mt-8">
more coming
</div>

<v-click>

<div class="text-4xl mt-14">
Break it. Tell us.
</div>

</v-click>

---
layout: center
class: text-center kcd-brand
---

# Key Takeaway

<div class="text-2xl opacity-75 mt-3" style="max-width: 58rem; margin-left:auto; margin-right:auto;">Most of OTel's metric cost is <strong>attribute processing</strong> &mdash; not the update.</div>

<div class="mt-10 mb-10" style="max-width: 36rem; margin-left: auto; margin-right: auto;">

<v-clicks>

<div class="flex items-baseline gap-10 text-4xl mb-7">
  <div class="text-right shrink-0" style="width: 16rem;">on every call</div>
  <div class="text-left opacity-60">the problem</div>
</div>

<div class="flex items-baseline gap-10 text-4xl mb-7">
  <div class="text-right shrink-0" style="width: 16rem;">at startup</div>
  <div class="text-left font-bold" style="white-space: nowrap;">the fix <span class="font-normal opacity-50 text-xl">&nbsp;bound instruments</span></div>
</div>

<div class="flex items-baseline gap-10 text-4xl mb-7">
  <div class="text-right shrink-0" style="width: 16rem;">in your code</div>
  <div class="text-left opacity-60">the mistake</div>
</div>

</v-clicks>

</div>

<v-click>

<div class="text-xl opacity-70">
Flexible by default. Fast when you need it.
</div>

</v-click>

---
layout: center
class: text-center kcd-brand
---

# Thank you

<div class="mx-auto mt-6 mb-5 bg-white" style="width: 300px; padding: 18px; border-radius: 18px;">
  <img src="/linkedin-qr.png" style="width: 100%; display: block;" />
</div>

<div class="text-xl opacity-80">
linkedin.com/in/cijothomas
</div>

<div class="mt-6 text-lg opacity-70">
<strong>Cijo Thomas</strong> · OpenTelemetry maintainer · Microsoft
</div>

