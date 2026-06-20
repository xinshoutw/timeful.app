<template>
  <v-dialog v-model="dialog" max-width="400px" content-class="tw-m-0">
    <v-card>
      <v-card-title>
        <span class="tw-text-xl tw-font-medium">功能尚未開放</span>
        <v-spacer />
        <v-btn
          absolute
          @click="dialog = false"
          icon
          class="tw-right-0 tw-mr-2 tw-self-center"
        >
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </v-card-title>
      <v-card-text class="tw-text-very-dark-gray">
        你來得有點早！我們正在考慮為 Timeful 加入資料夾功能，一旦有足夠的使用者需求就會推出
        <v-textarea
          v-model="folderUsageFeedback"
          label="你想要用資料夾來做什麼？"
          rows="3"
          class="tw-mt-4"
          outlined
          dense
        ></v-textarea>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn text @click="dialog = false">關閉</v-btn>
        <v-btn color="primary" @click="submitFeedback">送出</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import { mapActions } from "vuex"

export default {
  name: "FeatureNotReadyDialog",
  props: {
    value: Boolean,
  },
  data() {
    return {
      folderUsageFeedback: "",
    }
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
  methods: {
    ...mapActions(["showInfo"]),
    submitFeedback() {
      if (this.folderUsageFeedback.trim() !== "") {
        this.$posthog?.capture("folder_usage_feedback_submitted", {
          feedback: this.folderUsageFeedback,
        })
        // Optionally, you can clear the textarea and close the dialog
        this.folderUsageFeedback = ""
        this.dialog = false
        this.showInfo("感謝你的回饋！")
      } else {
        // Optionally, handle empty feedback (e.g., show a message)
        console.log("Feedback is empty")
      }
    },
  },
}
</script>

<style scoped></style>
