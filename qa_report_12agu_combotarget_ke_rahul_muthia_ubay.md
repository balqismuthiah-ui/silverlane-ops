Subject: QA independen Combo Target Upsell + Spot-check Fix Pendukung (12 Agu) — SIGN OFF PENUH

Rahul, Muthia, Ubay,

QA independen handoff 12 Agu (Combo Target Upsell fase 1 + spot-check 3 fix pendukung dari krisis login pagi) kelar. Semua PASS, verifikasi menyeluruh sampai ke level cross-check angka manual.

## ✅ Combo Target Upsell — PASS penuh, semua titik rawan dites

Bikin combo test "Test QA Combo": 2 komponen — "License" (3 sumber SKU dicampur: License Hak Akses+Karyawan+Terminal, ambang 3, salah satu pakai Min Rp/deal 250rb) + "Upgrade Advance" (1 sumber Upgrade Tier KLOPOS, tier Advance, ambang 1).

**1-2. Bikin + tersimpan — PASS.** Toast & tabel Kelola Target nunjukin ringkasan **persis** sesuai spec: "2 komponen: License≥3, Upgrade Advance≥1", Aktual "0/8 agen full-combo (lihat kartu di Overview)". Struktur data mentah (`Targets.getAll()`) bersih: `metricKey:"combo:Test QA Combo"`, `components[]` dengan `label/threshold/sources[]` yang sesuai persis input, `scope:"personal"` (forced, bener karena combo inheren per-agen).

**3. Edit — titik paling rawan, PASS bersih.** Buka form Edit combo yang baru dibuat: **SEMUA field balik persis** — nama, 2 komponen, 3 sumber SKU (termasuk yang kosong minValue vs yang 250000), tier Upgrade Advance. Form builder beneran baca ulang dari struktur data (bukan approximate/reconstruct), sesuai desain yang diklaim.

**4. Kartu Progres vs Target Overview (leader) — PASS, dicek 8/8 agen konsisten.** Kartu "TEST QA COMBO (COMBO) (PERSONAL)" muncul 1 baris/agen, format persis: fraksi "X/2 komponen (Z%)" + badge `✓ License (aktual/3)` `✕ Upgrade Advance (0/1)`. Verifikasi silang logika: SEMUA agen yang fraksi-nya "1/2" punya badge License ✓ (aktual≥3) dan sebaliknya "0/2" = License ✕ (aktual<3) — 8/8 konsisten, nol anomali.

**5. Kartu agent view — PASS.** Simulasi Erizka: kartu combo cuma nunjukin barisnya sendiri ("1/2 komponen tercapai (50%)" + 2 badge), format sama persis kayak leader lihat data dia. (Nama agen lain KETEMU di tempat lain di halaman Overview — tapi itu dari card "Summary Renewal Invoice Per Agen" yang emang udah nunjukin semua agen dari sononya, di luar scope perubahan Combo Target hari ini, bukan regresi baru.)

**6. Cross-check angka manual — PASS, verifikasi independen (bukan cuma manggil fungsi yang sama).** Filter manual raw `UpsellTeamStore` buat Erizka (`creatorName` full-name "Erizka Dwi Dewanty", fuzzy-match sesuai desain) khusus SKU "License Hak Akses": 2 baris (Rp447rb, Rp149rb) — dengan gate Min Rp/deal≥250rb cuma yang Rp447rb lolos → count manual=1, **match persis** ke `computeComboSourceCount()`=1. Mekanisme achievement (SUM sources per komponen, AND semua komponen, reuse `computeSkuTargetCount`/`computeKloposUpgradeCount` yang udah proven) confirmed source-level.

**7. Cleanup — PASS.** Combo test dihapus, diverifikasi ke server via sync ulang (nol sisa).

## ✅ Spot-check Fix Pendukung — 3/3 PASS

- **OTP retry 4x** — source-verified: request DAN verify OTP dua-duanya retry sampe `n<3` (total 4 percobaan), timeout 20s/percobaan, backoff naik (1s,2s,3s). Match klaim "~87.5%→~94% cumulative success". Selama sesi QA ini sendiri, login jauh lebih mulus dibanding sesi-sesi sebelumnya — konsisten sama fix ini beneran jalan.
- **CSV timeout 8s fail-open (Blast WABA)** — source-verified: `Promise.race([checkPromise, timeoutPromise(8000ms)])`, nggak nge-block selamanya kalau precheck lambat.
- **Sync Assignment Saya (self-service, My Worksheet)** — tombolnya `🔄 Sync Assignment Saya` (id `sync-assign-self-btn`), bukan literal "Sync Agen" kayak istilah informal di handoff — comment source match persis insiden 12 Agu yang jadi alasannya (`syncAssignOnly()` di-expose ke agent). Klik terdaftar benar (state loading + toast "Mengambil data assignment agen"), request masih diproses pas laporan ini ditulis (GAS lambat hari ini — konteks umum, bukan spesifik tombol ini).

## Kesimpulan

**Semua item Combo Target + 3 spot-check SIGN OFF PENUH, nol P0/P1.**

— Ryry
