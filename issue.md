# Planning: Fullstack Excel Processing Application

## 🛠️ Technology Stack
- **Runtime & Package Manager**: [Bun](https://bun.sh/)
- **Backend Framework**: [ElysiaJS](https://elysiajs.com/)
- **Database**: Postgres (via Drizzle ORM atau Prisma)
- **Excel Parsing**: `exceljs` atau `xlsx` (SheetJS)
- **Frontend**: React + Tailwind CSS (via Vite)

## 📋 Objectives
Membangun aplikasi web fullstack untuk mengunggah, memproses, dan menyimpan data dari file Excel (XLSX/CSV) ke dalam database Postgres, dengan antarmuka yang modern, dinamis, dan responsif.

## 🏗️ Architecture & Project Structure
Proyek akan menggunakan arsitektur monorepo sederhana yang berisi backend dan frontend dalam satu repositori.

```text
/
├── backend/            # ElysiaJS + Bun
│   ├── src/
│   │   ├── controllers/  # Route handlers
│   │   ├── db/           # Database setup & schema
│   │   ├── services/     # Excel parsing logic
│   │   └── index.ts      # App entry point
│   └── package.json
└── frontend/           # React + Vite + Tailwind
    ├── src/
    │   ├── components/   # UI Components (Uploader, Table)
    │   ├── hooks/        # Custom React hooks (Data fetching)
    │   └── App.tsx
    ├── tailwind.config.js
    └── package.json
```

## 🚀 Implementation Phases

### Phase 1: Setup & Initialization
- [ ] Inisialisasi workspace.
- [ ] Setup backend menggunakan ElysiaJS (`bun create elysia backend`).
- [ ] Setup frontend menggunakan React + Vite (`bun create vite frontend --template react-ts`).
- [ ] Konfigurasi Tailwind CSS di frontend untuk styling.
- [ ] Setup database Postgres (bisa menggunakan Docker Compose atau cloud provider seperti Supabase/Neon).
- [ ] Konfigurasi ORM (Drizzle direkomendasikan karena sangat cepat dan cocok dengan Bun) dan definisikan schema untuk menampung data dari Excel.

### Phase 2: Backend Development (API & Data Parsing)
- [ ] Konfigurasi koneksi database di backend.
- [ ] Buat plugin/handler CORS di Elysia.
- [ ] Buat endpoint API untuk upload file (`POST /api/upload`).
- [ ] Integrasikan library `exceljs` untuk membaca dan mem-parsing buffer file Excel yang diunggah.
- [ ] Buat logika validasi dan transformasi data baris Excel menjadi objek JSON.
- [ ] Lakukan *bulk insert* data hasil parsing ke dalam Postgres.
- [ ] Buat endpoint API (`GET /api/data`) untuk mengambil data yang tersimpan (dengan fitur paginasi/search).

### Phase 3: Frontend Development (UI/UX)
- [ ] Desain antarmuka utama yang terlihat premium dengan Tailwind CSS (menggunakan glassmorphism, gradient, dsb).
- [ ] Implementasi komponen **Drag-and-Drop File Uploader** dengan feedback visual saat file ditarik dan proses upload berjalan.
- [ ] Integrasi API upload ke backend menggunakan `fetch` atau `axios`.
- [ ] Tampilkan toast notification (sukses/gagal).
- [ ] Implementasi komponen **Data Table** modern untuk menampilkan data yang berhasil disimpan di database.
- [ ] Tambahkan animasi dan transisi mikro saat berinteraksi (hover state, loading state).

### Phase 4: Polish & Deployment
- [ ] Lakukan error handling menyeluruh (contoh: jika format kolom Excel tidak valid, atau file terlalu besar).
- [ ] Pastikan UI responsif di perangkat mobile maupun desktop.
- [ ] Siapkan script untuk build produksi (`bun run build`).

## 📦 Key Dependencies Preparation

**Backend:**
```bash
# Di dalam folder backend/
bun add elysia @elysiajs/cors
bun add exceljs
bun add drizzle-orm postgres
bun add -D drizzle-kit
```

**Frontend:**
```bash
# Di dalam folder frontend/
bun add lucide-react clsx tailwind-merge # Icon & utility classes
bun add -D tailwindcss postcss autoprefixer
```
