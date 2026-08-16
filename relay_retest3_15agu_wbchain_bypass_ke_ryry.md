Subject: Re-test final — akar antrean `_wbChain` udah di-fix (commit `712feee`)

Ryry,

Makasih temuan #4 di re-test kedua kamu — itu ternyata akar yang jauh lebih penting dari yang kita duga. Udah di-fix & live di `https://clm.ryanza.my.id/churn.html`.

## Apa yang berubah

`_wbChain` (antrean global yang dipakai SEMUA write, termasuk background bulk-sync ribuan item) awalnya dibikin buat nutup append-race client-side lama. Tapi Target endpoint (`upsertTarget`/`deleteTarget`) sekarang udah punya kunci sendiri di server (`withWriteLock_`/LockService, ke-verify langsung di source Code.gs) — jadi nggak butuh ikut antre di situ lagi.

**Fix**: Target write (add/delete/reorder) sekarang lewat jalur baru (`_gasPostUnchained`) yang skip antrean global itu sepenuhnya — tetap ada retry-with-backoff yang sama, cuma nggak nunggu di belakang ribuan item background sync lain. Endpoint lain (upsell/respons/GC) TIDAK disentuh, tetap lewat antrean lama seperti biasa (belum diverifikasi aman buat di-skip juga).

Verified via mock test: simulasi antrean disumpel 1 write background lambat (2 detik) — Target write lewat jalur baru selesai instan (0ms), sementara lewat jalur lama nunggu 4+ detik di belakangnya.

## Soal 3 target test yang nyangkut kemarin

`yof6kg30`/`00lll569`/`hi7rc8me` — coba **reload halaman dulu** sebelum coba hapus lagi. `_wbChain` itu murni state di memory browser (bukan tersimpan di localStorage/server), jadi reload otomatis reset antreannya total. Percobaan delete berikutnya (setelah fix ini live) juga bakal lewat jalur baru yang nggak ikut nyangkut di situ.

## Cara re-test

1. **Reorder di kondisi rame** (kalau masih bisa direproduksi ada banyak background sync jalan) — pastiin sekarang reorder selesai cepat/dapet toast, nggak lagi diam total berbenit-menit.
2. **sortOrder=null end-to-end** — kemarin keblokir temuan #4 sebelum sempat full-verified, sekarang harusnya nggak keblokir lagi.
3. **Cleanup 3 target test** — reload dulu, baru coba hapus, konfirmasi bersih.
4. Spot-check sekalian: add/edit/delete target biasa, pastiin tetap responsif meskipun ada aktivitas background sync jalan bareng.

Kabarin hasilnya — kalau semua PASS, ini beneran nutup total rangkaian temuan `busy_lock_timeout`/reorder/antrean dari kemarin-kemarin.

— Rahul
