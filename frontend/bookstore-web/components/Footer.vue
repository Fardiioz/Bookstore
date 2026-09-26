<template>
  <footer class="site-footer mt-20 py-12">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="grid grid-cols-1 md:grid-cols-[1.5fr_1fr_1fr_1fr] gap-10">
        <div>
          <span class="footer-brand block mb-3">BookStore</span>
          <p class="footer-text text-sm leading-relaxed max-w-xs">
            Platform toko buku online terlengkap dengan koleksi buku berkualitas
            untuk mencerdaskan kehidupan bangsa.
          </p>
        </div>

        <nav aria-label="Navigasi utama">
          <h4 class="footer-heading mb-4">Navigasi Utama</h4>
          <ul class="space-y-2.5 text-sm">
            <li><NuxtLink to="/" class="footer-link">Home</NuxtLink></li>
            <li>
              <NuxtLink to="/user/katalog" class="footer-link"
                >Katalog Buku</NuxtLink
              >
            </li>
            <li>
              <NuxtLink to="/our-story" class="footer-link">Our Story</NuxtLink>
            </li>
            <li>
              <NuxtLink to="/blog" class="footer-link">Blog & Berita</NuxtLink>
            </li>
          </ul>
        </nav>

        <nav aria-label="Bantuan dan kontak">
          <h4 class="footer-heading mb-4">Bantuan & Kontak</h4>
          <ul class="space-y-2.5 text-sm">
            <li>
              <NuxtLink to="/contact" class="footer-link"
                >Hubungi Kami</NuxtLink
              >
            </li>
            <li>
              <NuxtLink to="/forget-password" class="footer-link"
                >Lupa Password</NuxtLink
              >
            </li>
            <li>
              <NuxtLink to="/user/chat" class="footer-link"
                >Live Chat Support</NuxtLink
              >
            </li>
          </ul>
        </nav>

        <div>
          <h4 class="footer-heading mb-4">Jam Operasional</h4>
          <p class="footer-text text-sm">Senin - Sabtu</p>
          <p class="footer-text text-sm">08.00 - 20.00 WIB</p>
        </div>
      </div>

      <div class="footer-bottom mt-10 pt-6 text-xs text-center md:text-left">
        &copy; {{ new Date().getFullYear() }} BookStore Project. All rights
        reserved.
      </div>
    </div>
  </footer>
</template>

<style scoped>
/* Warna mengikuti token di main.css, jadi mode terang dan gelap otomatis sinkron */
.site-footer {
  background-color: var(--surface);
  border-top: 1px solid var(--border);
  color: var(--text);
}

.footer-brand {
  font-family: var(--font-display);
  font-size: 1.25rem;
  font-weight: 600;
  letter-spacing: -0.01em;
  color: var(--text);
}

.footer-heading {
  font-size: 0.875rem;
  font-weight: 600;
  color: var(--text);
}

.footer-text {
  color: var(--muted);
}

.footer-link {
  color: var(--muted);
  text-underline-offset: 3px;
  transition: color 0.15s ease;
}

.footer-link:hover {
  color: var(--text);
  text-decoration: underline;
}

.footer-bottom {
  border-top: 1px solid var(--border);
  color: var(--muted);
}
</style>
