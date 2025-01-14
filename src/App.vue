<script setup>
import { ref } from 'vue'
import GoogleSignIn from './components/GoogleSignIn.vue'
import Map from './components/Map.vue'

const userInfo = ref(null);
const updateUserInfo = (data) => {
    userInfo.value = data;
};
const localUserInfo = localStorage.getItem('userInfo');
if (localUserInfo) updateUserInfo(JSON.parse(localUserInfo))
</script>

<template>
    <Map v-if="userInfo" />
    <GoogleSignIn v-else @user-logged-in="updateUserInfo" />
</template>

<style scoped></style>