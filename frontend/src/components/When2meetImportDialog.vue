<template>
  <v-dialog v-model="dialog" max-width="600px" content-class="tw-m-0">
    <v-card>
      <v-card-title>
        <span class="tw-text-xl tw-font-medium"
          >將 When2meet 轉換為 Timeful</span
        >
        <v-spacer />
        <v-btn
          absolute
          @click="$emit('input', false)"
          icon
          class="tw-right-0 tw-mr-2 tw-self-center"
        >
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </v-card-title>
      <v-card-text class="tw-text-very-dark-gray">
        <p class="tw-mb-4">
          如果有人不小心傳了 When2meet 連結給你，你可以按照以下步驟將它轉換為 Timeful：
        </p>
        <ol class="tw-mb-6 tw-list-decimal tw-pl-5">
          <li>前往你的 When2meet 活動連結</li>
          <li>在網址開頭加上「better」</li>
          <li>🪄💫✨</li>
          <li>將連結傳回群組聊天</li>
        </ol>
        <p>
          就這樣！日期範圍、時間範圍，以及所有現有的回覆都會一併複製過來
        </p>

        <p class="tw-mb-2 tw-font-medium">觀看操作方式：</p>
        <div class="video-container tw-mb-4">
          <!-- Replace YOUR_YOUTUBE_VIDEO_ID with the actual ID -->
          <iframe
            width="560"
            height="315"
            src="https://www.youtube.com/embed/k7t4ZplhtT8?si=oBv7cevr6gsz09wp"
            title="YouTube video player"
            frameborder="0"
            allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
            referrerpolicy="strict-origin-when-cross-origin"
            allowfullscreen
          ></iframe>
        </div>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="primary" text @click="dialog = false"> 了解！ </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  name: "When2meetImportDialog",
  props: {
    value: Boolean, // v-model
  },
  computed: {
    dialog: {
      get() {
        return this.value
      },
      set(val) {
        this.$emit("input", val)
      },
    },
  },
}
</script>

<style scoped>
.video-container {
  position: relative;
  padding-bottom: 56.25%; /* 16:9 aspect ratio */
  height: 0;
  overflow: hidden;
}

.video-container iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
