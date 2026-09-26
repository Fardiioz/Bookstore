<template>
  <div class="page max-w-4xl mx-auto px-4 py-16">
    <div class="text-center mb-12">
      <span class="eyebrow px-3 py-1 text-xs font-semibold">KONTAK</span>
      <h1 class="page-title text-2xl sm:text-3xl mt-4 mb-3">Hubungi Kami</h1>
      <p class="muted text-sm max-w-md mx-auto">
        Ada pertanyaan atau kendala? Kami siap membantu Anda kapan saja.
      </p>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
      <!-- Informasi Kontak -->
      <div class="panel p-8 space-y-6">
        <h3 class="panel-title font-semibold text-lg">Informasi Kontak</h3>

        <div class="flex items-start gap-4">
          <div
            class="info-icon w-10 h-10 flex items-center justify-center shrink-0"
          >
            <LucideMapPin class="w-4 h-4" />
          </div>
          <div>
            <h5 class="cell-strong font-semibold text-sm">Alamat</h5>
            <p class="muted text-sm mt-1">
              Jl. Pendidikan No. 123, Kota Sekolahan, Indonesia
            </p>
          </div>
        </div>

        <div class="flex items-start gap-4">
          <div
            class="info-icon w-10 h-10 flex items-center justify-center shrink-0"
          >
            <LucidePhone class="w-4 h-4" />
          </div>
          <div>
            <h5 class="cell-strong font-semibold text-sm">
              Telepon / WhatsApp
            </h5>
            <p class="muted text-sm mt-1">+62 812-3456-7890</p>
          </div>
        </div>

        <div class="flex items-start gap-4">
          <div
            class="info-icon w-10 h-10 flex items-center justify-center shrink-0"
          >
            <LucideMail class="w-4 h-4" />
          </div>
          <div>
            <h5 class="cell-strong font-semibold text-sm">Email</h5>
            <p class="muted text-sm mt-1">support@bookstore.com</p>
          </div>
        </div>

        <div class="panel-divider pt-4">
          <NuxtLink
            to="/user/chat"
            class="btn-primary inline-flex items-center gap-2 px-4 py-2.5 text-sm font-semibold"
          >
            <LucideMessageCircle class="w-4 h-4" />
            Gunakan Live Chat Langsung
          </NuxtLink>
        </div>
      </div>

      <!-- Form Pesan -->
      <div class="panel p-8">
        <h3 class="panel-title font-semibold text-lg mb-4">Kirim Pesan</h3>
        <form @submit.prevent="sendMessage" class="space-y-4">
          <div>
            <label class="label block text-xs font-semibold mb-1"
              >Nama Lengkap</label
            >
            <input
              v-model="form.name"
              type="text"
              required
              class="field w-full px-3.5 py-2.5 text-sm"
              placeholder="Masukkan nama"
            />
          </div>
          <div>
            <label class="label block text-xs font-semibold mb-1">Email</label>
            <input
              v-model="form.email"
              type="email"
              required
              class="field w-full px-3.5 py-2.5 text-sm"
              placeholder="alamat@email.com"
            />
          </div>
          <div>
            <label class="label block text-xs font-semibold mb-1">Pesan</label>
            <textarea
              v-model="form.message"
              rows="4"
              required
              class="field w-full px-3.5 py-2.5 text-sm resize-none"
              placeholder="Tuliskan pesan Anda..."
            ></textarea>
          </div>
          <button
            type="submit"
            class="btn-primary w-full py-3 text-sm font-semibold inline-flex items-center justify-center gap-2"
          >
            Kirim Pesan Sekarang
            <LucideArrowRight class="w-4 h-4" />
          </button>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  MapPin as LucideMapPin,
  Phone as LucidePhone,
  Mail as LucideMail,
  MessageCircle as LucideMessageCircle,
  ArrowRight as LucideArrowRight,
} from "lucide-vue-next";

const form = reactive({
  name: "",
  email: "",
  message: "",
});

const sendMessage = () => {
  const toast = useToast();
  toast.success("Terima kasih! Pesan Anda telah terkirim.");
  form.name = "";
  form.email = "";
  form.message = "";
};
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

.label {
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

.panel-title {
  font-family: var(--font-display);
  color: var(--text);
}

.panel-divider {
  border-top: 1px solid var(--border);
}

.info-icon {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  color: var(--text-2);
}

.field {
  background-color: var(--bg);
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
</style>
