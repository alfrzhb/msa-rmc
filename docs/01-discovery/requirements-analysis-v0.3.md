# RMC MSA Analysis Tool
## Requirements Analysis v0.3 — DRAFT

**Tanggal:** 22 Oktober 2026  
**Perusahaan:** PT Ratama Mitra Kualitas  
**Target:** Satu perusahaan klien  
**Status:** DRAFT — Not Frozen  
**Repository:** [alfrzhb/msa-rmc](https://github.com/alfrzhb/msa-rmc)

> Dokumen ini adalah hasil review kebutuhan awal, bukan spesifikasi final yang telah disetujui secara menyeluruh. Persetujuan parsial dibedakan dari keputusan FROZEN. Tidak ada metode, rumus, atau kriteria MSA yang ditetapkan sebagai final.

---

### 1. Business Objective

Membangun aplikasi Measurement System Analysis (MSA) berbasis web yang sederhana, mudah digunakan, dan mampu menggantikan proses analisis manual yang mengandalkan spreadsheet serta penggunaan software statistik yang terlalu kompleks untuk kebutuhan operasional klien.

Aplikasi harus mendukung input pengukuran, kalkulasi, interpretasi hasil, histori studi, keterlacakan aktivitas, dan pembuatan laporan PDF.

### 2. MVP Scope

**Termasuk MVP:**
- Login individual karyawan.
- Dashboard sederhana.
- Pengelolaan studi MSA Variable dan Attribute.
- Input data manual dan paste dari Excel.
- Kalkulasi statistik otomatis.
- Penyimpanan Draft dan hasil Final.
- Histori serta pencarian studi.
- Audit trail.
- Export dan print PDF.

**Di luar MVP:**
- Modul SPC.
- Multi-client management.
- Integrasi langsung dengan Minitab.
- Advanced analytics yang belum diminta klien.
- Multi-level approval workflow.
- Import file XLSX secara penuh.

Arsitektur harus memungkinkan perluasan fitur tanpa mengharuskan implementasi multi-client pada MVP.

### 3. User Roles & Permissions

**Admin**
- Membuat, mengaktifkan, dan menonaktifkan akun pengguna.
- Menetapkan role dan hak akses modul.
- Mengelola administrasi aplikasi.

**Supervisor**
- Membuat dan mengelola studi MSA.
- Melihat seluruh studi yang berada dalam cakupan kewenangannya.
- Mengedit Draft milik operator lain.
- Membuat revisi terhadap studi Final.
- Menghapus secara logis (*soft delete*) studi MSA.
- Melihat audit trail dan mengekspor laporan.

**Operator**
- Membuat studi MSA baru.
- Menginput data dan menjalankan kalkulasi.
- Menyimpan dan melanjutkan Draft.
- Mengedit studi miliknya sendiri selama belum Final.
- Melihat histori sesuai hak akses.
- Melihat hasil dan mengekspor laporan.

**Access rules:**
- Semua akses wajib diautentikasi dan diotorisasi di backend.
- Supervisor dapat mengubah studi milik operator.
- Operator tidak dapat mengubah studi milik pengguna lain.
- Pembuatan akun melalui Admin, tanpa registrasi publik.
- Hak melihat seluruh histori masih perlu dikonfirmasi.

### 4. Main Application Workflow

1. User login menggunakan akun individual.
2. Sistem membuka Dashboard.
3. User membuka menu MSA.
4. Sistem menampilkan daftar studi yang tersimpan.
5. User menekan Create New MSA.
6. User memilih Variable MSA atau Attribute MSA.
7. User mengisi Study Setup.
8. User memasukkan data pengukuran atau keputusan.
9. User dapat menyimpan Draft.
10. Sistem memvalidasi input dan menjalankan kalkulasi.
11. Sistem menampilkan hasil serta interpretasi.
12. User mereview dan melakukan Finalize Study.
13. Studi Final tersimpan dalam History.
14. User dapat mengunduh atau mencetak PDF.

### 5. Study Lifecycle

Status utama yang diusulkan:

- **Draft:** studi dibuat dan dapat dikerjakan.
- **Finalized:** data dan perhitungan telah diselesaikan dan dikunci.
- **Revision:** perubahan terhadap studi Final dilakukan melalui versi revisi yang terhubung ke hasil sebelumnya.

Aturan:
- Studi Draft dapat dihitung tanpa otomatis menjadi Final.
- Studi Final tidak dapat diubah langsung.
- Supervisor mengelola proses revisi.
- Perubahan input membatalkan validitas hasil kalkulasi sebelumnya sampai dihitung ulang.
- Versi final terdahulu tetap dipertahankan untuk audit.
- Soft delete tidak langsung menghilangkan rekaman studi.

### 6. Variable MSA — Proposed Design

**Kandidat awal:** Crossed Gage R&R menggunakan metode Average & Range.

Konfigurasi referensi awal:
- 10 parts.
- 3 appraisers.
- 3 trials.
- Data pengukuran numerik.

Kebutuhan:
- Identitas studi, part, karakteristik, alat ukur, dan satuan.
- Identitas appraiser.
- Input pengukuran berulang.
- Kalkulasi Repeatability, Reproducibility, GRR, dan Part Variation.
- Ringkasan persentase variasi, NDC, dan interpretasi.
- Validasi kelengkapan data serta kondisi matematis khusus.

**Pending:**
- Konfirmasi kecocokan desain crossed dengan proses klien.
- Apakah jumlah part, appraiser, dan trial harus dapat diubah.
- Rumus, konstanta, pembulatan, dan metode persentase variasi.
- Standar judgement.
- Penggunaan ANOVA pada fase berikutnya.

### 7. Attribute MSA — Proposed Design

**Kandidat awal:** Attribute Agreement Analysis menggunakan kategori OK/NG dan Known Reference.

Konfigurasi referensi awal:
- 50 samples.
- 3 appraisers.
- 2 trials.
- Known Reference untuk setiap sampel.

Output yang diusulkan:
- Kappa setiap appraiser dibandingkan dengan reference.
- Persentase kesesuaian terhadap reference.
- Konsistensi keputusan antarpengulangan.
- Interpretasi hasil berdasarkan kriteria tervalidasi.

Agreement antarapraiser dapat dipertimbangkan sebagai indikator tambahan tanpa membuat alur input baru.

**Pending:**
- Konfirmasi jumlah sampel dan pengulangan.
- Konfirmasi apakah reference selalu tersedia.
- Definisi perhitungan Kappa dan metrik pendamping.
- Standar judgement Attribute.
- Aturan untuk dataset tidak lengkap atau tidak valid.

### 8. Input & Validation

- Input dilakukan melalui tabel interaktif.
- Mendukung navigasi keyboard.
- Mendukung paste data dari Excel sesuai struktur tabel.
- Validasi dilakukan sebelum kalkulasi.
- Sistem memberikan pesan kesalahan yang spesifik.
- Data Draft tidak harus lengkap.
- Kalkulasi hanya dapat dilakukan jika semua persyaratan metode terpenuhi.
- Identitas appraiser dibedakan dari identitas akun yang memasukkan data.

### 9. Results & Reporting

Hasil analisis harus mengutamakan keterbacaan.

Tampilan utama:
- Ringkasan angka statistik.
- Status penilaian.
- Interpretasi singkat.
- Informasi metode yang digunakan.
- Detail kalkulasi bila diperlukan.

Laporan PDF mencakup:
- Identitas perusahaan dan studi.
- Tanggal dan identitas pembuat.
- Jenis dan versi metode.
- Informasi alat serta appraiser.
- Ringkasan input pengukuran.
- Hasil statistik dan kesimpulan.
- Identitas revisi laporan.

Laporan Final harus dapat diunduh kembali tanpa mengubah isi historisnya.

### 10. Dashboard & Study History

Dashboard menyediakan:
- Create New MSA.
- Total studi.
- Jumlah Draft dan Finalized.
- Daftar studi terbaru.

Study History menyediakan:
- Pencarian berdasarkan ID dan nama studi.
- Filter tipe MSA, status, dan tanggal.
- Informasi pembuat dan waktu perubahan.
- Akses detail serta laporan studi.

### 11. Non-Functional Requirements

- Web-based.
- Desktop-first dan responsive.
- Online-only untuk MVP.
- Akses berdasarkan role dan kepemilikan studi.
- Password disimpan menggunakan mekanisme hashing yang aman.
- Audit trail untuk tindakan penting.
- Backup database terjadwal dan prosedur restore yang teruji.
- Perhitungan deterministik dan dapat diuji.
- Calculation engine dipisahkan dari UI.
- Metode dan versinya tersimpan bersama hasil studi.
- Tidak menyimpan data sensitif klien dalam repository publik.

### 12. Verification Requirements

Sebelum release:
- Hasil perhitungan diverifikasi terhadap dataset referensi yang telah diaudit.
- Unit test untuk rumus, threshold, dan edge cases.
- Permission testing untuk ketiga role.
- Pengujian Draft, Finalize, Revision, dan soft delete.
- Validasi tampilan serta konsistensi laporan PDF.
- User Acceptance Testing bersama perwakilan klien.

### 13. Decision Register

**Disetujui melalui review:**
- Tiga role: Admin, Supervisor, Operator.
- Operator hanya mengedit studi miliknya.
- Supervisor dapat mengelola studi operator.
- Input manual dan paste Excel untuk MVP.
- Studi Final diperbarui melalui mekanisme revisi.

**Masih berupa rekomendasi:**
- Rincian permission Admin.
- Status dan mekanisme Finalize.
- Konfigurasi default jumlah pengukuran.
- Format dashboard dan laporan.
- Pilihan hosting serta arsitektur teknis.

**Memerlukan pembahasan khusus:**
- Metode MSA Variable.
- Metode dan indikator MSA Attribute.
- Calculation Specification dan judgement criteria.
- Fleksibilitas konfigurasi studi.

### 14. Document Control

**Current version:** v0.3  
**Status:** DRAFT  
**Approval:** Pending  
**GitHub commit:** DRAFT publication (not approval)

Dokumen tidak boleh dinyatakan FROZEN sebelum direview dan disetujui secara eksplisit.

---

**End of Requirements Analysis v0.3 — DRAFT**
