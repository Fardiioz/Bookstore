<template>
  <div
    class="app-shell flex flex-col md:flex-row antialiased"
    :class="
      isChatPage
        ? 'h-screen overflow-hidden pb-16 md:pb-0'
        : 'min-h-screen pb-24 md:pb-0'
    "
  >
    <!-- Desktop Sidebar -->
    <Sidebar />

    <!-- Right Main Content Wrapper -->
    <div class="flex-grow flex flex-col min-w-0 min-h-0">
      <!-- Header Bar -->
      <Header />

      <!-- Main Content Container -->
      <main
        class="app-main flex-1 flex flex-col min-h-0"
        :class="
          isChatPage ? 'p-0 overflow-hidden' : 'p-3 sm:p-6 overflow-y-auto'
        "
      >
        <div
          :class="
            isChatPage
              ? 'flex-1 flex flex-col h-full overflow-hidden'
              : 'app-panel min-h-full p-4 sm:p-6'
          "
        >
          <slot />
        </div>
      </main>
    </div>

    <!-- Mobile Bottom Navigation Bar (hanya tampil di mobile < md) -->
    <nav
      class="bottom-nav md:hidden fixed bottom-0 left-0 right-0 z-40"
      aria-label="Navigasi bawah"
    >
      <div class="flex items-stretch justify-around h-16">
        <NuxtLink
          v-for="item in bottomNavItems"
          :key="item.path"
          :to="item.path"
          class="bnav-item relative flex flex-col items-center justify-center gap-1 flex-1"
          :class="{ 'bnav-item--active': route.path === item.path }"
        >
          <div class="relative">
            <component :is="item.icon" class="w-5 h-5" />
            <span
              v-if="item.path.includes('/chat') && hasUnreadChat"
              class="bnav-dot absolute -top-0.5 -right-1 w-2 h-2 rounded-full"
              aria-label="Ada pesan belum dibaca"
            ></span>
          </div>
          <span class="text-[11px] leading-none">{{ item.label }}</span>
        </NuxtLink>
      </div>
    </nav>

    <!-- Floating Chat Widget (pojok kanan bawah di semua halaman) -->
    <FloatingChat />

    <!-- Toast Notifications (pojok kanan atas) -->
    <Toast />
  </div>
</template>

<script setup lang="ts">
import {
  Tag as LucideTag,
  Book as LucideBook,
  ShoppingCart as LucideShoppingCart,
  BarChart3 as LucideBarChart3,
  MessageSquare as LucideMessageSquare,
  Home as LucideHome,
  Store as LucideStore,
  History as LucideHistory,
} from "lucide-vue-next";

const route = useRoute();
const authStore = useAuthStore();
const cartStore = useCartStore();
const { initTheme } = useTheme();

const isChatPage = computed(() => route.path.includes("/chat"));
const api = useApi();
const hasUnreadChat = ref(false);

const checkUnreadChat = async () => {
  if (!authStore.isAuthenticated) {
    hasUnreadChat.value = false;
    return;
  }
  if (isChatPage.value) {
    hasUnreadChat.value = false;
    try {
      await api.post("/api/chats/mark-as-read");
    } catch (e) {}
    return;
  }
  try {
    const res: any = await api.get("/api/chats/unread-count");
    // Dukung dua bentuk respons: { unread_count } atau { data: { unread_count } }
    const count = res?.unread_count ?? res?.data?.unread_count ?? 0;
    hasUnreadChat.value = count > 0;
  } catch (e) {
    hasUnreadChat.value = false;
  }
};

let navPollTimer: any = null;

const adminBottomNav = [
  { label: "Kategori", path: "/admin/kategori", icon: LucideTag },
  { label: "Buku", path: "/admin/buku", icon: LucideBook },
  { label: "Kasir", path: "/admin/kasir", icon: LucideShoppingCart },
  { label: "Laporan", path: "/admin/laporan", icon: LucideBarChart3 },
  { label: "Chat", path: "/admin/chat", icon: LucideMessageSquare },
];

const userBottomNav = [
  { label: "Home", path: "/", icon: LucideHome },
  { label: "Katalog", path: "/user/katalog", icon: LucideStore },
  { label: "Keranjang", path: "/user/keranjang", icon: LucideShoppingCart },
  { label: "Pesanan", path: "/user/riwayat", icon: LucideHistory },
];

const bottomNavItems = computed(() => {
  return authStore.isAdmin ? adminBottomNav : userBottomNav;
});

onMounted(async () => {
  initTheme();
  if (!authStore.initialized) {
    await authStore.fetchUser();
  }
  if (authStore.isAuthenticated) {
    await cartStore.fetchCart();
    checkUnreadChat();
    navPollTimer = setInterval(checkUnreadChat, 5000);
  }
});

onUnmounted(() => {
  if (navPollTimer) clearInterval(navPollTimer);
});
</script>

<style scoped>
/* Semua warna mengikuti token di main.css, jadi mode terang dan gelap sinkron */
.app-shell {
  background-color: var(--bg);
  color: var(--text);
  transition:
    background-color 0.2s ease,
    color 0.2s ease;
}

.app-main {
  background-color: var(--bg);
}

.app-panel {
  background-color: var(--surface);
  color: var(--text);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
}

.bottom-nav {
  background-color: var(--surface);
  border-top: 1px solid var(--border);
  padding-bottom: env(safe-area-inset-bottom, 0px);
}

.bnav-item {
  color: var(--muted);
  font-weight: 500;
  transition: color 0.15s ease;
}

.bnav-item--active {
  color: var(--text);
  font-weight: 600;
}

/* penanda halaman aktif: garis tipis di tepi atas item */
.bnav-item--active::before {
  content: "";
  position: absolute;
  top: 0;
  left: 25%;
  right: 25%;
  height: 2px;
  background-color: var(--primary);
}

.bnav-dot {
  background-color: #b0394f;
  box-shadow: 0 0 0 2px var(--surface);
}
</style>
