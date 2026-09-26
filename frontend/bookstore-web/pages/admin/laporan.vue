<template>
  <div class="page space-y-6">
    <!-- Header -->
    <div>
      <h1 class="page-title">Laporan Penjualan</h1>
      <p class="page-sub text-sm mt-1">
        Lihat detail transaksi penjualan buku berdasarkan tahun
      </p>
    </div>

    <!-- Top Filter Bar -->
    <div class="flex flex-col sm:flex-row items-center justify-between gap-4">
      <!-- Search Box -->
      <div class="relative w-full sm:w-96">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Cari kode pesanan atau judul buku..."
          class="field w-full pl-10 pr-4 py-2.5 text-sm"
        />
        <LucideSearch class="field-icon w-4 h-4 absolute left-3.5 top-3" />
      </div>

      <!-- Year Picker -->
      <div
        class="year-box flex items-center gap-3 px-4 py-2.5 text-sm font-semibold"
      >
        <input
          v-model.number="filterYear"
          type="number"
          placeholder="2026"
          class="year-input bg-transparent text-center w-16 outline-none font-semibold"
          @change="fetchReport"
        />
        <LucideCalendar class="w-4 h-4" />
      </div>
    </div>

    <!-- Table Container -->
    <div class="table-wrap overflow-hidden">
      <div v-if="loading" class="state-text text-center py-14 text-sm">
        Memuat laporan...
      </div>
      <div
        v-else-if="filteredRows.length === 0"
        class="state-text text-center py-14 text-sm"
      >
        Tidak ada data transaksi.
      </div>
      <div v-else class="overflow-x-auto">
        <table class="tbl w-full text-left text-xs">
          <thead>
            <tr>
              <th class="px-4 py-3">Kode Pesanan</th>
              <th class="px-4 py-3">Judul Buku</th>
              <th class="px-4 py-3 text-center">Tanggal</th>
              <th class="px-4 py-3 text-center">Jumlah</th>
              <th class="px-4 py-3 text-right">Total Harga</th>
              <th class="px-4 py-3 text-center">Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, idx) in paginatedRows" :key="idx">
              <td class="px-4 py-3">
                <span class="code uppercase">{{ item.kode_pesanan }}</span>
              </td>
              <td class="cell-strong px-4 py-3 font-semibold">
                {{ item.judul_buku }}
              </td>
              <td class="cell-muted px-4 py-3 text-center">
                {{ item.tanggal }}
              </td>
              <td
                class="cell-strong px-4 py-3 text-center tabular-nums font-semibold"
              >
                {{ item.jumlah }}
              </td>
              <td
                class="cell-strong px-4 py-3 text-right tabular-nums font-semibold"
              >
                Rp {{ formatPrice(item.total_harga) }}
              </td>
              <td class="px-4 py-3">
                <div class="flex items-center justify-center gap-1.5">
                  <button
                    type="button"
                    @click="openDetailModal(item.order)"
                    class="row-action inline-flex items-center justify-center w-7 h-7"
                    title="Lihat Detail Pesanan"
                  >
                    <LucideInfo class="w-3.5 h-3.5" />
                  </button>
                  <button
                    type="button"
                    @click="openReceiptModal(item.order)"
                    class="row-action inline-flex items-center justify-center w-7 h-7"
                    title="Lihat Struk"
                  >
                    <LucideReceipt class="w-3.5 h-3.5" />
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Pagination Controls -->
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

          <span class="state-text text-xs ml-2">
            Hal {{ currentPage }} dari {{ totalPages }}
          </span>
        </div>
      </div>
    </div>

    <!-- Download Button -->
    <div class="flex justify-end gap-3">
      <a
        :href="excelExportUrl"
        target="_blank"
        class="btn-outline inline-flex items-center gap-1.5 px-4 py-2.5 text-sm font-semibold"
      >
        <LucideFileSpreadsheet class="w-4 h-4" />
        Download Excel
      </a>

      <a
        :href="pdfExportUrl"
        target="_blank"
        class="btn-primary inline-flex items-center gap-1.5 px-4 py-2.5 text-sm font-semibold"
      >
        <LucideFileDown class="w-4 h-4" />
        Download PDF
      </a>
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
            <div
              v-if="selectedDetailOrder.pelanggan"
              class="flex justify-between gap-3"
            >
              <span class="muted">Pelanggan</span>
              <span class="cell-strong font-medium">{{
                selectedDetailOrder.pelanggan
              }}</span>
            </div>
            <div class="flex justify-between gap-3">
              <span class="muted">Tanggal Pesan</span>
              <span class="cell-strong font-medium">{{
                formatDate(selectedDetailOrder.created_at)
              }}</span>
            </div>
            <div
              v-if="selectedDetailOrder.status"
              class="flex justify-between gap-3 items-center"
            >
              <span class="muted">Status</span>
              <span
                class="status px-2.5 py-0.5 text-[11px] font-medium capitalize"
                >{{ selectedDetailOrder.status }}</span
              >
            </div>
          </div>

          <!-- Daftar buku -->
          <div class="space-y-2">
            <h4 class="label font-semibold text-sm">Buku Dipesan</h4>
            <div class="tbl-body">
              <div
                v-for="(detail, i) in getOrderDetailList(selectedDetailOrder)"
                :key="i"
                class="tbl-row p-3 text-xs space-y-1"
              >
                <div class="cell-strong font-semibold text-sm">
                  {{ detail.judul_buku }}
                </div>
                <div class="flex justify-between muted tabular-nums">
                  <span
                    >{{ detail.jumlah }} x Rp
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
            <div
              v-if="selectedReceiptOrder.pelanggan"
              class="flex justify-between"
            >
              <span class="muted">Pelanggan</span>
              <span>{{ selectedReceiptOrder.pelanggan }}</span>
            </div>
            <div class="flex justify-between">
              <span class="muted">Tanggal</span>
              <span>{{ formatDate(selectedReceiptOrder.created_at) }}</span>
            </div>
          </div>

          <div class="struk-divider"></div>

          <div class="space-y-2">
            <div
              v-for="(detail, i) in getOrderDetailList(selectedReceiptOrder)"
              :key="i"
              class="space-y-0.5"
            >
              <div class="font-medium">{{ detail.judul_buku }}</div>
              <div class="flex justify-between muted tabular-nums">
                <span
                  >{{ detail.jumlah }} x Rp
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
  </div>
</template>

<script setup lang="ts">
import {
  Search as LucideSearch,
  Calendar as LucideCalendar,
  ChevronLeft as LucideChevronLeft,
  ChevronRight as LucideChevronRight,
  ChevronsLeft as LucideChevronsLeft,
  ChevronsRight as LucideChevronsRight,
  FileSpreadsheet as LucideFileSpreadsheet,
  FileDown as LucideFileDown,
  Info as LucideInfo,
  Receipt as LucideReceipt,
  X as LucideX,
} from "lucide-vue-next";

definePageMeta({
  middleware: "admin",
});

const api = useApi();
const loading = ref(true);
const searchQuery = ref("");
const filterYear = ref(new Date().getFullYear());
const currentPage = ref(1);
const itemsPerPage = 6;

const orders = ref<any[]>([]);

// Modal detail pesanan
const showDetailModal = ref(false);
const selectedDetailOrder = ref<any>(null);

// Modal struk
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

// Ambil daftar item buku dari order (mendukung format items / orderDetails / fallback)
const getOrderDetailList = (order: any) => {
  if (!order) return [];
  if (order.items && Array.isArray(order.items) && order.items.length > 0) {
    return order.items.map((item: any) => ({
      judul_buku: item.buku?.judul || item.judul_buku || "Buku",
      jumlah: item.jumlah,
      harga_satuan: item.harga_satuan,
      subtotal: item.subtotal || item.jumlah * item.harga_satuan,
    }));
  }
  if (
    order.orderDetails &&
    Array.isArray(order.orderDetails) &&
    order.orderDetails.length > 0
  ) {
    return order.orderDetails.map((detail: any) => ({
      judul_buku: detail.book?.nama_buku || detail.judul_buku || "Buku",
      jumlah: detail.qty,
      harga_satuan: detail.harga_satuan,
      subtotal: detail.subtotal,
    }));
  }
  return [
    {
      judul_buku: order.pelanggan
        ? `Order oleh ${order.pelanggan}`
        : "Detail Buku",
      jumlah: order.total_items || 1,
      harga_satuan: order.total_harga,
      subtotal: order.total_harga,
    },
  ];
};

// --- Detail Pesanan ---
const openDetailModal = (order: any) => {
  if (!order) return;
  selectedDetailOrder.value = order;
  showDetailModal.value = true;
};

const closeDetailModal = () => {
  showDetailModal.value = false;
  selectedDetailOrder.value = null;
};

// --- Struk ---
const openReceiptModal = (order: any) => {
  if (!order) return;
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

const pdfExportUrl = computed(() => {
  let url = `${api.apiBase}/api/admin/reports/export-pdf`;
  if (filterYear.value) {
    const year = filterYear.value;
    const startDate = `${year}-01-01`;
    const endDate = `${year}-12-31`;
    url += `?start_date=${startDate}&end_date=${endDate}`;
  }
  return url;
});

const excelExportUrl = computed(() => {
  let url = `${api.apiBase}/api/admin/reports/export-excel`;
  if (filterYear.value) {
    const year = filterYear.value;
    const startDate = `${year}-01-01`;
    const endDate = `${year}-12-31`;
    url += `?start_date=${startDate}&end_date=${endDate}`;
  }
  return url;
});

const fetchReport = async () => {
  loading.value = true;
  try {
    // Build date range for the selected year
    const year = filterYear.value;
    const startDate = `${year}-01-01`;
    const endDate = `${year}-12-31`;

    const res = await api.get("/api/admin/reports", {
      params: {
        start_date: startDate,
        end_date: endDate,
      },
    });
    orders.value = res.orders || [];
    currentPage.value = 1; // Reset ke page 1 saat fetch
  } catch (e) {
    console.error(e);
  } finally {
    loading.value = false;
  }
};

// Flatten order items for table display matching mockup columns
const tableRows = computed(() => {
  const rows: any[] = [];
  orders.value.forEach((order) => {
    // Check if this is the old format (with items array) or new format (with orderDetails)
    if (order.items && Array.isArray(order.items) && order.items.length > 0) {
      // Old format with items
      order.items.forEach((item: any) => {
        rows.push({
          kode_pesanan: order.kode_pesanan,
          judul_buku: item.buku?.judul || item.judul_buku || "Buku",
          tanggal: order.created_at
            ? new Date(order.created_at).toLocaleDateString("id-ID")
            : "21/4/2026",
          jumlah: item.jumlah,
          total_harga: item.subtotal || item.jumlah * item.harga_satuan,
          order,
        });
      });
    } else if (
      order.orderDetails &&
      Array.isArray(order.orderDetails) &&
      order.orderDetails.length > 0
    ) {
      // New format with orderDetails
      order.orderDetails.forEach((detail: any) => {
        rows.push({
          kode_pesanan: order.kode_pesanan,
          judul_buku: detail.book?.nama_buku || detail.judul_buku || "Buku",
          tanggal: order.created_at
            ? new Date(order.created_at).toLocaleDateString("id-ID")
            : "21/4/2026",
          jumlah: detail.qty,
          total_harga: detail.subtotal,
          order,
        });
      });
    } else {
      // Fallback format
      rows.push({
        kode_pesanan: order.kode_pesanan,
        judul_buku: order.pelanggan
          ? `Order oleh ${order.pelanggan}`
          : "Detail Buku",
        tanggal: order.created_at
          ? new Date(order.created_at).toLocaleDateString("id-ID")
          : "21/4/2026",
        jumlah: order.total_items || 1,
        total_harga: order.total_harga,
        order,
      });
    }
  });
  return rows;
});

const filteredRows = computed(() => {
  if (!searchQuery.value.trim()) return tableRows.value;
  const q = searchQuery.value.toLowerCase();
  return tableRows.value.filter(
    (r) =>
      r.kode_pesanan.toLowerCase().includes(q) ||
      r.judul_buku.toLowerCase().includes(q),
  );
});

const paginatedRows = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return filteredRows.value.slice(start, end);
});

const totalPages = computed(() => {
  return Math.ceil(filteredRows.value.length / itemsPerPage);
});

onMounted(fetchReport);
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
.field-icon {
  color: var(--muted);
}

.cell-strong {
  color: var(--text);
}

.muted {
  color: var(--muted);
}

.label {
  color: var(--text-2);
}

/* input pencarian */
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

/* year picker */
.year-box {
  background-color: var(--inset);
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
  color: var(--text);
}

.year-input {
  color: var(--text);
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

/* aksi baris */
.row-action {
  color: var(--text-2);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  background: transparent;
  transition:
    background-color 0.15s ease,
    color 0.15s ease;
}

.row-action:hover {
  background-color: var(--inset);
  color: var(--text);
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

/* tombol download */
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

.btn-primary:hover {
  background-color: var(--primary-hover);
  border-color: var(--primary-hover);
}

.btn-primary:active {
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

/* modal (sama seperti halaman riwayat pesanan) */
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

.tbl-body {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  overflow: hidden;
}

.tbl-row {
  border-top: 1px solid var(--border);
}

.tbl-row:first-child {
  border-top: 0;
}

.total-row {
  background-color: var(--inset);
  border-top: 1px solid var(--border);
}

.status {
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-sm);
  color: var(--text-2);
}

/* struk */
.struk {
  color: var(--text);
}

.struk-divider {
  border-top: 1px dashed var(--border-strong);
}
</style>
