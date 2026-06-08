<script setup>
import { ref, computed, onMounted } from "vue";
import Swal from "sweetalert2";
import Header from "./shares/Header.vue";
import InputNewBhnAjar from "./shares/Input-bahanajar.vue";
import dataBahanAjar from "../contains/dataBahanAjar.json";

const apps = { ...dataBahanAjar };
const emit = defineEmits(["logout", "navigate"]);
const showAddModal = ref(false);
const selectedStockFilter = ref("all");
const selectedUPBJJ = ref("");
const selectedKategori = ref("");
const searchQuery = ref("");
const stokData = ref([]);

const handleLogout = () => {
  emit("logout");
};

const handleNavigate = (page) => {
  emit("navigate", page);
};

const loadStockFromLocalStorage = () => {
  // Cek localStorage terlebih dahulu
  const stored = localStorage.getItem("stok");
  if (stored) {
    try {
      const parsed = JSON.parse(stored);
      if (Array.isArray(parsed) && parsed.length > 0) {
        stokData.value = parsed;
        return;
      }
    } catch (error) {
      console.error("Error parsing localStorage stok:", error);
    }
  }

  // Fallback ke apps.stok dari dataBahanAjar.json
  if (Array.isArray(apps.stok) && apps.stok.length > 0) {
    stokData.value = [...apps.stok];
  } else {
    stokData.value = [];
  }
};

onMounted(() => {
  loadStockFromLocalStorage();
});

const filteredStocks = computed(() => {
  return stokData.value.filter((item) => {
    const statusMatch =
      selectedStockFilter.value === "all" ||
      (selectedStockFilter.value === "stokKosong" && item.qty === 0) ||
      (selectedStockFilter.value === "stokTidakAman" &&
        item.qty > 0 &&
        item.qty <= item.safety) ||
      (selectedStockFilter.value === "stokAman" && item.qty >= item.safety);

    const upbjjMatch =
      !selectedUPBJJ.value || item.upbjj === selectedUPBJJ.value;

    const kategoriMatch =
      !selectedKategori.value || item.kategori === selectedKategori.value;

    const query = searchQuery.value.trim().toLowerCase();
    const searchMatch =
      !query ||
      item.kode.toLowerCase().includes(query) ||
      item.judul.toLowerCase().includes(query) ||
      item.kategori.toLowerCase().includes(query) ||
      item.upbjj.toLowerCase().includes(query) ||
      item.lokasiRak.toLowerCase().includes(query);

    return statusMatch && upbjjMatch && kategoriMatch && searchMatch;
  });
});

const saveStockToLocalStorage = () => {
  localStorage.setItem("stok", JSON.stringify(stokData.value));
  apps.stok = stokData.value;
};

const handleEditStock = (index) => {
  const item = filteredStocks.value[index];
  if (!item) return;

  Swal.fire({
    title: "Edit Stok",
    html: `
      <input type="number" id="qty" class="swal2-input" placeholder="Jumlah Stok" value="${item.qty}">
      <input type="number" id="safety" class="swal2-input" placeholder="Safety Stock" value="${item.safety}">
    `,
    focusConfirm: false,
    showCancelButton: true,
    confirmButtonText: "Simpan",
    cancelButtonText: "Batal",
    preConfirm: () => {
      const qty = parseInt(document.getElementById("qty").value);
      const safety = parseInt(document.getElementById("safety").value);
      if (isNaN(qty) || isNaN(safety)) {
        Swal.showValidationMessage("Harap masukkan angka yang valid");
        return false;
      }
      return { qty, safety };
    },
  }).then((result) => {
    if (result.isConfirmed) {
      const { qty, safety } = result.value;
      const globalIndex = stokData.value.findIndex(
        (stokItem) => stokItem.kode === item.kode,
      );
      if (globalIndex !== -1) {
        stokData.value[globalIndex].qty = qty;
        stokData.value[globalIndex].safety = safety;
        saveStockToLocalStorage();

        Swal.fire({
          icon: "success",
          title: "Berhasil!",
          text: "Stok berhasil diperbarui.",
        });
      }
    }
  });
};

const formatHarga = (value) => {
  return new Intl.NumberFormat("id-ID", {
    style: "currency",
    currency: "IDR",
  }).format(value);
};

const handleHapusStock = (index) => {
  const item = filteredStocks.value[index];
  if (!item) return;

  Swal.fire({
    title: "Apakah Anda yakin?",
    text: "Data stok akan dihapus secara permanen.",
    icon: "warning",
    showCancelButton: true,
    confirmButtonColor: "#d33",
    cancelButtonColor: "#3085d6",
    confirmButtonText: "Ya, hapus!",
    cancelButtonText: "Batal",
  }).then((result) => {
    if (result.isConfirmed) {
      stokData.value = stokData.value.filter(
        (stokItem) => stokItem.kode !== item.kode,
      );
      saveStockToLocalStorage();

      Swal.fire({
        icon: "success",
        title: "Dihapus!",
        text: "Data stok berhasil dihapus.",
      });
    }
  });
};

const handleAddNew = () => {
  showAddModal.value = true;
};

const handleSaveSuccess = (newItem) => {
  stokData.value = [...stokData.value, newItem];
  saveStockToLocalStorage();
  showAddModal.value = false;

  Swal.fire({
    icon: "success",
    title: "Berhasil!",
    text: "Stok bahan ajar berhasil ditambahkan.",
    confirmButtonColor: "#3085d6",
  });
};

const handleEditData = (index) => {
  const item = filteredStocks.value[index];
  if (!item) return;
  // Build plain HTML option lists because Vue directives won't work inside SweetAlert2 HTML string
  const kategoriOptions = apps.kategoriList
    .map(
      (k) =>
        `<option value="${k}" ${k === item.kategori ? "selected" : ""}>${k}</option>`,
    )
    .join("");

  const upbjjOptions = apps.upbjjList
    .map(
      (u) =>
        `<option value="${u}" ${u === item.upbjj ? "selected" : ""}>${u}</option>`,
    )
    .join("");

  Swal.fire({
    title: "Edit Data Bahan Ajar",
    html: `
      <div class="pt-5">
        <div class="flex gap-4 items-end">
          <div>
            <label for="kode" class="block text-xs font-semibold text-gray-500 mb-1">Kode Bahan Ajar</label>
            <input type="text" id="kode" required value="${item.kode}"
              class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm" />
          </div>

          <div class="lg:col-span-2 w-full">
            <label for="judul" class="block text-xs font-semibold text-gray-500 mb-1">Nama Bahan Ajar</label>
            <input type="text" id="judul" value="${item.judul}" required
              class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm" />
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
            value="${item.harga}"
            class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm"
          />
        
            </div>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-2 gap-4 items-end mt-4">
          <div>
            <label for="kategori" class="block text-xs font-semibold text-gray-500 mb-1">Kategori</label>
            <select id="kategori" class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm">
              ${kategoriOptions}
            </select>
          </div>

          <div>
            <label for="upbjj" class="block text-xs font-semibold text-gray-500 mb-1">UPBJJ</label>
            <select id="upbjj" class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm">
              ${upbjjOptions}
            </select>
          </div>
        </div>

        <div class="grid grid-cols-1 gap-4 mt-4">
          <div>
            <label for="lokasiRak" class="block text-xs font-semibold text-gray-500 mb-1">Lokasi Rak/Gudang</label>
            <input type="text" id="lokasiRak" value="${item.lokasiRak}"
              class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm" />
          </div>

          <div>
            <label for="catatanHTML" class="block text-xs font-semibold text-gray-500 mb-1">Catatan</label>
            <textarea id="catatanHTML" rows="3" placeholder="Catatan tambahan, misal: Edisi terbaru"
              class="w-full px-4 py-2.5 rounded-lg border border-gray-300 bg-gray-50 focus:bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-sm">${item.catatanHTML || ""}</textarea>
          </div>
        </div>
      </div>
    `,
    focusConfirm: false,
    showCancelButton: true,
    confirmButtonText: "Simpan",
    cancelButtonText: "Batal",
    preConfirm: () => {
      const judul = document.getElementById("judul").value.trim();
      const lokasiRak = document.getElementById("lokasiRak").value.trim();
      const catatanHTML = document.getElementById("catatanHTML").value.trim();
      const kategori = document.getElementById("kategori").value;
      const upbjj = document.getElementById("upbjj").value;
      const hargaRaw = document.getElementById("harga").value;
      const harga =
        hargaRaw === undefined || hargaRaw === null || hargaRaw === ""
          ? null
          : parseFloat(hargaRaw);

      if (!judul) {
        Swal.showValidationMessage("Judul tidak boleh kosong");
        return false;
      }

      if (harga !== null && isNaN(harga)) {
        Swal.showValidationMessage("Harga harus berupa angka");
        return false;
      }

      return { judul, lokasiRak, catatanHTML, kategori, upbjj, harga };
    },
  }).then((result) => {
    if (result.isConfirmed) {
      const { judul, lokasiRak, catatanHTML, kategori, upbjj, harga } =
        result.value;
      const globalIndex = stokData.value.findIndex(
        (stokItem) => stokItem.kode === item.kode,
      );
      if (globalIndex !== -1) {
        stokData.value[globalIndex].judul = judul;
        stokData.value[globalIndex].lokasiRak = lokasiRak;
        stokData.value[globalIndex].catatanHTML = catatanHTML;
        stokData.value[globalIndex].kategori = kategori;
        stokData.value[globalIndex].upbjj = upbjj;
        if (harga !== null) stokData.value[globalIndex].harga = harga;
        saveStockToLocalStorage();

        Swal.fire({
          icon: "success",
          title: "Berhasil!",
          text: "Data bahan ajar berhasil diperbarui.",
        });
      }
    }
  });
};
</script>
<template>
  <Header
    :activePage="'Stock'"
    @logout="handleLogout"
    @navigate="handleNavigate"
  />

  <!-- stock starts -->
  <section class="w-full pt-37.5 px-10 py-5 space-y-6">
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
    <!-- data bahan ajar start -->
    <div class="shadow-lg p-6 rounded-lg bg-white">
      <!-- filtered starts -->
      <div class="flex flex-col md:flex-row md:items-center gap-2 py-4">
        <select
          v-model="selectedStockFilter"
          class="rounded-lg border px-3 py-2 bg-white cursor-pointer border-gray hover:border-gray-300 transition-colors"
        >
          <option value="all">Semua Stok</option>
          <option value="stokKosong">Stok Kosong</option>
          <option value="stokTidakAman">Stok tidak Aman</option>
          <option value="stokAman">Stok aman</option>
        </select>

        <select
          v-model="selectedUPBJJ"
          class="rounded-lg border px-3 py-2 bg-white cursor-pointer border-gray hover:border-gray-300 transition-colors"
        >
          <option value="">Semua UPBJJ</option>
          <option v-for="upbjj in apps.upbjjList" :key="upbjj" :value="upbjj">
            {{ upbjj }}
          </option>
        </select>

        <select
          v-model="selectedKategori"
          class="rounded-lg border px-3 py-2 bg-[#0099ff] cursor-pointer border-[#0099ff] hover:border-gray-300 transition-colors"
        >
          <option value="">Semua Kategori</option>
          <option
            v-for="kategori in apps.kategoriList"
            :key="kategori"
            :value="kategori"
          >
            {{ kategori }}
          </option>
        </select>

        <!-- Tambahkan Stock Button starts -->
        <button
          class="bg-yellow-400 px-4 py-2 rounded-md hover:bg-blue-600 transition-colors font-semibold cursor-pointer flex items-center gap-2"
          @click="handleAddNew"
        >
          <i class="fa-solid fa-plus">+</i>
          <h1>Tambah Stock</h1>
        </button>
      </div>
      <!-- Tambahkan Stock Button ends -->
      <!-- filtered end -->
      <h2 class="text-2xl font-bold mb-4">Rekap Stok Bahan Ajar</h2>
      <div class="overflow-x-auto">
        <table class="min-w-full border border-gray-200 text-sm">
          <thead class="bg-gray-100 text-left">
            <tr>
              <th class="p-3 border-b">Kode Lokasi</th>
              <th class="p-3 border-b">Judul</th>
              <th class="p-3 border-b">Kategori</th>
              <th class="p-3 border-b">UPBJJ</th>
              <th class="p-3 border-b">Lokasi Rak</th>
              <th class="p-3 border-b">Harga</th>
              <th class="p-3 border-b">Stok</th>
              <th class="p-3 border-b">Safety Stock</th>
              <th class="p-3 border-b">Catatan HTML</th>
              <th class="p-3 border-b">Status</th>
              <th class="p-3 border-b">Aksi</th>
            </tr>
          </thead>
          <tbody id="stockTableBody">
            <tr
              v-for="(item, index) in filteredStocks"
              :key="item.kode + index"
              class="border-b hover:bg-gray-50"
            >
              <td class="p-3">{{ item.kode }}</td>
              <td class="p-3">{{ item.judul }}</td>
              <td class="p-3">{{ item.kategori }}</td>
              <td class="p-3">{{ item.upbjj }}</td>
              <td class="p-3">{{ item.lokasiRak }}</td>
              <td class="p-3" @input="formatHarga">
                {{ formatHarga(item.harga) }}
              </td>
              <td class="p-3">{{ item.qty }}</td>
              <td class="p-3">{{ item.safety }}</td>
              <td class="p-3" v-html="item.catatanHTML"></td>
              <td class="p-3">
                <span class="flex items-center gap-1">
                  <!-- Kosong -->
                  <template
                    v-if="item.qty === 0"
                    class="bg-amber-400 rounded p-2"
                  >
                    <svg
                      xmlns="http://www.w3.org/2000/svg"
                      height="24px"
                      viewBox="0 -960 960 960"
                      width="24px"
                      fill="#EA3323"
                    >
                      <path
                        d="M330-120 120-330v-300l210-210h300l210 210v300L630-120H330Zm36-190 114-114 114 114 56-56-114-114 114-114-56-56-114 114-114-114-56 56 114 114-114 114 56 56Zm-2 110h232l164-164v-232L596-760H364L200-596v232l164 164Zm116-280Z"
                      />
                    </svg>
                  </template>

                  <!-- Tidak safety -->
                  <template v-else-if="item.qty <= item.safety">
                    <svg
                      xmlns="http://www.w3.org/2000/svg"
                      height="24px"
                      viewBox="0 -960 960 960"
                      width="24px"
                      fill="#F19E39"
                    >
                      <path
                        d="m40-120 440-760 440 760H40Zm138-80h604L480-720 178-200Zm330.5-51.5Q520-263 520-280t-11.5-28.5Q497-320 480-320t-28.5 11.5Q440-297 440-280t11.5 28.5Q463-240 480-240t28.5-11.5ZM440-360h80v-200h-80v200Zm40-100Z"
                      />
                    </svg>
                  </template>

                  <!-- Aman -->
                  <template v-else>
                    <svg
                      xmlns="http://www.w3.org/2000/svg"
                      height="24"
                      viewBox="0 -960 960 960"
                      width="24"
                      fill="#75FB4C"
                    >
                      <path
                        d="m424-296 282-282-56-56-226 226-114-114-56 56 170 170Zm56 216q-83 0-156-31.5T197-197q-54-54-85.5-127T80-480q0-83 31.5-156T197-763q54-54 127-85.5T480-880q83 0 156 31.5T763-763q54 54 85.5 127T880-480q0 83-31.5 156T763-197q-54 54-127 85.5T480-80Z"
                      />
                    </svg>
                  </template>
                </span>
              </td>

              <td class="p-3 flex items-center gap-2">
                <button
                  class="bg-blue-500 text-white px-4 py-2 rounded-md hover:bg-blue-600 transition-colors font-semibold cursor-pointer"
                  @click="() => handleEditStock(index)"
                >
                  Edit Stok
                </button>
                <button
                  class="text-white px-4 rounded-md transition-colors font-semibold cursor-pointer"
                  @click="handleHapusStock(index)"
                >
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    height="24px"
                    viewBox="0 -960 960 960"
                    width="24px"
                    fill="#000000"
                  >
                    <path
                      d="M280-120q-33 0-56.5-23.5T200-200v-520h-40v-80h200v-40h240v40h200v80h-40v520q0 33-23.5 56.5T680-120H280Zm400-600H280v520h400v-520ZM360-280h80v-360h-80v360Zm160 0h80v-360h-80v360ZM280-720v520-520Z"
                    />
                  </svg>
                </button>
                <button
                  class="text-white rounded-md transition-colors font-semibold cursor-pointer"
                  @click="() => handleEditData(index)"
                >
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    height="24px"
                    viewBox="0 -960 960 960"
                    width="24px"
                    fill="#000000"
                  >
                    <path
                      d="M200-200h57l391-391-57-57-391 391v57Zm-80 80v-170l528-527q12-11 26.5-17t30.5-6q16 0 31 6t26 18l55 56q12 11 17.5 26t5.5 30q0 16-5.5 30.5T817-647L290-120H120Zm640-584-56-56 56 56Zm-141 85-28-29 57 57-29-28Z"
                    />
                  </svg>
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <!-- data bahan ajar end -->
    <!-- stock ends -->
    <div
      v-if="showAddModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4"
      @click.self="showAddModal = false"
    >
      <div
        class="bg-white rounded-lg shadow-2xl w-full max-w-3xl max-h-[90vh] overflow-y-auto"
      >
        <div
          class="sticky top-0 bg-white border-b border-gray-200 p-6 flex justify-between items-center"
        >
          <h2 class="text-2xl font-bold text-gray-800">
            Tambah Stock Bahan Ajar
          </h2>
          <button
            @click="showAddModal = false"
            class="text-gray-500 hover:text-gray-700 text-2xl font-bold"
          >
            ×
          </button>
        </div>
        <div class="p-6">
          <InputNewBhnAjar @save-success="handleSaveSuccess" />
        </div>
      </div>
    </div>
  </section>
</template>
