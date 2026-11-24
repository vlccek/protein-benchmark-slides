<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount, watch, nextTick } from 'vue'
import { useSlideContext } from '@slidev/client'

const props = withDefaults(
  defineProps<{
    mode?: 'folding' | 'docking'
  }>(),
  {
    mode: 'folding',
  }
)

const { $clicks, $page, $nav, $renderContext } = useSlideContext()

const isFoldingMode = computed(() => props.mode === 'folding')
const minFrame = computed(() => (isFoldingMode.value ? 0 : 12))
const maxFrame = computed(() => (isFoldingMode.value ? 12 : 20))

const step = ref(0)
const currentFrame = ref(props.mode === 'folding' ? 0 : 12)
const isPlaying = ref(false)
const reprMode = ref<'hybrid' | 'sticks' | 'backbone' | 'spheres' | 'surface'>('hybrid')
const showSurface = ref(true)
const showPeptideSurface = ref(false)
const showLabels = ref(true)
const selectedResi = ref<number | null>(null)
const isLoaded = ref(false)
const loadError = ref('')

const viewerContainer = ref<HTMLElement | null>(null)
let $3Dmol: any = null
let viewer: any = null
let mdm2Model: any = null
let p53Model: any = null
let animTimer: any = null
let resizeObserver: ResizeObserver | null = null
let lastWidth = 0
let demoSyncChannel: BroadcastChannel | null = null
let isReceivingDemoSync = false
let isPointerDown = false

// 13-AA p53 transactivation peptide (PDB: 1YCR, Chain B, residues 17..29)
const sequence = [
  { resi: 17, aa: 'E', mut: 'L', anchor: false, desc: 'Glu17 — Charged N-terminal cap (mutated to Leu17 in PDI)' },
  { resi: 18, aa: 'T', mut: 'T', anchor: false, desc: 'Thr18 — Polar H-bond donor stabilizing the first helical turn' },
  { resi: 19, aa: 'F', mut: 'F', anchor: true, desc: 'Phe19 ★ — Anchor 1 (Aromatic ring): plugs into MDM2 Tyr67/Gln72 sub-pocket' },
  { resi: 20, aa: 'S', mut: 'E', anchor: false, desc: 'Ser20 — Solvent-exposed residue (mutated to Glu20 to boost helicity)' },
  { resi: 21, aa: 'D', mut: 'H', anchor: false, desc: 'Asp21 — Acidic sidechain (mutated to His21 in PDI)' },
  { resi: 22, aa: 'L', mut: 'Y', anchor: false, desc: 'Leu22 — Mutated to Tyr22 in PDI → adds π-stacking with MDM2 His96/Tyr67' },
  { resi: 23, aa: 'W', mut: 'W', anchor: true, desc: 'Trp23 ★ — Anchor 2 (Bulky double-ring indole): buries deep into Leu54/Ile61/Val93 pocket' },
  { resi: 24, aa: 'K', mut: 'A', anchor: false, desc: 'Lys24 — Mutated to Ala24 in PDI to maximize α-helical propensity' },
  { resi: 25, aa: 'L', mut: 'Q', anchor: false, desc: 'Leu25 — Flank residue (mutated to Gln25 in PDI)' },
  { resi: 26, aa: 'L', mut: 'L', anchor: true, desc: 'Leu26 ★ — Anchor 3 (Aliphatic branch): locks into MDM2 His96/Ile99 sub-pocket' },
  { resi: 27, aa: 'P', mut: 'T', anchor: false, desc: 'Pro27 ⚠ — Helix-breaker ring! Kinks WT p53 C-terminus; mutated to Thr27 in PDI' },
  { resi: 28, aa: 'E', mut: 'S', anchor: false, desc: 'Glu28 — Flexible C-terminal tail residue' },
  { resi: 29, aa: 'N', mut: 'N', anchor: false, desc: 'Asn29 — C-terminal end of the 13-AA peptide' },
]

const foldingStages = [
  {
    badge: '1. Unfolded 1D chain',
    title: 'Disordered 13-AA chain',
    targetFrame: 0,
    desc: 'Extended chain in water; hydrophobic anchors point in different directions.',
    metric: 'Unfolded span: 40.6 Å',
    tupleVars: [
      { k: 'Chain', v: '13 AA (Glu17 → Asn29)' },
      { k: 'Anchors', v: 'Phe19, Trp23, Leu26 unaligned' },
    ],
  },
  {
    badge: '2. Folded α-helix',
    title: 'Coiling aligns the anchors',
    targetFrame: 12,
    desc: 'Backbone rotation coils the chain into a helix, placing all 3 anchors on one face.',
    metric: 'Helix span: 23.2 Å',
    tupleVars: [
      { k: '3D face', v: 'Phe19, Trp23, Leu26 aligned' },
      { k: 'C-tail', v: 'Pro27 kinks the end' },
    ],
  },
]

const dockingStages = [
  {
    badge: '1. Target & pocket',
    title: 'MDM2 binding pocket',
    targetFrame: 12,
    desc: 'Folded 13-AA binder poised above the orange hydrophobic cleft.',
    metric: 'Unbound (11.5 Å above)',
    tupleVars: [
      { k: 'Target', v: 'MDM2' },
      { k: 'Pocket', v: 'Hydrophobic cleft (54–99)' },
    ],
  },
  {
    badge: '2. Dock & flaw',
    title: 'WT docked — Pro27 kink',
    targetFrame: 20,
    desc: 'Three anchors plug in, but Pro27 kinks the helix end.',
    metric: 'WT: Kd = 600 nM',
    tupleVars: [
      { k: 'Anchors', v: 'Phe19, Trp23, Leu26' },
      { k: 'Flaw', v: 'Pro27 kink (entropy cost)' },
    ],
  },
  {
    badge: '3. Failed edits',
    title: 'Dead-end mutations',
    targetFrame: 20,
    desc: 'Naive edits either leave a cavity or clash with the pocket wall.',
    metric: 'Fail: Kd > 50,000 nM',
    tupleVars: [
      { k: 'W23A', v: 'Empty cavity' },
      { k: 'L26W', v: 'Steric clash (Ile99)' },
    ],
  },
  {
    badge: '4. Engineered',
    title: 'Super-binder (PDI)',
    targetFrame: 20,
    desc: 'Removes the Pro27 kink and adds rim π-stacking.',
    metric: 'Kd = 0.9 nM (650× gain)',
    tupleVars: [
      { k: 'Edits', v: 'P27T + L22Y + K24A' },
      { k: 'Why', v: 'Rigid helix + Tyr22 stack' },
    ],
  },
]

const stages = computed(() => (isFoldingMode.value ? foldingStages : dockingStages))
const maxStep = computed(() => stages.value.length - 1)

// Semantic state helpers across both modes
const isDeadEndStep = computed(() => !isFoldingMode.value && step.value === 2)
const isEngineeredStep = computed(() => !isFoldingMode.value && step.value === 3)

const pocketResidues = [54, 57, 58, 61, 62, 67, 72, 75, 91, 93, 96, 99]

function generatePdbData(rawPdb: string) {
  const mdm2HeaderLines: string[] = []
  interface AtomRec {
    line: string
    name: string
    resi: number
    xyz: [number, number, number]
  }
  const atomsA: AtomRec[] = []
  const atomsB: AtomRec[] = []
  const caA: [number, number, number][] = []
  const caB: [number, number, number][] = []

  for (const line of rawPdb.split('\n')) {
    if (line.startsWith('HELIX') || line.startsWith('SHEET')) {
      if (line.includes(' A ')) mdm2HeaderLines.push(line)
    } else if (line.startsWith('ATOM')) {
      const chain = line[21]
      const name = line.slice(12, 16).trim()
      const resi = parseInt(line.slice(22, 26), 10)
      const x = parseFloat(line.slice(30, 38))
      const y = parseFloat(line.slice(38, 46))
      const z = parseFloat(line.slice(46, 54))
      if (chain === 'A') {
        atomsA.push({ line, name, resi, xyz: [x, y, z] })
        if (name === 'CA') caA.push([x, y, z])
      } else if (chain === 'B') {
        atomsB.push({ line, name, resi, xyz: [x, y, z] })
        if (name === 'CA') caB.push([x, y, z])
      }
    }
  }

  const centerA = [0, 0, 0]
  caA.forEach(p => { centerA[0] += p[0]; centerA[1] += p[1]; centerA[2] += p[2] })
  centerA[0] /= caA.length; centerA[1] /= caA.length; centerA[2] /= caA.length

  const centerB = [0, 0, 0]
  caB.forEach(p => { centerB[0] += p[0]; centerB[1] += p[1]; centerB[2] += p[2] })
  centerB[0] /= caB.length; centerB[1] /= caB.length; centerB[2] /= caB.length

  const ex = [caB[caB.length - 1][0] - caB[0][0], caB[caB.length - 1][1] - caB[0][1], caB[caB.length - 1][2] - caB[0][2]]
  const exLen = Math.hypot(ex[0], ex[1], ex[2]) || 1
  ex[0] /= exLen; ex[1] /= exLen; ex[2] /= exLen

  const outRaw = [centerB[0] - centerA[0], centerB[1] - centerA[1], centerB[2] - centerA[2]]
  const dotOutEx = outRaw[0] * ex[0] + outRaw[1] * ex[1] + outRaw[2] * ex[2]
  const ey = [outRaw[0] - dotOutEx * ex[0], outRaw[1] - dotOutEx * ex[1], outRaw[2] - dotOutEx * ex[2]]
  const eyLen = Math.hypot(ey[0], ey[1], ey[2]) || 1
  ey[0] /= eyLen; ey[1] /= eyLen; ey[2] /= eyLen

  const ez = [
    ex[1] * ey[2] - ex[2] * ey[1],
    ex[2] * ey[0] - ex[0] * ey[2],
    ex[0] * ey[1] - ex[1] * ey[0],
  ]

  const toCameraFrame = (pt: [number, number, number]): [number, number, number] => {
    const dx = pt[0] - centerB[0]
    const dy = pt[1] - centerB[1]
    const dz = pt[2] - centerB[2]
    return [
      dx * ex[0] + dy * ex[1] + dz * ex[2],
      dx * ey[0] + dy * ey[1] + dz * ey[2],
      dx * ez[0] + dy * ez[1] + dz * ez[2],
    ]
  }

  const mdm2Lines = [...mdm2HeaderLines]
  for (const a of atomsA) {
    const [rx, ry, rz] = toCameraFrame(a.xyz)
    const x = rx.toFixed(3).padStart(8, ' ')
    const y = ry.toFixed(3).padStart(8, ' ')
    const z = rz.toFixed(3).padStart(8, ' ')
    mdm2Lines.push(`${a.line.slice(0, 30)}${x}${y}${z}${a.line.slice(54)}`)
  }
  mdm2Lines.push('END')

  const atomsBAligned = atomsB.map(a => ({
    ...a,
    xyz: toCameraFrame(a.xyz),
  }))

  const rotAxis = (
    pts: [number, number, number][],
    indices: number[],
    origin: [number, number, number],
    axis: [number, number, number],
    angleDeg: number
  ) => {
    const len = Math.hypot(axis[0], axis[1], axis[2])
    if (len < 1e-6) return
    const ux = axis[0] / len, uy = axis[1] / len, uz = axis[2] / len
    const rad = (angleDeg * Math.PI) / 180
    const c = Math.cos(rad), s = Math.sin(rad)
    for (const idx of indices) {
      const vx = pts[idx][0] - origin[0]
      const vy = pts[idx][1] - origin[1]
      const vz = pts[idx][2] - origin[2]
      const dot = vx * ux + vy * uy + vz * uz
      const cx = uy * vz - uz * vy
      const cy = uz * vx - ux * vz
      const cz = ux * vy - uy * vx
      pts[idx] = [
        origin[0] + vx * c + cx * s + ux * dot * (1 - c),
        origin[1] + vy * c + cy * s + uy * dot * (1 - c),
        origin[2] + vz * c + cz * s + uz * dot * (1 - c),
      ]
    }
  }

  const deltas: Record<number, [number, number]> = {
    18: [-42.1, -18.8], 19: [-81.8, 199.3], 20: [-80.7, 182.2],
    21: [-67.2, 192.4], 22: [-71.5, 189.0], 23: [-66.5, 171.9],
    24: [-58.5, 154.8], 25: [-45.9, 126.4], 26: [-40.9, -3.2],
    27: [-80.9, 28.0],  28: [-51.4, 140.1],
  }

  const caIndices = atomsBAligned.map((a, i) => (a.name === 'CA' ? i : -1)).filter(i => i >= 0)

  const trajLines: string[] = []
  for (let frame = 0; frame <= 20; frame++) {
    const tFoldRaw = Math.min(1.0, frame / 12.0)
    const tFold = tFoldRaw * tFoldRaw * (3 - 2 * tFoldRaw)
    const scale = 1.0 - tFold

    let tDock = 0
    if (frame <= 12) {
      tDock = 0.15 * (frame / 12.0)
    } else {
      const s = (frame - 12.0) / 8.0
      tDock = 0.15 + 0.85 * (s * s * (3 - 2 * s))
    }

    const xyz: [number, number, number][] = atomsBAligned.map(a => [a.xyz[0], a.xyz[1], a.xyz[2]])

    for (let r = 23; r <= 28; r++) {
      const [dphi, dpsi] = deltas[r]
      const iN = atomsBAligned.findIndex(a => a.resi === r && a.name === 'N')
      const iCA = atomsBAligned.findIndex(a => a.resi === r && a.name === 'CA')
      const iC = atomsBAligned.findIndex(a => a.resi === r && a.name === 'C')
      if (r > 23) {
        const mPhi = atomsBAligned.map((a, i) => ((a.resi > r || (a.resi === r && a.name !== 'N' && a.name !== 'CA')) ? i : -1)).filter(i => i >= 0)
        rotAxis(xyz, mPhi, xyz[iCA], [xyz[iCA][0] - xyz[iN][0], xyz[iCA][1] - xyz[iN][1], xyz[iCA][2] - xyz[iN][2]], scale * dphi)
      }
      const mPsi = atomsBAligned.map((a, i) => ((a.resi > r || (a.resi === r && a.name === 'O')) ? i : -1)).filter(i => i >= 0)
      rotAxis(xyz, mPsi, xyz[iC], [xyz[iC][0] - xyz[iCA][0], xyz[iC][1] - xyz[iCA][1], xyz[iC][2] - xyz[iCA][2]], scale * dpsi)
    }

    for (let r = 22; r >= 18; r--) {
      const [dphi, dpsi] = deltas[r]
      const iN = atomsBAligned.findIndex(a => a.resi === r && a.name === 'N')
      const iCA = atomsBAligned.findIndex(a => a.resi === r && a.name === 'CA')
      const iC = atomsBAligned.findIndex(a => a.resi === r && a.name === 'C')
      const mPsi = atomsBAligned.map((a, i) => ((a.resi < r || (a.resi === r && a.name !== 'O' && a.name !== 'C')) ? i : -1)).filter(i => i >= 0)
      rotAxis(xyz, mPsi, xyz[iCA], [xyz[iCA][0] - xyz[iC][0], xyz[iCA][1] - xyz[iC][1], xyz[iCA][2] - xyz[iC][2]], scale * dpsi)
      const mPhi = atomsBAligned.map((a, i) => ((a.resi < r || (a.resi === r && a.name === 'N')) ? i : -1)).filter(i => i >= 0)
      rotAxis(xyz, mPhi, xyz[iN], [xyz[iN][0] - xyz[iCA][0], xyz[iN][1] - xyz[iCA][1], xyz[iN][2] - xyz[iCA][2]], scale * dphi)
    }

    const pFirst = xyz[caIndices[0]]
    const pLast = xyz[caIndices[caIndices.length - 1]]
    const curVec: [number, number, number] = [pLast[0] - pFirst[0], pLast[1] - pFirst[1], pLast[2] - pFirst[2]]
    const curLen = Math.hypot(curVec[0], curVec[1], curVec[2]) || 1
    curVec[0] /= curLen; curVec[1] /= curLen; curVec[2] /= curLen
    const crossAxis: [number, number, number] = [0, curVec[2], -curVec[1]]
    const crossLen = Math.hypot(crossAxis[0], crossAxis[1], crossAxis[2])
    if (crossLen > 1e-5) {
      const angleDeg = (Math.acos(Math.max(-1, Math.min(1, curVec[0]))) * 180) / Math.PI
      const allIdx = atomsBAligned.map((_, i) => i)
      const w23Idx = atomsBAligned.findIndex(a => a.resi === 23 && a.name === 'CA')
      rotAxis(xyz, allIdx, xyz[w23Idx], crossAxis, angleDeg)
    }

    const shiftY = isFoldingMode.value ? 0 : (1.0 - tDock) * 11.5
    trajLines.push(`MODEL        ${frame + 1}`)
    for (let i = 0; i < atomsBAligned.length; i++) {
      const x = xyz[i][0].toFixed(3).padStart(8, ' ')
      const y = (xyz[i][1] + shiftY).toFixed(3).padStart(8, ' ')
      const z = xyz[i][2].toFixed(3).padStart(8, ' ')
      const base = atomsBAligned[i].line
      trajLines.push(`${base.slice(0, 30)}${x}${y}${z}${base.slice(54)}`)
    }
    trajLines.push('ENDMDL')
  }
  trajLines.push('END')

  return {
    mdm2Pdb: mdm2Lines.join('\n'),
    p53TrajPdb: trajLines.join('\n'),
  }
}

function focusCamera() {
  if (!viewer || !p53Model) return
  viewer.resize()
  if (isFoldingMode.value || !mdm2Model) {
    viewer.zoomTo({ model: p53Model })
    viewer.zoom(currentFrame.value < 6 ? 1.15 : 1.32)
  } else {
    // Frame both the pocket and the peptide (which hovers above the pocket in step 1)
    viewer.zoomTo({ or: [{ model: mdm2Model, resi: pocketResidues }, { model: p53Model }] })
    viewer.zoom(1.05)
  }
  viewer.render()
}

function applyStyles() {
  if (!viewer || !p53Model || !$3Dmol) return

  viewer.removeAllShapes()
  viewer.removeAllLabels()
  viewer.removeAllSurfaces()

  // Style MDM2 Target Receptor ONLY in 'docking' mode (Light-Theme Optimized Colors)
  if (mdm2Model && !isFoldingMode.value) {
    mdm2Model.setStyle({}, { cartoon: { color: '#64748b', opacity: 0.88 } })
    mdm2Model.setStyle(
      { resi: pocketResidues },
      {
        cartoon: { color: '#f59e0b', opacity: 0.95 },
        stick: { radius: 0.16, colorscheme: 'orangeCarbon', opacity: 0.92 },
      }
    )

    if (showSurface.value) {
      viewer.addSurface(
        $3Dmol.SurfaceType.VDW,
        { opacity: 0.55, color: '#cbd5e1' },
        { model: mdm2Model },
        { model: mdm2Model }
      )
      viewer.addSurface(
        $3Dmol.SurfaceType.VDW,
        { opacity: 0.72, color: '#f59e0b' },
        { model: mdm2Model, resi: pocketResidues },
        { model: mdm2Model, resi: pocketResidues }
      )
    }
  }

  const mainColor = isDeadEndStep.value
    ? '#e11d48'
    : isEngineeredStep.value
      ? '#059669'
      : '#0891b2'

  if (reprMode.value === 'hybrid') {
    const useRibbon = currentFrame.value >= 7
    p53Model.setStyle(
      {},
      {
        cartoon: useRibbon ? { color: mainColor, thickness: 0.48 } : undefined,
        stick: { radius: useRibbon ? 0.2 : 0.28, colorscheme: 'cyanCarbon' },
      }
    )
    p53Model.setStyle(
      { resi: [19, 23, 26] },
      {
        cartoon: useRibbon ? { color: mainColor, thickness: 0.5 } : undefined,
        stick: { radius: 0.35, colorscheme: isDeadEndStep.value ? 'redCarbon' : 'cyanCarbon' },
        sphere: { radius: 0.5, color: isDeadEndStep.value ? '#e11d48' : '#0284c7', opacity: 0.92 },
      }
    )
    p53Model.setStyle(
      { resi: [27] },
      {
        cartoon: useRibbon ? { color: isEngineeredStep.value ? '#059669' : '#d97706' } : undefined,
        stick: { radius: 0.32, color: isEngineeredStep.value ? '#059669' : '#d97706' },
      }
    )
  } else if (reprMode.value === 'sticks') {
    p53Model.setStyle({}, { stick: { radius: 0.3, colorscheme: 'cyanCarbon' } })
    p53Model.setStyle({ resi: [19, 23, 26] }, { stick: { radius: 0.4, colorscheme: 'orangeCarbon' } })
  } else if (reprMode.value === 'backbone') {
    p53Model.setStyle(
      {},
      {
        line: { linewidth: 4, color: mainColor },
        sphere: { atoms: 'CA', radius: 0.68, color: mainColor },
      }
    )
    p53Model.setStyle(
      { resi: [19, 23, 26], atom: 'CA' },
      { sphere: { radius: 0.95, color: '#d97706' } }
    )
  } else if (reprMode.value === 'spheres') {
    p53Model.setStyle({}, { sphere: { scale: 0.82, colorscheme: 'cyanCarbon' } })
    p53Model.setStyle({ resi: [19, 23, 26] }, { sphere: { scale: 0.96, color: '#d97706' } })
  } else if (reprMode.value === 'surface') {
    const useRibbon = currentFrame.value >= 7
    p53Model.setStyle(
      {},
      {
        cartoon: useRibbon ? { color: mainColor, thickness: 0.45 } : undefined,
        stick: { radius: 0.22, colorscheme: 'cyanCarbon' },
      }
    )
    p53Model.setStyle(
      { resi: [19, 23, 26] },
      {
        stick: { radius: 0.35, colorscheme: isDeadEndStep.value ? 'redCarbon' : 'orangeCarbon' },
      }
    )
  }

  // Optional 3D Molecular Surface / Mesh ("Plocha") around the 13-AA Peptide
  if (reprMode.value === 'surface' || showPeptideSurface.value) {
    viewer.addSurface(
      $3Dmol.SurfaceType.MS,
      { opacity: 0.58, color: mainColor },
      { model: p53Model },
      { model: p53Model }
    )
    viewer.addSurface(
      $3Dmol.SurfaceType.MS,
      { opacity: 0.78, color: isDeadEndStep.value ? '#e11d48' : '#0284c7' },
      { model: p53Model, resi: [19, 23, 26] },
      { model: p53Model }
    )
  }

  if (selectedResi.value !== null) {
    p53Model.addStyle(
      { resi: selectedResi.value },
      { sphere: { radius: 0.78, color: '#f59e0b', opacity: 0.95 } }
    )
  }

  if (showLabels.value) {
    const labelStyle = {
      fontSize: 11,
      fontColor: '#0f172a',
      backgroundColor: '#ffffff',
      backgroundOpacity: 0.92,
      borderThickness: 1.2,
      borderColor: '#0284c7',
    }
    viewer.addLabel('Phe19 ★', { ...labelStyle }, { model: p53Model, resi: 19, atom: 'CA' })
    viewer.addLabel(
      isDeadEndStep.value ? 'Trp23→Ala (Cavity!)' : 'Trp23 ★',
      { ...labelStyle, borderColor: isDeadEndStep.value ? '#e11d48' : '#0284c7' },
      { model: p53Model, resi: 23, atom: 'CA' }
    )
    viewer.addLabel(
      isDeadEndStep.value ? 'Leu26→Trp (Clash!)' : 'Leu26 ★',
      { ...labelStyle, borderColor: isDeadEndStep.value ? '#e11d48' : '#0284c7' },
      { model: p53Model, resi: 26, atom: 'CA' }
    )
    viewer.addLabel(
      isEngineeredStep.value ? 'P27T (Helix Fixed!)' : 'Pro27 (Kink)',
      { ...labelStyle, borderColor: isEngineeredStep.value ? '#059669' : '#d97706' },
      { model: p53Model, resi: 27, atom: 'CA' }
    )

    if (mdm2Model && !isFoldingMode.value) {
      viewer.addLabel(
        'MDM2 Pocket',
        { fontSize: 11, fontColor: '#78350f', backgroundColor: '#fef3c7', backgroundOpacity: 0.94, borderThickness: 1, borderColor: '#d97706' },
        { model: mdm2Model, resi: 62, atom: 'CA' }
      )
    }
  }

  // Dead-End Mutants (H_neg): 3D Steric Clash & Cavity Spheres
  if (isDeadEndStep.value && currentFrame.value >= 18) {
    viewer.addSphere({
      center: { x: 3.03, y: -2.15, z: 2.43 },
      radius: 2.8,
      color: '#e11d48',
      opacity: 0.65,
      wireframe: true,
    })
    viewer.addSphere({
      center: { x: 2.97, y: -7.02, z: 0.63 },
      radius: 2.6,
      color: '#f43f5e',
      opacity: 0.45,
    })
  }

  // Engineered Super-Binder (Δ, R -> E): π-stacking & H-bond network cylinders
  if (isEngineeredStep.value && currentFrame.value >= 18) {
    viewer.addCylinder({
      start: { x: -5.46, y: -1.05, z: 2.88 },
      end: { x: 1.11, y: 1.77, z: 6.41 },
      radius: 0.22,
      color: '#059669',
      dashed: true,
    })
    viewer.addCylinder({
      start: { x: 2.97, y: -7.02, z: 0.63 },
      end: { x: -1.74, y: -4.64, z: 3.91 },
      radius: 0.22,
      color: '#10b981',
      dashed: true,
    })
    viewer.addSphere({
      center: { x: -5.46, y: -1.05, z: 2.88 },
      radius: 1.4,
      color: '#059669',
      opacity: 0.65,
    })
  }

  viewer.render()
}

function animateToFrame(target: number) {
  if (animTimer) clearInterval(animTimer)
  isPlaying.value = true
  applyStyles()
  animTimer = setInterval(() => {
    if (currentFrame.value === target) {
      clearInterval(animTimer)
      animTimer = null
      isPlaying.value = false
      applyStyles()
      focusCamera()
      return
    }
    const dir = target > currentFrame.value ? 1 : -1
    currentFrame.value += dir
    if (p53Model && viewer) {
      p53Model.setFrame(currentFrame.value)
      if (currentFrame.value === 7 || currentFrame.value === 12 || currentFrame.value === 20) {
        applyStyles()
      } else {
        viewer.render()
      }
    }
  }, 45)
}

function isPrimaryRenderContext() {
  const ctx = $renderContext?.value
  return !ctx || ctx === 'slide' || ctx === 'presenter'
}

function postDemoSync(payload: Record<string, any>) {
  if (isReceivingDemoSync || !isPrimaryRenderContext()) return
  demoSyncChannel?.postMessage({ mode: props.mode, ...payload })
}

function startPlayLoop() {
  if (animTimer) clearInterval(animTimer)
  const startF = minFrame.value
  const endF = maxFrame.value
  if (currentFrame.value >= endF) {
    currentFrame.value = startF
    if (p53Model) p53Model.setFrame(startF)
    applyStyles()
  }
  isPlaying.value = true
  animTimer = setInterval(() => {
    if (currentFrame.value >= endF) {
      clearInterval(animTimer)
      animTimer = null
      isPlaying.value = false
      applyStyles()
      return
    }
    currentFrame.value += 1
    if (p53Model && viewer) {
      p53Model.setFrame(currentFrame.value)
      if (currentFrame.value === 7 || currentFrame.value === 12 || currentFrame.value === 20) {
        applyStyles()
      } else {
        viewer.render()
      }
    }
  }, 55)
}

function togglePlayTrajectory() {
  if (isPlaying.value) {
    if (animTimer) clearInterval(animTimer)
    animTimer = null
    isPlaying.value = false
    postDemoSync({ type: 'play', playing: false, frame: currentFrame.value })
    return
  }
  startPlayLoop()
  postDemoSync({ type: 'play', playing: true, frame: currentFrame.value })
}

function onScrubFrame(e: Event) {
  const val = parseInt((e.target as HTMLInputElement).value, 10)
  if (animTimer) {
    clearInterval(animTimer)
    animTimer = null
    isPlaying.value = false
  }
  currentFrame.value = val
  if (p53Model && viewer) {
    p53Model.setFrame(val)
    applyStyles()
  }
  postDemoSync({ type: 'frame', frame: val })
}

function applyStage(idx: number) {
  const clamped = Math.max(0, Math.min(maxStep.value, idx))
  step.value = clamped
  const target = stages.value[clamped].targetFrame
  if (currentFrame.value !== target) {
    animateToFrame(target)
  } else {
    applyStyles()
    focusCamera()
  }
}

function selectStage(idx: number) {
  const clamped = Math.max(0, Math.min(maxStep.value, idx))
  if ($clicks && typeof $clicks.value === 'number' && $clicks.value !== clamped) {
    $clicks.value = clamped
  }
  if ($nav?.value && $page?.value && $nav.value.currentPage === $page.value) {
    $nav.value.go($page.value, clamped)
  }
  applyStage(clamped)
  postDemoSync({ type: 'stage', step: clamped })
}

function selectResidue(resi: number) {
  selectedResi.value = selectedResi.value === resi ? null : resi
  applyStyles()
  postDemoSync({ type: 'residue', resi: selectedResi.value })
}

watch(
  () => $clicks?.value,
  (newClick) => {
    if (typeof newClick === 'number') {
      const clamped = Math.max(0, Math.min(maxStep.value, newClick))
      if (step.value !== clamped) {
        applyStage(clamped)
      }
    }
  }
)

watch(
  () => $nav?.value?.currentPage === $page?.value,
  (isActive) => {
    if (isActive && viewer) {
      const c = typeof $clicks?.value === 'number' ? Math.max(0, Math.min(maxStep.value, $clicks.value)) : 0
      step.value = c
      currentFrame.value = stages.value[c].targetFrame
      if (p53Model) p53Model.setFrame(currentFrame.value)
      setTimeout(() => {
        applyStyles()
        focusCamera()
      }, 60)
    }
  }
)

watch([reprMode, showSurface, showPeptideSurface, showLabels], () => {
  applyStyles()
  postDemoSync({
    type: 'ui',
    reprMode: reprMode.value,
    showSurface: showSurface.value,
    showPeptideSurface: showPeptideSurface.value,
    showLabels: showLabels.value,
  })
})

const selectedResidueInfo = computed(() =>
  sequence.find(s => s.resi === selectedResi.value) || null
)

function syncCameraView() {
  if (!viewer || !isPrimaryRenderContext()) return
  try {
    const view = viewer.getView()
    if (Array.isArray(view)) {
      postDemoSync({ type: 'camera', view })
    }
  } catch {}
}

onMounted(async () => {
  if (typeof BroadcastChannel !== 'undefined' && isPrimaryRenderContext()) {
    demoSyncChannel = new BroadcastChannel(`slidev-protein-demo-${props.mode}`)
    demoSyncChannel.onmessage = (ev) => {
      const msg = ev.data
      if (!msg || msg.mode !== props.mode) return
      isReceivingDemoSync = true
      try {
        if (msg.type === 'stage' && typeof msg.step === 'number') {
          if (step.value !== msg.step) {
            applyStage(msg.step)
          }
        } else if (msg.type === 'frame' && typeof msg.frame === 'number') {
          if (animTimer) {
            clearInterval(animTimer)
            animTimer = null
            isPlaying.value = false
          }
          currentFrame.value = msg.frame
          if (p53Model && viewer) {
            p53Model.setFrame(msg.frame)
            applyStyles()
          }
        } else if (msg.type === 'play') {
          if (typeof msg.frame === 'number') {
            currentFrame.value = msg.frame
            if (p53Model) p53Model.setFrame(msg.frame)
          }
          if (msg.playing) {
            startPlayLoop()
          } else {
            if (animTimer) clearInterval(animTimer)
            animTimer = null
            isPlaying.value = false
            applyStyles()
          }
        } else if (msg.type === 'residue') {
          selectedResi.value = msg.resi ?? null
          applyStyles()
        } else if (msg.type === 'ui') {
          reprMode.value = msg.reprMode ?? reprMode.value
          showSurface.value = msg.showSurface ?? showSurface.value
          showPeptideSurface.value = msg.showPeptideSurface ?? showPeptideSurface.value
          showLabels.value = msg.showLabels ?? showLabels.value
          applyStyles()
        } else if (msg.type === 'camera' && Array.isArray(msg.view) && viewer) {
          viewer.setView(msg.view)
          viewer.render()
        }
      } finally {
        setTimeout(() => {
          isReceivingDemoSync = false
        }, 0)
      }
    }
  }

  try {
    const mod = await import('3dmol')
    $3Dmol = mod.default?.createViewer ? mod.default : (window as any).$3Dmol || mod

    const res = await fetch(`${import.meta.env.BASE_URL}1ycr.pdb`)
    if (!res.ok) throw new Error('Failed to fetch /1ycr.pdb')
    const rawPdb = await res.text()
    const { mdm2Pdb, p53TrajPdb } = generatePdbData(rawPdb)

    await nextTick()
    if (!viewerContainer.value) return
    viewer = $3Dmol.createViewer(viewerContainer.value, {
      backgroundColor: '#f8fafc',
      antialias: true,
    })

    // Sync 3D camera rotations & zoom across Presenter and Slides
    const hostEl = viewerContainer.value
    hostEl.addEventListener('pointerdown', () => { isPointerDown = true })
    hostEl.addEventListener('pointerup', () => {
      if (isPointerDown) {
        isPointerDown = false
        syncCameraView()
      }
    })
    hostEl.addEventListener('pointermove', () => {
      if (isPointerDown) syncCameraView()
    })
    hostEl.addEventListener('wheel', () => {
      setTimeout(syncCameraView, 16)
    }, { passive: true })

    // Only load the MDM2 Target Receptor on the Docking slide!
    if (!isFoldingMode.value) {
      mdm2Model = viewer.addModel(mdm2Pdb, 'pdb')
    }
    p53Model = viewer.addModelsAsFrames(p53TrajPdb, 'pdb')

    const initialClick = typeof $clicks?.value === 'number' ? Math.max(0, Math.min(maxStep.value, $clicks.value)) : 0
    step.value = initialClick
    currentFrame.value = stages.value[initialClick].targetFrame
    p53Model.setFrame(currentFrame.value)

    applyStyles()
    focusCamera()
    setTimeout(() => focusCamera(), 200)

    resizeObserver = new ResizeObserver((entries) => {
      if (!viewer || !viewerContainer.value) return
      const w = entries[0]?.contentRect?.width || viewerContainer.value.clientWidth
      viewer.resize()
      if (w > 320 && Math.abs(w - lastWidth) > 40) {
        lastWidth = w
        focusCamera()
      } else {
        viewer.render()
      }
    })
    resizeObserver.observe(viewerContainer.value)

    isLoaded.value = true
  } catch (err: any) {
    loadError.value = err?.message || 'Error initializing 3D WebGL viewer'
  }
})

onBeforeUnmount(() => {
  if (animTimer) clearInterval(animTimer)
  if (resizeObserver) resizeObserver.disconnect()
  demoSyncChannel?.close()
})
</script>

<template>
  <div class="select-none -mt-2">
    <!-- Dynamic Slide Header + Stage Stepper Bar -->
    <div class="flex items-center justify-between gap-2 mb-1.5">
      <div class="text-sm font-bold text-slate-900 tracking-tight truncate">
        {{ isFoldingMode ? 'Interactive 3D: 1D Chain → 3D α-Helix (p53)' : 'Interactive 3D: Docking & Engineering (p53 + MDM2)' }}
      </div>
      <div class="text-[11px] font-mono text-slate-400 whitespace-nowrap">
        ← / →
      </div>
    </div>

    <div
      class="grid gap-1.5 mb-2 text-[10px] font-mono"
      :class="isFoldingMode ? 'grid-cols-2' : 'grid-cols-4'"
    >
      <button
        v-for="(s, idx) in stages"
        :key="idx"
        @click="selectStage(idx)"
        class="px-2.5 py-1 rounded-lg border text-left transition-all duration-150 cursor-pointer"
        :class="
          step === idx
            ? 'bg-teal-50 border-teal-500 text-teal-900 shadow-sm ring-1 ring-teal-500/30'
            : 'bg-white border-slate-200 text-slate-500 hover:bg-slate-50'
        "
      >
        <div class="font-bold truncate">{{ s.badge }}</div>
      </button>
    </div>

    <!-- Main Full-Height Arena (415px tall): 9 Cols 3D Box + 3 Cols Compact Info Sidebar -->
    <div class="grid grid-cols-12 gap-2.5 items-stretch" style="height: 415px;">
      <!-- Left 9 Columns: Dedicated Sequence Strip + Unobstructed 340px 3D Canvas + Dedicated Bottom Controls -->
      <div
        class="col-span-9 rounded-xl bg-slate-50 border border-slate-300 shadow-sm overflow-hidden flex flex-col justify-between"
        style="height: 415px;"
      >
        <!-- Top Sequence Bar -->
        <div class="flex items-center justify-between gap-2 px-2.5 py-1 bg-white border-b border-slate-200 z-10">
          <div class="flex items-center gap-1">
            <span class="text-[9px] font-mono uppercase tracking-wider text-teal-800 font-bold mr-1">
              p53 (13 AA):
            </span>
            <button
              v-for="item in sequence"
              :key="item.resi"
              @click="selectResidue(item.resi)"
              :title="item.desc"
              class="px-1.5 py-0.5 rounded font-mono text-[10px] font-bold transition-all cursor-pointer border leading-tight"
              :class="[
                selectedResi === item.resi
                  ? 'ring-2 ring-amber-500 bg-amber-100 border-amber-500 text-amber-950 scale-110'
                  : isDeadEndStep && (item.resi === 23 || item.resi === 26)
                    ? 'bg-rose-100 border-rose-500 text-rose-900 animate-pulse'
                    : isEngineeredStep && item.aa !== item.mut
                      ? 'bg-emerald-100 border-emerald-500 text-emerald-900'
                      : item.anchor
                        ? 'bg-cyan-100 border-cyan-500 text-cyan-900'
                        : item.resi === 27
                          ? 'bg-amber-100 border-amber-500 text-amber-900'
                          : 'bg-slate-50 border-slate-200 text-slate-700 hover:bg-slate-100',
              ]"
            >
              <span>{{ isEngineeredStep ? item.mut : isDeadEndStep && item.resi === 23 ? 'A' : isDeadEndStep && item.resi === 26 ? 'W' : item.aa }}</span>
              <span class="block text-[7px] font-normal opacity-75 leading-none">{{ item.resi }}</span>
            </button>
          </div>

          <div class="text-[9px] font-mono text-slate-600 whitespace-nowrap">
            Frame <b class="text-cyan-700">{{ currentFrame }}/{{ maxFrame }}</b>
          </div>
        </div>

        <!-- Middle Unobstructed 3Dmol WebGL Canvas (340px tall) -->
        <div class="relative w-full" style="height: 340px;">
          <div
            ref="viewerContainer"
            class="mol-canvas-host"
            style="width: 100%; height: 340px; position: relative;"
          ></div>

          <div
            v-if="!isLoaded"
            class="absolute inset-0 flex items-center justify-center bg-slate-50/95 text-xs font-mono text-teal-800 z-20"
          >
            {{ loadError || 'Loading 3D WebGL Structure (1YCR.pdb)...' }}
          </div>
        </div>

        <!-- Bottom Playback & Representation Bar -->
        <div class="flex items-center justify-between gap-2 px-2.5 py-1 bg-white border-t border-slate-200 z-10 text-[9px] font-mono">
          <div class="flex items-center gap-1.5">
            <button
              @click="togglePlayTrajectory"
              class="px-2.5 py-0.5 rounded bg-teal-50 border border-teal-400 text-teal-900 hover:bg-teal-100 font-bold cursor-pointer"
            >
              {{ isPlaying ? '⏸ Pause' : isFoldingMode ? '▶ Play 3D Folding' : '▶ Play Pocket Docking' }}
            </button>

            <input
              type="range"
              :min="minFrame"
              :max="maxFrame"
              :value="currentFrame"
              @input="onScrubFrame"
              class="w-20 accent-teal-600 cursor-pointer"
            />
          </div>

          <div v-if="selectedResidueInfo" class="text-amber-800 font-semibold truncate max-w-[240px]">
            {{ selectedResidueInfo.desc }}
          </div>

          <div class="flex items-center gap-1">
            <div class="flex rounded overflow-hidden border border-slate-300">
              <button
                v-for="mode in (['hybrid', 'sticks', 'backbone', 'spheres', 'surface'] as const)"
                :key="mode"
                @click="reprMode = mode"
                class="px-1.5 py-0.5 capitalize cursor-pointer transition-colors"
                :class="reprMode === mode ? 'bg-cyan-100 text-cyan-900 font-bold' : 'bg-white text-slate-600 hover:bg-slate-50'"
                :title="mode === 'surface' ? 'Show 3D Molecular Surface (Plocha)' : `Representation: ${mode}`"
              >
                {{ mode === 'hybrid' ? 'Ribbon' : mode === 'backbone' ? 'Cα' : mode === 'surface' ? 'Surface' : mode }}
              </button>
            </div>

            <button
              @click="showLabels = !showLabels"
              class="px-1.5 py-0.5 rounded border cursor-pointer transition-colors"
              :class="showLabels ? 'bg-teal-100 border-teal-400 text-teal-900 font-bold' : 'bg-white border-slate-300 text-slate-500'"
              title="Toggle 3D Labels On/Off"
            >
              Labels
            </button>

            <button
              @click="showPeptideSurface = !showPeptideSurface"
              class="px-1.5 py-0.5 rounded border cursor-pointer transition-colors"
              :class="showPeptideSurface || reprMode === 'surface' ? 'bg-indigo-100 border-indigo-400 text-indigo-900 font-bold' : 'bg-white border-slate-300 text-slate-500'"
              title="Toggle Peptide 3D Surface Envelope (Plocha)"
            >
              Mesh
            </button>

            <button
              v-if="!isFoldingMode"
              @click="showSurface = !showSurface"
              class="px-1.5 py-0.5 rounded border cursor-pointer transition-colors"
              :class="showSurface ? 'bg-amber-100 border-amber-400 text-amber-900 font-bold' : 'bg-white border-slate-300 text-slate-500'"
              title="Toggle Target Active Pocket Surface (MDM2)"
            >
              Pocket
            </button>

            <button
              @click="focusCamera"
              class="px-1.5 py-0.5 rounded bg-slate-100 border border-slate-300 text-slate-700 hover:bg-slate-200 cursor-pointer"
              title="Re-center & Zoom 3D Camera"
            >
              ⟲
            </button>
          </div>
        </div>
      </div>

      <!-- Right 3 Columns (25% width): Clean Compact Info Sidebar -->
      <div
        class="col-span-3 rounded-xl bg-white border border-slate-200 p-3.5 flex flex-col justify-between"
        style="height: 415px;"
      >
        <div>
          <h3 class="text-sm font-bold text-slate-900 mb-2 leading-snug">
            {{ stages[step].title }}
          </h3>

          <p class="text-xs text-slate-600 leading-relaxed mb-3">
            {{ stages[step].desc }}
          </p>

          <div class="space-y-1.5 border-t border-slate-200 pt-2.5">
            <div
              v-for="(kv, i) in stages[step].tupleVars"
              :key="i"
              class="text-[11px] font-mono leading-snug"
            >
              <span class="text-teal-700 font-bold">{{ kv.k }}: </span>
              <span class="text-slate-700">{{ kv.v }}</span>
            </div>
          </div>
        </div>

        <div class="pt-2.5 border-t border-slate-200 space-y-2.5">
          <div
            class="text-xs font-mono font-bold"
            :class="
              isDeadEndStep
                ? 'text-rose-700'
                : isEngineeredStep
                  ? 'text-emerald-700'
                  : 'text-cyan-700'
            "
          >
            {{ stages[step].metric }}
          </div>

          <div class="flex justify-between gap-1.5">
            <button
              @click="selectStage(Math.max(0, step - 1))"
              :disabled="step === 0"
              class="flex-1 py-1 rounded bg-slate-100 hover:bg-slate-200 border border-slate-300 text-slate-700 disabled:opacity-35 text-[10px] font-mono cursor-pointer"
            >
              ← Prev
            </button>
            <button
              @click="selectStage(Math.min(maxStep, step + 1))"
              :disabled="step === maxStep"
              class="flex-1 py-1 rounded bg-teal-50 hover:bg-teal-100 border border-teal-400 text-teal-900 font-bold disabled:opacity-35 text-[10px] font-mono cursor-pointer"
            >
              Next →
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.mol-canvas-host :deep(canvas) {
  width: 100% !important;
  height: 100% !important;
  display: block !important;
}
</style>
