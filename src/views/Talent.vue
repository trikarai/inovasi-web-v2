<template>
  <div class="home">
    <Navbar />
    <transition name="slide" mode="out-in">
      <router-view></router-view>
    </transition>
    <v-spacer></v-spacer>
    <Footer />
  </div>
</template>

<script>
// @ is an alias to /src
import auth from "@/config/auth";
import Navbar from "@/components/talent/TalentNavBar";
import Footer from "@/components/Footer";
export default {
  name: "Home",
  components: { Navbar, Footer },
  data: function() {
    return {
      user: ""
    };
  },
  created: function() {
    this.stateProfileStatus();
  },
  methods: {
    stateProfileStatus: function() {
      var payload = JSON.parse(auth.getAuthData());
      this.$store.commit("setProfile", payload);
    }
  }
};
</script>
<style scoped>
.slide-fade-enter-active {
  transition: all 0.3s ease;
}
.slide-fade-leave-active {
  transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
}
.slide-fade-enter,
.slide-fade-leave-to {
  transform: translateX(10px);
  opacity: 0;
}
</style>

