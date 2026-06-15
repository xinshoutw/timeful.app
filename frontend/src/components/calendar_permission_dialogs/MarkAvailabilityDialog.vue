<template>
  <v-dialog
    :value="value"
    @input="(e) => $emit('input', e)"
    width="400"
    content-class="tw-m-0"
  >
    <v-card class="tw-p-4 sm:tw-p-6">
      <div class="tw-text-md mb-1 tw-text-center">
        How would you like to add <br class="tw-block sm:tw-hidden" />
        your availability?
      </div>
      <div class="tw-pb-4 tw-text-center tw-text-xs tw-text-dark-gray">
        Draw on the calendar to mark when you're available
      </div>
      <div class="tw-flex tw-flex-col tw-gap-2">
        <v-btn @click="setAvailabilityManually" block color="primary">
          Add manually
        </v-btn>
      </div>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  name: "MarkAvailabilityDialog",

  props: {
    value: { type: Boolean, required: true },
    initialState: { type: String, default: "choices" },
  },

  methods: {
    setAvailabilityManually() {
      this.$emit("setAvailabilityManually")
    },
  },

  watch: {
    value() {
      // no-op, kept for interface compatibility
    },
  },
}
</script>
