# Prompt Toko Online — React + Vite + Duitku POP

Repository ini menyimpan template prompt terstruktur untuk merencanakan, mengaudit, membangun, menguji, dan memelihara aplikasi toko online berbasis React + Vite dengan integrasi pembayaran Duitku POP.

## Isi repository

- [Master Prompt Toko Online](prompts/master-prompt-toko-online-react-vite-duitku-pop.md) — prompt utama berbahasa Indonesia dengan variabel proyek, workflow change-control, arsitektur e-commerce, keamanan, integrasi Duitku POP, pengujian, dan format laporan.
- [Dokumentasi dan sumber resmi Duitku](docs/dokumentasi-duitku-pop.md) — indeks tautan resmi, ringkasan alur POP, prinsip signature/callback, dan checklist verifikasi.

## Cara menggunakan

1. Salin file master prompt ke sesi AI yang memiliki akses ke repository proyek.
2. Isi variabel seperti `{{PROJECT_NAME}}`, `{{GITHUB_REPOSITORY}}`, `{{WORK_MODE}}`, `{{TASK_GOAL}}`, dan `{{CHANGE_BOUNDARIES}}`.
3. Mulai dengan mode audit read-only bila mengerjakan repository yang sudah ada.
4. Verifikasi kontrak Duitku pada dokumentasi resmi terkini sebelum coding.
5. Minta persetujuan eksplisit untuk perubahan yang memerlukan otorisasi. Izin coding tidak otomatis mengizinkan migration, secrets, commit/push, merge, atau deployment.

## Catatan

Dokumen ini adalah template engineering, bukan jaminan bahwa aplikasi siap production atau bahwa suatu integrasi merchant telah diverifikasi. Persyaratan API dan metode pembayaran dapat berubah; sumber resmi Duitku adalah rujukan utama.
