<script setup lang="ts">
import { useNav, useSlideContext } from '@slidev/client'

const { $page } = useSlideContext()
const { total } = useNav()
</script>

<template>
  <footer
    v-if="$page > 1 && $page < total"
    class="absolute bottom-3 right-5 text-[11px] font-mono text-slate-400 pointer-events-none select-none z-50"
  >
    {{ $page }} / {{ total }}
  </footer>
</template>
