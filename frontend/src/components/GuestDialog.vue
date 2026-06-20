<template>
  <v-dialog
    :value="value"
    @input="(e) => $emit('input', e)"
    width="400"
    content-class="tw-m-0"
  >
    <v-card>
      <v-card-title class="tw-flex">
        <div>以訪客身分繼續</div>
        <v-spacer />
        <v-btn icon @click="$emit('input', false)">
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </v-card-title>
      <v-card-text>
        <v-form
          ref="form"
          v-model="formValid"
          lazy-validation
          class="tw-flex tw-flex-col tw-gap-y-4"
          onsubmit="return false;"
        >
          <v-text-field
            v-model="name"
            @keyup.enter="submit"
            :rules="nameRules"
            placeholder="輸入你的名字..."
            autofocus
            hide-details="auto"
            autocomplete="off"
            solo
          ></v-text-field>
          <v-text-field
            v-if="event.collectEmails"
            v-model="email"
            @keyup.enter="submit"
            :rules="emailRules"
            placeholder="輸入你的 Email..."
            hint="活動建立者要求提供你的 Email，只有他們能看到"
            persistent-hint
            solo
          ></v-text-field>
          <div class="tw-flex">
            <v-spacer />
            <v-btn
              @click="submit"
              class="tw-bg-green"
              :dark="formValid"
              :disabled="!formValid"
            >
              繼續
            </v-btn>
          </div>
        </v-form>
      </v-card-text>
    </v-card>
  </v-dialog>
</template>

<script>
import { isPhone, validateEmail } from "@/utils"

export default {
  name: "GuestDialog",

  emits: ["input", "submit"],

  props: {
    value: { type: Boolean, required: true },
    event: { type: Object, required: true },
    respondents: { type: Array, required: true },
  },

  data() {
    return {
      formValid: false,
      name: "",
      email: "",
      nameRules: [],
      emailRules: [],
    }
  },

  computed: {
    isPhone() {
      return isPhone(this.$vuetify)
    },
  },

  methods: {
    submit() {
      // Set rules only on submit
      this.nameRules = [
        (name) => !!name || "請輸入名字",
        (name) => !this.respondents.includes(name) || "此名字已被使用",
      ]
      this.emailRules = [
        (email) => !!email || "請輸入 Email",
        (email) => !!validateEmail(email) || "Email 格式不正確",
      ]

      this.$nextTick(() => {
        if (!this.$refs.form.validate()) return

        this.$emit("submit", { name: this.name, email: this.email })
      })
    },
  },

  watch: {
    value() {
      if (this.value) {
        this.name = ""
        this.email = ""
        this.nameRules = []
        this.emailRules = []

        this.$refs.form?.resetValidation()
      }
    },
    name() {
      // Default rules before submitting
      this.nameRules = [
        (name) => !this.respondents.includes(name) || "此名字已被使用",
      ]
    },
    email() {
      // Default rules before submitting
      this.emailRules = []
    },
  },
}
</script>
