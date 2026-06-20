<template></template>

<script>
import { get, post, getEventsCreated, deleteEventsCreated } from "@/utils"
import { mapMutations, mapState } from "vuex"
import { authTypes } from "@/constants"

export default {
  name: "Auth",

  computed: {
    ...mapState(["authUser"]),
  },

  methods: {
    ...mapMutations(["setAuthUser"]),
  },

  async created() {
    let { error, code, scope, state } = this.$route.query
    if (error) this.$router.replace({ name: "home" })

    if (state) state = JSON.parse(decodeURIComponent(state))

    try {
      const user = await post("/auth/sign-in", {
        code,
        scope: scope ?? state?.scope,
        timezoneOffset: new Date().getTimezoneOffset(),
        eventsToLink: getEventsCreated(),
      })
      deleteEventsCreated()

      this.setAuthUser(user)

      this.$posthog?.identify(user._id, {
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
      })

      if (state) {
        let authUser
        switch (state.type) {
          case authTypes.EVENT_ADD_AVAILABILITY:
            this.$router.replace({
              name: "event",
              params: { eventId: state.eventId, fromSignIn: true },
            })
            break
          case authTypes.EVENT_SIGN_IN:
            this.$router.replace({
              name: "event",
              params: { eventId: state.eventId },
            })
            break
          case authTypes.GROUP_CREATE:
            this.$router.replace({
              name: "home",
              params: {
                openNewGroup: true,
              },
            })
            break
          case authTypes.GROUP_SIGN_IN:
            this.$router.replace({
              name: "group",
              params: { groupId: state.groupId },
            })
            break
          case authTypes.GROUP_ADD_AVAILABILITY:
            this.$router.replace({
              name: "group",
              params: { groupId: state.eventId, fromSignIn: true },
            })
            authUser = await get("/user/profile")
            this.setAuthUser(authUser)
            break
          case authTypes.EVENT_CONTACTS:
            if (state.eventId == "") {
              this.$router.replace({
                name: "home",
                params: {
                  contactsPayload: state.payload,
                  openNewGroup: state.openNewGroup,
                },
              })
            } else {
              this.$router.replace({
                name: "event",
                params: {
                  eventId: state.eventId,
                  contactsPayload: state.payload,
                },
              })
            }
            break
          case authTypes.UPGRADE:
            try {
              const params = JSON.parse(state.upgradeParams)
              const res = await post("/stripe/create-checkout-session", {
                priceId: params.priceId,
                userId: this.authUser._id,
                isSubscription: params.isSubscription,
                originUrl: params.originUrl,
              })
              window.location.href = res.url
            } catch (e) {
              console.error(e)
              this.$router.replace({ name: "home" })
            }
            break
          default:
            this.$router.replace({ name: "home" })
        }
      } else {
        this.$router.replace({ name: "home" })
      }
    } catch (err) {
      console.error(err)
    }
  },
}
</script>
