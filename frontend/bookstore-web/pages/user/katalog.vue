<template>
  <div class="page max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-10">
    <!-- Header -->
    <div
      class="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-8"
    >
      <div>
        <h1 class="page-title text-2xl sm:text-3xl">Katalog Buku</h1>
        <p class="muted text-sm mt-2">
          Cari judul buku favoritmu atau filter berdasarkan kategori
        </p>
      </div>

      <!-- Search & Filter Controls -->
      <div class="flex flex-wrap items-center gap-3">
        <div class="relative w-full sm:w-64">
          <input
            v-model="searchQuery"
            @input="onSearchInput"
            type="text"
            placeholder="Cari judul buku..."
            class="field w-full pl-10 pr-4 py-2.5 text-sm"
          />
          <LucideSearch class="field-icon w-4 h-4 absolute left-3.5 top-3" />
        </div>

        <select
          v-model="selectedCategory"
          @change="fetchBooks"
          class="field px-3.5 py-2.5 text-sm"
        >
          <option value="">Semua Kategori</option>
          <option v-for="cat in categories" :key="cat.id" :value="cat.id">
            {{ cat.nama_kategori }}
          </option>
        </select>
      </div>
    </div>

    <!-- Books Grid Loading / Error / Empty States -->
    <div v-if="loading" class="muted text-center py-20 text-sm">
      Memuat buku...
    </div>

    <div v-else-if="errorMsg" class="empty-box text-center py-16 px-4">
      <h3 class="empty-title font-semibold text-base">Gagal memuat buku</h3>
      <p class="muted text-sm mt-1">{{ errorMsg }}</p>
      <button
        type="button"
        @click="fetchBooks"
        class="btn-primary mt-4 px-4 py-2 text-sm font-semibold"
      >
        Coba lagi
      </button>
    </div>

    <div v-else-if="books.length === 0" class="empty-box text-center py-20">
      <LucideSearchX class="w-8 h-8 mx-auto mb-3" :stroke-width="1.5" />
      <h3 class="empty-title font-semibold text-base">Buku tidak ditemukan</h3>
      <p class="muted text-sm mt-1">
        Coba kata kunci pencarian atau kategori lain.
      </p>
    </div>

    <!-- Grid List Buku -->
    <div
      v-else
      class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-5"
    >
      <div
        v-for="book in books"
        :key="book.id"
        @click="goToDetail(book)"
        class="card overflow-hidden flex flex-col justify-between group cursor-pointer"
      >
        <div>
          <div
            class="cover h-48 flex items-center justify-center overflow-hidden relative"
          >
            <img
              v-if="book.gambar && !brokenImages[book.id]"
              :src="book.gambar"
              :alt="book.nama_buku"
              @error="brokenImages[book.id] = true"
              class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-300"
            />
            <LucideBookOpen
              v-else
              class="w-9 h-9 cover-icon"
              :stroke-width="1.5"
            />
            <span
              class="tag-badge absolute top-3 left-3 text-[10px] font-medium px-2 py-0.5"
            >
              {{ book.category_name || "Buku" }}
            </span>
          </div>

          <div class="p-4">
            <h3 class="book-title font-semibold text-sm mb-1 line-clamp-1">
              {{ book.nama_buku }}
            </h3>
            <p class="muted text-[11px] mb-2">
              Tahun terbit: {{ book.tahun_terbit || "-" }}
              &middot; Stok:
              <span
                :class="book.stok > 0 ? 'stok-ok' : 'stok-habis'"
                class="font-semibold"
              >
                {{ book.stok }}
              </span>
            </p>
            <p class="desc-text text-xs line-clamp-2 leading-relaxed">
              {{ book.deskripsi || "Tidak ada deskripsi" }}
            </p>
          </div>
        </div>

        <div class="card-foot p-4 pt-3 flex items-center justify-between">
          <div>
            <span class="muted text-[10px] block font-medium">Harga</span>
            <span class="price-text text-sm font-semibold">
              Rp {{ formatNumber(book.harga_jual) }}
            </span>
          </div>

          <button
            type="button"
            @click.stop="addToCart(book)"
            :disabled="book.stok <= 0"
            class="btn-primary px-3.5 py-2 text-xs font-semibold inline-flex items-center gap-1.5 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            <LucideShoppingCart class="w-3.5 h-3.5" />
            <span v-if="book.stok > 0">Beli</span>
            <span v-else>Habis</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  Search as LucideSearch,
  SearchX as LucideSearchX,
  BookOpen as LucideBookOpen,
  ShoppingCart as LucideShoppingCart,
} from "lucide-vue-next";

const route = useRoute();
const router = useRouter();
const api = useApi();
const cartStore = useCartStore();
const authStore = useAuthStore();

const searchQuery = ref("");
const selectedCategory = ref<string | number>(
  route.query.category_id ? String(route.query.category_id) : "",
);
const categories = ref<any[]>([]);
const books = ref<any[]>([]);
const loading = ref(true);
const errorMsg = ref("");
const brokenImages = reactive<Record<string | number, boolean>>({});

const formatNumber = (val: number) => {
  return new Intl.NumberFormat("id-ID").format(val || 0);
};

const goToDetail = (book: any) => {
  router.push(`/books/${book.id}`);
};

const extractList = (payload: any, depth = 0): any[] => {
  if (Array.isArray(payload)) return payload;
  if (!payload || typeof payload !== "object" || depth > 3) return [];
  const keys = ["data", "books", "items", "results", "rows"];
  for (const key of keys) {
    if (key in payload) {
      const found = extractList(payload[key], depth + 1);
      if (found.length || Array.isArray(payload[key])) return found;
    }
  }
  return [];
};

let searchTimer: ReturnType<typeof setTimeout> | null = null;
const onSearchInput = () => {
  if (searchTimer) clearTimeout(searchTimer);
  searchTimer = setTimeout(fetchBooks, 350);
};

const fetchBooks = async () => {
  loading.value = true;
  errorMsg.value = "";
  try {
    const params: any = {};
    if (searchQuery.value) params.search = searchQuery.value;
    if (selectedCategory.value) params.category_id = selectedCategory.value;

    const res = await api.get("/api/books", { params });
    let data = extractList(res?.data ?? res);

    data = [...data].sort((a: any, b: any) => {
      if (a.stok > 0 && b.stok <= 0) return -1;
      if (a.stok <= 0 && b.stok > 0) return 1;
      return 0;
    });

    books.value = data;
  } catch (e: any) {
    console.error(e);
    books.value = [];
    errorMsg.value =
      e?.data?.message ||
      e?.message ||
      "Terjadi kesalahan saat menghubungi server.";
  } finally {
    loading.value = false;
  }
};

/**
 * Pengecekan Login & Penambahan ke Keranjang
 */
const addToCart = async (book: any) => {
  // Cek apakah user sudah terautentikasi
  const isUserLoggedIn =
    authStore.isLoggedIn ||
    authStore.isAuthenticated ||
    Boolean(authStore.token);

  if (!isUserLoggedIn) {
    // ❌ BELUM LOGIN: Langsung redirect tanpa notifikasi
    return navigateTo({
      path: "/login",
      query: { redirect: route.fullPath },
    });
  }

  // ✅ SUDAH LOGIN: Tambahkan ke keranjang dan tampilkan notifikasi khusus
  try {
    await cartStore.addToCart(book, 1);

    // Tampilkan notifikasi khusus untuk user yang sudah login
    const toast = useToast();
    toast.success(
      `'${book.nama_buku}' berhasil ditambahkan ke keranjang belanja!`,
    );
  } catch (err: any) {
    const toast = useToast();
    toast.error(err.message || "Gagal menambahkan buku ke keranjang.");
  }
};

onMounted(async () => {
  try {
    const catRes = await api.get("/api/categories");
    categories.value = extractList(catRes?.data ?? catRes);
  } catch (e) {}

  await fetchBooks();
});

onUnmounted(() => {
  if (searchTimer) clearTimeout(searchTimer);
});
</script>

<style scoped>
.page {
  color: var(--text);
}

.page-title {
  font-family: var(--font-display);
  font-weight: 600;
  letter-spacing: -0.01em;
  color: var(--text);
}

.muted,
.field-icon {
  color: var(--muted);
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

.empty-box {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  color: var(--text-2);
}

.empty-title {
  color: var(--text);
}

.card {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  transition: border-color 0.15s ease;
}

.card:hover {
  border-color: var(--border-strong);
}

.cover {
  background-color: var(--inset);
  border-bottom: 1px solid var(--border);
}

.cover-icon {
  color: var(--muted);
}

.tag-badge {
  background-color: var(--surface);
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-sm);
  color: var(--text-2);
}

.book-title {
  color: var(--text);
}

.desc-text {
  color: var(--text-2);
}

.stok-ok {
  color: var(--success, #4d6b3f);
}

.stok-habis {
  color: var(--danger, #b0394f);
}

:global(:root.dark) .stok-ok,
:global(:root[data-theme="dark"]) .stok-ok {
  color: #b3c0a4;
}

:global(:root.dark) .stok-habis,
:global(:root[data-theme="dark"]) .stok-habis {
  color: #e68a9a;
}

.card-foot {
  border-top: 1px solid var(--border);
}

.price-text {
  color: var(--text);
}

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
</style>
