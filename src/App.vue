<script setup>
import { ref, onMounted } from "vue";
import Login from "./components/Login.vue";
import Dashboard from "./components/Dashboard.vue";
import Stock from "./components/Stock.vue";
import Tracking from "./components/Tracking.vue";
import HistoryBahanAjar from "./components/History-bahanajar.vue";

const isVerified = ref(false);
const activePage = ref("Dashboard");

onMounted(() => {
  isVerified.value = localStorage.getItem("isVerified") === "true";
});

const handleLoginSuccess = () => {
  localStorage.setItem("isVerified", "true");
  isVerified.value = true;
  activePage.value = "Dashboard";
};

const handleLogout = () => {
  localStorage.removeItem("isVerified");
  localStorage.removeItem("namaUser");
  isVerified.value = false;
  activePage.value = "Dashboard";
};

const handleNavigate = (page) => {
  activePage.value = page;
};
</script>

<template>
  <main>
    <Login v-if="!isVerified" @login-success="handleLoginSuccess" />
    <Dashboard
      v-else-if="activePage === 'Dashboard'"
      @logout="handleLogout"
      @navigate="handleNavigate"
    />
    <Stock
      v-else-if="activePage === 'Stock'"
      @logout="handleLogout"
      @navigate="handleNavigate"
    />
    <Tracking
      v-else-if="activePage === 'Tracking'"
      @logout="handleLogout"
      @navigate="handleNavigate"
    />
    <HistoryBahanAjar
      v-else-if="activePage === 'HistoryBahanAjar'"
      @logout="handleLogout"
      @navigate="handleNavigate"
    />
  </main>
</template>
