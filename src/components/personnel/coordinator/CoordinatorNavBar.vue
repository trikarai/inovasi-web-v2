<template>
  <nav>
    <v-app-bar text app color="primary">
      <v-app-bar-nav-icon class="white--text" @click="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title class="text-uppercase white--text">
        <span class="font-weight-light">START</span>
        <span class>Mikti</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn text @click="rightDrawer =! rightDrawer">
        <v-icon>settings</v-icon>
      </v-btn>
    </v-app-bar>

    <div>
      <v-breadcrumbs :items="items" divider=">" style="visibility:hidden"></v-breadcrumbs>
    </div>

    <v-navigation-drawer app v-model="drawer" :mini-variant.sync="miniVariant">
      <!-- list head-->
      <v-list class="pa-1">
        <v-list-item v-if="miniVariant" @click.stop="miniVariant = !miniVariant">
          <v-list-item-action>
            <v-icon>chevron_right</v-icon>
          </v-list-item-action>
        </v-list-item>
        <v-list-item>
          <v-list-item-avatar>
            <img src="https://randomuser.me/api/portraits/men/85.jpg" />
          </v-list-item-avatar>
          <v-list-item-content>
            <v-list-item-title>{{user.data.name}}</v-list-item-title>
          </v-list-item-content>
          <v-list-item-action>
            <v-btn icon @click.stop="miniVariant = !miniVariant">
              <v-icon>chevron_left</v-icon>
            </v-btn>
          </v-list-item-action>
        </v-list-item>
      </v-list>
      <!-- link router -->
      <v-list>
        <!--dashboar-->
        <v-list-item router to="/coordinator/dashboard">
          <v-list-item-action>
            <v-icon color="#676767">dashboard</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title class="grey--text">Dashboard</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <!-- profile-->
        <v-list-group prepend-icon="account_circle" value="true" no-action>
          <template v-slot:activator>
            <v-list-item>
              <v-list-item-title>Profile</v-list-item-title>
            </v-list-item>
          </template>
          <template v-if="!miniVariant">
            <v-list-item v-for="link in linkProfile" :key="link.text" router :to="link.route">
              <v-list-item-title>{{link.text}}</v-list-item-title>
              <v-list-item-action>
                <v-icon>{{link.icon}}</v-icon>
              </v-list-item-action>
            </v-list-item>
          </template>
        </v-list-group>
        <!--sub list other-->
        <v-list-item v-for="link in links" :key="link.text" router :to="link.route">
          <v-list-item-action>
            <v-icon color="#676767">{{link.icon}}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title class="grey--text">{{link.text}}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
      <v-spacer></v-spacer>
      <v-divider></v-divider>
      <v-list dense class="py-0">
        <v-list-item @click="gotoTalentMenu()">
          <v-list-item-action>
            <v-icon color="#676767">pages</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title class="grey--text">Talent Menu</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item v-if="checkProgramMentorship" @click="gotoMentorMenu()">
          <v-list-item-action>
            <v-icon color="#676767">pages</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title class="grey--text">Mentor Menu</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-navigation-drawer temporary right v-model="rightDrawer" fixed>
      <v-list>
        <v-list-item>
          <v-list-item-avatar>
            <v-icon>exit_to_app</v-icon>
          </v-list-item-avatar>
          <v-list-item-content></v-list-item-content>
          <v-list-item-action>
            <v-btn small color="red" @click="logout" v-if="checkLogin">
              <span>{{ $vuetify.lang.t('$vuetify.action.signout') }}</span>
            </v-btn>
          </v-list-item-action>
        </v-list-item>
        <v-list-item>
          <v-list-item-avatar>
            <v-icon>language</v-icon>
          </v-list-item-avatar>
          <v-list-item-content></v-list-item-content>
          <v-list-item-action>
            <LocaleSwitcher />
          </v-list-item-action>
        </v-list-item>
        <v-list-item>
          <v-list-item-avatar>
            <v-icon>brightness_4</v-icon>
          </v-list-item-avatar>
          <v-list-item-content>Dark Theme</v-list-item-content>
          <v-list-item-action>
            <v-switch color="black" @change="switchTheme"></v-switch>
          </v-list-item-action>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
  </nav>
</template>
<script>
import auth from "@/config/auth";
import LocaleSwitcher from "../../LocaleSwitcher";
export default {
  data: function() {
    return {
      drawer: true,
      rightDrawer: false,
      miniVariant: false,
      clipped: true,
      fixed: false,
      user: "",
      linkProfile: [
        {
          icon: "perm_identity",
          text: "My Profile",
          route: "/coordinator/profile"
        }
      ],
      links: [
        {
          icon: "local_library",
          text: "Program",
          route: "/coordinator/program"
        }
      ],
      items: [
        {
          text: "Dashboard",
          disabled: false,
          to: "breadcrumbs_dashboard"
        },
        {
          text: "Link 1",
          disabled: false,
          to: "breadcrumbs_link_1"
        },
        {
          text: "Link 2",
          disabled: true,
          to: "breadcrumbs_link_2"
        }
      ]
    };
  },
  components: {
    LocaleSwitcher
  },
  created: function() {
    this.user = JSON.parse(auth.getAuthData());
  },
  computed: {
    checkLogin() {
      return this.$store.state.isLoggedIn;
    },
    checkProgramMentorship() {
      var check;
      if (this.user.data.programmeMentorships.length !== 0) {
        check = true;
      } else {
        check = false;
      }
      return check;
    }
  },
  methods: {
    logout: function() {
      var app = this;
      localStorage.removeItem("lbUser");
      app.$router.replace({ path: "/login" });
      app.$store.state.isLoggedIn = false;
    },
    switchTheme: function() {
      this.$vuetify.theme.dark = !this.$vuetify.theme.dark;
    },
    gotoTalentMenu: function() {
      localStorage.setItem("dashboard", "talent");
      this.$router.push({ path: "/talent/dashboard" });
    },
    gotoMentorMenu: function() {
      localStorage.setItem("dashboard", "mentor");
      this.$router.push({ path: "/mentor/dashboard" });
    }
  }
};
</script>
<style scoped>
</style>

