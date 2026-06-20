<template>
  <!-- Google sign in not supported dialog -->
  <v-dialog :value="value" width="400" content-class="tw-m-0">
    <v-card>
      <v-card-title>不支援此瀏覽器登入</v-card-title>
      <v-card-text>
        <p>
          這通常發生在你從 Instagram 或 Messenger 開啟 Timeful 連結時
        </p>
        <p>
          請點擊右上角的三個點，然後選擇「在系統瀏覽器中開啟」或「在 Chrome 中開啟」來解決此問題
        </p>
      </v-card-text>
      <v-card-actions>
        <v-spacer />
        <v-btn text class="tw-text-green" @click="$emit('input', false)">
          好
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  name: "SignInNotSupportedDialog",

  props: {
    value: { type: Boolean, required: true },
  },
}
</script>
