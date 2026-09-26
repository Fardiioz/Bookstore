<template>
  <div class="min-h-[80vh] flex items-center justify-center px-4 py-12">
    <div class="card max-w-md w-full p-8">
      <div class="text-center mb-8">
        <div
          class="icon-box w-12 h-12 flex items-center justify-center mx-auto mb-3"
        >
          <LucideMailCheck class="w-5 h-5" />
        </div>
        <h2 class="card-title text-xl">Lupa Password</h2>
        <p class="muted text-sm mt-2">
          <span v-if="step === 1"
            >Langkah 1: Masukkan email terdaftar untuk menerima kode verifikasi
            6 digit</span
          >
          <span v-else-if="step === 2"
            >Langkah 2: Masukkan 6 digit kode yang dikirim ke
            {{ verifiedEmail }}</span
          >
          <span v-else>Langkah 3: Buat password baru Anda</span>
        </p>
      </div>

      <div v-if="message" class="alert-success mb-4 p-3.5 text-sm font-medium">
        {{ message }}
      </div>

      <div
        v-if="errorMessage"
        class="alert-error mb-4 p-3.5 text-sm font-medium"
      >
        {{ errorMessage }}
      </div>

      <!-- Step 1: Input Email -->
      <form v-if="step === 1" @submit.prevent="sendCode" class="space-y-4">
        <div>
          <label class="label block text-xs font-semibold mb-1"
            >Email Terdaftar</label
          >
          <input
            v-model="emailInput"
            type="email"
            required
            class="field w-full px-4 py-2.5 text-sm"
            placeholder="alamat@email.com"
          />
        </div>

        <button
          type="submit"
          :disabled="loading"
          class="btn-primary w-full py-3 text-sm font-semibold disabled:opacity-50 mt-2 inline-flex items-center justify-center gap-2"
        >
          <template v-if="loading">Mengirim Email...</template>
          <template v-else>
            <LucideSend class="w-4 h-4" />
            Kirim Kode 6 Digit Ke Email
          </template>
        </button>
      </form>

      <!-- Step 2: Input 6-Digit OTP Code -->
      <form
        v-else-if="step === 2"
        @submit.prevent="verifyCode"
        class="space-y-4"
      >
        <div>
          <label class="label block text-xs font-semibold mb-1">
            Kode Verifikasi 6 Digit (Cek Email Kamu)
          </label>
          <input
            v-model="otpCode"
            type="text"
            maxlength="6"
            required
            class="field field--otp w-full px-4 py-3 text-center text-2xl font-semibold tracking-widest"
            placeholder="123456"
          />
        </div>

        <button
          type="submit"
          :disabled="loading || otpCode.length !== 6"
          class="btn-primary w-full py-3 text-sm font-semibold disabled:opacity-50 inline-flex items-center justify-center gap-2"
        >
          <template v-if="loading">Memverifikasi Kode...</template>
          <template v-else>
            Verifikasi Kode
            <LucideArrowRight class="w-4 h-4" />
          </template>
        </button>

        <div class="text-center pt-2">
          <button
            type="button"
            @click="step = 1"
            class="link-btn text-xs font-medium"
          >
            Kirim Ulang Kode Ke Email
          </button>
        </div>
      </form>

      <!-- Step 3: Input New Password -->
      <form v-else @submit.prevent="handleReset" class="space-y-4">
        <div>
          <label class="label block text-xs font-semibold mb-1"
            >Password Baru</label
          >
          <input
            v-model="newPassword"
            type="password"
            required
            class="field w-full px-4 py-2.5 text-sm"
            placeholder="Minimal 6 karakter"
          />
        </div>

        <div>
          <label class="label block text-xs font-semibold mb-1"
            >Konfirmasi Password Baru</label
          >
          <input
            v-model="confirmPassword"
            type="password"
            required
            class="field w-full px-4 py-2.5 text-sm"
            placeholder="Ulangi password baru"
          />
        </div>

        <button
          type="submit"
          :disabled="loading"
          class="btn-primary w-full py-3 text-sm font-semibold disabled:opacity-50"
        >
          <span v-if="loading">Memperbarui Password...</span>
          <span v-else>Simpan Password Baru</span>
        </button>
      </form>

      <div class="muted mt-6 text-center text-sm">
        Kembali ke
        <NuxtLink to="/login" class="link-strong ml-1 font-semibold"
          >Halaman Login</NuxtLink
        >
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  MailCheck as LucideMailCheck,
  Send as LucideSend,
  ArrowRight as LucideArrowRight,
} from "lucide-vue-next";

definePageMeta({
  layout: "auth",
});

const api = useApi();

const step = ref(1);
const emailInput = ref("");
const verifiedEmail = ref("");
const otpCode = ref("");
const newPassword = ref("");
const confirmPassword = ref("");

const loading = ref(false);
const message = ref("");
const errorMessage = ref("");

// Step 1: Send 6-Digit Code to Email
const sendCode = async () => {
  loading.value = true;
  message.value = "";
  errorMessage.value = "";

  try {
    const res = await api.post<{ message: string; email: string }>(
      "/api/forgot-password/send-code",
      {
        email: emailInput.value,
      },
    );
    verifiedEmail.value = res.email;
    message.value = res.message;
    step.value = 2;
  } catch (err: any) {
    errorMessage.value =
      err.data?.message ||
      err.data?.errors?.email?.[0] ||
      "Gagal mengirim kode email.";
  } finally {
    loading.value = false;
  }
};

// Step 2: Verify 6-Digit Code
const verifyCode = async () => {
  loading.value = true;
  message.value = "";
  errorMessage.value = "";

  try {
    const res = await api.post<{ message: string }>(
      "/api/forgot-password/verify-code",
      {
        email: verifiedEmail.value,
        code: otpCode.value,
      },
    );
    message.value = res.message;
    step.value = 3;
  } catch (err: any) {
    errorMessage.value = err.data?.message || "Kode verifikasi tidak valid.";
  } finally {
    loading.value = false;
  }
};

// Step 3: Reset Password
const handleReset = async () => {
  if (newPassword.value !== confirmPassword.value) {
    errorMessage.value = "Konfirmasi password tidak cocok.";
    return;
  }

  loading.value = true;
  message.value = "";
  errorMessage.value = "";

  try {
    const res = await api.post<{ message: string }>(
      "/api/forgot-password/reset",
      {
        email: verifiedEmail.value,
        code: otpCode.value,
        password: newPassword.value,
        password_confirmation: confirmPassword.value,
      },
    );
    const toast = useToast();
    toast.success(res.message || "Password berhasil diperbarui!");
    navigateTo("/login");
  } catch (err: any) {
    errorMessage.value = err.data?.message || "Gagal memperbarui password.";
  } finally {
    loading.value = false;
  }
};
</script>

<style scoped>
/* Semua warna mengikuti token di main.css, jadi mode terang dan gelap sinkron */
.card {
  background-color: var(--surface);
  color: var(--text);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

.card-title {
  font-family: var(--font-display);
  font-weight: 600;
  color: var(--text);
}

.muted {
  color: var(--muted);
}

.label {
  color: var(--text-2);
}

.icon-box {
  background-color: var(--primary);
  color: var(--on-primary);
  border-radius: var(--radius-md);
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

.field--otp {
  background-color: var(--inset);
  letter-spacing: 0.3em;
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

.link-btn {
  color: var(--text-2);
  text-decoration: underline;
  text-underline-offset: 3px;
  transition: color 0.15s ease;
}

.link-btn:hover {
  color: var(--text);
}

.link-strong {
  color: var(--primary);
  transition: opacity 0.15s ease;
}

.link-strong:hover {
  opacity: 0.8;
}

.alert-success {
  background-color: color-mix(in srgb, #4d6b3f 10%, transparent);
  border: 1px solid color-mix(in srgb, #4d6b3f 40%, transparent);
  border-radius: var(--radius-md);
  color: #4d6b3f;
}

.alert-error {
  background-color: color-mix(in srgb, #b0394f 8%, transparent);
  border: 1px solid color-mix(in srgb, #b0394f 45%, transparent);
  border-radius: var(--radius-md);
  color: #b0394f;
}

:global(:root.dark) .alert-success,
:global(:root[data-theme="dark"]) .alert-success {
  color: #b3c0a4;
}

:global(:root.dark) .alert-error,
:global(:root[data-theme="dark"]) .alert-error {
  color: #e68a9a;
}
</style>
