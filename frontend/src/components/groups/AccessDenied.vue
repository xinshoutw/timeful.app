<template>
  <div
    class="tw-flex tw-h-full tw-flex-col tw-items-center tw-justify-center tw-p-2"
  >
    <v-img
      alt="schejie sad"
      src="@/assets/schejie/sad.png"
      transition="fade-transition"
      contain
      class="tw-mb-6 tw-h-[150px] tw-flex-none sm:tw-h-[200px]"
    />
    <div class="tw-mb-8 tw-max-w-[30rem]">
      <h1 class="tw-mb-2 tw-text-center tw-text-2xl tw-font-medium">
        你需要存取權限
      </h1>
      <div class="tw-text-center tw-text-dark-gray">
        請要求群組擁有者將此群組分享給你的 Email：<br />
        <span class="tw-font-medium tw-text-green">{{ authUser.email }}</span>
      </div>
    </div>
    <v-btn href="/home" color="primary">回到首頁</v-btn>
  </div>
</template>

<script>
import { mapState } from "vuex"

export default {
  name: "AccessDenied",

  computed: {
    ...mapState(["authUser"]),
  },
}
</script>
