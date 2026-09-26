<template>
  <div
    v-if="authStore.isAuthenticated && !isChatPage"
    class="fixed bottom-20 md:bottom-6 right-4 md:right-6 z-50 select-none"
  >
    <button
      type="button"
      @click="navigateToChat"
      class="chat-fab relative flex items-center gap-2 px-4 py-2.5 text-xs font-semibold"
      :aria-label="
        unreadCount > 0
          ? `Buka chat, ${unreadCount} pesan belum dibaca`
          : 'Buka chat'
      "
    >
      <LucideMessageCircle class="w-4 h-4" :stroke-width="2" />
      <span class="hidden sm:inline">Live Chat</span>

      <!-- Unread Badge Counter -->
      <span
        v-if="unreadCount > 0"
        class="chat-fab-badge absolute -top-2 -right-2 min-w-[1.375rem] h-[1.375rem] px-1 text-[11px] font-bold rounded-full flex items-center justify-center"
      >
        {{ unreadCount > 99 ? "99+" : unreadCount }}
      </span>
    </button>
  </div>
</template>

<script setup lang="ts">
import { LucideMessageCircle } from "lucide-vue-next";

const route = useRoute();
const authStore = useAuthStore();
const api = useApi();

const unreadCount = ref(0);
let pollTimer: any = null;

const isChatPage = computed(() => {
  return route.path.includes("/chat");
});

const navigateToChat = async () => {
  unreadCount.value = 0;
  try {
    await api.post("/api/chats/mark-as-read");
  } catch (e) {}
  if (authStore.isAdmin) {
    navigateTo("/admin/chat");
  } else {
    navigateTo("/user/chat");
  }
};

const checkUnread = async () => {
  if (!authStore.isAuthenticated || isChatPage.value) {
    unreadCount.value = 0;
    return;
  }
  try {
    const res: any = await api.get("/api/chats/unread-count");
    unreadCount.value = res.unread_count ?? res.data?.unread_count ?? 0;
  } catch (e) {
    unreadCount.value = 0;
  }
};

onMounted(() => {
  if (authStore.isAuthenticated) {
    checkUnread();
    pollTimer = setInterval(checkUnread, 5000);
  }
});

onUnmounted(() => {
  if (pollTimer) clearInterval(pollTimer);
});
</script>

<style scoped>
/* Warna mengikuti token di main.css, jadi mode terang dan gelap otomatis sinkron */
.chat-fab {
  background-color: var(--primary);
  color: var(--on-primary);
  border: 1px solid var(--primary);
  border-radius: var(--radius-lg);
  transition:
    background-color 0.15s ease,
    border-color 0.15s ease,
    transform 0.1s ease;
}

.chat-fab:hover {
  background-color: var(--primary-hover);
  border-color: var(--primary-hover);
}

.chat-fab:active {
  transform: translateY(1px);
}

.chat-fab-badge {
  background-color: #b0394f;
  color: #ffffff;
  /* cincin sewarna latar halaman agar badge terpisah dari tombol */
  box-shadow: 0 0 0 2px var(--bg);
}
</style>
