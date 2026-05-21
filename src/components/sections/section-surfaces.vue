<script setup>
import { computed } from 'vue'
import { cn } from '../../lib/cn'

const props = defineProps({
  headline: {
    type: String,
    default: 'Wherever you are,<br />Remi is too.',
  },
  tabs: {
    type: Array,
    default: () => [
      {
        id: 'slack',
        label: 'Slack',
        description: 'Keep members, creators, and internal teams moving without leaving the conversation.',
      },
      {
        id: 'imessage',
        label: 'iMessage',
        description: 'Let high-touch conversations stay personal while Remi remembers the details.',
      },
      {
        id: 'web',
        label: 'Web',
        description: 'Give every member a home base for courses, events, discussions, and support.',
      },
    ],
  },
})

const headlineParts = computed(() =>
  props.headline
    .split(/<br\s*\/?>/i)
    .map((part) => part.trim())
    .filter(Boolean),
)

const cards = computed(() =>
  props.tabs.map((tab, index) => ({
    id: tab.id || `channel-${index + 1}`,
    title: tab.label || `Channel ${index + 1}`,
    body: tab.description || '',
  })),
)
</script>

<template>
  <section id="channels" :class="cn('relative mx-auto max-w-[1400px] scroll-mt-24 px-6 py-12 md:py-16')" data-section-surfaces>
    <div :class="cn('mb-12 max-w-3xl')">
      <h2 :class="cn('text-4xl font-normal leading-none tracking-tight text-balance text-foreground')">
        <template v-for="(part, index) in headlineParts" :key="`${part}-${index}`">
          <br v-if="index > 0">
          <span :class="cn(index > 0 && 'text-foreground/50')">
            {{ part }}
          </span>
        </template>
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
      <article v-for="card in cards" :key="card.id">
        <div :class="cn('flex min-h-120 h-full flex-col justify-end gap-10 p-8')">
          <div :class="cn('flex flex-col gap-0')">
            <h3 :class="cn('text-2xl font-normal leading-tight tracking-tight text-balance text-foreground')">
              {{ card.title }}
            </h3>
            <p :class="cn('text-xl leading-tight text-foreground/40')">
              {{ card.body }}
            </p>
          </div>
        </div>
      </article>
    </div>
  </section>
</template>
