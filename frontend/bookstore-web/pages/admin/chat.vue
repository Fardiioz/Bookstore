<template>
  <div class="page h-full flex flex-col overflow-hidden">
    <!-- Admin Header Mini Bar -->
    <div class="topbar px-4 py-3 flex items-center justify-between shrink-0">
      <div class="flex items-center gap-3">
        <h1 class="page-title text-base">Live Chat Admin</h1>
        <span
          class="live-badge inline-flex items-center gap-1.5 px-2 py-0.5 text-[11px] font-medium"
        >
          <span class="live-dot w-1.5 h-1.5 rounded-full"></span>
          Live
        </span>
      </div>
    </div>

    <div class="flex-1 flex overflow-hidden relative">
      <!-- Daftar Percakapan Pelanggan -->
      <aside class="conv-side w-full md:w-72 lg:w-80 flex flex-col shrink-0">
        <div class="conv-head p-4 font-semibold text-xs">
          Daftar Obrolan Pelanggan
        </div>

        <div class="flex-grow overflow-y-auto custom-scrollbar">
          <div v-if="loadingUsers" class="state-text text-center py-10 text-xs">
            Memuat pengguna...
          </div>
          <div
            v-else-if="userList.length === 0"
            class="state-text text-center py-10 text-xs"
          >
            Belum ada obrolan masuk.
          </div>
          <div
            v-for="u in userList"
            :key="u.id"
            @click="selectUser(u)"
            class="conv-item p-4 cursor-pointer flex items-center gap-3"
            :class="{ 'conv-item--active': selectedUser?.id === u.id }"
          >
            <div class="avatar w-10 h-10 shrink-0 text-sm">
              <img
                v-if="u.foto"
                :src="u.foto"
                :alt="u.name"
                class="w-full h-full object-cover"
              />
              <span v-else>{{ u.name.charAt(0).toUpperCase() }}</span>
            </div>
            <div class="overflow-hidden flex-grow">
              <div class="flex justify-between items-baseline">
                <h4 class="conv-name font-semibold text-sm truncate">
                  {{ u.name }}
                </h4>
                <span class="state-text text-[10px] ml-2 shrink-0">{{
                  u.last_time
                }}</span>
              </div>
              <div class="flex items-center justify-between mt-0.5">
                <p class="state-text text-xs truncate flex-1">
                  {{ u.last_message || "Obrolan baru" }}
                </p>
                <span
                  v-if="u.unread > 0"
                  class="unread-pill ml-2 min-w-[1.25rem] h-5 px-1.5 text-[11px] font-semibold flex items-center justify-center shrink-0"
                >
                  {{ u.unread }}
                </span>
              </div>
            </div>
          </div>
        </div>
      </aside>

      <!-- Panel Chat -->
      <main class="chat-main flex-1 hidden md:flex flex-col min-w-0 relative">
        <div
          v-if="!selectedUser"
          class="state-text flex-grow flex flex-col items-center justify-center p-8 text-center"
        >
          <LucideMessageCircle class="w-9 h-9 mb-3" :stroke-width="1.5" />
          <h4 class="empty-title font-semibold text-sm">
            Pilih Obrolan Pelanggan
          </h4>
          <p class="text-xs mt-1">
            Pilih pengguna dari daftar di sebelah kiri untuk mulai membaca &amp;
            membalas pesan.
          </p>
        </div>

        <template v-else>
          <!-- Header pengguna aktif -->
          <div class="user-head p-4 flex items-center justify-between">
            <div class="flex items-center gap-3">
              <div class="avatar w-9 h-9 text-xs">
                {{ selectedUser.name.charAt(0).toUpperCase() }}
              </div>
              <div>
                <h4 class="conv-name font-semibold text-sm">
                  {{ selectedUser.name }}
                </h4>
                <p class="state-text text-[11px]">
                  @{{ selectedUser.username }}
                </p>
              </div>
            </div>
          </div>

          <div
            ref="chatContainer"
            class="msg-area flex-grow p-6 overflow-y-auto space-y-4 custom-scrollbar"
          >
            <div
              v-for="chat in chats"
              :key="chat.id"
              :class="chat.sender === 'admin' ? 'justify-end' : 'justify-start'"
              class="flex items-end gap-2"
            >
              <div
                v-if="chat.sender === 'user'"
                class="mini-avatar mini-avatar--user w-7 h-7 text-xs shrink-0"
              >
                U
              </div>

              <div
                :class="
                  chat.sender === 'admin' ? 'bubble--admin' : 'bubble--user'
                "
                class="bubble p-3.5 max-w-sm text-sm space-y-1"
              >
                <p class="font-semibold text-[11px] opacity-75">
                  {{
                    chat.sender === "admin" ? "Saya (Admin)" : selectedUser.name
                  }}
                </p>
                <p class="leading-relaxed whitespace-pre-wrap">
                  {{ chat.pesan }}
                </p>
                <p class="text-[10px] text-right opacity-60">
                  {{ chat.created_at }}
                </p>
              </div>

              <div
                v-if="chat.sender === 'admin'"
                class="mini-avatar mini-avatar--admin w-7 h-7 text-xs shrink-0"
              >
                A
              </div>
            </div>
          </div>

          <!-- Form balasan -->
          <form
            @submit.prevent="sendMessage"
            class="reply-form p-4 flex items-center gap-3 shrink-0 relative z-10"
          >
            <input
              v-model="pesanInput"
              type="text"
              placeholder="Balas pesan ke pelanggan..."
              required
              class="field flex-grow px-4 py-3 text-sm"
            />
            <button
              type="submit"
              :disabled="sending"
              class="btn-primary inline-flex items-center gap-2 px-5 py-3 text-sm font-semibold shrink-0 disabled:opacity-50 disabled:cursor-not-allowed"
            >
              <LucideSend class="w-4 h-4" />
              Balas
            </button>
          </form>
        </template>
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  MessageCircle as LucideMessageCircle,
  Send as LucideSend,
} from "lucide-vue-next";

definePageMeta({
  middleware: "admin",
});

const api = useApi();

const userList = ref<any[]>([]);
const loadingUsers = ref(true);
const selectedUser = ref<any>(null);
const chats = ref<any[]>([]);
const pesanInput = ref("");
const sending = ref(false);
const chatContainer = ref<HTMLElement | null>(null);
let pollTimer: any = null;

const scrollToBottom = () => {
  nextTick(() => {
    if (chatContainer.value) {
      chatContainer.value.scrollTop = chatContainer.value.scrollHeight;
    }
  });
};

const fetchUserList = async () => {
  try {
    const res = await api.get("/api/chats");
    userList.value = res.data || [];
  } catch (e) {
    console.error(e);
  } finally {
    loadingUsers.value = false;
  }
};

const fetchConversation = async () => {
  if (!selectedUser.value) return;
  try {
    const res = await api.get(`/api/chats/user/${selectedUser.value.id}`);
    chats.value = res.data || [];
    scrollToBottom();
  } catch (e) {
    console.error(e);
  }
};

const selectUser = async (u: any) => {
  selectedUser.value = u;
  u.unread = 0;
  try {
    await api.post("/api/chats/mark-as-read", { user_id: u.id });
  } catch (e) {}
  await fetchConversation();
};

const sendMessage = async () => {
  if (!selectedUser.value || !pesanInput.value.trim()) return;

  sending.value = true;
  const msg = pesanInput.value;
  pesanInput.value = "";

  try {
    const toast = useToast();
    await api.post("/api/chats", {
      user_id: selectedUser.value.id,
      pesan: msg,
    });
    toast.success("Pesan berhasil dikirim!");
    await fetchConversation();
  } catch (err: any) {
    const toast = useToast();
    toast.error(err.data?.message || "Gagal membalas pesan");
  } finally {
    sending.value = false;
  }
};

onMounted(() => {
  fetchUserList();
  pollTimer = setInterval(() => {
    fetchUserList();
    if (selectedUser.value) fetchConversation();
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

.page-title {
  font-family: var(--font-display);
  font-weight: 600;
  letter-spacing: -0.01em;
  color: var(--text);
}

.state-text {
  color: var(--muted);
}

.empty-title,
.conv-name {
  color: var(--text);
}

.topbar {
  background-color: var(--surface);
  border-bottom: 1px solid var(--border);
}

.live-badge {
  color: var(--text-2);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
}

.live-dot {
  background-color: var(--success);
}

/* daftar percakapan */
.conv-side {
  background-color: var(--surface);
  border-right: 1px solid var(--border);
}

.conv-head {
  color: var(--text-2);
  border-bottom: 1px solid var(--border);
}

.conv-item {
  border-bottom: 1px solid var(--border);
  border-left: 2px solid transparent;
  transition: background-color 0.15s ease;
}

.conv-item:hover {
  background-color: var(--inset);
}

.conv-item--active {
  background-color: var(--inset);
  border-left-color: var(--primary);
}

.avatar {
  border-radius: 9999px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: var(--primary);
  color: var(--on-primary);
  font-weight: 600;
}

.unread-pill {
  background-color: var(--danger);
  color: #ffffff;
  border-radius: 9999px;
}

:global(:root.dark) .unread-pill,
:global(:root[data-theme="dark"]) .unread-pill {
  color: #1b1828;
}

/* panel chat */
.chat-main {
  background-color: var(--bg);
}

.user-head {
  background-color: var(--surface);
  border-bottom: 1px solid var(--border);
}

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

.mini-avatar--user {
  background-color: var(--inset);
  color: var(--text-2);
  border: 1px solid var(--border-strong);
}

.mini-avatar--admin {
  background-color: var(--primary);
  color: var(--on-primary);
}

.bubble {
  border-radius: var(--radius-lg);
}

.bubble--admin {
  background-color: var(--primary);
  color: var(--on-primary);
  border-bottom-right-radius: 2px;
}

.bubble--user {
  background-color: var(--surface);
  color: var(--text);
  border: 1px solid var(--border);
  border-bottom-left-radius: 2px;
}

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
</style>
