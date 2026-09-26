<template>
  <header
    class="site-header sticky top-0 z-50 h-16 md:h-20 px-4 md:px-8 flex items-center justify-between shrink-0"
  >
    <!-- Brand (mobile) -->
    <div class="flex items-center gap-3">
      <NuxtLink to="/" class="md:hidden brand">BookStore</NuxtLink>
    </div>

    <!-- Kontrol kanan: tema, profil, menu mobile -->
    <div class="flex items-center gap-3">
      <!-- Theme Switcher -->
      <button
        type="button"
        @click="toggleTheme()"
        class="icon-btn"
        :title="isDark ? 'Ubah ke Tema Terang' : 'Ubah ke Tema Gelap'"
        :aria-label="isDark ? 'Ubah ke tema terang' : 'Ubah ke tema gelap'"
      >
        <LucideSun v-if="isDark" class="w-4 h-4" />
        <LucideMoon v-else class="w-4 h-4" />
      </button>

      <!-- Profil pengguna (klik untuk edit profile) -->
      <NuxtLink
        :to="authStore.isAdmin ? '/admin/edit-profile' : '/user/edit-profile'"
        class="profile flex items-center gap-2.5"
        title="Edit Profile"
        aria-label="Edit Profile"
      >
        <div class="text-right leading-tight hidden sm:block">
          <div class="profile-name text-xs font-semibold">
            {{ authStore.user?.name || "User" }}
          </div>
          <div class="profile-role text-[11px]">
            {{
              authStore.isAdmin
                ? "Admin Kasir"
                : authStore.user?.role || "Pelanggan"
            }}
          </div>
        </div>
        <div class="avatar shrink-0">
          <img
            v-if="userAvatarUrl"
            :src="userAvatarUrl"
            :alt="authStore.user?.name || 'User'"
            class="w-full h-full object-cover"
          />
          <span v-else>
            {{ (authStore.user?.name || "U").charAt(0).toUpperCase() }}
          </span>
        </div>
      </NuxtLink>

      <!-- Toggle menu mobile -->
      <button
        type="button"
        @click="showMobileMenu = !showMobileMenu"
        class="icon-btn md:hidden"
        :aria-expanded="showMobileMenu"
        aria-label="Buka menu"
      >
        <LucideMenu class="w-4 h-4" />
      </button>
    </div>

    <!-- Drawer menu mobile -->
    <div
      v-if="showMobileMenu"
      class="drawer md:hidden fixed inset-0 z-50 flex flex-col p-6"
    >
      <div class="drawer-top flex items-center justify-between pb-4 mb-6">
        <span class="brand">BookStore</span>
        <button
          type="button"
          @click="showMobileMenu = false"
          class="icon-btn"
          aria-label="Tutup menu"
        >
          <LucideX class="w-4 h-4" />
        </button>
      </div>

      <button
        type="button"
        @click="toggleTheme()"
        class="drawer-theme w-full py-3 text-xs font-semibold flex items-center justify-center gap-2 mb-4"
      >
        <LucideSun v-if="isDark" class="w-4 h-4" />
        <LucideMoon v-else class="w-4 h-4" />
        <span>{{
          isDark ? "Ganti ke tema terang" : "Ganti ke tema gelap"
        }}</span>
      </button>

      <nav class="flex-1 overflow-y-auto" aria-label="Menu utama">
        <NuxtLink
          v-for="item in mobileNavItems"
          :key="item.path"
          :to="item.path"
          @click="showMobileMenu = false"
          class="drawer-link flex items-center justify-between py-3.5 px-3 text-sm"
          :class="{ 'drawer-link--active': route.path === item.path }"
        >
          <span>{{ item.label }}</span>
          <LucideChevronRight class="w-4 h-4 drawer-chevron" />
        </NuxtLink>
      </nav>

      <div class="drawer-bottom pt-4">
        <button
          type="button"
          @click="handleLogout"
          class="btn-danger w-full py-3 text-sm font-semibold flex items-center justify-center gap-2"
        >
          <LucideLogOut class="w-4 h-4" />
          <span>Keluar</span>
        </button>
      </div>
    </div>
  </header>
</template>

<script setup lang="ts">
import {
  LucideSun,
  LucideMoon,
  LucideMenu,
  LucideX,
  LucideChevronRight,
  LucideLogOut,
} from "lucide-vue-next";

const route = useRoute();
const api = useApi();
const authStore = useAuthStore();
const { isDark, toggleTheme, initTheme } = useTheme();

const showMobileMenu = ref(false);

onMounted(() => {
  initTheme();
});

// Sinkronkan tema ke <html> agar token warna di main.css (mode terang/gelap)
// berlaku di seluruh halaman, bukan hanya di komponen ini.
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

const pageTitleMap: Record<string, string> = {
  "/admin/kategori": "Kelola Kategori",
  "/admin/buku": "Katalog Buku",
  "/admin/pengguna": "Kelola Pengguna",
  "/admin/kasir": "Kasir & Scan QR",
  "/admin/laporan": "Laporan Penjualan",
  "/admin/chat": "Live Chat Admin",
  "/user/katalog": "Katalog Buku",
  "/user/keranjang": "Keranjang Belanja",
  "/user/riwayat": "Riwayat Pesanan",
  "/user/chat": "Live Chat Bantuan",
};

const pageTitle = computed(() => {
  return (
    pageTitleMap[route.path] || (route.meta.title as string) || "BookStore"
  );
});

const adminMobileNav = [
  { label: "Edit Profile", path: "/admin/edit-profile" },
  { label: "Kelola Kategori", path: "/admin/kategori" },
  { label: "Katalog Buku", path: "/admin/buku" },
  { label: "Kelola Pengguna", path: "/admin/pengguna" },
  { label: "Kasir & Scan QR Code", path: "/admin/kasir" },
  { label: "Laporan Penjualan", path: "/admin/laporan" },
  { label: "Live Chat", path: "/admin/chat" },
];

const userMobileNav = [
  { label: "Edit Profile", path: "/user/edit-profile" },
  { label: "Katalog Buku", path: "/user/katalog" },
  { label: "Keranjang Belanja", path: "/user/keranjang" },
  { label: "Riwayat Pesanan", path: "/user/riwayat" },
  { label: "Live Chat", path: "/user/chat" },
];

const mobileNavItems = computed(() => {
  return authStore.isAdmin ? adminMobileNav : userMobileNav;
});

const userAvatarUrl = computed(() => {
  if (!authStore.user?.foto) return null;
  if (authStore.user.foto.startsWith("http")) return authStore.user.foto;
  return `${api.apiBase}/storage/${authStore.user.foto}`;
});

const handleLogout = async () => {
  showMobileMenu.value = false;
  if (confirm("Apakah Anda yakin ingin keluar?")) {
    await authStore.logout();
  }
};
</script>

<style scoped>
/* Semua warna mengikuti token di main.css, jadi mode terang dan gelap sinkron */
.site-header {
  background-color: var(--surface);
  border-bottom: 1px solid var(--border);
  color: var(--text);
}

.brand {
  font-family: var(--font-display);
  font-size: 1.125rem;
  font-weight: 600;
  letter-spacing: -0.01em;
  color: var(--text);
}

.icon-btn {
  width: 2.25rem;
  height: 2.25rem;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  background: transparent;
  color: var(--text-2);
  transition:
    background-color 0.15s ease,
    border-color 0.15s ease,
    color 0.15s ease;
}

.icon-btn:hover {
  background-color: var(--inset);
  border-color: var(--border-strong);
  color: var(--text);
}

.profile {
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  padding: 0.3125rem 0.625rem;
  cursor: pointer;
  transition:
    background-color 0.15s ease,
    border-color 0.15s ease;
}

.profile:hover {
  background-color: var(--inset);
  border-color: var(--border-strong);
}

.profile-name {
  color: var(--text);
}

.profile-role {
  color: var(--muted);
}

.avatar {
  width: 2rem;
  height: 2rem;
  border-radius: 9999px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: var(--primary);
  color: var(--on-primary);
  font-size: 0.75rem;
  font-weight: 600;
}

/* drawer */
.drawer {
  background-color: var(--bg);
  color: var(--text);
}

.drawer-top,
.drawer-bottom {
  border-color: var(--border);
}

.drawer-top {
  border-bottom: 1px solid var(--border);
}

.drawer-bottom {
  border-top: 1px solid var(--border);
}

.drawer-theme {
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  background-color: var(--surface);
  color: var(--text);
}

.drawer-link {
  color: var(--text-2);
  border-bottom: 1px solid var(--border);
  border-left: 2px solid transparent;
  transition:
    color 0.15s ease,
    background-color 0.15s ease;
}

.drawer-link:hover {
  color: var(--text);
  background-color: var(--inset);
}

.drawer-link--active {
  color: var(--text);
  font-weight: 600;
  border-left-color: var(--primary);
  background-color: var(--inset);
}

.drawer-chevron {
  color: var(--muted);
}

.btn-danger {
  color: #b0394f;
  border: 1px solid color-mix(in srgb, #b0394f 45%, transparent);
  border-radius: var(--radius-md);
  background-color: transparent;
  transition: background-color 0.15s ease;
}

.btn-danger:hover {
  background-color: color-mix(in srgb, #b0394f 10%, transparent);
}

:global(:root.dark) .btn-danger,
:global(:root[data-theme="dark"]) .btn-danger {
  color: #e68a9a;
  border-color: color-mix(in srgb, #e68a9a 45%, transparent);
}
</style>
