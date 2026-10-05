---
theme: seriph
title: "OpenTelemetry Metrics Just Got 25× Faster"
info: |
  Observability Summit Europe 2026 — Lightning Talk (10 min)
  October 5, 2026 · Prague · 12:00–12:10
  by Cijo Thomas, Microsoft
layout: cover
class: text-center summit-brand summit-cover
fonts:
  provider: none
drawings:
  persist: false
transition: slide-left
mdc: true
---

<div class="text-lg opacity-75 mb-6">October 5, 2026 · Prague</div>

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

# What does a Metrics SDK do?

<div class="text-xl opacity-65 mt-1">Raw measurements in. Aggregated metric points out.</div>

<div class="metric-flow mt-7">
<div class="metric-card">
<div class="metric-label">Incoming measurements</div>
<div class="text-lg mb-3"><code>fruits_sold</code> · name + color</div>
<div class="metric-measure">+1 &nbsp; apple, red</div>
<div class="metric-measure">+2 &nbsp; banana, yellow</div>
<div class="metric-measure">+3 &nbsp; apple, red</div>
<div class="metric-measure">+1 &nbsp; banana, yellow</div>
<div class="text-lg opacity-50 mt-3">… more sales keep arriving</div>
</div>
<div v-click="1" class="metric-arrow">→</div>
<div v-click="1" class="metric-card metric-card-output">
<div class="metric-label">SDK aggregates → export</div>
<div class="text-lg mb-3"><code>fruits_sold</code></div>
<div class="metric-point"><span>apple, red</span><strong>4</strong></div>
<div class="metric-point"><span>banana, yellow</span><strong>3</strong></div>
<div class="text-lg opacity-60 mt-4">One point per attribute combination</div>
</div>
</div>

<!--
The Metrics SDK takes individual measurements and aggregates them before export. Say we're counting fruits sold, with name and color as attributes. One red apple, two yellow bananas, three more red apples, one more banana: the SDK keeps two points, with totals four and three. That's the beauty of metrics: with fixed cardinality, aggregation storage doesn't grow with the number of sales. There is still work on every measurement. What is that work?
-->

---

# Which point gets the next sale?

<div class="text-xl opacity-65 mt-1">Same attributes, even when they arrive in a different order.</div>

<div class="metric-input mt-6"><code>fruits_sold.add(1, [name="apple", color="red"])</code></div>

<div class="metric-pipeline mt-6">
<div v-click class="metric-stage">
<div class="metric-step">1</div>
<div class="text-2xl font-bold">Sort attributes</div>
<div class="text-lg opacity-65 mt-2">Put keys in a consistent order</div>
<div class="text-sm mt-3"><code>color=red, name=apple</code><br/><span class="opacity-50">one consistent key order</span></div>
</div>
<div class="metric-arrow">→</div>
<div v-click class="metric-stage">
<div class="metric-step">2</div>
<div class="text-2xl font-bold">Find the point</div>
<div class="text-lg opacity-65 mt-2">Hash attributes<br/>Look up the aggregate</div>
<div class="text-lg mt-3"><strong>apple, red → 4</strong></div>
</div>
<div class="metric-arrow">→</div>
<div v-click class="metric-stage metric-card-output">
<div class="metric-step">3</div>
<div class="text-2xl font-bold">Update</div>
<div class="text-lg opacity-65 mt-2">Add the measurement</div>
<div class="text-4xl font-bold mt-4">4 → 5</div>
</div>
</div>

<v-click>

<div class="text-2xl mt-7 text-center">Most of the hot-path work is <strong>finding the point</strong>.</div>

</v-click>

<!--
Now another red apple arrives, with the attributes supplied in a different order. Attribute order must not create a different point. In the Rust SDK path we're discussing, we first sort the attributes into a consistent key order. Then hash those attributes and look up the matching aggregate. Finally, add one to its total. If the combination is new, the SDK needs to create a point, subject to cardinality limits. This diagram shows an existing point. Most of the cost in our benchmark is getting to that point, not incrementing it. Let's look at the numbers.
-->

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

<div class="text-xl opacity-60 mb-2">instead of this</div>

```rust
// hot path
counter.add(1, &[
    KeyValue::new("protocol", "tcp"),
]);
```

<div class="text-lg opacity-60 mt-3">attributes processed <strong>every call</strong></div>

</div>


<div v-click>

<div class="text-xl opacity-60 mb-2">do this instead</div>

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
</div>

<v-click>

<div class="text-3xl mt-8 text-center">
<strong>No attributes on the hot path.</strong>
<div class="text-xl opacity-65 mt-3">Bind once. Update through the stored handle.</div>
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
| `Counter::add` | ~50 ns | **~2 ns** | **~25×** |
| `Histogram::record` | ~60 ns | **~6.6 ns** | **~9×** |

</div>

---
layout: center
class: text-center
---

# So is it a magic bullet?

<div v-click="1" class="text-6xl font-bold mt-6">No.</div>

<div class="mt-10 text-3xl">
<div v-click="2" class="mb-7">Every attribute value <strong>known upfront</strong></div>
<div v-click="3">Somewhere handy to <strong>keep the bound instrument</strong></div>
<div v-click="3" class="text-xl opacity-60 mt-3">A field on the object doing the work.</div>
</div>

<div v-click="4" class="mt-7">
<div class="text-2xl font-bold">Most HTTP request metrics aren’t a natural fit.</div>
<div class="text-xl opacity-65 mt-2">Route, method, and status vary per request.</div>
</div>

<!--
Binding works when the attributes are known ahead of time and the code can keep the handle close to the work. Most HTTP request metrics are not a natural fit: route, method, and status vary per request. Selecting a bound handle by that combination can recreate the lookup. Route templates can be bounded, while raw paths can have high cardinality. The useful exception is code that already holds the right handle. Next, show that with a TCP and UDP router.
-->

---

# Keep the counter on the router

<div class="grid grid-cols-2 gap-8 mt-7">

<div>
<div class="text-xl opacity-65 mb-3">Setup: one handle per router</div>

```rust
let tcp_router = PacketRouter {
    packets: counter.bind(&[
        KeyValue::new("protocol", "tcp"),
    ]),
};

let udp_router = PacketRouter {
    packets: counter.bind(&[
        KeyValue::new("protocol", "udp"),
    ]),
};
```

</div>

<div v-click="1">
<div class="text-xl opacity-65 mb-3">Hot path: update the stored handle</div>

```rust
struct PacketRouter {
    packets: BoundCounter<u64>,
}

impl PacketRouter {
    fn route_packet(&self) {
        self.packets.add(1);
    }
}

tcp_router.route_packet();
udp_router.route_packet();
```

</div>

</div>

<div v-click="2" class="text-2xl mt-8 text-center">Each router already has <strong>the right metric point</strong>.</div>

<!--
At setup, the TCP router binds protocol equals TCP, and the UDP router binds protocol equals UDP. Each stores its own bound counter as a field. Both use the same route_packet method. On the hot path, that method updates its counter field directly. No attributes need to be constructed, and no map needs to be searched for a handle.
-->

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
class: text-center summit-brand
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
class: text-center summit-brand
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

