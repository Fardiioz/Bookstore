<template>
  <div class="page max-w-4xl mx-auto px-4 py-16">
    <div class="text-center mb-12">
      <span class="eyebrow px-3 py-1 text-xs font-semibold">TENTANG KAMI</span>
      <h1 class="page-title text-2xl sm:text-3xl mt-4 mb-3">
        Our Story - Cerita BookStore
      </h1>
      <p class="muted text-sm max-w-lg mx-auto">
        Perjalanan kami menghadirkan platform toko buku online terbaik dan
        terpercaya.
      </p>
    </div>

    <div class="panel p-8 md:p-12 space-y-6 text-sm leading-relaxed">
      <p class="body-text">
        <strong class="cell-strong">BookStore</strong> didirikan dengan semangat
        untuk mempermudah akses literatur dan buku-buku berkualitas bagi seluruh
        masyarakat Indonesia. Kami percaya bahwa membaca adalah jendela dunia
        dan kunci utama menuju kemajuan.
      </p>

      <p class="body-text">
        Melalui platform digital yang dibangun dengan teknologi terbaru (Laravel
        12 API &amp; Nuxt 3), kami berkomitmen menyajikan pengalaman berbelanja
        yang cepat, aman, dan nyaman dari perangkat apa saja.
      </p>

      <div class="grid grid-cols-1 md:grid-cols-3 gap-5 pt-6 panel-divider">
        <div class="value-card p-5 text-center">
          <div
            class="value-icon w-11 h-11 flex items-center justify-center mx-auto mb-3"
          >
            <LucideTarget class="w-5 h-5" :stroke-width="1.5" />
          </div>
          <h4 class="cell-strong font-semibold text-sm mb-1">Misi Kami</h4>
          <p class="muted text-xs leading-relaxed">
            Menyediakan koleksi buku terlengkap dengan jaminan kualitas terbaik.
          </p>
        </div>
        <div class="value-card p-5 text-center">
          <div
            class="value-icon w-11 h-11 flex items-center justify-center mx-auto mb-3"
          >
            <LucideZap class="w-5 h-5" :stroke-width="1.5" />
          </div>
          <h4 class="cell-strong font-semibold text-sm mb-1">
            Kemudahan Layanan
          </h4>
          <p class="muted text-xs leading-relaxed">
            Proses pemesanan transparan dengan kode pesanan unik &amp; invoice
            instan.
          </p>
        </div>
        <div class="value-card p-5 text-center">
          <div
            class="value-icon w-11 h-11 flex items-center justify-center mx-auto mb-3"
          >
            <LucideMessageCircle class="w-5 h-5" :stroke-width="1.5" />
          </div>
          <h4 class="cell-strong font-semibold text-sm mb-1">
            Dukungan Responsif
          </h4>
          <p class="muted text-xs leading-relaxed">
            Layanan live chat langsung dengan admin toko untuk konsultasi.
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  Target as LucideTarget,
  Zap as LucideZap,
  MessageCircle as LucideMessageCircle,
} from "lucide-vue-next";
</script>

<style scoped>
/* Semua warna mengikuti token di main.css, jadi mode terang dan gelap sinkron */
.page {
  color: var(--text);
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

.body-text {
  color: var(--text-2);
}

.eyebrow {
  display: inline-block;
  background-color: var(--inset);
  border: 1px solid var(--border-strong);
  border-radius: 9999px;
  color: var(--text-2);
  letter-spacing: 0.04em;
}

.panel {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

.panel-divider {
  border-top: 1px solid var(--border);
}

.value-card {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

.value-icon {
  background-color: var(--surface);
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
  color: var(--text-2);
}
</style>
