---
theme: default
title: "Research Idea: Transforming Protein Literature into Causal Benchmarks"
class: text-center
highlighter: shiki
drawings:
  persist: false
transition: slide-left
mdc: true
colorSchema: light
editor: false
contextMenu: false
hideInToc: true
---

# Towards a Causal NLP Benchmark from<br><span class="bg-gradient-to-r from-teal-700 via-cyan-700 to-indigo-700 bg-clip-text text-transparent">Protein Engineering Literature</span>

<div class="text-lg text-slate-600 mt-2 font-light max-w-2xl mx-auto">
Initial Idea & Architecture Proposal
</div>

<div class="mt-6 flex justify-center gap-2.5 flex-wrap">
  <span class="px-3 py-1 rounded-full bg-teal-50 border border-teal-300 text-teal-800 text-xs font-mono font-bold uppercase shadow-sm">Motivation & Problem</span>
  <span class="px-3 py-1 rounded-full bg-cyan-50 border border-cyan-300 text-cyan-800 text-xs font-mono font-bold uppercase shadow-sm">Atomic 9-Tuple</span>
  <span class="px-3 py-1 rounded-full bg-indigo-50 border border-indigo-300 text-indigo-800 text-xs font-mono font-bold uppercase shadow-sm">Lineage DAG</span>
  <span class="px-3 py-1 rounded-full bg-purple-50 border border-purple-300 text-purple-800 text-xs font-mono font-bold uppercase shadow-sm">Feedback & Next Steps</span>
</div>

<!--
Speaker Notes:
Hi everyone. Today I want to present an initial research idea for my PhD: building a new dataset and benchmark from protein engineering literature.
The goal of this presentation is not to show finished experimental results, but to present the conceptual architecture—how we model engineering decisions—and get your feedback on whether this framing makes sense and what challenges you foresee.
-->

---
transition: slide-left
---

# Outline

<div class="mt-10 max-w-2xl mx-auto space-y-4 text-left">
  <div class="flex items-center gap-5"><span class="text-3xl font-black text-teal-600 w-10">01</span><span class="text-lg font-semibold text-slate-900">Protein engineering in <s>10</s> 20 minutes</span></div>
  <div class="flex items-center gap-5"><span class="text-3xl font-black text-cyan-600 w-10">02</span><span class="text-lg font-semibold text-slate-900">Current benchmarks their problem</span></div>
  <div class="flex items-center gap-5"><span class="text-3xl font-black text-sky-600 w-10">03</span><span class="text-lg font-semibold text-slate-900">Where and how to get data</span></div>
  <div class="flex items-center gap-5"><span class="text-3xl font-black text-indigo-600 w-10">04</span><span class="text-lg font-semibold text-slate-900">Depedency graphs and build thing of their graph </span></div>
  <div class="flex items-center gap-5"><span class="text-3xl font-black text-purple-600 w-10">05</span><span class="text-lg font-semibold text-slate-900">Data sources &amp; open questions</span></div>
</div>

<!--
Speaker Notes:
1. Crash course: proteins, amino acids, folding, target/binder roles, interactive docking walkthrough.
2. Why fitness tables (ProteinGym), static sequences and QA datasets miss the iterative reasoning behind mutations.
3. Representing one engineering step as a causal transition (SARS-CoV-2 example) and deriving 7+ tasks.
4. Connecting steps into a DAG to capture dead-ends (H_neg) and multi-parent epistasis.
5. Where to get data at FIT, LLM extraction reliability, annotation QC, scope, target venues.
-->

---
src: ./slides/01-primer.md
---
---
src: ./slides/02-benchmarks-gap.md
---
---
src: ./slides/03-causal-9tuple.md
---
---
src: ./slides/04-lineage-dag.md
---
---
src: ./slides/05-discussion.md
---
