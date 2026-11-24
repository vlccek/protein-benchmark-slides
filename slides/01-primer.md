---
layout: default
class: text-center
transition: slide-left
---

<div class="h-full flex flex-col items-center justify-center">
  <div class="text-[7rem] leading-none font-black bg-gradient-to-br from-teal-600 to-cyan-500 bg-clip-text text-transparent">01</div>
  <div class="text-3xl font-bold text-slate-900 mt-2">Protein engineering in 10 minutes</div>
  <div class="text-base text-slate-500 mt-3 max-w-xl">From a 1D sequence to a 3D machine — and why we rewrite it</div>
</div>

---
transition: slide-left
---

# A protein is a 1D chain

DNA → mRNA → a linear chain of amino acids. 

<div class="mt-2 flex justify-center">
  <img src="/protein-1d-code.jpg" alt="DNA to mRNA to 1D amino acid chain" class="rounded-xl border border-slate-200 shadow-sm h-[395px] w-full max-w-4xl object-contain bg-white" />
</div>

<div class="mt-3 flex justify-center gap-3 text-xs">
  <span class="px-3 py-1.5 rounded-lg bg-teal-50 border border-teal-200 text-teal-900"><b>20-letter</b> alphabet</span>
  <span class="px-3 py-1.5 rounded-lg bg-cyan-50 border border-cyan-200 text-cyan-900"><b>50–1,000+</b> residues</span>
  <span class="px-3 py-1.5 rounded-lg bg-slate-50 border border-slate-200 text-slate-600">Enzymes · antibodies · receptors</span>
</div>

<!--
Speaker Notes:
Let's start with what a protein actually is:
1. Proteins are the main molecular machines in biology—enzymes that catalyze reactions, antibodies that recognize pathogens, and receptors that transmit signals.
2. Chemically, a protein is just a linear 1D chain of amino acids, encoded by DNA and translated via mRNA.
3. Instead of 4 DNA bases, proteins are written in a 20-letter chemical alphabet, typically 50 to 1,000+ residues long.
-->

---
transition: slide-left
---

# The Building Block: Amino Acid Anatomy

Same **backbone** every time — only the **sidechain** on the **α-carbon** changes.

<div class="mt-3 p-4 rounded-xl bg-white border border-slate-200 shadow-sm flex items-center justify-center">
  <svg viewBox="0 0 800 330" class="w-full max-w-4xl h-[325px]">
    <line x1="26" y1="190" x2="70" y2="190" stroke="#cbd5e1" stroke-width="2.5" stroke-dasharray="7 5" stroke-linecap="round" />
    <line x1="762" y1="190" x2="792" y2="190" stroke="#cbd5e1" stroke-width="2.5" stroke-dasharray="7 5" stroke-linecap="round" />
    <text style="font-size:10px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:600" x="26" y="230" fill="#94a3b8">previous AA</text>
    <text style="font-size:10px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:600" x="792" y="230" text-anchor="end" fill="#94a3b8">next AA</text>
    <line x1="121" y1="190" x2="152" y2="190" stroke="#475569" stroke-width="3" stroke-linecap="round" />
    <line x1="180" y1="126" x2="180" y2="163" stroke="#475569" stroke-width="3" stroke-linecap="round" />
    <line x1="207" y1="190" x2="339" y2="190" stroke="#475569" stroke-width="3" stroke-linecap="round" />
    <line x1="370" y1="159" x2="370" y2="102" stroke="#4338ca" stroke-width="3" stroke-linecap="round" />
    <line x1="370" y1="221" x2="370" y2="255" stroke="#475569" stroke-width="3" stroke-linecap="round" />
    <line x1="401" y1="190" x2="518" y2="190" stroke="#475569" stroke-width="3" stroke-linecap="round" />
    <line x1="539" y1="164" x2="539" y2="130" stroke="#475569" stroke-width="3" stroke-linecap="round" />
    <line x1="551" y1="164" x2="551" y2="130" stroke="#475569" stroke-width="3" stroke-linecap="round" />
    <line x1="571" y1="190" x2="629" y2="190" stroke="#475569" stroke-width="3" stroke-linecap="round" />
    <line x1="673" y1="190" x2="699" y2="190" stroke="#475569" stroke-width="3" stroke-linecap="round" />
    <circle cx="105" cy="190" r="16" fill="#f8fafc" stroke="#94a3b8" stroke-width="2" />
    <text style="font-size:13px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:bold" x="105" y="195" text-anchor="middle" fill="#475569">H</text>
    <circle cx="180" cy="110" r="16" fill="#f8fafc" stroke="#94a3b8" stroke-width="2" />
    <text style="font-size:13px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:bold" x="180" y="115" text-anchor="middle" fill="#475569">H</text>
    <circle cx="180" cy="190" r="27" fill="#e0f2fe" stroke="#0284c7" stroke-width="2.5" />
    <text style="font-size:17px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:bold" x="180" y="196" text-anchor="middle" fill="#0c4a6e">N</text>
    <text style="font-size:11px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:600" x="180" y="72" text-anchor="middle" fill="#0369a1">Amino group (N-terminus)</text>
    <circle cx="370" cy="190" r="31" fill="#ccfbf1" stroke="#0f766e" stroke-width="3" />
    <text style="font-size:18px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:bold" x="370" y="197" text-anchor="middle" fill="#134e4a">Cα</text>
    <text style="font-size:12px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:bold" x="332" y="243" text-anchor="end" fill="#0f766e">α-carbon</text>
    <circle cx="370" cy="272" r="17" fill="#f8fafc" stroke="#94a3b8" stroke-width="2" />
    <text style="font-size:13px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:bold" x="370" y="277" text-anchor="middle" fill="#475569">H</text>
    <rect x="276" y="52" width="188" height="50" rx="25" fill="#eef2ff" stroke="#4338ca" stroke-width="2.5" />
    <text style="font-size:15px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:bold" x="370" y="83" text-anchor="middle" fill="#312e81">Sidechain (R)</text>
    <text style="font-size:11px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:600" x="370" y="30" text-anchor="middle" fill="#4338ca">20 variants — this is the only part that changes</text>
    <circle cx="545" cy="190" r="27" fill="#fef3c7" stroke="#d97706" stroke-width="2.5" />
    <text style="font-size:17px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:bold" x="545" y="196" text-anchor="middle" fill="#78350f">C</text>
    <circle cx="545" cy="106" r="24" fill="#ffe4e6" stroke="#e11d48" stroke-width="2.5" />
    <text style="font-size:16px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:bold" x="545" y="112" text-anchor="middle" fill="#881337">O</text>
    <circle cx="651" cy="190" r="23" fill="#ffe4e6" stroke="#e11d48" stroke-width="2.5" />
    <text style="font-size:16px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:bold" x="651" y="196" text-anchor="middle" fill="#881337">O</text>
    <circle cx="715" cy="190" r="16" fill="#f8fafc" stroke="#94a3b8" stroke-width="2" />
    <text style="font-size:13px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:bold" x="715" y="195" text-anchor="middle" fill="#475569">H</text>
    <text style="font-size:11px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:600" x="623" y="262" text-anchor="middle" fill="#b45309">Carboxyl group —COOH (C-terminus)</text>
    <circle cx="105" cy="190" r="27" fill="none" stroke="#f43f5e" stroke-width="2" stroke-dasharray="5 4" />
    <rect x="623" y="160" width="112" height="60" rx="30" fill="none" stroke="#f43f5e" stroke-width="2" stroke-dasharray="5 4" />
    <text style="font-size:10px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:bold" x="105" y="145" text-anchor="middle" fill="#e11d48">−H</text>
    <text style="font-size:10px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:bold" x="679" y="145" text-anchor="middle" fill="#e11d48">−OH</text>
    <text style="font-size:11.5px;font-family:ui-sans-serif, system-ui, sans-serif;font-weight:600" x="400" y="316" text-anchor="middle" fill="#64748b">Peptide bond: this −H and that −OH leave as H₂O, so C bonds straight to the next N</text>
  </svg>
</div>

<div class="mt-3 flex justify-center gap-3 text-xs flex-wrap">
  <span class="px-3 py-1.5 rounded-lg bg-teal-50 border border-teal-200 text-teal-900"><b>N – Cα – C</b> shared backbone</span>
  <span class="px-3 py-1.5 rounded-lg bg-indigo-50 border border-indigo-200 text-indigo-900"><b>20 sidechains (R)</b> · size, charge, polarity</span>
  <span class="px-3 py-1.5 rounded-lg bg-rose-50 border border-rose-200 text-rose-900"><b>Peptide bond</b> · condenses −H &amp; −OH → H₂O</span>
</div>

<!--
Speaker Notes:
Before looking at how chains fold, here is the simplest view of a single amino acid:
1. Every amino acid has the exact same backbone (N - Cα - C) centered on the alpha-carbon (Cα).
2. The carboxyl group is written out on the right: the carbon carries a DOUBLE bond to one oxygen and a single bond to an -OH. Four bonds on carbon, three on nitrogen - the valences are right.
3. What distinguishes the 20 amino acids is the Sidechain (R-group) attached to Cα. That sidechain is what gives each letter its chemical personality when the protein folds.

On the -OH and partial charges (if asked):
- Oxygen is far more electronegative than carbon or hydrogen, so it pulls the shared electrons towards itself. BOTH oxygens of the carboxyl group carry a partial negative charge (delta minus); the hydroxyl hydrogen is left partially positive (delta plus). The same is true of the nitrogen in the amino group.
- These partial charges are the whole point: they are what makes the group polar, and they are why the backbone can donate and accept hydrogen bonds - which is exactly what holds an alpha-helix together (i to i+4) on the next slides.
- Careful with wording: a partial charge (delta minus) is NOT a full charge. But at physiological pH (~7.4) the carboxyl group is mostly deprotonated anyway - it loses the H entirely and becomes -COO(minus), a real full negative charge spread over both oxygens. The amino group is correspondingly protonated to -NH3(plus). So a free amino acid in water is really a zwitterion: positive at one end, negative at the other, neutral overall.
- The neutral -NH2 / -COOH form drawn here is the textbook convention, because it makes the peptide-bond condensation easy to see: the -OH leaves from one residue, an -H leaves from the next one's nitrogen, and together they go off as water.

On electronic / electrostatic interactions (if asked):
- Yes, at the fundamental level, ALL chemical bonds and non-covalent interactions in proteins are electronic/electrostatic (driven by electron distribution and attraction between opposite charges—not "chemical reactions" in the covalent sense for non-covalent folding, though):
  - Covalent bonds (like C–N, C–C) share electron pairs between atoms.
  - Hydrogen bonds & salt bridges are electrostatic attractions between partial (delta+/delta-) or full (+ / -) charges.
  - Even van der Waals / dispersion forces arise from temporary fluctuations in electron clouds. (And the hydrophobic effect is driven by water's hydrogen-bond network pushing nonpolar groups together.)

On single bond vs. double bond (if asked):
- Single bond (1 shared electron pair, sigma bond): longer, weaker than a double bond, and allows FREE ROTATION around the bond axis (like the N–Cα and Cα–C hinges, which is how a protein chain bends and folds).
- Double bond (2 shared electron pairs, sigma + pi bond): shorter, stronger, and RIGID—the pi bond locks rotation (like C=O here).
- Bonus connection to proteins: when amino acids link into a peptide bond (C–N), electron resonance gives it ~40% double-bond character, which locks the peptide plane flat!
-->

---
transition: slide-left
---

# 1° Primary: the sequence

A string over a 20-letter alphabet, $S \in \Sigma^L$. No 3D shape yet — but it is already encoded.

<div class="mt-3 p-8 rounded-xl bg-white border border-slate-200 shadow-sm h-[325px] flex flex-col items-center justify-center text-center">
  <div class="text-[11px] font-mono text-slate-400 mb-4 flex items-center justify-between w-full max-w-3xl px-2">
    <span>N-terminus</span>
    <span>1D Polypeptide Chain (<i>S</i> ∈ Σ<sup><i>L</i></sup>)</span>
    <span>C-terminus</span>
  </div>
  <div class="flex items-center justify-center gap-2.5 flex-wrap">
    <span class="w-12 h-12 rounded-full bg-teal-50 border-2 border-teal-300 text-teal-900 font-mono font-bold text-base inline-flex items-center justify-center shadow-xs">M</span>
    <span class="text-slate-300 font-bold text-lg">–</span>
    <span class="w-12 h-12 rounded-full bg-teal-50 border-2 border-teal-300 text-teal-900 font-mono font-bold text-base inline-flex items-center justify-center shadow-xs">E</span>
    <span class="text-slate-300 font-bold text-lg">–</span>
    <span class="w-12 h-12 rounded-full bg-teal-50 border-2 border-teal-300 text-teal-900 font-mono font-bold text-base inline-flex items-center justify-center shadow-xs">T</span>
    <span class="text-slate-300 font-bold text-lg">–</span>
    <span class="w-12 h-12 rounded-full bg-teal-50 border-2 border-teal-300 text-teal-900 font-mono font-bold text-base inline-flex items-center justify-center shadow-xs">F</span>
    <span class="text-slate-300 font-bold text-lg">–</span>
    <span class="w-12 h-12 rounded-full bg-teal-50 border-2 border-teal-300 text-teal-900 font-mono font-bold text-base inline-flex items-center justify-center shadow-xs">S</span>
    <span class="text-slate-300 font-bold text-lg">–</span>
    <span class="w-12 h-12 rounded-full bg-teal-50 border-2 border-teal-300 text-teal-900 font-mono font-bold text-base inline-flex items-center justify-center shadow-xs">D</span>
    <span class="text-slate-300 font-bold text-lg">–</span>
    <span class="w-12 h-12 rounded-full bg-teal-50 border-2 border-teal-300 text-teal-900 font-mono font-bold text-base inline-flex items-center justify-center shadow-xs">L</span>
    <span class="text-slate-300 font-bold text-lg">–</span>
    <span class="w-12 h-12 rounded-full bg-teal-50 border-2 border-teal-300 text-teal-900 font-mono font-bold text-base inline-flex items-center justify-center shadow-xs">W</span>
    <span class="text-slate-300 font-bold text-lg">–</span>
    <span class="w-12 h-12 rounded-full bg-teal-50 border-2 border-teal-300 text-teal-900 font-mono font-bold text-base inline-flex items-center justify-center shadow-xs">K</span>
    <span class="text-slate-300 font-bold text-lg">–</span>
    <span class="w-12 h-12 rounded-full bg-teal-50 border-2 border-teal-300 text-teal-900 font-mono font-bold text-base inline-flex items-center justify-center shadow-xs">L</span>
    <span class="text-slate-400 font-mono text-base pl-3">…</span>
  </div>
</div>

<div class="mt-3 flex justify-center gap-3 text-xs flex-wrap">
  <span class="px-3 py-1.5 rounded-lg bg-teal-50 border border-teal-200 text-teal-900"><b>Covalent backbone</b> · fixed N→C order</span>
  <span class="px-3 py-1.5 rounded-lg bg-cyan-50 border border-cyan-200 text-cyan-900"><b>50–1,000+</b> residues</span>
  <span class="px-3 py-1.5 rounded-lg bg-indigo-50 border border-indigo-200 text-indigo-900"><b>Encoded physics</b> · sequence dictates folding</span>
</div>

<!--
Speaker Notes:
1° Primary structure is just the raw 1D sequence of amino acids—written in our 20-letter alphabet. There is no 3D shape yet, but the physical properties of each residue already encode how it will fold in water.

On the peptide bond (those "–" dashes between residues):
- How it forms (condensation / dehydration): the carboxyl group (-COOH) of one amino acid reacts with the amino group (-NH₂) of the next one. The -OH and one -H leave together as a water molecule (H₂O), forming a strong covalent C–N amide bond called a peptide bond.
- Why direction matters (N → C): because one end always has a free amino group (N-terminus) and the other a free carboxyl group (C-terminus), the sequence has a fixed direction—just like reading text left-to-right.
- Geometry bonus: although drawn as a single C–N bond, electron resonance with the adjacent C=O gives the peptide bond ~40% double-bond character. That makes the peptide bond itself rigid and flat (planar, ω = 180°), while the single bonds on either side of Cα (φ and ψ) can still rotate!
-->

---
transition: slide-left
---

# 2° Secondary: local motifs

Backbone hydrogen bonds between residues **close together in the chain** — nothing distant touches yet.

<div class="mt-4 flex justify-center">
  <img src="/protein-secondary.jpg" alt="alpha-helix and beta-sheet stabilised by backbone hydrogen bonds" class="rounded-xl border border-slate-200 shadow-sm h-[360px] object-contain bg-white" />
</div>

<div class="mt-4 flex justify-center gap-3 text-xs">
  <span class="px-3 py-1.5 rounded-lg bg-teal-50 border border-teal-200 text-teal-900"><b>α-helix</b> &nbsp;H-bond <span class="font-mono">i → i+4</span></span>
  <span class="px-3 py-1.5 rounded-lg bg-cyan-50 border border-cyan-200 text-cyan-900"><b>β-turn</b> &nbsp;<span class="font-mono">i → i+3</span></span>
  <span class="px-3 py-1.5 rounded-lg bg-slate-50 border border-slate-200 text-slate-600">Residues far apart in the chain still have not met</span>
</div>

<!--
Speaker Notes:
The only thing that matters here: secondary structure comes from LOCAL interactions. An alpha-helix is held by a backbone hydrogen bond between residue i and residue i+4 — four apart. A beta-turn is i to i+3. So at this level the chain only talks to its immediate neighbours.
Caveat if anyone asks: beta-SHEETS can pair strands that are far apart in the sequence — that pairing is really already tertiary contact. Helix and turn are the purely local ones.
Next we watch a chain actually fold, then see what distant residues do once it has.
-->

---
transition: slide-left
---

# 3° Tertiary: the fold

Now distant residues meet: the chain packs into a compact domain with a hydrophobic core and **binding pockets**.

<div class="mt-3">
  <PdbStructureCard pdbUrl="/1ycr.pdb" :chainFilter="['A']" colorMode="ss" height="360px" :zoom="1.35" :showControls="true" />
</div>

<div class="mt-3 flex items-center justify-between text-xs">
  <span class="text-slate-600">Toggle <b>+ Surface</b> for the binding cleft, <b>+ Atoms</b> for the sidechains lining it.</span>
  <span class="font-mono text-[11px] text-slate-400">PDB 1YCR · chain A · MDM2</span>
</div>

<!--
Speaker Notes:
3° Tertiary structure is the full 3D fold of a single polypeptide chain—here shown on the real crystal structure of MDM2 (1YCR, Chain A).
Try clicking "+ Surface" to see the 3D binding cleft, and "+ Atoms" to see the individual sidechain atoms lining that pocket. This 3D shape is what drives biological function.
-->

---
transition: slide-left
---

# 4° Quaternary: the complex

Several folded chains dock together. **This interface is what we engineer.**

<div class="mt-3">
  <PdbStructureCard pdbUrl="/6m0j.pdb" :chainFilter="['A', 'E']" colorMode="chain" height="360px" :zoom="1.35" :showControls="true" />
</div>

<div class="mt-3 flex items-center justify-between text-xs">
  <span class="text-slate-600">Human ACE2 (teal) + viral Spike RBD (indigo). Mutations at this contact surface tune binding.</span>
  <span class="font-mono text-[11px] text-slate-400">PDB 6M0J · chains A + E</span>
</div>

<!--
Speaker Notes:
4° Quaternary structure is when multiple folded 3D chains assemble into a functional complex—here shown on the real crystal structure of SARS-CoV-2 Spike RBD bound to human ACE2 (PDB 6M0J).

Which is which:
- Teal (larger protein, Chain A): Human ACE2 — the receptor on the surface of human cells.
- Indigo / purple-blue (smaller protein, Chain E): SARS-CoV-2 Spike RBD (Receptor-Binding Domain) — the viral protein that grabs onto ACE2 to infect the cell.

Toggle "+ Surface" and "+ Atoms" to see the interface where the two proteins lock together. Both 3° and 4° structure directly determine function—and that interface is what protein engineering mutates (e.g. designing a minibinder to block this exact Spike–ACE2 contact!).
-->

---
transition: slide-left
---

# Folding is an optimization problem

The chain searches (φ, ψ) space for the lowest free energy: $\;x^{*} = \arg\min_{x} G(x)$

<div class="grid grid-cols-12 gap-4 mt-1 items-center text-xs text-left">
  <div class="col-span-7">
    <svg viewBox="0 0 520 300" class="w-full rounded-xl border border-slate-200 bg-white" font-family="ui-sans-serif, system-ui, sans-serif">
      <defs>
        <linearGradient id="fold-fill" x1="0" y1="0" x2="0" y2="1">
          <stop offset="0%" stop-color="#e0f2fe" stop-opacity="0.15" />
          <stop offset="100%" stop-color="#0e7490" stop-opacity="0.22" />
        </linearGradient>
        <marker id="arr-green" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" fill="#15803d" /></marker>
        <marker id="arr-red" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" fill="#dc2626" /></marker>
        <marker id="arr-slate" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" fill="#64748b" /></marker>
      </defs>
      <!-- axes -->
      <line x1="28" y1="275" x2="28" y2="18" stroke="#64748b" stroke-width="1.5" marker-end="url(#arr-slate)" />
      <line x1="28" y1="275" x2="505" y2="275" stroke="#64748b" stroke-width="1.5" marker-end="url(#arr-slate)" />
      <text x="20" y="150" style="font-size:11px" fill="#334155" text-anchor="middle" transform="rotate(-90 20 150)">Free energy G</text>
      <text x="500" y="291" style="font-size:11px" fill="#334155" text-anchor="end">Conformation space (φ, ψ)ⁿ</text>
      <!-- landscape -->
      <path d="M40,30 L42,32 L44,34 L47,37 L49,40 L51,43 L53,46 L55,50 L58,54 L60,58 L62,62 L64,66 L66,70 L69,73 L71,76 L73,78 L75,80 L77,82 L80,84 L82,85 L84,87 L86,89 L88,91 L91,94 L93,97 L95,100 L97,104 L99,107 L102,111 L104,115 L106,119 L108,123 L110,126 L113,130 L115,135 L117,139 L119,144 L121,150 L124,156 L126,163 L128,169 L130,175 L132,181 L135,186 L137,189 L139,192 L141,192 L143,192 L146,190 L148,187 L150,184 L152,180 L154,177 L157,173 L159,170 L161,168 L163,167 L165,166 L168,166 L170,166 L172,168 L174,169 L176,172 L179,174 L181,176 L183,178 L185,180 L187,181 L190,183 L192,183 L194,184 L196,184 L198,184 L201,184 L203,184 L205,184 L207,184 L209,185 L212,187 L214,188 L216,190 L218,192 L220,194 L223,196 L225,198 L227,200 L229,201 L231,202 L234,203 L236,204 L238,206 L240,208 L242,210 L245,212 L247,216 L249,220 L251,224 L253,229 L256,233 L258,238 L260,242 L262,246 L264,248 L267,250 L269,250 L271,249 L273,247 L275,244 L278,241 L280,236 L282,232 L284,228 L286,223 L289,220 L291,216 L293,214 L295,211 L297,209 L300,208 L302,206 L304,204 L306,203 L308,201 L311,199 L313,197 L315,195 L317,193 L319,191 L322,189 L324,187 L326,186 L328,186 L330,185 L333,185 L335,185 L337,185 L339,185 L341,185 L344,185 L346,184 L348,183 L350,181 L352,179 L355,177 L357,174 L359,172 L361,170 L363,169 L366,167 L368,167 L370,167 L372,167 L374,168 L377,170 L379,171 L381,173 L383,174 L385,175 L388,174 L390,173 L392,171 L394,168 L396,164 L399,159 L401,155 L403,150 L405,145 L407,141 L410,138 L412,136 L414,133 L416,131 L418,130 L421,128 L423,126 L425,124 L427,122 L429,119 L432,115 L434,112 L436,109 L438,105 L440,102 L443,99 L445,96 L447,94 L449,92 L451,90 L454,89 L456,87 L458,85 L460,83 L462,81 L465,79 L467,76 L469,72 L471,68 L473,65 L476,61 L478,57 L480,53 L480,275 L40,275 Z" fill="url(#fold-fill)" stroke="none" />
      <path d="M40,30 L42,32 L44,34 L47,37 L49,40 L51,43 L53,46 L55,50 L58,54 L60,58 L62,62 L64,66 L66,70 L69,73 L71,76 L73,78 L75,80 L77,82 L80,84 L82,85 L84,87 L86,89 L88,91 L91,94 L93,97 L95,100 L97,104 L99,107 L102,111 L104,115 L106,119 L108,123 L110,126 L113,130 L115,135 L117,139 L119,144 L121,150 L124,156 L126,163 L128,169 L130,175 L132,181 L135,186 L137,189 L139,192 L141,192 L143,192 L146,190 L148,187 L150,184 L152,180 L154,177 L157,173 L159,170 L161,168 L163,167 L165,166 L168,166 L170,166 L172,168 L174,169 L176,172 L179,174 L181,176 L183,178 L185,180 L187,181 L190,183 L192,183 L194,184 L196,184 L198,184 L201,184 L203,184 L205,184 L207,184 L209,185 L212,187 L214,188 L216,190 L218,192 L220,194 L223,196 L225,198 L227,200 L229,201 L231,202 L234,203 L236,204 L238,206 L240,208 L242,210 L245,212 L247,216 L249,220 L251,224 L253,229 L256,233 L258,238 L260,242 L262,246 L264,248 L267,250 L269,250 L271,249 L273,247 L275,244 L278,241 L280,236 L282,232 L284,228 L286,223 L289,220 L291,216 L293,214 L295,211 L297,209 L300,208 L302,206 L304,204 L306,203 L308,201 L311,199 L313,197 L315,195 L317,193 L319,191 L322,189 L324,187 L326,186 L328,186 L330,185 L333,185 L335,185 L337,185 L339,185 L341,185 L344,185 L346,184 L348,183 L350,181 L352,179 L355,177 L357,174 L359,172 L361,170 L363,169 L366,167 L368,167 L370,167 L372,167 L374,168 L377,170 L379,171 L381,173 L383,174 L385,175 L388,174 L390,173 L392,171 L394,168 L396,164 L399,159 L401,155 L403,150 L405,145 L407,141 L410,138 L412,136 L414,133 L416,131 L418,130 L421,128 L423,126 L425,124 L427,122 L429,119 L432,115 L434,112 L436,109 L438,105 L440,102 L443,99 L445,96 L447,94 L449,92 L451,90 L454,89 L456,87 L458,85 L460,83 L462,81 L465,79 L467,76 L469,72 L471,68 L473,65 L476,61 L478,57 L480,53" fill="none" stroke="#0f172a" stroke-width="2.2" stroke-linejoin="round" />
      <!-- unfolded ensemble -->
      <circle cx="44" cy="33" r="4.5" fill="#94a3b8" />
      <circle cx="476" cy="60" r="4.5" fill="#94a3b8" />
      <text x="54" y="26" style="font-size:10.5px" fill="#475569" font-weight="600">Unfolded chain</text>
      <text x="54" y="38" style="font-size:9px" fill="#64748b">high G, huge entropy</text>
      <!-- productive folding path -->
      <path d="M210,40 C250,70 265,150 268,236" fill="none" stroke="#15803d" stroke-width="2.2" stroke-dasharray="6 4" marker-end="url(#arr-green)" />
      <text x="214" y="34" style="font-size:10.5px" fill="#15803d" font-weight="700">folding ⟶ minimize G</text>
      <!-- native state -->
      <circle cx="269" cy="250" r="6.5" fill="#15803d" stroke="#fff" stroke-width="1.5" />
      <text x="282" y="256" style="font-size:10.5px" fill="#15803d" font-weight="700">Native fold</text>
      <text x="282" y="268" style="font-size:9px" fill="#166534">global minimum → function</text>
      <!-- misfolding into kinetic trap -->
      <path d="M72,62 C88,110 112,160 134,186" fill="none" stroke="#dc2626" stroke-width="2.2" marker-end="url(#arr-red)" />
      <text x="96" y="92" style="font-size:10.5px" fill="#dc2626" font-weight="700">misfolding</text>
      <circle cx="141" cy="192" r="6.5" fill="#dc2626" stroke="#fff" stroke-width="1.5" />
      <text x="152" y="214" style="font-size:10.5px" fill="#b91c1c" font-weight="700">Misfolded</text>
      <text x="152" y="226" style="font-size:9px" fill="#b91c1c">local min / kinetic trap</text>
      <!-- trap -> aggregation -->
      <path d="M135,200 L82,244" fill="none" stroke="#dc2626" stroke-width="1.8" stroke-dasharray="4 3" marker-end="url(#arr-red)" />
      <text x="40" y="262" style="font-size:9.5px" fill="#991b1b" font-weight="600">aggregation</text>
      <text x="40" y="272" style="font-size:8.5px" fill="#991b1b">amyloid, disease</text>
      <!-- barrier -->
      <text x="170" y="184" style="font-size:9px" fill="#475569" text-anchor="middle">ΔG‡</text>
      <!-- chaperone escape -->
      <path d="M146,184 C165,125 228,130 259,236" fill="none" stroke="#64748b" stroke-width="1.4" stroke-dasharray="2 3" marker-end="url(#arr-slate)" />
      <text x="196" y="132" style="font-size:8.5px" fill="#64748b" text-anchor="middle">chaperones</text>
      <!-- intermediate -->
      <circle cx="385" cy="175" r="5" fill="#d97706" stroke="#fff" stroke-width="1.5" />
      <text x="385" y="196" style="font-size:9.5px" fill="#b45309" text-anchor="middle" font-weight="600">intermediate</text>
      <text x="385" y="207" style="font-size:8.5px" fill="#b45309" text-anchor="middle">(shallow trap)</text>
    </svg>
  </div>
  <div class="col-span-5 space-y-2.5">
    <div class="p-3 rounded-xl bg-white border border-slate-200">
      <div class="font-semibold text-slate-900 mb-1">Objective &amp; Search Space</div>
      <p class="text-slate-600 leading-relaxed">
        <b>Anfinsen:</b> the sequence alone encodes the native fold. Variables = (φ, ψ) per residue; objective = <b>G</b>.
      </p>
    </div>
    <div class="p-3 rounded-xl bg-white border border-slate-200">
      <div class="font-semibold text-slate-900 mb-1">Levinthal's Paradox</div>
      <p class="text-slate-600 leading-relaxed">
        <b>3<sup>100</sup> ≈ 10<sup>47</sup></b> conformations for a 100-AA chain — yet it folds in milliseconds. The landscape is a <b>funnel</b>, not a random search.
      </p>
    </div>
    <div class="p-3 rounded-xl bg-red-50 border border-red-200">
      <div class="font-semibold text-red-800 mb-1">Misfolding = Stuck in a Local Minimum</div>
      <p class="text-slate-600 leading-relaxed">
        <b>Kinetic traps</b> = non-functional folds that aggregate (amyloid-β, prions). Every mutation reshapes G (<b>ΔΔG</b>) — deepening the native well, or a trap.
      </p>
    </div>
  </div>
</div>

<!--
Speaker Notes:
Stepping back from the four structural levels: all of that folding is one optimization problem. The chain explores conformation space—the two rotatable backbone angles (phi, psi) per residue from the amino-acid slide—looking for the global minimum of free energy G. Anfinsen showed that the sequence alone determines this native state.
Levinthal's paradox: exhaustive search is impossible (3^100 conformations), so the landscape must be funnel-shaped and biased toward the native state.
But the landscape is rugged: the chain can get stuck in local minima—misfolded states. These are non-functional and can aggregate into amyloids (Alzheimer's, Parkinson's, prion diseases). Chaperones help proteins escape traps.
Link to engineering: every mutation reshapes this landscape (ddG). Good mutations deepen the native well; bad ones create or deepen traps. This is why predicting mutation effects is hard and why we need causal data.
-->

---
transition: slide-left
clicks: 1
---

<ProteinFoldingDemo mode="folding" />

<!--
Speaker Notes:
Here is Part 1 of our 3D walkthrough—focusing purely on the 13-AA chain itself (no target protein yet):
- Step 1 (Click 0): In free solution, the 13 amino acids (ETFSDLWKLLPEN) form an extended 1D chain (40.6 Å). Notice how the three hydrophobic sidechains—Phe19, Trp23, and Leu26—point in different directions.
- Step 2 (Click 1): Press Right Arrow (Next). By rotating the backbone (Phi, Psi) angles to (-60, -45), the chain coils into a compact 3D alpha-helix (23.2 Å, 3.6 AA per turn). Because Phe19 (i), Trp23 (i+4), and Leu26 (i+7) are spaced 3-4 residues apart in 1D, folding aligns all three onto the exact same bottom face!
-->

---
transition: slide-left
---

# Why engineer proteins?

Evolution optimized for survival — not for therapeutics or industrial chemistry.

<div class="mt-4 flex justify-center">
  <img src="/protein-engineering-why.jpg" alt="Wild-type vs engineered protein after mutation" class="rounded-xl border border-slate-200 shadow-sm h-[360px] object-contain bg-white" />
</div>

<div class="mt-4 flex justify-center gap-3 text-xs">
  <span class="px-3 py-1.5 rounded-lg bg-emerald-50 border border-emerald-200 text-emerald-900">Edit the sequence <span class="font-mono">S<sub>in</sub> —Δ→ S<sub>out</sub></span></span>
  <span class="px-3 py-1.5 rounded-lg bg-cyan-50 border border-cyan-200 text-cyan-900">Reshape the 3D surface</span>
  <span class="px-3 py-1.5 rounded-lg bg-rose-50 border border-rose-200 text-rose-900">One wrong edit collapses the fold</span>
</div>

<!--
Speaker Notes:
Finally, why do we engineer proteins—and why is this hard?
1. Natural "wild-type" proteins often have weak binding, poor thermal stability, or work only under narrow physiological conditions.
2. By introducing targeted mutations (Delta) in the 1D sequence, we reshape the 3D interface to achieve tight, selective binding or higher activity.
3. Because 3D folding is tightly coupled, a single wrong mutation can destroy function, while the right combination unlocks orders-of-magnitude gains. Next we fix the vocabulary: target, locus, binder.
-->

---
transition: slide-left
---

# Concrete Example: Blocking SARS-CoV-2 Spike

To stop a virus, we don't need to destroy it—just **plug every Spike protein** so it cannot enter the cell:

<div class="grid grid-cols-12 gap-5 mt-4 items-center text-xs text-left">
  <div class="col-span-6 flex justify-center">
    <img src="/shutterstock_1701931039.webp" alt="SARS-CoV-2 virus with surface Spike proteins" class="rounded-xl border border-slate-200 shadow-sm max-h-[310px] w-full object-contain bg-white" />
  </div>
  <div class="col-span-6 space-y-3">
    <div class="p-3.5 rounded-xl bg-white border border-rose-200 shadow-sm">
      <div class="font-mono text-[10px] font-bold text-rose-700 mb-1">1. WHAT THE SPIKE PROTEIN DOES</div>
      <div class="font-semibold text-slate-900 mb-1">The Viral Entry Key</div>
      <p class="text-slate-600 leading-relaxed">
        The Spike trimers on the virus surface use their tip (<b>RBD</b>) to latch onto the human <b>ACE2</b> receptor—unlocking entry into host cells.
      </p>
    </div>
    <div class="p-3.5 rounded-xl bg-white border border-teal-200 shadow-sm">
      <div class="font-mono text-[10px] font-bold text-teal-700 mb-1">2. THE ENGINEERING IDEA</div>
      <div class="font-semibold text-slate-900 mb-1">Cap Every Spike with a Minibinder</div>
      <p class="text-slate-600 leading-relaxed">
        Engineer a small, ultra-tight <b>minibinder</b> that outcompetes ACE2. Covering the Spike tips sterically blocks cell attachment—neutralizing the virus.
      </p>
    </div>
  </div>
</div>

<div class="mt-4 flex justify-center gap-3 text-xs flex-wrap">
  <span class="px-3 py-1.5 rounded-lg bg-rose-50 border border-rose-200 text-rose-900"><b>Target</b> · SARS-CoV-2 Spike RBD</span>
  <span class="px-3 py-1.5 rounded-lg bg-teal-50 border border-teal-200 text-teal-900"><b>Mechanism</b> · Competitive blockade of ACE2</span>
  <span class="px-3 py-1.5 rounded-lg bg-indigo-50 border border-indigo-200 text-indigo-900"><b>Precedent</b> · De novo minibinders (LCB1 / LCB3)</span>
</div>

<!--
Speaker Notes:
Here is a concrete real-world motivation for protein engineering—and the exact example we will use later in our 9-tuple benchmark:
1. On the surface of SARS-CoV-2 are Spike proteins. The tip of each Spike (the Receptor-Binding Domain, RBD) acts as a molecular key that binds the human ACE2 receptor so the virus can enter the cell.
2. The engineering idea is simple and powerful: what if we design a small protein ("minibinder") that binds that Spike RBD much more tightly than ACE2? If we flood/cap all the Spike proteins on the virus, it physically cannot dock to our cells—neutralizing the infection.
3. This is not just theory: Baker Lab (Cao et al., Science 2020) engineered picomolar de novo minibinders (LCB1, LCB3) doing exactly this. Next, let's fix the four core roles in our schema: Target, Locus, and Binder.
-->

---
transition: slide-left
---

# The cast: target, locus, binder

The four roles every protein engineering paper talks about — and their schema symbols.

<div class="grid grid-cols-2 gap-4 mt-5 text-sm text-left">

  <div class="p-4 rounded-xl bg-white border border-teal-200 shadow-sm">
    <div class="font-mono text-[11px] font-bold text-teal-700 mb-1">PEPTIDE &lt; 50 AA &nbsp;·&nbsp; PROTEIN 50–1000+ AA</div>
    <div class="font-bold text-slate-900 mb-1">Chain length changes the physics</div>
    <p class="text-slate-600 text-xs leading-relaxed">A short peptide is floppy in water and folds only on binding. A protein folds on its own into a stable 3D scaffold.</p>
  </div>

  <div class="p-4 rounded-xl bg-white border border-amber-200 shadow-sm">
    <div class="font-mono text-[11px] font-bold text-amber-700 mb-1">THE LOCK</div>
    <div class="font-bold text-slate-900 mb-1">Target <i>T</i> &nbsp;+&nbsp; Locus <i>L</i></div>
    <p class="text-slate-600 text-xs leading-relaxed">The protein we want to block, and the specific pocket on its surface we aim at. <b>Two separate decisions</b> — hence two fields.</p>
  </div>

  <div class="p-4 rounded-xl bg-white border border-emerald-200 shadow-sm">
    <div class="font-mono text-[11px] font-bold text-emerald-700 mb-1">THE KEY</div>
    <div class="font-bold text-slate-900 mb-1">Binder <i>S</i><sub>in</sub></div>
    <p class="text-slate-600 text-xs leading-relaxed">The peptide, miniprotein or antibody we engineer to plug into <i>L</i> with high affinity (low <i>K<sub>d</sub></i>).</p>
  </div>

  <div class="p-4 rounded-xl bg-slate-50 border border-slate-300 shadow-sm flex flex-col justify-center">
    <div class="font-bold text-slate-900 mb-1">Everything is a 1D string</div>
    <p class="text-slate-600 text-xs leading-relaxed">Peptide, protein, target, binder — all of it is <i>S</i> ∈ Σ<sup><i>L</i></sup>. That is what makes this tractable for an LLM.</p>
  </div>

</div>

<!--
Speaker Notes:
Now that we've seen how a 13-AA chain folds into an alpha-helix, let's clarify the roles before we dock it to a target:
1. A Peptide (< 50 AA) is a short chain—like our 13-AA p53 segment—often flexible in solution until it binds.
2. A Protein (50-1000+ AA) is a long chain that autonomously folds into a 3D scaffold with surface cavities.
3. The Target (T) is the receptor protein we want to block (MDM2), and its Active Site or Binding Locus (L) is the 3D pocket on its surface.
4. A Ligand is the broad term for any molecule binding that pocket, while a Peptide/Protein Binder (S_in) is an amino acid chain engineered to fold and lock into that pocket.
-->

---
transition: slide-left
clicks: 3
---

<ProteinFoldingDemo mode="docking" />

<!--
Speaker Notes:
Now in Part 2 of the 3D walkthrough, we bring in the Target Protein (MDM2, bottom) and step through docking and engineering using Right/Left Arrow keys:
- Step 1 (Click 0): Active Site Identification (T, L). MDM2 exposes a deep hydrophobic cleft (orange surface) below our folded 13-AA helix.
- Step 2 (Click 1): Binder Docking & Bottleneck (S_in, D). Pressing Next docks the helix straight into the MDM2 pocket. Pro27 kinks the right end of the helix, creating an entropic penalty (Kd = 600 nM).
- Step 3 (Click 2): Negative History (H_neg). Why naive mutations fail: W23A leaves a vacuum cavity, while L26W causes a steric clash against the Ile99 pocket wall (Kd > 50,000 nM).
- Step 4 (Click 3): Engineered Super-Binder (Delta, R -> E). Fixing the Pro27 kink (P27T) and adding Tyr22 pi-stacking boosts affinity 650-fold (Kd = 0.9 nM).
-->
