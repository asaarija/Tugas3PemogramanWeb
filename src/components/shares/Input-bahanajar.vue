<script setup>
import { ref } from "vue";
import Swal from "sweetalert2";
import dataBahanAjar from "../../contains/dataBahanAjar.json";

const apps = { ...dataBahanAjar };
const emit = defineEmits(["save-success"]);

const newStock = ref({
  kode: "",
  judul: "",
  kategori: apps.kategoriList[0] || "",
  upbjj: apps.upbjjList[0] || "",
  lokasiRak: "",
  harga: 0,
  qty: 0,
  safety: 0,
  catatanHTML: "",
});

const resetForm = () => {
  newStock.value = {
    kode: "",
    judul: "",
    kategori: apps.kategoriList[0] || "",
    upbjj: apps.upbjjList[0] || "",
    lokasiRak: "",
    harga: 0,
    qty: 0,
    safety: 0,
    catatanHTML: "",
  };
};

const handleSubmit = (event) => {
  event.preventDefault();
  if (!newStock.value.kode || !newStock.value.judul) {
    Swal.fire({
      icon: "error",
      title: "Data belum lengkap",
      text: "Harap isi kode dan judul bahan ajar sebelum menyimpan.",
      confirmButtonColor: "#3085d6",
    });
    return;
  }

  emit("save-success", { ...newStock.value });
  resetForm();
};
</script>
<template>
  <!-- add new start -->
  <form
    id="stokForm"
    class="w-full shadow-lg p-3 rounded"
    @submit.prevent="handleSubmit"
  >
    <div class="flex gap-2">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        height="24px"
        viewBox="0 -960 960 960"
        width="24px"
        fill="#F19E39"
      >
        <path
          d="M440-280h80v-160h160v-80H520v-160h-80v160H280v80h160v160Zm40 200q-83 0-156-31.5T197-197q-54-54-85.5-127T80-480q0-83 31.5-156T197-763q54-54 127-85.5T480-880q83 0 156 31.5T763-763q54 54 85.5 127T880-480q0 83-31.5 156T763-197q-54 54-127 85.5T480-80Zm0-80q134 0 227-93t93-227q0-134-93-227t-227-93q-134 0-227 93t-93 227q0 134 93 227t227 93Zm0-320Z"
        />
      </svg>
      <h1 class="font-bold">Tambahkan Bahan Ajar</h1>
    </div>
    <div class="pt-5">
      <div
        class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 items-end"
      >
        <div>
          <label
            for="kode"
            class="block text-xs font-semibold text-gray-500 mb-1"
            >Kode Bahan Ajar</label
          >
          <input
            type="text"
            id="kode"
            v-model="newStock.kode"
            placeholder="Misal: EKMA4116"
            required
            class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm"
          />
        </div>

        <div class="lg:col-span-2">
          <label
            for="nama"
            class="block text-xs font-semibold text-gray-500 mb-1"
            >Nama Bahan Ajar</label
          >
          <input
            type="text"
            id="nama"
            v-model="newStock.judul"
            placeholder="Misal: Pemrograman Berbasis Website"
            required
            class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm"
          />
        </div>

        <div>
          <label
            for="stok"
            class="block text-xs font-semibold text-gray-500 mb-1"
            >Jumlah Stok</label
          >
          <input
            type="number"
            id="stok"
            v-model.number="newStock.qty"
            min="0"
            required
            class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm"
          />
        </div>
      </div>
      <div
        class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 items-end"
      >
        <div>
          <label
            for="kategori"
            class="block text-xs font-semibold text-gray-500 mb-1"
            >Kategori</label
          >
          <select
            id="kategori"
            v-model="newStock.kategori"
            class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm"
          >
            <option
              v-for="kategori in apps.kategoriList"
              :key="kategori"
              :value="kategori"
            >
              {{ kategori }}
            </option>
          </select>
        </div>

        <div>
          <label
            for="upbjj"
            class="block text-xs font-semibold text-gray-500 mb-1"
            >UPBJJ</label
          >
          <select
            id="upbjj"
            v-model="newStock.upbjj"
            class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm"
          >
            <option v-for="upbjj in apps.upbjjList" :key="upbjj" :value="upbjj">
              {{ upbjj }}
            </option>
          </select>
        </div>

        <div>
          <label
            for="harga"
            class="block text-xs font-semibold text-gray-500 mb-1"
            >Harga</label
          >
          <input
            type="number"
            id="harga"
            v-model.number="newStock.harga"
            min="0"
            class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm"
          />
        </div>

        <div>
          <label
            for="safety"
            class="block text-xs font-semibold text-gray-500 mb-1"
            >Safety Stock</label
          >
          <input
            type="number"
            id="safety"
            v-model.number="newStock.safety"
            min="0"
            class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm"
          />
        </div>
      </div>

      <div class="grid grid-cols-1 gap-4 mt-4">
        <div>
          <label
            for="lokasi"
            class="block text-xs font-semibold text-gray-500 mb-1"
            >Lokasi Rak/Gudang</label
          >
          <input
            type="text"
            id="lokasi"
            v-model="newStock.lokasiRak"
            placeholder="Misal: Rak A1, Gudang 3"
            class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm"
          />
        </div>

        <div>
          <label
            for="catatan"
            class="block text-xs font-semibold text-gray-500 mb-1"
            >Catatan</label
          >
          <textarea
            id="catatan"
            v-model="newStock.catatanHTML"
            rows="3"
            placeholder="Catatan tambahan, misal: Edisi terbaru"
            class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm"
          ></textarea>
        </div>
      </div>

      <div
        class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 items-end mt-4"
      >
        <div>
          <button
            type="submit"
            class="w-full bg-yellow-500 hover:bg-yellow-400 text-white font-bold py-2.5 px-4 rounded-lg shadow transition-all transform hover:-translate-y-0.5 flex items-center justify-center gap-2 text-sm"
          >
            <i class="fa-solid fa-save"></i> Simpan Data
          </button>
        </div>
      </div>
    </div>
  </form>
  <!-- add new end -->
</template>
