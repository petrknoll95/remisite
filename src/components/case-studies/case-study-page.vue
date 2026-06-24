<script setup>
import { PhArrowLeft, PhArrowRight } from '@phosphor-icons/vue'
import { computed, onBeforeUnmount, watchEffect } from 'vue'
import { caseStudies } from '../../lib/case-studies'
import cn from '../../lib/cn'
import Button from '../global/button.vue'
import GlobalFooter from '../global/global-footer.vue'
import GlobalHeader from '../header/global-header.vue'
import SectionCta from '../sections/section-cta.vue'

const requestedSlug = computed(() => {
  if (typeof window === 'undefined') {
    return caseStudies[0]?.slug || ''
  }

  const normalizedPath = window.location.pathname.replace(/\/+$/, '')
  const slug = normalizedPath.split('/').filter(Boolean)[1]

  return slug || caseStudies[0]?.slug || ''
})

const caseStudy = computed(() =>
  caseStudies.find((entry) => entry.slug === requestedSlug.value) || null
)
const fallbackCaseStudy = computed(() => caseStudies[0] || null)
const caseStudyIndex = computed(() =>
  caseStudy.value ? caseStudies.findIndex((entry) => entry.slug === caseStudy.value.slug) : -1
)
const previousCaseStudy = computed(() =>
  caseStudyIndex.value >= 0 ? caseStudies[caseStudyIndex.value + 1] || null : null
)
const nextCaseStudy = computed(() =>
  caseStudyIndex.value > 0 ? caseStudies[caseStudyIndex.value - 1] || null : null
)

const goBackToCaseStudies = () => {
  if (typeof window === 'undefined') {
    return
  }

  const referrer = document.referrer ? new URL(document.referrer) : null
  const isSameOriginReferrer = referrer?.origin === window.location.origin

  if (isSameOriginReferrer && window.history.length > 1) {
    window.history.back()
    return
  }

  window.location.href = '/#case-studies'
}

const heroImage = computed(() =>
  caseStudy.value?.coverImageUrl || caseStudy.value?.metadata.ogImage || ''
)
const caseStudyStats = computed(() => caseStudy.value?.stats || [])
const caseStudyDetails = computed(() => {
  if (!caseStudy.value) {
    return []
  }

  const metadata = caseStudy.value.metadata

  return [
    {
      label: 'Industry',
      value: caseStudy.value.industryLabel || metadata.industryLabel || metadata.industry,
    },
    {
      label: 'Use case',
      value: metadata.useCase,
    },
    {
      label: 'Company size',
      value: metadata.companySize,
    },
    {
      label: 'Region',
      value: metadata.region,
    },
  ].filter((item) => item.value)
})
const hasPexelsCredit = computed(() =>
  Boolean(
    caseStudy.value?.metadata.pexelsPhotographer &&
    caseStudy.value?.metadata.pexelsPhotographerUrl &&
    caseStudy.value?.metadata.pexelsPhotoUrl
  )
)
const originalTitle = typeof document !== 'undefined' ? document.title : ''
const changedMeta = new Map()

const setMetaTag = (attribute, key, content) => {
  if (typeof document === 'undefined' || !content) {
    return
  }

  const selector = `meta[${attribute}="${key}"]`
  let tag = document.head.querySelector(selector)

  if (!changedMeta.has(selector)) {
    changedMeta.set(selector, {
      existed: Boolean(tag),
      content: tag?.getAttribute('content') || '',
    })
  }

  if (!tag) {
    tag = document.createElement('meta')
    tag.setAttribute(attribute, key)
    document.head.appendChild(tag)
  }

  tag.setAttribute('content', content)
}

watchEffect(() => {
  if (typeof document === 'undefined' || !caseStudy.value) {
    return
  }

  const pageTitle = caseStudy.value.metadata.ogTitle || caseStudy.value.title
  const pageDescription = caseStudy.value.description
  const ogDescription = caseStudy.value.metadata.ogDescription || pageDescription

  document.title = `${pageTitle} | emi`
  setMetaTag('name', 'description', pageDescription)
  setMetaTag('name', 'author', caseStudy.value.metadata.author)
  setMetaTag('property', 'og:title', pageTitle)
  setMetaTag('property', 'og:description', ogDescription)
  setMetaTag('property', 'og:type', 'article')
  setMetaTag('property', 'og:url', window.location.href)
  setMetaTag('property', 'og:image', caseStudy.value.metadata.ogImage || caseStudy.value.coverImageUrl)
  setMetaTag('property', 'article:published_time', caseStudy.value.date)
})

onBeforeUnmount(() => {
  if (typeof document === 'undefined') {
    return
  }

  document.title = originalTitle

  changedMeta.forEach((snapshot, selector) => {
    const tag = document.head.querySelector(selector)

    if (!tag) {
      return
    }

    if (!snapshot.existed) {
      tag.emove()
      return
    }

    tag.setAttribute('content', snapshot.content)
  })
})
</script>

<template>
  <div class="min-h-svh bg-background text-foreground">
    <GlobalHeader theme="light" />

    <main v-if="caseStudy">
      <section class="relative w-full px-6 pb-6 pt-[calc(var(--header-height)+4em)] text-foreground"
        data-case-study-hero>
        <div class="relative w-full">
          <div class="flex w-full flex-col items-stretch justify-between gap-20">
            <div class="w-full grid md:grid-cols-[1fr_3fr_1fr] gap-y-8 items-start max-w-(--content-width) mx-auto">
              <button type="button" aria-label="Back to case studies"
                class="inline-flex w-16 h-8 shrink-0 items-center justify-center rounded-full text-foreground transition-colors outline-0 outline-offset-0 focus-visible:outline-none shadow-[0_0_0_1px_var(--color-border)]"
                @click="goBackToCaseStudies">
                <PhArrowLeft class="size-4" weight="regular" aria-hidden="true" />
              </button>
              <div class="flex flex-col items-start gap-8">
                <h1 class="text-pretty text-5xl font-normal leading-none tracking-tight">
                  {{ caseStudy.title }}
                </h1>
              </div>
            </div>
            <figure v-if="heroImage"
              class="relative w-full aspect-4/3 lg:aspect-2/1 overflow-hidden rounded-2xl bg-muted flex flex-col justify-end p-2">
              <img :src="heroImage" :alt="caseStudy.metadata.coverImageAlt || caseStudy.title"
                class="absolute inset-0 size-full object-cover">
            </figure>
          </div>
        </div>
      </section>

      <section class="w-full px-6 py-20 md:py-24 lg:py-32" data-case-study-contents>
        <div class="mx-auto grid w-full max-w-(--content-width) gap-x-24 gap-y-12 lg:grid-cols-[1fr_3fr_1fr]">
          <aside
            class="flex w-full flex-col gap-6 lg:sticky lg:top-[calc(var(--header-height)+2em)] lg:self-start lg:max-w-sm"
            aria-label="Case study details">
            <dl v-if="caseStudyDetails.length" class="flex flex-col">
              <div v-for="detail in caseStudyDetails" :key="detail.label"
                class="border-t border-border py-4 first:border-t-0 flex flex-col gap-1">
                <dt class="text-sm font-normal leading-none tracking-tight text-foreground">
                  {{ detail.label }}
                </dt>
                <dd class="text-sm leading-none text-muted-foreground">
                  {{ detail.value }}
                </dd>
              </div>
            </dl>
          </aside>

          <div class="flex min-w-0 flex-col gap-12">
            <dl v-if="caseStudyStats.length" class="flex gap-2 relative z-2" data-case-study-stats>
              <div v-for="stat in caseStudyStats" :key="`${stat.metric}-${stat.label}`"
                class="min-w-0 flex-1 bg-muted backdrop-blur-sm rounded-xl p-4 flex flex-col justify-between min-h-40 gap-8 text-foreground">
                <dt class="text-sm leading-snug text-pretty opacity-50 capitalize">
                  {{ stat.label }}
                </dt>
                <dd class="mt-2 text-2xl font-normal leading-none tracking-tight tabular-nums">
                  {{ stat.metric }}
                </dd>
              </div>
            </dl>
            <article class="case-study-content" v-html="caseStudy.html" :class="cn('*:first:mt-0!')" />

            <nav v-if="previousCaseStudy || nextCaseStudy" class="mt-16 grid gap-8 sm:grid-cols-2"
              aria-label="Case study navigation">
              <a v-if="previousCaseStudy" :href="previousCaseStudy.path"
                class="group flex min-w-0 items-start gap-4 text-left focus-visible:outline-2 focus-visible:outline-offset-4 focus-visible:outline-ring">
                <span
                  class="inline-flex w-12 h-10 shrink-0 items-center justify-center rounded-full bg-muted text-foreground transition-transform group-hover:-translate-x-0.5">
                  <PhArrowLeft class="size-4" weight="regular" aria-hidden="true" />
                </span>
                <span class="min-w-0 flex flex-col gap-1">
                  <span class="block text-sm font-medium leading-tight tracking-tight text-foreground">
                    Previous story
                  </span>
                  <span class="block text-sm leading-tight text-muted-foreground max-w-[24ch] truncate">
                    {{ previousCaseStudy.title }}
                  </span>
                </span>
              </a>
              <span v-else aria-hidden="true" class="hidden sm:block" />

              <a v-if="nextCaseStudy" :href="nextCaseStudy.path"
                class="group flex min-w-0 items-center justify-end gap-4 text-right focus-visible:outline-2 focus-visible:outline-offset-4 focus-visible:outline-ring">
                <span class="min-w-0 flex flex-col gap-1">
                  <span class="block text-sm font-medium leading-tight tracking-tight text-foreground">
                    Next story
                  </span>
                  <span class="block text-sm leading-tight text-muted-foreground max-w-[24ch] truncate">
                    {{ nextCaseStudy.title }}
                  </span>
                </span>
                <span
                  class="inline-flex w-12 h-10 shrink-0 items-center justify-center rounded-full bg-muted text-foreground transition-transform group-hover:translate-x-0.5">
                  <PhArrowRight class="size-4" weight="regular" aria-hidden="true" />
                </span>
              </a>
            </nav>
          </div>
        </div>
      </section>

      <SectionCta />
    </main>

    <main v-else class="min-h-svh px-6 pt-32">
      <section class="mx-auto flex max-w-2xl flex-col items-start gap-6 py-24">
        <p class="text-sm font-medium text-muted-foreground">Case Study</p>
        <h1 class="text-balance text-5xl font-normal leading-none tracking-tight">
          Case study not found
        </h1>
        <p class="text-pretty text-lg leading-8 text-muted-foreground">
          This Markdown file is not available yet.
        </p>
        <Button v-if="fallbackCaseStudy" :href="fallbackCaseStudy.path">
          Open the example case study
        </Button>
      </section>
    </main>

    <GlobalFooter />
  </div>
</template>

<style scoped>
.case-study-content {
  color: var(--color-foreground);
  font-size: var(--text-sm);
  line-height: 1.35em;
}

.case-study-content :deep(*) {
  text-wrap: pretty;
}

.case-study-content :deep(h1),
.case-study-content :deep(h2),
.case-study-content :deep(h3) {
  color: var(--color-foreground);
  font-weight: 400;
  letter-spacing: var(--tracking-tight);
  line-height: 1.12;
  scroll-margin-top: calc(var(--header-height) + 2em);
  text-wrap: balance;
}

.case-study-content :deep(h1) {
  margin: 3em 0 1em;
  font-size: 2em;
}

.case-study-content :deep(h2) {
  margin: 2.75em 0 0.75em;
  font-size: 1.5em;
}

.case-study-content :deep(h3) {
  margin: 2.5em 0 0.6em;
  font-size: 1.2em;
}

@media (min-width: 768px) {
  .case-study-content :deep(h1) {
    font-size: 2.5em;
  }

  .case-study-content :deep(h2) {
    font-size: 1.85em;
  }

  .case-study-content :deep(h3) {
    font-size: 1.35em;
  }
}

.case-study-content :deep(p) {
  margin: 1em 0;
}

.case-study-content :deep(a) {
  color: var(--color-foreground);
  text-decoration: underline;
  text-decoration-thickness: 0.08em;
  text-underline-offset: 0.18em;
}

.case-study-content :deep(strong) {
  color: var(--color-foreground);
  font-weight: 600;
}

.case-study-content :deep(ul),
.case-study-content :deep(ol) {
  display: grid;
  gap: 0.65em;
  margin: 1.25em 0;
  padding-left: 1.35em;
}

.case-study-content :deep(ul) {
  list-style: disc;
}

.case-study-content :deep(ol) {
  list-style: decimal;
}

.case-study-content :deep(blockquote) {
  margin: 3em 0;
  border: none;
  padding: 3em 2em;
  background-color: var(--color-muted);
  border-radius: 0.5em;
  color: var(--color-foreground);
  font-size: 1.5em;
  font-weight: 400;
  line-height: 1.125em;
  letter-spacing: -0.015em;
  text-wrap: balance;
  text-align: center;
}

.case-study-content :deep(blockquote p) {
  margin: 0;
}

.case-study-content :deep(blockquote cite) {
  display: block;
  margin-top: 3em;
  color: var(--color-foreground);
  opacity: 0.5;
  font-size: 0.625em;
  font-style: normal;
  line-height: 1.4;
}

.case-study-content :deep(figure) {
  margin: 2em 0;
}

.case-study-content :deep(img) {
  width: 100%;
  border-radius: 0.5em;
  background: var(--color-muted);
}

.case-study-content :deep(code) {
  border-radius: 0.25em;
  background: var(--color-muted);
  padding: 0.125em 0.35em;
  color: var(--color-foreground);
  font-size: 0.92em;
}
</style>
