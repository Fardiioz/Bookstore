<template>
  <div class="page space-y-6">
    <!-- Header -->
    <div
      class="flex flex-col sm:flex-row sm:items-center justify-between gap-4"
    >
      <div>
        <h1 class="page-title">Manajemen Buku</h1>
        <p class="page-sub text-sm mt-1">
          Kelola koleksi buku toko. Nama buku harus unik.
        </p>
      </div>
      <button
        type="button"
        @click="openCreateModal"
        class="btn-primary inline-flex items-center gap-1.5 px-4 py-2.5 text-sm font-semibold whitespace-nowrap"
      >
        <LucidePlus class="w-4 h-4" />
        Tambah Buku
      </button>
    </div>

    <!-- Books Table -->
    <div class="table-wrap overflow-hidden">
      <div v-if="loading" class="state-text text-center py-14 text-sm">
        Memuat data buku...
      </div>
      <div
        v-else-if="books.length === 0"
        class="state-text text-center py-14 text-sm"
      >
        <LucideBookOpen class="w-8 h-8 mx-auto mb-3" :stroke-width="1.5" />
        Belum ada buku terdaftar.
      </div>
      <div v-else class="overflow-x-auto">
        <table class="tbl w-full text-left text-xs">
          <thead>
            <tr>
              <th class="px-4 py-3">Cover</th>
              <th class="px-4 py-3">Judul Buku</th>
              <th class="px-4 py-3">Kategori</th>
              <th class="px-4 py-3">Tgl / Thn Terbit</th>
              <th class="px-4 py-3 text-right">Stok</th>
              <th class="px-4 py-3 text-right">Harga Modal</th>
              <th class="px-4 py-3 text-right">Harga Jual</th>
              <th class="px-4 py-3 text-right">Keuntungan</th>
              <th class="px-4 py-3 text-right">Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="book in books" :key="book.id">
              <td class="px-4 py-3">
                <div
                  class="thumb w-10 h-12 overflow-hidden flex items-center justify-center shrink-0"
                >
                  <img
                    v-if="book.gambar"
                    :src="book.gambar"
                    :alt="book.nama_buku"
                    class="w-full h-full object-cover"
                  />
                  <LucideBookOpen v-else class="w-4 h-4 thumb-icon" />
                </div>
              </td>
              <td
                class="cell-strong px-4 py-3 font-semibold text-sm max-w-[160px] truncate"
                :title="book.nama_buku"
              >
                {{ book.nama_buku }}
              </td>
              <td class="px-4 py-3 whitespace-nowrap">
                <span
                  class="tag inline-block px-2 py-0.5 text-[11px] font-medium whitespace-nowrap"
                >
                  {{ book.category_name || "Buku" }}
                </span>
              </td>
              <td class="cell-muted px-4 py-3 whitespace-nowrap">
                {{ book.tanggal_terbit || "-" }}
                <span class="text-[11px]"> ({{ book.tahun_terbit }})</span>
              </td>
              <td
                class="px-4 py-3 text-right tabular-nums font-semibold"
                :class="book.stok > 0 ? 'cell-strong' : 'text-danger'"
              >
                {{ book.stok }}
              </td>
              <td
                class="cell-muted px-4 py-3 text-right tabular-nums whitespace-nowrap"
              >
                Rp {{ formatNumber(book.harga_modal) }}
              </td>
              <td
                class="cell-strong px-4 py-3 text-right tabular-nums font-semibold whitespace-nowrap"
              >
                Rp {{ formatNumber(book.harga_jual) }}
              </td>
              <td
                class="cell-strong px-4 py-3 text-right tabular-nums font-semibold whitespace-nowrap"
              >
                Rp {{ formatNumber(book.keuntungan) }}
              </td>
              <td class="px-4 py-3 text-right">
                <div class="flex items-center justify-end gap-1.5">
                  <button
                    type="button"
                    @click="openEditModal(book)"
                    class="icon-action"
                    title="Edit Buku"
                    aria-label="Edit buku"
                  >
                    <LucidePencil class="w-3.5 h-3.5" />
                  </button>
                  <button
                    type="button"
                    @click="deleteBook(book)"
                    class="icon-action icon-action--danger"
                    title="Hapus Buku"
                    aria-label="Hapus buku"
                  >
                    <LucideTrash2 class="w-3.5 h-3.5" />
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Modal Form -->
    <div
      v-if="showModal"
      class="modal-overlay fixed inset-0 z-50 flex items-center justify-center p-4"
    >
      <div
        class="modal p-6 max-w-xl w-full space-y-4 max-h-[90vh] overflow-y-auto"
        role="dialog"
        aria-modal="true"
      >
        <div class="flex items-center justify-between">
          <h3 class="modal-title text-lg">
            {{ isEdit ? "Edit Buku" : "Tambah Buku Baru" }}
          </h3>
          <button
            type="button"
            @click="showModal = false"
            class="icon-action"
            aria-label="Tutup"
          >
            <LucideX class="w-4 h-4" />
          </button>
        </div>

        <div v-if="errorMessage" class="alert-error p-3 text-xs font-medium">
          {{ errorMessage }}
        </div>

        <form @submit.prevent="saveBook" class="space-y-4">
          <div>
            <label class="label block text-xs font-semibold mb-1">
              Judul Buku <span class="req">*</span>
            </label>
            <input
              v-model="form.nama_buku"
              type="text"
              required
              class="field w-full px-4 py-2.5 text-sm"
              placeholder="Masukkan judul buku yang unik"
            />
          </div>

          <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
            <div>
              <label class="label block text-xs font-semibold mb-1"
                >Kategori <span class="req">*</span></label
              >
              <select
                v-model="form.category_id"
                required
                class="field w-full px-4 py-2.5 text-sm"
              >
                <option value="">-- Pilih Kategori --</option>
                <option v-for="cat in categories" :key="cat.id" :value="cat.id">
                  {{ cat.nama_kategori }}
                </option>
              </select>
            </div>

            <div>
              <label class="label block text-xs font-semibold mb-1"
                >Tanggal Terbit <span class="req">*</span></label
              >
              <input
                v-model="form.tanggal_terbit"
                type="date"
                required
                class="field w-full px-4 py-2.5 text-sm"
              />
              <span v-if="computedTahun" class="hint text-[11px] mt-1 block">
                Tahun terbit: {{ computedTahun }}
              </span>
            </div>
          </div>

          <div class="grid grid-cols-3 gap-4">
            <div>
              <label class="label block text-xs font-semibold mb-1">Stok</label>
              <input
                v-model.number="form.stok"
                type="number"
                min="0"
                required
                class="field w-full px-3 py-2.5 text-sm"
              />
            </div>
            <div>
              <label class="label block text-xs font-semibold mb-1"
                >Harga Modal</label
              >
              <input
                v-model.number="form.harga_modal"
                type="number"
                min="0"
                required
                class="field w-full px-3 py-2.5 text-sm"
              />
            </div>
            <div>
              <label class="label block text-xs font-semibold mb-1"
                >Harga Jual</label
              >
              <input
                v-model.number="form.harga_jual"
                type="number"
                min="0"
                required
                class="field w-full px-3 py-2.5 text-sm"
              />
            </div>
          </div>

          <!-- Profit Preview -->
          <div class="profit-box p-3 text-xs flex justify-between items-center">
            <span class="label font-medium"
              >Keuntungan otomatis (jual - modal)</span
            >
            <span class="profit-value text-sm font-semibold tabular-nums"
              >Rp {{ formatNumber(computedKeuntungan) }}</span
            >
          </div>

          <div>
            <label class="label block text-xs font-semibold mb-1"
              >Deskripsi Buku</label
            >
            <textarea
              v-model="form.deskripsi"
              rows="3"
              class="field w-full px-4 py-2.5 text-sm resize-none"
              placeholder="Ringkasan atau deskripsi buku..."
            />
          </div>

          <div>
            <label class="label block text-xs font-semibold mb-1"
              >Gambar / Cover Buku</label
            >
            <input
              @change="handleFileChange"
              type="file"
              accept="image/*"
              class="field-file w-full text-xs"
            />
          </div>

          <div class="modal-actions flex justify-end gap-3 pt-4">
            <button
              type="button"
              @click="showModal = false"
              class="btn-ghost px-4 py-2 text-sm font-semibold"
            >
              Batal
            </button>
            <button
              type="submit"
              :disabled="submitting"
              class="btn-primary px-5 py-2 text-sm font-semibold disabled:opacity-50 disabled:cursor-not-allowed"
            >
              {{ submitting ? "Menyimpan..." : "Simpan Buku" }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  Pencil as LucidePencil,
  Trash2 as LucideTrash2,
  Plus as LucidePlus,
  BookOpen as LucideBookOpen,
  X as LucideX,
} from "lucide-vue-next";

definePageMeta({
  middleware: "admin",
});

const api = useApi();
const books = ref<any[]>([]);
const categories = ref<any[]>([]);
const loading = ref(true);
const showModal = ref(false);
const isEdit = ref(false);
const editId = ref<number | null>(null);
const submitting = ref(false);
const errorMessage = ref("");
const selectedGambar = ref<File | null>(null);

const form = reactive({
  nama_buku: "",
  category_id: "",
  tanggal_terbit: "",
  stok: 0,
  harga_modal: 0,
  harga_jual: 0,
  deskripsi: "",
});

const formatNumber = (val: number) => {
  return new Intl.NumberFormat("id-ID").format(val || 0);
};

const computedKeuntungan = computed(() => {
  const profit =
    (Number(form.harga_jual) || 0) - (Number(form.harga_modal) || 0);
  return profit > 0 ? profit : 0;
});

const computedTahun = computed(() => {
  if (!form.tanggal_terbit) return "";
  return new Date(form.tanggal_terbit).getFullYear();
});

const fetchBooks = async () => {
  loading.value = true;
  try {
    const res = await api.get("/api/books");
    books.value = res.data || [];
  } catch (e) {
    console.error(e);
  } finally {
    loading.value = false;
  }
};

const fetchCategories = async () => {
  try {
    const res = await api.get("/api/categories");
    categories.value = res.data || [];
  } catch (e) {}
};

const handleFileChange = (e: Event) => {
  const target = e.target as HTMLInputElement;
  if (target.files && target.files[0]) {
    selectedGambar.value = target.files[0];
  }
};

const openCreateModal = () => {
  isEdit.value = false;
  editId.value = null;
  form.nama_buku = "";
  form.category_id = "";
  form.tanggal_terbit = "";
  form.stok = 10;
  form.harga_modal = 50000;
  form.harga_jual = 75000;
  form.deskripsi = "";
  selectedGambar.value = null;
  errorMessage.value = "";
  showModal.value = true;
};

const openEditModal = (b: any) => {
  isEdit.value = true;
  editId.value = b.id;
  form.nama_buku = b.nama_buku;
  form.category_id = b.category_id;
  form.tanggal_terbit = b.tanggal_terbit;
  form.stok = b.stok;
  form.harga_modal = b.harga_modal;
  form.harga_jual = b.harga_jual;
  form.deskripsi = b.deskripsi || "";
  selectedGambar.value = null;
  errorMessage.value = "";
  showModal.value = true;
};

const saveBook = async () => {
  submitting.value = true;
  errorMessage.value = "";

  try {
    const formData = new FormData();
    formData.append("nama_buku", form.nama_buku);
    formData.append("category_id", String(form.category_id));
    formData.append("tanggal_terbit", form.tanggal_terbit);
    formData.append("stok", String(form.stok));
    formData.append("harga_modal", String(form.harga_modal));
    formData.append("harga_jual", String(form.harga_jual));
    formData.append("deskripsi", form.deskripsi);
    if (selectedGambar.value) {
      formData.append("gambar", selectedGambar.value);
    }

    if (isEdit.value && editId.value) {
      formData.append("_method", "PUT");
      await api.post(`/api/admin/books/${editId.value}`, formData);
    } else {
      await api.post("/api/admin/books", formData);
    }

    showModal.value = false;
    await fetchBooks();
  } catch (err: any) {
    errorMessage.value =
      err.data?.message ||
      err.data?.errors?.nama_buku?.[0] ||
      "Gagal menyimpan data buku.";
  } finally {
    submitting.value = false;
  }
};

const deleteBook = async (b: any) => {
  if (!confirm(`Yakin ingin menghapus buku '${b.nama_buku}'?`)) return;
  try {
    const toast = useToast();
    await api.delete(`/api/admin/books/${b.id}`);
    toast.success("Buku berhasil dihapus!");
    await fetchBooks();
  } catch (err: any) {
    const toast = useToast();
    toast.error(err.data?.message || "Gagal menghapus buku.");
  }
};

onMounted(async () => {
  await fetchCategories();
  await fetchBooks();
});
</script>

<style scoped>
/* Semua warna mengikuti token di main.css, jadi mode terang dan gelap sinkron */
.page {
  --danger: #b0394f;
  --success: #4d6b3f;
  color: var(--text);
}

:global(:root.dark) .page,
:global(:root[data-theme="dark"]) .page {
  --danger: #e68a9a;
  --success: #b3c0a4;
}

.page-title {
  font-family: var(--font-display);
  font-size: 1.5rem;
  font-weight: 600;
  letter-spacing: -0.01em;
  color: var(--text);
}

.page-sub,
.state-text,
.cell-muted,
.hint {
  color: var(--muted);
}

.cell-strong {
  color: var(--text);
}

.text-danger {
  color: var(--danger);
}

.req {
  color: var(--danger);
}

/* tombol */
.btn-primary {
  background-color: var(--primary);
  color: var(--on-primary);
  border: 1px solid var(--primary);
  border-radius: var(--radius-md);
  transition:
    background-color 0.15s ease,
    border-color 0.15s ease,
    transform 0.1s ease;
}

.btn-primary:hover:not(:disabled) {
  background-color: var(--primary-hover);
  border-color: var(--primary-hover);
}

.btn-primary:active:not(:disabled) {
  transform: translateY(1px);
}

.btn-ghost {
  color: var(--text-2);
  border-radius: var(--radius-md);
  transition:
    background-color 0.15s ease,
    color 0.15s ease;
}

.btn-ghost:hover {
  background-color: var(--inset);
  color: var(--text);
}

.icon-action {
  width: 2rem;
  height: 2rem;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  color: var(--text-2);
  background: transparent;
  transition:
    background-color 0.15s ease,
    border-color 0.15s ease,
    color 0.15s ease;
}

.icon-action:hover {
  background-color: var(--inset);
  border-color: var(--border-strong);
  color: var(--text);
}

.icon-action--danger:hover {
  color: var(--danger);
  border-color: color-mix(in srgb, var(--danger) 50%, transparent);
  background-color: color-mix(in srgb, var(--danger) 8%, transparent);
}

/* tabel */
.table-wrap {
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

.tbl thead tr {
  background-color: var(--inset);
  border-bottom: 1px solid var(--border);
}

.tbl th {
  color: var(--muted);
  font-weight: 600;
  white-space: nowrap;
}

.tbl tbody tr {
  border-bottom: 1px solid var(--border);
  transition: background-color 0.15s ease;
}

.tbl tbody tr:last-child {
  border-bottom: 0;
}

.tbl tbody tr:hover {
  background-color: var(--inset);
}

.thumb {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
}

.thumb-icon {
  color: var(--muted);
}

.tag {
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-sm);
  color: var(--text-2);
}

/* modal */
.modal-overlay {
  background-color: rgb(20 18 31 / 0.55);
}

.modal {
  background-color: var(--surface);
  color: var(--text);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

.modal-title {
  font-family: var(--font-display);
  font-weight: 600;
  color: var(--text);
}

.modal-actions {
  border-top: 1px solid var(--border);
}

.label {
  color: var(--text-2);
}

.field {
  background-color: var(--surface);
  color: var(--text);
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
  outline: none;
  transition: border-color 0.15s ease;
}

.field::placeholder {
  color: var(--muted);
}

.field:focus {
  border-color: var(--primary);
}

.field-file {
  color: var(--muted);
}

.field-file::file-selector-button {
  margin-right: 1rem;
  padding: 0.5rem 1rem;
  font-size: 0.75rem;
  font-weight: 600;
  color: var(--text);
  background-color: var(--inset);
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
  cursor: pointer;
}

.alert-error {
  color: var(--danger);
  border: 1px solid color-mix(in srgb, var(--danger) 45%, transparent);
  border-radius: var(--radius-md);
  background-color: color-mix(in srgb, var(--danger) 8%, transparent);
}

.profit-box {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
}

.profit-value {
  color: var(--success);
}
</style>
