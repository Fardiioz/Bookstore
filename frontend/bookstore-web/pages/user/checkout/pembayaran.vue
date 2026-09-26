<template>
  <div class="max-w-2xl mx-auto px-4 py-10">
    <!-- Header Page -->
    <div class="flex items-center gap-3 mb-8">
      <NuxtLink
        to="/user/keranjang"
        :class="
          isDark
            ? 'bg-slate-800 text-white border-slate-700'
            : 'bg-white border-2 border-[#1A1A1A] shadow-[2px_2px_0px_#1A1A1A]'
        "
        class="w-10 h-10 rounded-xl flex items-center justify-center font-black transition-all hover:translate-x-[-2px]"
      >
        ←
      </NuxtLink>
      <h1
        :class="
          isDark
            ? 'text-white'
            : 'text-slate-900 bg-[#FFE566] px-4 py-1.5 inline-block rounded-2xl border-2 border-[#1A1A1A] shadow-[4px_4px_0px_#1A1A1A]'
        "
        class="text-2xl font-black"
      >
        Pilih Pembayaran
      </h1>
    </div>

    <!-- Tampilan Kosong / Item Belum Dipilih -->
    <div
      v-if="cartStore.checkoutSelection.length === 0"
      :class="
        isDark
          ? 'bg-slate-900 border-slate-800 text-white'
          : 'bg-[#D4B8FF] border-3 border-[#1A1A1A] shadow-[6px_6px_0px_#1A1A1A] text-slate-900'
      "
      class="text-center py-14 rounded-3xl"
    >
      <p class="font-black text-sm mb-4">
        Belum ada item yang dipilih untuk checkout.
      </p>
      <NuxtLink
        to="/user/keranjang"
        :class="
          isDark
            ? 'bg-indigo-600 text-white'
            : 'bg-[#C8F53F] text-black border-2 border-[#1A1A1A] shadow-[3px_3px_0px_#1A1A1A] hover:bg-[#b8e82f]'
        "
        class="px-5 py-2.5 rounded-xl text-xs font-black inline-block transition-all active:translate-x-[2px] active:translate-y-[2px]"
      >
        Kembali ke Keranjang
      </NuxtLink>
    </div>

    <!-- Container Utama -->
    <div v-else class="space-y-6">
      <!-- Ringkasan Pesanan -->
      <div
        :class="
          isDark
            ? 'bg-slate-900 border-slate-800 text-slate-100'
            : 'bg-[#FFF8EC] border-3 border-[#1A1A1A] shadow-[6px_6px_0px_#1A1A1A] text-slate-900'
        "
        class="p-6 rounded-3xl space-y-4 transition-colors duration-300"
      >
        <h3
          :class="
            isDark
              ? 'text-white border-slate-800'
              : 'text-slate-900 border-[#1A1A1A]'
          "
          class="font-black text-sm border-b-2 pb-3"
        >
          Ringkasan Pesanan
        </h3>

        <div class="space-y-2">
          <div
            v-for="item in cartStore.checkoutSelection"
            :key="item.book_id"
            class="flex justify-between text-xs font-bold"
          >
            <span :class="isDark ? 'text-slate-300' : 'text-slate-800'">
              {{ item.nama_buku }}
              <span class="text-slate-500 font-black">x{{ item.qty }}</span>
            </span>
            <span class="font-black"
              >Rp {{ formatNumber(item.harga_jual * item.qty) }}</span
            >
          </div>
        </div>

        <div
          :class="
            isDark
              ? 'text-white border-slate-800'
              : 'text-slate-900 border-[#1A1A1A]'
          "
          class="flex justify-between items-center text-sm font-black pt-4 border-t-2"
        >
          <span>Total Bayar</span>
          <span
            :class="
              isDark
                ? 'bg-indigo-600 text-white'
                : 'bg-[#C8F53F] text-black border-2 border-[#1A1A1A] shadow-[2px_2px_0px_#1A1A1A]'
            "
            class="px-3 py-1 rounded-xl"
          >
            Rp {{ formatNumber(totalPrice) }}
          </span>
        </div>
      </div>

      <!-- Metode Pembayaran -->
      <div
        :class="
          isDark
            ? 'bg-slate-900 border-slate-800 text-slate-100'
            : 'bg-[#FFF8EC] border-3 border-[#1A1A1A] shadow-[6px_6px_0px_#1A1A1A] text-slate-900'
        "
        class="p-6 rounded-3xl space-y-4 transition-colors duration-300"
      >
        <h3
          :class="
            isDark
              ? 'text-white border-slate-800'
              : 'text-slate-900 border-[#1A1A1A]'
          "
          class="font-black text-sm border-b-2 pb-3"
        >
          Metode Pembayaran
        </h3>

        <!-- Cash (Aktif) -->
        <label
          :class="
            isDark
              ? 'border-slate-700 bg-slate-950 text-white'
              : 'border-2 border-[#1A1A1A] bg-white text-slate-900 shadow-[3px_3px_0px_#1A1A1A]'
          "
          class="flex items-center gap-3.5 p-4 rounded-2xl cursor-pointer transition-all"
        >
          <input
            type="radio"
            v-model="selectedMethod"
            value="cash"
            class="w-4 h-4 accent-black"
          />
          <div
            :class="
              isDark
                ? 'bg-slate-800'
                : 'bg-[#FFE566] border border-black shadow-[1.5px_1.5px_0px_#1A1A1A]'
            "
            class="w-10 h-10 rounded-xl flex items-center justify-center text-xl shrink-0"
          >
            💵
          </div>
          <div class="flex-1">
            <p class="font-black text-sm">Cash / Bayar di Kasir</p>
            <p
              :class="isDark ? 'text-slate-400' : 'text-slate-600'"
              class="text-[11px] font-bold mt-0.5"
            >
              Bayar langsung saat mengambil pesanan di toko
            </p>
          </div>
        </label>

        <!-- Transfer Bank (Disabled) -->
        <div
          :class="
            isDark
              ? 'border-slate-800 bg-slate-950/50'
              : 'border-2 border-dashed border-slate-300 bg-slate-50/50'
          "
          class="flex items-center gap-3.5 p-4 rounded-2xl opacity-60 cursor-not-allowed"
        >
          <input type="radio" disabled class="w-4 h-4" />
          <div
            class="w-10 h-10 rounded-xl bg-slate-200 dark:bg-slate-800 flex items-center justify-center text-xl shrink-0"
          >
            🏦
          </div>
          <div class="flex-1">
            <p class="font-black text-sm text-slate-500">Transfer Bank</p>
            <p class="text-[11px] font-bold text-slate-400 mt-0.5">
              Belum tersedia
            </p>
          </div>
          <span
            :class="
              isDark
                ? 'bg-slate-800 text-slate-400'
                : 'bg-slate-200 text-slate-700 border border-slate-300'
            "
            class="text-[9px] font-black uppercase px-2.5 py-1 rounded-lg"
          >
            Segera Hadir
          </span>
        </div>

        <!-- E-Wallet / QRIS (Disabled) -->
        <div
          :class="
            isDark
              ? 'border-slate-800 bg-slate-950/50'
              : 'border-2 border-dashed border-slate-300 bg-slate-50/50'
          "
          class="flex items-center gap-3.5 p-4 rounded-2xl opacity-60 cursor-not-allowed"
        >
          <input type="radio" disabled class="w-4 h-4" />
          <div
            class="w-10 h-10 rounded-xl bg-slate-200 dark:bg-slate-800 flex items-center justify-center text-xl shrink-0"
          >
            📱
          </div>
          <div class="flex-1">
            <p class="font-black text-sm text-slate-500">QRIS / E-Wallet</p>
            <p class="text-[11px] font-bold text-slate-400 mt-0.5">
              Belum tersedia
            </p>
          </div>
          <span
            :class="
              isDark
                ? 'bg-slate-800 text-slate-400'
                : 'bg-slate-200 text-slate-700 border border-slate-300'
            "
            class="text-[9px] font-black uppercase px-2.5 py-1 rounded-lg"
          >
            Segera Hadir
          </span>
        </div>
      </div>

      <!-- Tombol Buat Pesanan -->
      <button
        @click="confirmOrder"
        :disabled="loading"
        :class="
          isDark
            ? 'bg-indigo-600 hover:bg-indigo-700 text-white'
            : 'bg-[#C8F53F] hover:bg-[#b8e82f] text-black border-2 border-[#1A1A1A] shadow-[4px_4px_0px_#1A1A1A] active:translate-x-[2px] active:translate-y-[2px]'
        "
        class="w-full py-3.5 rounded-2xl font-black text-sm transition-all disabled:opacity-50 mt-2"
      >
        <span v-if="loading">Memproses Pesanan...</span>
        <span v-else>Buat Pesanan Sekarang &rarr;</span>
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
definePageMeta({
  middleware: "auth",
});

const api = useApi();
const cartStore = useCartStore();
const { isDark } = useTheme();
const loading = ref(false);
const selectedMethod = ref("cash");

const formatNumber = (val: number) =>
  new Intl.NumberFormat("id-ID").format(val);

const totalPrice = computed(() =>
  cartStore.checkoutSelection.reduce((sum, i) => sum + i.harga_jual * i.qty, 0),
);

onMounted(() => {
  // guard: jika user refresh atau langsung akses page ini tanpa item checkout
  if (cartStore.checkoutSelection.length === 0) {
    navigateTo("/user/keranjang");
  }
});

const confirmOrder = async () => {
  loading.value = true;
  try {
    const payload = {
      items: cartStore.checkoutSelection.map((i) => ({
        book_id: i.book_id,
        qty: i.qty,
      })),
      payment_method: selectedMethod.value,
    };

    const res = await api.post<{ message: string; data: any }>(
      "/api/orders",
      payload,
    );

    for (const item of cartStore.checkoutSelection) {
      await cartStore.removeFromCart(item.book_id);
    }
    cartStore.clearCheckoutSelection();

    const toast = useToast();
    toast.success(res.message || "Pesanan berhasil dibuat!");
    navigateTo("/user/riwayat");
  } catch (err: any) {
    const toast = useToast();
    toast.error(err.data?.message || "Gagal memproses checkout.");
  } finally {
    loading.value = false;
  }
};
</script>
