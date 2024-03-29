<template>
  <LoadingSpinner v-if="isLoading" />
  <div>
    <FavoriteButton :isFavorited="isFavorited" @toggle-favorite="toggleFavBtn" />
    <WordDetials :word="word" :description="description" :innerImg="innerImg" />
  </div>
</template>


<script>
import LoadingSpinner from "@/components/common/LoadingSpinner.vue";
import WordDetials from "@/components/common/WordDetials.vue";
import FavoriteButton from '@/components/common/FavoriteButton.vue';


import { db, auth } from "@/firebaseConfig";
import {
  doc,
  getDoc,
  updateDoc,
  arrayUnion,
  arrayRemove,
  onSnapshot,
} from "@firebase/firestore";
import { onAuthStateChanged } from "firebase/auth";

import { mapGetters } from "vuex";
import { useToast } from "vue-toastification";

export default {
  components: {
    WordDetials,
    LoadingSpinner,
    FavoriteButton
  },
  data() {
    return {
      dictionaryKey: null,
      word: "",
      innerImg: "",
      description: "",
      isLoading: true,
      isFavorited: false,
    };
  },
  computed: {
    ...mapGetters({
      isLoggedIn: "getUserLoginState",
    }),
    toastOptions() {
      return {
        position: "top-right",
        timeout: 4000,
        closeOnClick: true,
        pauseOnFocusLoss: true,
        pauseOnHover: true,
        draggable: true,
        draggablePercent: 0.6,
        showCloseButtonOnHover: false,
        hideProgressBar: true,
        closeButton: "button",
        icon: true,
        rtl: false,
      };
    },
  },
  methods: {
    async toggleFavBtn() {
      this.isFavorited = !this.isFavorited;

      // Update user's favorites in firestore
      const user = auth.currentUser;
      if (user) {
        const userRef = doc(db, "users", user.uid);
        const favoritesUpdate = {
          [`favorites.dictionary.${this.dictionaryKey}`]: this.isFavorited
            ? arrayUnion(this.word)
            : arrayRemove(this.word),
        };
        await updateDoc(userRef, favoritesUpdate);
      } else {
        console.log("Login first to use this feature!");
        const toast = useToast();
        toast.warning('Login first to use favorite feature!', this.toastOptions);
        this.isFavorited = false;
      }
    },
    checkIfWordIsFavorite() {
      onAuthStateChanged(auth, (user) => {
        if (user) {
          const userRef = doc(db, "users", user.uid);
          onSnapshot(userRef, (snapshot) => {
            const userData = snapshot.data();
            const userFavoritesArray =
              userData.favorites.dictionary[this.dictionaryKey];
            console.log(userFavoritesArray);
            userFavoritesArray.forEach((item) => {
              if (item === this.word) {
                this.isFavorited = true;
              }
            });
          });
        }
      });
    },
  },
  mounted() {
    console.log(this.word, this.description);
    // this.checkIfWordIsFavorite();
  },

  watch: {
    $route: {
      immediate: true,
      handler: function (newRoute) {
        const queryWord = newRoute.query.word;
        const newLetter = newRoute.params.letter;
        const dictionaryRef = doc(db, "dictionary", newLetter);
        getDoc(dictionaryRef).then((doc) => {
          if (doc.exists()) {
            const words = doc.data().words;
            words.forEach((sign) => {
              if (sign.word === queryWord) {
                this.dictionaryKey = newLetter;
                this.word = sign.word;
                this.innerImg = sign.innerImg;
                this.description = sign.description;
                this.isLoading = false;
                this.checkIfWordIsFavorite();
              }
            });
          }
        });
      },
    },
  },
};
</script>


<style lang="scss" scoped>

</style>