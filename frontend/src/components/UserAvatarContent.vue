<template>
  <v-avatar v-if="user" :size="size">
    <img
      v-if="user.picture"
      :src="user.picture"
      referrerpolicy="no-referrer"
    />
    <img
      v-else-if="gravatarUrl"
      :src="gravatarUrl"
      referrerpolicy="no-referrer"
      @error="gravatarFailed = true"
    />
    <div
      v-else
      :class="`tw-flex tw-size-full tw-items-center tw-justify-center tw-bg-[linear-gradient(-25deg,#2b6cb0,#63b3ed,#2b6cb0)] tw-text-${textSize} tw-text-white`"
    >
      {{ user.firstName?.charAt(0) ?? user.email?.charAt(0) ?? "" }}
    </div>
  </v-avatar>
</template>

<script>
async function sha256Hex(str) {
  const buf = await crypto.subtle.digest(
    "SHA-256",
    new TextEncoder().encode(str)
  )
  return Array.from(new Uint8Array(buf))
    .map((b) => b.toString(16).padStart(2, "0"))
    .join("")
}

export default {
  name: "UserAvatarContent",
  props: {
    user: Object,
    size: { type: Number, default: 48 },
  },

  data: () => ({
    emailHash: null,
    gravatarFailed: false,
  }),

  computed: {
    textSize() {
      return this.size <= 24 ? "xs" : "lg"
    },
    gravatarUrl() {
      if (this.gravatarFailed || !this.emailHash) return null
      return `https://gravatar.com/avatar/${this.emailHash}?s=${this.size * 2}&d=404`
    },
  },

  watch: {
    "user.email": {
      immediate: true,
      async handler(email) {
        if (!email) {
          this.emailHash = null
          return
        }
        this.gravatarFailed = false
        this.emailHash = await sha256Hex(email.trim().toLowerCase())
      },
    },
  },
}
</script>
