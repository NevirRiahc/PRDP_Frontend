<template>
   <i-frame v-model:src="url"></i-frame>
</template>

<script setup>
import iFrame from '@/components/iFrame'
import { ref } from 'vue';

// const url = ref(import.meta.env.VITE_APP_BASE_API + "/swagger-ui/index.html")
const url = ref("http://47.97.185.74:5601/")
// const url = ref("http://124.71.192.57:8022")
// const url = ref("https://jenkins.flybycloud.com/")
console.log(import.meta.env.VITE)
</script>
