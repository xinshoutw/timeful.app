<template>
  <v-card
    :flat="dialog"
    :class="{ 'tw-py-4': !dialog, 'tw-flex-1': dialog }"
    class="tw-relative tw-flex tw-max-w-[28rem] tw-flex-col tw-overflow-hidden tw-rounded-lg tw-transition-all"
  >
    <v-card-title class="tw-mb-2 tw-flex tw-gap-2 tw-px-4 sm:tw-px-8">
      <div>
        <div class="tw-mb-1">
          {{ edit ? "編輯活動" : "新增活動" }}
        </div>
        <div
          v-if="dialog && showHelp"
          class="tw-text-xs tw-font-normal tw-italic tw-text-dark-gray"
        >
          適合一次性或定期會議
        </div>
      </div>
      <v-spacer />
      <template v-if="dialog">
        <v-btn v-if="showHelp" icon @click="helpDialog = true">
          <v-icon>mdi-information-outline</v-icon>
        </v-btn>
        <v-btn v-else @click="$emit('input', false)" icon>
          <v-icon>mdi-close</v-icon>
        </v-btn>
        <HelpDialog v-model="helpDialog">
          <template v-slot:header>活動</template>
          <div class="tw-mb-4">
            使用活動來收集大家有空的時段，並在特定日期間進行比較
          </div>
        </HelpDialog>
      </template>
    </v-card-title>
    <v-card-text
      ref="cardText"
      class="tw-relative tw-flex-1 tw-overflow-auto tw-px-4 tw-py-1 sm:tw-px-8"
    >
      <AlertText v-if="edit && event?.ownerId == 0" class="tw-mb-4">
        因為建立時未登入，任何人都能編輯此活動
      </AlertText>
      <v-form
        ref="form"
        v-model="formValid"
        lazy-validation
        class="tw-flex tw-flex-col tw-gap-y-6"
        :disabled="loading"
      >
        <v-text-field
          ref="name-field"
          v-model="name"
          placeholder="為你的活動命名..."
          hide-details="auto"
          solo
          @keyup.enter="blurNameField"
          :rules="nameRules"
          autofocus
          required
        />

        <SlideToggle
          v-if="daysOnlyEnabled && !edit"
          class="tw-w-full"
          v-model="daysOnly"
          :options="daysOnlyOptions"
        />

        <div>
          <v-expand-transition>
            <div v-if="!daysOnly">
              <div class="tw-mb-2 tw-text-lg tw-text-black">
                什麼時間可以？
              </div>
              <v-expand-transition>
                <div v-if="!specificTimesEnabled">
                  <div
                    class="tw-mb-2 tw-flex tw-items-baseline tw-justify-center tw-space-x-2"
                  >
                    <v-select
                      :value="startTime"
                      @input="(t) => (startTime = t.time)"
                      menu-props="auto"
                      :items="times"
                      return-object
                      hide-details
                      solo
                    ></v-select>
                    <div>至</div>
                    <v-select
                      :value="endTime"
                      @input="(t) => (endTime = t.time)"
                      menu-props="auto"
                      :items="times"
                      return-object
                      hide-details
                      solo
                    ></v-select>
                  </div>
                </div>
              </v-expand-transition>
              <div class="tw-mb-2">
                <v-checkbox
                  v-model="specificTimesEnabled"
                  messages="在下一步指定時段"
                >
                  <template v-slot:label>
                    <span
                      class="tw-text-sm"
                      :class="
                        specificTimesEnabled
                          ? 'tw-text-black'
                          : 'tw-text-very-dark-gray'
                      "
                    >
                      為每一天單獨設定時段
                    </span>
                  </template>
                  <template v-slot:message="{ key, message }">
                    <v-expand-transition>
                      <div
                        v-if="specificTimesEnabled"
                        class="tw-pointer-events-auto -tw-mt-1 tw-ml-[32px] tw-text-xs tw-text-dark-gray"
                      >
                        {{ message }}
                      </div>
                    </v-expand-transition>
                  </template>
                </v-checkbox>
              </div>
            </div>
          </v-expand-transition>

          <div class="tw-mb-2 tw-text-lg tw-text-black">
            {{
              selectedDateOption === dateOptions.SPECIFIC
                ? "哪些時段可以？"
                : "哪幾天可以？"
            }}
          </div>
          <v-select
            v-if="!edit && !daysOnly"
            v-model="selectedDateOption"
            :items="Object.values(dateOptions)"
            solo
            hide-details
            class="tw-mb-4"
          />

          <v-expand-transition>
            <div v-if="selectedDateOption === dateOptions.SPECIFIC || daysOnly">
              <div class="tw-mb-2 tw-text-xs tw-text-dark-gray">
                拖曳來選取多個日期
              </div>
              <v-input
                v-model="selectedDays"
                hide-details="auto"
                :rules="selectedDaysRules"
                key="date-picker"
              >
                <DatePicker
                  v-model="selectedDays"
                  :minCalendarDate="minCalendarDate"
                  :startCalendarOnMonday="startOnMonday"
                />
              </v-input>
            </div>
            <div v-else-if="selectedDateOption === dateOptions.DOW">
              <v-input
                v-model="selectedDaysOfWeek"
                hide-details="auto"
                :rules="selectedDaysRules"
                key="days-of-week"
                class="tw-w-fit"
              >
                <v-btn-toggle
                  v-model="selectedDaysOfWeek"
                  multiple
                  solo
                  color="primary"
                >
                  <v-btn depressed v-show="!startOnMonday"> 日 </v-btn>
                  <v-btn depressed> 一 </v-btn>
                  <v-btn depressed> 二 </v-btn>
                  <v-btn depressed> 三 </v-btn>
                  <v-btn depressed> 四 </v-btn>
                  <v-btn depressed> 五 </v-btn>
                  <v-btn depressed> 六 </v-btn>
                  <v-btn depressed v-show="startOnMonday"> 日 </v-btn>
                </v-btn-toggle>
              </v-input>
              <v-checkbox class="tw-mt-2" v-model="startOnMonday" hide-details>
                <template v-slot:label>
                  <span class="tw-text-sm tw-text-very-dark-gray">
                    第一天為星期一
                  </span>
                </template>
              </v-checkbox>
            </div>
          </v-expand-transition>
        </div>

        <v-checkbox
          v-if="!guestEvent && authUser"
          v-model="notificationsEnabled"
          hide-details
          class="tw-mt-2"
        >
          <template v-slot:label>
            <span class="tw-text-sm tw-text-very-dark-gray"
              >每次有人加入時 Email 通知我</span
            >
          </template>
        </v-checkbox>
        <v-checkbox
          v-else-if="!guestEvent"
          disabled
          messages="test"
          off-icon="mdi-checkbox-blank-off-outline"
          class="tw-mt-2"
        >
          <template v-slot:label>
            <span class="tw-text-sm"
              >每次有人加入時 Email 通知我</span
            >
          </template>
          <template v-slot:message="{ key, message }">
            <div
              class="tw-pointer-events-auto -tw-mt-1 tw-ml-[32px] tw-text-xs tw-text-dark-gray"
            >
              <span class="tw-font-medium tw-text-very-dark-gray"
                ><a @click="$emit('signIn')">登入</a>
                來使用此功能
              </span>
            </div>
          </template>
        </v-checkbox>

        <div class="tw-flex tw-flex-col tw-gap-2">
          <ExpandableSection
            v-if="authUser && !guestEvent"
            label="Email 提醒"
            v-model="showEmailReminders"
            :auto-scroll="dialog"
          >
            <div class="tw-flex tw-flex-col tw-gap-5 tw-pt-2">
              <EmailInput
                v-show="authUser"
                ref="emailInput"
                labelColor="tw-text-very-dark-gray"
                :addedEmails="addedEmails"
                @update:emails="(newEmails) => (emails = newEmails)"
              >
                <template v-slot:header>
                  <div class="tw-flex tw-gap-1">
                    <div class="tw-text-very-dark-gray">
                      提醒大家填寫活動
                    </div>

                    <v-tooltip
                      top
                      content-class="tw-bg-very-dark-gray tw-shadow-lg tw-opacity-100 tw-py-4"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-icon small v-bind="attrs" v-on="on"
                          >mdi-information-outline
                        </v-icon>
                      </template>
                      <div>
                        提醒 Email 會在活動建立當天、隔天以及三天後寄出<br />當所有人都填寫完畢時，你也會收到 Email 通知
                      </div>
                    </v-tooltip>
                  </div>
                </template>
              </EmailInput>
            </div>
          </ExpandableSection>

          <ExpandableSection
            v-model="showAdvancedOptions"
            label="進階選項"
            :auto-scroll="dialog"
          >
            <div class="tw-flex tw-flex-col tw-gap-5 tw-pt-2">
              <div v-if="!edit" class="tw-flex tw-items-center tw-gap-x-2">
                <div class="tw-text-sm tw-text-black">時間間隔：</div>
                <v-select
                  v-model="timeIncrement"
                  dense
                  class="-tw-mt-[2px] tw-w-24 tw-grow-0 tw-text-sm"
                  menu-props="auto"
                  hide-details
                  :items="timeIncrementItems"
                ></v-select>
              </div>
              <v-checkbox
                v-if="authUser && !guestEvent"
                v-model="collectEmails"
                hide-details
              >
                <template v-slot:label>
                  <span class="tw-text-sm tw-text-black">
                    收集回覆者的 Email
                  </span>
                </template>
                <template v-slot:message="{ key, message }">
                  <div
                    class="-tw-mt-1 tw-ml-[32px] tw-text-xs tw-text-dark-gray"
                  >
                    {{ message }}
                  </div>
                </template>
              </v-checkbox>
              <v-checkbox
                v-else-if="!guestEvent"
                disabled
                messages="test"
                off-icon="mdi-checkbox-blank-off-outline"
              >
                <template v-slot:label>
                  <span class="tw-text-sm"
                    >收集回覆者的 Email</span
                  >
                </template>
                <template v-slot:message="{ key, message }">
                  <div
                    class="tw-pointer-events-auto -tw-mt-1 tw-ml-[32px] tw-text-xs tw-text-dark-gray"
                  >
                    <span class="tw-font-medium tw-text-very-dark-gray"
                      ><a @click="$emit('signIn')">登入</a>
                      來使用此功能
                    </span>
                  </div>
                </template>
              </v-checkbox>
              <v-checkbox
                v-if="authUser && !guestEvent"
                v-model="blindAvailabilityEnabled"
                messages="僅活動建立者可見"
              >
                <template v-slot:label>
                  <span class="tw-text-sm tw-text-black">
                    對回覆者隱藏其他人的回覆
                  </span>
                </template>
                <template v-slot:message="{ key, message }">
                  <div
                    class="-tw-mt-1 tw-ml-[32px] tw-text-xs tw-text-dark-gray"
                  >
                    {{ message }}
                  </div>
                </template>
              </v-checkbox>
              <v-checkbox
                v-else-if="!guestEvent"
                disabled
                messages="僅活動建立者可見 "
                off-icon="mdi-checkbox-blank-off-outline"
              >
                <template v-slot:label>
                  <span class="tw-text-sm"
                    >對回覆者隱藏其他人的回覆</span
                  >
                </template>
                <template v-slot:message="{ key, message }">
                  <div
                    class="tw-pointer-events-auto -tw-mt-1 tw-ml-[32px] tw-text-xs tw-text-dark-gray"
                  >
                    {{ message }}
                    <span class="tw-font-medium tw-text-very-dark-gray"
                      ><a @click="$emit('signIn')">登入</a>
                      來使用此功能
                    </span>
                  </div>
                </template>
              </v-checkbox>
              <v-checkbox
                v-if="authUser && !guestEvent"
                v-model="sendEmailAfterXResponsesEnabled"
                hide-details
              >
                <template v-slot:label>
                  <div
                    :class="!sendEmailAfterXResponsesEnabled && 'tw-opacity-50'"
                    class="tw-flex tw-items-center tw-gap-x-2 tw-text-sm tw-text-very-dark-gray"
                  >
                    <div>收到</div>
                    <v-text-field
                      v-model="sendEmailAfterXResponses"
                      @click="
                        (e) => {
                          e.preventDefault()
                          e.stopPropagation()
                        }
                      "
                      :disabled="!sendEmailAfterXResponsesEnabled"
                      dense
                      class="email-me-after-text-field -tw-mt-[2px] tw-w-10"
                      menu-props="auto"
                      hide-details
                      type="number"
                      min="1"
                    ></v-text-field>
                    <div>則回覆後 Email 通知我</div>
                  </div>
                </template>
              </v-checkbox>
              <TimezoneSelector
                v-model="timezone"
                label="時區"
                @input="trackTimezoneChange"
              />
            </div>
          </ExpandableSection>
        </div>
      </v-form>
    </v-card-text>
    <v-card-actions class="tw-relative tw-px-4 sm:tw-px-8">
      <div class="tw-relative tw-w-full">
        <v-btn
          :disabled="!formValid"
          block
          :loading="loading"
          color="primary"
          class="tw-mt-4 tw-bg-green"
          @click="submit"
        >
          {{
            specificTimesEnabled ? "下一步" : edit ? "儲存變更" : "建立活動"
          }}
        </v-btn>
        <div
          :class="formValid ? 'tw-invisible' : 'tw-visible'"
          class="tw-mt-1 tw-text-xs tw-text-red"
        >
          請先修正表單錯誤再繼續
        </div>
      </div>
    </v-card-actions>

    <OverflowGradient
      v-if="hasMounted"
      :scrollContainer="$refs.cardText"
      class="tw-bottom-[90px]"
    />
  </v-card>
</template>

<style>
.email-me-after-text-field input {
  padding: 0px !important;
}
</style>

<script>
import { eventTypes, dayIndexToDayString } from "@/constants"
import {
  post,
  put,
  timeNumToTimeString,
  dateToTimeNum,
  getISODateString,
  isPhone,
  getDateWithTimezone,
  getTimeOptions,
  addEventToCreatedList,
  prefersStartOnMonday,
} from "@/utils"
import { mapActions, mapState } from "vuex"
import TimezoneSelector from "./schedule_overlap/TimezoneSelector.vue"
import HelpDialog from "./HelpDialog.vue"
import EmailInput from "./event/EmailInput.vue"
import DatePicker from "@/components/DatePicker.vue"
import SlideToggle from "./SlideToggle.vue"
import AlertText from "@/components/AlertText.vue"
import OverflowGradient from "@/components/OverflowGradient.vue"
import { guestUserId } from "@/constants"
import dayjs from "dayjs"
import utcPlugin from "dayjs/plugin/utc"
import timezonePlugin from "dayjs/plugin/timezone"
import ExpandableSection from "./ExpandableSection.vue"
dayjs.extend(utcPlugin)
dayjs.extend(timezonePlugin)

export default {
  name: "NewEvent",

  emits: ["input"],

  props: {
    event: { type: Object },
    edit: { type: Boolean, default: false },
    dialog: { type: Boolean, default: true },
    contactsPayload: { type: Object, default: () => ({}) },
    showHelp: { type: Boolean, default: false },
    folderId: { type: String, default: null },
    isDialogOpen: { type: Boolean, default: false },
  },

  components: {
    TimezoneSelector,
    HelpDialog,
    EmailInput,
    DatePicker,
    SlideToggle,
    ExpandableSection,
    AlertText,
    OverflowGradient,
  },

  data: () => ({
    formValid: true,
    name: "",
    startTime: 9,
    endTime: 17,
    specificTimesEnabled: false,
    loading: false,
    selectedDays: [],
    selectedDaysOfWeek: [],
    startOnMonday: prefersStartOnMonday(),
    notificationsEnabled: true,

    daysOnly: false,
    daysOnlyOptions: Object.freeze([
      { text: "日期與時間", value: false },
      { text: "僅日期", value: true },
    ]),

    // Date options
    dateOptions: Object.freeze({
      SPECIFIC: "指定日期",
      DOW: "星期幾",
    }),
    selectedDateOption: "指定日期",

    // Email reminders
    showEmailReminders: false,
    emails: [], // For email reminders

    // Advanced options
    showAdvancedOptions: false,
    timeIncrement: 15,
    collectEmails: false,
    blindAvailabilityEnabled: false,
    timezone: {},
    sendEmailAfterXResponsesEnabled: false,
    sendEmailAfterXResponses: 3,

    helpDialog: false,

    // Unsaved changes
    initialEventData: {},

    hasMounted: false,
  }),

  mounted() {
    if (Object.keys(this.contactsPayload).length > 0) {
      this.toggleEmailReminders(true)

      /** Get previously filled out data after enabling contacts  */
      this.name = this.contactsPayload.name
      this.startTime = this.contactsPayload.startTime
      this.endTime = this.contactsPayload.endTime
      this.daysOnly = this.contactsPayload.daysOnly
      this.selectedDateOption = this.contactsPayload.selectedDateOption
      this.selectedDaysOfWeek = this.contactsPayload.selectedDaysOfWeek
      this.selectedDays = this.contactsPayload.selectedDays
      this.notificationsEnabled = this.contactsPayload.notificationsEnabled
      this.timezone = this.contactsPayload.timezone
      this.specificTimesEnabled = this.contactsPayload.specificTimesEnabled

      this.$refs.form.resetValidation()
    }

    this.$nextTick(() => {
      this.hasMounted = true
    })
  },

  computed: {
    ...mapState(["authUser", "daysOnlyEnabled"]),
    nameRules() {
      return [(v) => !!v || "請輸入活動名稱"]
    },
    selectedDaysRules() {
      return [
        (selectedDays) =>
          selectedDays.length > 0 || "請至少選擇一天",
      ]
    },
    addedEmails() {
      if (Object.keys(this.contactsPayload).length > 0)
        return this.contactsPayload.emails
      return this.event && this.event.remindees
        ? this.event.remindees.map((r) => r.email)
        : []
    },
    times() {
      return getTimeOptions()
    },
    minCalendarDate() {
      if (this.edit) {
        return ""
      }

      let today = new Date()
      let dd = String(today.getDate()).padStart(2, "0")
      let mm = String(today.getMonth() + 1).padStart(2, "0")
      let yyyy = today.getFullYear()

      return yyyy + "-" + mm + "-" + dd
    },
    isPhone() {
      return isPhone(this.$vuetify)
    },
    guestEvent() {
      return this.event && this.event.ownerId == guestUserId
    },
    timeIncrementItems() {
      return [
        { text: "15 分鐘", value: 15 },
        { text: "30 分鐘", value: 30 },
        { text: "60 分鐘", value: 60 },
      ]
    },
  },

  methods: {
    ...mapActions(["showError", "setEventFolder"]),
    blurNameField() {
      this.$refs["name-field"].blur()
    },
    reset() {
      this.name = ""
      this.startTime = 9
      this.endTime = 17
      this.specificTimesEnabled = false
      this.selectedDays = []
      this.selectedDaysOfWeek = []
      this.notificationsEnabled = true
      this.daysOnly = false
      this.selectedDateOption = "指定日期"
      this.emails = []
      this.showAdvancedOptions = false
      this.blindAvailabilityEnabled = false
      this.sendEmailAfterXResponsesEnabled = false
      this.sendEmailAfterXResponses = 3
      this.collectEmails = false
      this.startOnMonday = prefersStartOnMonday()

      this.$refs.form.resetValidation()
    },
    submit() {
      if (!this.$refs.form.validate()) return

      this.selectedDays.sort()

      // Get duration of event
      let duration = this.endTime - this.startTime
      if (duration <= 0) duration += 24

      // Get date objects for each selected day
      let dates = []
      let type = ""
      if (this.daysOnly) {
        duration = 0
        type = eventTypes.SPECIFIC_DATES

        for (const day of this.selectedDays) {
          const date = new Date(`${day} 00:00:00Z`)
          dates.push(date)
        }

        this.specificTimesEnabled = false
      } else {
        const startTimeString = timeNumToTimeString(this.startTime)
        if (this.selectedDateOption === this.dateOptions.SPECIFIC) {
          type = eventTypes.SPECIFIC_DATES

          for (const day of this.selectedDays) {
            const date = dayjs.tz(
              `${day} ${startTimeString}`,
              this.timezone.value
            )
            dates.push(date.toDate())
          }
        } else if (this.selectedDateOption === this.dateOptions.DOW) {
          type = eventTypes.DOW

          this.selectedDaysOfWeek.sort((a, b) => a - b)
          this.selectedDaysOfWeek = this.selectedDaysOfWeek.filter(
            (dayIndex) => {
              return this.startOnMonday ? dayIndex !== 0 : dayIndex !== 7
            }
          )
          for (const dayIndex of this.selectedDaysOfWeek) {
            const day = dayIndexToDayString[dayIndex]
            const date = dayjs.tz(
              `${day} ${startTimeString}`,
              this.timezone.value
            )

            // The reference dates (dayIndexToDayString) are from June 2018, which may have
            // a different DST offset than the current date. Adjust so the stored UTC time
            // corresponds to the user's current timezone offset.
            const refOffset = date.utcOffset()
            const currentOffset = dayjs().tz(this.timezone.value).utcOffset()
            dates.push(
              date.subtract(currentOffset - refOffset, "minutes").toDate()
            )
          }
        }
      }

      this.loading = true

      const payload = {
        name: this.name,
        duration: duration,
        dates: dates,
        hasSpecificTimes: this.specificTimesEnabled,
        notificationsEnabled: !this.authUser
          ? false
          : this.notificationsEnabled,
        blindAvailabilityEnabled: this.blindAvailabilityEnabled,
        daysOnly: this.daysOnly,
        remindees: this.emails,
        type: type,
        sendEmailAfterXResponses: this.sendEmailAfterXResponsesEnabled
          ? parseInt(this.sendEmailAfterXResponses)
          : -1,
        collectEmails: this.collectEmails,
        startOnMonday: this.startOnMonday,
        timeIncrement: this.timeIncrement,
        creatorPosthogId: this.$posthog?.get_distinct_id(),
      }

      const posthogPayload = {
        eventName: this.name,
        eventDuration: duration,
        eventDates: JSON.stringify(dates),
        eventHasSpecificTimes: this.specificTimesEnabled,
        eventNotificationsEnabled: !this.authUser
          ? false
          : this.notificationsEnabled,
        eventBlindAvailabilityEnabled: this.blindAvailabilityEnabled,
        eventDaysOnly: this.daysOnly,
        eventRemindees: this.emails,
        eventType: type,
        eventSendEmailAfterXResponses: this.sendEmailAfterXResponsesEnabled
          ? parseInt(this.sendEmailAfterXResponses)
          : -1,
        eventCollectEmails: this.collectEmails,
        eventStartOnMonday: this.startOnMonday,
        eventTimeIncrement: this.timeIncrement,
      }

      if (!this.edit) {
        // Create new event on backend
        post("/events", payload)
          .then(async ({ eventId, shortId }) => {
            if (this.authUser) {
              await this.setEventFolder({ eventId, folderId: this.folderId })
            }
            this.$router.push({
              name: "event",
              params: {
                eventId: shortId ?? eventId,
                initialTimezone: this.timezone,
              },
            })

            this.$emit("input", false)
            this.reset()

            posthogPayload.eventId = eventId
            this.$posthog?.capture("Event created", posthogPayload)

            if (!this.authUser) {
              // Add eventId to localStorage, so the user can claim it later
              addEventToCreatedList(eventId)
            }
          })
          .catch((err) => {
            this.showError(
              "建立活動時發生問題！請稍後再試"
            )
            console.error(err)
          })
          .finally(() => {
            this.loading = false
          })
      } else {
        // Edit event on backend
        if (this.event) {
          put(`/events/${this.event._id}`, payload)
            .then(() => {
              posthogPayload.eventId = this.event._id
              this.$posthog?.capture("Event edited", posthogPayload)

              // this.$emit("input", false)
              // this.reset()
              localStorage.setItem(`from-edit-event-${this.event._id}`, "true")
              window.location.reload()
            })
            .catch((err) => {
              this.showError(
                "編輯活動時發生問題！請稍後再試"
              )
              console.log(err)
            })
            .finally(() => {
              this.loading = false
            })
        }
      }
    },

    toggleEmailReminders(delayed = false) {
      if (delayed) {
        setTimeout(
          () => (this.showEmailReminders = !this.showEmailReminders),
          300
        )
      } else {
        this.showEmailReminders = !this.showEmailReminders
      }
    },

    /** Populates the form fields based on this.event */
    updateFieldsFromEvent() {
      if (this.event) {
        this.name = this.event.name

        // Set start time, accounting for the timezone
        this.startTime = Math.floor(
          dateToTimeNum(getDateWithTimezone(this.event.dates[0]), true)
        )
        this.startTime %= 24

        this.endTime = (this.startTime + this.event.duration) % 24
        this.notificationsEnabled = this.event.notificationsEnabled
        this.blindAvailabilityEnabled = this.event.blindAvailabilityEnabled
        this.daysOnly = this.event.daysOnly
        this.specificTimesEnabled = this.event.hasSpecificTimes
        this.startOnMonday = this.event.startOnMonday
        this.collectEmails = this.event.collectEmails
        this.timeIncrement = this.event.timeIncrement ?? 15

        if (
          this.event.sendEmailAfterXResponses !== null &&
          this.event.sendEmailAfterXResponses > 0
        ) {
          this.sendEmailAfterXResponsesEnabled = true
          this.sendEmailAfterXResponses = this.event.sendEmailAfterXResponses
        }

        if (this.event.daysOnly) {
          this.selectedDateOption = this.dateOptions.SPECIFIC
          const selectedDays = []
          for (let date of this.event.dates) {
            selectedDays.push(getISODateString(date, true))
          }
          this.selectedDays = selectedDays
        } else {
          if (this.event.type === eventTypes.SPECIFIC_DATES) {
            this.selectedDateOption = this.dateOptions.SPECIFIC
            const selectedDays = []
            for (let date of this.event.dates) {
              date = getDateWithTimezone(date)

              selectedDays.push(getISODateString(date, true))
            }
            this.selectedDays = selectedDays
          } else if (this.event.type === eventTypes.DOW) {
            this.selectedDateOption = this.dateOptions.DOW
            const selectedDaysOfWeek = []
            for (let date of this.event.dates) {
              date = getDateWithTimezone(date)

              if (this.event.startOnMonday && date.getUTCDay() === 0) {
                selectedDaysOfWeek.push(7)
              } else {
                selectedDaysOfWeek.push(date.getUTCDay())
              }
            }
            this.selectedDaysOfWeek = selectedDaysOfWeek
            if (this.event.startOnMonday) {
              this.startOnMonday = true
            }
          }
        }
      }
    },
    resetToEventData() {
      this.updateFieldsFromEvent()
      this.$refs.emailInput.reset()
    },
    setInitialEventData() {
      this.initialEventData = {
        name: this.name,
        startTime: this.startTime,
        endTime: this.endTime,
        specificTimesEnabled: this.specificTimesEnabled,
        daysOnly: this.daysOnly,
        selectedDays: this.selectedDays,
        selectedDaysOfWeek: this.selectedDaysOfWeek,
        selectedDateOption: this.selectedDateOption,
        notificationsEnabled: this.notificationsEnabled,
        emails: [...this.emails],
        blindAvailabilityEnabled: this.blindAvailabilityEnabled,
        sendEmailAfterXResponsesEnabled: this.sendEmailAfterXResponsesEnabled,
        sendEmailAfterXResponses: this.sendEmailAfterXResponses,
        timeIncrement: this.timeIncrement,
      }
    },
    hasEventBeenEdited() {
      return (
        this.name !== this.initialEventData.name ||
        this.startTime !== this.initialEventData.startTime ||
        this.endTime !== this.initialEventData.endTime ||
        this.specificTimesEnabled !==
          this.initialEventData.specificTimesEnabled ||
        this.selectedDateOption !== this.initialEventData.selectedDateOption ||
        JSON.stringify(this.selectedDays) !==
          JSON.stringify(this.initialEventData.selectedDays) ||
        JSON.stringify(this.selectedDaysOfWeek) !==
          JSON.stringify(this.initialEventData.selectedDaysOfWeek) ||
        this.daysOnly !== this.initialEventData.daysOnly ||
        this.notificationsEnabled !==
          this.initialEventData.notificationsEnabled ||
        JSON.stringify(this.emails) !==
          JSON.stringify(this.initialEventData.emails) ||
        this.blindAvailabilityEnabled !==
          this.initialEventData.blindAvailabilityEnabled ||
        this.sendEmailAfterXResponsesEnabled !==
          this.initialEventData.sendEmailAfterXResponsesEnabled ||
        this.sendEmailAfterXResponses !==
          this.initialEventData.sendEmailAfterXResponses
      )
    },
    trackTimezoneChange(newTimezone) {
      this.$posthog.capture("timezone_selected_in_new_event_dialog", {
        timezone: newTimezone?.value,
      })
    },
  },

  watch: {
    event: {
      immediate: true,
      handler() {
        this.updateFieldsFromEvent()
        this.setInitialEventData()
      },
    },
    selectedDateOption() {
      // Reset the other date / day selection when date option is changed
      if (this.selectedDateOption === this.dateOptions.SPECIFIC) {
        this.selectedDaysOfWeek = []
      } else if (this.selectedDateOption === this.dateOptions.DOW) {
        this.selectedDays = []
      }
    },
    startOnMonday() {
      localStorage.setItem("startCalendarOnMonday", this.startOnMonday)
    },
    isDialogOpen(newVal) {
      if (newVal) {
        this.reset()
      }
    },
  },
}
</script>
