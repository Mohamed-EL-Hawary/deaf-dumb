<template>
  <div>
    <header>
      <TheNavbar />
    </header>
    <router-view />
    <TheFooter />
  </div>
</template>

<script>
import TheNavbar from "./components/TheNavbar.vue";
import TheFooter from "./components/TheFooter.vue";
import { mapActions, mapGetters } from "vuex";
import { onAuthStateChanged } from "@firebase/auth";
import { auth, db } from "./firebaseConfig";
import { doc, updateDoc } from "@firebase/firestore";

export default {
  components: {
    TheNavbar,
    TheFooter,
  },
  data() {
    return {
    };
  },
  methods: {
    ...mapActions(['setLoginState'])
  },
  computed: {
    ...mapGetters({
      isLoggedIn: 'getUserLoginState'
    })
  },
  mounted() {
    console.log(this.getUserName)
    onAuthStateChanged(auth, (user) => {
      if (user) {
        // User is Signed in
        updateDoc(doc(db, 'users', user.uid), {
          isLoggedIn: this.isLoggedIn
        })

        if (user.photoURL) {
          this.$store.dispatch('setUserPhoto', user.photoURL);
        }else {
          this.$store.dispatch('setUserPhoto', '');
        }

      } else {
        // User is signed out
        console.log("User is signed out");
        this.setLoginState(false);
      }
    });
  },
};
</script>

<style lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Manrope:wght@200;300;400;500;600;700;800&display=swap");
// @import "./scss/custom.scss";

#app {
  // font-family: Avenir, Helvetica, Arial, sans-serif;
  font-family: "Manrope", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  // text-align: center;
}

.background {
  position: absolute;
  display: block;
  top: 0;
  left: 0;
  z-index: 0;
}
</style>
