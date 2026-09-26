<template>
  <aside
    class="sidebar hidden md:flex w-64 h-screen sticky top-0 p-5 flex-col justify-between select-none shrink-0 overflow-y-auto custom-scrollbar"
  >
    <div>
      <!-- Brand -->
      <NuxtLink to="/" class="brand block px-3 py-3 mb-5">BookStore</NuxtLink>

      <!-- Sidebar Search Box -->
      <div class="relative mb-5">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Cari menu..."
          aria-label="Cari menu"
          class="search-input w-full pl-9 pr-3 py-2 text-sm outline-none"
        />
        <LucideSearch class="search-icon w-4 h-4 absolute left-3 top-2.5" />
      </div>

      <!-- Navigation Menu -->
      <nav class="space-y-1" aria-label="Menu dashboard">
        <template v-for="item in filteredNavItems" :key="item.path">
          <NuxtLink
            :to="item.path"
            class="nav-item flex items-center gap-3 px-3 py-2.5 text-sm"
            :class="{ 'nav-item--active': isRouteActive(item.path) }"
          >
            <component :is="item.icon" class="nav-icon w-4 h-4" />
            <span>{{ item.label }}</span>
          </NuxtLink>

          <!-- Divider -->
          <div v-if="item.divider" class="nav-divider my-3"></div>
        </template>
      </nav>
    </div>

    <!-- Bottom Actions -->
    <div class="sidebar-bottom pt-4">
      <button
        type="button"
        @click="handleLogout"
        class="btn-logout w-full flex items-center gap-3 px-3 py-2.5 text-sm"
      >
        <LucideLogOut class="w-4 h-4" />
        <span>Keluar</span>
      </button>
    </div>
  </aside>
</template>

<script setup lang="ts">
import {
  BookOpen as LucideBookOpen,
  Search as LucideSearch,
  Tag as LucideTag,
  Book as LucideBook,
  Users as LucideUsers,
  ShoppingBag as LucideShoppingBag,
  FileText as LucideFileText,
  ShoppingCart as LucideShoppingCart,
  History as LucideHistory,
  MessageSquare as LucideMessageSquare,
  LogOut as LucideLogOut,
  Store as LucideStore,
} from "lucide-vue-next";

const route = useRoute();
const authStore = useAuthStore();
const { isDark } = useTheme();

const searchQuery = ref("");
const api = useApi();
const unreadCount = ref(0);
let pollTimer: any = null;

const isChatPage = computed(() => route.path.includes("/chat"));

const checkUnread = async () => {
  if (!authStore.isAuthenticated) return;
  try {
    const res = await api.get("/api/chats/unread-count");
    unreadCount.value = res?.unread_count || 0;
  } catch (e) {
    unreadCount.value = 0;
  }
};

onMounted(() => {
  if (authStore.isAuthenticated) {
    checkUnread();
    pollTimer = setInterval(checkUnread, 4000);
  }
});

onUnmounted(() => {
  if (pollTimer) clearInterval(pollTimer);
});

const chatPath = computed(() => {
  return authStore.isAdmin ? "/admin/chat" : "/user/chat";
});

const adminNavItems = [
  { label: "Kategori", path: "/admin/kategori", icon: LucideTag },
  { label: "Katalog Buku", path: "/admin/buku", icon: LucideBook },
  { label: "Pengguna", path: "/admin/pengguna", icon: LucideUsers },
  { label: "Kasir & Scan", path: "/admin/kasir", icon: LucideShoppingBag },
  {
    label: "Laporan",
    path: "/admin/laporan",
    icon: LucideFileText,
    divider: true,
  },
];

const userNavItems = [
  { label: "Katalog Buku", path: "/user/katalog", icon: LucideStore },
  { label: "Keranjang", path: "/user/keranjang", icon: LucideShoppingCart },
  {
    label: "Pesanan anda",
    path: "/user/riwayat",
    icon: LucideHistory,
    divider: true,
  },
];

const navItems = computed(() => {
  return authStore.isAdmin ? adminNavItems : userNavItems;
});

const filteredNavItems = computed(() => {
  if (!searchQuery.value.trim()) return navItems.value;
  const q = searchQuery.value.toLowerCase();
  return navItems.value.filter((item) => item.label.toLowerCase().includes(q));
});

const isRouteActive = (path: string) => {
  if (path === "/admin/kasir") return route.path === "/admin/kasir";
  return route.path === path || route.path.startsWith(path + "/");
};

const handleLogout = async () => {
  if (confirm("Apakah Anda yakin ingin keluar?")) {
    await authStore.logout();
  }
};
</script>

<style scoped>
/* Semua warna mengikuti token di main.css, jadi mode terang dan gelap sinkron */
.sidebar {
  background-color: var(--surface);
  border-right: 1px solid var(--border);
  color: var(--text);
}

.brand {
  font-family: var(--font-display);
  font-size: 1.375rem;
  font-weight: 600;
  letter-spacing: -0.01em;
  color: var(--text);
}

.search-input {
  background-color: var(--inset);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  color: var(--text);
  transition: border-color 0.15s ease;
}

.search-input::placeholder {
  color: var(--muted);
}

.search-input:focus {
  border-color: var(--primary);
}

.search-icon {
  color: var(--muted);
}

.nav-item {
  color: var(--muted);
  font-weight: 500;
  border-radius: var(--radius-md);
  border-left: 2px solid transparent;
  transition:
    color 0.15s ease,
    background-color 0.15s ease;
}

.nav-item:hover {
  color: var(--text);
  background-color: var(--inset);
}

.nav-item--active {
  color: var(--text);
  font-weight: 600;
  background-color: var(--inset);
  border-left-color: var(--primary);
  border-top-left-radius: 0;
  border-bottom-left-radius: 0;
}

.nav-icon {
  color: currentColor;
  flex-shrink: 0;
}

.nav-divider {
  border-top: 1px solid var(--border);
}

.sidebar-bottom {
  border-top: 1px solid var(--border);
}

.btn-logout {
  color: var(--muted);
  border-radius: var(--radius-md);
  transition:
    color 0.15s ease,
    background-color 0.15s ease;
}

.btn-logout:hover {
  color: #b0394f;
  background-color: color-mix(in srgb, #b0394f 10%, transparent);
}

:global(:root.dark) .btn-logout:hover,
:global(:root[data-theme="dark"]) .btn-logout:hover {
  color: #e68a9a;
  background-color: color-mix(in srgb, #e68a9a 12%, transparent);
}
</style>
