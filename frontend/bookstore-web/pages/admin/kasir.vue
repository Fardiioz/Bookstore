<template>
  <div class="page space-y-6">
    <!-- Toast Lokal (menggantikan useToast() global agar posisi & z-index terkontrol) -->
    <div
      class="toast-stack fixed top-4 right-4 z-[9999] flex flex-col gap-2 w-[calc(100%-2rem)] max-w-sm"
    >
      <transition-group name="toast-fade">
        <div
          v-for="t in toasts"
          :key="t.id"
          class="toast p-3.5 flex items-start gap-2.5 shadow-lg"
          :class="t.type === 'error' ? 'toast--error' : 'toast--success'"
        >
          <component
            :is="t.type === 'error' ? LucideXCircle : LucideCheckCircle2"
            class="w-5 h-5 shrink-0 mt-0.5"
          />
          <div class="flex-1 min-w-0">
            <p class="text-sm font-semibold whitespace-pre-line break-words">
              {{ t.message }}
            </p>
          </div>
          <button
            type="button"
            @click="dismissToast(t.id)"
            class="shrink-0 opacity-70 hover:opacity-100"
          >
            <LucideX class="w-4 h-4" />
          </button>
        </div>
      </transition-group>
    </div>

    <!-- Search / USB Scanner / Camera Scanner -->
    <div
      class="flex flex-col md:flex-row items-stretch md:items-center justify-between gap-4"
    >
      <div class="flex items-center gap-2 w-full md:w-auto flex-1">
        <div class="relative w-full max-w-md">
          <input
            ref="scannerInput"
            v-model="scanQuery"
            type="text"
            placeholder="Scan QR Code / Masukkan Kode (A021)..."
            class="field w-full pl-10 pr-4 py-2.5 text-sm"
            @keyup.enter="handleScanSubmit"
          />
          <LucideQrCode class="field-icon w-4 h-4 absolute left-3.5 top-3" />
        </div>
        <button
          type="button"
          @click="startCameraScanner"
          class="btn-outline inline-flex items-center gap-1.5 px-4 py-2.5 text-sm font-semibold whitespace-nowrap"
        >
          <LucideCamera class="w-4 h-4" />
          Scan dengan Kamera
        </button>
      </div>

      <div class="flex items-center gap-2 flex-wrap">
        <span class="muted text-xs font-medium">Status</span>
        <button
          type="button"
          @click="filterStatus = ''"
          class="chip px-3 py-1.5 text-xs font-semibold"
          :class="{ 'chip--active': filterStatus === '' }"
        >
          Semua
        </button>
        <button
          type="button"
          @click="filterStatus = 'pending'"
          class="chip px-3 py-1.5 text-xs font-semibold"
          :class="{ 'chip--active': filterStatus === 'pending' }"
        >
          Pending
        </button>
        <button
          type="button"
          @click="filterStatus = 'confirmed'"
          class="chip px-3 py-1.5 text-xs font-semibold"
          :class="{ 'chip--active': filterStatus === 'confirmed' }"
        >
          Confirmed
        </button>
        <button
          type="button"
          @click="filterStatus = 'completed'"
          class="chip px-3 py-1.5 text-xs font-semibold"
          :class="{ 'chip--active': filterStatus === 'completed' }"
        >
          Completed
        </button>
      </div>
    </div>

    <!-- Modal Scanner Kamera Web / HP -->
    <div
      v-if="showCameraModal"
      class="modal-overlay fixed inset-0 z-50 flex items-center justify-center p-4"
    >
      <div
        class="modal p-6 max-w-md w-full space-y-4 text-center relative overflow-hidden"
        role="dialog"
        aria-modal="true"
      >
        <div class="modal-head flex justify-between items-center pb-3">
          <h3 class="modal-title text-lg flex items-center gap-2">
            <LucideCamera class="w-4 h-4" />
            Pemindai Kamera QR Code
          </h3>
          <button
            type="button"
            @click="stopCameraScanner"
            class="btn-ghost inline-flex items-center gap-1.5 px-3 py-1.5 text-xs font-semibold"
          >
            <LucideX class="w-4 h-4" />
            Tutup Kamera
          </button>
        </div>

        <div
          class="video-box relative aspect-square w-full overflow-hidden flex items-center justify-center"
        >
          <video ref="videoRef" class="w-full h-full object-cover"></video>
          <canvas ref="canvasRef" class="hidden"></canvas>
          <div
            class="scan-frame absolute inset-0 m-10 pointer-events-none flex items-end justify-center"
          >
            <span class="scan-hint text-xs font-medium px-3 py-1 mb-2"
              >Posisikan QR di dalam kotak</span
            >
          </div>
        </div>

        <p class="muted text-xs">
          Arahkan kamera ke QR Code di layar HP pembeli untuk memindai otomatis.
        </p>
      </div>
    </div>

    <!-- Banner Notifikasi Scan Berhasil -->
    <div
      v-if="scanSuccessBanner"
      class="banner p-4 flex items-start justify-between gap-4"
      role="status"
    >
      <div class="flex items-start gap-3">
        <LucideCheckCircle2 class="banner-icon w-5 h-5 shrink-0 mt-0.5" />
        <div>
          <h3 class="font-semibold text-sm">
            Scan QR Code berhasil dan dicatat di admin
          </h3>
          <p class="muted text-xs mt-1 leading-relaxed">
            Kode pesanan:
            <span class="code">{{ scanSuccessCode }}</span
            >. Transaksi terdeteksi. Mohon proses pembayaran atau konfirmasi,
            lalu berikan struk atau invoice langsung kepada pelanggan.
          </p>
        </div>
      </div>
      <button
        type="button"
        @click="scanSuccessBanner = false"
        class="btn-ghost inline-flex items-center gap-1.5 px-3 py-1.5 text-xs font-semibold shrink-0"
      >
        <LucideX class="w-4 h-4" />
        Tutup
      </button>
    </div>

    <!-- Pesanan terpilih / hasil scan -->
    <div
      v-if="selectedOrder"
      id="selected-order-section"
      class="order-card space-y-4"
    >
      <div class="panel p-6 relative">
        <div class="flex items-start justify-between gap-4 mb-4">
          <div>
            <h2 class="order-title text-xl">
              Kode Pesanan:
              <span class="code code--lg uppercase">{{
                selectedOrder.kode_pesanan
              }}</span>
            </h2>
            <p class="muted text-xs mt-1">
              Pelanggan:
              {{ selectedOrder.user_name || selectedOrder.pelanggan || "User" }}
            </p>
          </div>
          <div class="flex items-center gap-2">
            <span
              :class="getStatusBadgeClass(selectedOrder.status)"
              class="status px-2.5 py-0.5 text-[11px] font-medium"
            >
              {{ selectedOrder.status }}
            </span>
            <button
              type="button"
              @click="selectedOrder = null"
              class="btn-ghost inline-flex items-center gap-1.5 px-2.5 py-1 text-xs font-semibold print:hidden"
            >
              <LucideX class="w-4 h-4" />
              Tutup
            </button>
          </div>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-4 gap-6 items-center">
          <!-- Tabel rincian -->
          <div class="lg:col-span-3 overflow-x-auto">
            <table class="tbl tbl--bordered w-full text-left text-xs">
              <thead>
                <tr>
                  <th class="px-4 py-2.5">Judul Buku</th>
                  <th class="px-4 py-2.5">Tanggal</th>
                  <th class="px-4 py-2.5 text-center">Qty</th>
                  <th class="px-4 py-2.5 text-right">Harga Satuan</th>
                  <th class="px-4 py-2.5 text-right">Subtotal</th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="detail in selectedOrder.details ||
                  selectedOrder.items ||
                  []"
                  :key="detail.id"
                >
                  <td class="cell-strong px-4 py-2.5 font-semibold">
                    {{ detail.buku?.judul || detail.nama_buku }}
                  </td>
                  <td class="muted px-4 py-2.5">
                    {{ formatDate(selectedOrder.created_at) }}
                  </td>
                  <td
                    class="px-4 py-2.5 text-center tabular-nums font-semibold"
                  >
                    {{ detail.qty }}
                  </td>
                  <td class="px-4 py-2.5 text-right tabular-nums">
                    Rp.{{ formatPrice(detail.harga_satuan) }}
                  </td>
                  <td class="px-4 py-2.5 text-right tabular-nums font-semibold">
                    Rp.{{ formatPrice(detail.subtotal) }}
                  </td>
                </tr>
                <tr class="total-row">
                  <td
                    colspan="5"
                    class="px-4 py-2.5 text-right font-semibold text-sm"
                  >
                    Total Tagihan: Rp.
                    {{ formatPrice(selectedOrder.total_harga) }}
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- QR Code -->
          <div
            class="qr-box lg:col-span-1 flex flex-col items-center justify-center p-2"
          >
            <QrCodeDisplay
              :value="selectedOrder.kode_pesanan"
              :size="130"
              show-label
            />
            <p
              class="muted text-[11px] text-center mt-2 flex items-center gap-1"
            >
              <LucideCheck class="w-3.5 h-3.5" />
              QR Code terverifikasi
            </p>
          </div>
        </div>
      </div>

      <!-- Aksi kasir -->
      <div class="flex flex-col sm:flex-row gap-3 print:hidden">
        <button
          v-if="selectedOrder.status === 'pending'"
          type="button"
          @click="confirmOrder(selectedOrder)"
          class="btn-primary flex-1 inline-flex items-center justify-center gap-2 py-3 text-sm font-semibold"
        >
          <LucideCheck class="w-4 h-4" />
          Konfirmasi Pesanan
        </button>

        <button
          v-if="selectedOrder.status === 'confirmed'"
          type="button"
          @click="openPayModal(selectedOrder)"
          class="btn-primary flex-1 inline-flex items-center justify-center gap-2 py-3 text-sm font-semibold"
        >
          <LucideBanknote class="w-4 h-4" />
          Bayar Uang Tunai
        </button>

        <button
          type="button"
          @click="printReceipt"
          class="btn-outline flex-1 inline-flex items-center justify-center gap-2 py-3 text-sm font-semibold"
        >
          <LucidePrinter class="w-4 h-4" />
          Print Struk Kasir
        </button>

        <button
          type="button"
          @click="downloadInvoicePdf(selectedOrder.id)"
          class="btn-outline flex-1 inline-flex items-center justify-center gap-2 py-3 text-sm font-semibold"
        >
          <LucideFileDown class="w-4 h-4" />
          Unduh Invoice PDF
        </button>
      </div>
    </div>

    <!-- Daftar pesanan -->
    <div class="table-wrap overflow-hidden">
      <div v-if="loading" class="muted text-center py-16 text-sm">
        Memuat daftar pesanan...
      </div>
      <div
        v-else-if="filteredOrders.length === 0"
        class="muted text-center py-16 text-sm"
      >
        Tidak ada pesanan ditemukan.
      </div>
      <div v-else class="overflow-x-auto">
        <table class="tbl w-full text-left text-xs">
          <thead>
            <tr>
              <th class="px-4 py-3">Kode Pesanan</th>
              <th class="px-4 py-3">Pelanggan</th>
              <th class="px-4 py-3">Tanggal</th>
              <th class="px-4 py-3 text-right">Total Tagihan</th>
              <th class="px-4 py-3">Status</th>
              <th class="px-4 py-3 text-right">Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="order in paginatedOrders" :key="order.id">
              <td class="px-4 py-3">
                <span class="code uppercase">{{ order.kode_pesanan }}</span>
              </td>
              <td class="cell-strong px-4 py-3 font-semibold">
                {{ order.user_name || order.pelanggan || "User" }}
              </td>
              <td class="muted px-4 py-3">
                {{ formatDate(order.created_at) }}
              </td>
              <td
                class="cell-strong px-4 py-3 text-right tabular-nums font-semibold"
              >
                Rp. {{ formatPrice(order.total_harga) }}
              </td>
              <td class="px-4 py-3">
                <span
                  :class="getStatusBadgeClass(order.status)"
                  class="status px-2 py-0.5 text-[11px] font-medium"
                >
                  {{ order.status }}
                </span>
              </td>
              <td class="px-4 py-3 text-right">
                <button
                  type="button"
                  @click="selectOrder(order)"
                  class="btn-outline px-3 py-1.5 text-xs font-semibold"
                >
                  Detail Struk
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Paginasi -->
      <div v-if="totalPages > 1" class="pager p-4">
        <div class="flex items-center justify-center gap-2 flex-wrap">
          <button
            type="button"
            @click="currentPage = 1"
            :disabled="currentPage === 1"
            class="page-btn inline-flex items-center gap-1 px-3 h-8 text-xs font-semibold"
          >
            <LucideChevronsLeft class="w-4 h-4" />
            Awal
          </button>

          <button
            type="button"
            @click="currentPage--"
            :disabled="currentPage === 1"
            class="page-btn inline-flex items-center gap-1 px-3 h-8 text-xs font-semibold"
          >
            <LucideChevronLeft class="w-4 h-4" />
            Sebelumnya
          </button>

          <div class="flex items-center gap-1">
            <button
              v-for="page in totalPages"
              :key="page"
              type="button"
              @click="currentPage = page"
              class="page-btn w-8 h-8 text-xs font-semibold flex items-center justify-center"
              :class="{ 'page-btn--active': currentPage === page }"
            >
              {{ page }}
            </button>
          </div>

          <button
            type="button"
            @click="currentPage++"
            :disabled="currentPage === totalPages"
            class="page-btn inline-flex items-center gap-1 px-3 h-8 text-xs font-semibold"
          >
            Berikutnya
            <LucideChevronRight class="w-4 h-4" />
          </button>

          <button
            type="button"
            @click="currentPage = totalPages"
            :disabled="currentPage === totalPages"
            class="page-btn inline-flex items-center gap-1 px-3 h-8 text-xs font-semibold"
          >
            Akhir
            <LucideChevronsRight class="w-4 h-4" />
          </button>

          <span class="muted text-xs ml-2">
            Hal {{ currentPage }} dari {{ totalPages }}
          </span>
        </div>
      </div>
    </div>

    <!-- Modal Pembayaran Tunai -->
    <div
      v-if="showPayModal"
      class="modal-overlay fixed inset-0 z-50 flex items-center justify-center p-4"
    >
      <div
        class="modal p-6 max-w-md w-full space-y-4"
        role="dialog"
        aria-modal="true"
      >
        <div class="modal-head pb-3">
          <h3 class="modal-title text-lg">Kasir Pembayaran Tunai</h3>
          <p class="muted text-xs mt-1">
            Kode Pesanan:
            <span class="code uppercase">{{
              activePayOrder?.kode_pesanan
            }}</span>
          </p>
        </div>

        <div class="total-box p-4 text-center">
          <span class="muted text-xs block">Total Tagihan Harus Dibayar</span>
          <span class="total-value text-2xl tabular-nums"
            >Rp. {{ formatPrice(activePayOrder?.total_harga || 0) }}</span
          >
        </div>

        <form @submit.prevent="processPayment" class="space-y-4">
          <div>
            <label class="label block text-xs font-semibold mb-2">
              Pilih Nominal Uang Tunai
            </label>

            <!-- Tombol nominal cepat -->
            <div class="grid grid-cols-4 gap-2 mb-3">
              <button
                v-for="d in denominations"
                :key="d"
                type="button"
                @click="addCash(d)"
                class="btn-outline py-2 text-[11px] font-semibold tabular-nums"
              >
                +{{ formatShort(d) }}
              </button>
            </div>

            <div class="flex gap-2 mb-3">
              <button
                type="button"
                @click="setExactCash"
                class="btn-outline flex-1 inline-flex items-center justify-center gap-1.5 py-2 text-[11px] font-semibold"
              >
                <LucideBanknote class="w-4 h-4" />
                Uang Pas
              </button>
              <button
                type="button"
                @click="resetCash"
                class="btn-ghost flex-1 inline-flex items-center justify-center gap-1.5 py-2 text-[11px] font-semibold"
              >
                <LucideRotateCcw class="w-4 h-4" />
                Reset
              </button>
            </div>

            <label class="label block text-xs font-semibold mb-1">
              Atau Ketik Manual (Rp)
            </label>
            <input
              v-model.number="cashInput"
              type="number"
              required
              :min="activePayOrder?.total_harga"
              class="field w-full px-4 py-3 text-base font-semibold tabular-nums"
              placeholder="100000"
            />
          </div>

          <div
            class="change-box p-3.5 flex justify-between items-center text-xs"
          >
            <span class="font-semibold">Uang Kembalian</span>
            <span
              :class="computedKembalian >= 0 ? 'change-ok' : 'change-neg'"
              class="font-semibold text-base tabular-nums"
            >
              Rp. {{ formatPrice(computedKembalian) }}
            </span>
          </div>

          <div class="flex justify-end gap-3 pt-2">
            <button
              type="button"
              @click="showPayModal = false"
              class="btn-ghost px-4 py-2 text-xs font-semibold"
            >
              Batal
            </button>
            <button
              type="submit"
              :disabled="submittingPay || computedKembalian < 0"
              class="btn-primary px-5 py-2.5 text-xs font-semibold disabled:opacity-50 disabled:cursor-not-allowed"
            >
              Selesaikan Pembayaran dan Lunas
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  QrCode as LucideQrCode,
  Camera as LucideCamera,
  X as LucideX,
  CheckCircle2 as LucideCheckCircle2,
  XCircle as LucideXCircle,
  Check as LucideCheck,
  Banknote as LucideBanknote,
  Printer as LucidePrinter,
  FileDown as LucideFileDown,
  RotateCcw as LucideRotateCcw,
  ChevronLeft as LucideChevronLeft,
  ChevronRight as LucideChevronRight,
  ChevronsLeft as LucideChevronsLeft,
  ChevronsRight as LucideChevronsRight,
} from "lucide-vue-next";
import jsQR from "jsqr";

definePageMeta({
  middleware: "admin",
});

const api = useApi();
const orders = ref<any[]>([]);
const loading = ref(true);
const filterStatus = ref("");
const currentPage = ref(1);
const itemsPerPage = 6;

const scanQuery = ref("");
const selectedOrder = ref<any>(null);
const scannerInput = ref<HTMLInputElement | null>(null);

const scanSuccessBanner = ref(false);
const scanSuccessCode = ref("");

const showPayModal = ref(false);
const activePayOrder = ref<any>(null);
const cashInput = ref<number | "">("");
const denominations = [1000, 2000, 5000, 10000, 20000, 50000, 100000];

/**
 * ---------------------------------------------------------------------
 * TOAST LOKAL
 * Menggantikan pemanggilan useToast() global. Dibuat lokal di sini agar
 * posisi (top-right, kecil) dan durasi auto-dismiss bisa dikontrol penuh
 * dan TIDAK menimpa navbar/header di atasnya.
 * ---------------------------------------------------------------------
 */
interface LocalToast {
  id: number;
  message: string;
  type: "success" | "error";
}
const toasts = ref<LocalToast[]>([]);
let toastCounter = 0;

const showLocalToast = (
  message: string,
  type: "success" | "error" = "success",
  duration = 4000,
) => {
  const id = ++toastCounter;
  toasts.value.push({ id, message, type });
  if (process.client) {
    setTimeout(() => dismissToast(id), duration);
  }
};

const dismissToast = (id: number) => {
  toasts.value = toasts.value.filter((t) => t.id !== id);
};

const formatShort = (val: number) => {
  if (val >= 1000) return `${val / 1000}rb`;
  return `${val}`;
};

const addCash = (amount: number) => {
  cashInput.value = (Number(cashInput.value) || 0) + amount;
};

const setExactCash = () => {
  cashInput.value = activePayOrder.value?.total_harga || 0;
};

const resetCash = () => {
  cashInput.value = 0;
};
const submittingPay = ref(false);

// Camera scanner state
const showCameraModal = ref(false);
const videoRef = ref<HTMLVideoElement | null>(null);
const canvasRef = ref<HTMLCanvasElement | null>(null);
let cameraStream: MediaStream | null = null;
let animationFrameId: number | null = null;

const formatPrice = (val: number) => {
  return new Intl.NumberFormat("id-ID").format(val || 0);
};

const formatDate = (dateStr: string) => {
  if (!dateStr) return "-";
  const d = new Date(dateStr);
  return `${d.getDate()}/${d.getMonth() + 1}/${d.getFullYear()}`;
};

const getStatusBadgeClass = (status: string) => {
  switch (status) {
    case "pending":
      return "status--pending";
    case "confirmed":
      return "status--confirmed";
    case "completed":
      return "status--completed";
    default:
      return "status--default";
  }
};

const filteredOrders = computed(() => {
  if (!filterStatus.value) return orders.value;
  return orders.value.filter((o) => o.status === filterStatus.value);
});

const paginatedOrders = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return filteredOrders.value.slice(start, end);
});

const totalPages = computed(() => {
  return Math.ceil(filteredOrders.value.length / itemsPerPage);
});

const computedKembalian = computed(() => {
  if (!activePayOrder.value || !cashInput.value) return 0;
  return Number(cashInput.value) - Number(activePayOrder.value.total_harga);
});

const fetchOrders = async () => {
  loading.value = true;
  try {
    const res = await api.get("/api/orders");
    orders.value = res.data || [];
  } catch (e) {
    console.error(e);
  } finally {
    loading.value = false;
  }
};

const selectOrder = (order: any) => {
  selectedOrder.value = order;
};

const handleScanSubmit = async () => {
  const query = scanQuery.value.trim().toLowerCase();
  if (!query) return;

  try {
    const res = await api.post("/api/orders/scan", { kode_pesanan: query });
    const matched = res.data;

    selectedOrder.value = matched;
    scanSuccessCode.value = matched.kode_pesanan;
    scanSuccessBanner.value = true;
    scanQuery.value = "";

    // Play audio beep sound on scan success
    if (process.client) {
      try {
        const audioCtx = new (
          window.AudioContext || (window as any).webkitAudioContext
        )();
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = "sine";
        osc.frequency.setValueAtTime(880, audioCtx.currentTime);
        gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.start();
        osc.stop(audioCtx.currentTime + 0.15);
      } catch (e) {
        // Ignore audio context errors
      }
    }

    // Notifikasi toast lokal untuk Admin
    showLocalToast(
      `Scan Berhasil & Tersimpan di Database!\nKode Pesanan: ${matched.kode_pesanan}\nPelanggan: ${matched.user_name || matched.pelanggan || "User"}\nStatus DB: ${matched.status.toUpperCase()}\nTotal: Rp. ${formatPrice(matched.total_harga)}\nStatus pesanan otomatis diperbarui ke 'CONFIRMED'.`,
      "success",
    );
    await fetchOrders();
  } catch (err: any) {
    showLocalToast(
      err.data?.message ||
        `Pesanan dengan kode "${scanQuery.value}" tidak ditemukan di Database!`,
      "error",
    );
  }
};

// Camera Scanner Logic
const startCameraScanner = async () => {
  showCameraModal.value = true;
  await nextTick();
  try {
    cameraStream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: "environment" },
    });
    if (videoRef.value) {
      videoRef.value.srcObject = cameraStream;
      videoRef.value.setAttribute("playsinline", "true");
      videoRef.value.play();
      requestAnimationFrame(scanVideoFrame);
    }
  } catch (err) {
    showLocalToast(
      "Gagal mengakses kamera. Pastikan izin kamera telah diberikan di browser.",
      "error",
    );
    showCameraModal.value = false;
  }
};

const scanVideoFrame = () => {
  if (!showCameraModal.value || !videoRef.value) return;

  if (videoRef.value.readyState === videoRef.value.HAVE_ENOUGH_DATA) {
    const video = videoRef.value;
    const canvas = canvasRef.value || document.createElement("canvas");
    canvas.width = video.videoWidth;
    canvas.height = video.videoHeight;
    const ctx = canvas.getContext("2d");
    if (ctx) {
      ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
      const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      const code = jsQR(imageData.data, imageData.width, imageData.height, {
        inversionAttempts: "dontInvert",
      });

      if (code && code.data) {
        scanQuery.value = code.data;
        stopCameraScanner();
        handleScanSubmit();
        return;
      }
    }
  }
  animationFrameId = requestAnimationFrame(scanVideoFrame);
};

const stopCameraScanner = () => {
  if (animationFrameId) cancelAnimationFrame(animationFrameId);
  if (cameraStream) {
    cameraStream.getTracks().forEach((track) => track.stop());
    cameraStream = null;
  }
  showCameraModal.value = false;
};

const confirmOrder = async (order: any) => {
  try {
    const res = await api.put(`/api/admin/orders/${order.id}/confirm`);
    showLocalToast(res.message || "Pesanan berhasil dikonfirmasi!", "success");
    await fetchOrders();
    if (selectedOrder.value && selectedOrder.value.id === order.id) {
      selectedOrder.value.status = "confirmed";
    }
  } catch (err: any) {
    showLocalToast(
      err.data?.message || "Gagal mengonfirmasi pesanan.",
      "error",
    );
  }
};

const openPayModal = (order: any) => {
  activePayOrder.value = order;
  cashInput.value = 0;
  showPayModal.value = true;
};

const processPayment = async () => {
  if (!activePayOrder.value || !cashInput.value) return;

  submittingPay.value = true;
  try {
    const res = await api.put(
      `/api/admin/orders/${activePayOrder.value.id}/pay`,
      {
        cash: cashInput.value,
      },
    );
    showPayModal.value = false;
    showLocalToast(
      res.message ||
        "Pembayaran berhasil diselesaikan! Struk / Invoice dapat segera diserahkan kepada pelanggan.",
      "success",
    );
    await fetchOrders();
    if (
      selectedOrder.value &&
      selectedOrder.value.id === activePayOrder.value.id
    ) {
      selectedOrder.value.status = "completed";
      selectedOrder.value.cash = cashInput.value;
    }
  } catch (err: any) {
    showLocalToast(err.data?.message || "Gagal memproses pembayaran.", "error");
  } finally {
    submittingPay.value = false;
  }
};

// Menutup semua overlay (modal bayar, modal kamera, banner) sebelum print,
// supaya yang tercetak hanya kartu invoice (.order-card), bukan overlay lain.
const printReceipt = () => {
  showPayModal.value = false;
  showCameraModal.value = false;
  scanSuccessBanner.value = false;
  nextTick(() => {
    window.print();
  });
};

const downloadInvoicePdf = (orderId: number) => {
  const url = `${api.apiBase}/api/orders/${orderId}/invoice-pdf`;
  window.open(url, "_blank");
};

// Global listener for USB hardware QR code scanner
let scannerBuffer = "";
let lastKeyTime = Date.now();

const onGlobalKeydown = (e: KeyboardEvent) => {
  const currentTime = Date.now();
  // Hardware scanners type very rapidly (< 50ms per key)
  if (currentTime - lastKeyTime > 100) {
    scannerBuffer = "";
  }
  lastKeyTime = currentTime;

  if (e.key === "Enter") {
    if (scannerBuffer.length > 2) {
      scanQuery.value = scannerBuffer;
      handleScanSubmit();
      scannerBuffer = "";
    }
  } else if (e.key.length === 1) {
    scannerBuffer += e.key;
  }
};

onMounted(() => {
  fetchOrders();
  if (process.client) {
    window.addEventListener("keydown", onGlobalKeydown);
  }
});

// Watch filterStatus to reset pagination
watch(filterStatus, () => {
  currentPage.value = 1;
});

onUnmounted(() => {
  stopCameraScanner();
  if (process.client) {
    window.removeEventListener("keydown", onGlobalKeydown);
  }
});
</script>

<style scoped>
/* Semua warna mengikuti token di main.css, jadi mode terang dan gelap sinkron */
.page {
  --danger: #b0394f;
  --success: #4d6b3f;
  --warn: #8a6d1f;
  --info: #505168;
  color: var(--text);
}

:global(:root.dark) .page,
:global(:root[data-theme="dark"]) .page {
  --danger: #e68a9a;
  --success: #b3c0a4;
  --warn: #dcc48e;
  --info: #cbd2ba;
}

.muted,
.field-icon {
  color: var(--muted);
}

.cell-strong {
  color: var(--text);
}

.label {
  color: var(--text-2);
}

/* toast lokal */
.toast-stack {
  pointer-events: none;
}

.toast {
  pointer-events: auto;
  background-color: var(--surface);
  color: var(--text);
  border: 1px solid var(--border);
  border-left: 3px solid var(--success);
  border-radius: var(--radius-md);
}

.toast--success {
  border-left-color: var(--success);
}

.toast--success svg {
  color: var(--success);
}

.toast--error {
  border-left-color: var(--danger);
}

.toast--error svg {
  color: var(--danger);
}

.toast-fade-enter-active,
.toast-fade-leave-active {
  transition: all 0.2s ease;
}

.toast-fade-enter-from,
.toast-fade-leave-to {
  opacity: 0;
  transform: translateY(-8px);
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

.btn-outline {
  color: var(--text);
  background-color: transparent;
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
  transition:
    background-color 0.15s ease,
    border-color 0.15s ease;
}

.btn-outline:hover {
  background-color: var(--inset);
  border-color: var(--text-2);
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

/* filter status */
.chip {
  color: var(--text-2);
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
  transition:
    background-color 0.15s ease,
    color 0.15s ease,
    border-color 0.15s ease;
}

.chip:hover {
  background-color: var(--inset);
}

.chip--active,
.chip--active:hover {
  background-color: var(--primary);
  color: var(--on-primary);
  border-color: var(--primary);
}

/* input */
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

/* kode pesanan */
.code {
  font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
  font-weight: 600;
  letter-spacing: 0.03em;
  color: var(--text);
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  padding: 0.125rem 0.375rem;
}

.code--lg {
  font-size: 1rem;
}

/* status */
.status {
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-sm);
  text-transform: capitalize;
  color: var(--text-2);
}

.status--pending {
  color: var(--warn);
  border-color: color-mix(in srgb, var(--warn) 50%, transparent);
  background-color: color-mix(in srgb, var(--warn) 10%, transparent);
}

.status--confirmed {
  color: var(--info);
  border-color: color-mix(in srgb, var(--info) 45%, transparent);
  background-color: color-mix(in srgb, var(--info) 10%, transparent);
}

.status--completed {
  color: var(--success);
  border-color: color-mix(in srgb, var(--success) 50%, transparent);
  background-color: color-mix(in srgb, var(--success) 10%, transparent);
}

/* panel dan tabel */
.panel {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

.order-title {
  font-family: var(--font-display);
  font-weight: 600;
  color: var(--text);
}

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

.tbl--bordered {
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  border-collapse: separate;
  border-spacing: 0;
  overflow: hidden;
}

.total-row,
.total-row:hover {
  background-color: var(--inset);
}

.qr-box {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

/* banner */
.banner {
  background-color: var(--surface);
  color: var(--text);
  border: 1px solid var(--border);
  border-left: 3px solid var(--success);
  border-radius: var(--radius-md);
}

.banner-icon {
  color: var(--success);
}

/* modal */
.modal-overlay {
  background-color: rgb(20 18 31 / 0.6);
}

.modal {
  background-color: var(--surface);
  color: var(--text);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

.modal-head {
  border-bottom: 1px solid var(--border);
}

.modal-title {
  font-family: var(--font-display);
  font-weight: 600;
  color: var(--text);
}

.video-box {
  background-color: #14121f;
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
}

.scan-frame {
  border: 2px solid var(--accent);
  border-radius: var(--radius-md);
}

.scan-hint {
  background-color: rgb(20 18 31 / 0.8);
  color: #eaefd3;
  border-radius: var(--radius-sm);
}

.total-box {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
}

.total-value {
  font-weight: 600;
  color: var(--text);
}

.change-box {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  color: var(--text-2);
}

.change-ok {
  color: var(--success);
}

.change-neg {
  color: var(--danger);
}

/* paginasi */
.pager {
  border-top: 1px solid var(--border);
}

.page-btn {
  color: var(--text-2);
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
  transition:
    background-color 0.15s ease,
    color 0.15s ease;
}

.page-btn:hover:not(:disabled) {
  background-color: var(--inset);
}

.page-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.page-btn--active,
.page-btn--active:hover:not(:disabled) {
  background-color: var(--primary);
  color: var(--on-primary);
  border-color: var(--primary);
}

@media print {
  :global(body *) {
    visibility: hidden;
  }
  .order-card,
  .order-card * {
    visibility: visible !important;
  }
  .order-card {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
  }
  /* struk selalu hitam di atas putih, apa pun tema yang aktif */
  .order-card,
  .order-card * {
    color: #000 !important;
    background: #fff !important;
    border-color: #000 !important;
    box-shadow: none !important;
  }
}
</style>
