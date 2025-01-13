<template>
  <div class="page-container">
    <div id="g_id_onload" :data-client_id="clientId" data-context="signin" data-ux_mode="redirect"
      data-auto_prompt="false"></div>
    <div class="g_id_signin"></div>
  </div>
</template>

<script>
import { onMounted } from "vue";

export default {
  name: "GoogleSignIn",
  props: {},
  emits: ["user-logged-in"],
  data() {
    return {
      clientId: import.meta.env.VITE_GOOGLE_CLIENT_ID,
    };
  },
  methods: {
    loadGoogleSignIn() {
      if (window.google) {
        window.google.accounts.id.initialize({
          client_id: this.clientId,
          callback: this.handleCredentialResponse,
        });

        window.google.accounts.id.renderButton(
          document.querySelector(".g_id_signin"),
          {
            theme: "outline",
            size: "large",
            type: "standard",
            shape: "pill",
            text: "signin_with",
            logo_alignment: "left",
          }
        );
      }
    },
    handleCredentialResponse(response) {
      const token = response.credential;
      fetch(`https://www.googleapis.com/oauth2/v3/tokeninfo?id_token=${token}`)
        .then((res) => res.json())
        .then((data) => {
          const userInfo = {
            name: data.name,
            email: data.email,
            picture: data.picture,
          }
          this.$emit("user-logged-in", {
            ...userInfo
          });
          localStorage.setItem('userInfo', JSON.stringify(userInfo))
        })
        .catch((error) => {
          console.error("Google API verification failed:", error);
        });
    },
  },
  mounted() {
    this.loadGoogleSignIn();
  },
};
</script>

<style></style>
