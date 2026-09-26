<template>
  <div class="page space-y-6">
    <!-- Header -->
    <div
      class="flex flex-col sm:flex-row sm:items-center justify-between gap-4"
    >
      <div>
        <h1 class="page-title">Manajemen Kategori</h1>
        <p class="muted text-sm mt-1">
          Kelola kategori buku. Nama kategori harus unik.
        </p>
      </div>
      <button
        type="button"
        @click="openCreateModal"
        class="btn-primary inline-flex items-center gap-1.5 px-4 py-2.5 text-sm font-semibold whitespace-nowrap"
      >
        <LucidePlus class="w-4 h-4" />
        Tambah Kategori
      </button>
    </div>

    <!-- Table -->
    <div class="table-wrap overflow-hidden">
      <div v-if="loading" class="muted text-center py-14 text-sm">
        Memuat kategori...
      </div>
      <div
        v-else-if="categories.length === 0"
        class="muted text-center py-14 text-sm"
      >
        <LucideTag class="w-8 h-8 mx-auto mb-3" :stroke-width="1.5" />
        Belum ada kategori.
      </div>
      <div v-else class="overflow-x-auto">
        <table class="tbl w-full text-left text-xs">
          <thead>
            <tr>
              <th class="px-4 py-3">No</th>
              <th class="px-4 py-3">Nama Kategori</th>
              <th class="px-4 py-3">Jumlah Buku</th>
              <th class="px-4 py-3">Tanggal Dibuat</th>
              <th class="px-4 py-3 text-right">Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(cat, idx) in categories" :key="cat.id">
              <td class="muted px-4 py-3 tabular-nums">{{ idx + 1 }}</td>
              <td class="cell-strong px-4 py-3 font-semibold text-sm">
                {{ cat.nama_kategori }}
              </td>
              <td class="px-4 py-3">
                <span
                  class="tag inline-block px-2 py-0.5 text-[11px] font-medium tabular-nums"
                >
                  {{ cat.books_count || 0 }} Buku
                </span>
              </td>
              <td class="muted px-4 py-3">{{ cat.created_at }}</td>
              <td class="px-4 py-3 text-right">
                <div class="flex items-center justify-end gap-1.5">
                  <button
                    type="button"
                    @click="openEditModal(cat)"
                    class="icon-action"
                    title="Edit Kategori"
                    aria-label="Edit kategori"
                  >
                    <LucidePencil class="w-3.5 h-3.5" />
                  </button>
                  <button
                    type="button"
                    @click="deleteCategory(cat)"
                    class="icon-action icon-action--danger"
                    title="Hapus Kategori"
                    aria-label="Hapus kategori"
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
        class="modal p-6 max-w-md w-full space-y-4"
        role="dialog"
        aria-modal="true"
      >
        <div class="flex items-center justify-between">
          <h3 class="modal-title text-lg">
            {{ isEdit ? "Edit Kategori" : "Tambah Kategori" }}
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

        <form @submit.prevent="saveCategory" class="space-y-4">
          <div>
            <label class="label block text-xs font-semibold mb-1">
              Nama Kategori <span class="req">*</span>
            </label>
            <input
              v-model="form.nama_kategori"
              type="text"
              required
              class="field w-full px-4 py-2.5 text-sm"
              placeholder="Contoh: Novel Fiksi"
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
              {{ submitting ? "Menyimpan..." : "Simpan" }}
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
  Tag as LucideTag,
  X as LucideX,
} from "lucide-vue-next";

definePageMeta({
  middleware: "admin",
});

const api = useApi();
const categories = ref<any[]>([]);
const loading = ref(true);
const showModal = ref(false);
const isEdit = ref(false);
const editId = ref<number | null>(null);
const submitting = ref(false);
const errorMessage = ref("");

const form = reactive({
  nama_kategori: "",
});

const fetchCategories = async () => {
  loading.value = true;
  try {
    const res = await api.get("/api/categories");
    categories.value = res.data || [];
  } catch (e) {
    console.error(e);
  } finally {
    loading.value = false;
  }
};

const openCreateModal = () => {
  isEdit.value = false;
  editId.value = null;
  form.nama_kategori = "";
  errorMessage.value = "";
  showModal.value = true;
};

const openEditModal = (cat: any) => {
  isEdit.value = true;
  editId.value = cat.id;
  form.nama_kategori = cat.nama_kategori;
  errorMessage.value = "";
  showModal.value = true;
};

const saveCategory = async () => {
  submitting.value = true;
  errorMessage.value = "";

  try {
    if (isEdit.value && editId.value) {
      await api.put(`/api/admin/categories/${editId.value}`, form);
    } else {
      await api.post("/api/admin/categories", form);
    }
    showModal.value = false;
    await fetchCategories();
  } catch (err: any) {
    errorMessage.value =
      err.data?.message ||
      err.data?.errors?.nama_kategori?.[0] ||
      "Gagal menyimpan kategori";
  } finally {
    submitting.value = false;
  }
};

const deleteCategory = async (cat: any) => {
  if (!confirm(`Yakin ingin menghapus kategori '${cat.nama_kategori}'?`))
    return;
  try {
    const toast = useToast();
    await api.delete(`/api/admin/categories/${cat.id}`);
    toast.success("Kategori berhasil dihapus!");
    await fetchCategories();
  } catch (err: any) {
    const toast = useToast();
    toast.error(err.data?.message || "Gagal menghapus kategori");
  }
};

onMounted(fetchCategories);
</script>

<style scoped>
/* Semua warna mengikuti token di main.css, jadi mode terang dan gelap sinkron */
.page {
  --danger: #b0394f;
  color: var(--text);
}

:global(:root.dark) .page,
:global(:root[data-theme="dark"]) .page {
  --danger: #e68a9a;
}

.page-title {
  font-family: var(--font-display);
  font-size: 1.5rem;
  font-weight: 600;
  letter-spacing: -0.01em;
  color: var(--text);
}

.muted {
  color: var(--muted);
}

.cell-strong {
  color: var(--text);
}

.label {
  color: var(--text-2);
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

.alert-error {
  color: var(--danger);
  border: 1px solid color-mix(in srgb, var(--danger) 45%, transparent);
  border-radius: var(--radius-md);
  background-color: color-mix(in srgb, var(--danger) 8%, transparent);
}
</style>
