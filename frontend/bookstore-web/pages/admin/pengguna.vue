<template>
  <div
    class="space-y-6 font-sans antialiased text-slate-800 dark:text-slate-200"
  >
    <!-- Header -->
    <header
      :class="
        isDark
          ? 'bg-[#232033] border-[#36324A]'
          : 'bg-[#F5F8EC] border-[#B3C0A4]'
      "
      class="p-6 rounded-xl border transition-colors duration-200"
    >
      <h1
        :class="isDark ? 'text-white' : 'text-[#27233A]'"
        class="text-xl font-serif font-bold tracking-tight"
      >
        Manajemen Pengguna
      </h1>
      <p
        :class="isDark ? 'text-[#B3C0A4]' : 'text-[#505168]'"
        class="text-xs mt-1"
      >
        Kelola hak akses dan informasi pengguna terdaftar dalam sistem toko
        buku.
      </p>
    </header>

    <!-- Table Container -->
    <div
      :class="
        isDark
          ? 'bg-[#232033] border-[#36324A]'
          : 'bg-[#F5F8EC] border-[#B3C0A4]'
      "
      class="rounded-xl border overflow-hidden transition-colors duration-200"
    >
      <!-- Loading State -->
      <div
        v-if="loading"
        :class="isDark ? 'text-[#B3C0A4]' : 'text-[#505168]'"
        class="text-center py-12 text-xs font-medium tracking-wide"
      >
        Memuat data pengguna...
      </div>

      <!-- Table Content -->
      <div v-else class="overflow-x-auto">
        <table class="w-full text-left text-xs border-collapse">
          <thead>
            <tr
              :class="
                isDark
                  ? 'bg-[#1A1825] border-[#36324A] text-[#DCC48E]'
                  : 'bg-[#EAEFD3] border-[#B3C0A4] text-[#27233A]'
              "
              class="border-b font-serif uppercase tracking-wider text-[11px] font-semibold"
            >
              <th scope="col" class="px-5 py-3.5">Foto</th>
              <th scope="col" class="px-5 py-3.5">Nama Lengkap</th>
              <th scope="col" class="px-5 py-3.5">Username</th>
              <th scope="col" class="px-5 py-3.5">Email</th>
              <th scope="col" class="px-5 py-3.5">No. Telp</th>
              <th scope="col" class="px-5 py-3.5">Role</th>
              <th scope="col" class="px-5 py-3.5 text-right">Aksi</th>
            </tr>
          </thead>
          <tbody
            :class="isDark ? 'divide-[#36324A]' : 'divide-[#B3C0A4]/40'"
            class="divide-y font-normal"
          >
            <tr
              v-for="user in users"
              :key="user.id"
              :class="isDark ? 'hover:bg-[#2A273D]' : 'hover:bg-[#EAEFD3]/50'"
              class="transition-colors duration-150"
            >
              <!-- Avatar -->
              <td class="px-5 py-3">
                <div
                  :class="
                    isDark
                      ? 'bg-[#1A1825] border-[#36324A] text-[#DCC48E]'
                      : 'bg-[#EAEFD3] border-[#B3C0A4] text-[#27233A]'
                  "
                  class="w-8 h-8 rounded-full border flex items-center justify-center overflow-hidden shrink-0 font-serif font-bold text-xs"
                >
                  <img
                    v-if="user.foto"
                    :src="user.foto"
                    :alt="user.name"
                    class="w-full h-full object-cover"
                  />
                  <span v-else>{{ user.name.charAt(0).toUpperCase() }}</span>
                </div>
              </td>

              <!-- User Details -->
              <td
                :class="isDark ? 'text-slate-100' : 'text-[#27233A]'"
                class="px-5 py-3 font-medium"
              >
                {{ user.name }}
              </td>
              <td
                :class="isDark ? 'text-slate-400' : 'text-[#505168]'"
                class="px-5 py-3 font-mono text-[11px]"
              >
                @{{ user.username }}
              </td>
              <td
                :class="isDark ? 'text-slate-400' : 'text-[#505168]'"
                class="px-5 py-3"
              >
                {{ user.email }}
              </td>
              <td
                :class="isDark ? 'text-slate-400' : 'text-[#505168]'"
                class="px-5 py-3"
              >
                {{ user.no_telp || "-" }}
              </td>

              <!-- Role Badge -->
              <td class="px-5 py-3">
                <span
                  :class="
                    user.role === 'admin'
                      ? isDark
                        ? 'bg-[#DCC48E]/15 text-[#DCC48E] border-[#DCC48E]/30'
                        : 'bg-[#27233A] text-[#EAEFD3] border-[#27233A]'
                      : isDark
                        ? 'bg-[#1A1825] text-slate-300 border-[#36324A]'
                        : 'bg-[#EAEFD3] text-[#505168] border-[#B3C0A4]'
                  "
                  class="px-2 py-0.5 rounded border text-[10px] font-semibold tracking-wide uppercase"
                >
                  {{ user.role }}
                </span>
              </td>

              <!-- Actions -->
              <td class="px-5 py-3 text-right">
                <div class="flex items-center justify-end gap-2">
                  <button
                    @click="openEditModal(user)"
                    :class="
                      isDark
                        ? 'border-[#36324A] text-slate-300 hover:text-white hover:bg-[#36324A]'
                        : 'border-[#B3C0A4] text-[#27233A] hover:bg-[#EAEFD3]'
                    "
                    class="p-1.5 rounded-lg border transition-all focus:outline-none focus:ring-2 focus:ring-[#DCC48E]"
                    title="Edit Pengguna"
                    aria-label="Edit Pengguna"
                  >
                    <LucidePencil class="w-3.5 h-3.5" />
                  </button>
                  <button
                    @click="deleteUser(user)"
                    :class="
                      isDark
                        ? 'border-[#36324A] text-rose-400 hover:bg-rose-950/30 hover:border-rose-900/50'
                        : 'border-[#B3C0A4] text-rose-700 hover:bg-rose-50 hover:border-rose-200'
                    "
                    class="p-1.5 rounded-lg border transition-all focus:outline-none focus:ring-2 focus:ring-rose-500"
                    title="Hapus Pengguna"
                    aria-label="Hapus Pengguna"
                  >
                    <LucideTrash2 class="w-3.5 h-3.5" />
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Modal Edit User -->
    <div
      v-if="showModal"
      class="fixed inset-0 bg-[#1A1825]/60 backdrop-blur-xs z-50 flex items-center justify-center p-4 transition-opacity"
    >
      <div
        :class="
          isDark
            ? 'bg-[#232033] border-[#36324A] text-slate-100'
            : 'bg-[#F5F8EC] border-[#B3C0A4] text-[#27233A]'
        "
        class="rounded-xl p-6 max-w-md w-full border space-y-5"
      >
        <div
          class="flex items-center justify-between border-b pb-3"
          :class="isDark ? 'border-[#36324A]' : 'border-[#B3C0A4]/40'"
        >
          <h3 class="font-serif font-bold text-base tracking-tight">
            Edit Data Pengguna
          </h3>
          <button
            @click="showModal = false"
            :class="
              isDark
                ? 'text-slate-400 hover:text-white'
                : 'text-[#505168] hover:text-[#27233A]'
            "
            class="w-6 h-6 flex items-center justify-center rounded text-sm transition-colors"
            aria-label="Tutup Modal"
          >
            ✕
          </button>
        </div>

        <div v-if="errorMessage" class="text-xs font-medium text-rose-600">
          {{ errorMessage }}
        </div>

        <form @submit.prevent="saveUser" class="space-y-4">
          <!-- Email: read-only, tidak bisa diubah -->
          <div>
            <label
              :class="isDark ? 'text-slate-300' : 'text-[#505168]'"
              class="block text-xs font-medium mb-1.5"
            >
              Email (tidak dapat diubah)
            </label>
            <input
              :value="form.email"
              type="email"
              disabled
              readonly
              :class="
                isDark
                  ? 'bg-[#1A1825]/60 border-[#36324A] text-slate-500'
                  : 'bg-[#EAEFD3]/70 border-[#B3C0A4] text-[#505168]'
              "
              class="w-full px-3 py-2 text-xs rounded-lg border outline-none cursor-not-allowed"
            />
          </div>

          <div>
            <label
              :class="isDark ? 'text-slate-300' : 'text-[#505168]'"
              class="block text-xs font-medium mb-1.5"
            >
              Nama Lengkap
            </label>
            <input
              v-model="form.name"
              type="text"
              required
              :class="
                isDark
                  ? 'bg-[#1A1825] border-[#36324A] text-white focus:border-[#DCC48E]'
                  : 'bg-white border-[#B3C0A4] text-[#27233A] focus:border-[#27233A]'
              "
              class="w-full px-3 py-2 text-xs rounded-lg border outline-none transition-colors"
            />
          </div>

          <div>
            <label
              :class="isDark ? 'text-slate-300' : 'text-[#505168]'"
              class="block text-xs font-medium mb-1.5"
            >
              Username
            </label>
            <input
              v-model="form.username"
              type="text"
              required
              :class="
                isDark
                  ? 'bg-[#1A1825] border-[#36324A] text-white focus:border-[#DCC48E]'
                  : 'bg-white border-[#B3C0A4] text-[#27233A] focus:border-[#27233A]'
              "
              class="w-full px-3 py-2 text-xs rounded-lg border outline-none transition-colors"
            />
          </div>

          <div>
            <label
              :class="isDark ? 'text-slate-300' : 'text-[#505168]'"
              class="block text-xs font-medium mb-1.5"
            >
              No. Telepon
            </label>
            <input
              v-model="form.no_telp"
              @input="sanitizePhoneInput"
              type="tel"
              inputmode="numeric"
              pattern="[0-9+]*"
              minlength="9"
              maxlength="15"
              :class="
                isDark
                  ? 'bg-[#1A1825] border-[#36324A] text-white focus:border-[#DCC48E]'
                  : 'bg-white border-[#B3C0A4] text-[#27233A] focus:border-[#27233A]'
              "
              class="w-full px-3 py-2 text-xs rounded-lg border outline-none transition-colors"
              placeholder="Contoh: 081234567890"
            />
            <p
              :class="isDark ? 'text-slate-500' : 'text-[#505168]/70'"
              class="text-[10px] mt-1"
            >
              Hanya angka, 9–15 digit (boleh diawali "+").
            </p>
          </div>

          <div>
            <label
              :class="isDark ? 'text-slate-300' : 'text-[#505168]'"
              class="block text-xs font-medium mb-1.5"
            >
              Role Peran
            </label>
            <select
              v-model="form.role"
              :class="
                isDark
                  ? 'bg-[#1A1825] border-[#36324A] text-white focus:border-[#DCC48E]'
                  : 'bg-white border-[#B3C0A4] text-[#27233A] focus:border-[#27233A]'
              "
              class="w-full px-3 py-2 text-xs rounded-lg border outline-none transition-colors cursor-pointer"
            >
              <option value="user">User / Pembeli</option>
              <option value="admin">Administrator</option>
            </select>
          </div>

          <div
            :class="isDark ? 'border-[#36324A]' : 'border-[#B3C0A4]/40'"
            class="flex justify-end gap-2 pt-3 border-t"
          >
            <button
              type="button"
              @click="showModal = false"
              :class="
                isDark
                  ? 'text-slate-400 hover:text-white hover:bg-[#1A1825]'
                  : 'text-[#505168] hover:text-[#27233A] hover:bg-[#EAEFD3]'
              "
              class="px-3.5 py-1.5 text-xs font-medium rounded-lg transition-colors"
            >
              Batal
            </button>
            <button
              type="submit"
              :disabled="submitting"
              :class="
                isDark
                  ? 'bg-[#DCC48E] text-[#1A1825] hover:bg-[#d0b57a]'
                  : 'bg-[#27233A] text-[#EAEFD3] hover:bg-[#36304d]'
              "
              class="px-4 py-1.5 text-xs font-semibold rounded-lg transition-colors disabled:opacity-50"
            >
              {{ submitting ? "Menyimpan..." : "Simpan Perubahan" }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  Pencil as LucidePencil,
  Trash2 as LucideTrash2,
} from "lucide-vue-next";

definePageMeta({
  middleware: "admin",
});

const api = useApi();
const { isDark } = useTheme();
const users = ref<any[]>([]);
const loading = ref(true);
const showModal = ref(false);
const editUserObj = ref<any>(null);
const submitting = ref(false);
const errorMessage = ref("");

// NOTE: email sengaja TIDAK dikirim ke backend saat update.
// Field-nya hanya ditampilkan (read-only) sebagai referensi di modal.
const form = reactive({
  email: "",
  name: "",
  username: "",
  no_telp: "",
  role: "user",
});

const fetchUsers = async () => {
  loading.value = true;
  try {
    const res = await api.get("/api/admin/users");
    users.value = res.data || [];
  } catch (e) {
    console.error(e);
  } finally {
    loading.value = false;
  }
};

// Hanya izinkan angka dan tanda "+" di awal
const sanitizePhoneInput = () => {
  let val = form.no_telp.replace(/[^0-9+]/g, "");
  const plusCount = (val.match(/\+/g) || []).length;
  if (plusCount > 0) {
    val = "+" + val.replace(/\+/g, "");
  }
  form.no_telp = val;
};

const openEditModal = (u: any) => {
  editUserObj.value = u;
  form.email = u.email;
  form.name = u.name;
  form.username = u.username;
  form.no_telp = u.no_telp || "";
  form.role = u.role;
  errorMessage.value = "";
  showModal.value = true;
};

const saveUser = async () => {
  if (!editUserObj.value) return;
  submitting.value = true;
  errorMessage.value = "";

  try {
    const toast = useToast();
    // Hanya kirim field yang memang boleh diubah (tanpa email & password)
    await api.put(`/api/admin/users/${editUserObj.value.id}`, {
      name: form.name,
      username: form.username,
      no_telp: form.no_telp,
      role: form.role,
    });
    toast.success("Data pengguna berhasil diubah!");
    showModal.value = false;
    await fetchUsers();
  } catch (err: any) {
    errorMessage.value =
      err.data?.message ||
      err.data?.errors?.username?.[0] ||
      err.data?.errors?.no_telp?.[0] ||
      "Gagal mengubah data pengguna";
  } finally {
    submitting.value = false;
  }
};

const deleteUser = async (u: any) => {
  if (!confirm(`Yakin menghapus pengguna '${u.name}'?`)) return;
  try {
    const toast = useToast();
    await api.delete(`/api/admin/users/${u.id}`);
    toast.success("Pengguna berhasil dihapus!");
    await fetchUsers();
  } catch (err: any) {
    const toast = useToast();
    toast.error(err.data?.message || "Gagal menghapus pengguna");
  }
};

onMounted(fetchUsers);
</script>
