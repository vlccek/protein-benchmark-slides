---
layout: default
class: text-center
transition: slide-left
---

<div class="h-full flex flex-col items-center justify-center">
  <div class="text-[7rem] leading-none font-black bg-gradient-to-br from-cyan-600 to-sky-500 bg-clip-text text-transparent">02</div>
  <div class="text-3xl font-bold text-slate-900 mt-2">Existing benchmarks</div>
  <div class="text-base text-slate-500 mt-3 max-w-xl">What already exists, and what is missing</div>
</div>

---
transition: slide-left
---

# What exists today

Four kinds of protein datasets are common in the literature.

<div class="mt-6 text-sm">

| Kind | Examples | What is actually inside |
| :--- | :--- | :--- |
| **Mutation tables** | `ProteinGym`, `SKEMPI 2.0` | CSV rows: `mutation → score` |
| **3D structures** | `PDB`, `PDBbind`, `SAbDab` | Coordinates of solved complexes |
| **Protein–text** | `Swiss-Prot`, `Mol-Instructions`, `ProteinLMBench` | Database fields wrapped in prompt templates |
| **Paper Q&amp;A** | `LitQA2`, `PubMedQA` | Factual questions over paper text |

</div>

<!--
Speaker Notes:
Here is the landscape of existing protein datasets. We want something genuinely NLP-like that tests scientific reasoning, including the "why" and failed attempts:
1. Mutation tables (ProteinGym, SKEMPI 2.0): Not NLP at all — just CSV numbers, no explanation of why a mutation was chosen.
2. 3D structures (PDB, PDBbind, SAbDab): Not NLP — only final winning structures (survivor bias, no failed trials).
3. Protein–text (Swiss-Prot, Mol-Instructions, ProteinLMBench): Looks like NLP, but it is rigid templates wrapped around static UniProt/Swiss-Prot facts about native proteins.
4. Paper Q&A (LitQA2, PubMedQA): Real text, but static reading comprehension rather than design decisions.
Let's look at concrete examples of each.
-->

---
transition: slide-left
---

# Mutation tables &amp; 3D databases

Useful for regression and structure prediction, but not NLP and no record of the engineering process.

<div class="grid grid-cols-2 gap-6 mt-6 text-xs text-left">

  <div>
    <div class="flex items-baseline justify-between mb-2">
      <span class="font-bold text-sm text-slate-900">Mutation tables</span>
      <span class="font-mono text-[11px] text-slate-500">ProteinGym, SKEMPI 2.0</span>
    </div>
    <pre class="p-3.5 rounded-lg bg-slate-50 border border-slate-200 font-mono text-[11px] text-slate-700 leading-relaxed">variant, mutation, ddG
p53_wt,  P27T,     -2.1 kcal/mol
p53_wt,  W23A,     +4.8 kcal/mol
p53_wt,  L26W,     +5.3 kcal/mol</pre>
    <p class="text-slate-600 mt-3 leading-relaxed">
      Numbers only. You see the score, never why someone tried <code>P27T</code> or what structural problem it solved.
    </p>
  </div>

  <div>
    <div class="flex items-baseline justify-between mb-2">
      <span class="font-bold text-sm text-slate-900">3D structure databases</span>
      <span class="font-mono text-[11px] text-slate-500">PDB, PDBbind, SAbDab</span>
    </div>
    <pre class="p-3.5 rounded-lg bg-slate-50 border border-slate-200 font-mono text-[11px] text-slate-700 leading-relaxed">PDB: 6M0J (ACE2 + Spike RBD)
ATOM  1  N  MET A  1  12.4  8.1  3.2
# Final winning complex only.
# Failed variants are never deposited.</pre>
    <p class="text-slate-600 mt-3 leading-relaxed">
      <b>Survivor bias:</b> only the winning crystal gets uploaded. Dead-ends stay buried in the paper text.
    </p>
  </div>

</div>

<!--
Speaker Notes:
Mutation tables like ProteinGym or SKEMPI give you CSV rows: mutation in, number out. There is no text, no hypothesis, and no explanation of why the mutation was chosen.
3D repositories like PDB suffer from survivor bias: nobody spends months crystallizing a failed mutant, so only the final winner is stored.
-->

---
transition: slide-left
---

# Protein–text datasets are templates

Datasets like `Mol-Instructions` and `ProteinLMBench` wrap static `Swiss-Prot` / UniProt entries in fixed prompts.

<div class="mt-5 p-4 rounded-lg bg-slate-50 border border-slate-200 font-mono text-[11.5px] text-slate-700 space-y-2.5 text-left leading-relaxed">
  <div>
    <span class="text-slate-400">Prompt:</span>
    "Please describe the general function of the following protein sequence: <code>MKTLLLTLVVVTIVCLDLGYT...</code>"
  </div>
  <div class="border-t border-slate-200 pt-2.5">
    <span class="text-slate-400">Target:</span>
    "Binds zinc ions and catalyzes the hydrolysis of peptide bonds in the extracellular matrix."
  </div>
</div>

<div class="mt-5 space-y-2 text-sm text-slate-700 text-left">
  <div>• <b>Templated lookup:</b> the same handful of prompt templates filled from UniProt tables.</div>
  <div>• <b>Static native facts:</b> describes what nature built, never how a weak binder was improved across rounds.</div>
</div>

<!--
Speaker Notes:
This is a key distinction: Mol-Instructions and ProteinLMBench look like NLP because they have text inputs and outputs.
In practice they are built by slotting Swiss-Prot / UniProt database fields into fixed prompt templates. That tests database memorization of native proteins, not scientific reasoning or iterative engineering.
-->

---
transition: slide-left
---

# Paper Q&amp;A vs. what we want

`LitQA2` and `PubMedQA` use real paper prose, but only test factual retrieval.

<div class="grid grid-cols-2 gap-6 mt-5 text-xs text-left">

  <div>
    <div class="flex items-baseline justify-between mb-2">
      <span class="font-bold text-sm text-slate-900">Paper Q&amp;A</span>
      <span class="font-mono text-[11px] text-slate-500">LitQA2, PubMedQA</span>
    </div>
    <div class="p-3.5 rounded-lg bg-slate-50 border border-slate-200 font-mono text-[11px] text-slate-700 space-y-2 leading-relaxed">
      <div><b>Q:</b> Which mutation improved affinity of Minibinder_v1.2 in Cao et al.?</div>
      <div><b>A:</b> F24 + K28 (K_d = 3.2 nM).</div>
    </div>
    <p class="text-slate-600 mt-3 leading-relaxed">
      Give the paper and the answer leaks in the text. Hide the paper and it becomes trivia recall.
    </p>
  </div>

  <div>
    <div class="flex items-baseline justify-between mb-2">
      <span class="font-bold text-sm text-teal-800">What we want (NLP-like design)</span>
      <span class="font-mono text-[11px] text-teal-700">structured + natural text</span>
    </div>
    <div class="p-3.5 rounded-lg bg-teal-50/50 border border-teal-200 font-mono text-[11px] text-slate-800 space-y-2 leading-relaxed">
      <div><b>Context:</b> Baseline binds at 450 nM. Cavity near Tyr369; W23A failed (lost anchor). Goal: K_d &lt; 10 nM.</div>
      <div><b>Model:</b> Mutate pos 24→Phe to fill the cavity and π-stack with Tyr369.</div>
    </div>
    <p class="text-slate-700 mt-3 leading-relaxed">
      Real scientific language with the outcome masked out, so the model actually has to reason.
    </p>
  </div>

</div>

<!--
Speaker Notes:
Literature QA benchmarks like LitQA2 do use real paper text, which is closer to NLP, but they are static Q&A retrieval. You cannot cleanly mask the outcome and ask the model to act like an engineer without leaking the answer.
On the right is what we want instead: real natural language from papers, but split into clean variables so we can hide the answer and test genuine reasoning.
-->

---
transition: slide-left
---

# What we want to benchmark

Can an LLM reason through an engineering step in real scientific language?

<div class="mt-6 max-w-3xl space-y-3.5 text-left text-sm">

  <div class="flex items-baseline gap-4">
    <span class="font-mono text-xs font-bold text-teal-700 w-5 shrink-0">1.</span>
    <div>
      <span class="font-semibold text-slate-900">Diagnose the flaw</span>
      <span class="text-slate-600"> — spot what is physically wrong with a weak binder (cavity, clash, kink).</span>
    </div>
  </div>

  <div class="flex items-baseline gap-4">
    <span class="font-mono text-xs font-bold text-teal-700 w-5 shrink-0">2.</span>
    <div>
      <span class="font-semibold text-slate-900">Propose the next edit</span>
      <span class="text-slate-600"> — suggest a mutation and biophysical rationale while avoiding failed paths.</span>
    </div>
  </div>

  <div class="flex items-baseline gap-4">
    <span class="font-mono text-xs font-bold text-teal-700 w-5 shrink-0">3.</span>
    <div>
      <span class="font-semibold text-slate-900">Predict or explain outcomes</span>
      <span class="text-slate-600"> — judge whether an edit works in wet-lab, or explain why a variant failed.</span>
    </div>
  </div>

  <div class="flex items-baseline gap-4">
    <span class="font-mono text-xs font-bold text-teal-700 w-5 shrink-0">4.</span>
    <div>
      <span class="font-semibold text-slate-900">No templates, no leakage</span>
      <span class="text-slate-600"> — natural prose from real papers, with clean separation between input and answer.</span>
    </div>
  </div>

</div>

<!--
Speaker Notes:
To sum up what we actually want to test before moving to the formal schema:
1. Diagnosing structural flaws from observations.
2. Proposing concrete mutations and explaining why, while learning from negative history.
3. Predicting or explaining wet-lab outcomes.
4. Doing all of this in real scientific language rather than UniProt templates or leaky Q&A.
Next, we formalize one engineering step into the 9-tuple.
-->
