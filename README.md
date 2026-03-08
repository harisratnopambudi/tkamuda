# 📝 Simulasi TKA Matematika – SDS Plus 2 AlMuhajirin

Aplikasi **Computer-Based Test (CBT)** untuk simulasi Tes Kemampuan Akademik (TKA) Matematika SD. Dibangun sebagai single HTML file dengan Supabase sebagai backend.

![Preview](https://img.shields.io/badge/status-production-brightgreen) ![HTML](https://img.shields.io/badge/stack-HTML%20%2F%20JS%20%2F%20CSS-orange) ![Supabase](https://img.shields.io/badge/backend-Supabase-3ECF8E?logo=supabase)

---

## ✨ Fitur Utama

### Untuk Siswa
- **Login** dengan pilihan kelas dan nama (tanpa password)
- **39 soal** dengan 3 tipe: Pilihan Ganda, Benar/Salah, dan Multi-Pilih
- **Timer countdown** dengan peringatan saat waktu hampir habis
- **Navigasi bebas** antar soal lewat grid navigator
- **Auto-submit** saat waktu habis
- **Gambar soal** — setiap soal dan opsi jawaban bisa dilengkapi gambar
- **Session recovery** — jika koneksi terputus atau browser ditutup, jawaban tetap tersimpan dan ujian bisa dilanjutkan dari soal terakhir
- **Hasil ujian** — tampilkan/sembunyikan skor sesuai pengaturan admin
- **Pembahasan** — perbandingan jawaban siswa vs kunci jawaban (bisa diaktifkan admin)

### Untuk Admin
- **Dashboard** — rekap nilai seluruh siswa, filter per kelas, export CSV
- **🟢 Live Monitor** — pantau progress siswa yang sedang mengerjakan secara realtime (auto-refresh 5 detik), dikelompokkan per kelas
- **Data Siswa** — manajemen kelas dan daftar siswa (tambah, hapus, reset jawaban)
- **Analisa Soal** — persentase jawaban benar per soal dari seluruh peserta
- **Foto Soal** — upload gambar untuk teks soal dan setiap opsi jawaban (tersimpan di Supabase Storage)
- **Kunci Jawaban** — tampilan kunci lengkap 39 soal
- **Pengaturan:**
  - Durasi ujian (menit)
  - Acak urutan soal per siswa
  - Tampilkan/sembunyikan skor kepada siswa
  - Tampilkan/sembunyikan pembahasan & kunci jawaban
  - Ganti password admin

---

## 🛠️ Tech Stack

| Komponen | Teknologi |
|---|---|
| Frontend | HTML5, CSS3, Vanilla JavaScript |
| Backend | [Supabase](https://supabase.com) (PostgreSQL + Storage) |
| Font | Nunito & Baloo 2 (Google Fonts) |
| Deploy | Bisa di-host di mana saja (static file) |

---

## 🗄️ Setup Supabase

### 1. Buat Project di Supabase

Daftar di [supabase.com](https://supabase.com), buat project baru, lalu catat **Project URL** dan **anon key**.

### 2. Buat Tabel-tabel

Jalankan SQL berikut di **SQL Editor** Supabase:

```sql
-- Tabel pengaturan aplikasi
create table settings (
  id int primary key default 1,
  duration int default 90,
  password text default 'admin123',
  show_score boolean default true,
  show_pembahasan boolean default false,
  shuffle_soal boolean default false
);
insert into settings (id) values (1);

-- Tabel kelas dan daftar siswa
create table classes (
  id uuid primary key default gen_random_uuid(),
  name text unique not null,
  students jsonb default '[]'
);

-- Tabel hasil ujian
create table results (
  id uuid primary key default gen_random_uuid(),
  student_name text,
  class_name text,
  score int,
  benar int,
  salah int,
  kosong int,
  duration int,
  answers jsonb,
  submitted_at timestamptz default now(),
  unique(student_name, class_name)
);

-- Tabel progress realtime (Live Monitor)
create table progress (
  id text primary key,
  student_name text,
  class_name text,
  current_q int default 0,
  answered int default 0,
  total int default 39,
  started_at timestamptz,
  updated_at timestamptz default now(),
  status text default 'active'
);

-- Kolom tambahan untuk settings
alter table settings add column if not exists show_score boolean default true;
alter table settings add column if not exists show_pembahasan boolean default false;
alter table settings add column if not exists shuffle_soal boolean default false;
```

### 3. Setup Row Level Security (RLS)

```sql
-- Aktifkan RLS lalu izinkan akses anonymous
alter table settings enable row level security;
alter table classes enable row level security;
alter table results enable row level security;
alter table progress enable row level security;

create policy "Public access" on settings for all using (true) with check (true);
create policy "Public access" on classes for all using (true) with check (true);
create policy "Public access" on results for all using (true) with check (true);
create policy "Public access" on progress for all using (true) with check (true);
```

### 4. Setup Storage (Foto Soal)

```sql
-- Buat bucket untuk gambar soal
insert into storage.buckets (id, name, public)
values ('soal-images', 'soal-images', true);

-- Policy akses storage
create policy "Public read" on storage.objects
  for select using (bucket_id = 'soal-images');

create policy "Anon upload" on storage.objects
  for insert with check (bucket_id = 'soal-images');

create policy "Anon delete" on storage.objects
  for delete using (bucket_id = 'soal-images');
```

### 5. Konfigurasi di index.html

Buka file `index.html`, cari baris berikut dan ganti dengan URL dan key project Anda:

```javascript
const SUPABASE_URL = 'https://YOUR_PROJECT_ID.supabase.co';
const SUPABASE_KEY = 'YOUR_ANON_KEY';
```

---

## 🚀 Cara Deploy

Karena ini single HTML file, tidak perlu build process apapun. Cukup:

### Opsi A — GitHub Pages
1. Push `index.html` ke repository GitHub
2. Buka **Settings → Pages**
3. Source: **Deploy from branch → main → / (root)**
4. Aplikasi live di `https://username.github.io/nama-repo`

### Opsi B — Netlify / Vercel
1. Drag & drop folder ke [netlify.com/drop](https://app.netlify.com/drop)
2. Atau connect repository GitHub untuk auto-deploy

### Opsi C — Hosting Lokal / LAN Sekolah
1. Taruh `index.html` di folder server (Apache/Nginx/XAMPP)
2. Atau buka langsung di browser sebagai file local (beberapa fitur Supabase tetap butuh koneksi internet)

---

## 🗂️ Struktur Soal

Soal disimpan dalam array `QUESTIONS` di dalam file HTML. Tersedia 3 tipe soal:

```javascript
// Tipe 1: Pilihan Ganda
{
  id: 1, type: 'single',
  text: 'Pertanyaan...',
  options: ['A', 'B', 'C', 'D'],
  answer: 2  // index jawaban benar (0-based)
}

// Tipe 2: Benar / Salah
{
  id: 2, type: 'truefalse',
  text: 'Pertanyaan...',
  statements: [
    { text: 'Pernyataan 1', answer: true },
    { text: 'Pernyataan 2', answer: false },
  ]
}

// Tipe 3: Multi Pilih
{
  id: 3, type: 'multiselect',
  text: 'Pertanyaan...',
  options: ['A', 'B', 'C', 'D'],
  answers: [0, 2, 3]  // semua index yang benar
}
```

---

## 🔑 Login Admin

Password admin dapat diubah di panel **Admin → ⚙️ Pengaturan → Ganti Password Admin**.

---

## 📱 Live Monitor

Panel **🟢 Live Monitor** memungkinkan admin memantau seluruh siswa secara realtime:

- Dikelompokkan per kelas dengan tab filter
- Setiap card menampilkan: nama siswa, progress bar, nomor soal saat ini, jumlah soal terjawab, dan waktu pengerjaan
- Status warna: 🟢 Hijau = aktif, 🟡 Kuning = koneksi bermasalah, ✅ Abu = selesai
- Auto-refresh setiap **5 detik**

---

## 🔌 Session Recovery (Anti Putus Koneksi)

Jika koneksi siswa terputus di tengah ujian:
- Jawaban otomatis tersimpan di **localStorage** setiap kali ada perubahan
- Saat siswa login kembali dengan nama & kelas yang sama, muncul dialog untuk **melanjutkan dari soal terakhir**
- Sesi otomatis kedaluwarsa setelah **4 jam**
- Notifikasi muncul saat koneksi putus/kembali

---

## 📄 Lisensi

Proyek ini dibuat untuk keperluan internal SDS Plus 2 AlMuhajirin. Bebas dimodifikasi untuk kebutuhan sekolah lain.

---

*Dibuat dengan ❤️ untuk kemudahan pelaksanaan simulasi TKA di sekolah*
