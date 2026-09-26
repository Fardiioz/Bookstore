<template>
  <div class="page max-w-4xl mx-auto space-y-6">
    <!-- Header Scan / Cari Pesanan -->
    <div
      class="scan-bar p-5 flex flex-col md:flex-row items-stretch md:items-center justify-between gap-4"
    >
      <div class="flex items-center gap-2 w-full md:w-auto flex-1">
        <div class="relative w-full max-w-md">
          <input
            v-model="scanQuery"
            type="text"
            placeholder="Scan QR Code / Masukkan Kode (A021)..."
            class="field w-full pl-10 pr-4 py-3 text-sm"
            @keyup.enter="handleUserScanSubmit"
          />
          <LucideQrCode class="field-icon w-4 h-4 absolute left-3.5 top-3.5" />
        </div>
        <button
          type="button"
          @click="startCameraScanner"
          class="btn-outline inline-flex items-center gap-1.5 px-4 py-3 text-sm font-semibold whitespace-nowrap"
        >
          <LucideCamera class="w-4 h-4" />
          Scan Kamera
        </button>
      </div>

      <div class="count-box text-xs font-medium px-3.5 py-2">
        Total: {{ orders.length }} Pesanan
      </div>
    </div>

    <!-- Modal Scanner Kamera -->
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
              >Arahkan QR ke sini</span
            >
          </div>
        </div>

        <p class="muted text-xs">
          Arahkan kamera HP ke QR Code pesanan untuk memindai otomatis.
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
            Scan QR Code berhasil terdeteksi
          </h3>
          <p class="muted text-xs mt-1">
            Kode pesanan:
            <span class="code">{{ scanSuccessCode }}</span> ditemukan dalam
            riwayat Anda. Status berhasil diperbarui.
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

    <div v-if="loading" class="muted text-center py-16 text-sm">
      Memuat riwayat pesanan...
    </div>

    <div v-else-if="orders.length === 0" class="empty-panel text-center py-16">
      <div
        class="empty-icon w-16 h-16 flex items-center justify-center mx-auto mb-3"
      >
        <LucidePackage class="w-7 h-7" :stroke-width="1.5" />
      </div>
      <h3 class="empty-title font-semibold text-base">Belum Ada Pesanan</h3>
      <p class="muted text-sm mt-1 mb-6">Anda belum melakukan pesanan buku.</p>
      <NuxtLink
        to="/user/katalog"
        class="btn-primary px-5 py-3 text-sm font-semibold inline-flex items-center gap-2"
      >
        Mulai Belanja
        <LucideArrowRight class="w-4 h-4" />
      </NuxtLink>
    </div>

    <div v-else class="space-y-6 print:space-y-0">
      <div
        v-for="order in orders"
        :key="order.id"
        :id="'order-card-' + order.kode_pesanan"
        class="order-card space-y-4 transition-all duration-300"
        :class="{
          'order-card--highlight': highlightedOrderCode === order.kode_pesanan,
        }"
      >
        <div class="panel p-6 relative">
          <div class="flex flex-wrap items-center justify-between gap-3 mb-4">
            <h2 class="order-title text-xl">
              Kode Pesanan:
              <span class="code code--lg uppercase">{{
                order.kode_pesanan
              }}</span>
            </h2>
            <span
              :class="getStatusBadgeClass(order.status)"
              class="status px-2.5 py-0.5 text-[11px] font-medium"
            >
              {{ order.status }}
            </span>
          </div>

          <!-- Petunjuk pembeli -->
          <div class="notice p-3 mb-4 text-xs flex items-start gap-2">
            <LucideMegaphone class="w-4 h-4 shrink-0 mt-0.5" />
            <span
              >Tunjukkan QR Code di bawah kepada kasir/admin. Setelah di-scan
              dan berhasil dicatat, mohon tunggu struk atau invoice diserahkan
              langsung oleh admin.</span
            >
          </div>

          <!-- Rincian + QR -->
          <div class="grid grid-cols-1 lg:grid-cols-4 gap-6 items-center">
            <div class="lg:col-span-3 overflow-x-auto">
              <!-- Header - hidden on mobile -->
              <div
                class="tbl-head hidden sm:grid grid-cols-[2fr_1fr_0.6fr_1fr_1fr] gap-2 text-[11px] font-semibold py-3 px-4"
              >
                <span>Judul Buku</span>
                <span class="text-center">Tanggal</span>
                <span class="text-right">Jumlah</span>
                <span class="text-right">Harga Satuan</span>
                <span class="text-right">Total</span>
              </div>

              <div class="tbl-body">
                <div
                  v-for="detail in order.details || order.items || []"
                  :key="detail.id"
                  class="tbl-row grid grid-cols-2 sm:grid-cols-[2fr_1fr_0.6fr_1fr_1fr] gap-x-2 gap-y-1.5 p-4 text-xs"
                >
                  <span
                    class="col-span-2 sm:col-span-1 cell-strong font-semibold"
                  >
                    {{ detail.buku?.judul || detail.nama_buku }}
                  </span>

                  <span class="sm:text-center muted">
                    <span class="sm:hidden font-semibold">Tanggal: </span
                    >{{ formatDate(order.created_at) }}
                  </span>

                  <span
                    class="sm:text-right cell-strong font-semibold tabular-nums"
                  >
                    <span class="sm:hidden muted font-normal">Jumlah: </span
                    >{{ detail.qty }}
                  </span>

                  <span class="sm:text-right muted tabular-nums">
                    <span class="sm:hidden font-semibold">Harga: </span>Rp
                    {{ formatPrice(detail.harga_satuan) }}
                  </span>

                  <span
                    class="sm:text-right cell-strong font-semibold tabular-nums"
                  >
                    <span class="sm:hidden muted font-normal">Total: </span>Rp
                    {{ formatPrice(detail.subtotal) }}
                  </span>
                </div>

                <!-- Baris total -->
                <div class="total-row flex justify-between items-center p-4">
                  <span class="text-sm font-semibold">Total Harga</span>
                  <span class="text-base font-semibold tabular-nums"
                    >Rp {{ formatPrice(order.total_harga) }}</span
                  >
                </div>
              </div>
            </div>

            <!-- QR Code -->
            <div
              class="qr-box lg:col-span-1 flex flex-col items-center justify-center p-2"
            >
              <QrCodeDisplay
                :value="order.kode_pesanan"
                :size="150"
                show-label
              />
              <p class="muted text-[11px] text-center mt-2">
                Tunjukkan QR ke kasir
              </p>
            </div>
          </div>
        </div>

        <!-- Aksi -->
        <div class="flex flex-col sm:flex-row gap-3 print:hidden flex-wrap">
          <button
            type="button"
            @click="openDetailModal(order)"
            class="btn-outline flex-1 inline-flex items-center justify-center gap-2 py-3 text-sm font-semibold"
          >
            <LucideInfo class="w-4 h-4" />
            <span>Lihat Detail Pesanan</span>
          </button>

          <button
            type="button"
            @click="openReceiptModal(order)"
            class="btn-outline flex-1 inline-flex items-center justify-center gap-2 py-3 text-sm font-semibold"
          >
            <LucideReceipt class="w-4 h-4" />
            <span>Lihat Struk</span>
          </button>

          <button
            type="button"
            @click="downloadInvoicePdf(order.id)"
            :disabled="order.status !== 'completed'"
            class="btn-outline flex-1 inline-flex items-center justify-center gap-2 py-3 text-sm font-semibold disabled:opacity-50 disabled:cursor-not-allowed"
          >
            <LucideFileText class="w-4 h-4" />
            <span>{{
              order.status === "completed"
                ? "Unduh Invoice PDF"
                : "Tunggu Status Selesai"
            }}</span>
          </button>

          <button
            type="button"
            @click="openReviewModal(order)"
            :disabled="order.status !== 'completed'"
            class="btn-outline flex-1 inline-flex items-center justify-center gap-2 py-3 text-sm font-semibold disabled:opacity-50 disabled:cursor-not-allowed"
          >
            <LucideStar class="w-4 h-4" />
            <span>{{
              order.status === "completed"
                ? "Beri Rating & Ulasan"
                : "Tunggu Status Selesai"
            }}</span>
          </button>
        </div>
      </div>
    </div>

    <!-- Modal Detail Pesanan -->
    <div
      v-if="showDetailModal"
      class="modal-overlay fixed inset-0 z-50 flex items-center justify-center p-4"
    >
      <div
        class="modal p-6 max-w-lg w-full space-y-4 max-h-[90vh] overflow-y-auto"
        role="dialog"
        aria-modal="true"
      >
        <div class="modal-head flex justify-between items-center pb-3">
          <h3 class="modal-title text-lg flex items-center gap-2">
            <LucideInfo class="w-4 h-4" />
            Detail Pesanan
          </h3>
          <button
            type="button"
            @click="closeDetailModal()"
            class="icon-action"
            aria-label="Tutup"
          >
            <LucideX class="w-4 h-4" />
          </button>
        </div>

        <div v-if="selectedDetailOrder" class="space-y-4">
          <!-- Info umum -->
          <div class="order-info p-3 text-sm space-y-1.5">
            <div class="flex justify-between gap-3">
              <span class="muted">Kode Pesanan</span>
              <span class="code">{{ selectedDetailOrder.kode_pesanan }}</span>
            </div>
            <div class="flex justify-between gap-3">
              <span class="muted">Tanggal Pesan</span>
              <span class="cell-strong font-medium">{{
                formatDate(selectedDetailOrder.created_at)
              }}</span>
            </div>
            <div class="flex justify-between gap-3 items-center">
              <span class="muted">Status</span>
              <span
                :class="getStatusBadgeClass(selectedDetailOrder.status)"
                class="status px-2.5 py-0.5 text-[11px] font-medium"
                >{{ selectedDetailOrder.status }}</span
              >
            </div>
          </div>

          <!-- Daftar buku -->
          <div class="space-y-2">
            <h4 class="label font-semibold text-sm">Buku Dipesan</h4>
            <div class="tbl-body">
              <div
                v-for="detail in selectedDetailOrder.details ||
                selectedDetailOrder.items ||
                []"
                :key="detail.id"
                class="tbl-row p-3 text-xs space-y-1"
              >
                <div class="cell-strong font-semibold text-sm">
                  {{ detail.buku?.judul || detail.nama_buku }}
                </div>
                <div class="flex justify-between muted tabular-nums">
                  <span
                    >{{ detail.qty }} x Rp
                    {{ formatPrice(detail.harga_satuan) }}</span
                  >
                  <span class="cell-strong font-semibold"
                    >Rp {{ formatPrice(detail.subtotal) }}</span
                  >
                </div>
              </div>
              <div class="total-row flex justify-between items-center p-3">
                <span class="text-sm font-semibold">Total Harga</span>
                <span class="text-base font-semibold tabular-nums"
                  >Rp {{ formatPrice(selectedDetailOrder.total_harga) }}</span
                >
              </div>
            </div>
          </div>
        </div>

        <div class="modal-actions flex gap-3 pt-4">
          <button
            type="button"
            @click="closeDetailModal()"
            class="btn-ghost flex-1 py-2.5 text-sm font-semibold"
          >
            Tutup
          </button>
        </div>
      </div>
    </div>

    <!-- Modal Struk -->
    <div
      v-if="showReceiptModal"
      class="modal-overlay fixed inset-0 z-50 flex items-center justify-center p-4"
    >
      <div
        class="modal p-6 max-w-sm w-full space-y-4 max-h-[90vh] overflow-y-auto"
        role="dialog"
        aria-modal="true"
      >
        <div
          class="modal-head flex justify-between items-center pb-3 print:hidden"
        >
          <h3 class="modal-title text-lg flex items-center gap-2">
            <LucideReceipt class="w-4 h-4" />
            Struk Pesanan
          </h3>
          <button
            type="button"
            @click="closeReceiptModal()"
            class="icon-action"
            aria-label="Tutup"
          >
            <LucideX class="w-4 h-4" />
          </button>
        </div>

        <div
          v-if="selectedReceiptOrder"
          id="struk-content"
          class="struk space-y-3 text-xs"
        >
          <div class="text-center space-y-0.5">
            <p class="font-semibold text-sm">TOKO BUKU</p>
            <p class="muted">Struk Pembelian</p>
          </div>

          <div class="struk-divider"></div>

          <div class="space-y-1">
            <div class="flex justify-between">
              <span class="muted">Kode</span>
              <span class="code">{{ selectedReceiptOrder.kode_pesanan }}</span>
            </div>
            <div class="flex justify-between">
              <span class="muted">Tanggal</span>
              <span>{{ formatDate(selectedReceiptOrder.created_at) }}</span>
            </div>
            <div class="flex justify-between">
              <span class="muted">Status</span>
              <span class="capitalize">{{ selectedReceiptOrder.status }}</span>
            </div>
          </div>

          <div class="struk-divider"></div>

          <div class="space-y-2">
            <div
              v-for="detail in selectedReceiptOrder.details ||
              selectedReceiptOrder.items ||
              []"
              :key="detail.id"
              class="space-y-0.5"
            >
              <div class="font-medium">
                {{ detail.buku?.judul || detail.nama_buku }}
              </div>
              <div class="flex justify-between muted tabular-nums">
                <span
                  >{{ detail.qty }} x Rp
                  {{ formatPrice(detail.harga_satuan) }}</span
                >
                <span>Rp {{ formatPrice(detail.subtotal) }}</span>
              </div>
            </div>
          </div>

          <div class="struk-divider"></div>

          <div class="flex justify-between font-semibold text-sm">
            <span>Total</span>
            <span>Rp {{ formatPrice(selectedReceiptOrder.total_harga) }}</span>
          </div>

          <div class="struk-divider"></div>

          <p class="text-center muted">Terima kasih atas pembelian Anda</p>
        </div>

        <div class="modal-actions flex gap-3 pt-4 print:hidden">
          <button
            type="button"
            @click="closeReceiptModal()"
            class="btn-ghost flex-1 py-2.5 text-sm font-semibold"
          >
            Tutup
          </button>
          <button
            type="button"
            @click="printReceipt()"
            class="btn-primary flex-1 py-2.5 text-sm font-semibold"
          >
            Cetak Struk
          </button>
        </div>
      </div>
    </div>

    <!-- Modal Ulasan -->
    <div
      v-if="showReviewModal"
      class="modal-overlay fixed inset-0 z-50 flex items-center justify-center p-4"
    >
      <div
        class="modal p-6 max-w-md w-full space-y-4 max-h-[90vh] overflow-y-auto"
        role="dialog"
        aria-modal="true"
      >
        <div class="modal-head flex justify-between items-center pb-3">
          <h3 class="modal-title text-lg flex items-center gap-2">
            <LucideStar class="w-4 h-4" />
            Rating & Ulasan
          </h3>
          <button
            type="button"
            @click="closeReviewModal()"
            class="icon-action"
            aria-label="Tutup"
          >
            <LucideX class="w-4 h-4" />
          </button>
        </div>

        <!-- Info pesanan -->
        <div class="order-info p-3 text-sm">
          Kode Pesanan:
          <span class="code">{{ selectedOrder?.kode_pesanan }}</span>
        </div>

        <!-- Rating -->
        <div class="space-y-2">
          <label class="label block font-semibold text-sm"
            >Berikan Rating</label
          >
          <div class="flex gap-1">
            <button
              v-for="star in [1, 2, 3, 4, 5]"
              :key="star"
              type="button"
              @click="reviewForm.rating = star"
              class="star-btn w-9 h-9 flex items-center justify-center"
              :aria-label="`${star} bintang`"
            >
              <LucideStar
                class="w-5 h-5"
                :class="
                  star <= reviewForm.rating ? 'star-filled' : 'star-empty'
                "
                :fill="star <= reviewForm.rating ? 'currentColor' : 'none'"
              />
            </button>
          </div>
          <p class="muted text-xs">{{ reviewForm.rating }} dari 5 bintang</p>
        </div>

        <!-- Pilih buku -->
        <div class="space-y-2">
          <label class="label block font-semibold text-sm">Pilih Buku</label>
          <div class="space-y-2 max-h-48 overflow-y-auto">
            <button
              v-for="detail in selectedOrder?.details ||
              selectedOrder?.items ||
              []"
              :key="detail.id"
              type="button"
              @click="reviewForm.book_id = detail.book_id"
              class="book-pick w-full text-left p-3 text-sm font-medium flex items-center gap-2"
              :class="{
                'book-pick--active': reviewForm.book_id === detail.book_id,
              }"
            >
              <LucideBookOpen class="w-4 h-4 shrink-0" />
              {{ detail.buku?.nama_buku || detail.nama_buku }}
            </button>
          </div>
        </div>

        <!-- Komentar -->
        <div class="space-y-2">
          <label class="label block font-semibold text-sm"
            >Komentar (opsional)</label
          >
          <textarea
            v-model="reviewForm.komentar"
            placeholder="Tuliskan pengalaman Anda dengan buku ini..."
            class="field w-full p-3 text-sm resize-none h-20"
          ></textarea>
          <p class="muted text-xs">
            {{ reviewForm.komentar?.length || 0 }} / 1000 karakter
          </p>
        </div>

        <!-- Aksi -->
        <div class="modal-actions flex gap-3 pt-4">
          <button
            type="button"
            @click="closeReviewModal()"
            class="btn-ghost flex-1 py-2.5 text-sm font-semibold"
          >
            Batal
          </button>
          <button
            type="button"
            @click="submitReview"
            :disabled="
              !reviewForm.rating || !reviewForm.book_id || submittingReview
            "
            class="btn-primary flex-1 py-2.5 text-sm font-semibold disabled:opacity-50 disabled:cursor-not-allowed"
          >
            {{ submittingReview ? "Mengirim..." : "Kirim Ulasan" }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  FileText as LucideFileText,
  QrCode as LucideQrCode,
  Camera as LucideCamera,
  X as LucideX,
  CheckCircle2 as LucideCheckCircle2,
  Package as LucidePackage,
  ArrowRight as LucideArrowRight,
  Megaphone as LucideMegaphone,
  Star as LucideStar,
  BookOpen as LucideBookOpen,
  Info as LucideInfo,
  Receipt as LucideReceipt,
} from "lucide-vue-next";
import jsQR from "jsqr";

definePageMeta({
  middleware: "auth",
});

const api = useApi();
const orders = ref<any[]>([]);
const loading = ref(true);
let pollTimer: any = null;

const scanQuery = ref("");
const scanSuccessBanner = ref(false);
const scanSuccessCode = ref("");
const highlightedOrderCode = ref("");

// Camera scanner state
const showCameraModal = ref(false);
const videoRef = ref<HTMLVideoElement | null>(null);
const canvasRef = ref<HTMLCanvasElement | null>(null);
let cameraStream: MediaStream | null = null;
let animationFrameId: number | null = null;

// Review modal state
const showReviewModal = ref(false);
const selectedOrder = ref<any>(null);
const submittingReview = ref(false);
const reviewForm = ref({
  rating: 5,
  book_id: null as number | null,
  komentar: "",
});

// Detail pesanan modal state
const showDetailModal = ref(false);
const selectedDetailOrder = ref<any>(null);

// Struk modal state
const showReceiptModal = ref(false);
const selectedReceiptOrder = ref<any>(null);

const formatPrice = (val: number) => {
  return new Intl.NumberFormat("id-ID").format(val || 0);
};

const formatDate = (dateStr: string) => {
  if (!dateStr) return "21/4/2026";
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

const downloadInvoicePdf = (orderId: number) => {
  const url = `${api.apiBase}/api/orders/${orderId}/invoice-pdf`;
  window.open(url, "_blank");
};

// --- Detail Pesanan ---
const openDetailModal = (order: any) => {
  selectedDetailOrder.value = order;
  showDetailModal.value = true;
};

const closeDetailModal = () => {
  showDetailModal.value = false;
  selectedDetailOrder.value = null;
};

// --- Struk ---
const openReceiptModal = (order: any) => {
  selectedReceiptOrder.value = order;
  showReceiptModal.value = true;
};

const closeReceiptModal = () => {
  showReceiptModal.value = false;
  selectedReceiptOrder.value = null;
};

const printReceipt = () => {
  if (!process.client) return;
  const content = document.getElementById("struk-content");
  if (!content) return;

  const printWindow = window.open("", "_blank", "width=380,height=600");
  if (!printWindow) return;

  printWindow.document.write(`
    <html>
      <head>
        <title>Struk - ${selectedReceiptOrder.value?.kode_pesanan || ""}</title>
        <style>
          body { font-family: ui-monospace, Menlo, Consolas, monospace; font-size: 12px; color: #000; padding: 16px; }
          .flex { display: flex; }
          .justify-between { justify-content: space-between; }
          .text-center { text-align: center; }
          .font-semibold { font-weight: 600; }
          .font-medium { font-weight: 500; }
          .space-y-3 > * + * { margin-top: 8px; }
          .space-y-2 > * + * { margin-top: 6px; }
          .space-y-1 > * + * { margin-top: 3px; }
          .space-y-0\\.5 > * + * { margin-top: 2px; }
          .muted { color: #555; }
          .struk-divider { border-top: 1px dashed #999; margin: 8px 0; }
          .text-sm { font-size: 13px; }
        </style>
      </head>
      <body>${content.innerHTML}</body>
    </html>
  `);
  printWindow.document.close();
  printWindow.focus();
  printWindow.onload = () => {
    printWindow.print();
    printWindow.close();
  };
};

const openReviewModal = (order: any) => {
  selectedOrder.value = order;
  // Set first book as default
  if (order.details && order.details.length > 0) {
    reviewForm.value.book_id = order.details[0].book_id;
  } else if (order.items && order.items.length > 0) {
    reviewForm.value.book_id = order.items[0].book_id;
  }
  reviewForm.value.rating = 5;
  reviewForm.value.komentar = "";
  showReviewModal.value = true;
};

const closeReviewModal = () => {
  showReviewModal.value = false;
  selectedOrder.value = null;
  reviewForm.value = {
    rating: 5,
    book_id: null,
    komentar: "",
  };
};

const submitReview = async () => {
  if (!reviewForm.value.rating || !reviewForm.value.book_id) {
    const toast = useToast();
    toast.error("Rating dan pilih buku terlebih dahulu");
    return;
  }

  submittingReview.value = true;
  try {
    const toast = useToast();
    await api.post("/api/reviews", {
      book_id: reviewForm.value.book_id,
      rating: reviewForm.value.rating,
      komentar: reviewForm.value.komentar || null,
    });
    toast.success(
      "Ulasan berhasil ditambahkan! Terima kasih atas penilaian Anda.",
    );
    closeReviewModal();
  } catch (err: any) {
    const toast = useToast();
    toast.error(err.data?.message || "Gagal menambahkan ulasan");
  } finally {
    submittingReview.value = false;
  }
};

const fetchOrders = async (silent = false) => {
  if (!silent) loading.value = true;
  try {
    const res = await api.get("/api/orders");
    orders.value = res.data || [];
  } catch (e) {
    console.error(e);
  } finally {
    if (!silent) loading.value = false;
  }
};

const handleUserScanSubmit = async () => {
  const query = scanQuery.value.trim().toLowerCase();
  if (!query) return;

  try {
    const res = await api.post("/api/orders/scan", { kode_pesanan: query });
    const matched = res.data;

    scanSuccessCode.value = matched.kode_pesanan;
    scanSuccessBanner.value = true;
    highlightedOrderCode.value = matched.kode_pesanan;
    scanQuery.value = "";

    // Play audio beep sound
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
        // Ignore audio errors
      }
    }

    // Scroll to order element
    if (process.client) {
      setTimeout(() => {
        const el = document.getElementById(
          "order-card-" + matched.kode_pesanan,
        );
        if (el) el.scrollIntoView({ behavior: "smooth", block: "center" });
      }, 100);
    }

    const toast = useToast();
    toast.success(
      `Scan Berhasil & Status Diperbarui!\n\nKode Pesanan: ${matched.kode_pesanan}\nStatus Terbaru DB: ${matched.status.toUpperCase()}\nTotal: Rp. ${formatPrice(matched.total_harga)}\n\nStatus pesanan berhasil diubah menjadi 'CONFIRMED'.`,
    );
    await fetchOrders();
  } catch (err: any) {
    const toast = useToast();
    toast.error(
      err.data?.message ||
        `Pesanan dengan kode "${scanQuery.value}" tidak ditemukan dalam Riwayat Anda!`,
    );
  }
};

// Camera Scanner Logic for User Page
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
    const toast = useToast();
    toast.error(
      "Gagal mengakses kamera. Pastikan izin kamera telah diberikan di browser.",
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
        handleUserScanSubmit();
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

// Global listener for USB hardware QR code scanner on User page
let scannerBuffer = "";
let lastKeyTime = Date.now();

const onGlobalKeydown = (e: KeyboardEvent) => {
  const currentTime = Date.now();
  if (currentTime - lastKeyTime > 100) {
    scannerBuffer = "";
  }
  lastKeyTime = currentTime;

  if (e.key === "Enter") {
    if (scannerBuffer.length > 2) {
      scanQuery.value = scannerBuffer;
      handleUserScanSubmit();
      scannerBuffer = "";
    }
  } else if (e.key.length === 1) {
    scannerBuffer += e.key;
  }
};

onMounted(() => {
  fetchOrders();
  // Poll every 5s so when admin scans & updates status, user sees update live
  pollTimer = setInterval(() => {
    fetchOrders(true);
  }, 5000);

  if (process.client) {
    window.addEventListener("keydown", onGlobalKeydown);
  }
});

onUnmounted(() => {
  if (pollTimer) clearInterval(pollTimer);
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

/* header scan */
.scan-bar {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

.count-box {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  color: var(--text-2);
  white-space: nowrap;
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
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
  background: transparent;
  transition:
    background-color 0.15s ease,
    border-color 0.15s ease;
}

.btn-outline:hover:not(:disabled) {
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
  transition: background-color 0.15s ease;
}

.icon-action:hover {
  background-color: var(--inset);
  color: var(--text);
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

/* kartu pesanan */
.panel {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

.order-card--highlight .panel {
  border-color: var(--accent);
  box-shadow: 0 0 0 2px color-mix(in srgb, var(--accent) 50%, transparent);
}

.order-title {
  font-family: var(--font-display);
  font-weight: 600;
  color: var(--text);
}

.notice {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  color: var(--text-2);
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

/* tabel rincian */
.tbl-head {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-bottom: none;
  border-radius: var(--radius-md) var(--radius-md) 0 0;
  color: var(--muted);
}

.tbl-body {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  overflow: hidden;
}

.tbl-row {
  border-top: 1px solid var(--border);
  transition: background-color 0.15s ease;
}

.tbl-row:first-child {
  border-top: 0;
}

.tbl-row:hover {
  background-color: var(--inset);
}

.total-row {
  background-color: var(--inset);
  border-top: 1px solid var(--border);
}

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

.qr-box {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
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

.modal-actions {
  border-top: 1px solid var(--border);
}

.order-info {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  color: var(--text-2);
}

/* struk */
.struk {
  color: var(--text);
}

.struk-divider {
  border-top: 1px dashed var(--border-strong);
}

/* bintang rating */
.star-btn {
  color: var(--muted);
  transition: transform 0.1s ease;
}

.star-btn:hover {
  transform: scale(1.08);
}

.star-filled {
  color: var(--accent);
}

.star-empty {
  color: var(--border-strong);
}

/* pilih buku */
.book-pick {
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
  color: var(--text-2);
  background-color: var(--bg);
  transition:
    background-color 0.15s ease,
    border-color 0.15s ease;
}

.book-pick:hover {
  background-color: var(--inset);
}

.book-pick--active {
  background-color: var(--primary);
  color: var(--on-primary);
  border-color: var(--primary);
}

/* video kamera */
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

@media print {
  body * {
    visibility: hidden;
  }
  .order-card,
  .order-card * {
    visibility: visible;
  }
  .order-card {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
  }
  .order-card,
  .order-card * {
    color: #000 !important;
    background: #fff !important;
    border-color: #000 !important;
    box-shadow: none !important;
  }
}
</style>
