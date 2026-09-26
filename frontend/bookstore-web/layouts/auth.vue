<template>
  <div
    class="auth-layout min-h-screen antialiased flex flex-col justify-center items-center p-4"
  >
    <main class="w-full">
      <slot />
    </main>
  </div>
</template>

<script setup lang="ts">
const { isDark, initTheme } = useTheme();

onMounted(() => {
  initTheme();
});

// Sinkronkan tema ke <html> agar token warna di main.css berlaku di halaman
// yang tidak memakai header/navbar (login, register, dan sejenisnya).
watch(
  isDark,
  (dark) => {
    if (import.meta.client) {
      const root = document.documentElement;
      root.classList.toggle("dark", !!dark);
      root.dataset.theme = dark ? "dark" : "light";
    }
  },
  { immediate: true },
);
</script>

<style scoped>
/* Font tidak di-set di sini agar mewarisi Hanken Grotesk dari main.css */
.auth-layout {
  background-color: var(--bg);
  color: var(--text);
  transition:
    background-color 0.2s ease,
    color 0.2s ease;
}
</style>
