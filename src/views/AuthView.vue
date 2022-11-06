<template>
  <div v-if="!user" class="before-signin">
    <button @click="sendEmailLink">send email link</button>
    <div>please login.</div>
  </div>
  <div v-else class="after-signedin">
    <button @click="logout">logout</button>
    <div>hello, {{ user }}!</div>
  </div>
</template>

<script lang="ts">
import { initializeApp } from "firebase/app";

import {
  getAuth,
  isSignInWithEmailLink,
  onAuthStateChanged,
  sendSignInLinkToEmail,
  signInWithEmailLink,
  signOut,
} from "firebase/auth";

import { defineComponent } from "vue";

const firebaseConfig = {
  apiKey: "AIzaSyBVGGJepZnqYgOLIVaL04aMYX7WfGlaVbo",
  authDomain: "mail-link-test-2ebfa.firebaseapp.com",
  projectId: "mail-link-test-2ebfa",
  storageBucket: "mail-link-test-2ebfa.appspot.com",
  messagingSenderId: "435055448985",
  appId: "1:435055448985:web:57df823cf8a0c738f1f908",
  measurementId: "G-9ZN8RSFT9P",
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
auth.useDeviceLanguage();

export default defineComponent({
  name: "AuthView",
  data() {
    return { user: auth.currentUser?.email };
  },
  mounted() {
    onAuthStateChanged(auth, (user) => {
      if (user) {
        // User is signed in, see docs for a list of available properties
        // https://firebase.google.com/docs/reference/js/firebase.User
        this.user = user.email;
        // ...
      } else {
        // User is signed out
        // ...
        this.signinByEmailLink();
      }
    });
  },
  methods: {
    signinByEmailLink() {
      if (isSignInWithEmailLink(auth, window.location.href)) {
        // Additional state parameters can also be passed via URL.
        // This can be used to continue the user's intended action before triggering
        // the sign-in operation.
        // Get the email if available. This should be available if the user completes
        // the flow on the same device where they started it.
        let email = window.localStorage.getItem("emailForSignIn");
        if (!email) {
          // User opened the link on a different device. To prevent session fixation
          // attacks, ask the user to provide the associated email again. For example:
          email = window.prompt("Please provide your email for confirmation");
        }
        if (!email) return;
        // The client SDK will parse the code from the link for you.
        signInWithEmailLink(auth, email, window.location.href)
          .then((result) => {
            // Clear email from storage.
            window.localStorage.removeItem("emailForSignIn");
            // You can access the new user via result.user
            // Additional user info profile not available via:
            // result.additionalUserInfo.profile == null
            // You can check if the user is new or existing:
            // result.additionalUserInfo.isNewUser
            this.user = result.user.email ?? "";
            this.$router.push("/auth");
          })
          .catch((error) => {
            // Some error occurred, you can inspect the code: error.code
            // Common errors could be invalid email and invalid or expired OTPs.
            console.log(error.code);
            console.log(error.message);
          });
      }
    },
    sendEmailLink() {
      const email = window.prompt("Please provide your email for verification");
      if (!email) return;
      const actionCodeSettings = {
        // URL you want to redirect back to. The domain (www.example.com) for this
        // URL must be in the authorized domains list in the Firebase Console.
        url: "http://localhost:8080/auth",
        // This must be true.
        handleCodeInApp: true,
      };
      sendSignInLinkToEmail(auth, email, actionCodeSettings)
        .then(() => {
          // The link was successfully sent. Inform the user.
          // Save the email locally so you don't need to ask the user for it again
          // if they open the link on the same device.
          window.localStorage.setItem("emailForSignIn", email);
          // ...
        })
        .catch((error) => {
          console.log(error.code);
          console.log(error.message);
          // ...
        });
    },
    logout() {
      signOut(auth)
        .then(() => {
          // Sign-out successful.
          this.user = "";
        })
        .catch((error) => {
          // An error happened.
          console.log(error.code);
          console.log(error.message);
        });
    },
  },
});
</script>
