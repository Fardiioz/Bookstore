<template>
  <div class="page">
    <!-- Hero Section -->
    <section
      class="hero py-16 sm:py-20 px-4 sm:px-6 lg:px-8 overflow-hidden mb-12"
    >
      <div
        class="max-w-7xl mx-auto relative z-10 grid grid-cols-1 md:grid-cols-2 gap-12 items-center"
      >
        <div>
          <span
            class="eyebrow inline-flex items-center gap-1.5 px-3 py-1 text-xs font-semibold mb-6"
          >
            <LucideSparkles class="w-3.5 h-3.5" />
            Platform Toko Buku Online
          </span>
          <h1
            class="hero-title text-3xl sm:text-4xl lg:text-5xl leading-tight mb-6"
          >
            Temukan Buku Impianmu &amp; Perluas Wawasanmu
          </h1>
          <p class="muted text-base mb-8 leading-relaxed max-w-md">
            Jelajahi ribuan koleksi buku fiksi, teknologi, bisnis, hingga sains
            dengan harga terbaik dan transaksi mudah.
          </p>
          <div class="flex flex-wrap gap-3">
            <NuxtLink
              to="/user/katalog"
              class="btn-primary px-5 py-3 text-sm font-semibold inline-flex items-center gap-2"
            >
              <LucideStore class="w-4 h-4" />
              Jelajahi Katalog Buku
            </NuxtLink>
            <NuxtLink
              to="/our-story"
              class="btn-outline px-5 py-3 text-sm font-semibold inline-flex items-center gap-2"
            >
              Tentang Kami
            </NuxtLink>
          </div>
        </div>
        <div class="relative flex justify-center">
          <div class="hero-card w-64 h-80 flex flex-col justify-between p-7">
            <LucideBookOpen class="w-9 h-9" :stroke-width="1.5" />
            <div>
              <h3 class="hero-card-title text-xl mb-2">BookStore Digital</h3>
              <p class="hero-card-text text-xs leading-relaxed">
                Solusi belanja buku cepat, aman, dan mudah dari mana saja.
              </p>
            </div>
            <div
              class="hero-card-tag text-[11px] font-medium px-3 py-1.5 w-max"
            >
              Nuxt 3 + Laravel 12
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Categories Section -->
    <section class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 mb-12">
      <div class="flex flex-wrap justify-between items-end gap-4 mb-6">
        <div>
          <h2 class="section-title text-xl sm:text-2xl">Kategori Pilihan</h2>
          <p class="muted text-sm mt-1">
            Pilih kategori favoritmu untuk menemukan buku yang relevan
          </p>
        </div>
        <NuxtLink
          to="/user/katalog"
          class="btn-outline text-xs font-semibold px-3 py-1.5 inline-flex items-center gap-1.5"
        >
          Lihat Semua
          <LucideArrowRight class="w-3.5 h-3.5" />
        </NuxtLink>
      </div>

      <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-5 gap-4">
        <div
          v-for="cat in categories"
          :key="cat.id"
          @click="navigateToCatalog(cat.id)"
          class="cat-card p-5 cursor-pointer group text-center"
        >
          <div
            class="cat-icon w-11 h-11 flex items-center justify-center mx-auto mb-3"
          >
            <LucideFolder class="w-5 h-5" :stroke-width="1.5" />
          </div>
          <h3 class="cell-strong font-semibold text-sm">
            {{ cat.nama_kategori }}
          </h3>
        </div>
      </div>
    </section>

    <!-- Featured Books Section -->
    <section class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 pb-12">
      <h2 class="section-title text-xl sm:text-2xl mb-6">Buku Terbaru</h2>

      <div v-if="loading" class="muted text-center py-12 text-sm">
        Memuat buku...
      </div>
      <div
        v-else-if="books.length === 0"
        class="muted text-center py-12 text-sm"
      >
        Belum ada buku tersedia.
      </div>
      <div v-else class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-5">
        <div
          v-for="book in books"
          :key="book.id"
          @click="goToDetail(book)"
          class="card overflow-hidden flex flex-col justify-between cursor-pointer group"
        >
          <div>
            <div
              class="cover h-44 flex items-center justify-center overflow-hidden relative"
            >
              <img
                v-if="book.gambar"
                :src="book.gambar"
                :alt="book.nama_buku"
                class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-300"
              />
              <LucideBookOpen
                v-else
                class="w-8 h-8 cover-icon"
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
              <p class="muted text-[11px] mb-1">
                Terbit: {{ book.tahun_terbit || "-" }}
              </p>
              <p class="muted text-[11px] mb-2">
                Stok:
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
              <span class="price-text text-sm font-semibold"
                >Rp {{ formatNumber(book.harga_jual) }}</span
              >
            </div>
            <button
              type="button"
              @click.stop="addToCart(book)"
              :disabled="book.stok <= 0"
              class="btn-primary px-3.5 py-2 text-xs font-semibold inline-flex items-center gap-1.5 disabled:opacity-50 disabled:cursor-not-allowed"
            >
              <LucideShoppingCart class="w-3.5 h-3.5" />
              <span v-if="book.stok > 0">Tambah</span>
              <span v-else>Stok Habis</span>
            </button>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup lang="ts">
import {
  Sparkles as LucideSparkles,
  Store as LucideStore,
  BookOpen as LucideBookOpen,
  Folder as LucideFolder,
  ArrowRight as LucideArrowRight,
  ShoppingCart as LucideShoppingCart,
} from "lucide-vue-next";

const router = useRouter();
const api = useApi();
const cartStore = useCartStore();

const categories = ref<any[]>([]);
const books = ref<any[]>([]);
const loading = ref(true);

const formatNumber = (val: number) => {
  return new Intl.NumberFormat("id-ID").format(val);
};

const navigateToCatalog = (catId: number) => {
  navigateTo(`/user/katalog?category_id=${catId}`);
};

const goToDetail = (book: any) => {
  router.push(`/books/${book.id}`);
};

const addToCart = (book: any) => {
  try {
    const toast = useToast();
    cartStore.addToCart(book, 1);
    toast.success(`'${book.nama_buku}' berhasil ditambahkan ke keranjang!`);
  } catch (err: any) {
    const toast = useToast();
    toast.error(err.message || "Gagal menambahkan ke keranjang");
  }
};

onMounted(async () => {
  try {
    const catRes = await api.get("/api/categories");
    categories.value = catRes.data || [];

    const bookRes = await api.get("/api/books");
    let data = (bookRes.data || []).slice(0, 8); // Get first 8, then sort

    // Sort: stok > 0 first, then stok = 0 at the bottom
    data.sort((a: any, b: any) => {
      if (a.stok > 0 && b.stok <= 0) return -1;
      if (a.stok <= 0 && b.stok > 0) return 1;
      return 0;
    });

    books.value = data;
  } catch (e) {
    console.error(e);
  } finally {
    loading.value = false;
  }
});
</script>

<style scoped>
/* Semua warna mengikuti token di main.css, jadi mode terang dan gelap sinkron */
.page {
  color: var(--text);
}

.muted {
  color: var(--muted);
}

.cell-strong {
  color: var(--text);
}

/* hero */
.hero {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

.eyebrow {
  background-color: var(--inset);
  border: 1px solid var(--border-strong);
  border-radius: 9999px;
  color: var(--text-2);
}

.hero-title {
  font-family: var(--font-display);
  font-weight: 600;
  letter-spacing: -0.01em;
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

.btn-outline {
  color: var(--text);
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
  background: transparent;
  transition:
    background-color 0.15s ease,
    border-color 0.15s ease;
}

.btn-outline:hover {
  background-color: var(--inset);
  border-color: var(--text-2);
}

.hero-card {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  color: var(--text);
}

.hero-card-title {
  font-family: var(--font-display);
  font-weight: 600;
  color: var(--text);
}

.hero-card-text {
  color: var(--text-2);
}

.hero-card-tag {
  background-color: var(--surface);
  border: 1px solid var(--border-strong);
  border-radius: 9999px;
  color: var(--text-2);
}

/* kategori */
.section-title {
  font-family: var(--font-display);
  font-weight: 600;
  letter-spacing: -0.01em;
  color: var(--text);
}

.cat-card {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  transition:
    border-color 0.15s ease,
    background-color 0.15s ease;
}

.cat-card:hover {
  border-color: var(--border-strong);
  background-color: var(--inset);
}

.cat-icon {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  color: var(--text-2);
  transition: transform 0.15s ease;
}

.cat-card:hover .cat-icon {
  transform: scale(1.06);
}

/* kartu buku */
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
  color: #4d6b3f;
}

.stok-habis {
  color: #b0394f;
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
</style>
