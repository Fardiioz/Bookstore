<template>
  <div class="page max-w-4xl mx-auto px-4 py-10">
    <div class="flex items-center justify-between gap-4 mb-8">
      <div>
        <h1 class="page-title text-2xl sm:text-3xl">Keranjang Belanja</h1>
        <p class="muted text-sm mt-2">
          Periksa kembali daftar buku yang akan Anda beli
        </p>
      </div>

      <button
        v-if="cartStore.items.length > 0"
        type="button"
        @click="cartStore.clearCart()"
        class="btn-ghost-danger text-xs font-semibold px-3 py-1.5"
      >
        Kosongkan Keranjang
      </button>
    </div>

    <!-- Keranjang kosong -->
    <div
      v-if="cartStore.items.length === 0"
      class="empty-panel text-center py-16 px-6 max-w-xl mx-auto"
    >
      <div
        class="empty-icon w-16 h-16 flex items-center justify-center mx-auto mb-4"
      >
        <LucideShoppingCart class="w-7 h-7" :stroke-width="1.5" />
      </div>
      <h3 class="empty-title font-semibold text-lg mb-1">
        Keranjang Belanja Masih Kosong
      </h3>
      <p class="muted text-sm mb-6 leading-relaxed">
        Jelajahi katalog kami untuk menemukan buku favoritmu dan mulai tambahkan
        ke keranjang.
      </p>
      <NuxtLink
        to="/user/katalog"
        class="btn-primary px-5 py-3 text-sm font-semibold inline-flex items-center gap-2"
      >
        <LucideStore class="w-4 h-4" />
        Ke Katalog Buku Sekarang
        <LucideArrowRight class="w-4 h-4" />
      </NuxtLink>
    </div>

    <div v-else class="grid grid-cols-1 md:grid-cols-3 gap-8">
      <!-- Daftar item -->
      <div class="md:col-span-2 space-y-3">
        <!-- Bar Pilih Semua -->
        <div class="select-bar p-3 flex items-center justify-between">
          <label class="flex items-center gap-2 cursor-pointer">
            <input
              type="checkbox"
              :checked="isAllSelected"
              @change="toggleSelectAll"
              class="checkbox w-4 h-4"
            />
            <span class="text-xs font-semibold">
              Pilih Semua ({{ cartStore.items.length }} buku)
            </span>
          </label>
          <span class="muted text-xs font-medium"
            >{{ selectedIds.length }} dipilih</span
          >
        </div>

        <div
          v-for="item in cartStore.items"
          :key="item.book_id"
          class="item-card p-4 flex items-center justify-between gap-4 transition-opacity"
          :class="{ 'item-card--dim': !selectedIds.includes(item.book_id) }"
        >
          <div class="flex items-center gap-4 min-w-0">
            <input
              type="checkbox"
              :checked="selectedIds.includes(item.book_id)"
              @change="toggleSelect(item.book_id)"
              class="checkbox w-4 h-4 shrink-0"
            />

            <div
              class="thumb w-14 h-18 overflow-hidden flex items-center justify-center shrink-0"
            >
              <img
                v-if="item.gambar"
                :src="item.gambar"
                :alt="item.nama_buku"
                class="w-full h-full object-cover"
              />
              <LucideBookOpen v-else class="w-5 h-5 thumb-icon" />
            </div>
            <div class="min-w-0">
              <h4 class="item-title font-semibold text-sm line-clamp-1">
                {{ item.nama_buku }}
              </h4>
              <p class="item-price text-xs font-semibold mt-1">
                Rp {{ formatNumber(item.harga_jual) }}
              </p>
              <p class="muted text-[11px] mt-0.5">Sisa stok: {{ item.stok }}</p>
            </div>
          </div>

          <!-- Kontrol qty -->
          <div class="flex items-center gap-2 shrink-0">
            <div class="qty-box flex items-center overflow-hidden">
              <button
                type="button"
                @click="updateQty(item.book_id, item.qty - 1)"
                class="qty-btn w-7 h-7 flex items-center justify-center"
                aria-label="Kurangi jumlah"
              >
                <LucideMinus class="w-3.5 h-3.5" />
              </button>
              <span class="qty-value px-2 text-xs font-semibold tabular-nums">{{
                item.qty
              }}</span>
              <button
                type="button"
                @click="updateQty(item.book_id, item.qty + 1)"
                class="qty-btn w-7 h-7 flex items-center justify-center"
                aria-label="Tambah jumlah"
              >
                <LucidePlus class="w-3.5 h-3.5" />
              </button>
            </div>

            <button
              type="button"
              @click="cartStore.removeFromCart(item.book_id)"
              class="remove-btn w-8 h-8 flex items-center justify-center"
              aria-label="Hapus dari keranjang"
              title="Hapus"
            >
              <LucideTrash2 class="w-4 h-4" />
            </button>
          </div>
        </div>
      </div>

      <!-- Ringkasan Pesanan -->
      <div class="summary-card p-6 h-max space-y-4">
        <h3 class="summary-title font-semibold text-base pb-3">
          Ringkasan Pesanan
        </h3>

        <div class="flex justify-between text-sm">
          <span class="muted">Item Dipilih</span>
          <span class="cell-strong font-semibold"
            >{{ selectedTotalItems }} buku</span
          >
        </div>

        <div
          class="summary-total flex justify-between items-center text-sm pt-3"
        >
          <span class="cell-strong font-semibold">Total Harga</span>
          <span class="total-value text-base font-semibold tabular-nums">
            Rp {{ formatNumber(selectedTotalPrice) }}
          </span>
        </div>

        <button
          type="button"
          @click="goToPayment"
          :disabled="selectedIds.length === 0"
          class="btn-primary w-full py-3.5 text-sm font-semibold disabled:opacity-40 disabled:cursor-not-allowed mt-2 inline-flex items-center justify-center gap-2"
        >
          <span v-if="selectedIds.length === 0">Pilih Buku Dulu</span>
          <template v-else>
            <span>Lanjut ke Pembayaran</span>
            <LucideArrowRight class="w-4 h-4" />
          </template>
        </button>

        <p class="muted text-[11px] text-center leading-relaxed">
          Kode pesanan akan otomatis di-generate oleh sistem backend (Format
          A021).
        </p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  ShoppingCart as LucideShoppingCart,
  Store as LucideStore,
  ArrowRight as LucideArrowRight,
  BookOpen as LucideBookOpen,
  Minus as LucideMinus,
  Plus as LucidePlus,
  Trash2 as LucideTrash2,
} from "lucide-vue-next";

definePageMeta({
  middleware: "auth",
});

const api = useApi();
const cartStore = useCartStore();

const selectedIds = ref<number[]>([]);

const formatNumber = (val: number) => {
  return new Intl.NumberFormat("id-ID").format(val);
};

const isAllSelected = computed(
  () =>
    cartStore.items.length > 0 &&
    selectedIds.value.length === cartStore.items.length,
);

const toggleSelectAll = () => {
  selectedIds.value = isAllSelected.value
    ? []
    : cartStore.items.map((i) => i.book_id);
};

const toggleSelect = (bookId: number) => {
  const idx = selectedIds.value.indexOf(bookId);
  if (idx === -1) {
    selectedIds.value.push(bookId);
  } else {
    selectedIds.value.splice(idx, 1);
  }
};

const selectedItems = computed(() =>
  cartStore.items.filter((i) => selectedIds.value.includes(i.book_id)),
);

const selectedTotalItems = computed(() =>
  selectedItems.value.reduce((sum, i) => sum + i.qty, 0),
);

const selectedTotalPrice = computed(() =>
  selectedItems.value.reduce((sum, i) => sum + i.harga_jual * i.qty, 0),
);

const updateQty = async (bookId: number, qty: number) => {
  try {
    const toast = useToast();
    await cartStore.updateQty(bookId, qty);
  } catch (err: any) {
    const toast = useToast();
    toast.error(err.message || "Stok tidak mencukupi");
  }
};

onMounted(async () => {
  await cartStore.fetchCart();
  // default: semua item ke-select pas pertama load
  selectedIds.value = cartStore.items.map((i) => i.book_id);
});

const goToPayment = () => {
  if (selectedIds.value.length === 0) return;
  // simpan pilihan ke store biar bisa diakses di halaman pembayaran
  cartStore.setCheckoutSelection(selectedItems.value);
  navigateTo("/user/checkout/pembayaran");
};
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

.btn-ghost-danger {
  color: var(--danger);
  border-radius: var(--radius-md);
  transition: opacity 0.15s ease;
}

.btn-ghost-danger:hover {
  opacity: 0.8;
}

/* keadaan kosong */
.empty-panel {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

.empty-icon {
  background-color: var(--inset);
  color: var(--text-2);
  border-radius: var(--radius-lg);
}

.empty-title {
  color: var(--text);
  font-family: var(--font-display);
}

/* bar pilih semua */
.select-bar {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
}

.checkbox {
  accent-color: var(--primary);
  border-radius: 4px;
}

/* item keranjang */
.item-card {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

.item-card--dim {
  opacity: 0.5;
}

.thumb {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
}

.thumb-icon {
  color: var(--muted);
}

.item-title {
  color: var(--text);
}

.item-price {
  color: var(--text-2);
}

/* qty control */
.qty-box {
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
}

.qty-btn {
  color: var(--text-2);
  background: transparent;
  transition: background-color 0.15s ease;
}

.qty-btn:hover {
  background-color: var(--inset);
}

.qty-value {
  color: var(--text);
  border-left: 1px solid var(--border);
  border-right: 1px solid var(--border);
}

.remove-btn {
  color: var(--danger);
  border-radius: var(--radius-md);
  transition: background-color 0.15s ease;
}

.remove-btn:hover {
  background-color: color-mix(in srgb, var(--danger) 10%, transparent);
}

/* ringkasan */
.summary-card {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

.summary-title {
  color: var(--text);
  font-family: var(--font-display);
  border-bottom: 1px solid var(--border);
}

.summary-total {
  border-top: 1px solid var(--border);
}

.total-value {
  color: var(--text);
}

/* tombol utama */
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
