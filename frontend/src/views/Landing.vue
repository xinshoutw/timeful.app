<template>
  <div class="tw-bg-light-gray">
    <div
      class="tw-relative tw-m-auto tw-mb-12 tw-flex tw-max-w-6xl tw-flex-col tw-px-4 sm:tw-mb-20"
    >
      <!-- Header -->
      <div class="tw-mb-16 sm:tw-mb-28">
        <div class="tw-flex tw-items-center tw-pt-5">
          <Logo type="timeful" />

          <v-spacer />

          <LandingPageHeader>
            <div v-if="authUser" class="tw-ml-2">
              <AuthUserMenu />
            </div>
            <v-btn v-else text :to="{ name: 'sign-in' }">Sign in</v-btn>
          </LandingPageHeader>
        </div>
      </div>

      <div class="tw-flex tw-flex-col tw-items-center">
        <div
          class="tw-mb-6 tw-flex tw-max-w-[26rem] tw-flex-col tw-items-center sm:tw-w-[35rem] sm:tw-max-w-none"
        >
          <div
            id="header"
            class="tw-mb-4 tw-text-center tw-text-2xl tw-font-medium sm:tw-text-4xl lg:tw-text-4xl xl:tw-text-5xl"
          >
            <h1>Find a time to meet</h1>
          </div>

          <div
            class="lg:tw-text-md tw-text-left tw-text-center tw-text-sm tw-text-very-dark-gray sm:tw-text-lg md:tw-text-lg xl:tw-text-lg"
          >
            Coordinate group meetings without the back and forth.
            <br class="tw-hidden sm:tw-block" />
            Integrates with your
            <v-tooltip
              top
              content-class="tw-bg-very-dark-gray tw-shadow-lg tw-opacity-100"
            >
              <template v-slot:activator="{ on, attrs }">
                <span
                  class="tw-cursor-pointer tw-border-b tw-border-dashed tw-border-dark-gray"
                  v-bind="attrs"
                  v-on="on"
                  >calendar</span
                >
              </template>
              <span
                >Timeful allows you to autofill your availability from Google
                Calendar,<br class="tw-hidden sm:tw-block" />
                Outlook, Apple Calendar, or an ICS feed URL.</span
              > </v-tooltip
            >.
          </div>
        </div>

        <div class="tw-mb-12 tw-space-y-2">
          <v-btn
            class="tw-block tw-self-center tw-rounded-lg tw-bg-green tw-px-10 tw-text-base sm:tw-px-10 lg:tw-px-12"
            dark
            @click="authUser ? openDashboard() : (newDialog = true)"
            large
            :x-large="$vuetify.breakpoint.mdAndUp"
          >
            {{ authUser ? "Open dashboard" : "Create event" }}
          </v-btn>
          <div
            v-if="!authUser"
            class="tw-text-center tw-text-xs tw-text-dark-gray sm:tw-text-sm"
          >
            It's free! No login required.
          </div>
        </div>
        <div class="tw-relative tw-w-full">
          <!-- Green background -->
          <div
            class="tw-absolute -tw-bottom-12 tw-left-1/2 tw-h-[85%] tw-w-screen -tw-translate-x-1/2 tw-bg-green sm:-tw-bottom-20"
          ></div>

          <!-- Hero video -->
          <div
            class="tw-relative tw-z-20 tw-w-full tw-rounded-lg tw-border tw-border-light-gray-stroke tw-bg-white tw-shadow-xl sm:tw-rounded-xl md:tw-mx-auto md:tw-w-fit"
          >
            <div
              class="tw-relative tw-mx-4 tw-aspect-square md:tw-size-[700px] lg:tw-size-[800px]"
            >
              <v-img
                class="tw-absolute tw-left-0 tw-top-0 tw-z-20 tw-size-full tw-transition-opacity tw-duration-300"
                :class="{ 'tw-opacity-0': isVideoPlaying }"
                src="@/assets/img/hero.jpg"
                transition="fade-transition"
                contain
              />
              <vue-vimeo-player
                video-url="https://player.vimeo.com/video/1083205305?h=d58bef862a"
                :player-width="800"
                :player-height="800"
                :options="{
                  muted: true,
                  playsinline: true,
                  responsive: true,
                }"
                :controls="false"
                :autoplay="true"
                :loop="true"
                @play="onPlay"
              />
            </div>
          </div>
        </div>
      </div>
    </div>

    <Footer />

    <!-- New event dialog -->
    <NewDialog v-model="newDialog" no-tabs @signIn="signIn" />
  </div>
</template>

<style scoped>
@media screen and (min-width: 375px) and (max-width: 640px) {
  #header {
    font-size: 1.875rem !important; /* 30px */
    line-height: 2.25rem !important; /* 36px */
  }
}
</style>

<script>
import { isPhone } from "@/utils"
import NewDialog from "@/components/NewDialog.vue"
import LandingPageHeader from "@/components/landing/LandingPageHeader.vue"
import Logo from "@/components/Logo.vue"
import { vueVimeoPlayer } from "vue-vimeo-player"
import Footer from "@/components/Footer.vue"
import { mapState } from "vuex"
import AuthUserMenu from "@/components/AuthUserMenu.vue"

export default {
  name: "Landing",

  metaInfo: {
    title: "Timeful - Find a time to meet",
  },

  components: {
    NewDialog,
    LandingPageHeader,
    Logo,
    vueVimeoPlayer,
    Footer,
    AuthUserMenu,
  },

  data: () => ({
    newDialog: false,
    isVideoPlaying: false,
  }),

  computed: {
    ...mapState(["authUser"]),
    isPhone() {
      return isPhone(this.$vuetify)
    },
  },

  methods: {
    signIn() {
      this.$router.push({ name: "sign-in" })
    },
    onPlay() {
      setTimeout(() => {
        this.isVideoPlaying = true
      }, 1000)
    },
    openDashboard() {
      this.$router.push({ name: "home" })
    },
  },
}
</script>
