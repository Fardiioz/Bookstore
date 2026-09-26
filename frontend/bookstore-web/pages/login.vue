<template>
  <div class="min-h-[80vh] flex items-center justify-center px-4 py-12">
    <div class="card max-w-md w-full p-8">
      <div class="text-center mb-8">
        <div
          class="icon-box w-12 h-12 flex items-center justify-center mx-auto mb-3"
        >
          <LucideKeyRound class="w-5 h-5" />
        </div>
        <h2 class="card-title text-xl">Selamat Datang Kembali</h2>
        <p class="muted text-sm mt-1">Masuk ke akun BookStore Anda</p>
      </div>

      <div
        v-if="errorMessage"
        class="alert-error mb-4 p-3.5 text-sm font-medium"
      >
        {{ errorMessage }}
      </div>

      <form @submit.prevent="handleLogin" class="space-y-4">
        <div>
          <label class="label block text-xs font-semibold mb-1"
            >Username atau Email</label
          >
          <input
            v-model="form.login"
            type="text"
            required
            class="field w-full px-4 py-2.5 text-sm"
            placeholder="Masukkan username/email"
          />
        </div>

        <div>
          <div class="flex justify-between items-center mb-1">
            <label class="label text-xs font-semibold">Password</label>
            <NuxtLink
              to="/forget-password"
              class="link-btn text-xs font-medium"
            >
              Lupa Password?
            </NuxtLink>
          </div>
          <input
            v-model="form.password"
            type="password"
            required
            class="field w-full px-4 py-2.5 text-sm"
            placeholder="••••••••"
          />
        </div>

        <button
          type="submit"
          :disabled="loading"
          class="btn-primary w-full py-3 text-sm font-semibold disabled:opacity-50 mt-2 inline-flex items-center justify-center gap-2"
        >
          <span v-if="loading">Memproses...</span>
          <template v-else>
            Masuk Sekarang
            <LucideArrowRight class="w-4 h-4" />
          </template>
        </button>
      </form>

      <div class="muted mt-6 text-center text-sm">
        Belum punya akun?
        <NuxtLink to="/register" class="link-strong ml-1 font-semibold"
          >Daftar Akun Baru</NuxtLink
        >
      </div>

      <!-- Info Akun Demo -->
      <div class="demo-box mt-8 pt-6 text-xs">
        <p class="cell-strong font-semibold mb-1.5">Akun Demo Testing</p>
        <p class="muted">
          Admin:
          <code class="code">admin</code> /
          <code class="code">password</code>
        </p>
        <p class="muted mt-1">
          User:
          <code class="code">userdemo</code> /
          <code class="code">password</code>
        </p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  KeyRound as LucideKeyRound,
  ArrowRight as LucideArrowRight,
} from "lucide-vue-next";

definePageMeta({
  layout: "auth",
});

const authStore = useAuthStore();

const form = reactive({
  login: "",
  password: "",
});

const loading = ref(false);
const errorMessage = ref("");

const handleLogin = async () => {
  loading.value = true;
  errorMessage.value = "";

  try {
    await authStore.login(form);
    if (authStore.isAdmin) {
      navigateTo("/admin/kategori");
    } else {
      navigateTo("/user/katalog");
    }
  } catch (err: any) {
    errorMessage.value =
      err.data?.message ||
      err.data?.errors?.login?.[0] ||
      "Login gagal. Periksa kembali data Anda.";
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

.cell-strong {
  color: var(--text);
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

.alert-error {
  background-color: color-mix(in srgb, #b0394f 8%, transparent);
  border: 1px solid color-mix(in srgb, #b0394f 45%, transparent);
  border-radius: var(--radius-md);
  color: #b0394f;
}

:global(:root.dark) .alert-error,
:global(:root[data-theme="dark"]) .alert-error {
  color: #e68a9a;
}

.demo-box {
  border-top: 1px solid var(--border);
}

.code {
  font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
  font-weight: 600;
  color: var(--text);
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  padding: 0.0625rem 0.375rem;
}
</style>
