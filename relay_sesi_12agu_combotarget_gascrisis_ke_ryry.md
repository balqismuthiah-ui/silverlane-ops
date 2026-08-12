Subject: Handoff QA — Combo Target Upsell (fitur baru) + fix krisis login GAS (12 Agu)

Ryry,

Sesi hari ini (12 Agu) ada 2 bagian: 1 fitur baru gede (Combo Target Upsell) + serangkaian fix darurat krisis login GAS pagi-siang tadi. Semua udah live di **`https://clm.ryanza.my.id/churn.html`**. Commit range: `24f8d0d` s.d. `b717193` (12 commit).

## 🆕 1. PRIORITAS UTAMA — Combo Target Upsell, fase 1 (`b717193`)

Fitur baru di tab **"⚙️ Kelola Target"**: leader sekarang bisa bikin 1 target yang isinya GABUNGAN beberapa syarat (komponen), bukan cuma 1 metrik tunggal kayak target biasa. Semua komponen harus tercapai bareng (AND) baru target-nya dianggap "achieved". Nol hardcode — leader bikin sendiri lewat form, bebas ganti tiap bulan.

**Cara akses:** Kelola Target → "+ Tambah Target" → dropdown **"Tipe Target"** pilih **"Kombinasi (Combo Target)"**.

**Cara QA (butuh leader role):**
1. **Bikin combo baru**: isi nama combo bebas (mis. "Test QA Combo"), klik "+ Tambah Komponen" 2-3 kali. Tiap komponen: isi nama + ambang batas, lalu "+ Tambah Sumber" — pilih tipe **SKU** (dropdown addonType, ada field opsional "Min Rp/deal") atau **Upgrade Tier (KLOPOS)** (dropdown Starter/Advance/Enterprise/Prime Plus). Coba campur beberapa SKU dalam 1 komponen (mis. 3 SKU License digabung).
2. **Simpan** → cek muncul di tabel Kelola Target (kolom Target nunjukin ringkasan "N komponen: Label≥angka, ..."), kolom Aktual nunjukin "X/Y agen full-combo".
3. **Edit combo yang barusan dibuat** → pastiin SEMUA yang diisi tadi (nama, komponen, sumber, threshold, min Rp/deal) muncul balik persis kayak yang disimpan — ini titik paling rawan bug (form builder-nya baca ulang dari struktur data, bukan dari input field yang di-preserve browser).
4. **Cek kartu "Progres vs Target" di Overview** (bulan yang sama dengan target combo tadi):
   - Login **leader** → muncul kartu combo dengan 1 baris per agen, tiap baris ada badge ✓/✕ per komponen (hijau=tercapai, merah=belum) + fraksi "X/Y komponen".
   - Login **agen** (kalau "Tampilkan ke Agen" dicentang) → kartu combo cuma nunjukin datanya sendiri, format sama (badge per komponen), BUKAN data agen lain.
5. **Hapus** target combo test setelah selesai QA (tombol ✕ di tabel Kelola Target) — biar nggak numpuk di data produksi.
6. **Angka aktual-nya sendiri** (opsional, kalau mau cross-check lebih dalam): komponen tipe SKU narik dari `UpsellTeamStore` (sama sumber kayak target per-SKU yang udah ada), komponen tipe "Upgrade Tier" narik dari KLOPOS Master kolom `productPackage` yang mengandung " upgrade" (mis. "Advance upgrade") di-join ke agen lewat outlet → tab Renewal. Kalau mau verifikasi manual, cari 1 outlet yang kamu tau paketnya baru upgrade pas renewal, cek attribusinya nyambung ke agen yang bener.

**Yang SENGAJA belum ada (fase 2, jangan dilaporkan sebagai bug):** upgrade mid-kontrak yang tercatat sebagai SKU "Upsell - Upgrade" (parsing teks bebas `bundleName`) — ditunda, cuma upgrade yang nempel di siklus renewal (Jalur B) yang udah jalan di fase 1 ini.

## 🔴 2. Krisis login GAS pagi-siang tadi — ringkasan buat konteks (nggak perlu di-QA ulang manual, background aja)

Ada window ~4 jam (10:00-14:00) di mana beberapa fungsi GAS (`requestOtp`, `data`, `assignments`) sempat 404/hang. Root cause AKHIRNYA ketemu: bukan infra Google, tapi **Script Properties kosong** di project GAS baru hasil emergency redeploy ("CODE AI 2") — udah di-fix manual, login normal lagi. Detail lengkap ada di `relay_gas_codeai_eskalasi_multi_endpoint_ke_ubay.md` kalau perlu.

Selama proses itu, ada beberapa fix client-side yang IKUT LIVE dan worth di-spot-check (bukan prioritas tinggi, tapi bagian dari commit range hari ini):

- **`80be3d9`, `24f8d0d`** — fix biar sesi nggak ke-logout paksa cuma gara-gara 1x gagal network pas re-check otorisasi (dulu infra-gagal dianggap sama kayak beneran unauthorized). Cara cek: pastiin sesi nggak tiba-tiba logout sendiri pas lagi kerja normal (susah direproduksi sengaja, ini lebih ke "monitoring", bukan test aktif).
- **`36436cf`** — download CSV Blast WABA nggak lagi bisa hang lama kalau pre-check-nya lambat (ada timeout 8 detik + fail-open). Cek: buka Blast WABA → download CSV → harusnya tetep jalan walau agak lambat, nggak stuck selamanya.
- **`801961a`** — tombol baru **"📥 Sync Agen"** (self-service) di tab My Worksheet, buat agen yang datanya kosong bisa sync sendiri tanpa nunggu leader. Cek: login sebagai agen, kalau My Worksheet kosong, klik tombol ini, harusnya keisi data assignment-nya.
- **`d565adb` s.d. `636f717`** — OTP login (request + verify) sekarang retry otomatis 4x kalau GAS lambat/timeout (dulu langsung gagal 1x coba). Cek: login normal (request OTP → masukin kode) harusnya tetep mulus kayak biasa, cuma sekarang lebih tahan banting kalau GAS lagi lemot.

## 📌 3. Reminder backlog lama — Auto-Paid Response (LIVE 4 Agu, belum pernah masuk QA)

Bukan bagian dari sesi hari ini, tapi masih kelewat dari handoff-handoff sebelumnya: fitur auto-mark RESPON POSITIF/PAID CRM/PAID SELF ONBOARD pas `paymentStatus` sync jadi 'paid' (deployed 4 Agu, case yang cockpit/force/XL1BIZ di-exclude, override manual di periode sama tetap dihormati) belum pernah di-QA sama sekali. Kalau ada slot, tolong sekalian masuk radar — gue bisa tulis spec detailnya kalau perlu, belum ada dokumen tertulis buat ini.

## Ringkasan prioritas QA

1. **Combo Target (poin 1)** — fokus utama, fitur baru penuh permukaannya.
2. Login/OTP + tombol Sync Agen (poin 2, bagian bawah) — spot-check aja, bukan regresi besar.
3. Auto-Paid Response (poin 3) — kalau ada waktu lebih, backlog lama yang masih nunggak.

Kabarin kalau ada temuan atau begitu semua PASS.

— Rahul
