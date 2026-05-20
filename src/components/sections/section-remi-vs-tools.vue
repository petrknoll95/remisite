<script setup>
import { computed, ref } from 'vue'
import { cn } from '../../lib/cn'
import RemiLogo from '../global/site-logo.vue'
import LogoChatgpt from '../logos/logo-chatgpt.vue'
import LogoCopilot from '../logos/logo-copilot.vue'
import LogoPerplexity from '../logos/logo-perplexity.vue'
import LogoZapier from '../logos/logo-zapier.vue'
import { Shader, FilmGrain, Swirl } from 'shaders/vue'

const comparisons = [
  {
    useCase: 'Meeting follow-ups',
    competitor: 'Copilot',
    competitorLogo: LogoCopilot,
    tools: 'Copilot summarizes your meeting. ChatGPT or Claude can turn notes into a cleaner recap.',
    remi: 'Creates the tasks, sends the follow-ups, updates CRM, and keeps tracking what is still open.',
  },
  {
    useCase: 'Client status updates',
    competitor: 'ChatGPT',
    competitorLogo: LogoChatgpt,
    tools: 'ChatGPT can draft an update if you paste in the context and explain what changed.',
    remi: 'Pulls the latest project context, identifies blockers, drafts the update, and routes it to the right people.',
  },
  {
    useCase: 'Recurring operations',
    competitor: 'Zapier',
    competitorLogo: LogoZapier,
    tools: 'Zapier can trigger workflows you manually define and maintain step by step.',
    remi: 'Understands the recurring obligation, gathers the inputs, prepares the work, and follows through.',
  },
  {
    useCase: 'Finding business context',
    competitor: 'Perplexity',
    competitorLogo: LogoPerplexity,
    tools: 'Perplexity can find public answers and summarize sources, but it does not know your internal business context.',
    remi: 'Searches across Slack, email, docs, CRM, and project tools with memory of how it all connects.',
  },
]

const activeComparisonIndex = ref(0)
const activeComparison = computed(() => comparisons[activeComparisonIndex.value] || comparisons[0])
</script>

<template>
  <section :class="cn('relative mx-auto max-w-[1400px] px-6 py-24')" data-remi-vs-tools>
    <div :class="cn('mb-12 max-w-4xl')">
      <h2 :class="cn('text-4xl font-normal leading-none tracking-tight text-balance text-foreground')">
        AI tools answer questions.<br />
        <span :class="cn('text-foreground/50')">
          Operations need follow-through.
        </span>
      </h2>
    </div>

    <div :class="cn('grid gap-2 lg:grid-cols-[0.45fr_1fr]')">
      <div :class="cn(
        'relative flex flex-col gap-2',
      )" role="tablist" aria-label="Use case comparison">
        <button v-for="(comparison, index) in comparisons" :key="comparison.useCase" type="button" role="tab"
          :aria-selected="activeComparisonIndex === index" :class="cn(
            'relative flex-1 z-1 flex w-full cursor-pointer items-center justify-between gap-4 rounded-2xl px-6 py-5 text-left transition-colors',
            activeComparisonIndex === index ? 'bg-foreground text-background' : 'text-foreground/50 bg-foreground/5 hover:bg-foreground/10 hover:text-foreground',
          )" @click="activeComparisonIndex = index">
          <span :class="cn('text-xl font-normal leading-tight tracking-tight')">
            {{ comparison.useCase }}
          </span>
        </button>
      </div>

      <article :class="cn(
        'relative',
      )">
        <div :class="cn('grid gap-2 md:grid-cols-2')">
          <div
            :class="cn('rounded-2xl bg-foreground/5 p-6 min-h-80 md:min-h-120 flex flex-col items-start justify-between')">
            <div :class="cn('flex items-center gap-3 text-foreground')">
              <component :is="activeComparison.competitorLogo" :class="cn('size-10 shrink-0')" />
              <p :class="cn('text-sm font-normal leading-none text-foreground/50')">
                {{ activeComparison.competitor }}
              </p>
            </div>
            <p :class="cn('text-2xl font-normal leading-tight tracking-tight text-foreground text-pretty')">
              {{ activeComparison.tools }}
            </p>
          </div>

          <div
            :class="cn('relative rounded-2xl overflow-hidden bg-cyan-100 p-6 text-foreground min-h-80 md:min-h-120 flex flex-col items-start justify-between')">
            <p :class="cn('relative z-1 text-sm font-normal leading-non')">
              <RemiLogo />
            </p>
            <p :class="cn('relative z-1 text-2xl font-normal leading-tight tracking-tight text-pretty')">
              {{ activeComparison.remi }}
            </p>
            <div :class="cn('absolute inset-0')">
              <Shader class="absolute inset-0 h-full w-full" disable-telemetry>
                <FilmGrain :strength="0.5" :bias="0.4" :animated="true">
                  <Swirl color-a="oklch(0.98 0.024 259.597)" color-b="oklch(0.858 0.07 259.597)" color-space="oklch"
                    :speed="0.5" :detail="1.35" :blend="50" />
                </FilmGrain>
              </Shader>
            </div>
          </div>
        </div>
      </article>
    </div>
  </section>
</template>
