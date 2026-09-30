# Dokumentasi Teknologi dan Rasionalisasi Pemilihan Stack (Tech Stack & Rationale) — Sae Link

Dokumen ini menyajikan rincian menyeluruh mengenai seluruh teknologi, kerangka kerja (*framework*), pustaka (*library*), basis data, serta perangkat pendukung yang digunakan dalam pengembangan platform **Sae Link**, disertai alasan objektif dan rasionalisasi teknis di balik setiap keputusan pemilihan teknologi.

---

## 1. Arsitektur Sistem: Decoupled / Headless SPA Architecture

Sae Link menerapkan arsitektur *Decoupled* (terpisah) modern dengan membagi sistem menjadi dua aplikasi utama yang berkomunikasi melalui antarmuka RESTful API:
1. **Frontend**: Berjalan sebagai **Single Page Application (SPA)** berkecepatan tinggi yang dibangun di atas **Vite 6**, **React 19**, **React Router 7**, dan **TypeScript**. Bertanggung jawab penuh melayani antarmuka publik, landing page, sistem autentikasi, dashboard pengguna, dan editor interaktif (Live Smartphone Builder).
2. **Backend**: Berjalan di atas runtime **PHP 8.3+** menggunakan **Laravel 13.x**. Bertanggung jawab melayani logika bisnis inti, manajemen basis data relasional, pemrosesan antrean asinkron (*queues*), sistem autentikasi sesi & token, keamanan jaringan (SSRF & Security Headers), integrasi pihak ketiga, serta panel kontrol administrasi (*Superadmin Panel*) berbasis server-side Blade.

```
┌──────────────────────────────────────────────────────────┐
│                   KLIEN / PENGUNJUNG                     │
│  (Browser Desktop, Tablet, Smartphone Android/iOS)      │
└──────────────┬────────────────────────────▲──────────────┘
               │                            │
      HTTP/HTTPS Request             JSON / Web Assets
               │                            │
┌──────────────▼────────────────────────────┴──────────────┐
│                  FRONTEND (SPA)                          │
│  • Vite 6 + React 19 + React Router 7 + TypeScript       │
│  • Tailwind CSS v4 + Framer Motion 13 + Zustand 5        │
│  • React Helmet Async (Dynamic Meta & SEO)               │
│  • NextCompat Layer (Kompatibilitas Modular)             │
└──────────────┬────────────────────────────▲──────────────┘
               │                            │
         RESTful API Calls            JSON Response
      (Bearer / Sanctum Cookie)     (Status, Data, Token)
               │                            │
┌──────────────▼────────────────────────────┴──────────────┐
│                  BACKEND (REST API)                      │
│  • Laravel 13.x Framework (PHP 8.3+)                     │
│  • Laravel Sanctum 4.x + Socialite 5.x                   │
│  • Intervention Image 4.3            │
│  • Google Sheets Service + SSRF Protection               │
│  • Superadmin Panel (Blade Views & Auth)                 │
└──────────────┬────────────────────────────▲──────────────┘
               │                            │
          Read / Write                  Flysystem
               │                            │
┌──────────────▼─────────────┐   ┌──────────▼──────────────┐
│  BASIS DATA (MySQL/MariaDB)│   │ PENYIMPANAN MEDIA       │
│  • 23+ Model Relasional    │   │ • Storage Public Disk   │
│  • ACID Compliance         │   │ • WebP Compressed Media │
│  • Native JSON Blocks      │   │ • Fallback Direct Route │
└────────────────────────────┘   └─────────────────────────┘
```

### Alasan & Keuntungan Pemilihan Arsitektur Decoupled SPA:
- **Kecepatan Interaksi Sisi Klien Maksimal (Zero Latency Experience)**:
  Pada modul inti seperti *Live Smartphone Builder*, setiap manipulasi warna, font, susunan blok, dan teks direfleksikan secara instan ke mockup visual di layar tanpa jeda putaran *round-trip* ke server. Arsitektur SPA memastikan transisi antar-menu di dashboard berjalan mulus seketika tanpa *page reload*.
- **Pemisahan Tanggung Jawab (*Separation of Concerns*)**:
  Pengembangan antarmuka pengguna dapat berfokus penuh pada estetika, responsivitas mikro-interaksi, dan kenyamanan pengguna (*user experience*), sementara backend berfokus pada ketahanan transaksi finansial, validasi keamanan, manajemen integritas data, dan automasi antrean.
- **Efisiensi Beban Komputasi Server**:
  Dengan memindahkan *rendering pipeline* antarmuka sepenuhnya ke peramban (*client-side*), server backend hanya bertugas memproses data terstruktur berbasis JSON murni. Hal ini memangkas konsumsi CPU dan RAM server secara drastis dibandingkan arsitektur yang merender HTML lengkap di setiap permintaan pengunjung.
- **Fleksibilitas Distribusi & Deployment Ganda (*Unified or Standalone*)**:
  - **Mode Decoupled**: Frontend dan backend dapat di-deploy secara independen pada domain atau peladen yang berbeda (misal: frontend pada CDN / static host, backend pada VPS / cloud compute).
  - **Mode Unified SPA**: Hasil kompilasi statis frontend (`dist/index.html` dan aset terkait) dapat disajikan langsung oleh rute *catch-all* Laravel (`Route::get('/{any}')`), memungkinkan implementasi *single-server* yang sangat praktis dan hemat biaya tanpa memerlukan infrastruktur Node.js khusus di tahap produksi.

---

## 2. Frontend Stack & Alasan Pemilihan

Frontend Sae Link dibangun menggunakan ekosistem JavaScript/TypeScript mutakhir yang mengutamakan kecepatan muat instan, arsitektur modular, pengetikan statis tipe-aman, dan rendering visual yang memukau.

### Tabel Ringkasan Frontend

| Teknologi / Pustaka | Versi | Kategori | Peran Utama dalam Sistem |
| :--- | :--- | :--- | :--- |
| **Vite** | 6.0+ | Build Tool & Dev Server | Kompilasi kilat, Instant Hot Module Replacement (HMR), optimasi bundler Rollup |
| **React & React DOM** | 19.2+ | UI Library | Deklarasi komponen reaktif, concurrent rendering, state management lokal |
| **React Router DOM** | 7.1+ | Routing Client-Side | Sistem perutean SPA deklaratif, *nested layouts*, dynamic slug routing |
| **TypeScript** | 5.7+ | Bahasa Pemrograman | *Type-safety* statis menyeluruh, pencegahan galat runtime, kontrak API |
| **Tailwind CSS** | 4.0+ | Styling Framework | Engine Lightning CSS berbasis Rust, styling utility-first, arsitektur `@theme` |
| **@tailwindcss/vite** | 4.0+ | Vite Compiler Plugin | Integrasi langsung Tailwind CSS v4 ke alur kerja Vite tanpa dependensi PostCSS |
| **Zustand** | 5.0+ | State Management | Global state terisolasi untuk Live Builder, sesi autentikasi, dan modal sistem |
| **Framer Motion** | 13.2+ | Animasi & Mikro-interaksi | Animasi kurva Apple easing, transisi unroll dropdown, feedback interaksi |
| **React Helmet Async** | 2.0+ | SEO & Head Manager | Pengelolaan tag `<title>`, deskripsi, Open Graph, dan Twitter card dinamis |
| **React Hook Form** | 7.87+ | Form Handling | Manajemen formulir performan tinggi berbasis *uncontrolled components* |
| **Zod** | 3.25+ | Skema Validasi | Validasi skema tipe-aman untuk masukan formulir dan struktur respons data |
| **Axios** | 1.20+ | HTTP Client | Komunikasi RESTful API, interceptor token, auto-deteksi IP host LAN/Wi-Fi |
| **Lucide React & React Icons** | 1.43+ / 5.5+ | Aset Ikonografi | Set ikon SVG berbasis vektor yang ringan, modern, dan *tree-shakeable* |
| **Sonner** | 2.0+ | Notifikasi Toast | Feedback pesan status pengguna berbasis tumpukan (*stacked notifications*) |
| **QRCode** | 1.5+ | Generator Barcode | Konversi instan URL shortlink dan profil bio-link menjadi QR Code di browser |
| **ExcelJS** | 4.4+ | Utilitas Spreadsheet | Pembuatan dan penataan format berkas Excel (.xlsx) langsung di sisi klien |
| **clsx & tailwind-merge** | 2.1+ / 3.6+ | Class Utilities | Penggabungan kelas CSS dinamis dan resolusi otomatis konflik kelas Tailwind |
| **NextCompat Layer** | Kustom Internal | Kompatibilitas Framework | Abstraksi modul (`NextLink`, `NextImage`, `useNextRouter`) di atas ekosistem Vite |

---

### Rincian & Rasionalisasi Frontend

#### Vite 6.0+
- **Rasionalisasi**:
  - Menghilangkan *cold start* lambat pada lingkungan pengembangan dengan menyajikan kode sumber melalui native ES Modules (ESM), sementara modul eksternal diproses awal menggunakan *esbuild* berbasis Go yang luar biasa cepat.
  - Fitur Hot Module Replacement (HMR) berjalan hampir seketika (*instant updates*) tanpa memuat ulang seluruh halaman peramban, menghemat waktu iterasi desain secara drastis saat mengutak-atik Live Builder.
  - Membawa fitur *integrated dev server proxy* (`/api` dan `/storage`) yang mengeliminasi masalah Cross-Origin Resource Sharing (CORS) selama proses pengembangan lokal.

#### React 19.2+ & React DOM
- **Rasionalisasi**:
  - Menyediakan fondasi rendering terbaru dengan optimasi internal mesin React (Concurrent Rendering) yang menjaga antarmuka tetap responsif terhadap input pengguna bahkan saat melakukan kalkulasi tema yang berat.
  - Memberikan kompatibilitas jangka panjang dengan ekosistem library UI modern yang memanfaatkan standar arsitektur React terkini.

#### React Router 7.1+ (`react-router-dom`)
- **Rasionalisasi**:
  - Standar industri teruji untuk routing Single Page Application di ekosistem React.
  - Mendukung struktur *nested routes* yang memisahkan layout autentikasi (`AuthLayout`) dan dashboard pengguna (`DashboardLayout`) secara rapi dengan lazy-loading berbasis `React.Suspense` dan komponen `BrandLoadingScreen`.
  - Mengelola rute slug dinamis tingkat lanjut seperti `/:username` (bio-link publik), `/:username/:microsite` (halaman microsite bercabang), serta `/:username/catalog/:blockId` (detail produk katalog WhatsApp).

#### React Helmet Async 2.0+
- **Rasionalisasi**:
  - Mengelola elemen `<head>` HTML (seperti `<title>`, `<meta name="description">`, OpenGraph image/title, dan Twitter cards) secara dinamis per halaman di lingkungan Single Page Application.
  - Bersifat *asynchronous* dan *thread-safe*, mencegah kebocoran memori (*memory leak*) dan konflik metadata saat pengguna berpindah halaman secara cepat.

#### TypeScript 5.7+
- **Rasionalisasi**:
  - Mencegah galat umum JavaScript pada waktu eksekusi (*runtime error*) melalui pemeriksaan tipe statis ketat saat proses kompilasi.
  - Memastikan konsistensi struktur data yang dikirim dan diterima dari RESTful API backend melalui kontrak antarmuka (*interface definitions*) yang ketat.
  - Memfasilitasi refaktorisasi kode berskala besar dengan bantuan fitur autocompletion dan navigasi kode terintegrasi di editor IDE.

#### Tailwind CSS v4 & `@tailwindcss/vite`
- **Rasionalisasi**:
  - Menggunakan engine Lightning CSS berbasis Rust yang menawarkan proses kompilasi instan dan efisiensi berkas CSS akhir yang sangat kecil melalui pemangkasan kelas tidak terpakai (*automatic purge*).
  - Mengadopsi arsitektur konfigurasi berbasis CSS murni (`@theme`), menghilangkan ketergantungan berkas `tailwind.config.js` atau pipeline PostCSS yang rumit.
  - Pendekatan *utility-first* mempercepat implementasi komponen desain tanpa menciptakan duplikasi berkas *stylesheet* kustom yang sulit dirawat.

#### Zustand 5.0+
- **Rasionalisasi**:
  - Mengelola state global aplikasi (seperti `useAuthStore` untuk status autentikasi, `useBuilderStore` untuk pengaturan live builder, dan `useInAppNotifications`) dengan arsitektur berbasis hooks yang sangat ringkas dan bebas *boilerplate*.
  - Menghilangkan kebutuhan pembungkus *Context Provider* bertingkat di level root, menghindari re-render yang tidak diinginkan pada komponen yang tidak berlangganan ke state tersebut.
  - Sangat ideal untuk modul Live Smartphone Builder, di mana perubahan warna, susunan blok, atau teks judul harus dipantulkan seketika ke mockup smartphone tanpa penurunan *frame rate*.

#### Framer Motion 13.2+
- **Rasionalisasi**:
  - Pustaka animasi deklaratif berstandar industri untuk React.
  - Digunakan untuk mengimplementasikan transisi kurva eksponensial (Apple UI easing `cubic-bezier(0.16, 1, 0.3, 1)`) pada komponen dropdown dan menu lipat (*unroll animation*), memberikan kesan antarmuka yang sangat elegan dan premium.
  - Menyediakan animasi layout cair saat pengguna melakukan *drag-and-drop* untuk menyusun ulang urutan blok tautan di builder.

#### React Hook Form & Zod
- **Rasionalisasi**:
  - React Hook Form meminimalisir re-render saat pengguna mengetik data formulir dengan memanfaatkan *uncontrolled components*, menjaga performa tetap optimal pada halaman formulir panjang.
  - Zod melengkapi penanganan formulir dengan validasi berbasis skema deklaratif yang menghasilkan pesan kesalahan terstruktur dan secara otomatis mengekstrak tipe data TypeScript yang valid.

#### Axios 1.20+ & Fitur Smart IP Auto-Detection
- **Rasionalisasi**:
  - Memiliki interceptor bawaan untuk menyisipkan Bearer token autentikasi serta mendeteksi respons kedaluwarsa sesi (kode status 401) guna mengarahkan pengguna kembali ke halaman login secara otomatis.
  - **Fitur Cerdas IP Auto-Detection**: Klien Axios dilengkapi logika pendeteksi host dinamis ([`src/lib/axios.ts`](src/lib/axios.ts)). Jika frontend diakses dari browser smartphone melalui jaringan Wi-Fi lokal (misal: `http://192.168.1.91:4000`), Axios secara otomatis menyesuaikan target API ke alamat IP lokal backend terkait (`http://192.168.1.91:8000/api`) tanpa perlu mengubah variabel lingkungan secara manual.

#### NextCompat Layer (`@/components/compat/*`)
- **Rasionalisasi**:
  - Menyediakan shim kompatibilitas internal (`NextLink`, `NextImage`, `useNextRouter`) yang mengemulasi API navigasi dan optimasi gambar Next.js di atas React Router 7 dan tag `<img>` standar modern.
  - Memungkinkan penggunaan kembali ribuan baris kode komponen antarmuka yang ada secara aman dan cepat tanpa terikat pada ketergantungan runtime server Next.js.

#### Dynamic Branding Synchronization (`BrandingProvider`)
- **Rasionalisasi**:
  - Mengambil konfigurasi identitas brand platform (logo brand, favicon, judul aplikasi, dan watermark) secara dinamis dari endpoint backend `/api/public/branding`.
  - Menerapkan pembaruan favicon dan judul dokumen secara langsung di sisi browser dengan strategi caching lokal dan *safe fallback* bawaan.

---

## 3. Backend Stack & Alasan Pemilihan

Backend Sae Link dirancang untuk keandalan tinggi, pemrosesan transaksi yang aman, pengelolaan basis data relasional yang ketat, dan automasi tugas latar belakang.

### Tabel Ringkasan Backend

| Teknologi / Pustaka | Versi | Kategori | Peran Utama dalam Sistem |
| :--- | :--- | :--- | :--- |
| **PHP** | 8.3+ | Bahasa Pemrograman | Runtime server performa tinggi, JIT compiler, dan sistem pengetikan ketat |
| **Laravel Framework** | 13.x | Framework Backend | Arsitektur REST API terpusat (`bootstrap/app.php`), Routing, Middleware, ORM, Queues |
| **Laravel Sanctum** | 4.0+ | Sistem Autentikasi | Autentikasi SPA berbasis cookie stateful dan API token bearer |
| **Laravel Socialite** | 5.31+ | Autentikasi OAuth | Integrasi login pihak ketiga menggunakan Google OAuth 2.0 |
| **Eloquent ORM** | Bawaan Laravel | Object-Relational Mapper | Pemodelan data relasional, query builder, validasi model, tipe kolom JSON |
| **Intervention Image** | 4.3+ | Pengolahan Citra | Kompresi citra otomatis, cropping, dan konversi WebP modern |
| **PragmaRX Google2FA** | 9.1+ | Keamanan Akun | Autentikasi dua faktor berbasis Time-based One-Time Password (TOTP) |
| **Blade View Engine** | Bawaan Laravel | Templating Server-Side | Antarmuka panel administrasi internal Superadmin yang terisolasi |
| **Laravel Queue & Job** | Bawaan Laravel | Background Worker | Eksekusi asinkron pengiriman email OTP, log audit, dan webhook pihak ketiga |
| **Laravel Observers** | Bawaan Laravel | Model Lifecycle Hooks | Automasi pembersihan dan invalidasi cache (`CacheService`) saat data berubah |
| **Security Headers Middleware** | Kustom Laravel | Keamanan Jaringan | Penerapan proteksi header HTTP (CSP, HSTS, X-Frame-Options, Referrer-Policy) |
| **Reverse Proxy & Trust Proxies** | Bawaan Laravel | Konfigurasi Jaringan | Resolusi aman header proksi (`trustProxies(at: '*')`, Cloudflare `CF-Connecting-IP`) |
| **Direct Storage & Setup Helper** | Kustom Laravel | Kemudahan Deployment | Fallback serving `/storage/{path}` dan endpoint setup aman tanpa akses SSH |

---

### Rincian & Rasionalisasi Backend

#### PHP 8.3+
- **Rasionalisasi**:
  - Menawarkan peningkatan kecepatan eksekusi dan efisiensi memori yang signifikan berkat evolusi mesin Zend Engine dan compiler Just-In-Time (JIT).
  - Membawa fitur sintaksis modern seperti *typed class constants*, *readonly classes*, *match expressions*, dan *property promotion* yang membuat basis kode lebih bersih, terstruktur, dan minim potensi kesalahan logika.

#### Laravel 13.x Framework
- **Rasionalisasi**:
  - Mengadopsi arsitektur konfigurasi ramping modern terpusat pada [`bootstrap/app.php`](bootstrap/app.php), menyederhanakan konfigurasi routing, middleware, dan penanganan exception tanpa file konfigurasi yang berserakan.
  - Ekosistem enterprise lengkap: database migrations, seeders, model factory, task scheduling, queue workers, dan rate limiting terintegrasi.
  - Memiliki fitur rate limiting granular terkonfigurasi di route API:
    - `throttle:auth`: Mencegah serangan brute-force pada endpoint login, register, dan 2FA.
    - `throttle:analytics`: Membatasi spam pencatatan klik dan views pada shortlink dan profil publik.
    - `throttle:leads`: Melindungi formulir prospek dari injeksi bot spam.
    - `throttle:uploads`: Membatasi frekuensi unggahan media agar tidak membebani kapasitas disk peladen.

#### Laravel Sanctum 4.x
- **Rasionalisasi**:
  - Menyediakan mekanisme autentikasi ganda yang ideal: session berbasis cookie HTTP-Only yang aman untuk Single Page Application dan token bearer untuk kebutuhan API.
  - Melindungi aplikasi dari serangan Cross-Site Request Forgery (CSRF) tanpa kompleksitas berlebih seperti pada arsitektur OAuth2 penuh.

#### Laravel Socialite 5.31+
- **Rasionalisasi**:
  - Mengabstraksi alur integrasi OAuth 2.0 yang rumit dengan penyedia layanan identitas eksternal (Google OAuth) menjadi antarmuka yang seragam dan mudah diuji, baik melalui flow redirect maupun code-exchange API.

#### Intervention Image 4.3+
- **Rasionalisasi**:
  - Halaman bio-link publik sangat bergantung pada kecepatan muat aset visual (avatar profil, banner, dan foto katalog produk).
  - Pustaka ini secara otomatis mengompres, memotong (*crop*), dan mengonversi berkas citra yang diunggah pengguna ke format **WebP** modern sebelum disimpan ke media penyimpanan permanen, menghemat ruang disk dan bandwidth transfer data hingga 70%.

#### Midtrans PHP SDK 2.6+
- **Rasionalisasi**:
  - Gateway pembayaran berlisensi Bank Indonesia yang mendukung seluruh kanal pembayaran populer di Indonesia (QRIS instan, GoPay, ShopeePay, Virtual Account BCA/Mandiri/BNI/BRI, dan Kartu Kredit).
  - Memudahkan verifikasi hash signature kriptografis pada endpoint webhook (`/api/webhooks/midtrans`) untuk memastikan keaslian notifikasi status pembayaran dari peladen Midtrans.

#### PragmaRX Google2FA 9.1+
- **Rasionalisasi**:
  - Mengimplementasikan spesifikasi RFC 6238 standar industri untuk Two-Factor Authentication (2FA).
  - Kompatibel dengan aplikasi otentikator populer seperti Google Authenticator, Microsoft Authenticator, dan Authy, lengkap dengan sistem pembuatan *recovery codes* darurat.

#### Blade Templating Engine (Panel Superadmin)
- **Rasionalisasi**:
  - Digunakan khusus untuk antarmuka panel administrasi internal Superadmin (`/admin/*`).
  - Menjaga modul administrasi sensitif tetap terisolasi di sisi server tanpa mengekspos endpoint API publik tambahan, meningkatkan lapisan keamanan sistem secara signifikan.
  - Dilengkapi fitur operasional lengkap:
    - **Dashboard Finansial & Kas**: Pemantauan omzet bruto, pencatatan mutasi penarikan dana kas keluar (*withdrawals*), dan rekonsiliasi saldo bersih.
    - **Manajemen Pengguna & Hak Akses**: Promosi akun admin, suspensi akun, dan pemberian/pencabutan kuota paket langganan secara manual.
    - **Moderasi Konten & Halaman**: Deteksi profil mencurigakan, fitur penonaktifan sementara (*suspend page*), dan penandaan aman (*mark safe*).
    - **Manajemen Banner & Theme Events**: Pemasangan banner iklan internal serta aktivasi tema musiman (Ramadhan, tahun baru, event promo).
    - **Pusat Notifikasi & Broadcast**: Pengiriman notifikasi sistem dan siaran pesan global kepada seluruh pengguna aktif.
    - **Monitor Antrean & Kesehatan Server**: Pelacakan status queue failed jobs, retry mekanisme massal, dan pembersihan antrean.
    - **Pengaturan Global & Branding**: Pengubahan logo, favicon, judul dokumen, dan watermark platform secara visual.

#### Storage Fallback & Kemudahan Shared Hosting
- **Rasionalisasi**:
  - Menyediakan rute fallback khusus `Route::get('/storage/{path}')` di [`backend/routes/web.php`](routes/web.php) yang menyajikan berkas unggahan publik secara langsung jika symlink penyimpanan (`storage:link`) tidak dapat dibuat oleh sistem operasi hosting (misalnya pada cPanel/shared hosting gratis seperti InfinityFree).
  - Menyediakan endpoint utilitas `/hosting-setup/{action}` yang diamankan dengan `SETUP_SECRET_KEY` di file `.env` untuk menjalankan instruksi `migrate`, `storage:link`, atau `optimize:clear` langsung melalui browser web tanpa membutuhkan akses terminal SSH.

#### Security Headers Middleware & Reverse Proxy Support
- **Rasionalisasi**:
  - Middleware `SecurityHeaders` menyuntikkan header proteksi keamanan peramban tingkat tinggi pada setiap respons HTTP:
    - `X-Frame-Options: SAMEORIGIN` (mencegah serangan clickjacking)
    - `X-Content-Type-Options: nosniff` (mencegah eksploitasi MIME sniffing)
    - `X-XSS-Protection: 1; mode=block` (mitigasi Cross-Site Scripting)
    - `Referrer-Policy: strict-origin-when-cross-origin`
    - `Permissions-Policy: camera=(), microphone=(), geolocation=()`
    - `Strict-Transport-Security (HSTS)` (penegakan HTTPS saat koneksi aman)
  - Konfigurasi `trustProxies(at: '*')` memastikan peladen mengenali alamat IP pengunjung yang sebenarnya di balik layanan reverse proxy seperti Cloudflare, AWS CloudFront, atau Nginx.

---

## 4. Basis Data & Penyimpanan

### MySQL 8.0+ / MariaDB Relasional
- **Peran**: Penyimpanan utama untuk 23 entitas relasional dan operasional sistem (Pengguna, Halaman, Blok, Transaksi, Penarikan Kas, Log Klik/Views, Prospek, Pengaturan Global, dan Notifikasi).
- **Rasionalisasi**:
  - Menjamin kepatuhan **ACID** (*Atomicity, Consistency, Isolation, Durability*) yang sangat krusial untuk pencatatan transaksi langganan, verifikasi pembayaran, mutasi penarikan saldo kas, dan pencatatan audit administrasi.
  - Mendukung tipe data **JSON native**, memungkinkan penyimpanan properti kustomisasi blok (warna, ikon, border, link, data produk katalog) dengan sangat fleksibel tanpa harus merusak normalisasi skema tabel induk.
  - Pengindeksan komposit optimal pada kolom pencarian frekuensi tinggi (seperti `slug`, `user_id`, `created_at`, `key`, dan status aktif) menjamin kecepatan kueri tetap dalam orde milidetik meskipun volume data membesar.

### Sistem Penyimpanan Berkas (Laravel Storage / Public Disk)
- **Peran**: Penyimpanan berkas statis hasil unggahan pengguna dan administrator (foto profil, banner microsite, bukti transfer manual, bukti penarikan kas, gambar produk, logo brand, dan favicon).
- **Rasionalisasi**:
  - Lapisan abstraksi Flysystem pada Laravel memungkinkan transisi fleksibel dari media penyimpanan lokal (*Local Disk*) ke penyimpanan objek awan (seperti AWS S3, Cloudflare R2, DigitalOcean Spaces, atau MinIO) tanpa merombak logika kode aplikasi saat skala sistem bertumbuh.

---

## 5. Layanan & Integrasi Pihak Ketiga

### 1. Integrasi Google Sheets API (`GoogleSheetsService`)
- **Tujuan**: Sinkronisasi data formulir prospek (*leads*) yang masuk di microsite pengguna langsung ke lembar kerja Google Sheets milik pemilik akun.
- **Rasionalisasi**: Memberikan nilai tambah signifikan bagi pengguna bisnis tanpa mengharuskan mereka memasang integrasi webhook yang rumit secara manual. Mendukung alur OAuth 2-way, perbaikan token kedaluwarsa otomatis, dan sinkronisasi manual.

### 2. TikWM API (`resolve-video`)
- **Tujuan**: Resolusi tautan video TikTok agar dapat diputar langsung di blok video microsite tanpa watermark yang mengganggu tata letak visual.
- **Rasionalisasi**: Meningkatkan retensi pengunjung di halaman profil dengan menyediakan pemutaran video langsung yang mulus dan bebas watermark pihak ketiga.

### 3. Layanan WhatsApp Smart Routing & Checkout
- **Tujuan**: Pembuatan tautan pemesanan produk otomatis langsung ke nomor WhatsApp penjual dengan format pesan terstruktur serta pembagian beban admin CS melalui rotasi nomor.
- **Rasionalisasi**: WhatsApp merupakan kanal transaksi penjualan langsung paling dominan di pasar Indonesia, memangkas friksi alur konversi dari melihat produk hingga melakukan pembelian.

---

## 6. Mekanisme Keamanan dan Ketahanan Sistem

Sae Link menerapkan langkah-langkah proteksi proaktif yang diintegrasikan langsung ke dalam arsitektur kode:

### Proteksi SSRF (*Server-Side Request Forgery*) via `SsrfProtectionService`
- **Rasionalisasi**: Sistem memungkinkan pengguna memasukkan tautan eksternal (seperti target webhook atau tautan video embed). Layanan ini memvalidasi alamat IP target sebelum melakukan panggilan HTTP untuk memastikan sistem backend tidak dapat diperdaya untuk memindai jaringan privat internal (seperti `localhost`, `127.0.0.1`, atau subnet privat `10.x.x.x` / `192.168.x.x`).

### Anti-CSV / Formula Injection pada Modul Ekspor
- **Rasionalisasi**: Saat mengekspor data prospek atau transaksi ke format spreadsheet (`.xlsx` atau `.csv`), karakter awalan berisiko (`=`, `+`, `-`, `@`, tab) disanitasi secara otomatis untuk mencegah eksekusi formula berbahaya ketika file dibuka di Microsoft Excel atau Google Sheets oleh administrator.

### Rate Limiting & Proteksi Brute Force
- **Rasionalisasi**: Penerapan middleware pembatas frekuensi panggilan (*rate limiter*) pada rute publik yang sensitif (seperti `/api/login`, `/api/register`, `/api/public/shortlink/{slug}/click`, dan `/api/blocks/{block_id}/leads`) untuk mencegah eksploitasi Denial of Service (DoS), bot spam formulir, dan manipulasi analitik.

### Model Observers & Strategi Invalidation Cache
- **Rasionalisasi**: Setiap kali data halaman atau blok dimodifikasi oleh pengguna, Laravel Observers secara otomatis mendeteksi kejadian tersebut dan menghapus kunci cache terkait dari `CacheService`. Hal ini menjamin bahwa pengunjung umum selalu mendapatkan data terbaru sekaligus memastikan server tidak terbebani kueri database berulang untuk profil yang sedang viral.

### Deteksi Klien Aman via `AgentService` & Pelacakan Trafik Valid
- **Rasionalisasi**:
  - Parser User-Agent mandiri memetakan jenis peramban, OS, dan gawai pengunjung secara lokal tanpa dependensi API eksternal.
  - Dilengkapi sistem resolusi IP pengunjung asli via header proksi (`CF-Connecting-IP`, `X-Forwarded-For`, `X-Real-IP`) serta regex filter bot/crawler otomatis untuk menjamin bahwa metrik analitik tayangan (*views*) dan klik (*clicks*) benar-benar mencerminkan kunjungan manusia yang valid.

### Jejak Audit Administratif (`AdminAuditLog`)
- **Rasionalisasi**: Seluruh tindakan administratif kritis yang dilakukan superadmin (penarikan saldo, penyesuaian branding, suspensi halaman, pemberian paket kuota, dan penghapusan data) dicatat secara terperinci beserta alamat IP, user-agent, dan rekaman perubahan status data demi akuntabilitas sistem.

---

## 7. Tooling, Lingkungan Pengembangan, dan Standar Mutu Kode

### Composer & npm
- **Rasionalisasi**: Pengelola paket standar de-facto untuk masing-masing ekosistem PHP dan Node.js yang memastikan replikasi dependensi yang deterministik melalui berkas `composer.lock` dan `package-lock.json`.

### Laravel Pint & ESLint
- **Rasionalisasi**:
  - **Laravel Pint**: Menegakkan standar gaya penulisan kode PHP berbasis PSR-12 secara otomatis.
  - **ESLint**: Memvalidasi aturan kode TypeScript dan praktik terbaik React (seperti kelengkapan dependensi hook), menjaga kualitas basis kode tetap seragam dan mudah dibaca oleh seluruh kontributor tim.

### Git Version Control
- **Rasionalisasi**: Melacak seluruh riwayat modifikasi kode sumber, memfasilitasi kerja sama tim, dan menyederhanakan alur implementasi integrasi dan pengiriman berkelanjutan (CI/CD).

---

## 8. Kesimpulan

Kombinasi teknologi pada **Sae Link** dipilih secara terukur berdasarkan prinsip keseimbangan antara:
1. **Kecepatan dan Responsivitas Ekstrem**: Pemanfaatan Vite 6 dan React 19 SPA menjamin waktu muat kilat serta responsivitas zero-latency pada Live Smartphone Builder.
2. **Kenyamanan Pengembang (*Developer Experience*)**: Arsitektur utility Tailwind CSS v4, type-safety TypeScript 5, dan ekosistem modern Laravel 13 mempercepat siklus peluncuran fitur baru.
3. **Efisiensi Infrastruktur**: Offloading rendering ke sisi klien dan kompresi WebP otomatis memangkas kebutuhan komputasi server dan bandwidth secara drastis.
4. **Keamanan Transaksional dan Keandalan Data**: Integritas data terjamin melalui MySQL ACID compliance, proteksi SSRF, Security Headers, serta autentikasi 2FA TOTP berstandar industri.
