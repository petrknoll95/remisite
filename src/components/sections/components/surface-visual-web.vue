<script setup>
import HomeSliderUiFiles from '../../home/slider-ui/home-slider-ui-files.vue'
import { Shader, FilmGrain, Swirl } from 'shaders/vue'
</script>

<template>
  <div class="relative flex size-full items-start justify-start overflow-hidden p-8">
    <Shader class="absolute inset-0 h-full w-full" disable-telemetry>
      <FilmGrain :strength="0.5" :bias="0.4" :displacement="0.3" :animated="true">
        <Swirl color-a="oklch(0.96 0.08 170)" color-b="oklch(0.9 0.05 215)" color-space="oklch" :speed="0.55"
          :detail="1.45" :blend="44" />
      </FilmGrain>
    </Shader>
    <div class="relative z-10 w-[42rem] max-w-none shrink-0">
      <HomeSliderUiFiles class="!mx-0 text-[1.35em]" />
    </div>
  </div>
</template>
