<template>
  <div
    class="tw-flex tw-min-h-screen tw-items-center tw-justify-center tw-bg-light-gray tw-px-4"
  >
    <div class="tw-w-full tw-max-w-[420px]">
      <!-- Logo -->
      <div class="tw-mb-8 tw-flex tw-justify-center">
        <router-link :to="{ name: 'landing' }">
          <v-img
            alt="Timeful Logo"
            class="shrink tw-cursor-pointer"
            contain
            src="@/assets/timeful_logo_with_text.png"
            transition="fade-transition"
            width="160"
          />
        </router-link>
      </div>

      <v-card class="tw-rounded-xl tw-px-2 tw-py-4">
        <!-- Main sign-in screen -->
        <template v-if="step === 'select'">
          <v-card-title class="tw-flex tw-flex-col tw-items-center tw-pb-0">
            <div class="tw-text-2xl tw-font-medium">
              {{ isSignUp ? "建立帳號" : "歡迎回來" }}
            </div>
            <div class="tw-mt-1 tw-text-sm tw-font-normal tw-text-dark-gray">
              {{ isSignUp ? "註冊以開始使用" : "登入你的帳號" }}
            </div>
          </v-card-title>
          <v-card-text class="tw-flex tw-flex-col tw-items-center tw-pt-6">
            <div class="tw-mb-4 tw-flex tw-w-full tw-flex-col tw-gap-y-2">
              <div>
                <div class="tw-mb-1 tw-text-sm tw-font-medium">電子郵件</div>
                <v-text-field
                  v-model="email"
                  class="tw-mb-2"
                  placeholder="輸入你的電子郵件..."
                  type="email"
                  solo
                  hide-details="auto"
                  :error-messages="emailError"
                  @keydown.enter="submitEmail"
                />
                <v-btn
                  block
                  color="primary"
                  :loading="sending"
                  :disabled="sending"
                  @click="submitEmail"
                >
                  {{ isSignUp ? "使用 Email 註冊" : "使用 Email 繼續" }}
                </v-btn>
              </div>
            </div>
            <div class="tw-text-center tw-text-xs">
              繼續即表示你同意我們的
              <router-link
                class="tw-text-blue"
                :to="{ name: 'privacy-policy' }"
              >
                隱私權政策
              </router-link>
            </div>
          </v-card-text>
        </template>

        <!-- Onboarding: name entry for new users -->
        <template v-else-if="step === 'onboarding'">
          <v-card-title class="tw-flex tw-items-center">
            <v-btn icon small @click="step = 'select'" class="tw-mr-1">
              <v-icon>mdi-arrow-left</v-icon>
            </v-btn>
            你叫什麼名字？
          </v-card-title>
          <v-card-text>
            <p class="tw-mb-4 tw-text-sm tw-text-dark-gray">
              我們只需要一些資訊來建立你的帳號
            </p>
            <div class="tw-mb-1 tw-text-sm tw-font-medium">名字</div>
            <v-text-field
              v-model="firstName"
              placeholder="名字"
              solo
              hide-details="auto"
              autofocus
              @keydown.enter="
                $refs.lastNameField && $refs.lastNameField.focus()
              "
              class="tw-mb-3"
            />
            <div class="tw-mb-1 tw-text-sm tw-font-medium">姓氏</div>
            <v-text-field
              ref="lastNameField"
              v-model="lastName"
              placeholder="姓氏（選填）"
              solo
              hide-details="auto"
              @keydown.enter="submitOnboarding"
              class="tw-mb-3"
            />
            <div class="tw-mb-1 tw-text-sm tw-font-medium">電子郵件</div>
            <v-text-field
              :value="email"
              placeholder="電子郵件..."
              solo
              hide-details="auto"
              disabled
              background-color="#f5f5f5"
              class="tw-mb-3"
            />
            <v-btn
              block
              color="primary"
              :loading="sending"
              :disabled="!firstName.trim() || sending"
              @click="submitOnboarding"
            >
              繼續
            </v-btn>
          </v-card-text>
        </template>

        <!-- OTP code input -->
        <template v-else-if="step === 'otp'">
          <v-card-title class="tw-flex tw-items-center">
            <v-btn
              icon
              small
              @click="step = isNewUser ? 'onboarding' : 'select'"
              class="tw-mr-1"
            >
              <v-icon>mdi-arrow-left</v-icon>
            </v-btn>
            輸入驗證碼
          </v-card-title>
          <v-card-text>
            <p class="tw-mb-4 tw-text-sm tw-text-dark-gray">
              請輸入已寄送至
              <strong>{{ email }}</strong>
              的 6 位數驗證碼
            </p>
            <div class="tw-mb-1 tw-text-sm tw-font-medium">
              驗證碼
            </div>
            <v-text-field
              v-model="otpCode"
              placeholder="輸入 6 位數驗證碼..."
              solo
              hide-details="auto"
              maxlength="6"
              :error-messages="otpError"
              @keydown.enter="verifyOtp"
              autofocus
              class="tw-mb-2"
            />
            <v-btn
              block
              color="primary"
              :loading="verifying"
              :disabled="otpCode.length !== 6 || verifying"
              @click="verifyOtp"
            >
              驗證
            </v-btn>
            <div class="tw-mt-3 tw-text-center">
              <v-btn
                text
                x-small
                :disabled="resendCooldown > 0"
                @click="resendOtp"
              >
                {{
                  resendCooldown > 0
                    ? `重新發送（${resendCooldown}s）`
                    : "重新發送驗證碼"
                }}
              </v-btn>
            </div>
          </v-card-text>
        </template>
      </v-card>

      <div
        class="tw-mt-4 tw-rounded-xl tw-bg-light-gray-stroke/50 tw-py-4 tw-text-center tw-text-sm tw-text-dark-gray"
      >
        <template v-if="isSignUp">
          已經有帳號了？
          <router-link
            class="tw-font-medium tw-text-green"
            :to="{ name: 'sign-in', query: $route.query }"
            >登入</router-link
          >
        </template>
        <template v-else>
          還沒有帳號？
          <router-link
            class="tw-font-medium tw-text-green"
            :to="{ name: 'sign-up', query: $route.query }"
            >註冊</router-link
          >
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import { authTypes } from "@/constants"
import { post } from "@/utils"
import { mapMutations } from "vuex"
import Logo from "@/components/Logo.vue"

export default {
  name: "SignIn",

  props: {
    initialIsSignUp: { type: Boolean, default: false },
  },

  metaInfo() {
    return {
      title: this.isSignUp ? "Sign Up - Timeful" : "Sign In - Timeful",
    }
  },

  components: {
    Logo,
  },

  computed: {
    upgradeRedirect() {
      return this.$route.query.redirect === "upgrade"
    },
  },

  data() {
    return {
      isSignUp: this.initialIsSignUp,
      step: "select",
      email: "",
      firstName: "",
      lastName: "",
      otpCode: "",
      emailError: "",
      otpError: "",
      sending: false,
      verifying: false,
      isNewUser: false,
      resendCooldown: 0,
      resendTimer: null,
    }
  },

  methods: {
    ...mapMutations(["setAuthUser"]),
    validateEmail() {
      const email = this.email.trim()
      if (!email) {
        this.emailError = "請輸入電子郵件地址"
        return false
      }
      if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
        this.emailError = "請輸入有效的電子郵件地址"
        return false
      }
      if (email.includes("+")) {
        this.emailError = "不允許使用含有「+」的電子郵件別名"
        return false
      }
      return true
    },
    async submitEmail() {
      if (this.sending) return
      this.emailError = ""
      if (!this.validateEmail()) return
      this.sending = true
      try {
        const res = await post("/auth/otp/check-email", { email: this.email })
        this.isNewUser = res.isNewUser
        if (this.isNewUser) {
          this.step = "onboarding"
        } else {
          await this.sendOtpEmail()
          this.step = "otp"
          this.otpCode = ""
          this.otpError = ""
        }
      } catch (err) {
        this.emailError = "發生錯誤，請再試一次"
      } finally {
        this.sending = false
      }
    },
    async submitOnboarding() {
      if (!this.firstName.trim() || this.sending) return
      this.sending = true
      try {
        await this.sendOtpEmail()
        this.step = "otp"
        this.otpCode = ""
        this.otpError = ""
      } catch (err) {
        this.otpError = "驗證碼發送失敗，請再試一次"
      } finally {
        this.sending = false
      }
    },
    async sendOtpEmail() {
      await post("/auth/otp/send", { email: this.email })
      this.startResendCooldown()
    },
    async resendOtp() {
      if (this.sending || this.resendCooldown > 0) return
      this.sending = true
      try {
        await this.sendOtpEmail()
        this.otpCode = ""
        this.otpError = ""
      } catch (err) {
        this.otpError = "驗證碼重新發送失敗，請再試一次"
      } finally {
        this.sending = false
      }
    },
    async verifyOtp() {
      if (this.otpCode.length !== 6 || this.verifying) return
      this.otpError = ""
      this.verifying = true
      try {
        const body = {
          email: this.email,
          code: this.otpCode,
          timezoneOffset: new Date().getTimezoneOffset(),
        }
        if (this.isNewUser) {
          body.firstName = this.firstName.trim()
          body.lastName = this.lastName.trim()
        }
        const user = await post("/auth/otp/verify", body)
        this.setAuthUser(user)
        this.$posthog?.identify(user._id, {
          email: user.email,
          firstName: user.firstName,
          lastName: user.lastName,
        })
        await this.handlePostAuthRedirect(user)
      } catch (err) {
        const errorCode = err?.parsed?.error
        if (errorCode === "otp-expired") {
          this.otpError = "驗證碼已過期，請重新取得"
        } else if (errorCode === "otp-too-many-attempts") {
          this.otpError = "嘗試次數過多，請重新取得驗證碼"
        } else {
          this.otpError = "驗證碼錯誤，請再試一次"
        }
      } finally {
        this.verifying = false
      }
    },
    async handlePostAuthRedirect(user) {
      if (this.upgradeRedirect) {
        try {
          const params = JSON.parse(this.$route.query.upgradeParams)
          const res = await post("/stripe/create-checkout-session", {
            priceId: params.priceId,
            userId: user._id,
            isSubscription: params.isSubscription,
            originUrl: params.originUrl,
          })
          window.location.href = res.url
          return
        } catch (e) {
          console.error(e)
        }
      }
      this.$router.replace({ name: "home" })
    },
    startResendCooldown() {
      this.resendCooldown = 30
      if (this.resendTimer) clearInterval(this.resendTimer)
      this.resendTimer = setInterval(() => {
        this.resendCooldown--
        if (this.resendCooldown <= 0) {
          clearInterval(this.resendTimer)
          this.resendTimer = null
        }
      }, 1000)
    },
  },

  beforeDestroy() {
    if (this.resendTimer) clearInterval(this.resendTimer)
  },
}
</script>
