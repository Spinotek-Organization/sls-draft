# Tech Stack

**Arsitektur Teknis Spinotek Learning System**

Spinotek Learning System dibangun menggunakan teknologi modern yang telah dikuasai oleh tim, dengan fokus pada produktivitas pengembangan, performa, dan kemampuan untuk berkembang.

## Stack Utama

```mermaid
graph TB
    subgraph CLIENT["🖥️ Frontend"]
        VUE["Vue 3\nComposition API"]
        TW["Tailwind CSS\nWhite Label Theming"]
        AXIOS["Axios\nHTTP Client"]
    end

    subgraph SERVER["⚙️ Backend"]
        API["Laravel REST API\napi.php Routes"]
        SANCTUM["Laravel Sanctum\nAPI Authentication"]
        QUEUE["Laravel Queue\nBackground Jobs"]
        SCHEDULER["Laravel Scheduler\nCron Jobs"]
        STORAGE["Laravel Storage\nFile Management"]
    end

    subgraph DATA["💾 Data Layer"]
        MYSQL["MySQL 8\nPrimary Database"]
        REDIS["Redis\nCache · Queue · Sessions"]
        S3["S3 / MinIO\nFile Storage"]
    end

    CLIENT -->|"REST API"| SERVER --> DATA

    style CLIENT fill:#1a1a2e,stroke:#42b883,color:#ffffff
    style SERVER fill:#0f3460,stroke:#e94560,color:#ffffff
    style DATA fill:#533483,stroke:#e94560,color:#ffffff
```

| Layer | Teknologi | Versi | Peran |
|---|---|---|---|
| **Frontend** | Vue 3 | 3.x (Composition API) | UI interaktif, komponen reusable |
| **Styling** | Tailwind CSS | 3.x / 4.x | Styling utility-first, white-label theming |
| **HTTP Client** | Axios | — | Request ke REST API Laravel |
| **Build Tool** | Vite | — | Bundling frontend (built-in Laravel) |
| **Backend** | Laravel | 11+ | REST API, auth, business logic, multi-tenancy |
| **Auth** | Laravel Sanctum | — | Token-based API authentication |
| **Database** | MySQL | 8.x | Data utama (user, soal, ujian, course, dll) |
| **Cache** | Redis | 7.x | Caching, queue driver, session store |
| **Storage** | S3 / MinIO | — | File materi, tugas, sertifikat PDF |

---

## Arsitektur Frontend

Vue 3 diinstall langsung di dalam project Laravel melalui **Vite** (sudah built-in di Laravel).

Frontend berkomunikasi dengan backend melalui **REST API** menggunakan **Axios**.

```
resources/
├── js/
│   ├── app.js              → Entry point Vue
│   ├── components/         → Komponen reusable
│   ├── composables/        → Composition API logic
│   ├── pages/              → Halaman utama
│   ├── stores/             → Pinia state management
│   └── api/                → API service layer (axios)
├── views/
│   └── app.blade.php       → Single Blade entry point
└── css/
    └── app.css             → Tailwind CSS
```

**Keuntungan pendekatan ini:**

- **Satu project, satu repo** — Vue dan Laravel dalam satu codebase
- **Load data cepat** — Vue fetch data via REST API secara async, bisa parallel request
- **Full kontrol** — Bisa lazy-load komponen, caching response, optimistic UI
- **API reusable** — API yang sama bisa dipakai untuk mobile app nanti
- **Familiar** — Tim sudah terbiasa dengan pola Laravel API + Vue

---

## Arsitektur Multi-Tenancy

```mermaid
graph TB
    subgraph TENANTS["🏢 Institusi"]
        T1["lms.kampus-a.ac.id"]
        T2["learning.sekolah-b.sch.id"]
        T3["training.company.com"]
    end

    subgraph APP["⚙️ Spinotek Learning System"]
        TENANT_ID["Tenant Identification\n(subdomain / domain)"]
        MW["Middleware\nTenant Resolver"]
        SCOPE["Data Scoping\nPer Tenant"]
    end

    subgraph DB["💾 Database Strategy"]
        SHARED["Shared Database\ndengan tenant_id column"]
    end

    TENANTS --> TENANT_ID --> MW --> SCOPE --> DB

    style TENANTS fill:#1a1a2e,stroke:#e94560,color:#ffffff
    style APP fill:#16213e,stroke:#0f3460,color:#ffffff
    style DB fill:#533483,stroke:#e94560,color:#ffffff
```

**Pendekatan:** Shared database dengan kolom `tenant_id` pada setiap tabel.

**Package:** `stancl/tenancy` untuk otomatis resolve tenant berdasarkan domain dan scoping data.

**Alasan:**
- Lebih efisien untuk fase awal (1 database, 1 deployment)
- Mudah di-maintain dibanding multi-database
- Bisa migrasi ke database per-tenant nanti jika diperlukan

---

## Role & Permission

```mermaid
graph LR
    subgraph ROLES["👥 User Roles"]
        R1["🔑 Super Admin\nSpionotek"]
        R2["👔 Admin Institusi"]
        R3["👨‍🏫 Pengajar"]
        R4["👨‍🎓 Pelajar"]
        R5["👨‍👩‍👧 Orang Tua"]
    end

    subgraph ACCESS["🔐 Access Level"]
        A1["Manage all tenants"]
        A2["Manage own institution"]
        A3["Manage courses, exams, grades"]
        A4["Access learning, take exams"]
        A5["View progress & reports (READ ONLY)"]
    end

    R1 --> A1
    R2 --> A2
    R3 --> A3
    R4 --> A4
    R5 --> A5

    style ROLES fill:#1a1a2e,stroke:#e94560,color:#ffffff
    style ACCESS fill:#16213e,stroke:#0f3460,color:#ffffff
```

**Package:** `spatie/laravel-permission`

| Role | Deskripsi | Akses |
|---|---|---|
| **Super Admin** | Tim Spinotek | Kelola semua tenant dan konfigurasi sistem |
| **Admin Institusi** | Staff institusi | Kelola institusi mereka (user, setting, branding) |
| **Pengajar** | Dosen / guru / trainer | Kelola course, soal, ujian, nilai |
| **Pelajar** | Siswa / mahasiswa / peserta | Akses materi, ikut ujian, lihat nilai |
| **Orang Tua** | Wali siswa | Lihat progres & laporan anak (read-only) |

---

## Package Ecosystem

### Core Packages

| Package | Fungsi |
|---|---|
| `laravel/framework` | Core backend |
| `laravel/sanctum` | API token authentication |
| `stancl/tenancy` | Multi-tenant architecture |
| `spatie/laravel-permission` | Role-based access control |

### Feature Packages

| Package | Fungsi | Modul |
|---|---|---|
| `barryvdh/laravel-dompdf` | Generate sertifikat PDF | Certification |
| `maatwebsite/laravel-excel` | Import/export soal & nilai | Exam, Analytics |
| `spatie/laravel-medialibrary` | Upload materi & file | LMS |
| `laravel/reverb` | WebSocket real-time | Exam (timer), Notification |
| `openai-php/laravel` | Integrasi OpenAI API | AI modules |

### Development & DevOps

| Tool | Fungsi |
|---|---|
| `laravel/pint` | Code style & formatting |
| `pestphp/pest` | Testing framework |
| `laravel/telescope` | Debugging & monitoring (dev) |
| `laravel/horizon` | Queue monitoring (production) |
| `laravel/forge` / Ploi | Server deployment |

---

## White Label Theming

Tailwind CSS sangat cocok untuk white-label karena warna bisa dikonfigurasi secara dinamis:

```
Setiap tenant memiliki konfigurasi:
├── Primary Color     → Warna utama institusi
├── Secondary Color   → Warna aksen
├── Logo              → Logo institusi
├── Favicon           → Favicon institusi
├── Domain            → Custom domain
└── Email Identity    → Nama pengirim email
```

Implementasi menggunakan CSS custom properties yang di-set berdasarkan tenant config, sehingga Tailwind classes tetap sama tapi warna berubah per institusi.

---

## Infrastruktur Deployment

```mermaid
graph TB
    subgraph PRODUCTION["🌐 Production"]
        LB["Load Balancer"]
        APP1["App Server 1"]
        APP2["App Server 2"]
        WORKER["Queue Workers"]
        CRON["Task Scheduler"]
    end

    subgraph SERVICES["🔧 Services"]
        DB["MySQL 8"]
        CACHE["Redis"]
        STORAGE_SVC["Object Storage\n(S3 / MinIO)"]
        MAIL["Mail Service\n(SES / Mailgun)"]
    end

    LB --> APP1
    LB --> APP2
    APP1 --> DB
    APP2 --> DB
    APP1 --> CACHE
    APP2 --> CACHE
    WORKER --> DB
    WORKER --> CACHE
    APP1 --> STORAGE_SVC
    APP2 --> STORAGE_SVC
    APP1 --> MAIL

    style PRODUCTION fill:#1a1a2e,stroke:#e94560,color:#ffffff
    style SERVICES fill:#16213e,stroke:#0f3460,color:#ffffff
```

| Opsi Deployment | Keterangan |
|---|---|
| **Cloud Hosted** | Di-manage Spinotek (DigitalOcean / AWS) via Laravel Forge |
| **On-Premise** | Di-host di server institusi (untuk kebutuhan data sovereignty) |
| **Hybrid** | Kombinasi cloud + on-premise sesuai kebijakan institusi |

---

## Ringkasan Arsitektur

```
┌──────────────────────────────────────────────────────┐
│                    FRONTEND                          │
│           Vue 3 + Tailwind CSS + Axios               │
│              (via Vite, dalam Laravel)                │
├──────────────────────────────────────────────────────┤
│                    REST API                          │
│              Laravel Sanctum (auth)                   │
├──────────────────────────────────────────────────────┤
│                    BACKEND                           │
│    Laravel 11+ (API Routes, Multi-Tenancy)            │
│    Queue · Scheduler · Storage · Broadcasting        │
├──────────────────────────────────────────────────────┤
│                   DATA LAYER                         │
│           MySQL 8 · Redis · S3/MinIO                 │
└──────────────────────────────────────────────────────┘
```
