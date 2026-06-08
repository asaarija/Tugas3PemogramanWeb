<script setup>
import Header from "./shares/Header.vue";

const emit = defineEmits(["logout", "navigate"]);

const handleLogout = () => {
  emit("logout");
};

const handleNavigate = (page) => {
  emit("navigate", page);
};
</script>
<template>
  <Header
    :activePage="'HistoryBahanAjar'"
    @logout="handleLogout"
    @navigate="handleNavigate"
  />
  <div class="w-full h-screen bg-white">
    <svg
      xmlns="http://www.w3.org/2000/svg"
      viewBox="0 0 1440 320"
      class="w-full"
    >
      <path
        fill="#0099ff"
        fill-opacity="1"
        d="M0,224L48,224C96,224,192,224,288,234.7C384,245,480,267,576,240C672,213,768,139,864,128C960,117,1056,171,1152,181.3C1248,192,1344,160,1392,144L1440,128L1440,0L1392,0C1344,0,1248,0,1152,0C1056,0,960,0,864,0C768,0,672,0,576,0C480,0,384,0,288,0C192,0,96,0,48,0L0,0Z"
      ></path>
    </svg>
    <div class="bg-white bg-opacity-80 p-8 w-full rounded">
      <h1 class="text-6xl font-extrabold mb-4">History Bahan Ajar</h1>
      <p>Fitur ini sedang dalam pengembangan. Mohon ditunggu!</p>
    </div>
  </div>
</template>
