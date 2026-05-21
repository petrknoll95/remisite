<script setup>
import { PhArrowRight } from '@phosphor-icons/vue'
</script>

<template>
  <a
    href="#"
    rel="noopener noreferrer"
    class="relative bg-blue-500/10 backdrop-blur-sm text-blue-500 font-medium rounded-full hover:bg-blue-500/20 transition-colors"
    data-announcement-bar
  >
    <div class="w-full flex items-center justify-center gap-2 px-3 h-8 text-center font-sans text-sm">
      <span>Remi launches in a private beta.</span>
      <span><PhArrowRight size="16" weight="regular" aria-hidden="true" /></span>
    </div>
  </a>
</template>
