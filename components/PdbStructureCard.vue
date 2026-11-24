<script setup lang="ts">
import { ref, watch, onMounted, onBeforeUnmount } from 'vue'
import { createSyncState } from '@slidev/client'

const props = withDefaults(
  defineProps<{
    pdbUrl: string
    chainFilter?: string[]
    colorMode?: 'ss' | 'chain' | 'solid'
    solidColor?: string
    height?: string
    showControls?: boolean
    singleModelOnly?: boolean
    zoom?: number
    /** Mirror camera moves to other windows of this deck (presenter <-> slide window). */
    syncView?: boolean
  }>(),
  {
    colorMode: 'chain',
    solidColor: '#0f766e',
    height: '250px',
    showControls: false,
    singleModelOnly: false,
    zoom: 1,
    syncView: true,
  }
)

const container = ref<HTMLElement | null>(null)
const loaded = ref(false)
const error = ref('')
const showSurface = ref(false)
const showAtoms = ref(false)

// With a surface on, the molecule is visibly larger than the cartoon that zoomTo() was
// fitted to, so it would spill out of the frame. Zoom out relatively (keeps rotation).
const SURFACE_ZOOM_OUT = 0.85
let surfaceZoomed = false

let $3Dmol: any = null
let viewer: any = null
let resizeObserver: ResizeObserver | null = null

// --- view sync across windows of the same browser ---------------------------
// Slidev's own sync primitive: BroadcastChannel, with a localStorage fallback.
// Same machine / same browser only (presenter window <-> slide window); it does
// not reach other devices over --remote.
// Carries the camera (v) plus the two display toggles (s = surface, a = atoms).
interface ViewState {
  v: number[] | null
  s: boolean
  a: boolean
}
const SYNC_THROTTLE_MS = 50
type SyncHandle = ReturnType<typeof createSyncState<ViewState>>
let sync: SyncHandle | null = null
let lastSent = 0
let pendingSend: ReturnType<typeof setTimeout> | null = null

// Depth counter, not a boolean: applying a remote toggle re-enters this guard
// (it re-runs applyStyle, which nudges the zoom), and a plain flag would be
// cleared by the inner release while the outer apply is still in flight.
let remoteDepth = 0
function asRemote(fn: () => void) {
  remoteDepth++
  try {
    fn()
  } catch {}
  // Release after the current tick, so side effects of fn() (3Dmol fires its
  // view-change callback synchronously) are still seen as "remote".
  setTimeout(() => {
    remoteDepth = Math.max(0, remoteDepth - 1)
  }, 0)
}

function sendView() {
  if (!viewer || !sync || remoteDepth > 0) return
  const now = Date.now()
  const wait = SYNC_THROTTLE_MS - (now - lastSent)
  if (wait > 0) {
    if (!pendingSend) pendingSend = setTimeout(() => { pendingSend = null; sendView() }, wait)
    return
  }
  lastSent = now
  try {
    // patch() (not a plain assignment) marks this as a local change, so our own
    // onPatch handler stays quiet and only genuinely remote updates get applied.
    sync.patch('v', viewer.getView())
  } catch {}
}

function sendToggles() {
  if (!sync || remoteDepth > 0) return
  try {
    sync.patch('s', showSurface.value)
    sync.patch('a', showAtoms.value)
  } catch {}
}

function applyRemote(st: ViewState) {
  asRemote(() => {
    // Toggles first: they change what the camera is looking at.
    if (typeof st.s === 'boolean' && st.s !== showSurface.value) showSurface.value = st.s
    if (typeof st.a === 'boolean' && st.a !== showAtoms.value) showAtoms.value = st.a
    if (viewer && st.v) viewer.setView(st.v)
  })
}

const CHAIN_COLORS: Record<string, string> = {
  A: '#0f766e', // teal-700
  E: '#4338ca', // indigo-700
  B: '#0e7490', // cyan-700
}

function applyStyle() {
  if (!viewer || !$3Dmol) return
  viewer.removeAllSurfaces()

  const cartoon: any = {
    opacity: showSurface.value && showAtoms.value ? 0.85 : 1.0,
  }

  if (props.colorMode === 'ss') {
    cartoon.colorfunc = (atom: any) => {
      if (atom.ss === 'h') return '#0f766e'
      if (atom.ss === 's') return '#0e7490'
      return '#64748b'
    }
  } else if (props.colorMode === 'solid') {
    cartoon.color = props.solidColor
  } else {
    cartoon.colorfunc = (atom: any) => CHAIN_COLORS[atom.chain] || '#4338ca'
  }

  const styleObj: any = { cartoon }
  if (showAtoms.value) {
    styleObj.stick = { radius: 0.14, colorscheme: 'Jmol' }
  }

  viewer.setStyle({}, styleObj)

  if (showSurface.value) {
    // Solid when it is the only thing on screen; see-through once sticks are shown underneath.
    const surfOpacity = showAtoms.value ? 0.45 : 0.95
    if (props.chainFilter && props.chainFilter.length > 1) {
      for (const ch of props.chainFilter) {
        const surfColor = ch === 'A' ? '#2dd4bf' : '#818cf8'
        viewer.addSurface($3Dmol.SurfaceType.VDW, { opacity: surfOpacity, color: surfColor }, { chain: ch })
      }
    } else {
      viewer.addSurface($3Dmol.SurfaceType.VDW, { opacity: surfOpacity, color: '#2dd4bf' })
    }
  }

  if (showSurface.value !== surfaceZoomed) {
    // Deterministic from the surface toggle, so every window computes the same
    // nudge on its own — publishing it would only cause a needless round trip.
    asRemote(() => viewer.zoom(showSurface.value ? SURFACE_ZOOM_OUT : 1 / SURFACE_ZOOM_OUT))
    surfaceZoomed = showSurface.value
  }

  viewer.render()
}

watch([showSurface, showAtoms], () => {
  applyStyle()
  sendToggles()
})

onMounted(async () => {
  if (!container.value) return
  try {
    const mod = await import('3dmol')
    $3Dmol = mod.default || mod
    const resolvedUrl = props.pdbUrl.startsWith('/')
      ? `${import.meta.env.BASE_URL}${props.pdbUrl.slice(1)}`
      : props.pdbUrl
    const res = await fetch(resolvedUrl)
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    let pdbText = await res.text()

    if (props.singleModelOnly) {
      pdbText = pdbText.split('ENDMDL')[0]
    }

    if (props.chainFilter && props.chainFilter.length > 0) {
      const keep = new Set(props.chainFilter)
      pdbText = pdbText
        .split('\n')
        .filter((line) => {
          if (!line.startsWith('ATOM  ') && !line.startsWith('HETATM')) return true
          return keep.has(line[21])
        })
        .join('\n')
    }

    viewer = $3Dmol.createViewer(container.value, {
      backgroundColor: 0xffffff,
      antialias: true,
    })

    viewer.addModel(pdbText, 'pdb')
    applyStyle()
    viewer.zoomTo()
    if (props.zoom !== 1) viewer.zoom(props.zoom)
    viewer.render()
    loaded.value = true

    if (props.syncView) {
      try {
        const initial: ViewState = { v: null, s: showSurface.value, a: showAtoms.value }
        sync = createSyncState<ViewState>(initial, initial)
        // Remote -> local. onPatch fires for changes that did NOT come from our
        // own patch() call, i.e. exactly the ones another window published.
        sync.onPatch((st) => applyRemote(st))
        // One channel per structure, so two different models never fight.
        sync.init(`pdb-view:${props.pdbUrl}:${(props.chainFilter || []).join('')}`)
        // Local -> remote
        viewer.setViewChangeCallback(() => sendView())
        // Adopt whatever another window had already published.
        if (sync.state.v || sync.state.s || sync.state.a) applyRemote(sync.state)
      } catch {
        sync = null
      }
    }

    resizeObserver = new ResizeObserver(() => {
      if (viewer && container.value && container.value.clientWidth > 0) {
        viewer.resize()
        viewer.render()
      }
    })
    resizeObserver.observe(container.value)
  } catch (e: any) {
    error.value = e?.message || 'Failed to load PDB'
  }
})

onBeforeUnmount(() => {
  if (pendingSend) clearTimeout(pendingSend)
  sync = null
  resizeObserver?.disconnect()
  if (viewer) {
    try {
      viewer.clear()
    } catch {}
    viewer = null
  }
})
</script>

<template>
  <div class="w-full">
    <div v-if="props.showControls" class="flex items-center justify-between gap-2 mb-1.5 min-h-[26px]">
      <div v-if="showAtoms" class="flex items-center gap-2.5 text-[10.5px] font-mono text-slate-600">
        <span class="inline-flex items-center gap-1"><span class="w-2.5 h-2.5 rounded-full inline-block border border-slate-400" style="background:#909090"></span>C</span>
        <span class="inline-flex items-center gap-1"><span class="w-2.5 h-2.5 rounded-full inline-block border border-blue-700" style="background:#3050F8"></span>N</span>
        <span class="inline-flex items-center gap-1"><span class="w-2.5 h-2.5 rounded-full inline-block border border-red-700" style="background:#FF0D0D"></span>O</span>
        <span class="inline-flex items-center gap-1"><span class="w-2.5 h-2.5 rounded-full inline-block border border-yellow-600" style="background:#FACC15"></span>S</span>
      </div>
      <div v-else />
      <div class="flex items-center gap-1.5">
        <button
          @click="showSurface = !showSurface"
          class="px-2.5 py-1 rounded-md border text-[11px] font-mono font-semibold transition-colors cursor-pointer"
          :class="
            showSurface
              ? 'bg-cyan-700 text-white border-cyan-700'
              : 'bg-slate-50 text-slate-700 border-slate-300 hover:bg-slate-100'
          "
        >
          {{ showSurface ? '✓ Surface' : '+ Surface' }}
        </button>
        <button
          @click="showAtoms = !showAtoms"
          class="px-2.5 py-1 rounded-md border text-[11px] font-mono font-semibold transition-colors cursor-pointer"
          :class="
            showAtoms
              ? 'bg-indigo-700 text-white border-indigo-700'
              : 'bg-slate-50 text-slate-700 border-slate-300 hover:bg-slate-100'
          "
        >
          {{ showAtoms ? '✓ Atoms' : '+ Atoms' }}
        </button>
      </div>
    </div>

    <div
      class="relative w-full rounded-lg overflow-hidden bg-white border border-slate-200"
      :style="{ height: props.height }"
    >
      <div ref="container" class="relative w-full h-full cursor-grab active:cursor-grabbing" />
      <div
        v-if="!loaded && !error"
        class="absolute inset-0 flex items-center justify-center text-[11px] text-slate-400 font-mono bg-white"
      >
        Loading 3D PDB...
      </div>
      <div
        v-if="error"
        class="absolute inset-0 flex items-center justify-center text-[11px] text-red-500 font-mono bg-white"
      >
        {{ error }}
      </div>
    </div>
  </div>
</template>
