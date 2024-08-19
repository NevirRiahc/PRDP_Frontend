<template>
   <i-frame v-model:src="url" async></i-frame>
</template>

<script setup>
import iFrame from '@/components/iFrame'
import { ref } from 'vue';

// const url = ref(import.meta.env.VITE_APP_BASE_API + "/swagger-ui/index.html")
// const url = ref("http://47.97.185.74:81/")
const url = ref("https://jenkins.flybycloud.com/")
console.log(import.meta.env.VITE)
</script>
