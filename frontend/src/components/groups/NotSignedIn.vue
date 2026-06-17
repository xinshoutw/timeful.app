<template>
  <v-fade-transition>
    <div
      v-if="loaded"
      class="tw-flex tw-h-full tw-flex-col tw-items-center tw-justify-center tw-p-2"
    >
      <div class="tw-mb-8 tw-flex tw-max-w-[26rem] tw-flex-col tw-items-center">
        <UserAvatarContent
          :user="owner"
          :size="90"
          class="tw-mb-4 tw-text-center"
        />
        <h1 class="tw-mb-2 tw-text-center tw-text-xl tw-font-medium">
          {{ owner.firstName ?? "" }} 邀請你加入<br />「{{ event.name }}」
        </h1>
        <div class="tw-text-center tw-text-dark-gray">
          立即加入群組，<br v-if="!isPhone" />
          與彼此分享你有空的時間！
        </div>
      </div>
      <v-btn @click="signIn" color="primary" class="tw-mb-8">
        登入來加入
      </v-btn>
    </div>
  </v-fade-transition>
</template>

<script>
import { get, isPhone } from "@/utils"
import UserAvatarContent from "@/components/UserAvatarContent.vue"

export default {
  name: "NotSignedIn",

  props: {
    event: { type: Object, required: true },
  },

  components: {
    UserAvatarContent,
  },

  data() {
    return {
      owner: {},
      loaded: false,
    }
  },

  computed: {
    isPhone() {
      return isPhone(this.$vuetify)
    },
  },

  methods: {
    signIn() {
      this.$router.push({ name: "sign-in" })
    },
  },

  async created() {
    try {
      this.owner = await get(`/users/${this.event.ownerId}`)
    } catch {
      this.owner = { firstName: "", picture: "" }
    } finally {
      this.loaded = true
    }
  },
}
</script>
