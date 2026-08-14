Subject: Handoff QA — Migrasi Targets ke Sheet + 3 fix produksi (13-14 Agu)

Ryry,

Sesi 13-14 Agu ini lanjutan dari handoff Combo Target 12 Agu — QA kamu tanggal itu udah cover fase 1 sebelum migrasi Sheet terjadi, jadi bagian penyimpanan Target-nya SEKARANG beda infrastruktur & perlu diverifikasi ulang, plus ada 3 fix produksi lain. Semua udah live di `https://clm.ryanza.my.id/churn.html`.

## 🔴 1. PRIORITAS UTAMA — Migrasi Targets: Script Properties → Sheet tab TARGETS

**Kenapa:** `targets_Aug` (Script Properties, batas ~9KB) kelewat batas begitu Combo Target nambah data, bikin beberapa target gagal sync ke server ("HTTP 404"). Dipindah ke tab Sheet baru `TARGETS` (spreadsheet sama kayak BLASTED_NUMBERS), pola udah proven (GC Watchlist/Leader Reassignment/BLASTED_NUMBERS).

**Cara QA (WAJIB, bandingin satu-satu, bukan sampling):**
1. Buka tab Sheet `TARGETS` — bandingin isinya (per baris, kolom `Data`) vs semua target yang kamu tau harusnya ada dari sebelum migrasi (Kelola Target di dashboard). Total ada 21 target Agustus yang dimigrasi — pastiin nggak ada yang hilang.
2. Di dashboard, buka "Kelola Target" — bikin 1 target test baru (metrik biasa, bukan combo), simpan, pastiin **nggak ada toast "gagal sync"**, dan muncul di Sheet `TARGETS` (bukan di Script Properties lama).
3. Edit 1 target existing, ubah nilai, simpan, cek Sheet-nya ke-update (bukan bikin baris baru — upsert-by-id).
4. Hapus target test itu, cek row-nya beneran ilang dari Sheet.
5. **4 Combo Target ini sempat kena bug tersendiri (components sempat kosong di server, sekarang udah di-resave manual) — WAJIB dicek ulang satu-satu, bukan diasumsikan beres:**
   - Food Order/Marketplace
   - Lisensi
   - Majoo Teams/Upgrade Enterprise
   - Add On Keuangan/Upgrade Advance
   
   Buka Edit Target masing-masing, pastiin semua komponen+sumbernya lengkap & bener (bandingin ke Sheet `TARGETS` kolom Data langsung kalau perlu), dan kartu Overview-nya nunjukin angka progress yang masuk akal (bukan 0/0).

## 2. Fix Combo Target `klopos_upgrade` — basis periode salah kolom tanggal (commit `c35007f`)

**Bug:** komponen combo tipe "Upgrade Tier (KLOPOS)" filter periodenya pakai `cohortMonths` (siklus expiry-renewal CASE) — padahal seharusnya pakai `tanggalBayar` (tanggal pembayaran aktual). Case yang expiry-cycle-nya beda bulan dari tanggal bayar aslinya jadi salah ke-exclude/include dari target bulan tertentu.

**Cara QA:** cari 1-2 outlet yang kamu tau persis productPackage-nya "Advance upgrade"/"Enterprise upgrade" + tanggal bayarnya, pastiin ke-hitung di target combo bulan yang SESUAI TANGGAL BAYAR (bukan cohort case-nya). Kalau ada combo target yang pakai sumber "Upgrade Tier" hidup di production, cross-check angkanya manual vs data KLOPOS Master (kolom `productPackage` + `tanggalBayar`, kolom AF "Tanggal Bayar").

## 3. Fix Invoice Tracker — dibatesin 3 bulan terakhir (GAS-side, tanpa commit churn.html)

**Kenapa:** `InvoiceTrackerStore` (localStorage) nyimpen SEMUA histori invoice tanpa batas, kena `QuotaExceededError` ("exceeded the quota") pas sync. Endpoint GAS `getInvoiceTracker` sekarang cuma balikin invoice dengan `createdDate` di rolling window: bulan berjalan + 2 bulan ke belakang (operator konfirmasi: invoice lebih lama dari itu udah expired >62 hari, nggak bisa reaktivasi lagi, jadi nggak perlu ditarik).

**Cara QA:**
1. Klik "📥 Sync Invoice Tracker" di Master & Assign — pastiin **sukses tanpa error quota**.
2. Cek tab Invoice Tracker & Invoice Summary — datanya cuma nyakup 3 bulan terakhir (created date), bukan tahun penuh. Ini **expected**, bukan bug — invoice lebih lama emang sengaja nggak ditarik lagi.
3. Cek invoice yang `createdDate`-nya kosong/nggak kebaca (kalau ada) — harusnya TETAP ikut ketarik (safety net, "0 jujur" — nggak diam-diam dibuang kalau nggak jelas statusnya).

## 4. Fix filter & badge "Lifecycle" di Master & Assign (commit `574780f`)

**Bug:** filter Lifecycle selalu balikin kosong, badge warnanya selalu abu-abu polos — root cause: opsi filter & map warna hardcode ke label HWCZ (Hot/Warm/Cold/Zombie/Churn), padahal `lifecycleStatus` asli isinya beda total (`active`/`grace_period`/`hot_period`/`cold_period`/`zombie_period`).

**Cara QA:**
1. Buka filter Lifecycle di Master & Assign, coba pilih tiap opsi satu-satu (5 status) — pastiin hasilnya BUKAN kosong, dan baris yang muncul emang punya `lifecycleStatus` yang sesuai (bisa cek di modal detail outlet, "Lifecycle Status").
2. Cek warna badge di kolom Lifecycle tabel: `active` & `hot_period` = hijau, `grace_period` = biru, `cold_period` = abu-abu, `zombie_period` = merah.

## Ringkasan prioritas QA

1. **Migrasi Targets ke Sheet (poin 1)** — paling kritis, ini infrastruktur penyimpanan yang berubah total, dan 4 combo target butuh verifikasi ulang spesifik.
2. Fix `klopos_upgrade` tanggal bayar (poin 2) — kalau ada combo target real yang pakai sumber ini.
3. Invoice Tracker window 3 bulan (poin 3) — cek sync sukses + datanya sesuai window.
4. Filter/badge Lifecycle (poin 4) — quick check 5 status.

Kabarin kalau ada temuan atau begitu semua PASS.

— Rahul
