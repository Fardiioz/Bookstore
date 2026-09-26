<template>
  <!-- Full Screen Edge-to-Edge Container -->
  <div class="page h-full flex flex-col overflow-hidden">
    <div class="chat-shell flex-1 flex overflow-hidden relative">
      <!-- ================= SIDEBAR ADMIN ================= -->
      <aside
        class="admin-side border-r flex flex-col shrink-0 z-10 relative select-none transition-[width] duration-200"
        :class="isCollapsed ? 'w-16' : 'w-72 sm:w-80'"
      >
        <!-- Expand / Collapse Toggle -->
        <button
          type="button"
          @click="isCollapsed = !isCollapsed"
          class="collapse-btn absolute -right-3.5 top-1/2 -translate-y-1/2 z-30 w-7 h-7 flex items-center justify-center"
          :title="
            isCollapsed ? 'Perbesar Sidebar Admin' : 'Kecilkan Sidebar Admin'
          "
        >
          <LucideChevronRight v-if="isCollapsed" class="w-3.5 h-3.5" />
          <LucideChevronLeft v-else class="w-3.5 h-3.5" />
        </button>

        <!-- Sidebar Header -->
        <div class="admin-side-head p-4 flex items-center gap-2 shrink-0">
          <LucideMessageCircle class="w-4 h-4 shrink-0" />
          <div v-if="!isCollapsed" class="overflow-hidden">
            <h3 class="conv-name font-semibold text-sm leading-tight">
              Daftar Admin
            </h3>
            <p class="muted text-[11px]">Pilih admin customer service</p>
          </div>
        </div>

        <!-- Admin List -->
        <div class="flex-1 overflow-y-auto p-2 space-y-1.5 custom-scrollbar">
          <div v-if="loadingAdmins" class="muted text-center py-8 text-xs">
            Memuat admin...
          </div>

          <div
            v-for="admin in admins"
            :key="admin.id"
            @click="selectAdmin(admin)"
            class="admin-item flex items-center gap-3 cursor-pointer group relative"
            :class="[
              selectedAdmin?.id === admin.id ? 'admin-item--active' : '',
              isCollapsed ? 'justify-center p-2.5' : 'p-3',
            ]"
          >
            <div class="relative shrink-0">
              <div class="avatar w-9 h-9 text-sm">
                <img
                  v-if="admin.foto"
                  :src="admin.foto"
                  :alt="admin.name"
                  class="w-full h-full object-cover"
                />
                <span v-else>{{ admin.name.charAt(0).toUpperCase() }}</span>
              </div>
              <span
                class="status-dot absolute -bottom-0.5 -right-0.5 w-3 h-3 rounded-full"
              ></span>
            </div>

            <div v-if="!isCollapsed" class="min-w-0 flex-1 text-left">
              <div class="flex items-center justify-between">
                <h4 class="conv-name font-semibold text-xs truncate">
                  {{ admin.name }}
                </h4>
                <span class="online-text text-[10px] font-medium">Online</span>
              </div>
              <p class="role-text text-[11px] truncate">
                {{ admin.role_title }}
              </p>
              <p class="muted text-[11px] truncate mt-0.5">
                {{ admin.last_message }}
              </p>
            </div>

            <span
              v-if="isCollapsed"
              class="tooltip absolute left-14 text-xs font-semibold px-2.5 py-1 whitespace-nowrap opacity-0 group-hover:opacity-100 transition-opacity pointer-events-none z-30"
            >
              {{ admin.name }} ({{ admin.role_title }})
            </span>
          </div>
        </div>
      </aside>

      <!-- ================= MAIN CHAT AREA ================= -->
      <main class="chat-main flex-1 flex flex-col min-w-0 relative">
        <template v-if="selectedAdmin">
          <!-- Chat Header -->
          <div
            class="chat-header p-4 flex items-center justify-between shrink-0"
          >
            <div class="flex items-center gap-3">
              <div class="avatar w-9 h-9 text-sm">
                <img
                  v-if="selectedAdmin.foto"
                  :src="selectedAdmin.foto"
                  :alt="selectedAdmin.name"
                  class="w-full h-full object-cover"
                />
                <span v-else>{{
                  selectedAdmin.name.charAt(0).toUpperCase()
                }}</span>
              </div>
              <div>
                <h4 class="conv-name font-semibold text-sm leading-tight">
                  {{ selectedAdmin.name }}
                </h4>
                <div class="flex items-center gap-2 text-[11px]">
                  <span class="role-text font-medium">{{
                    selectedAdmin.role_title
                  }}</span>
                  <span class="muted">&middot;</span>
                  <span class="online-text font-medium flex items-center gap-1">
                    <span class="live-dot w-1.5 h-1.5 rounded-full"></span>
                    Online
                  </span>
                </div>
              </div>
            </div>

            <div class="flex items-center gap-2">
              <!-- Selection Toolbar -->
              <transition name="fade">
                <div
                  v-if="selectedIds.size > 0"
                  class="flex items-center gap-2"
                >
                  <span class="muted text-xs font-medium"
                    >{{ selectedIds.size }} dipilih</span
                  >
                  <button
                    type="button"
                    @click="cancelSelection"
                    class="btn-ghost text-xs px-3 py-1.5 font-semibold"
                  >
                    Batal
                  </button>
                  <button
                    type="button"
                    @click.stop="openDeleteMenu"
                    class="btn-danger inline-flex items-center gap-1.5 text-xs px-3 py-1.5 font-semibold"
                  >
                    <LucideTrash2 class="w-3.5 h-3.5" />
                    Hapus
                  </button>
                </div>
              </transition>

              <button
                v-if="isCollapsed"
                type="button"
                @click="isCollapsed = false"
                class="btn-outline inline-flex items-center gap-1.5 text-xs font-semibold px-3 py-1.5"
              >
                <LucideUsers class="w-3.5 h-3.5" />
                Ganti Admin
              </button>
            </div>
          </div>

          <!-- Select Mode Info Banner -->
          <transition name="slide-down">
            <div
              v-if="selectedIds.size > 0"
              class="select-banner px-4 py-2 text-[11px] font-medium flex items-center gap-2"
            >
              <LucideInfo class="w-3.5 h-3.5 shrink-0" />
              <span
                >Mode seleksi aktif — ketuk bubble untuk pilih/batal. Tekan
                <strong>Batal</strong> untuk keluar.</span
              >
            </div>
          </transition>

          <!-- Messages Stream -->
          <div
            ref="chatContainer"
            class="msg-area flex-1 p-4 sm:p-6 overflow-y-auto space-y-3.5 custom-scrollbar"
            @click="closeContextMenu"
          >
            <div
              v-if="loadingChats && chats.length === 0"
              class="muted text-center py-12 text-xs"
            >
              Memuat percakapan...
            </div>
            <div
              v-else-if="chats.length === 0"
              class="muted text-center py-16 text-xs"
            >
              Belum ada obrolan khusus dengan {{ selectedAdmin.name }}. Tulis
              pesan pertama Anda!
            </div>

            <div
              v-for="chat in chats"
              :key="chat.id"
              :class="chat.sender === 'user' ? 'justify-end' : 'justify-start'"
              class="flex items-end gap-2.5 group/msg"
            >
              <!-- Admin Avatar -->
              <div
                v-if="chat.sender === 'admin'"
                class="mini-avatar mini-avatar--admin w-7 h-7 text-xs shrink-0"
              >
                A
              </div>

              <!-- Bubble + delete icon wrapper -->
              <div
                :class="
                  chat.sender === 'user' ? 'flex-row-reverse' : 'flex-row'
                "
                class="flex items-end gap-1.5 max-w-xs sm:max-w-md relative"
              >
                <!-- Trash icon on hover (own messages only) -->
                <button
                  v-if="chat.sender === 'user' && selectedIds.size === 0"
                  type="button"
                  @click.stop="openContextMenuSingle(chat, $event)"
                  class="trash-hover opacity-0 group-hover/msg:opacity-100 w-6 h-6 flex items-center justify-center shrink-0 transition-opacity"
                  title="Pilihan hapus"
                >
                  <LucideTrash2 class="w-3 h-3" />
                </button>

                <!-- Bubble -->
                <div
                  @mousedown="startLongPress(chat)"
                  @mouseup="cancelLongPress"
                  @mouseleave="cancelLongPress"
                  @touchstart.prevent="startLongPress(chat)"
                  @touchend="cancelLongPress"
                  @touchcancel="cancelLongPress"
                  @click.stop="onBubbleClick(chat)"
                  :class="[
                    chat.sender === 'user' ? 'bubble--user' : 'bubble--admin',
                    selectedIds.has(chat.id) ? 'bubble--selected' : '',
                    selectedIds.size > 0 ? 'cursor-pointer' : 'cursor-default',
                  ]"
                  class="bubble p-3.5 text-sm space-y-1 select-none transition-all duration-150"
                >
                  <div
                    v-if="selectedIds.has(chat.id)"
                    class="flex items-center gap-1 mb-1"
                  >
                    <LucideCheck class="w-3 h-3" />
                    <span class="text-[10px] font-semibold opacity-75"
                      >Dipilih</span
                    >
                  </div>
                  <p class="font-semibold text-[10px] opacity-75">
                    {{ chat.sender === "admin" ? selectedAdmin.name : "Saya" }}
                  </p>
                  <p class="leading-relaxed whitespace-pre-wrap">
                    {{ chat.pesan }}
                  </p>
                  <p class="text-[10px] text-right opacity-60">
                    {{ chat.created_at }}
                  </p>
                </div>
              </div>

              <!-- User Avatar -->
              <div
                v-if="chat.sender === 'user'"
                class="mini-avatar mini-avatar--user w-7 h-7 text-xs shrink-0"
              >
                Me
              </div>
            </div>
          </div>

          <!-- Form Kirim Pesan -->
          <form
            @submit.prevent="sendMessage"
            class="reply-form p-3.5 flex items-center gap-2.5 shrink-0 relative z-10"
          >
            <input
              v-model="pesanInput"
              type="text"
              :placeholder="`Tulis pesan untuk ${selectedAdmin.name}...`"
              required
              class="field flex-grow px-4 py-3 text-sm"
            />
            <button
              type="submit"
              :disabled="sending"
              class="btn-primary inline-flex items-center gap-1.5 px-5 py-3 text-sm font-semibold shrink-0 disabled:opacity-50 disabled:cursor-not-allowed"
            >
              <LucideSend class="w-4 h-4" />
              Kirim
            </button>
          </form>
        </template>

        <!-- Belum ada admin dipilih -->
        <template v-else>
          <div
            class="flex-1 flex flex-col items-center justify-center p-8 text-center"
          >
            <div
              class="empty-icon w-16 h-16 flex items-center justify-center mb-4"
            >
              <LucideMessageCircle class="w-7 h-7" :stroke-width="1.5" />
            </div>
            <h3 class="empty-title text-lg font-semibold mb-1">
              Selamat Datang di Live Chat Bantuan
            </h3>
            <p class="muted text-xs max-w-sm leading-relaxed mb-6">
              Silakan pilih salah satu Admin Support dari daftar di sebelah kiri
              untuk mulai berkonsultasi.
            </p>
            <button
              v-if="admins.length > 0"
              type="button"
              @click="selectAdmin(admins[0])"
              class="btn-primary inline-flex items-center gap-2 px-5 py-2.5 text-sm font-semibold"
            >
              Mulai Obrolan Dengan {{ admins[0].name }}
              <LucideArrowRight class="w-4 h-4" />
            </button>
          </div>
        </template>
      </main>
    </div>

    <!-- ==================== MENU HAPUS ==================== -->
    <transition name="pop">
      <div
        v-if="deleteMenu.show"
        :style="{ top: deleteMenu.y + 'px', left: deleteMenu.x + 'px' }"
        class="ctx-menu fixed z-50 overflow-hidden min-w-[220px]"
        @click.stop
      >
        <div class="ctx-menu-head px-4 py-2.5">
          <p class="muted text-[10px] font-semibold uppercase tracking-wide">
            Pilihan Hapus
          </p>
          <p class="muted text-[10px] mt-0.5">
            {{ deleteMenu.count }} pesan dipilih
          </p>
        </div>

        <!-- Hapus untuk Saya -->
        <button
          type="button"
          @click="doDeleteForMe"
          :disabled="processing"
          class="ctx-item w-full flex items-center gap-3 px-4 py-3 text-xs disabled:opacity-50"
        >
          <LucideEyeOff class="w-4 h-4 shrink-0" />
          <div class="text-left">
            <p class="font-semibold">Hapus untuk Saya</p>
            <p class="muted text-[10px] font-normal mt-0.5">
              Hanya Anda yang tidak akan melihat pesan ini
            </p>
          </div>
        </button>

        <!-- Hapus untuk Semua (hanya pesan sendiri) -->
        <button
          v-if="deleteMenu.canDeleteForAll"
          type="button"
          @click="doDeleteForAll"
          :disabled="processing"
          class="ctx-item ctx-item--danger w-full flex items-center gap-3 px-4 py-3 text-xs disabled:opacity-50 border-t"
        >
          <LucideTrash2 class="w-4 h-4 shrink-0" />
          <div class="text-left">
            <p class="font-semibold">Hapus untuk Semua Orang</p>
            <p class="muted text-[10px] font-normal mt-0.5">
              Pesan dihapus dari percakapan semua orang
            </p>
          </div>
        </button>

        <!-- Batal -->
        <button
          type="button"
          @click="closeContextMenu"
          class="ctx-item ctx-item--cancel w-full flex items-center gap-3 px-4 py-2.5 text-xs border-t"
        >
          <LucideX class="w-4 h-4" />
          <span class="font-semibold">Batal</span>
        </button>
      </div>
    </transition>

    <!-- Backdrop menu hapus -->
    <div
      v-if="deleteMenu.show"
      class="fixed inset-0 z-40"
      @click="closeContextMenu"
    ></div>
  </div>
</template>

<script setup lang="ts">
import {
  ChevronLeft as LucideChevronLeft,
  ChevronRight as LucideChevronRight,
  MessageCircle as LucideMessageCircle,
  Users as LucideUsers,
  Info as LucideInfo,
  Trash2 as LucideTrash2,
  Check as LucideCheck,
  Send as LucideSend,
  ArrowRight as LucideArrowRight,
  EyeOff as LucideEyeOff,
  X as LucideX,
} from "lucide-vue-next";

definePageMeta({
  middleware: "auth",
});

const api = useApi();

const admins = ref<any[]>([]);
const selectedAdmin = ref<any | null>(null);
const isCollapsed = ref(false);

const chats = ref<any[]>([]);
const pesanInput = ref("");
const loadingAdmins = ref(true);
const loadingChats = ref(false);
const sending = ref(false);
const chatContainer = ref<HTMLElement | null>(null);
let pollTimer: any = null;

// ── Selection & Long Press ────────────────────────────────────────
const selectedIds = ref<Set<number>>(new Set());
let longPressTimer: ReturnType<typeof setTimeout> | null = null;
let longPressTriggered = false;

/**
 * Start 500ms long press timer — entering select mode on hold
 */
const startLongPress = (chat: any) => {
  longPressTriggered = false;
  longPressTimer = setTimeout(() => {
    longPressTriggered = true;
    // Vibrate on mobile if supported
    if (navigator.vibrate) navigator.vibrate(60);
    // Enter select mode with this message pre-selected
    const next = new Set(selectedIds.value);
    next.add(chat.id);
    selectedIds.value = next;
  }, 500);
};

const cancelLongPress = () => {
  if (longPressTimer) {
    clearTimeout(longPressTimer);
    longPressTimer = null;
  }
};

/**
 * On normal click/tap:
 * - If already in select mode → toggle this bubble
 * - If not in select mode → do nothing (long press needed)
 */
const onBubbleClick = (chat: any) => {
  if (longPressTriggered) {
    longPressTriggered = false;
    return; // already handled in long press
  }
  if (selectedIds.value.size > 0) {
    toggleSelect(chat);
  }
  // else: normal click, do nothing
};

const toggleSelect = (chat: any) => {
  const next = new Set(selectedIds.value);
  if (next.has(chat.id)) {
    next.delete(chat.id);
  } else {
    next.add(chat.id);
  }
  selectedIds.value = next;
};

const cancelSelection = () => {
  selectedIds.value = new Set();
  closeContextMenu();
};

// ── Delete Context Menu ───────────────────────────────────────────
interface DeleteMenu {
  show: boolean;
  x: number;
  y: number;
  count: number;
  canDeleteForAll: boolean;
  singleChat: any | null;
}

const deleteMenu = ref<DeleteMenu>({
  show: false,
  x: 0,
  y: 0,
  count: 0,
  canDeleteForAll: false,
  singleChat: null,
});

const processing = ref(false);

/**
 * Open menu from the header "Hapus" button (batch mode)
 */
const openDeleteMenu = () => {
  // Work out if ALL selected are own messages (sender === 'user')
  const selectedChats = chats.value.filter((c: any) =>
    selectedIds.value.has(c.id),
  );
  const allOwn = selectedChats.every((c: any) => c.sender === "user");

  // Position in center of screen
  const x = Math.max(window.innerWidth / 2 - 110, 16);
  const y = Math.max(window.innerHeight / 2 - 120, 80);

  deleteMenu.value = {
    show: true,
    x,
    y,
    count: selectedIds.value.size,
    canDeleteForAll: allOwn,
    singleChat: null,
  };
};

/**
 * Open menu from the trash icon on a single message bubble (hover)
 */
const openContextMenuSingle = (chat: any, event: MouseEvent) => {
  // Pre-select just this message
  selectedIds.value = new Set([chat.id]);

  const rect = (event.target as HTMLElement).getBoundingClientRect();
  let x = rect.left - 220; // to the left of the trash icon
  let y = rect.top - 10;

  // Clamp to viewport
  if (x < 8) x = 8;
  if (y + 160 > window.innerHeight) y = window.innerHeight - 170;

  deleteMenu.value = {
    show: true,
    x,
    y,
    count: 1,
    canDeleteForAll: chat.sender === "user",
    singleChat: chat,
  };
};

const closeContextMenu = () => {
  deleteMenu.value.show = false;
};

/** Hapus untuk Saya — soft hide via chat_deletions table */
const doDeleteForMe = async () => {
  if (processing.value) return;
  processing.value = true;
  try {
    await api.post("/api/chats/delete-for-me", {
      ids: Array.from(selectedIds.value),
    });
    const removed = new Set(selectedIds.value);
    chats.value = chats.value.filter((c: any) => !removed.has(c.id));
    cancelSelection();
  } catch (err: any) {
    const toast = useToast();
    toast.error(err.data?.message || "Gagal menyembunyikan pesan");
  } finally {
    processing.value = false;
    closeContextMenu();
  }
};

/** Hapus untuk Semua Orang — hard delete (only own messages) */
const doDeleteForAll = async () => {
  if (processing.value) return;
  processing.value = true;
  try {
    await api.post("/api/chats/delete-for-all", {
      ids: Array.from(selectedIds.value),
    });
    const removed = new Set(selectedIds.value);
    chats.value = chats.value.filter((c: any) => !removed.has(c.id));
    cancelSelection();
  } catch (err: any) {
    const toast = useToast();
    toast.error(err.data?.message || "Gagal menghapus pesan untuk semua");
  } finally {
    processing.value = false;
    closeContextMenu();
  }
};

// ── Core Chat Logic ───────────────────────────────────────────────
const scrollToBottom = () => {
  nextTick(() => {
    if (chatContainer.value) {
      chatContainer.value.scrollTop = chatContainer.value.scrollHeight;
    }
  });
};

const fetchAdmins = async () => {
  loadingAdmins.value = true;
  try {
    const res = await api.get("/api/admins");
    admins.value = res.data || [];
  } catch (e) {
    console.error("Failed to fetch admins:", e);
  } finally {
    if (!admins.value || admins.value.length === 0) {
      admins.value = [
        {
          id: 1,
          name: "Administrator Toko Buku",
          username: "admin",
          role_title: "CS Support Utama",
          is_online: true,
          last_message: "Ada yang bisa saya bantu?",
          foto: null,
        },
      ];
    }
    loadingAdmins.value = false;
  }
};

const selectAdmin = async (admin: any) => {
  selectedAdmin.value = admin;
  isCollapsed.value = true;
  cancelSelection();
  await fetchChats();
};

const fetchChats = async () => {
  if (!selectedAdmin.value) return;
  loadingChats.value = true;
  try {
    const res = await api.get("/api/chats", {
      params: { admin_id: selectedAdmin.value.id },
    });
    chats.value = res.data || [];
    if (chats.value.length > 0) {
      try {
        await api.post("/api/chats/mark-as-read");
      } catch (err) {}
    }
    scrollToBottom();
  } catch (e: any) {
    if (
      e.status === 401 ||
      e.statusCode === 401 ||
      e.response?.status === 401
    ) {
      if (pollTimer) clearInterval(pollTimer);
    } else {
      console.error("Failed to fetch chats:", e);
    }
  } finally {
    loadingChats.value = false;
  }
};

const sendMessage = async () => {
  if (!pesanInput.value.trim() || !selectedAdmin.value) return;
  sending.value = true;
  const msg = pesanInput.value;
  pesanInput.value = "";
  try {
    await api.post("/api/chats", {
      admin_id: selectedAdmin.value.id,
      pesan: msg,
    });
    await fetchChats();
  } catch (err: any) {
    const toast = useToast();
    toast.error(err.data?.message || "Gagal mengirim pesan");
  } finally {
    sending.value = false;
  }
};

onMounted(async () => {
  await fetchAdmins();
  if (admins.value.length > 0) selectAdmin(admins.value[0]);
  pollTimer = setInterval(() => {
    if (selectedAdmin.value) fetchChats();
  }, 3000);
});

onUnmounted(() => {
  if (pollTimer) clearInterval(pollTimer);
});
</script>

<style scoped>
/* Semua warna mengikuti token di main.css, jadi mode terang dan gelap sinkron */
.page {
  --danger: #b0394f;
  --success: #4d6b3f;
  background-color: var(--bg);
  color: var(--text);
}

:global(:root.dark) .page,
:global(:root[data-theme="dark"]) .page {
  --danger: #e68a9a;
  --success: #b3c0a4;
}

.chat-shell {
  background-color: var(--bg);
}

.muted {
  color: var(--muted);
}

.conv-name {
  color: var(--text);
}

.role-text {
  color: var(--text-2);
}

.online-text {
  color: var(--success);
}

.live-dot {
  background-color: var(--success);
}

/* sidebar admin */
.admin-side {
  background-color: var(--surface);
  border-color: var(--border);
}

.admin-side-head {
  border-bottom: 1px solid var(--border);
  color: var(--text-2);
}

.collapse-btn {
  background-color: var(--primary);
  color: var(--on-primary);
  border: 2px solid var(--surface);
  border-radius: 9999px;
  box-shadow: 0 1px 4px rgb(0 0 0 / 0.15);
  transition: transform 0.15s ease;
}

.collapse-btn:hover {
  transform: translateY(-50%) scale(1.08);
}

.admin-item {
  border: 1px solid transparent;
  border-radius: var(--radius-md);
  transition:
    background-color 0.15s ease,
    border-color 0.15s ease;
}

.admin-item:hover {
  background-color: var(--inset);
}

.admin-item--active {
  background-color: var(--inset);
  border-color: var(--border-strong);
}

.avatar {
  border-radius: var(--radius-md);
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: var(--primary);
  color: var(--on-primary);
  font-weight: 600;
  flex-shrink: 0;
}

.status-dot {
  background-color: var(--success);
  box-shadow: 0 0 0 2px var(--surface);
}

.tooltip {
  background-color: var(--text);
  color: var(--surface);
  border-radius: var(--radius-md);
  box-shadow: 0 4px 10px rgb(0 0 0 / 0.2);
}

/* header chat */
.chat-main {
  background-color: var(--bg);
}

.chat-header {
  background-color: var(--surface);
  border-bottom: 1px solid var(--border);
}

.select-banner {
  background-color: var(--inset);
  border-bottom: 1px solid var(--border);
  color: var(--text-2);
}

.btn-ghost {
  color: var(--text-2);
  border-radius: var(--radius-md);
  transition:
    background-color 0.15s ease,
    color 0.15s ease;
}

.btn-ghost:hover {
  background-color: var(--inset);
  color: var(--text);
}

.btn-outline {
  color: var(--text);
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
  transition: background-color 0.15s ease;
}

.btn-outline:hover {
  background-color: var(--inset);
}

.btn-danger {
  color: #ffffff;
  background-color: var(--danger);
  border-radius: var(--radius-md);
  transition: opacity 0.15s ease;
}

.btn-danger:hover {
  opacity: 0.9;
}

.btn-primary {
  background-color: var(--primary);
  color: var(--on-primary);
  border: 1px solid var(--primary);
  border-radius: var(--radius-md);
  transition:
    background-color 0.15s ease,
    transform 0.1s ease;
}

.btn-primary:hover:not(:disabled) {
  background-color: var(--primary-hover);
}

.btn-primary:active:not(:disabled) {
  transform: translateY(1px);
}

/* pesan */
.msg-area {
  background-color: var(--bg);
}

.mini-avatar {
  border-radius: 9999px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
}

.mini-avatar--admin {
  background-color: var(--primary);
  color: var(--on-primary);
}

.mini-avatar--user {
  background-color: var(--inset);
  color: var(--text-2);
  border: 1px solid var(--border-strong);
}

.bubble {
  border-radius: var(--radius-lg);
}

.bubble--user {
  background-color: var(--primary);
  color: var(--on-primary);
  border-bottom-right-radius: 2px;
}

.bubble--admin {
  background-color: var(--surface);
  color: var(--text);
  border: 1px solid var(--border);
  border-bottom-left-radius: 2px;
}

.bubble--selected {
  outline: 2px solid var(--danger);
  outline-offset: 2px;
  opacity: 0.85;
}

.trash-hover {
  color: #ffffff;
  background-color: var(--danger);
  border-radius: 9999px;
}

/* form kirim */
.reply-form {
  background-color: var(--surface);
  border-top: 1px solid var(--border);
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

/* keadaan kosong */
.empty-icon {
  background-color: var(--inset);
  color: var(--text-2);
  border-radius: var(--radius-lg);
}

.empty-title {
  font-family: var(--font-display);
  color: var(--text);
}

/* menu konteks hapus */
.ctx-menu {
  background-color: var(--surface);
  color: var(--text);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  box-shadow: 0 8px 24px rgb(0 0 0 / 0.18);
}

.ctx-menu-head {
  border-bottom: 1px solid var(--border);
}

.ctx-item {
  color: var(--text);
  border-color: var(--border);
  transition: background-color 0.15s ease;
}

.ctx-item:hover:not(:disabled) {
  background-color: var(--inset);
}

.ctx-item--danger {
  color: var(--danger);
}

.ctx-item--cancel {
  color: var(--muted);
}

/* transisi */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-down-enter-active,
.slide-down-leave-active {
  transition: all 0.2s;
}
.slide-down-enter-from,
.slide-down-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}

.pop-enter-active {
  transition: all 0.15s cubic-bezier(0.34, 1.56, 0.64, 1);
}
.pop-leave-active {
  transition: all 0.1s ease-in;
}
.pop-enter-from,
.pop-leave-to {
  opacity: 0;
  transform: scale(0.85);
}
</style>
