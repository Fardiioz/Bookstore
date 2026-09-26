<template>
  <Teleport to="body">
    <div
      class="fixed top-4 right-4 z-[9999] pointer-events-none space-y-2 md:top-6 md:right-6 max-w-xs md:max-w-sm"
      role="status"
      aria-live="polite"
    >
      <TransitionGroup name="toast">
        <div
          v-for="toast in toasts"
          :key="toast.id"
          :class="getToastClass(toast.type)"
          class="toast pointer-events-auto flex items-start gap-3 px-4 py-3 text-sm font-medium"
        >
          <!-- Icon -->
          <component
            :is="getToastIcon(toast.type)"
            class="toast-icon w-4 h-4 flex-shrink-0 mt-0.5"
          />

          <!-- Message -->
          <div class="flex-1 min-w-0">
            <p class="break-words">{{ toast.message }}</p>
          </div>

          <!-- Close Button -->
          <button
            type="button"
            @click="removeToast(toast.id)"
            class="toast-close flex-shrink-0"
            title="Tutup"
            aria-label="Tutup notifikasi"
          >
            <X class="w-4 h-4" />
          </button>
        </div>
      </TransitionGroup>
    </div>
  </Teleport>
</template>

<script setup lang="ts">
import { CheckCircle2, XCircle, Info, AlertTriangle, X } from "lucide-vue-next";
import { useToast } from "~/composables/useToast";

const { toasts, remove } = useToast();

const removeToast = (id: string) => {
  remove(id);
};

const getToastIcon = (type: string) => {
  const icons: Record<string, any> = {
    success: CheckCircle2,
    error: XCircle,
    info: Info,
    warning: AlertTriangle,
  };
  return icons[type] || Info;
};

const getToastClass = (type: string) => {
  const known = ["success", "error", "info", "warning"];
  return known.includes(type) ? `toast--${type}` : "toast--info";
};
</script>

<style scoped>
/* Warna mengikuti token main.css (terang/gelap). Jenis notifikasi dibedakan
   lewat garis aksen di kiri dan warna ikon, bukan lewat latar berwarna. */
.toast {
  --toast-accent: var(--text-2);
  background-color: var(--surface);
  color: var(--text);
  border: 1px solid var(--border);
  border-left: 3px solid var(--toast-accent);
  border-radius: var(--radius-md);
  box-shadow: 0 2px 8px rgb(0 0 0 / 0.08);
}

.toast-icon {
  color: var(--toast-accent);
}

.toast--success {
  --toast-accent: #4d6b3f;
}

.toast--error {
  --toast-accent: #b0394f;
}

.toast--info {
  --toast-accent: var(--text-2);
}

.toast--warning {
  --toast-accent: #8a6d1f;
}

:global(:root.dark) .toast--success,
:global(:root[data-theme="dark"]) .toast--success {
  --toast-accent: #b3c0a4;
}

:global(:root.dark) .toast--error,
:global(:root[data-theme="dark"]) .toast--error {
  --toast-accent: #e68a9a;
}

:global(:root.dark) .toast--info,
:global(:root[data-theme="dark"]) .toast--info {
  --toast-accent: #cbd2ba;
}

:global(:root.dark) .toast--warning,
:global(:root[data-theme="dark"]) .toast--warning {
  --toast-accent: #dcc48e;
}

.toast-close {
  color: var(--muted);
  transition: color 0.15s ease;
}

.toast-close:hover {
  color: var(--text);
}

.toast-enter-active,
.toast-leave-active {
  transition:
    opacity 200ms ease,
    transform 200ms ease;
}

.toast-enter-from {
  opacity: 0;
  transform: translateY(-8px) translateX(16px);
}

.toast-leave-to {
  opacity: 0;
  transform: translateX(16px) translateY(-8px);
}

.toast-move {
  transition: transform 200ms ease;
}
</style>
