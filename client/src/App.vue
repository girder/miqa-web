<script>
import { mapActions, mapGetters } from "vuex";
export default {
  name: "App",
  components: {},
  inject: ["girderRest", "djangoRest"],
  provide() {
    return { userLevel: this.userLevel };
  },
  data: () => ({
    userLevel: { value: null }
  }),
  computed: {
    ...mapGetters(["sessionStatus"])
  },
  watch: {
    sessionStatus(status) {
      if (status === "timeout") {
        this.$prompt({
          title: "Session Expired",
          text: "Your session has expired and you will be logged out",
          positiveButton: "Ok"
        }).then(() => {
          this.logout();
        });
      }
    }
  },
  created() {
    this.setUserLevel();
  },
  methods: {
    ...mapActions(["logout"]),
    async setUserLevel() {
      if (!this.girderRest.user) {
        return;
      }
      this.userLevel.value = await this.getUserLevel();
    },
    async getUserLevel() {
      if (this.girderRest.user.admin) {
        return 0;
      }
      var roles = await Promise.all(
        this.girderRest.user.groups.map(async id => {
          var { data: group } = await this.girderRest.get(`group/${id}`);
          return group.name.toLowerCase();
        })
      );
      if (roles.indexOf("manager") !== -1) {
        return 1;
      }
      if (roles.indexOf("reviewer") !== -1) {
        return 2;
      }
      if (roles.indexOf("collaborator") !== -1) {
        return 3;
      }
      return null;
    }
  }
};
</script>

<template>
  <v-app id="app">
    <v-main>
      <router-view />
    </v-main>
  </v-app>
</template>

<style lang="scss">
html {
  overflow-y: auto !important;
}
</style>
