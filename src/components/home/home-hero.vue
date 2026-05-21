<script setup>
import Button from '../global/button.vue'
import { gsap } from 'gsap'
import { Shader, Godrays, Fog, Swirl, ChromaticAberration } from 'shaders/vue'
import { h, onBeforeUnmount, onMounted, ref } from 'vue'

const emit = defineEmits(['book-demo'])

const heroSectionRef = ref(null)
const heroShaderRef = ref(null)
const heroCopyRef = ref(null)

const heroMessages = [
  {
    text: 'Prep for my 2pm meeting',
    left: '50%',
    top: '31%',
    startDepth: -720,
    endDepth: 520,
    scale: 1.06,
    duration: 24,
    delay: -8,
    layer: 7,
  },
  {
    text: 'Draft the client follow-up',
    mentionRemi: true,
    left: '18%',
    top: '20%',
    startDepth: -980,
    endDepth: 440,
    scale: 0.82,
    duration: 31,
    delay: -19,
    layer: 1,
  },
  {
    text: 'Update CRM from this call',
    left: '82%',
    top: '18%',
    startDepth: -860,
    endDepth: 500,
    scale: 0.9,
    duration: 28,
    delay: -4,
    layer: 3,
  },
  {
    text: 'competitive-analysis.pdf',
    kind: 'file',
    left: '44%',
    top: '22%',
    startDepth: -920,
    endDepth: 470,
    scale: 0.88,
    duration: 30,
    delay: -14,
    layer: 3,
  },
  {
    text: 'Find the launch blockers',
    left: '24%',
    top: '46%',
    startDepth: -640,
    endDepth: 570,
    scale: 0.98,
    duration: 22,
    delay: -13,
    layer: 5,
  },
  {
    text: 'Reconcile open invoices',
    left: '76%',
    top: '43%',
    startDepth: -700,
    endDepth: 590,
    scale: 1.02,
    duration: 25,
    delay: -21,
    layer: 6,
  },
  {
    text: 'annual-report.pdf',
    kind: 'file',
    left: '74%',
    top: '25%',
    startDepth: -760,
    endDepth: 560,
    scale: 0.96,
    duration: 26,
    delay: -17,
    layer: 5,
  },
  {
    text: 'Summarize Slack decisions',
    left: '14%',
    top: '56%',
    startDepth: -940,
    endDepth: 460,
    scale: 0.86,
    duration: 34,
    delay: -2,
    layer: 2,
  },
  {
    text: 'Prep tomorrow\'s agenda',
    left: '86%',
    top: '55%',
    startDepth: -1040,
    endDepth: 430,
    scale: 0.84,
    duration: 36,
    delay: -25,
    layer: 1,
  },
  {
    text: 'Assign next steps to owners',
    left: '38%',
    top: '15%',
    startDepth: -780,
    endDepth: 540,
    scale: 0.94,
    duration: 27,
    delay: -15,
    layer: 4,
  },
  {
    text: 'pipeline-export.xlsx',
    kind: 'file',
    left: '27%',
    top: '28%',
    startDepth: -1080,
    endDepth: 420,
    scale: 0.8,
    duration: 38,
    delay: -9,
    layer: 1,
  },
  {
    text: 'Chase missing approvals',
    mentionRemi: true,
    left: '62%',
    top: '13%',
    startDepth: -900,
    endDepth: 520,
    scale: 0.9,
    duration: 32,
    delay: -11,
    layer: 3,
  },
  {
    text: 'Compare vendor quotes',
    left: '8%',
    top: '36%',
    startDepth: -1120,
    endDepth: 410,
    scale: 0.78,
    duration: 39,
    delay: -28,
    layer: 1,
  },
  {
    text: 'board-minutes.docx',
    kind: 'file',
    left: '55%',
    top: '43%',
    startDepth: -840,
    endDepth: 500,
    scale: 0.9,
    duration: 29,
    delay: -20,
    layer: 3,
  },
  {
    text: 'Turn notes into tasks',
    left: '91%',
    top: '34%',
    startDepth: -760,
    endDepth: 560,
    scale: 0.94,
    duration: 26,
    delay: -7,
    layer: 4,
  },
  {
    text: 'Check renewal risks',
    mentionRemi: true,
    left: '32%',
    top: '38%',
    startDepth: -680,
    endDepth: 600,
    scale: 1,
    duration: 23,
    delay: -18,
    layer: 5,
  },
  {
    text: 'Draft the weekly update',
    left: '68%',
    top: '33%',
    startDepth: -820,
    endDepth: 500,
    scale: 0.92,
    duration: 29,
    delay: -23,
    layer: 3,
  },
  {
    text: 'client-renewals.csv',
    kind: 'file',
    left: '6%',
    top: '50%',
    startDepth: -980,
    endDepth: 450,
    scale: 0.82,
    duration: 33,
    delay: -26,
    layer: 2,
  },
  {
    text: 'vendor-contract.pdf',
    kind: 'file',
    left: '94%',
    top: '48%',
    startDepth: -880,
    endDepth: 520,
    scale: 0.9,
    duration: 31,
    delay: -12,
    layer: 3,
  },
  {
    text: 'Clean up stale tickets',
    left: '8%',
    top: '76%',
    startDepth: -1060,
    endDepth: 430,
    scale: 0.8,
    duration: 37,
    delay: -5,
    layer: 1,
  },
  {
    text: 'Brief me before the call',
    left: '20%',
    top: '63%',
    startDepth: -600,
    endDepth: 610,
    scale: 1.04,
    duration: 21,
    delay: -9,
    layer: 6,
  },
  {
    text: 'Route this to finance',
    mentionRemi: true,
    left: '92%',
    top: '75%',
    startDepth: -960,
    endDepth: 470,
    scale: 0.86,
    duration: 35,
    delay: -31,
    layer: 2,
  },
]

const HERO_MESSAGE_FADE_IN_END = 0.01
const HERO_MESSAGE_FADE_OUT_START = 0.88
const HERO_MESSAGE_FADE_OUT_END = 0.895
const HERO_MESSAGE_SETTLE_MS = 1600
const HERO_MESSAGE_INTRO_SPEED_BOOST = 6
const HERO_MESSAGE_FOG_FAR_DEPTH = -1120
const HERO_MESSAGE_FOG_NEAR_DEPTH = -120
const HERO_MESSAGE_MIN_FOG_VISIBILITY = 0.28
const HERO_MESSAGE_MIN_DISTANCE = 16
const HERO_MESSAGE_RELAXED_MIN_DISTANCE = 11
const HERO_MESSAGE_MAX_PLACEMENT_ATTEMPTS = 44
const HERO_MESSAGE_DISTANCE_X_WEIGHT = 1.15
const HERO_MESSAGE_PLACEMENT_ZONES = [
  { left: [8, 92], top: [12, 30], weight: 0.4 },
  { left: [6, 94], top: [31, 50], weight: 0.38 },
  { left: [6, 24], top: [51, 78], weight: 0.11 },
  { left: [76, 94], top: [51, 78], weight: 0.11 },
]

const getInitialProgress = (message) => {
  const offset = Math.abs(message.delay) / message.duration

  return offset % 1
}

const getPercentValue = (value) => Number.parseFloat(value)

const getRandomInRange = ([min, max]) => min + Math.random() * (max - min)

const getRandomPlacementZone = () => {
  const totalWeight = HERO_MESSAGE_PLACEMENT_ZONES.reduce((total, zone) => total + zone.weight, 0)
  let threshold = Math.random() * totalWeight

  for (const zone of HERO_MESSAGE_PLACEMENT_ZONES) {
    threshold -= zone.weight

    if (threshold <= 0) {
      return zone
    }
  }

  return HERO_MESSAGE_PLACEMENT_ZONES[0]
}

const getRandomHeroMessagePosition = () => {
  const zone = getRandomPlacementZone()

  return {
    left: getRandomInRange(zone.left),
    top: getRandomInRange(zone.top),
  }
}

const isLowerCenterPosition = (position) => (
  position.top >= 58 &&
  position.left >= 28 &&
  position.left <= 72
)

const getHeroMessagePositionDistance = (positionA, positionB) => {
  const xDistance = (positionA.left - positionB.left) * HERO_MESSAGE_DISTANCE_X_WEIGHT
  const yDistance = positionA.top - positionB.top

  return Math.sqrt(xDistance * xDistance + yDistance * yDistance)
}

const getHeroMessageOpacityFactor = (progress) => {
  if (progress < HERO_MESSAGE_FADE_IN_END) {
    return progress / HERO_MESSAGE_FADE_IN_END
  }

  if (progress < HERO_MESSAGE_FADE_OUT_START) {
    return 1
  }

  if (progress < HERO_MESSAGE_FADE_OUT_END) {
    return 1 - (progress - HERO_MESSAGE_FADE_OUT_START) / (HERO_MESSAGE_FADE_OUT_END - HERO_MESSAGE_FADE_OUT_START)
  }

  return 0
}

const getHeroMessageIntroSpeed = (elapsedMs) => {
  const progress = Math.min(elapsedMs / HERO_MESSAGE_SETTLE_MS, 1)
  const remaining = 1 - progress

  return 1 + HERO_MESSAGE_INTRO_SPEED_BOOST * remaining * remaining
}

const getHeroMessageFogVisibility = (depth) => {
  const progress = Math.min(
    Math.max((depth - HERO_MESSAGE_FOG_FAR_DEPTH) / (HERO_MESSAGE_FOG_NEAR_DEPTH - HERO_MESSAGE_FOG_FAR_DEPTH), 0),
    1,
  )

  return HERO_MESSAGE_MIN_FOG_VISIBILITY + (1 - HERO_MESSAGE_MIN_FOG_VISIBILITY) * progress * progress
}

const heroMessageProgress = heroMessages.map(getInitialProgress)
const heroMessagePositions = heroMessages.map((message) => ({
  left: getPercentValue(message.left),
  top: getPercentValue(message.top),
}))

const getHeroMessagePlacementScore = (candidate, messageIndex) => {
  if (isLowerCenterPosition(candidate)) {
    return -1
  }

  return heroMessagePositions.reduce((closestDistance, position, index) => {
    if (index === messageIndex || getHeroMessageOpacityFactor(heroMessageProgress[index]) === 0) {
      return closestDistance
    }

    return Math.min(closestDistance, getHeroMessagePositionDistance(candidate, position))
  }, Number.POSITIVE_INFINITY)
}

const getConstrainedHeroMessagePosition = (messageIndex) => {
  let bestCandidate = null
  let bestScore = -1

  for (let attempt = 0; attempt < HERO_MESSAGE_MAX_PLACEMENT_ATTEMPTS; attempt += 1) {
    const candidate = getRandomHeroMessagePosition()
    const minDistance = attempt < HERO_MESSAGE_MAX_PLACEMENT_ATTEMPTS * 0.65
      ? HERO_MESSAGE_MIN_DISTANCE
      : HERO_MESSAGE_RELAXED_MIN_DISTANCE
    const score = getHeroMessagePlacementScore(candidate, messageIndex)

    if (score > bestScore) {
      bestCandidate = candidate
      bestScore = score
    }

    if (score >= minDistance) {
      return candidate
    }
  }

  return bestCandidate || getRandomHeroMessagePosition()
}

const rebalanceHeroMessagePositions = () => {
  heroMessages.forEach((message, index) => {
    if (getHeroMessageOpacityFactor(heroMessageProgress[index]) > 0) {
      heroMessagePositions[index] = getConstrainedHeroMessagePosition(index)
    }
  })
}

const heroBubbleRevealState = {
  opacity: 0,
}

const createHeroMessageStyle = (message, progress, position) => {
  const depth = message.startDepth + (message.endDepth - message.startDepth) * progress
  const opacity = getHeroMessageOpacityFactor(progress) * heroBubbleRevealState.opacity

  return {
    left: `${position.left}%`,
    top: `${position.top}%`,
    zIndex: Math.round(progress * 1000) + message.layer,
    opacity,
    '--message-fog-visibility': getHeroMessageFogVisibility(depth),
    transform: `translate3d(-50%, -50%, ${depth}px) scale(${message.scale})`,
  }
}

const heroMessageStyles = ref(heroMessages.map((message, index) => (
  createHeroMessageStyle(message, heroMessageProgress[index], heroMessagePositions[index])
)))

let heroMessageAnimationFrame = 0
let heroMessageStartTime = 0
let heroMessageLastFrameTime = 0
let heroIntroContext = null

const updateHeroMessageStyles = (now) => {
  const elapsedMs = now - heroMessageStartTime
  const deltaSeconds = Math.min((now - heroMessageLastFrameTime) / 1000, 0.05)
  const speed = getHeroMessageIntroSpeed(elapsedMs)

  heroMessageLastFrameTime = now
  heroMessageStyles.value = heroMessages.map((message, index) => {
    const previousProgress = heroMessageProgress[index]
    const nextProgress = (previousProgress + (deltaSeconds / message.duration) * speed) % 1

    if (nextProgress < previousProgress) {
      heroMessagePositions[index] = getConstrainedHeroMessagePosition(index)
    }

    heroMessageProgress[index] = nextProgress

    return createHeroMessageStyle(message, nextProgress, heroMessagePositions[index])
  })

  heroMessageAnimationFrame = requestAnimationFrame(updateHeroMessageStyles)
}

const playHeroIntroTimeline = () => {
  heroIntroContext = gsap.context(() => {
    const timeline = gsap.timeline({
      defaults: {
        ease: 'power2.out',
      },
    })

    timeline
      .fromTo(heroShaderRef.value, {
        autoAlpha: 0,
      }, {
        autoAlpha: 1,
        duration: 0.9,
      }, 0)
      .fromTo(heroBubbleRevealState, {
        opacity: 0,
      }, {
        opacity: 1,
        duration: 1.1,
      }, 0.35)
      .fromTo(heroCopyRef.value, {
        autoAlpha: 0,
        y: 24,
      }, {
        autoAlpha: 1,
        duration: 0.8,
        ease: 'power3.out',
        y: 0,
      }, 0.3)
  }, heroSectionRef.value)
}

onMounted(() => {
  const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches

  rebalanceHeroMessagePositions()

  if (prefersReducedMotion) {
    heroBubbleRevealState.opacity = 1
    heroMessageStyles.value = heroMessages.map((message, index) => (
      createHeroMessageStyle(message, 0.35, heroMessagePositions[index])
    ))
    gsap.set(heroShaderRef.value, {
      autoAlpha: 1,
    })
    gsap.set(heroCopyRef.value, {
      autoAlpha: 1,
      y: 0,
    })
    return
  }

  heroMessageStartTime = performance.now()
  heroMessageLastFrameTime = heroMessageStartTime
  heroMessageAnimationFrame = requestAnimationFrame(updateHeroMessageStyles)
  playHeroIntroTimeline()
})

onBeforeUnmount(() => {
  if (heroMessageAnimationFrame) {
    cancelAnimationFrame(heroMessageAnimationFrame)
  }

  if (heroIntroContext) {
    heroIntroContext.revert()
  }
})

const MessageBubble = {
  name: 'MessageBubble',
  props: {
    kind: {
      type: String,
      default: 'message',
    },
  },
  setup(props, { slots }) {
    return () =>
      h(
        'div',
        {
          class: [
            'max-w-[min(72vw,22em)] whitespace-nowrap rounded-[999em] px-[0.875em] py-[0.5em] text-[1em] font-normal leading-[1.25em] tracking-tight',
            props.kind === 'file' ? 'bg-white text-blue-500' : 'bg-blue-500/80 text-white',
          ],
        },
        slots.default ? slots.default() : null
      )
  }
}
</script>

<template>
  <section ref="heroSectionRef" class="relative w-full min-h-[clamp(800px,100svh,100svh)] grid grid-rows-1 grid-cols-1 md:py-4">
    <div ref="heroShaderRef" class="hero-shader absolute inset-0 h-full w-full opacity-0 mask-b-from-0% mask-b-to-100%">
      <Shader class="absolute inset-0 h-full w-full" disable-telemetry>
        <ChromaticAberration :strength="1">
          <Swirl color-a="oklch(0.99 0.2 259.597)" color-b="oklch(0.8 0.2 259.597)" color-space="oklch" :speed="1"
            :detail="1.5" :blend="50" />
          <Fog :animated="true" :speed="0.5" :color-a="'oklch(1 0 0)'" :color-b="'oklch(1 0 0/0)'"
            :mouse-influence="0.5" />
          <Godrays :animated="true" :center="{ x: 0.5, y: -1 }" :density="0.25" :intensity="1" :spotty="0.1" :speed="2"
            rayColor="oklch(0.95 0.3 259.597)" />
        </ChromaticAberration>
      </Shader>
    </div>
    <div class="w-full max-w-[1400px] px-6 py-24 mx-auto flex flex-col items-center justify-end text-center gap-8">
      <div class="z-1 flex flex-1">
        <div
          class="pointer-events-none absolute inset-0 z-1 overflow-hidden perspective-[1100px] transform-3d"
          aria-hidden="true"
        >
          <div class="absolute inset-x-0 top-0 h-4/5 transform-3d text-[clamp(0.75rem,1vw,0.95rem)]">
            <div
              v-for="(message, index) in heroMessages"
              :key="message.text"
              class="hero-message absolute will-change-transform transform-3d"
              :style="heroMessageStyles[index]"
            >
              <div class="hero-message-backdrop relative rounded-[999em]">
                <div class="hero-message-fog">
                  <MessageBubble :kind="message.kind">
                    <span
                      v-if="message.mentionRemi"
                      class="mention relative ml-[-0.375em] mr-[0.5em] px-[0.25em] font-medium text-white before:content-[''] before:absolute before:inset-x-[-0.125em] before:inset-y-[-0.25em] before:z-0 before:rounded-[999em] before:bg-white/20"
                    >
                      <span class="relative z-1">@Remi</span>
                    </span>
                    <span class="relative z-1">{{ message.text }}</span>
                  </MessageBubble>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div ref="heroCopyRef" class="hero-copy relative z-1 flex translate-y-6 flex-col items-center gap-8 opacity-0">
        <h1
          class="max-w-[20ch] text-[clamp(3.5rem,8vw,5.5rem)] leading-[1em] font-normal tracking-[-0.03em] text-balance text-foreground select-none">
          The last operations hire you'll ever need
        </h1>
        <p class="max-w-[48ch] text-base leading-normal tracking-tight text-muted-foreground">
          AI-powered operations assistant that learns how your business runs and gets work done for you.
        </p>
        <div class="flex items-center justify-start gap-2">
          <Button>Book a Demo</Button>
          <Button variant="secondary">Pricing</Button>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
.hero-shader {
  will-change: opacity;
}

.hero-copy {
  will-change: opacity, transform;
}

.hero-message {
  transform-origin: center;
  will-change: opacity, transform;
}

.hero-message-fog {
  opacity: var(--message-fog-visibility);
  will-change: opacity;
}

.hero-message-backdrop::before {
  position: absolute;
  inset: 0;
  border-radius: inherit;
  background: rgb(255 255 255 / 0.06);
  backdrop-filter: blur(0.4em);
  content: '';
  -webkit-backdrop-filter: blur(0.4em);
}
</style>
