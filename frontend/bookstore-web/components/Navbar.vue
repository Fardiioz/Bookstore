<template>
  <header class="site-header sticky top-0 z-50">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="flex items-center justify-between h-16">
        <!-- Logo -->
        <NuxtLink to="/" class="brand">BookStore</NuxtLink>

        <!-- Public Navigation Links -->
        <nav
          class="hidden md:flex items-center gap-7 text-sm"
          aria-label="Navigasi utama"
        >
          <NuxtLink
            to="/"
            class="nav-link"
            exact-active-class="nav-link--active"
            >Home</NuxtLink
          >
          <NuxtLink
            to="/user/katalog"
            class="nav-link"
            active-class="nav-link--active"
            >Katalog Buku</NuxtLink
          >
          <NuxtLink
            to="/our-story"
            class="nav-link"
            active-class="nav-link--active"
            >Our Story</NuxtLink
          >
          <NuxtLink to="/blog" class="nav-link" active-class="nav-link--active"
            >Blog</NuxtLink
          >
          <NuxtLink
            to="/contact"
            class="nav-link"
            active-class="nav-link--active"
            >Contact</NuxtLink
          >
        </nav>

        <!-- Right User Actions -->
        <div class="flex items-center gap-3">
          <!-- Cart Icon -->
          <NuxtLink
            to="/user/keranjang"
            class="icon-btn relative"
            title="Keranjang"
            aria-label="Keranjang belanja"
          >
            <LucideShoppingCart class="w-4 h-4" />
            <span
              v-if="cartStore.totalItems > 0"
              class="cart-badge absolute -top-1.5 -right-1.5 min-w-[1.125rem] h-[1.125rem] px-1 text-[10px] font-bold rounded-full flex items-center justify-center"
            >
              {{ cartStore.totalItems }}
            </span>
          </NuxtLink>

          <!-- Logged In User / Admin Menu -->
          <div v-if="authStore.isAuthenticated" class="flex items-center gap-3">
            <NuxtLink
              v-if="authStore.isAdmin"
              to="/admin/kategori"
              class="btn-outline hidden sm:inline-flex items-center gap-1.5 px-3 py-1.5 text-xs font-semibold"
            >
              <LucideLayoutDashboard class="w-3.5 h-3.5" />
              Dashboard Admin
            </NuxtLink>

            <NuxtLink
              v-else
              to="/user/riwayat"
              class="btn-outline hidden sm:inline-flex items-center gap-1.5 px-3 py-1.5 text-xs font-semibold"
            >
              <LucideClipboardList class="w-3.5 h-3.5" />
              Riwayat Pesanan
            </NuxtLink>

            <NuxtLink
              :to="authStore.isAdmin ? '/admin/chat' : '/user/chat'"
              class="icon-btn"
              title="Live Chat"
              aria-label="Live Chat"
            >
              <LucideMessageCircle class="w-4 h-4" />
            </NuxtLink>

            <div class="user-sep flex items-center gap-2 pl-3">
              <div class="avatar shrink-0">
                {{ authStore.user?.name?.charAt(0).toUpperCase() }}
              </div>
              <button
                type="button"
                @click="authStore.logout()"
                class="btn-ghost px-2.5 py-1 text-xs font-semibold"
              >
                Keluar
              </button>
            </div>
          </div>

          <!-- Guest Login / Register -->
          <div v-else class="flex items-center gap-2">
            <NuxtLink
              to="/login"
              class="btn-ghost px-4 py-2 text-sm font-semibold"
            >
              Login
            </NuxtLink>
            <NuxtLink
              to="/register"
              class="btn-primary px-4 py-2 text-sm font-semibold"
            >
              Register
            </NuxtLink>
          </div>
        </div>
      </div>
    </div>
  </header>
</template>

<script setup lang="ts">
import {
  LucideShoppingCart,
  LucideLayoutDashboard,
  LucideClipboardList,
  LucideMessageCircle,
} from "lucide-vue-next";

const authStore = useAuthStore();
const cartStore = useCartStore();

onMounted(async () => {
  if (!authStore.initialized) {
    await authStore.fetchUser();
  }
});
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
  font-size: 1.25rem;
  font-weight: 600;
  letter-spacing: -0.01em;
  color: var(--text);
}

/* tautan navigasi: garis bawah sebagai penanda halaman aktif */
.nav-link {
  color: var(--muted);
  font-weight: 500;
  padding: 0.375rem 0.125rem;
  border-bottom: 2px solid transparent;
  transition:
    color 0.15s ease,
    border-color 0.15s ease;
}

.nav-link:hover {
  color: var(--text);
}

.nav-link--active {
  color: var(--text);
  font-weight: 600;
  border-bottom-color: var(--primary);
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

.cart-badge {
  background-color: var(--primary);
  color: var(--on-primary);
  box-shadow: 0 0 0 2px var(--surface);
}

.btn-outline {
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
  color: var(--text);
  transition:
    background-color 0.15s ease,
    border-color 0.15s ease;
}

.btn-outline:hover {
  background-color: var(--inset);
  border-color: var(--text-2);
}

.btn-ghost {
  border-radius: var(--radius-md);
  color: var(--text-2);
  transition:
    background-color 0.15s ease,
    color 0.15s ease;
}

.btn-ghost:hover {
  background-color: var(--inset);
  color: var(--text);
}

.btn-primary {
  border: 1px solid var(--primary);
  border-radius: var(--radius-md);
  background-color: var(--primary);
  color: var(--on-primary);
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

.user-sep {
  border-left: 1px solid var(--border);
}

.avatar {
  width: 2rem;
  height: 2rem;
  border-radius: 9999px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: var(--primary);
  color: var(--on-primary);
  font-size: 0.75rem;
  font-weight: 600;
}
</style>
