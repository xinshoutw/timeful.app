<template>
  <div
    class="tw-mx-auto tw-mt-4 tw-flex tw-h-full tw-max-w-5xl tw-flex-col tw-items-center tw-justify-center"
  >
    <h2 v-if="state === states.CONFIRMING" class="tw-px-4 tw-text-2xl">
      確認回覆中...
    </h2>
    <h2
      v-else-if="state === states.CONFIRMED"
      class="tw-px-4 tw-text-base sm:tw-text-lg"
    >
      你的回覆已確認！可以關閉這個分頁了
    </h2>
    <h2
      v-else-if="state === states.ERROR"
      class="tw-px-4 tw-text-base sm:tw-text-lg"
    >
      確認回覆時發生錯誤，請重新整理頁面後再試一次
    </h2>
  </div>
</template>

<script>
import { post } from "@/utils"

export default {
  name: "Responded",

  props: {
    eventId: { type: String, required: true },
  },

  data() {
    return {
      state: "confirming",
      states: {
        CONFIRMING: "confirming",
        CONFIRMED: "confirmed",
        ERROR: "error",
      },
    }
  },

  created() {
    let { email } = this.$route.query
    post(`/events/${this.eventId}/responded`, { email })
      .then(() => {
        this.state = this.states.CONFIRMED
      })
      .catch((err) => {
        console.error(err)
        this.state = this.states.ERROR
      })
  },
}
</script>
