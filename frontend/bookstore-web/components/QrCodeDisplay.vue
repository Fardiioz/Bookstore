<template>
  <div class="qr-card flex flex-col items-center justify-center p-3">
    <div class="w-full flex items-center justify-center min-h-[140px]">
      <img
        v-if="qrDataUrl"
        :src="qrDataUrl"
        :alt="value"
        class="qr-img w-full h-auto max-w-[160px] aspect-square object-contain p-1"
      />
      <div
        v-else
        class="qr-loading w-32 h-32 flex items-center justify-center text-xs"
      >
        Membuat QR...
      </div>
    </div>
    <span v-if="showLabel" class="qr-label text-xs mt-2 px-2 py-0.5">
      {{ value }}
    </span>
  </div>
</template>

<script setup lang="ts">
import QRCode from "qrcode";

const props = defineProps({
  value: {
    type: String,
    required: true,
  },
  size: {
    type: Number,
    default: 200,
  },
  showLabel: {
    type: Boolean,
    default: false,
  },
});

const qrDataUrl = ref("");

const generateQR = async () => {
  if (!props.value) return;
  try {
    const url = await QRCode.toDataURL(props.value, {
      width: props.size,
      margin: 2,
      color: {
        dark: "#000000",
        light: "#ffffff",
      },
    });
    qrDataUrl.value = url;
  } catch (err) {
    console.error("Failed to generate QR code:", err);
  }
};

watch(() => props.value, generateQR, { immediate: true });
onMounted(generateQR);
</script>

<style scoped>
/* Kartu mengikuti token main.css (terang/gelap). */
.qr-card {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

/* Kode QR sengaja selalu hitam di atas putih, di kedua mode,
   supaya kontrasnya cukup untuk dipindai kamera atau scanner. */
.qr-img {
  background-color: #ffffff;
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
}

.qr-loading {
  color: var(--muted);
}

/* Kode pesanan memakai monospace agar karakter mudah dibedakan (0/O, 1/l) */
.qr-label {
  font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
  font-weight: 600;
  letter-spacing: 0.04em;
  color: var(--text-2);
  border-top: 1px solid var(--border);
  width: 100%;
  text-align: center;
  padding-top: 0.5rem;
  margin-top: 0.5rem;
}
</style>
