<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'

interface AAInfo {
  code1: string
  code3: string
  name: string
  group: string
  color: string
  prop: string
  structureHint: string
}

const groups = [
  {
    title: 'Hydrophobic Aliphatic (5)',
    items: [
      { code1: 'A', code3: 'Ala', name: 'Alanine', group: 'Aliphatic', color: 'border-cyan-300 bg-cyan-50 text-cyan-900', prop: 'Small methyl (-CH₃), high α-helix propensity', structureHint: 'R = -CH₃' },
      { code1: 'V', code3: 'Val', name: 'Valine', group: 'Aliphatic', color: 'border-cyan-300 bg-cyan-50 text-cyan-900', prop: 'Branched Cβ isopropyl group, bulky near backbone', structureHint: 'R = -CH(CH₃)₂' },
      { code1: 'L', code3: 'Leu', name: 'Leucine ★', group: 'Aliphatic', color: 'border-cyan-600 bg-cyan-100 text-cyan-950 ring-1 ring-cyan-500/40', prop: 'Isobutyl core anchor (p53 Anchor 3: Leu26)', structureHint: 'R = -CH₂-CH(CH₃)₂' },
      { code1: 'I', code3: 'Ile', name: 'Isoleucine', group: 'Aliphatic', color: 'border-cyan-300 bg-cyan-50 text-cyan-900', prop: 'β-branched hydrophobic pocket wall (MDM2 Ile61/99)', structureHint: 'R = -CH(CH₃)CH₂CH₃' },
      { code1: 'M', code3: 'Met', name: 'Methionine', group: 'Aliphatic', color: 'border-cyan-300 bg-cyan-50 text-cyan-900', prop: 'Thioether chain (-S-CH₃), flexible hydrophobic packing', structureHint: 'R = -CH₂CH₂-S-CH₃' },
    ],
  },
  {
    title: 'Aromatic Rings (3)',
    items: [
      { code1: 'F', code3: 'Phe', name: 'Phenylalanine ★', group: 'Aromatic', color: 'border-teal-600 bg-teal-100 text-teal-950 ring-1 ring-teal-500/40', prop: 'Planar benzene ring, π-stacking (p53 Anchor 1: Phe19)', structureHint: 'R = -CH₂-C₆H₅ (Benzene)' },
      { code1: 'W', code3: 'Trp', name: 'Tryptophan ★', group: 'Aromatic', color: 'border-teal-600 bg-teal-100 text-teal-950 ring-1 ring-teal-500/40', prop: 'Largest double-ring indole anchor (p53 Anchor 2: Trp23)', structureHint: 'R = -CH₂-Indole (2 rings)' },
      { code1: 'Y', code3: 'Tyr', name: 'Tyrosine', group: 'Aromatic', color: 'border-teal-300 bg-teal-50 text-teal-900', prop: 'Phenol ring (-OH): combines π-stacking + H-bonding (L22Y)', structureHint: 'R = -CH₂-C₆H₄-OH' },
    ],
  },
  {
    title: 'Polar Uncharged (5)',
    items: [
      { code1: 'S', code3: 'Ser', name: 'Serine', group: 'Polar', color: 'border-sky-300 bg-sky-50 text-sky-900', prop: 'Small hydroxyl (-CH₂OH), frequent H-bond donor/acceptor', structureHint: 'R = -CH₂-OH' },
      { code1: 'T', code3: 'Thr', name: 'Threonine', group: 'Polar', color: 'border-sky-300 bg-sky-50 text-sky-900', prop: 'Secondary hydroxyl (replaces Pro27 in P27T mutant)', structureHint: 'R = -CH(OH)CH₃' },
      { code1: 'C', code3: 'Cys', name: 'Cysteine', group: 'Polar', color: 'border-sky-300 bg-sky-50 text-sky-900', prop: 'Thiol (-SH) — forms covalent disulfide (-S-S-) bridges', structureHint: 'R = -CH₂-SH' },
      { code1: 'N', code3: 'Asn', name: 'Asparagine', group: 'Polar', color: 'border-sky-300 bg-sky-50 text-sky-900', prop: 'Short amide sidechain, caps helix ends (p53 Asn29)', structureHint: 'R = -CH₂-CONH₂' },
      { code1: 'Q', code3: 'Gln', name: 'Glutamine', group: 'Polar', color: 'border-sky-300 bg-sky-50 text-sky-900', prop: 'Longer polar amide, solvent & rim H-bonds', structureHint: 'R = -(CH₂)₂-CONH₂' },
    ],
  },
  {
    title: 'Charged: Acidic (−) & Basic (+) (5)',
    items: [
      { code1: 'D', code3: 'Asp', name: 'Aspartate (−)', group: 'Charged −', color: 'border-rose-300 bg-rose-50 text-rose-900', prop: 'Negative carboxylate (-COO⁻), forms salt bridges', structureHint: 'R = -CH₂-COO⁻' },
      { code1: 'E', code3: 'Glu', name: 'Glutamate (−)', group: 'Charged −', color: 'border-rose-300 bg-rose-50 text-rose-900', prop: 'Negative carboxylate, strong α-helix former', structureHint: 'R = -(CH₂)₂-COO⁻' },
      { code1: 'K', code3: 'Lys', name: 'Lysine (+)', group: 'Charged +', color: 'border-indigo-300 bg-indigo-50 text-indigo-900', prop: 'Positive primary amine (-NH₃⁺), electrostatic steering', structureHint: 'R = -(CH₂)₄-NH₃⁺' },
      { code1: 'R', code3: 'Arg', name: 'Arginine (+)', group: 'Charged +', color: 'border-indigo-300 bg-indigo-50 text-indigo-900', prop: 'Positive guanidinium group, bidentate salt bridges', structureHint: 'R = -(CH₂)₃-NHC(NH₂)₂⁺' },
      { code1: 'H', code3: 'His', name: 'Histidine (+/0)', group: 'Charged +', color: 'border-indigo-300 bg-indigo-50 text-indigo-900', prop: 'Imidazole ring (pKa ≈ 6.0): protonates near neutral pH & π-stacks', structureHint: 'R = -CH₂-Imidazole' },
    ],
  },
  {
    title: 'Special Backbone Geometry (2)',
    items: [
      { code1: 'G', code3: 'Gly', name: 'Glycine', group: 'Special', color: 'border-amber-300 bg-amber-50 text-amber-900', prop: 'No sidechain (R = H)! Minimal steric hindrance, ultra-flexible φ/ψ', structureHint: 'R = -H (Achiral)' },
      { code1: 'P', code3: 'Pro', name: 'Proline ⚠', group: 'Special', color: 'border-amber-600 bg-amber-100 text-amber-950 ring-1 ring-amber-500/40', prop: 'Sidechain loops back to N! Locks φ ≈ -65° → kinks/breaks α-helices', structureHint: 'Cyclic pyrrolidine ring' },
    ],
  },
]

const selectedAA = ref<AAInfo>(groups[1].items[1])
let syncChannel: BroadcastChannel | null = null

function selectAA(aa: AAInfo) {
  selectedAA.value = aa
  syncChannel?.postMessage({ code1: aa.code1 })
}

onMounted(() => {
  if (typeof BroadcastChannel !== 'undefined') {
    syncChannel = new BroadcastChannel('slidev-aa-primer-sync')
    syncChannel.onmessage = (ev) => {
      const code1 = ev.data?.code1
      if (!code1) return
      for (const grp of groups) {
        const found = grp.items.find(i => i.code1 === code1)
        if (found) {
          selectedAA.value = found
          break
        }
      }
    }
  }
})

onBeforeUnmount(() => {
  syncChannel?.close()
})
</script>

<template>
  <div class="grid grid-cols-12 gap-3 mt-1 select-none" style="height: 410px;">
    <!-- Left 6 Columns: Anatomy of an Amino Acid & Fixed vs Rotatable Angles -->
    <div class="col-span-6 rounded-xl bg-white border border-slate-200 shadow-sm p-3 flex flex-col justify-between">
      <div class="flex items-center justify-between">
        <span class="text-[10px] font-mono uppercase tracking-wider text-teal-800 font-bold">
          1. Polypeptide Geometry: Fixed Planes vs. Rotatable Hinges
        </span>
        <span class="px-2 py-0.5 rounded bg-amber-50 border border-amber-300 text-amber-800 text-[9px] font-mono font-bold">
          Why Folding = Choosing (φ, ψ)
        </span>
      </div>

      <!-- Clean High-Precision Light-Theme SVG Diagram with Scoped Explicit Pixel Font Sizes -->
      <div class="my-1 rounded-lg bg-slate-50 border border-slate-200 p-2 flex items-center justify-center">
        <svg viewBox="0 0 600 245" class="w-full h-[215px] aa-svg">
          <defs>
            <marker id="arr-cyan" viewBox="0 0 10 10" refX="6" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse">
              <path d="M 0 1 L 10 5 L 0 9 z" fill="#0284c7" />
            </marker>
            <marker id="arr-teal" viewBox="0 0 10 10" refX="6" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse">
              <path d="M 0 1 L 10 5 L 0 9 z" fill="#0d9488" />
            </marker>
            <marker id="arr-purple" viewBox="0 0 10 10" refX="6" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse">
              <path d="M 0 1 L 10 5 L 0 9 z" fill="#7c3aed" />
            </marker>
          </defs>

          <!-- Rigid Peptide Plane 1 (Left, C_{i-1} - N_i) -->
          <polygon
            points="18,95 175,95 162,188 6,188"
            fill="rgba(251, 191, 36, 0.14)"
            stroke="#d97706"
            stroke-width="1.5"
            stroke-dasharray="4 3"
          />
          <text x="20" y="110" fill="#b45309" class="svg-label-bold">
            PEPTIDE PLANE (ω = 180° FIXED)
          </text>

          <!-- Rigid Peptide Plane 2 (Right, C_i - N_{i+1}) -->
          <polygon
            points="415,95 588,95 575,188 402,188"
            fill="rgba(251, 191, 36, 0.14)"
            stroke="#d97706"
            stroke-width="1.5"
            stroke-dasharray="4 3"
          />
          <text x="418" y="110" fill="#b45309" class="svg-label-bold">
            PEPTIDE PLANE (ω = 180° FIXED)
          </text>

          <!-- C_{i-1} to N_i (omega bond - rigid double-bond character) -->
          <line x1="62" y1="142" x2="148" y2="142" stroke="#d97706" stroke-width="5" />
          <line x1="62" y1="134" x2="148" y2="134" stroke="#d97706" stroke-width="2" stroke-dasharray="3 3" />
          <text x="80" y="160" fill="#92400e" class="svg-label-bold">ω = 180°</text>
          <text x="72" y="173" fill="#dc2626" class="svg-label-xs">(Rigid / Locked)</text>

          <!-- C_{i-1} = O -->
          <line x1="58" y1="142" x2="40" y2="174" stroke="#dc2626" stroke-width="3" />
          <circle cx="37" cy="177" r="9" fill="#fee2e2" stroke="#dc2626" stroke-width="1.8" />
          <text x="33" y="181" fill="#991b1b" class="svg-label-xs">O</text>

          <!-- N_i - H -->
          <line x1="148" y1="142" x2="162" y2="112" stroke="#2563eb" stroke-width="2.5" />
          <circle cx="165" cy="108" r="8" fill="#dbeafe" stroke="#2563eb" stroke-width="1.5" />
          <text x="161" y="112" fill="#1e40af" class="svg-label-xs">H</text>

          <!-- N_i to CA_i (PHI rotatable bond!) -->
          <line x1="148" y1="142" x2="290" y2="142" stroke="#0284c7" stroke-width="4" />
          <path
            d="M 210,120 C 228,104 238,180 210,164"
            fill="none"
            stroke="#0284c7"
            stroke-width="2.2"
            marker-end="url(#arr-cyan)"
          />
          <text x="182" y="80" fill="#0369a1" class="svg-label-title">
            φ (Phi) ROTATABLE
          </text>
          <text x="186" y="93" fill="#0284c7" class="svg-label-xs">
            around N–Cα bond
          </text>

          <!-- CA_i to C_i (PSI rotatable bond!) -->
          <line x1="290" y1="142" x2="435" y2="142" stroke="#0d9488" stroke-width="4" />
          <path
            d="M 355,120 C 373,104 383,180 355,164"
            fill="none"
            stroke="#0d9488"
            stroke-width="2.2"
            marker-end="url(#arr-teal)"
          />
          <text x="328" y="80" fill="#0f766e" class="svg-label-title">
            ψ (Psi) ROTATABLE
          </text>
          <text x="332" y="93" fill="#0d9488" class="svg-label-xs">
            around Cα–C bond
          </text>

          <!-- Fixed Tetrahedral Bond Angle at Cα (109.5°) -->
          <path d="M 266,142 A 24,24 0 0,1 290,118" fill="none" stroke="#b45309" stroke-width="1.5" stroke-dasharray="2 2" />
          <text x="234" y="182" fill="#92400e" class="svg-label-xs">∠ 109.5° fixed tetrahedral</text>

          <!-- CA_i to Sidechain R (CHI rotatable bond!) -->
          <line x1="290" y1="142" x2="290" y2="52" stroke="#7c3aed" stroke-width="3.5" />
          <path
            d="M 272,90 C 262,76 318,76 308,90"
            fill="none"
            stroke="#7c3aed"
            stroke-width="2"
            marker-end="url(#arr-purple)"
          />
          <text x="314" y="65" fill="#6d28d9" class="svg-label-bold">
            χ₁ (Chi)
          </text>

          <!-- Sidechain R-group Box -->
          <rect x="225" y="8" width="130" height="38" rx="8" fill="#f3e8ff" stroke="#7c3aed" stroke-width="2" />
          <text x="238" y="24" fill="#581c87" class="svg-label-bold">
            SIDECHAIN (R-Group)
          </text>
          <text x="244" y="38" fill="#6b21a8" class="svg-label-xs">
            20 distinct chemistries
          </text>

          <!-- C_i to N_{i+1} (omega bond) -->
          <line x1="435" y1="142" x2="535" y2="142" stroke="#d97706" stroke-width="5" />
          <line x1="435" y1="134" x2="535" y2="134" stroke="#d97706" stroke-width="2" stroke-dasharray="3 3" />
          <text x="462" y="160" fill="#92400e" class="svg-label-bold">ω = 180°</text>
          <text x="454" y="173" fill="#dc2626" class="svg-label-xs">(Rigid / Locked)</text>

          <!-- C_i = O -->
          <line x1="435" y1="142" x2="416" y2="174" stroke="#dc2626" stroke-width="3" />
          <circle cx="413" cy="177" r="9" fill="#fee2e2" stroke="#dc2626" stroke-width="1.8" />
          <text x="409" y="181" fill="#991b1b" class="svg-label-xs">O</text>

          <!-- Atom Spheres -->
          <circle cx="62" cy="142" r="14" fill="#334155" stroke="#0f172a" stroke-width="2" />
          <text x="50" y="146" fill="#ffffff" class="svg-label-bold">Cᵢ₋₁</text>

          <circle cx="148" cy="142" r="15" fill="#1d4ed8" stroke="#1e3a8a" stroke-width="2" />
          <text x="141" y="146" fill="#ffffff" class="svg-label-bold">Nᵢ</text>

          <circle cx="290" cy="142" r="17" fill="#0d9488" stroke="#115e59" stroke-width="2.5" />
          <text x="281" y="146" fill="#ffffff" class="svg-label-title">Cα</text>

          <circle cx="435" cy="142" r="15" fill="#334155" stroke="#0f172a" stroke-width="2" />
          <text x="428" y="146" fill="#ffffff" class="svg-label-bold">Cᵢ</text>

          <circle cx="535" cy="142" r="15" fill="#1d4ed8" stroke="#1e3a8a" stroke-width="2" />
          <text x="524" y="146" fill="#ffffff" class="svg-label-bold">Nᵢ₊₁</text>

          <!-- Bottom Legend inside SVG -->
          <text x="16" y="216" fill="#334155" class="svg-label-sm">
            • <tspan fill="#b45309" font-weight="bold">Fixed Angles:</tspan> Planar peptide bond (ω = 180°) &amp; tetrahedral Cα angle (109.5°) cannot rotate.
          </text>
          <text x="16" y="234" fill="#334155" class="svg-label-sm">
            • <tspan fill="#0369a1" font-weight="bold">Rotatable Hinges (φ, ψ):</tspan> α-Helix forms at (φ ≈ −60°, ψ ≈ −45°); β-Sheet at (φ ≈ −135°, ψ ≈ +135°).
          </text>
        </svg>
      </div>

      <!-- Bottom Takeaway Box (Clean HTML/Unicode, no raw LaTeX $) -->
      <div class="grid grid-cols-2 gap-2 text-[10px] font-mono">
        <div class="p-2 rounded-lg bg-amber-50 border border-amber-300 text-amber-950 leading-snug">
          <b>Rigid Constraints:</b> Because <b>ω = 180°</b> is locked in a flat plane, the backbone only twists at <b>C<sub>α</sub></b> hinges (<b>φ, ψ</b>)!
        </div>
        <div class="p-2 rounded-lg bg-cyan-50 border border-cyan-300 text-cyan-950 leading-snug">
          <b>Proline Exception (Pro):</b> Sidechain loops back to <b>N<sub>i</sub></b>, locking <b>φ ≈ −65°</b> and kinking <b>α-helices</b>!
        </div>
      </div>
    </div>

    <!-- Right 6 Columns: All 20 Standard Amino Acids Grouped by Physics -->
    <div class="col-span-6 rounded-xl bg-white border border-slate-200 shadow-sm p-3 flex flex-col justify-between">
      <div>
        <div class="flex items-center justify-between mb-2">
          <span class="text-[10px] font-mono uppercase tracking-wider text-cyan-800 font-bold">
            2. The 20 Standard Amino Acids (Sidechain R-Groups)
          </span>
          <span class="text-[9px] font-mono text-slate-500">
            Click any AA to inspect
          </span>
        </div>

        <!-- 5 Physicochemical Categories -->
        <div class="space-y-1.5">
          <div v-for="(grp, gIdx) in groups" :key="gIdx">
            <div class="text-[9px] font-mono font-bold text-slate-600 mb-0.5 flex items-center justify-between">
              <span>{{ grp.title }}</span>
            </div>
            <div class="grid grid-cols-5 gap-1.5">
              <button
                v-for="aa in grp.items"
                :key="aa.code1"
                @click="selectAA(aa)"
                class="px-2 py-1 rounded-lg border text-left transition-all cursor-pointer"
                :class="[
                  aa.color,
                  selectedAA.code1 === aa.code1 ? 'ring-2 ring-amber-500 scale-[1.03] shadow-sm' : 'opacity-95 hover:opacity-100',
                ]"
              >
                <div class="flex items-baseline justify-between font-mono">
                  <span class="text-xs font-extrabold">{{ aa.code1 }}</span>
                  <span class="text-[9px] opacity-80">{{ aa.code3 }}</span>
                </div>
                <div class="text-[8px] truncate opacity-90 leading-tight">{{ aa.name }}</div>
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Selected Amino Acid Detail Bar -->
      <div class="mt-1.5 p-2 rounded-lg bg-amber-50/90 border border-amber-300 flex items-center justify-between gap-2 font-mono">
        <div>
          <div class="text-xs font-bold text-amber-950">
            {{ selectedAA.name }} ({{ selectedAA.code3 }} / '{{ selectedAA.code1 }}') — <span class="text-teal-800">{{ selectedAA.structureHint }}</span>
          </div>
          <div class="text-[10px] text-slate-700 mt-0.5">
            {{ selectedAA.prop }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.aa-svg text {
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace !important;
}
.aa-svg .svg-label-xs {
  font-size: 9.5px !important;
}
.aa-svg .svg-label-sm {
  font-size: 10.5px !important;
}
.aa-svg .svg-label-bold {
  font-size: 10.5px !important;
  font-weight: 700 !important;
}
.aa-svg .svg-label-title {
  font-size: 11.5px !important;
  font-weight: 700 !important;
}
</style>
