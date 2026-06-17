<template>
  <footer class="tw-bg-green tw-p-8 tw-text-white">
    <div class="tw-mx-auto tw-max-w-7xl">
      <div class="tw-flex tw-flex-col tw-items-center tw-gap-4 sm:tw-flex-row sm:tw-justify-between">
        <div>
          <div class="tw-mb-3 tw-flex tw-items-center">
            <v-icon class="tw-mr-1 tw-text-white">mdi-calendar-blank</v-icon>
            <span class="tw-text-2xl tw-font-medium">timeful</span>
          </div>
          <div class="tw-mb-4 tw-leading-snug">
            Made with 💚 by
            <a
              href="https://x.com/jonathanzliu/"
              target="_blank"
              class="tw-text-white tw-underline"
              >@jonathanzliu</a
            >
          </div>
          <div class="tw-flex tw-items-center tw-space-x-3">
            <a
              href="https://github.com/schej-it/timeful.app"
              aria-label="GitHub"
              target="_blank"
              ><v-icon class="tw-text-white/80 hover:tw-text-white"
                >mdi-github</v-icon
              ></a
            >
          </div>
        </div>
        <div class="tw-flex tw-flex-col tw-gap-2">
          <router-link :to="{ name: 'privacy-policy' }" class="tw-text-sm">
            Privacy Policy
          </router-link>
        </div>
      </div>
    </div>
  </footer>
</template>

<style scoped>
div:has(a) {
  @apply tw-leading-none;
}
a {
  @apply tw-text-white/80 !important;
}
a:hover {
  @apply tw-text-white !important;
}
</style>

<script>
export default {
  name: "Footer",
}
</script>
