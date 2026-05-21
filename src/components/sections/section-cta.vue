<script setup>
import { Shader, Godrays, Fog, Swirl, ChromaticAberration, LinearGradient } from 'shaders/vue'
import Button from '../global/button.vue'

const emit = defineEmits(['book-demo'])
</script>

<template>
  <section class="mx-auto max-w-[1400px] px-6 py-24" data-section-cta>
    <div class="relative overflow-hidden rounded-[2rem] bg-background px-6 py-20 text-center sm:px-10 lg:px-16">
      <div class="absolute inset-0 h-full w-full" aria-hidden="true">
        <Shader class="absolute inset-0 h-full w-full" disable-telemetry>
          <ChromaticAberration :strength="1">
            <Swirl
              color-a="oklch(0.99 0.2 259.597)"
              color-b="oklch(0.8 0.2 259.597)"
              color-space="oklch"
              :speed="1"
              :detail="1.5"
              :blend="50"
            />
            <LinearGradient
              :color-a="'oklch(1 0 0/0)'"
              :color-b="'oklch(1 0 0)'"
              :angle="0"
              :blend-mode="overlay"
              :start="{ x: 0.5, y: 0 }"
              :end="{ x: 0.5, y: 1 }"
            />
            <Fog
              :animated="true"
              :speed="0.5"
              :color-a="'oklch(1 0 0)'"
              :color-b="'oklch(1 0 0/0)'"
              :mouse-influence="1"
              :blend-mode="plusLighter"
            />
            <Godrays
              :animated="true"
              :center="{ x: 0.5, y: -1 }"
              :density="0.25"
              :intensity="1"
              :spotty="0.1"
              :speed="2"
              rayColor="oklch(0.95 0.3 259.597)"
            />
          </ChromaticAberration>
        </Shader>
      </div>

      <div class="relative z-1 mx-auto flex max-w-3xl flex-col items-center justify-center gap-8 min-h-120">

        <div class="flex flex-col items-center justify-center gap-16">
          <span class="text-5xl font-normal">Ready when your team is</span>
          <button class="relative select-none cursor-pointer rounded-full px-3 py-2 text-2xl font-normal text-background transition-colors tracking-[-0.02em] before:content-[''] before:absolute before:-inset-3 before:rounded-full before:z-0 before:bg-foreground before:transition-all before:duration-200 before:ease-in-out hover:before:-inset-4" @click="emit('book-demo')"><span class="relative z-10">Book a Demo</span></button>
        </div>
      </div>
    </div>
  </section>
</template>
