<template>
  <div class="page max-w-6xl mx-auto px-4 py-16">
    <div class="text-center mb-12">
      <span class="eyebrow px-3 py-1 text-xs font-semibold"
        >BLOG &amp; ARTIKEL</span
      >
      <h1 class="page-title text-2xl sm:text-3xl mt-4 mb-3">
        Berita &amp; Informasi Literatur
      </h1>
      <p class="muted text-sm max-w-lg mx-auto">
        Temukan rekomendasi buku, ulasan, dan tips membaca menarik di sini.
      </p>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
      <article
        v-for="post in posts"
        :key="post.id"
        class="card overflow-hidden flex flex-col justify-between"
      >
        <div>
          <div class="cover h-40 flex items-center justify-center">
            <component
              :is="post.icon"
              class="w-8 h-8 cover-icon"
              :stroke-width="1.5"
            />
          </div>
          <div class="p-6">
            <span
              class="tag inline-block text-[11px] font-medium px-2 py-0.5 mb-3"
            >
              {{ post.category }}
            </span>
            <h3 class="post-title font-semibold text-base mb-2 leading-snug">
              {{ post.title }}
            </h3>
            <p class="muted text-xs mb-3">
              {{ post.date }} &middot; Oleh {{ post.author }}
            </p>
            <p class="desc-text text-xs line-clamp-3 leading-relaxed">
              {{ post.excerpt }}
            </p>
          </div>
        </div>
        <div class="p-6 pt-0">
          <button
            type="button"
            class="btn-outline text-xs font-semibold px-3 py-2 inline-flex items-center gap-1.5"
          >
            Baca Selengkapnya
            <LucideArrowRight class="w-3.5 h-3.5" />
          </button>
        </div>
      </article>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  BookOpen as LucideBookOpen,
  Laptop as LucideLaptop,
  Lightbulb as LucideLightbulb,
  ArrowRight as LucideArrowRight,
} from "lucide-vue-next";

const posts = ref([
  {
    id: 1,
    icon: LucideBookOpen,
    category: "Rekomendasi",
    title: "5 Buku Fiksi Indonesia Terbaik yang Wajib Dibaca Tahun Ini",
    date: "10 September 2026",
    author: "Tim Redaksi",
    excerpt:
      "Daftar novel fiksi penuh inspirasi dan nilai budaya tinggi yang siap menemani waktu luang Anda.",
  },
  {
    id: 2,
    icon: LucideLaptop,
    category: "Teknologi",
    title: "Belajar Fullstack Web Development dengan Laravel 12 & Nuxt 3",
    date: "08 September 2026",
    author: "Admin Tech",
    excerpt:
      "Panduan langkah demi langkah merancang arsitektur API modern dan Single Page Application.",
  },
  {
    id: 3,
    icon: LucideLightbulb,
    category: "Tips Membaca",
    title: "Cara Membangun Kebiasaan Membaca Buku 15 Menit Setiap Hari",
    date: "05 September 2026",
    author: "Penulis Tamu",
    excerpt:
      "Tips sederhana namun efektif untuk konsisten membaca buku di tengah kesibukan harian.",
  },
]);
</script>

<style scoped>
/* Semua warna mengikuti token di main.css, jadi mode terang dan gelap sinkron */
.page {
  color: var(--text);
}

.page-title {
  font-family: var(--font-display);
  font-weight: 600;
  letter-spacing: -0.01em;
  color: var(--text);
}

.muted {
  color: var(--muted);
}

.eyebrow {
  display: inline-block;
  background-color: var(--inset);
  border: 1px solid var(--border-strong);
  border-radius: 9999px;
  color: var(--text-2);
  letter-spacing: 0.04em;
}

.card {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  transition: border-color 0.15s ease;
}

.card:hover {
  border-color: var(--border-strong);
}

.cover {
  background-color: var(--inset);
  border-bottom: 1px solid var(--border);
}

.cover-icon {
  color: var(--muted);
}

.tag {
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-sm);
  color: var(--text-2);
}

.post-title {
  color: var(--text);
}

.desc-text {
  color: var(--text-2);
}

.btn-outline {
  color: var(--text);
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
  background: transparent;
  transition:
    background-color 0.15s ease,
    border-color 0.15s ease;
}

.btn-outline:hover {
  background-color: var(--inset);
  border-color: var(--text-2);
}
</style>
