<template>
  <div class="mx-auto tw-w-full md:tw-w-2/3 lg:tw-w-1/2 tw-px-5 tw-pt-10">
    <h2 class="tw-mb-2.5 tw-text-xl tw-font-semibold">Cookie 偏好設定</h2>
    <div class="tw-mb-5 tw-flex tw-flex-col tw-gap-6">
      <div class="tw-rounded-lg tw-border tw-bg-white tw-p-5">
        <v-checkbox v-model="preferences.necessary" disabled>
          <template v-slot:label>
            <div>
              <strong class="tw-text-gray-800 tw-text-base tw-font-semibold">
                必要 Cookie
              </strong>
            </div>
          </template>
        </v-checkbox>
        <div
          class="tw-text-gray-600 tw-mt-4 tw-pl-0 tw-text-sm tw-leading-relaxed md:tw-mt-0 md:tw-pl-8"
        >
          <p>
            這些 Cookie 是網站正常運作所必需的，提供頁面導覽、使用者驗證和表單提交等基本功能
          </p>
        </div>
      </div>

      <div class="tw-rounded-lg tw-border tw-bg-white tw-p-5">
        <v-checkbox v-model="preferences.analytics">
          <template v-slot:label>
            <div>
              <strong class="tw-text-gray-800 tw-text-base tw-font-semibold">
                分析 Cookie
              </strong>
            </div>
          </template>
        </v-checkbox>
        <div
          class="tw-text-gray-600 tw-mt-4 tw-pl-0 tw-text-sm tw-leading-relaxed md:tw-mt-0 md:tw-pl-8"
        >
          <p>
            <strong>使用的服務：</strong>PostHog Analytics、Google Analytics（透過 Google Tag Manager）
          </p>
          <p>
            這些 Cookie 幫助我們了解訪客如何使用網站，透過匿名收集與回報資訊，協助我們改善網站效能與使用者體驗
          </p>
        </div>
      </div>

    </div>

    <div class="tw-flex tw-flex-wrap tw-gap-2">
      <button
        @click="savePreferences"
        class="tw-rounded-md tw-bg-blue tw-px-4 tw-py-2 tw-text-sm tw-font-medium tw-text-white hover:tw-bg-light-blue"
      >
        儲存偏好
      </button>
      <button
        @click="acceptAll"
        class="tw-rounded-md tw-bg-green tw-px-4 tw-py-2 tw-text-sm tw-font-medium tw-text-white hover:tw-bg-dark-green"
      >
        全部接受
      </button>
      <button
        @click="rejectAll"
        class="tw-rounded-md tw-px-4 tw-py-2 tw-text-sm tw-font-medium tw-text-white tw-bg-very-dark-gray"
      >
        全部拒絕（必要除外）
      </button>
    </div>
  </div>
</template>

<script>
import { getCookieConsent, setCookieConsent } from "@/utils/cookie_utils"

export default {
  name: "CookieSettings",
  data() {
    return {
      preferences: {
        necessary: true,
        analytics: true,
        advertising: true,
      },
    }
  },
  mounted() {
    this.loadCurrentSettings()
  },
  methods: {
    loadCurrentSettings() {
      const consent = getCookieConsent()
      if (consent) {
        this.preferences = { ...consent.preferences }
      }
    },

    savePreferences() {
      this.saveConsent()
    },

    acceptAll() {
      this.preferences = {
        necessary: true,
        analytics: true,
        advertising: true,
      }
      this.saveConsent()
    },

    rejectAll() {
      this.preferences = {
        necessary: true,
        analytics: false,
        advertising: false,
      }
      this.saveConsent()
    },
    saveConsent() {
      setCookieConsent(this.preferences)
      window.location.reload()
    },
  },
}
</script>
