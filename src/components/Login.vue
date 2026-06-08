<script setup>
import Swal from "sweetalert2";
import bgImage from "../assets/img/logo.jpg";
import { computed, ref } from "vue";
import { dataPengguna } from "../contains/data.js";

const emit = defineEmits(["loginSuccess"]);

// State untuk menyimpan input email dan password
const email = ref("");
const password = ref("");
const isFormValid = computed(
  () => email.value.trim().length > 0 && password.value.trim().length > 0,
);

const handleLogin = () => {
  const user = dataPengguna.find(
    (u) => u.email === email.value && u.password === password.value,
  );
  if (user) {
    localStorage.setItem("namaUser", user.nama);
    Swal.fire({
      title: "Login Berhasil",
      text: `Selamat Datang, ${user.nama}!`,
      icon: "success",
      confirmButtonColor: "blue",
      confirmButtonText: "Lanjutkan",
    }).then(() => {
      emit("loginSuccess");
    });
  } else {
    Swal.fire({
      title: "Login Gagal!",
      text: "Email atau Password salah. Silahkan coba lagi...",
      icon: "error",
      confirmButtonColor: "#1e3a8a",
      confirmButtonText: "OK",
    });
  }
};

// Logika untuk "Lupa Password?"

const handleForgetPassword = () => {
  // Logika untuk menangani lupa password
  Swal.fire({
    title: "Lupa Password",
    text: "Silahkan hubungi admin untuk reset password.",
    icon: "info",
    confirmButtonColor: "#1e3a8a",
    confirmButtonText: "OK",
  });
};

// Logika untuk "Belum punya akun?"
const handleDontHaveAccount = () => {
  // Logika untuk menangani pendaftaran akun baru
  Swal.fire({
    title: "Belum Punya Akun",
    text: "Silahkan hubungi admin untuk membuat akun.",
    icon: "info",
    confirmButtonColor: "#1e3a8a",
    confirmButtonText: "OK",
  });
};
</script>

<template>
  <svg
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 1440 320"
    class="absolute inset-0 -z-10 w-full"
  >
    <path
      fill="#0099ff"
      fill-opacity="1"
      d="M0,224L48,224C96,224,192,224,288,234.7C384,245,480,267,576,240C672,213,768,139,864,128C960,117,1056,171,1152,181.3C1248,192,1344,160,1392,144L1440,128L1440,0L1392,0C1344,0,1248,0,1152,0C1056,0,960,0,864,0C768,0,672,0,576,0C480,0,384,0,288,0C192,0,96,0,48,0L0,0Z"
    ></path>
  </svg>
  <section class="flex justify-center items-center min-h-screen">
    <div class="login-box w-100 p-8 pt-15 bg-white rounded-lg shadow-lg">
      <img
        :src="bgImage"
        alt="Logo UT"
        class="w-22 h-auto mx-auto mb-6 cursor-pointer"
      />
      <h1 class="text-3xl font-extrabold">Selamat Datang</h1>
      <!-- Form Start -->
      <form @submit.prevent="handleLogin" class="mt-10">
        <!-- Input Email -->
        <div class="flex items-center gap-2 mb-4 border-b">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            height="24px"
            viewBox="0 -960 960 960"
            width="20px"
            fill="#000000"
          >
            <path
              d="M168-192q-29.7 0-50.85-21.16Q96-234.32 96-264.04v-432.24Q96-726 117.15-747T168-768h624q29.7 0 50.85 21.16Q864-725.68 864-695.96v432.24Q864-234 842.85-213T792-192H168Zm312-240L168-611v347h624v-347L480-432Zm0-85 312-179H168l312 179Zm-312-94v-85 432-347Z"
            />
          </svg>
          <input
            type="email"
            name="email"
            id="email"
            placeholder="Email"
            class="p-3 w-full focus:outline-none"
            v-model="email"
            autocomplete="email"
            required
          />
        </div>
        <!-- Input Password -->
        <div>
          <div class="flex items-center gap-2 mb-4 border-b">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              height="24px"
              viewBox="0 -960 960 960"
              width="20px"
              fill="#000000"
            >
              <path
                d="M220-412q-28-28-28-68t28-68q28-28 68-28t68 28q28 28 28 68t-28 68q-28 28-68 28t-68-28Zm68 172q-100 0-170-70T48-480q0-100 70-170t170-70q65 0 120 32.5t88 87.5h344l120 120-180 168-84-60-72 60-96-72h-20q-24 68-85.5 106T288-240Zm0-72q63 0 111-40.5T454-456h98l70 52 71-59 81 58 82-76-46-47H449q-19-53-62.5-86.5T288-648q-70 0-119 49t-49 119q0 70 49 119t119 49Z"
              />
            </svg>
            <input
              type="password"
              name="password"
              id="password"
              v-model="password"
              placeholder="Password"
              class="p-3 w-full focus:outline-none"
              autocomplete="current-password"
              required
            />
          </div>
          <a
            href="#"
            @click="handleForgetPassword()"
            class="text-blue-600 underline text-right block hover:text-blue-950 cursor-pointer"
            >Lupa Password?</a
          >
        </div>
        <br />
        <button
          type="submit"
          :disabled="!isFormValid"
          class="p-2 w-full rounded font-bold transition-colors cursor-pointer"
          :class="
            isFormValid
              ? 'bg-yellow-300 hover:bg-yellow-400 text-black'
              : 'bg-gray-300 text-gray-700 cursor-not-allowed'
          "
        >
          Masuk
        </button>
      </form>
      <!-- Form End  -->

      <a
        href="#"
        @click="handleDontHaveAccount()"
        class="text-blue-600 underline text-right block hover:text-blue-950 cursor-pointer"
        >Belum punya akun?</a
      >
    </div>
  </section>
</template>
