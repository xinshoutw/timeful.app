<template>
  <div class="tw-mx-auto tw-mb-12 tw-mt-5 tw-max-w-6xl">
    <div class="tw-flex tw-flex-col tw-gap-16 tw-p-4">
      <!-- Name change section -->
      <div class="tw-flex tw-flex-col tw-gap-5">
        <div
          class="tw-text-xl tw-font-medium tw-text-dark-green sm:tw-text-2xl"
        >
          Profile
        </div>
        <div>
          <div class="tw-mb-1 tw-font-medium">Name</div>
          <div class="tw-flex tw-max-w-lg tw-items-center tw-gap-2">
            <v-text-field
              v-model="firstName"
              hide-details
              outlined
              placeholder="First name"
              :dense="isPhone"
            />
            <v-text-field
              v-model="lastName"
              hide-details
              outlined
              placeholder="Last name"
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
                  >Cancel</v-btn
                >
                <v-btn @click="saveName" color="primary">Save changes</v-btn>
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
          Billing
        </div>
        <div class="tw-flex tw-flex-col tw-gap-5 sm:tw-flex-row sm:tw-gap-28">
          <div class="tw-text-black">
            <v-btn @click="openBillingPortal">Manage billing</v-btn>
          </div>
        </div>
      </div>

      <!-- Question Section -->
      <div class="tw-flex tw-flex-col tw-gap-5">
        <div
          class="tw-text-xl tw-font-medium tw-text-dark-green sm:tw-text-2xl"
        >
          Have a question?
        </div>
        <div class="tw-flex tw-flex-col tw-gap-5 sm:tw-flex-row sm:tw-gap-28">
          <div class="tw-text-black">
            Email us at
            <a
              href="mailto:contact@timeful.app"
              class="tw-text-black tw-underline"
              >contact@timeful.app</a
            >
            with any questions!
          </div>
        </div>
      </div>

      <!-- Delete Account Section -->
      <div class="tw-mt-28 tw-flex tw-flex-row tw-justify-center">
        <div class="tw-w-64">
          <v-dialog v-model="deleteDialog" width="400" persistent>
            <template v-slot:activator="{ on, attrs }">
              <v-btn outlined class="tw-text-red" block v-bind="attrs" v-on="on"
                >Delete account</v-btn
              >
            </template>
            <v-card>
              <v-card-title>Are you sure?</v-card-title>
              <v-card-text class="tw-text-sm tw-text-dark-gray"
                >Are you sure you want to delete your account? All your account
                data will be lost.</v-card-text
              >
              <div class="tw-mx-6">
                <div class="tw-text-sm tw-text-dark-gray">
                  Type your email in the box below to confirm:
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
                <v-btn text @click="deleteDialog = false">Cancel</v-btn>
                <v-btn
                  text
                  color="error"
                  @click="deleteAccount()"
                  :disabled="authUser.email != deleteValidateEmail"
                  >Delete</v-btn
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
            "There was a problem opening the billing portal! Please try again later."
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
            "There was a problem deleting your account! Please try again later."
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
            "There was a problem updating your name! Please try again later."
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
