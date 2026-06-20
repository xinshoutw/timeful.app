<template>
  <div class="tw-mx-auto tw-mb-12 tw-mt-5 tw-max-w-6xl">
    <div class="tw-flex tw-flex-col tw-gap-16 tw-p-4">
      <!-- Name change section -->
      <div class="tw-flex tw-flex-col tw-gap-5">
        <div
          class="tw-text-xl tw-font-medium tw-text-dark-green sm:tw-text-2xl"
        >
          個人資料
        </div>
        <div>
          <div class="tw-mb-1 tw-font-medium">姓名</div>
          <div class="tw-flex tw-max-w-lg tw-items-center tw-gap-2">
            <v-text-field
              v-model="firstName"
              hide-details
              outlined
              placeholder="名字"
              :dense="isPhone"
            />
            <v-text-field
              v-model="lastName"
              hide-details
              outlined
              placeholder="姓氏"
              :dense="isPhone"
            />
          </div>
          <v-expand-transition>
            <div v-if="profileUnsavedChanges">
              <div class="tw-mt-4">
                <v-btn
                  @click="resetProfileChanges"
                  color="primary"
                  outlined
                  class="tw-mr-2"
                  >取消</v-btn
                >
                <v-btn @click="saveName" color="primary">儲存變更</v-btn>
              </div>
            </div>
          </v-expand-transition>
        </div>
      </div>

      <!-- Billing Section -->
      <div
        v-if="authUser.stripeCustomerId"
        class="tw-flex tw-flex-col tw-gap-5"
      >
        <div
          class="tw-text-xl tw-font-medium tw-text-dark-green sm:tw-text-2xl"
        >
          帳單
        </div>
        <div class="tw-flex tw-flex-col tw-gap-5 sm:tw-flex-row sm:tw-gap-28">
          <div class="tw-text-black">
            <v-btn @click="openBillingPortal">管理帳單</v-btn>
          </div>
        </div>
      </div>

      <!-- Question Section -->
      <div class="tw-flex tw-flex-col tw-gap-5">
        <div
          class="tw-text-xl tw-font-medium tw-text-dark-green sm:tw-text-2xl"
        >
          有問題嗎？
        </div>
        <div class="tw-flex tw-flex-col tw-gap-5 sm:tw-flex-row sm:tw-gap-28">
          <div class="tw-text-black">
            有任何問題歡迎寄信至
            <a
              href="mailto:contact@timeful.app"
              class="tw-text-black tw-underline"
              >contact@timeful.app</a
            >
          </div>
        </div>
      </div>

      <!-- Delete Account Section -->
      <div class="tw-mt-28 tw-flex tw-flex-row tw-justify-center">
        <div class="tw-w-64">
          <v-dialog v-model="deleteDialog" width="400" persistent>
            <template v-slot:activator="{ on, attrs }">
              <v-btn outlined class="tw-text-red" block v-bind="attrs" v-on="on"
                >刪除帳號</v-btn
              >
            </template>
            <v-card>
              <v-card-title>確定要刪除嗎？</v-card-title>
              <v-card-text class="tw-text-sm tw-text-dark-gray"
                >確定要刪除你的帳號嗎？所有帳號資料將會永久遺失</v-card-text
              >
              <div class="tw-mx-6">
                <div class="tw-text-sm tw-text-dark-gray">
                  請在下方輸入你的 Email 確認：
                </div>
                <v-text-field
                  v-model="deleteValidateEmail"
                  autofocus
                  class="tw-flex-initial tw-text-white"
                  :placeholder="authUser.email"
                />
              </div>
              <v-card-actions>
                <v-spacer />
                <v-btn text @click="deleteDialog = false">取消</v-btn>
                <v-btn
                  text
                  color="error"
                  @click="deleteAccount()"
                  :disabled="authUser.email != deleteValidateEmail"
                  >刪除</v-btn
                >
              </v-card-actions>
            </v-card>
          </v-dialog>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState, mapActions } from "vuex"
import { _delete, patch, isPhone, get } from "@/utils"
export default {
  name: "Settings",

  metaInfo: {
    title: "Settings - Timeful",
  },

  data: () => ({
    dialog: false,
    deleteDialog: false,
    deleteValidateEmail: "",

    // Profile settings
    firstName: "",
    lastName: "",
  }),

  computed: {
    ...mapState(["authUser"]),
    nameUnsavedChanges() {
      return (
        this.firstName !== this.authUser.firstName ||
        this.lastName !== this.authUser.lastName
      )
    },
    profileUnsavedChanges() {
      return this.nameUnsavedChanges
    },
    isPhone() {
      return isPhone(this.$vuetify)
    },
  },

  methods: {
    ...mapActions(["showError"]),
    openBillingPortal() {
      get(
        `/stripe/billing-portal?customerId=${encodeURIComponent(
          this.authUser.stripeCustomerId
        )}&returnUrl=${encodeURIComponent(window.location.href)}`
      )
        .then((res) => {
          window.location.href = res.url
        })
        .catch((err) => {
          this.showError(
            "開啟帳單管理頁面時發生問題，請稍後再試"
          )
        })
    },
    deleteAccount() {
      _delete(`/user`)
        .then(() => {
          window.location.reload()
        })
        .catch((err) => {
          this.showError(
            "刪除帳號時發生問題，請稍後再試"
          )
        })
    },
    resetProfileChanges() {
      this.firstName = this.authUser.firstName
      this.lastName = this.authUser.lastName
    },
    saveName() {
      patch(`/user/name`, {
        firstName: this.firstName,
        lastName: this.lastName,
      })
        .then(() => {
          window.location.reload()
        })
        .catch((err) => {
          this.showError(
            "更新姓名時發生問題，請稍後再試"
          )
        })
    },
  },

  created() {
    this.firstName = this.authUser.firstName
    this.lastName = this.authUser.lastName
  },
}
</script>
