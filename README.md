# POWER CALCULATOR ELECTRICAL BMI V1

Aplikasi web kalkulasi teknik listrik dengan Vite, Supabase Auth/Postgres, dan GitHub Pages.

## Fitur
- Login email/password dan pendaftaran akun melalui Supabase Auth.
- Kalkulator daya 1 fasa/3 fasa: kW, kVA, kVAR, dan output berdasarkan efisiensi.
- Kalkulator motor: arus, daya input/output, faktor daya, efisiensi.
- Kalkulator voltage drop kabel tembaga (estimasi resistif sederhana).
- Estimasi awal rating MCB/MCCB dari arus beban dan faktor desain.
- BOM panel dan export PDF.
- Riwayat kalkulasi per akun, simpan, hapus, export PDF.

## 1. Konfigurasi Supabase
Project URL: `https://chuddueeusvkoqlfvdwh.supabase.co`
Project ref: `chuddueeusvkoqlfvdwh`

Tabel `public.power_calculations` dan `public.power_bom_items` sudah dibuat pada project tersebut. Tabel kalkulasi memiliki kolom `user_id`, `project_name`, `calculation_type`, `input_data`, `result_data`, `notes`, `created_at`, `updated_at`. RLS membatasi setiap pengguna ke data miliknya.

Di Supabase Dashboard:
1. Authentication → Providers → Email: aktifkan Email provider.
2. Authentication → URL Configuration: masukkan URL GitHub Pages ke Site URL dan Redirect URLs, contoh `https://USERNAME.github.io/REPOSITORY_NAME/`.
3. Project Settings → API Keys: salin Project URL dan **publishable key** (`sb_publishable_...`). Jangan pernah menggunakan `service_role` atau secret key di browser.

Jika ingin memeriksa ulang tabel/policy, jalankan SQL berikut di SQL Editor (aman sebagai pemeriksaan; tidak membuat ulang tabel):

```sql
select column_name, data_type
from information_schema.columns
where table_schema='public' and table_name='power_calculations'
order by ordinal_position;

select policyname, cmd, roles
from pg_policies
where schemaname='public' and tablename='power_calculations';
```

## 2. Jalankan lokal

```bash
cp .env.example .env
# Isi VITE_SUPABASE_URL dan VITE_SUPABASE_ANON_KEY
npm install
npm run dev
```

Untuk production build:

```bash
npm run build
npm run preview
```

## 3. Deploy GitHub Pages

1. Buat repository GitHub, misalnya `power-calculator-bmi`.
2. Upload seluruh isi folder ini ke root repository dan push ke branch `main`.
3. Tambahkan variable `VITE_BASE_PATH` di GitHub Actions Variables: `/REPOSITORY_NAME/` (contoh `/power-calculator-bmi/`). Jika memakai custom domain pada GitHub Pages, isi `/`.
4. GitHub repository → Settings → Secrets and variables → Actions → Variables → New repository variable:
   - `VITE_SUPABASE_URL` = `https://chuddueeusvkoqlfvdwh.supabase.co`
   - `VITE_SUPABASE_ANON_KEY` = publishable key `sb_publishable_...` dari Supabase.
   - `VITE_BASE_PATH` = `/power-calculator-bmi/` (sesuaikan nama repository; custom domain gunakan `/`).
5. Settings → Pages → Build and deployment → Source: **GitHub Actions**.
6. Push commit ke `main`. Buka tab Actions dan tunggu workflow sukses.
7. URL aplikasi: `https://USERNAME.github.io/REPOSITORY_NAME/`.
8. Tambahkan URL tersebut ke Supabase Auth URL Configuration (Site URL dan Redirect URLs).

> Publishable key memang dirancang untuk digunakan di frontend. Keamanan data tetap bergantung pada RLS. Jangan masukkan `service_role`, secret key, atau password ke repository.

## 4. Struktur folder

```text
power-calculator-bmi-v1/
├── .github/workflows/deploy.yml
├── src/main.js
├── src/style.css
├── index.html
├── package.json
├── vite.config.js
├── .env.example
└── README.md
```

## 5. Pengujian

1. Buka URL aplikasi, daftar dengan email dan password yang valid.
2. Selesaikan konfirmasi email jika Supabase mewajibkannya, lalu login.
3. Power Calculator: uji 3 fasa 380 V, 10 A, PF 0,85. Hasil sekitar 5,59 kW dan 6,58 kVA.
4. Motor Calculator: masukkan efisiensi 90%; pastikan daya output lebih kecil dari daya input.
5. Cable Calculator: uji 380 V, 25 A, 30 m, 6 mm²; periksa hasil voltage drop.
6. Protection Calculator: uji arus 32 A dan faktor 1,25; estimasi arus desain 40 A.
7. Simpan kalkulasi, buka Riwayat, refresh browser, dan pastikan data masih tampil.
8. Export PDF kalkulasi dan BOM.
9. Hapus satu riwayat, lalu pastikan hanya riwayat tersebut yang hilang.
10. Buat akun kedua dan pastikan akun kedua tidak dapat melihat riwayat akun pertama (uji RLS).

## Batasan engineering yang wajib diperhatikan
- Voltage drop menggunakan pendekatan resistansi tembaga ρ=0,0175 Ω·mm²/m dan reaktansi asumsi; bukan pengganti perhitungan kabel lengkap berdasarkan tabel pabrikan/PUIL.
- Belum memperhitungkan suhu, grouping, metode pemasangan, harmonisa, material aluminium, panjang jalur aktual, dan faktor koreksi ampacity.
- Rekomendasi MCB/MCCB hanya rating arus awal. Pemilihan final harus memeriksa kurva trip, breaking capacity (Icu/Ics), arus hubung singkat, starting motor, koordinasi selektivitas, dan kemampuan hantar arus kabel.
- BOM tersimpan di tabel `public.power_bom_items`, terikat pada akun pengguna melalui RLS.
- PDF dibuat di browser. Tidak ada file proyek atau kalkulasi yang dikirim ke server PDF pihak ketiga.
