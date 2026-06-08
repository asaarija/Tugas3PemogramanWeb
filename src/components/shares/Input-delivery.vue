<script setup>
import { ref, computed, onMounted } from "vue";
import Swal from "sweetalert2";
import dataBahanAjar from "@/contains/dataBahanAjar.json";

const apps = { ...dataBahanAjar, deliveryRecords: [] };

const emit = defineEmits(["save-success"]);

const inputDONew = ref("");
const inputDateNew = ref("");
const inputNIMNew = ref("");
const inputNamaPenerimaNew = ref("");
const inputEkspedisiNew = ref("");
const selectPaketNew = ref("");

const deliveryRecords = ref([]);

const selectedPaket = computed(
  () => apps.paket.find((paket) => paket.kode === selectPaketNew.value) || null,
);

const formattedTotal = computed(() => {
  if (!selectedPaket.value) return "";
  return new Intl.NumberFormat("id-ID", {
    style: "currency",
    currency: "IDR",
    minimumFractionDigits: 0,
    maximumFractionDigits: 0,
  }).format(selectedPaket.value.harga);
});

onMounted(() => {
  // Load data dari localStorage
  const stored = localStorage.getItem("deliveryRecords");
  if (stored) {
    deliveryRecords.value = JSON.parse(stored);
    apps.deliveryRecords.length = 0;
    apps.deliveryRecords.push(...deliveryRecords.value);
  }
  // generate nomor DO awal
  generateNextDONumber();
});

const generateNextDONumber = () => {
  const year = new Date().getFullYear();
  const prefix = `DO${year}-`;
  let maxSeq = 0;

  // check existing delivery records loaded from localStorage
  deliveryRecords.value.forEach((r) => {
    if (typeof r.nomorDO === "string" && r.nomorDO.startsWith(prefix)) {
      const parts = r.nomorDO.split("-");
      const seq = parseInt(parts[1], 10);
      if (!isNaN(seq) && seq > maxSeq) maxSeq = seq;
    }
  });

  const next = maxSeq + 1;
  inputDONew.value = `${prefix}${String(next).padStart(3, "0")}`;
};

const handleSearch = () => {
  if (
    !inputDateNew.value ||
    !inputNIMNew.value.trim() ||
    !inputNamaPenerimaNew.value.trim() ||
    !inputEkspedisiNew.value ||
    !selectPaketNew.value
  ) {
    Swal.fire({
      title: "Form Belum Lengkap",
      text: "Silakan isi semua field yang diperlukan.",
      icon: "warning",
      confirmButtonColor: "#1e3a8a",
      confirmButtonText: "OK",
    });
    return;
  }

  const newRecord = {
    id: Date.now(),
    nomorDO: inputDONew.value,
    tanggal: inputDateNew.value,
    nim: inputNIMNew.value,
    namaPenerima: inputNamaPenerimaNew.value,
    ekspedisi: inputEkspedisiNew.value,
    bahanAjar: selectPaketNew.value,
    total: formattedTotal.value,
    status: "Dalam Proses",
  };

  deliveryRecords.value.push(newRecord);
  apps.deliveryRecords.length = 0;
  apps.deliveryRecords.push(...deliveryRecords.value);

  // Simpan ke localStorage
  localStorage.setItem(
    "deliveryRecords",
    JSON.stringify(deliveryRecords.value),
  );

  // Update stok di Stock - kurangi qty per item paket
  const storedStok = localStorage.getItem("stok");
  if (storedStok && selectedPaket.value) {
    try {
      const stokList = JSON.parse(storedStok);
      selectedPaket.value.isi.forEach((kode) => {
        const stokIndex = stokList.findIndex((item) => item.kode === kode);
        if (stokIndex !== -1) {
          stokList[stokIndex].qty = Math.max(0, stokList[stokIndex].qty - 1);
        }
      });
      localStorage.setItem("stok", JSON.stringify(stokList));
    } catch (error) {
      console.error("Error updating stock:", error);
    }
  }

  emit("save-success", newRecord);

  Swal.fire({
    title: "Berhasil!",
    text: "Data pengiriman telah ditambahkan dan stok telah diperbarui.",
    icon: "success",
    confirmButtonColor: "#1e3a8a",
    confirmButtonText: "OK",
  });

  // Reset form
  inputDateNew.value = "";
  inputNIMNew.value = "";
  inputNamaPenerimaNew.value = "";
  inputEkspedisiNew.value = "";
  selectPaketNew.value = "";

  // nomor DO otomatis akan digenerate untuk entri berikutnya
  generateNextDONumber();
};

const handleDelete = (id) => {
  Swal.fire({
    title: "Hapus Data?",
    text: "Anda yakin ingin menghapus data pengiriman ini?",
    icon: "warning",
    showCancelButton: true,
    confirmButtonColor: "#dc2626",
    cancelButtonColor: "#6b7280",
    confirmButtonText: "Ya, Hapus",
    cancelButtonText: "Batal",
  }).then((result) => {
    if (result.isConfirmed) {
      deliveryRecords.value = deliveryRecords.value.filter(
        (record) => record.id !== id,
      );
      // update localStorage and shared apps array
      localStorage.setItem(
        "deliveryRecords",
        JSON.stringify(deliveryRecords.value),
      );
      apps.deliveryRecords.length = 0;
      apps.deliveryRecords.push(...deliveryRecords.value);
      Swal.fire({
        title: "Terhapus!",
        text: "Data pengiriman telah dihapus.",
        icon: "success",
        confirmButtonColor: "#1e3a8a",
      });
    }
  });
};
</script>

<template>
  <div class="space-y-4">
    <div>
      <h1 class="text-2xl font-bold mb-4">Form Pengiriman Baru</h1>
    </div>

    <div class="bg-gray-50 rounded-lg border border-gray-200 p-6">
      <div class="space-y-4">
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div>
            <label
              for="nomorDO"
              class="block text-sm font-medium text-gray-700 mb-1"
              >Nomor DO:</label
            >
            <input
              v-model="inputDONew"
              type="text"
              id="nomorDO"
              readonly
              placeholder="Nomor DO akan digenerate otomatis"
              class="border bg-gray-100 border-gray-300 w-full rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>
          <div>
            <label
              for="tanggalDO"
              class="block text-sm font-medium text-gray-700 mb-1"
              >Tanggal DO:</label
            >
            <input
              v-model="inputDateNew"
              type="date"
              id="tanggalDO"
              class="border w-full border-gray-300 rounded-md px-4 py-2 focus:outline-none cursor-pointer focus:ring-2 focus:ring-blue-500"
            />
          </div>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-4 pt-3">
          <div>
            <label
              for="nim"
              class="block text-sm font-medium text-gray-700 mb-1"
              >NIM Penerima:</label
            >
            <input
              v-model="inputNIMNew"
              type="text"
              id="nim"
              placeholder="NIM Penerima"
              class="border border-gray-300 w-full rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>
          <div>
            <label
              for="namaPenerima"
              class="block text-sm font-medium text-gray-700 mb-1"
              >Nama Penerima:</label
            >
            <input
              v-model="inputNamaPenerimaNew"
              type="text"
              id="namaPenerima"
              placeholder="Nama Penerima"
              class="border border-gray-300 w-full rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-4 pt-3">
          <div>
            <label for="ekspedisi" class="block text-sm font-medium mb-1"
              >Ekspedisi:</label
            >
            <select
              v-model="inputEkspedisiNew"
              id="ekspedisi"
              class="border border-gray-300 w-full rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
            >
              <option value="" disabled>Pilih Ekspedisi</option>
              <option
                v-for="pengiriman in apps.pengirimanList"
                :key="pengiriman.kode"
                :value="pengiriman.kode"
              >
                {{ pengiriman.nama }} ({{ pengiriman.kode }})
              </option>
            </select>
          </div>
          <div>
            <label
              for="paket"
              class="block text-sm font-medium text-gray-700 mb-1"
              >Paket Bahan Ajar:</label
            >
            <select
              v-model="selectPaketNew"
              id="paket"
              class="border border-gray-300 w-full rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
            >
              <option value="" disabled>Pilih Paket Bahan Ajar</option>
              <option
                v-for="paket in apps.paket"
                :key="paket.kode"
                :value="paket.kode"
              >
                {{ paket.nama }} ({{ paket.kode }})
              </option>
            </select>
          </div>
        </div>
        <div class="py-3">
          <label
            for="totalHarga"
            class="block text-sm font-medium text-gray-700 mb-1"
            >Total Harga:</label
          >
          <input
            :value="formattedTotal"
            readonly
            id="totalHarga"
            placeholder="Total akan otomatis muncul setelah memilih paket"
            class="border border-gray-300 w-full rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500 bg-gray-100"
          />
        </div>

        <button
          @click="handleSearch"
          class="w-full bg-blue-600 mt-5 text-white px-6 py-2 rounded-md hover:bg-blue-700 transition-colors font-semibold cursor-pointer"
        >
          Simpan Data Pengiriman
        </button>
      </div>
    </div>
  </div>
</template>
