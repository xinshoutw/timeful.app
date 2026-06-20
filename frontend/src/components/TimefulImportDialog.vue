<template>
  <v-dialog v-model="dialog" max-width="500px" content-class="tw-m-0">
    <v-card>
      <v-card-title>
        <span class="tw-text-xl tw-font-medium">匯入 Timeful 活動</span>
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
        <p class="tw-mb-4">
          貼上其他 Timeful 實例的活動網址，即可連同所有現有回覆一起匯入
        </p>
        <v-text-field
          v-model="url"
          label="活動網址"
          placeholder="https://timeful.app/e/abc123"
          outlined
          dense
          :disabled="loading"
          :error-messages="error"
          @keydown.enter="importEvent"
        />
      </v-card-text>
      <v-card-actions>
        <v-spacer />
        <v-btn text @click="dialog = false" :disabled="loading">取消</v-btn>
        <v-btn
          color="primary"
          :loading="loading"
          :disabled="!url.trim() || loading"
          @click="importEvent"
        >
          匯入
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import { post } from "@/utils"

export default {
  name: "TimefulImportDialog",
  props: {
    value: Boolean,
  },
  data: () => ({
    url: "",
    loading: false,
    error: "",
  }),
  computed: {
    dialog: {
      get() {
        return this.value
      },
      set(val) {
        this.$emit("input", val)
        if (!val) {
          this.url = ""
          this.error = ""
        }
      },
    },
  },
  methods: {
    isBlockedUrl(urlStr) {
      try {
        const parsed = new URL(urlStr)
        const hostname = parsed.hostname
        if (hostname === window.location.hostname) return true
        // note, this is just extra client-side validation for optimistic UI. the server checks the outgoing ip properly
        if (
          /^(10\.|192\.168\.|172\.(1[6-9]|2\d|3[01])\.|127\.|169\.254\.|0\.0\.0\.0|localhost$|\[?::1\]?)/.test(
            hostname
          )
        )
          return true
        return false
      } catch {
        return false
      }
    },
    async importEvent() {
      if (!this.url.trim() || this.loading) return

      if (this.isBlockedUrl(this.url.trim())) {
        this.error = "不允許從此網址匯入"
        return
      }

      this.error = ""
      this.loading = true

      try {
        const result = await post("/events/import", { url: this.url.trim() })
        this.dialog = false
        this.$store.dispatch("showInfo", "活動匯入成功！")
        this.$router.push(`/e/${result.shortId}`)
      } catch (e) {
        const msg = e?.parsed?.error || "匯入活動失敗"
        const errorMessages = {
          "invalid-url": "無效的網址。請輸入有效的 Timeful 活動網址",
          "remote-fetch-failed": "無法連線至遠端伺服器",
          "remote-event-not-found": "在遠端伺服器上找不到該活動",
          "private-address": "不允許從此網址匯入",
          "remote-responses-failed":
            "已找到活動，但無法從遠端伺服器取得回覆",
        }
        this.error = errorMessages[msg] || msg
      } finally {
        this.loading = false
      }
    },
  },
}
</script>
