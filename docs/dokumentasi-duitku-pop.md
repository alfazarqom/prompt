# Dokumentasi Duitku POP — Sumber Resmi dan Checklist

**Tanggal pemeriksaan halaman:** 22 Oktober 2026  
**Sumber utama:** https://docs.duitku.com/pop/id/

> Dokumen ini adalah indeks dan ringkasan kerja, bukan pengganti dokumentasi provider. Buka ulang sumber resmi sebelum coding karena kontrak, parameter, channel, dan perilaku callback dapat berubah.

## 1. Tautan resmi

| Topik | URL | Tujuan |
|---|---|---|
| Duitku POP Indonesia | https://docs.duitku.com/pop/id/ | Sumber utama Create Invoice, signature, callback, frontend JS, redirect, error, dan demo |
| Duitku POP English | https://docs.duitku.com/pop/en/ | Referensi bahasa Inggris/pembanding |
| Duitku API Indonesia | https://docs.duitku.com/api/id/ | Hanya bila produk/API yang dipilih memang sesuai; jangan campur kontrak dengan POP |
| FAQ Duitku | https://www.duitku.com/frequently-asked-questions/ | Pertanyaan umum dan informasi merchant |
| Sandbox/demo | Ikuti tautan sandbox yang saat ini ada pada dokumentasi POP dan Merchant Portal | Pengujian tanpa transaksi produksi |
| Create Invoice sandbox | https://api-sandbox.duitku.com/api/merchant/createInvoice | Endpoint yang tercantum pada dokumentasi POP saat diperiksa |
| Create Invoice production | https://api-prod.duitku.com/api/merchant/createInvoice | Endpoint yang tercantum pada dokumentasi POP saat diperiksa |

## 2. Alur POP secara ringkas

1. Pelanggan checkout di toko.
2. Backend memvalidasi pesanan dan menghitung total dari sumber tepercaya.
3. Backend membuat request Create Invoice dengan signature sesuai kontrak POP.
4. Duitku merespons reference dan informasi pembayaran, termasuk URL pembayaran sesuai respons.
5. Frontend menggunakan mode yang dipilih: redirect melalui `paymentUrl` atau Duitku JS/popup.
6. Duitku mengirim HTTP callback ke endpoint backend merchant.
7. Backend memverifikasi callback, mencocokkan transaksi, dan menerapkan transisi status secara aman.
8. Halaman return meminta status dari backend; redirect browser bukan bukti pembayaran.

## 3. Create Invoice — referensi kontrak

Dokumentasi POP saat pemeriksaan mencantumkan:
- Method: HTTP POST.
- Content type: `application/json`.
- Sandbox: `https://api-sandbox.duitku.com/api/merchant/createInvoice`.
- Production: `https://api-prod.duitku.com/api/merchant/createInvoice`.
- Header: `Content-Type`, `x-duitku-timestamp`, `x-duitku-signature`, `x-duitku-merchantcode`.

Formula request signature yang ditampilkan:

```text
stringToSign = merchantCode + timestamp
signature = HMAC_SHA256(stringToSign, apiKey)
```

Timestamp didokumentasikan dalam milidetik. Sebelum implementasi, verifikasi formula, key, encoding, format output, dan headers pada sumber resmi.

Parameter contoh mencakup `paymentAmount`, `merchantOrderId`, `productDetails`, `email`, `callbackUrl`, `returnUrl`, serta parameter opsional/kondisional seperti `paymentMethod`, `phoneNumber`, `itemDetails`, `customerDetail`, dan `expiryPeriod`. Jangan menganggap daftar contoh sebagai daftar universal field wajib; cek tabel parameter terbaru dan metode pembayaran yang digunakan.

Dokumentasi juga menyatakan:
- `merchantOrderId` harus unik untuk transaksi baru.
- `paymentAmount` harus cocok dengan rincian item menurut kontrak.
- `additionalParam`, jika dipakai, memiliki aturan encoding.
- Fitur pembayaran khusus dapat mewajibkan field tambahan.

## 4. Callback — referensi kontrak

Dokumentasi POP saat diperiksa menjelaskan HTTP POST dengan content type `application/x-www-form-urlencoded`. Contoh field meliputi `merchantCode`, `amount`, `merchantOrderId`, `resultCode`, `reference`, dan `signature`; field tambahan bergantung pada transaksi/metode.

Formula callback signature yang ditampilkan:

```text
stringToSign = merchantCode + amount + merchantOrderId
signature = HMAC_SHA256(stringToSign, merchantKey)
```

**Jangan menyamakan formula callback dengan formula request Create Invoice.** Verifikasi nama key, urutan field, nilai string, encoding, dan acknowledgement sesuai halaman resmi terkini.

Checklist backend:
- [ ] Parse body berdasarkan content type resmi.
- [ ] Validasi field wajib dan format.
- [ ] Hitung ulang dan verifikasi signature di backend.
- [ ] Cocokkan merchant, order ID, amount, reference, environment, dan status lokal.
- [ ] Terapkan transisi status hanya jika valid.
- [ ] Buat pemrosesan idempoten terhadap retry/duplikasi dan aman terhadap concurrency.
- [ ] Jangan decrement stok atau menjalankan fulfillment lebih dari sekali.
- [ ] Jangan log secret, signature penuh, atau data pribadi yang tidak diperlukan.
- [ ] Berikan acknowledgement sesuai kontrak setelah pemrosesan yang benar.
- [ ] Rencanakan pemulihan/reconciliation untuk callback gagal atau tertunda.

Dokumentasi POP menyebut callback URL harus dapat diakses publik melalui port 80/443 dan callback memerlukan HTTP 200 acknowledgement. Perilaku dan batas retry harus dikonfirmasi ulang pada dokumentasi terkini.

## 5. Redirect dan Duitku JS

Duitku POP mendokumentasikan opsi Duitku JS/popup dan window redirection menggunakan `paymentUrl`.

- `returnUrl` mengembalikan pelanggan ke situs merchant.
- Query string/redirect dapat dimodifikasi pengguna.
- JS callback hanya untuk UX/notifikasi.
- Jangan memperbarui status payment/order hanya dari redirect atau callback JavaScript.
- Frontend harus meminta status terkini dari backend.

## 6. Persyaratan resmi vs rekomendasi keamanan

### Kontrak resmi — verifikasi pada dokumentasi terkini
- Endpoint, method, headers, content type, dan field sesuai mode POP.
- Formula signature request dan callback yang tepat.
- Callback URL dan acknowledgement sesuai kontrak.
- Penanganan response/status sesuai dokumentasi.
- Redirect dan JS callback tidak boleh menjadi sumber otoritatif status database.

### Rekomendasi engineering/security
- Simpan credential hanya di backend/secret manager.
- Hitung ulang total dan validasi produk/stok di backend.
- Gunakan unique constraint/idempotency untuk order/reference dan callback processing.
- Pisahkan status order, payment, dan shipment.
- Gunakan transaksi atomik atau pola durable untuk payment + stok.
- Audit RLS, grants, RPC, JWT configuration, dan Edge Functions bila memakai Supabase.
- Pisahkan sandbox/production dan sanitasi log.
- Gunakan reconciliation hanya melalui kemampuan resmi yang diverifikasi.

## 7. Checklist pengujian sandbox

- [ ] Invoice valid dibuat dan reference tersimpan.
- [ ] Secret hilang atau signature request salah gagal dengan aman.
- [ ] HTTP 200 dengan status bisnis gagal tidak dianggap sukses.
- [ ] Callback valid dan transaksi cocok diproses tepat sekali.
- [ ] Signature invalid atau amount/merchant/order/reference mismatch tidak mengubah status.
- [ ] Callback duplikat dan callback bersamaan tidak menimbulkan efek ganda.
- [ ] Callback terlambat/retry/expiry/database failure memiliki jalur pemulihan.
- [ ] Manipulasi return URL/JS callback tidak mengubah status database.
- [ ] Stok tidak oversell dan tidak dikurangi dua kali.
- [ ] Tidak ada secret pada bundle frontend, log, atau commit.
- [ ] Skenario yang tidak tersedia di sandbox diuji memakai mock/fixture terisolasi.

Catat expected result, actual result, bukti, tanggal, environment, dan skenario yang dilewati. Jangan menyatakan test lulus tanpa menjalankannya.

## 8. Referensi

- https://docs.duitku.com/pop/id/
- https://docs.duitku.com/pop/en/
- https://docs.duitku.com/api/id/
- https://www.duitku.com/frequently-asked-questions/

**Catatan verifikasi:** endpoint, formula, dan perilaku di atas merupakan ringkasan halaman POP yang diperiksa pada tanggal di awal dokumen. Verifikasi ulang sumber resmi sebelum membuat atau mengubah kode.
