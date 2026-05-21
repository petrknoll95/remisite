<script setup>
import { Shader, Godrays, Fog, Swirl, ChromaticAberration } from 'shaders/vue'
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
            <Fog
              :animated="true"
              :speed="0.5"
              :color-a="'oklch(1 0 0)'"
              :color-b="'oklch(1 0 0/0)'"
              :mouse-influence="0.5"
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

      <div class="relative z-1 mx-auto flex max-w-3xl flex-col items-center gap-8">
        <div class="space-y-4">
          <p class="text-sm font-medium tracking-tight text-foreground/45">
            Ready when your team is
          </p>
          <h2 class="text-[clamp(3rem,7vw,5rem)] font-normal leading-[0.95em] tracking-[-0.03em] text-balance text-foreground">
            Put Remi to work on your operations.
          </h2>
          <p class="mx-auto max-w-[42ch] text-base leading-normal tracking-tight text-foreground/55">
            Give Remi the context, tools, and handoffs your team depends on, then let it keep the work moving.
          </p>
        </div>

        <div class="flex flex-wrap items-center justify-center gap-2">
          <Button @click="emit('book-demo')">Book a Demo</Button>
          <Button variant="secondary" href="#pricing">Pricing</Button>
        </div>
      </div>
    </div>
  </section>
</template>
