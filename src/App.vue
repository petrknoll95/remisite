<script setup>
import { ref } from 'vue'
import AnnouncementBar from './components/global/announcement-bar.vue'
import BookDemoForm from './components/global/book-demo-form.vue'
import GlobalFooter from './components/global/global-footer.vue'
import GlobalHeader from './components/header/global-header.vue'
import GlobalModal from './components/global/global-modal.vue'
import HomeHero from './components/home/home-hero.vue'
import HomeSliderUiChat from './components/home/slider-ui/home-slider-ui-chat.vue'
import HomeSliderUiFiles from './components/home/slider-ui/home-slider-ui-files.vue'
import HomeSliderUiIntegrations from './components/home/slider-ui/home-slider-ui-integrations.vue'
import HomeSliderUiIntelligence from './components/home/slider-ui/home-slider-ui-intelligence.vue'
import SectionSurfaces from './components/sections/section-surfaces.vue'
import SectionCta from './components/sections/section-cta.vue'
import SectionFaq from './components/sections/section-faq.vue'
import SectionFeatureBento from './components/sections/section-feature-bento.vue'
import SectionHow from './components/sections/section-how.vue'
import SectionIntegrationsMarquee from './components/sections/section-integrations-marquee.vue'
import SectionProductOverview from './components/sections/section-product-overview.vue'
import SectionRemiVsTools from './components/sections/section-remi-vs-tools.vue'
import SectionSlider from './components/sections/section-slider.vue'

const isDemoModalOpen = ref(false)
</script>

<template>
  <div>
    <GlobalHeader @book-demo="isDemoModalOpen = true" />
    <main class="min-h-svh bg-background text-foreground overscroll-none">
      <HomeHero @book-demo="isDemoModalOpen = true" />
      <SectionHow />
      <SectionFeatureBento @book-demo="isDemoModalOpen = true" />
      <SectionRemiVsTools />
      <SectionSlider>
        <template #tasks>
          <HomeSliderUiChat />
        </template>
        <template #integrations>
          <HomeSliderUiIntegrations />
        </template>
        <template #files>
          <HomeSliderUiFiles />
        </template>
        <template #intelligence>
          <HomeSliderUiIntelligence />
        </template>
      </SectionSlider>
      <SectionProductOverview />
      <SectionSurfaces />
      <SectionIntegrationsMarquee />
      <SectionFaq />
      <SectionCta @book-demo="isDemoModalOpen = true" />
    </main>
    <GlobalFooter />
  </div>
  <GlobalModal v-model="isDemoModalOpen" label="Book a Demo">
    <BookDemoForm />
  </GlobalModal>
</template>
