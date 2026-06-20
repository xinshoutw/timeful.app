<template>
  <v-card
    :flat="dialog"
    :class="{ 'tw-py-4': !dialog, 'tw-flex-1': dialog }"
    class="tw-relative tw-flex tw-max-w-[28rem] tw-flex-col tw-overflow-hidden tw-rounded-lg tw-transition-all"
  >
    <v-card-title class="tw-mb-2 tw-flex tw-gap-2 tw-px-4 sm:tw-px-8">
      <div>
        <div class="tw-mb-1">
          {{ edit ? "編輯群組" : "新增群組" }}
        </div>
        <div
          v-if="dialog && showHelp"
          class="tw-text-xs tw-font-normal tw-italic tw-text-dark-gray"
        >
          適合查看每週行事曆空閒時間
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
          <template v-slot:header>共用行事曆群組</template>
          <div class="mb-4">
            使用共用行事曆群組來查看成員每週的行事曆空閒時間。你的實際行事曆活動不會被其他人看到
          </div>
        </HelpDialog>
      </template>
    </v-card-title>
    <v-card-text class="tw-flex-1 tw-overflow-auto tw-px-4 tw-py-1 sm:tw-px-8">
      <v-form
        ref="form"
        class="tw-flex tw-flex-col tw-space-y-6"
        v-model="formValid"
        lazy-validation
        :disabled="loading"
      >
        <v-text-field
          ref="name-field"
          v-model="name"
          placeholder="為你的群組命名..."
          hide-details="auto"
          solo
          @keyup.enter="blurNameField"
          :rules="nameRules"
          required
        />

        <div>
          <div class="tw-mb-2 tw-text-lg tw-text-black">時間範圍</div>
          <div class="tw-flex tw-items-baseline tw-justify-center tw-space-x-2">
            <v-select
              v-model="startTime"
              menu-props="auto"
              :items="times"
              hide-details
              solo
            ></v-select>
            <div>至</div>
            <v-select
              v-model="endTime"
              menu-props="auto"
              :items="times"
              hide-details
              solo
            ></v-select>
          </div>
        </div>

        <div>
          <div class="tw-mb-2 tw-text-lg tw-text-black">天數範圍</div>
          <v-input
            v-model="selectedDaysOfWeek"
            hide-details="auto"
            :rules="selectedDaysRules"
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

        <!-- <div v-if="!edit"> -->
        <EmailInput
          ref="emailInput"
          :addedEmails="addedEmails"
          @update:emails="(newEmails) => (emails = newEmails)"
        >
          <template v-slot:header>
            <div class="tw-mb-2 tw-text-lg tw-text-black">成員</div>
          </template>
        </EmailInput>
        <!-- </div> -->

        <div>
          <v-btn
            class="tw-justify-start tw-pl-0"
            block
            text
            @click="showAdvancedOptions = !showAdvancedOptions"
            ><span class="tw-mr-1">進階選項</span>
            <v-icon :class="`tw-rotate-${showAdvancedOptions ? '180' : '0'}`"
              >mdi-chevron-down</v-icon
            ></v-btn
          >
          <v-expand-transition>
            <div v-show="showAdvancedOptions">
              <div class="tw-my-2">
                <TimezoneSelector v-model="timezone" label="時區" />
              </div>
            </div>
          </v-expand-transition>
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
          {{ edit ? "儲存變更" : "建立群組" }}
        </v-btn>
        <div
          :class="formValid ? 'tw-invisible' : 'tw-visible'"
          class="tw-mt-1 tw-text-xs tw-text-red"
        >
          請先修正表單中的錯誤再繼續
        </div>
      </div>
    </v-card-actions>
  </v-card>
</template>

<script>
import {
  validateEmail,
  isPhone,
  post,
  put,
  timeNumToTimeString,
  dateToTimeNum,
  getDateWithTimezone,
} from "@/utils"
import { mapState, mapActions } from "vuex"
import { eventTypes, dayIndexToDayString } from "@/constants"
import HelpDialog from "./HelpDialog.vue"
import TimezoneSelector from "./schedule_overlap/TimezoneSelector.vue"
import EmailInput from "./event/EmailInput.vue"

import dayjs from "dayjs"
import utcPlugin from "dayjs/plugin/utc"
import timezonePlugin from "dayjs/plugin/timezone"
dayjs.extend(utcPlugin)
dayjs.extend(timezonePlugin)

export default {
  name: "NewGroup",

  emits: ["input"],

  props: {
    event: { type: Object },
    edit: { type: Boolean, default: false },
    dialog: { type: Boolean, default: true },
    showHelp: { type: Boolean, default: false },
    contactsPayload: { type: Object, default: () => ({}) },
    folderId: { type: String, default: null },
  },

  components: {
    HelpDialog,
    TimezoneSelector,
    EmailInput,
  },

  data: () => ({
    formValid: true,
    name: "",
    startTime: 9,
    endTime: 17,
    loading: false,
    selectedDaysOfWeek: [],
    startOnMonday: false,
    emails: [],

    showAdvancedOptions: false,
    timezone: {},

    helpDialog: false,
    initialEventData: {},
  }),

  computed: {
    ...mapState(["authUser"]),
    nameRules() {
      return [(v) => !!v || "請輸入名稱"]
    },
    selectedDaysRules() {
      return [
        (selectedDays) =>
          selectedDays.length > 0 || "請至少選擇一天",
      ]
    },
    formEmpty() {
      return (
        this.name === "" &&
        this.emails.length === 0 &&
        this.selectedDaysOfWeek.length === 0
      )
    },
    isPhone() {
      return isPhone(this.$vuetify)
    },
    times() {
      const times = []

      for (let h = 1; h < 12; ++h) {
        times.push({ text: `上午 ${h} 點`, value: h })
      }
      for (let h = 0; h < 12; ++h) {
        times.push({ text: `下午 ${h == 0 ? 12 : h} 點`, value: h + 12 })
      }
      times.push({ text: "上午 12 點", value: 0 })

      return times
    },
    otherEventAttendees() {
      return this.event && this.event.attendees
        ? this.event.attendees
            .map((a) => a.email)
            .filter((email) => email !== this.authUser.email)
        : []
    },
    addedEmails() {
      if (Object.keys(this.contactsPayload).length > 0)
        return this.contactsPayload.emails
      return this.otherEventAttendees
    },
  },

  mounted() {
    if (Object.keys(this.contactsPayload).length > 0) {
      this.name = this.contactsPayload.name
      this.startTime = this.contactsPayload.startTime
      this.endTime = this.contactsPayload.endTime
      this.selectedDaysOfWeek = this.contactsPayload.selectedDaysOfWeek
      this.startOnMonday = this.contactsPayload.startOnMonday

      this.$refs.form.resetValidation()
    }
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
      this.selectedDaysOfWeek = []

      this.$refs.form.resetValidation()
    },
    submit() {
      if (!this.$refs.form.validate()) return

      // Get duration of event
      let duration = this.endTime - this.startTime
      if (duration < 0) duration += 24

      // Populate dates
      const dates = []
      const startTimeString = timeNumToTimeString(this.startTime)
      this.selectedDaysOfWeek.sort((a, b) => a - b)
      this.selectedDaysOfWeek = this.selectedDaysOfWeek.filter((dayIndex) => {
        return this.startOnMonday ? dayIndex !== 0 : dayIndex !== 7
      })
      for (const dayIndex of this.selectedDaysOfWeek) {
        const day = dayIndexToDayString[dayIndex]
        const date = dayjs.tz(`${day} ${startTimeString}`, this.timezone.value)

        // The reference dates (dayIndexToDayString) are from June 2018, which may have
        // a different DST offset than the current date. Adjust so the stored UTC time
        // corresponds to the user's current timezone offset.
        const refOffset = date.utcOffset()
        const currentOffset = dayjs().tz(this.timezone.value).utcOffset()
        dates.push(date.subtract(currentOffset - refOffset, "minutes").toDate())
      }

      this.loading = true

      const name = this.name
      const type = eventTypes.GROUP
      const attendees = this.emails
      const startOnMonday = this.startOnMonday

      if (!this.edit) {
        // Create a new group
        post("/events", {
          name,
          duration,
          dates,
          attendees,
          type,
          startOnMonday,
          creatorPosthogId: this.$posthog?.get_distinct_id(),
        })
          .then(async ({ eventId, shortId }) => {
            if (this.authUser) {
              await this.setEventFolder({ eventId, folderId: this.folderId })
            }
            this.$router.push({
              name: "group",
              params: {
                groupId: shortId ?? eventId,
                initialTimezone: this.timezone,
              },
            })
            this.$emit("input", false)

            this.$posthog?.capture("Availability group created", {
              eventId: eventId,
              eventName: name,
              eventDuration: duration,
              eventDates: JSON.stringify(dates),
              eventAttendees: attendees,
              eventType: type,
              eventStartOnMonday: startOnMonday,
            })
          })
          .catch((err) => {
            this.showError(
              "建立群組時發生問題！請稍後再試"
            )
            console.error(err)
          })
          .finally(() => {
            this.loading = false
          })
      } else {
        // Edit group
        put(`/events/${this.event._id}`, {
          name,
          duration,
          dates,
          attendees,
          type,
          startOnMonday,
        })
          .then(() => {
            this.$posthog?.capture("Availability group edited", {
              eventId: this.event._id,
              eventName: name,
              eventDuration: duration,
              eventDates: JSON.stringify(dates),
              eventAttendees: attendees,
              eventType: type,
              eventStartOnMonday: startOnMonday,
            })

            this.$emit("input", false)
            this.reset()
            window.location.reload()
          })
          .catch((err) => {
            this.showError(
              "編輯群組時發生問題！請稍後再試"
            )
          })
          .finally(() => {
            this.loading = false
          })
      }
    },
    /** Populate fields with data from event */
    updateFieldsFromEvent() {
      if (this.event) {
        this.name = this.event.name

        // Set start time, accounting for the timezone
        this.startTime = Math.floor(
          dateToTimeNum(getDateWithTimezone(this.event.dates[0]), true)
        )
        this.startTime %= 24

        this.endTime = (this.startTime + this.event.duration) % 24
        this.startOnMonday = this.event.startOnMonday

        const selectedDaysOfWeek = []
        for (let date of this.event.dates) {
          date = getDateWithTimezone(date)

          if (this.startOnMonday && date.getUTCDay() === 0) {
            selectedDaysOfWeek.push(7)
          } else {
            selectedDaysOfWeek.push(date.getUTCDay())
          }
        }
        this.selectedDaysOfWeek = selectedDaysOfWeek

        this.emails = this.otherEventAttendees
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
        selectedDaysOfWeek: this.selectedDaysOfWeek,
        emails: [...this.emails],
      }
    },
    hasEventBeenEdited() {
      return (
        this.name !== this.initialEventData.name ||
        this.startTime !== this.initialEventData.startTime ||
        this.endTime !== this.initialEventData.endTime ||
        JSON.stringify(this.selectedDaysOfWeek) !==
          JSON.stringify(this.initialEventData.selectedDaysOfWeek) ||
        JSON.stringify(this.emails) !==
          JSON.stringify(this.initialEventData.emails)
      )
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
    formEmpty(val) {
      this.$emit("update:formEmpty", val)
    },
  },
}
</script>
