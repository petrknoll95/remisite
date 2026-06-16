<script setup>
import { cn } from '../../lib/cn'
import HowVisualScan from './components/how-visual-scan.vue'
import HowVisualReady from './components/how-visual-ready.vue'

const steps = [
  {
    title: 'Install Remi',
    body: 'Add Remi to Slack, then connect the tools your business already runs on.',
  },
  {
    title: 'Remi learns the business',
    body: 'She scans all your tools and context to build her knowledge base.',
    visual: HowVisualScan,
  },
  {
    title: 'Remi is ready to go',
    body: 'Ask questions, create documents, route follow-ups, and automate work.',
    visual: HowVisualReady,
  },
]
</script>

<template>
  <section id="how" :class="cn('relative mx-auto max-w-[1400px] scroll-mt-24 px-6 py-12 md:py-16')" data-section-how>
    <div :class="cn('mb-12 max-w-3xl')">
      <h2 :class="cn('text-4xl font-normal leading-none tracking-tight text-balance text-foreground')">
        Add Remi once.<br />
        <span :class="cn('text-foreground/50')">
          She learns how your business runs.
        </span>
      </h2>
    </div>

    <div
      :class="cn(
        'grid grid-cols-1 gap-4 md:grid-cols-3',
        '[&>article]:relative [&>article]:overflow-hidden [&>article]:rounded-4xl [&>article]:bg-background',
        '[&>article]:after:content-[\'\'] [&>article]:after:absolute [&>article]:after:inset-0 [&>article]:after:pointer-events-none [&>article]:after:rounded-[inherit]',
        '[&>article]:after:shadow-[0_0_0_1px_color-mix(in_oklch,var(--color-foreground)_5%,transparent)_inset] [&>article]:after:mix-blend-plus-darker',
      )"
    >
      <article v-for="(step, index) in steps" :key="step.title">
        <div :class="cn('relative flex min-h-120 h-full flex-col justify-between gap-3 p-4')">
          <span :class="cn('absolute top-4 left-4 text-lg size-10 rounded-full bg-foreground text-background flex items-center justify-center font-normal leading-none')">
            {{ String(index + 1).padStart(2, '') }}
          </span>
          <component :is="step.visual" v-if="step.visual" />
          <div :class="cn('flex flex-col gap-0 p-4')">
            <h3 :class="cn('text-xl font-normal leading-tight tracking-tight text-balance text-foreground')">
              {{ step.title }}
            </h3>
            <p :class="cn('text-lg leading-tight text-foreground/40')">
              {{ step.body }}
            </p>
          </div>
        </div>
      </article>
    </div>
  </section>
</template>
