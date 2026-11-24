---
layout: default
class: text-center
transition: slide-left
---

<div class="h-full flex flex-col items-center justify-center">
  <div class="text-[7rem] leading-none font-black bg-gradient-to-br from-indigo-600 to-violet-500 bg-clip-text text-transparent">04</div>
  <div class="text-3xl font-bold text-slate-900 mt-2">Campaign lineage DAG</div>
  <div class="text-base text-slate-500 mt-3 max-w-xl">Why raw papers confuse LMs — and how we build the graph</div>
</div>

<!--
Speaker Notes:
So far we focused on representing a single engineering step.
In this section, we look at why extracting those steps directly from raw paper prose is hard for language models, how we reconstruct the lineage graph deterministically without an LLM, and what the full campaign DAG gives us.
-->

---
transition: slide-left
---

# Raw papers are hard for LMs to navigate

Papers are written as a narrative for humans, not as a chronological log of experiments.

<div class="grid grid-cols-2 gap-6 mt-5 text-xs text-left">
  <div>
    <div class="font-bold text-sm text-slate-900 mb-2">How a paper is actually written</div>
    <div class="p-4 rounded-lg bg-slate-50 border border-slate-200 font-mono text-[11px] text-slate-700 space-y-2.5 leading-relaxed">
      <div><span class="text-slate-400">Abstract:</span> spoils the final winner (<code>v2.0</code>, 3.2 nM) upfront</div>
      <div><span class="text-slate-400">Results §2:</span> jumps straight to winning edits (<code>V24F</code>, <code>S28K</code>)</div>
      <div><span class="text-slate-400">Results §4:</span> mentions an earlier dead-end (<code>V24W</code> clash) in passing</div>
      <div><span class="text-slate-400">Supplement S3:</span> buries 12 failed variants and full sequences</div>
    </div>
  </div>
  <div>
    <div class="font-bold text-sm text-slate-900 mb-2">Why an LM gets lost</div>
    <div class="divide-y divide-slate-200 border-t border-b border-slate-200">
      <div class="py-3">
        <div class="font-semibold text-slate-900">Non-linear order</div>
        <div class="text-slate-600 mt-0.5">Hard to tell which failure happened <i>before</i> which design decision.</div>
      </div>
      <div class="py-3">
        <div class="font-semibold text-slate-900">Built-in answer leakage</div>
        <div class="text-slate-600 mt-0.5">Final outcomes sit right next to early hypotheses.</div>
      </div>
      <div class="py-3">
        <div class="font-semibold text-slate-900">Branching &amp; alias drift</div>
        <div class="text-slate-600 mt-0.5">The same variant appears as <code>clone 4</code>, <code>v1.2</code>, or <code>V24F</code> across parallel paths.</div>
      </div>
    </div>
  </div>
</div>

<div class="mt-4 text-xs text-slate-500 text-left">
  Before extracting scientific reasoning, we need the backbone: which variant came from which.
</div>

<!--
Speaker Notes:
A key practical obstacle is that scientific papers are optimized for storytelling, not chronological tracking.
The abstract spoils the winner, failed attempts appear pages later or in supplements, and variant names drift across sections.
If we ask an LM to build the experiment tree directly from prose, it easily confuses order and leaks outcomes.
-->

---
transition: slide-left
---

# Grounding every claim in evidence

Mandatory sentence-level citations — the LLM must point to an exact span for every fact it extracts.

<div class="mt-4 divide-y divide-slate-200 border-t border-b border-slate-200 text-left">
  <div class="grid grid-cols-12 gap-5 py-3 items-center">
    <div class="col-span-5">
      <div class="font-semibold text-sm text-slate-900">1. Number the paper <span class="font-mono text-[11px] font-normal text-slate-500">(script, no LLM)</span></div>
      <div class="text-xs text-slate-600 mt-1">Split prose, tables, and captions into indexed spans <code>[E1..En]</code>.</div>
    </div>
    <div class="col-span-7">
      <pre class="m-0 px-3 py-2 rounded bg-slate-50 border border-slate-200 font-mono text-[10.5px] text-slate-700 leading-snug whitespace-pre-wrap">[E12] "Minibinder_v1.2 bound patch 365–380 at 450 nM; cavity near Tyr369."
[E14] "V24W clashed with helix-2 and abolished binding."
[E19] "Table 1: Minibinder_v2.0 (V24F+S28K) | Kd = 3.2 nM"</pre>
    </div>
  </div>
  <div class="grid grid-cols-12 gap-5 py-3 items-center">
    <div class="col-span-5">
      <div class="font-semibold text-sm text-slate-900">2. Extract claim + pointer <span class="font-mono text-[11px] font-normal text-slate-500">(LLM)</span></div>
      <div class="text-xs text-slate-600 mt-1">Every extracted field or edge must cite the exact <code>[E#]</code> it came from.</div>
    </div>
    <div class="col-span-7">
      <pre class="m-0 px-3 py-2 rounded bg-slate-50 border border-slate-200 font-mono text-[10.5px] text-slate-700 leading-snug whitespace-pre-wrap">diagnosis: "cavity near Tyr369"  → [E12]
edge(WT → V24W): clash            → [E14]
outcome: Kd = 3.2 nM              → [E19]</pre>
    </div>
  </div>
  <div class="grid grid-cols-12 gap-5 py-3 items-center">
    <div class="col-span-5">
      <div class="font-semibold text-sm text-slate-900">3. Verify 1-to-1 <span class="font-mono text-[11px] font-normal text-slate-500">(check / filter)</span></div>
      <div class="text-xs text-slate-600 mt-1">Check one claim against one span; unsupported claims are dropped or marked inferred.</div>
    </div>
    <div class="col-span-7">
      <pre class="m-0 px-3 py-2 rounded bg-slate-50 border border-slate-200 font-mono text-[10.5px] text-slate-700 leading-snug whitespace-pre-wrap">does [E14] support "V24W clashed"? → ✓ (stated)
no direct [E#] for edge            → inferred / drop</pre>
    </div>
  </div>
</div>

<!--
Speaker Notes:
Here is how evidence-based extraction works in three simple steps:
1. A deterministic script—no LLM—splits the paper, tables, and captions into numbered spans [E1..En].
2. The LLM extracts each field or edge as a claim paired with its exact [E#] pointer.
3. Verification becomes a fast 1-to-1 check: one claim against one sentence. Anything without direct textual support is either dropped or explicitly flagged as inferred rather than stated.
-->

---
transition: slide-left
---

# Building a candidate parent graph

Sequence diffs propose candidate edges; paper prose confirms what actually happened.

<div class="mt-5 divide-y divide-slate-200 border-t border-b border-slate-200 text-left">
  <div class="grid grid-cols-12 gap-6 py-3.5 items-center">
    <div class="col-span-6">
      <div class="font-semibold text-sm text-slate-900">1. Collect variants</div>
      <div class="text-xs text-slate-600 mt-1">Pull sequences or mutation strings from tables, FASTA, and supplements.</div>
    </div>
    <div class="col-span-6">
      <pre class="m-0 px-3 py-2 rounded bg-slate-50 border border-slate-200 font-mono text-[11px] text-slate-700 leading-relaxed">WT, V24W, V24F, S28K, V24F+S28K</pre>
    </div>
  </div>
  <div class="grid grid-cols-12 gap-6 py-3.5 items-center">
    <div class="col-span-6">
      <div class="font-semibold text-sm text-slate-900">2. Sequence diff → candidate edges</div>
      <div class="text-xs text-slate-600 mt-1">Minimal mutation supersets propose candidate parent and merge links — without assuming subset inclusion equals true history.</div>
    </div>
    <div class="col-span-6">
      <pre class="m-0 px-3 py-2 rounded bg-slate-50 border border-slate-200 font-mono text-[11px] text-slate-700 leading-snug">WT ─▶ {V24F} ─┐
              ├─▶ {V24F, S28K}
WT ─▶ {S28K} ─┘
edge types: stated | inferred | unresolved</pre>
    </div>
  </div>
  <div class="grid grid-cols-12 gap-6 py-3.5 items-center">
    <div class="col-span-6">
      <div class="font-semibold text-sm text-slate-900">3. Ground with prose &amp; rounds</div>
      <div class="text-xs text-slate-600 mt-1">An LLM checks paper text to confirm edges, recover round order, and attach diagnosis, rationale, and failure causes.</div>
    </div>
    <div class="col-span-6">
      <pre class="m-0 px-3 py-2 rounded bg-slate-50 border border-slate-200 font-mono text-[11px] text-slate-700 leading-relaxed">WT → V24W [stated, round 1] ⇐ [E14]</pre>
    </div>
  </div>
</div>

<div class="mt-3 text-xs text-slate-600 text-left">
  Explicit uncertainty: sequence diffs propose the backbone; paper text separates confirmed lineage from inferred links.
</div>

<!--
Speaker Notes:
We do not pretend mutation subset inclusion alone proves the true experimental history, nor do we ask an LLM to invent the graph from scratch.
Instead, we collect variants and use sequence diffs to propose a candidate parent graph.
Then an LLM grounds those candidate edges in the paper text—confirming stated parents, flagging uncertain links, recovering round order, and attaching the scientific rationale.
-->

---
transition: slide-left
---

# What the campaign DAG unlocks

Nodes are variants; each edge is one structured step from Section 03.

<div class="grid grid-cols-12 gap-5 mt-3 items-center">
<div class="col-span-7 bg-slate-50 p-3 rounded-xl border border-slate-200 overflow-hidden flex justify-center">

```mermaid {scale: 0.62}
flowchart TD
    v0["v0: WT scaffold"]
    v1a["v1a: V24W (clash ❌)"]
    v1b["v1b: V24F (15 nM ✅)"]
    v1c["v1c: S28K (Tm +6°C ✅)"]
    v2a["v2a: +Loop L1 (high aff., insoluble ⚠️)"]
    v2b["v2b: V24F+S28K (3.2 nM ✅)"]
    v2c["v2c: V24F+A30T (soluble ✅)"]

    v0 --> v1a
    v0 --> v1b
    v0 --> v1c
    v1b --> v2a
    v1b --> v2b
    v1c --> v2b
    v1b -.-> v2c

    style v0 fill:#f8fafc,stroke:#64748b,color:#0f172a
    style v1a fill:#fee2e2,stroke:#ef4444,color:#991b1b
    style v2a fill:#fef3c7,stroke:#f59e0b,color:#92400e
    style v1b fill:#ccfbf1,stroke:#14b8a6,color:#115e59
    style v1c fill:#ccfbf1,stroke:#14b8a6,color:#115e59
    style v2b fill:#e0e7ff,stroke:#6366f1,color:#3730a3
    style v2c fill:#cffafe,stroke:#06b6d4,color:#155e75
```

</div>
<div class="col-span-5 divide-y divide-slate-200 border-t border-b border-slate-200 text-xs text-left">
  <div class="py-3">
    <div class="font-semibold text-slate-900">Leak-free negative history</div>
    <div class="text-slate-600 mt-1">Only failures from confirmed earlier rounds enter <code>failed attempts</code> — parallel siblings from the same round do not leak.</div>
  </div>
  <div class="py-3">
    <div class="font-semibold text-slate-900">Recombination &amp; epistasis</div>
    <div class="text-slate-600 mt-1">Two-parent edges (<code>v1b + v1c → v2b</code>) capture synergy vs. negative epistasis.</div>
  </div>
  <div class="py-3">
    <div class="font-semibold text-slate-900">Campaign-level search</div>
    <div class="text-slate-600 mt-1">Show a half-explored DAG and ask: <i>which branch to expand next, which to prune?</i></div>
  </div>
</div>
</div>

<!--
Speaker Notes:
Once we have the campaign DAG and round order, three things fall out naturally:
1. Only failures from confirmed earlier rounds enter negative history, preventing leakage from parallel siblings tested in the same round. Notice also that non-winning leaves are not always hard failures—some represent partial successes or constraint tradeoffs like high affinity but poor solubility.
2. Two-parent merges model recombination and epistasis directly.
3. Beyond single-step tasks, we can evaluate campaign-level search by showing a partially explored DAG and asking which branch to pursue next.
-->
