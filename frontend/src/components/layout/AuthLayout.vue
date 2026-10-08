<template>
  <div class="relative min-h-screen overflow-hidden bg-slate-50 dark:bg-[#080f1e]">
    <div class="pointer-events-none absolute inset-0 overflow-hidden" aria-hidden="true">
      <div class="absolute -left-40 top-10 h-96 w-96 rounded-full bg-blue-300/20 blur-3xl dark:bg-blue-500/10"></div>
      <div class="absolute -right-40 bottom-0 h-[30rem] w-[30rem] rounded-full bg-cyan-300/20 blur-3xl dark:bg-cyan-400/10"></div>
    </div>

    <main class="relative mx-auto grid min-h-screen w-full max-w-[1440px] gap-4 p-3 sm:p-5 lg:grid-cols-[minmax(0,1.05fr)_minmax(420px,0.95fr)] lg:gap-5">
      <section
        class="relative flex min-h-[250px] flex-col justify-between overflow-hidden rounded-[28px] bg-[#0b1730] p-7 text-white shadow-2xl shadow-blue-950/15 sm:p-10 lg:min-h-[calc(100vh-40px)] lg:p-12"
      >
        <div class="pointer-events-none absolute inset-0 overflow-hidden" aria-hidden="true">
          <div class="absolute -right-24 -top-28 h-96 w-96 rounded-full bg-blue-500/20 blur-3xl"></div>
          <div class="absolute -bottom-40 -left-24 h-[28rem] w-[28rem] rounded-full bg-cyan-400/15 blur-3xl"></div>
          <div class="absolute inset-0 opacity-[0.12] [background-image:linear-gradient(rgba(148,190,255,0.28)_1px,transparent_1px),linear-gradient(90deg,rgba(148,190,255,0.28)_1px,transparent_1px)] [background-size:56px_56px] [mask-image:linear-gradient(to_bottom,black,transparent_78%)]"></div>
          <div class="absolute -right-14 top-1/2 hidden h-72 w-72 -translate-y-1/2 rounded-full border border-blue-200/10 lg:block"></div>
          <div class="absolute -right-2 top-1/2 hidden h-48 w-48 -translate-y-1/2 rounded-full border border-cyan-200/10 lg:block"></div>
        </div>

        <div class="relative z-10 flex items-center gap-3">
          <img :src="siteLogo || '/logo.svg'" alt="" class="h-12 w-12 rounded-2xl shadow-lg shadow-blue-950/40" />
          <span class="text-lg font-semibold tracking-tight">{{ siteName }}</span>
        </div>

        <div class="relative z-10 max-w-xl py-8 lg:py-0">
          <div class="mb-5 inline-flex items-center gap-2 rounded-full border border-blue-200/15 bg-white/[0.06] px-3 py-1.5 text-xs font-medium tracking-wide text-blue-100/90 backdrop-blur">
            <span class="h-1.5 w-1.5 rounded-full bg-cyan-300 shadow-[0_0_12px_rgba(103,232,249,0.9)]"></span>
            AI API WORKSPACE
          </div>
          <h1 class="max-w-lg text-3xl font-semibold leading-tight tracking-tight sm:text-4xl lg:text-5xl">
            {{ siteSubtitle }}
          </h1>
          <p class="mt-5 max-w-md text-sm leading-6 text-slate-300/80 sm:text-base sm:leading-7">
            {{ supportCopy }}
          </p>
        </div>

        <div class="relative z-10 hidden items-center gap-2 text-xs text-slate-400 lg:flex">
          <span class="h-px w-8 bg-gradient-to-r from-cyan-300 to-blue-400"></span>
          <span>{{ currentYear }} · {{ siteName }}</span>
        </div>
      </section>

      <section class="flex min-h-[480px] items-center justify-center rounded-[28px] border border-white/70 bg-white/75 px-4 py-10 shadow-xl shadow-slate-900/[0.04] backdrop-blur-xl dark:border-white/10 dark:bg-slate-900/70 dark:shadow-black/20 sm:px-10 lg:min-h-[calc(100vh-40px)] lg:px-12">
        <div class="w-full max-w-md">
          <div class="rounded-2xl sm:px-2">
            <slot />
          </div>

          <div class="mt-6 text-center text-sm">
            <slot name="footer" />
          </div>

          <div class="mt-8 text-center text-xs text-slate-400 dark:text-slate-500 lg:hidden">
            &copy; {{ currentYear }} {{ siteName }}. All rights reserved.
          </div>
        </div>
      </section>
    </main>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { useAppStore } from '@/stores'
import { sanitizeUrl } from '@/utils/url'

const appStore = useAppStore()
const { locale } = useI18n()

const siteName = computed(() => appStore.siteName || 'ThisAI')
const siteLogo = computed(() => sanitizeUrl(appStore.siteLogo || '', { allowRelative: true, allowDataUrl: true }))
const siteSubtitle = computed(() => appStore.cachedPublicSettings?.site_subtitle || 'AI API 接入与管理 · Unified AI Gateway')
const supportCopy = computed(() => locale.value.toLowerCase().startsWith('zh')
  ? '统一管理 AI 工具接入与 API 用量。'
  : 'Connect your AI tools to one carefully managed gateway.')
const currentYear = computed(() => new Date().getFullYear())

onMounted(() => {
  appStore.fetchPublicSettings()
})
</script>
