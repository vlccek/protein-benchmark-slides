---
layout: default
class: text-center
transition: slide-left
---

<div class="h-full flex flex-col items-center justify-center">
  <div class="text-[7rem] leading-none font-black bg-gradient-to-br from-sky-600 to-indigo-500 bg-clip-text text-transparent">03</div>
  <div class="text-3xl font-bold text-slate-900 mt-2">Representing one engineering step</div>
  <div class="text-base text-slate-500 mt-3 max-w-xl">Turning a paragraph of paper prose into a structured causal record</div>
</div>

<!--
Speaker Notes:
Now that we know what existing benchmarks miss, let's look at how we structure a single engineering step from real paper text.
-->

---
transition: slide-left
---

# From paper prose to one structured step

Instead of storing a static protein, we extract one engineering decision as **9 atomic fields in 3 stages**.

<div class="grid grid-cols-2 gap-5 mt-4 text-xs text-left">
  <div>
    <div class="font-bold text-sm text-slate-900 mb-2">Paper excerpt</div>
    <div class="p-4 rounded-lg bg-slate-50 border border-slate-200 text-slate-700 leading-relaxed space-y-2">
      <p>
        Against the <b>SARS-CoV-2 Spike RBD</b>, our starting scaffold <code>Minibinder_v1.2</code> bound patch <b>365–380</b> weakly (<i>K</i><sub>d</sub> = 450 nM) because of an <b>unfilled hydrophobic cavity</b> near Tyr369.
      </p>
      <p>
        An earlier <code>V24W</code> mutant clashed with helix-2 and abolished binding.
      </p>
      <p>
        To reach <i>K</i><sub>d</sub> &lt; 10 nM while staying soluble, we introduced <code>V24F + S28K</code>: Phe24 fills the cavity and π-stacks with Tyr369, while Lys28 forms a salt bridge.
      </p>
      <p>
        The resulting <code>Minibinder_v2.0</code> reached <i>K</i><sub>d</sub> = 3.2 nM (140× gain).
      </p>
    </div>
  </div>
  <div>
    <div class="font-bold text-sm text-slate-900 mb-2">Structured record (9 fields in 3 stages)</div>
    <div class="space-y-2">
      <div class="p-2.5 rounded-lg bg-white border border-teal-200 border-l-4 border-l-teal-500 shadow-sm space-y-0.5">
        <div class="font-bold text-teal-800 text-[11px] uppercase tracking-wide">1. Problem &amp; State (6 fields)</div>
        <div><b>Target:</b> SARS-CoV-2 Spike RBD &nbsp;·&nbsp; <b>Locus:</b> patch 365–380</div>
        <div><b>Baseline:</b> <code>Minibinder_v1.2</code> (<i>K</i><sub>d</sub> = 450 nM)</div>
        <div><b>Diagnosis:</b> Unfilled hydrophobic cavity near Tyr369</div>
        <div><b>Failed attempts:</b> <code>V24W</code> clashed with helix-2</div>
        <div><b>Goal:</b> <i>K</i><sub>d</sub> &lt; 10 nM, maintain solubility</div>
      </div>
      <div class="p-2.5 rounded-lg bg-white border border-indigo-200 border-l-4 border-l-indigo-500 shadow-sm space-y-0.5">
        <div class="font-bold text-indigo-800 text-[11px] uppercase tracking-wide">2. Design Decision (2 fields)</div>
        <div><b>Intervention:</b> <code>V24F + S28K</code> (Rosetta + inspection)</div>
        <div><b>Rationale:</b> Phe24 fills cavity &amp; π-stacks with Tyr369; Lys28 adds salt bridge</div>
      </div>
      <div class="p-2.5 rounded-lg bg-white border border-emerald-200 border-l-4 border-l-emerald-500 shadow-sm space-y-0.5">
        <div class="font-bold text-emerald-800 text-[11px] uppercase tracking-wide">3. Wet-Lab Outcome (1 field)</div>
        <div><b>Outcome:</b> <code>Minibinder_v2.0</code>, <i>K</i><sub>d</sub> = 3.2 nM (140× gain — Success)</div>
      </div>
    </div>
  </div>
</div>

<!--
Speaker Notes:
On the left is a realistic paragraph from a protein engineering paper.
On the right is how we decompose that single engineering step into 9 atomic fields grouped into three natural stages:
1) What was the starting problem, constraints, and dead-ends? (6 fields)
2) What did the authors propose and why? (2 fields)
3) What happened in the wet lab? (1 field)
Notice there is no math notation to memorize here—just clean separation of the scientific story.
-->

---
transition: slide-left
---

# Mask one field, get a benchmark task

Each field is kept atomic and separate so we can hide one piece and test a specific reasoning skill.

<div class="mt-4 text-xs">

| Task | Model Input | Masked Target | Question Tested |
| :--- | :--- | :--- | :--- |
| **Flaw Diagnosis** | Target, locus, baseline, goal | **Diagnosis** | *Why is the current binder weak?* |
| **Forward Design** *(Flagship)* | Full problem state + failed attempts | **Intervention + Rationale** | *What to mutate next, avoiding dead-ends?* |
| **Outcome Prediction** | Problem state + intervention + rationale | **Wet-lab Outcome** | *Will this proposed edit actually work?* |
| **Mechanistic Abduction** | Problem state + intervention + outcome | **Rationale** | *Why did this mutation succeed or fail?* |

</div>

<div class="mt-2.5 text-[11px] text-slate-500 text-left">
  Other maskings give <b>Site Selection</b> (predict Locus), <b>Reverse Localization</b>, and <b>Failure Analysis</b> (predict why a dead-end failed) — 7+ tasks from one schema.
</div>


<!--
Speaker Notes:
This slide explains why we split the record into 9 atomic fields instead of leaving it as one summary blob.
By masking different fields, a single extracted record yields 7+ distinct benchmark tasks—here are the four flagship ones.
Critically, this separation prevents answer leakage: the model cannot see the post-experiment outcome when predicting a design, nor see the structural diagnosis when asked to diagnose the baseline itself.
-->

---
transition: slide-left
---

# How the schema handles real papers

Three details that matter once you extract from real literature.

<div class="mt-6 divide-y divide-slate-200 border-t border-b border-slate-200 text-left">
  <div class="grid grid-cols-12 gap-6 py-4 items-center">
    <div class="col-span-6">
      <div class="font-semibold text-sm text-slate-900">Dead-ends stay attached</div>
      <div class="text-xs text-slate-600 mt-1">Failed mutations are kept inside the record instead of dropped like in PDB.</div>
    </div>
    <div class="col-span-6">
      <pre class="m-0 px-3 py-2 rounded bg-slate-50 border border-slate-200 font-mono text-[11px] text-slate-700 leading-relaxed">failed: ["V24W → helix-2 clash"]</pre>
    </div>
  </div>
  <div class="grid grid-cols-12 gap-6 py-4 items-center">
    <div class="col-span-6">
      <div class="font-semibold text-sm text-slate-900">Null fields are informative</div>
      <div class="text-xs text-slate-600 mt-1">Empty fields encode the experiment type rather than missing data.</div>
    </div>
    <div class="col-span-6">
      <pre class="m-0 px-3 py-2 rounded bg-slate-50 border border-slate-200 font-mono text-[11px] text-slate-700 leading-relaxed">baseline: null  # de novo design
failed:   []    # first round (no prior dead-ends)</pre>
    </div>
  </div>
  <div class="grid grid-cols-12 gap-6 py-4 items-center">
    <div class="col-span-6">
      <div class="font-semibold text-sm text-slate-900">Every record is a directed edge</div>
      <div class="text-xs text-slate-600 mt-1">Chaining steps from one paper builds the full campaign graph <i>(next)</i>.</div>
    </div>
    <div class="col-span-6">
      <pre class="m-0 px-3 py-2 rounded bg-slate-50 border border-slate-200 font-mono text-[11px] text-slate-700 leading-relaxed">v1.2  ──[ V24F + S28K ]──▶  v2.0</pre>
    </div>
  </div>
</div>

<!--
Speaker Notes:
Three design choices make this schema practical across diverse literature:
1. Failed attempts are explicitly stored alongside the winning path.
2. Null fields carry meaning—for example, a de novo design root naturally has no Baseline variant.
3. Each 9-tuple is a directed transition from one variant to the next, which brings us directly to how steps chain together into a campaign lineage graph.
-->
