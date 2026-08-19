# Planning: Fullstack Cafe Membership CRM & Excel Closing Validation Engine

## 🎯 Business Rules & Specifications
- **Pendaftaran Member Baru:** 
  - Biaya: Rp 30.000.
  - Masa aktif: 3 bulan (dihitung sejak tanggal pendaftaran).
  - Bonus awal: Otomatis mendapatkan **Stamp #1** + **Voucher Reward #1** (Free 1 cup: Americano / Cappuccino / Latte).
- **Benefit Member Aktif:**
  - Diskon tetap 15% untuk setiap transaksi.
  - Tambah 1 Stamp untuk setiap minimum transaksi belanja Rp 50.000 (tidak ada sistem kelipatan, min. spending Rp 50k = 1 stamp).
    - *Catatan:* Perhitungan minimum belanja Rp 50.000 diambil dari **Grand Total** (setelah diskon 15% diaplikasikan).
  - **Aturan Transaksi:** Transaksi belanja biasa **TIDAK** menambah masa aktif (*expired date* tidak berubah).
  - Tidak dapat digabungkan dengan promo lainnya.
- **Loyalty Stamp Card Milestone:**
  - **Stamp #1:** Free Americano / Cappuccino / Latte (diperoleh saat daftar).
  - **Stamp #10:** Free 1 All Product.
  - **Siklus Kartu:** Setelah mencapai Stamp #10, member akan mendapatkan **Kartu Virtual Baru** (Stamp di-reset). Sistem akan memberikan nomor/urutan kartu sehingga bisa di-tracking member sedang berada di kartu ke-berapa.
- **Siklus Perpanjangan (*Renewal*) & Kedaluwarsa:**
  - **Perpanjangan (sebelum expired):** Biaya Rp 30.000 -> Menambah masa aktif +3 bulan dan mendapatkan bonus **1 Stamp**.
  - **Setelah Expired:** Jika masa aktif lewat tanpa perpanjangan, status member hangus/reset dan harus mendaftar ulang dari awal (registrasi baru Rp 30.000, reset stamp ke #1).

---

## 📋 Implementation Checklist

### 1. Database Connection & Schema (Postgres + Drizzle)
- [ ] Setup koneksi PostgreSQL menggunakan Drizzle ORM.
- [ ] Buat skema tabel `members`: `id`, `qr_code_id` (UUID v4), `name`, `phone`, `stamps`, `current_card_number` (default: 1), `joined_at`, `expired_at`.
- [ ] Buat skema tabel `vouchers`: `id`, `member_id`, `type` (e.g., 'FREE_CUP', 'FREE_ALL'), `is_used`, `created_at`.
- [ ] Buat skema tabel `transactions`: `id`, `member_id`, `amount`, `discount_applied`, `stamps_earned`, `type` (REGISTRATION / RENEWAL / PURCHASE), `created_at`.
- [ ] Buat skema tabel `pos_reconciliations`: `id`, `date`, `excel_total`, `system_total`, `is_fraud_detected`, `details`.
- [ ] Setup migrasi dan sinkronisasi database (`db:push`).

### 2. Backend Excel Engine (ElysiaJS + Bun)
- [ ] Integrasikan library `exceljs` untuk membaca laporan penutupan POS.
- [ ] Buat endpoint `POST /api/closing/upload` untuk menerima file laporan Excel.
- [ ] **Anti-Fraud Logic**: 
  - Validasi total diskon 15% di sistem kasir terhadap riwayat transaksi aktif di database CRM.
  - Cek kecocokan pengeluaran voucher (Free Cup / Free All) antara Excel POS dan database.
- [ ] Simpan hasil validasi ke tabel `pos_reconciliations` dan kembalikan response *Match/Discrepancy*.

### 3. Loyalty & Stamp System Logic
- [ ] **API Registrasi / Daftar Ulang**: 
  - Proses payment Rp 30.000, generate UUID QR, set `expired_at` 3 bulan.
  - Berikan otomatis 1 Stamp dan catat 1 voucher `FREE_CUP` di tabel `vouchers`.
- [ ] **API Transaksi Kasir**:
  - Tolak transaksi jika `expired_at` member sudah lewat (paksa daftar ulang).
  - Hitung **Grand Total** (setelah dipotong diskon 15%).
  - Validasi minimal belanja Grand Total >= Rp 50.000 -> Jika ya, berikan *tepat* 1 Stamp (tanpa kelipatan).
  - Jika stamp mencapai angka 10:
    - Catat voucher `FREE_ALL` di tabel `vouchers`.
    - Reset saldo `stamps` menjadi 0.
    - Naikkan `current_card_number` + 1 (member masuk ke siklus kartu virtual berikutnya).
  - **Ingat:** `expired_at` TIDAK diperbarui pada endpoint ini.
- [ ] **API Perpanjangan (Renewal)**:
  - Hanya bisa diakses jika member **belum** kedaluwarsa.
  - Tambah masa aktif +3 bulan dari `expired_at` saat ini, serta tambahkan 1 Stamp.
- [ ] **API Redeem Voucher**: Ubah flag `is_used = true` pada voucher yang dipilih pelanggan.

### 4. Frontend (React + Tailwind)
- [ ] **QR Scanner Kasir**: Implementasi `@zxing/browser` atau `html5-qrcode` agar kasir bisa langsung mendeteksi UUID member.
- [ ] **Dashboard Kasir**: 
  - Tampilkan alert kuning jika member mendekati expired, atau alert merah jika sudah expired.
  - Form transaksi yang otomatis menghitung apakah transaksi berhak dapat stamp atau tidak.
  - UI daftar Voucher yang dimiliki untuk langsung di-redeem.
- [ ] **Drag & Drop File Upload**: Komponen upload UI untuk kasir mengunggah Excel *closing* POS.
- [ ] **Dashboard Admin (Anti-Fraud)**: Halaman laporan rekonsiliasi yang memberi peringatan jika terdeteksi manipulasi diskon.

### 5. Security & Deployment
- [ ] **Security**: JWT Auth, Enkripsi UUID QR Code, perlindungan endpoint dengan role kasir vs admin.
- [ ] **Deployment**: Docker (Backend), Vercel/Netlify (Frontend), layanan Database tersentralisasi (Supabase/Neon).
