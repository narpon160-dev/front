<template>
  <div>
    <!-- ถ้าตัวแปร currentPage เป็น 'home' ให้แสดงหน้า Home.vue -->
    <Home v-if="currentPage === 'home'" @go-to-cal="handleRoomSelection" />

    <!-- ถ้าไม่ใช่ ให้แสดงหน้า cal.vue แทน พร้อมส่งข้อมูลห้องพักผ่าน props -->
    <Cal v-else :room-data="selectedRoomData" @go-back="currentPage = 'home'" />
  </div>
</template>

<script setup>
import { ref } from "vue";
import Home from "./components/home.vue";
import Cal from "./components/cal.vue";

const currentPage = ref("home");
const selectedRoomData = ref(null);

const handleRoomSelection = (data) => {
  console.log("ห้องที่เลือกมาจากหน้า Home:", data);
  selectedRoomData.value = data;
  currentPage.value = "cal";
};
</script>

<style>
body {
  margin: 0;
  padding: 0;
  background: transparent;
  font-family: "Segoe UI", Tahoma, sans-serif;
}
#app {
  max-width: 100% !important;
  padding: 0 !important;
  margin: 0 !important;
  width: 100%;
}
</style>
