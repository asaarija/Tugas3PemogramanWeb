<script setup>
import Swal from "sweetalert2";
import Header from "./shares/Header.vue";
import InputDelivery from "./shares/Input-delivery.vue";
import { ref, onMounted } from "vue";
import dataBahanAjar from "@/contains/dataBahanAjar.json";

const emit = defineEmits(["logout", "navigate"]);
const showInputModal = ref(false);
const allTrackingData = ref({});
const deliveryRecords = ref([]);
const progressDescription = ref("");

const initializeTrackingData = () => {
  if (dataBahanAjar?.tracking && Array.isArray(dataBahanAjar.tracking)) {
    dataBahanAjar.tracking.forEach((item) => {
      const [nomorDO, trackingInfo] = Object.entries(item)[0] ?? [];
      if (!nomorDO || !trackingInfo) return;
      allTrackingData.value[nomorDO] = {
        nomorDO,
        ...trackingInfo,
        total:
          typeof trackingInfo.total === "number"
            ? `Rp ${trackingInfo.total.toLocaleString("id-ID")},-`
            : trackingInfo.total,
      };
    });
  }
};

initializeTrackingData();

onMounted(() => {
  // Load data pengiriman baru dari localStorage
  const stored = localStorage.getItem("deliveryRecords");

  const upbjjList = JSON.parse(localStorage.getItem("upbjjList") || "[]");
  const upbjjNames = upbjjList.map((upbjj) => upbjj.nama);

  const pengirimanList = JSON.parse(
    localStorage.getItem("pengirimanList") || "[]",
  );
  const pengirimanNames = pengirimanList.map((p) => p.namaPenerima);

  const kategoriList = JSON.parse(localStorage.getItem("kategoriList") || "[]");
  const kategoriNames = kategoriList.map((k) => k.namaKategori);

  if (stored) {
    deliveryRecords.value = JSON.parse(stored);
    deliveryRecords.value.forEach((record) => {
      const savedProgress = Array.isArray(record.progress)
        ? record.progress
        : [
            {
              waktu: new Date().toLocaleString("id-ID"),
              keterangan: `Data pengiriman ditambahkan oleh ${record.namaPenerima}`,
            },
          ];

      allTrackingData.value[record.nomorDO] = {
        nomorDO: record.nomorDO,
        nama: record.namaPenerima,
        nim: record.nim,
        status: record.status,
        ekspedisi: record.ekspedisi,
        tanggalKirim: record.tanggal,
        paket: record.bahanAjar,
        total: record.total || "Rp 0,-",
        perjalanan: savedProgress,
      };
    });
  }
});

const handleLogout = () => {
  emit("logout");
};

const handleNavigate = (page) => {
  emit("navigate", page);
};

const trackingNumber = ref("");
const trackingResult = ref(null);
const handleTracking = () => {
  const key = trackingNumber.value.trim();
  const result = allTrackingData.value[key];
  if (result) {
    trackingResult.value = result;
  } else {
    Swal.fire({
      title: "Tracking Gagal!",
      text: "Nomor D O tidak ditemukan. Silahkan coba lagi...",
      icon: "error",
      confirmButtonColor: "#1e3a8a",
      confirmButtonText: "OK",
    });
    trackingResult.value = null;
  }
};

const resetTrackingSearch = () => {
  trackingNumber.value = "";
  trackingResult.value = null;
  progressDescription.value = "";
};

const handleEsc = () => {
  if (showInputModal.value) {
    showInputModal.value = false;
    return;
  }
  resetTrackingSearch();
};

const addTrackingProgress = () => {
  const description = progressDescription.value.trim();
  if (!trackingResult.value) {
    Swal.fire({
      title: "Tidak ada data tracking",
      text: "Silakan cari nomor DO terlebih dahulu.",
      icon: "warning",
      confirmButtonColor: "#1e3a8a",
      confirmButtonText: "OK",
    });
    return;
  }
  if (!description) {
    Swal.fire({
      title: "Keterangan kosong",
      text: "Masukkan keterangan progress sebelum menambah.",
      icon: "warning",
      confirmButtonColor: "#1e3a8a",
      confirmButtonText: "OK",
    });
    return;
  }

  const newProgress = {
    waktu: new Date().toLocaleString("id-ID"),
    keterangan: description,
  };

  trackingResult.value.perjalanan.push(newProgress);
  allTrackingData.value[trackingResult.value.nomorDO].perjalanan =
    trackingResult.value.perjalanan;

  if (deliveryRecords.value.length > 0) {
    const index = deliveryRecords.value.findIndex(
      (record) => record.nomorDO === trackingResult.value.nomorDO,
    );
    if (index !== -1) {
      deliveryRecords.value[index].progress = trackingResult.value.perjalanan;
      localStorage.setItem(
        "deliveryRecords",
        JSON.stringify(deliveryRecords.value),
      );
    }
  }

  progressDescription.value = "";
  Swal.fire({
    title: "Berhasil!",
    text: "Progress perjalanan telah ditambahkan.",
    icon: "success",
    confirmButtonColor: "#1e3a8a",
    confirmButtonText: "OK",
  });
};

const getStatusColor = () => {
  if (!trackingResult.value) return "bg-gray-100 text-gray-800";
  const status = trackingResult.value.status.toLowerCase();
  if (status.includes("dalam perjalanan")) {
    return "bg-yellow-100 text-yellow-800 border border-yellow-300";
  }
  if (status.includes("dikirim")) {
    return "bg-blue-100 text-blue-800 border border-blue-300";
  }
  return "bg-green-100 text-green-800 border border-green-300";
};

const handleSaveSuccess = (record) => {
  if (record && record.nomorDO) {
    const newTracking = {
      nomorDO: record.nomorDO,
      nama: record.namaPenerima,
      nim: record.nim,
      status: record.status,
      ekspedisi: record.ekspedisi,
      tanggalKirim: record.tanggal,
      paket: record.bahanAjar,
      total: record.total || "Rp 0,-",
      perjalanan: [
        {
          waktu: new Date().toLocaleString("id-ID"),
          keterangan: `Data pengiriman ditambahkan oleh ${record.namaPenerima}`,
        },
      ],
    };

    allTrackingData.value[record.nomorDO] = newTracking;
    trackingNumber.value = record.nomorDO;
    trackingResult.value = newTracking;
  }

  showInputModal.value = false;
};
</script>
<template>
  <section
    class="w-full min-h-screen relative overflow-hidden justify-center items-center flex flex-1 flex-col"
    @keydown.esc.window="handleEsc"
  >
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
    <Header
      :activePage="'Tracking'"
      @logout="handleLogout"
      @navigate="handleNavigate"
    />

    <!-- Form Pencarian Start -->
    <div class="mt-20 w-125 bg-gray-100 p-10 rounded-lg shadow-lg">
      <div
        class="bg-orange-100 w-16 h-16 rounded-full flex items-center justify-center mx-auto mb-4 text-orange-500"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          height="30px"
          viewBox="0 -960 960 960"
          width="30px"
          fill="#F19E39"
        >
          <path
            d="M784-120 532-372q-30 24-69 38t-83 14q-109 0-184.5-75.5T120-580q0-109 75.5-184.5T380-840q109 0 184.5 75.5T640-580q0 44-14 83t-38 69l252 252-56 56ZM380-400q75 0 127.5-52.5T560-580q0-75-52.5-127.5T380-760q-75 0-127.5 52.5T200-580q0 75 52.5 127.5T380-400Z"
          />
        </svg>
      </div>
      <h1 class="font-bold text-center text-4xl">Tracking Pengiriman</h1>

      <p class="text-center py-3">
        Fitur ini memungkinkan pengguna untuk melacak status pengiriman bahan
        ajar secara real-time, memberikan informasi tentang lokasi dan perkiraan
        waktu tiba.
      </p>
      <form class="w-full" @submit.prevent="handleTracking">
        <div class="flex gap-2 mt-5">
          <div class="flex border rounded-2xl w-full items-center gap-2">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              height="50px"
              viewBox="0 -960 960 960"
              width="50px"
              fill="#666666"
            >
              <path
                d="M440-183v-274L200-596v274l240 139Zm80 0 240-139v-274L520-457v274Zm-80 92L160-252q-19-11-29.5-29T120-321v-318q0-22 10.5-40t29.5-29l280-161q19-11 40-11t40 11l280 161q19 11 29.5 29t10.5 40v318q0 22-10.5 40T800-252L520-91q-19 11-40 11t-40-11Zm200-528 77-44-237-137-78 45 238 136Zm-160 93 78-45-237-137-78 45 237 137Z"
              />
            </svg>
            <input
              type="text"
              v-model="trackingNumber"
              class="w-full py-3 focus:outline-none"
              name="tracking_number"
              placeholder="Masukkan Nomor D O, misal : xxx.xxx.xxx"
            />
          </div>

          <button
            id="findBtn"
            type="button"
            @click="handleTracking"
            class="bg-blue-500 text-white p-3 rounded-2xl hover:bg-blue-600 cursor-pointer transition-colors"
          >
            Lacak
          </button>
        </div>
      </form>
    </div>
    <!-- Form Pencarian End -->

    <!-- Hasil Pencarian Start -->
    <div v-if="trackingResult" class="max-w-3xl mx-auto">
      <div
        class="bg-white rounded-2xl shadow-xl p-8 border border-gray-100 mt-6 animate-[fadeIn_0.5s_ease-out]"
      >
        <div
          class="flex flex-col md:flex-row justify-between items-start md:items-center border-b border-gray-100 pb-5 mb-6 gap-4"
        >
          <div>
            <h3 class="text-xl font-bold text-blue-900">Detail Pengiriman</h3>
            <p class="text-sm text-gray-500 mt-1">
              Nomor DO:
              <span class="font-bold text-gray-800">{{
                trackingResult.nomorDO
              }}</span>
            </p>
          </div>
          <span
            :class="`px-4 py-1.5 rounded-full text-sm font-semibold ${getStatusColor()}`"
            >{{ trackingResult.status }}</span
          >
        </div>
        <div
          class="grid grid-cols-1 md:grid-cols-2 gap-y-4 gap-x-8 mb-8 bg-gray-50 p-6 rounded-xl border border-gray-100"
        >
          <div class="flex flex-col">
            <span class="text-xs text-gray-500 mb-1 uppercase font-semibold"
              >Penerima</span
            ><span class="text-sm font-medium text-gray-800">{{
              trackingResult.nama
            }}</span>
          </div>
          <div class="flex flex-col">
            <span class="text-xs text-gray-500 mb-1 uppercase font-semibold"
              >Ekspedisi</span
            ><span class="text-sm font-medium text-gray-800"
              ><i class="fa-solid fa-truck text-orange-500 mr-2"></i
              >{{ trackingResult.ekspedisi }}</span
            >
          </div>
          <div class="flex flex-col">
            <span class="text-xs text-gray-500 mb-1 uppercase font-semibold"
              >Tanggal Kirim</span
            ><span class="text-sm font-medium text-gray-800">{{
              trackingResult.tanggalKirim
            }}</span>
          </div>
          <div class="flex flex-col">
            <span class="text-xs text-gray-500 mb-1 uppercase font-semibold"
              >Kode Paket</span
            ><span class="text-sm font-medium text-gray-800">{{
              trackingResult.paket
            }}</span>
          </div>
          <div class="flex flex-col">
            <span class="text-xs text-gray-500 mb-1 uppercase font-semibold"
              >Total Biaya</span
            ><span class="text-sm font-medium text-gray-800">{{
              trackingResult.total
            }}</span>
          </div>
        </div>
        <h4
          class="text-lg font-bold text-blue-900 mb-4 flex items-center gap-2"
        >
          <i class="fa-solid fa-route text-orange-500"></i> Riwayat Perjalanan
        </h4>
        <ul class="space-y-4">
          <li
            v-for="(event, index) in trackingResult.perjalanan"
            :key="index"
            class="rounded-2xl border border-gray-200 bg-gray-50 p-4"
          >
            <div
              class="flex items-center justify-between gap-3 text-sm text-gray-500 mb-2"
            >
              <span>{{ event.waktu }}</span>
              <span class="font-semibold text-gray-700"
                >Tahap {{ index + 1 }}</span
              >
            </div>
            <p class="text-gray-800">{{ event.keterangan }}</p>
          </li>
        </ul>

        <div class="mt-8 bg-gray-50 border border-gray-200 rounded-2xl p-6">
          <h4 class="text-lg font-bold text-blue-900 mb-3">
            Tambah Status Progress Perjalanan
          </h4>
          <textarea
            v-model="progressDescription"
            rows="3"
            class="w-full border border-gray-300 rounded-xl px-4 py-3 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
            placeholder="Masukkan keterangan status pengiriman..."
          ></textarea>
          <button
            @click="addTrackingProgress"
            class="mt-4 bg-blue-500 text-white px-6 py-3 rounded-full hover:bg-blue-600 transition-colors font-semibold"
          >
            Tambah Progress
          </button>
          <p class="mt-3 text-xs text-gray-500">
            Tekan <span class="font-semibold">Esc</span> untuk reset pencarian /
            tutup modal.
          </p>
        </div>
      </div>
    </div>

    <!-- Hasil Pencarian end -->
    <!-- Input Delivery Order start -->
    <div
      v-else
      class="max-w-3xl mt-6 p-8 bg-white rounded-2xl shadow-lg border border-gray-100 text-center"
    >
      <h3 class="text-xl font-bold text-gray-800 mb-2">
        Ingin Menambah Data Pengiriman ?
      </h3>
      <p class="text-sm text-gray-500">
        Masukkan data baru untuk Pengiriman bahan ajar dengan mengklik tombol di
        bawah ini.
      </p>
      <button
        @click="showInputModal = true"
        class="mt-4 bg-blue-500 text-white px-6 py-2 rounded-full shadow hover:bg-blue-600 transition-colors cursor-pointer"
      >
        Tambah Data Pengiriman
      </button>
    </div>

    <!-- Modal Input Delivery Start -->
    <div
      v-if="showInputModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4"
      @click.self="showInputModal = false"
      @keydown.esc.window="showInputModal = false"
    >
      <div
        class="bg-white rounded-lg shadow-2xl w-full max-w-2xl max-h-[90vh] overflow-y-auto"
      >
        <div
          class="sticky top-0 bg-white border-b border-gray-200 p-6 flex justify-between items-center"
        >
          <h2 class="text-2xl font-bold text-gray-800">
            Tambah Data Pengiriman
          </h2>
          <button
            @click="showInputModal = false"
            class="text-gray-500 cursor-pointer hover:text-gray-700 text-2xl font-bold"
          >
            ×
          </button>
        </div>
        <div class="p-6">
          <InputDelivery @save-success="handleSaveSuccess" />
        </div>
      </div>
    </div>
    <!-- Modal Input Delivery End -->
  </section>
</template>
