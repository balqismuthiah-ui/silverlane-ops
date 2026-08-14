Subject: Re-test — fix duplikat Target + urutan acak (commit `27c2e48`, live)

Ryry,

Langsung nindaklanjutin 2 temuan kamu di QA kemarin (duplikat `upsell_attach` + urutan acak-acak, akar `busy_lock_timeout` false negative) — udah di-fix & live di `https://clm.ryanza.my.id/churn.html`.

## Apa yang berubah

1. **`Targets.add`/`Targets.delete`** — kalau `upsertTarget`/`deleteTarget` gagal/timeout, sekarang client fresh-check dulu ke server (`_gasGet('targets')`) sebelum nampilin toast. Kalau ternyata datanya UDAH kesimpen/kehapus di server (false negative persis kasus yang kamu temuin), toast-nya jujur bilang sukses — bukan minta retry.
2. **`reorderTarget`** — rewrite total. Dulu nomorin ulang SEMUA target segrup periode pakai cache lokal (bisa basi). Sekarang fresh-fetch dari server dulu, terus CUMA swap 2 target yang ketuker (bukan renumber semua) — dari N write jadi 2 write tetap per-geser.

## Cara re-test

1. **Reproduksi false-negative kalau bisa**: kalau GAS lagi ngelag lagi hari ini (kondisi yang kemarin bikin `busy_lock_timeout`), coba add/delete target pas kondisi itu — perhatiin toast-nya, harusnya sekarang bisa bilang "server sempat lambat balas, datanya udah masuk" (bukan langsung "gagal") kalau ternyata di server udah sukses. Bandingin behavior-nya sama insiden kemarin.
2. **Reorder normal**: geser ▲/▼ beberapa target di 1 bulan, cek urutan-nya kesimpen bener (bandingin ke Sheet TARGETS langsung — sortOrder-nya harus jadi rapi `0,1,2,...` di grup itu, bukan random kayak yang ketemu kemarin: `0,1,2,7,3,4,17,5,8,11,...`).
3. **Reorder concurrent (kalau bisa disimulasiin)**: 2 browser beda leader, masing-masing geser target berbeda di bulan yang sama HAMPIR BARENGAN (tanpa refresh manual di antaranya) — pastiin urutan akhir konsisten di kedua browser setelah refresh, nggak ada yang ketiban ulang.
4. **Duplikat**: pastiin nggak ada cara baru buat bikin duplikat via add — coba add target yang sama beberapa kali cepat berturut-turut (edge case), lihat hasilnya di Sheet.

Kabarin hasilnya — kalau semua PASS, ini nutup temuan `busy_lock_timeout`/reorder dari QA kemarin.

— Rahul
