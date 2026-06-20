<template>
  <v-dialog
    :value="value"
    @input="(e) => $emit('input', e)"
    width="400"
    content-class="tw-m-0"
  >
    <v-card>
      <v-card-title>未儲存的變更</v-card-title>
      <v-card-text>
        <div>確定要離開這個頁面嗎？</div>
        <div>你所做的變更將不會被儲存</div>
      </v-card-text>
      <v-card-actions>
        <v-spacer />
        <v-btn text @click="$emit('input', false)" class="tw-text-green"
          >取消</v-btn
        >
        <v-btn text @click="$emit('leave')">離開頁面</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  name: "UnsavedChangesDialog",

  props: {
    value: { type: Boolean, required: true },
  },
}
</script>
