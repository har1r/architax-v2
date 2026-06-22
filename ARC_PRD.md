# Product Requirements Document (PRD)
## Sistem Pengarsipan dan Pengelolaan Permohonan Layanan Pajak Bumi dan Bangunan — Architax

---

## 1. Document Metadata

| Atribut | Keterangan |
|---|---|
| **Judul Dokumen** | PRD — Sistem Pengarsipan dan Pengelolaan Permohonan Layanan Pajak Bumi dan Bangunan (Architax) |
| **Modul** | Pengarsipan & Pengelolaan Permohonan Layanan Pajak Bumi dan Bangunan |
| **Status** | Final |
| **Sumber** | Cetak Biru (Blueprint) Final & Komprehensif — Tata Usaha Pengarsipan Pendapatan Daerah Sipetra |
| **Tanggal** | 22 Juni 2026 |

---

## 2. Product Overview & Objective

Sistem Sipetra dibangun di atas tiga entitas utama yang saling berhubungan, yaitu **Permohonan**, **Bundle**, dan **Manifest**. Satu Manifest dapat berisi banyak Bundle, dan satu Bundle dapat berisi banyak Permohonan.

*State Machine* pada sistem menggambarkan keadaan aktual objek (bukan aktivitas pengguna), sehingga setiap entitas (Permohonan, Bundle, Manifest) memiliki status independen masing-masing.

Sistem mengakomodasi **6 jenis permohonan layanan pajak daerah**, yaitu:

1. Mutasi Sebagian
2. Mutasi Habis Update
3. Mutasi Habis Reguler
4. Objek Pajak Baru
5. Pembetulan
6. Pengaktifan

---

## 3. User Roles & Permissions

Sistem memetakan 5 role yang masing-masing bertanggung jawab pada satu fase alur kerja (*happy path*):

| No | Role | Fase | Tanggung Jawab Utama (Aksi) |
|---|---|---|---|
| 1 | **PENGINPUT** | Fase 1 — Penerimaan Data | Menerima dokumen permohonan fisik dan menginput data awalnya. Dapat melakukan *Update* data selama status Permohonan `SUBMITTED` dan belum terikat `bundleId`. |
| 2 | **PENELITI** | Fase 2 — Verifikasi & Pengelompokan | Membuat Bundle baru (`DRAFT`), memverifikasi kesesuaian berkas, mencentang permohonan untuk dimasukkan ke Bundle, mencetak Surat Pengantar Bundle (serta Kertas Kerja khusus untuk layanan Mutasi Sebagian), menyusun fisik ke map, dan mengunci Bundle. Berwenang menekan tombol "Minta Revisi" dan menggunakan fitur "Keluarkan dari Bundle". |
| 3 | **PENGARSIP** | Fase 3 — Digitalisasi | Memindai fisik dokumen dari dalam map menjadi Arsip Digital (PDF) dan mengunggahnya. Berwenang menggunakan fitur "Unbundle" jika dokumen fisik cacat saat pemindaian. |
| 4 | **PENGIRIM** | Fase 4 — Logistik | Membuat Manifest (`DRAFT`), memasukkan Bundle ke Manifest, mencetak Surat Pengantar Manifest, menyegel kardus, mengunci Manifest, dan mengunggah Bukti Tanda Terima. Berwenang menggunakan fitur "Revisi Manifest" dan "Laporkan Bundle Hilang". |
| 5 | **PEMANTAU** | Fase 5 — Pemantauan | Mengecek *dashboard* permohonan yang fisiknya telah tiba di pusat, dan menandai permohonan selesai saat produk layanan riil telah terbit. Berwenang menggunakan fitur "Batal Selesai" (*Rollback*). |

---

## 4. Core Entities & State Machine

Sistem terdiri dari 3 entitas utama, masing-masing memiliki *state machine* independen.

### 4.1 Entitas: Permohonan

| Status | Definisi |
|---|---|
| `SUBMITTED` | Berkas dan data awal diterima, menunggu penelitian. |
| `REVISION` | Berkas dikembalikan karena tidak lengkap/tidak valid. Penginput memiliki hak akses untuk mengubah data. |
| `BUNDLED` | Permohonan valid dan dikunci ke dalam Bundle. |
| `ARCHIVED` | Dokumen fisik selesai dipindai menjadi Arsip Digital (PDF). |
| `COMPLETED` | Produk layanan dari pusat telah terbit dan ditandai selesai oleh Pemantau. |
| `REJECTED` | Status terminal. Terjadi secara otomatis oleh sistem bagi permohonan berstatus `REVISION` yang tidak diselesaikan hingga berganti tahun. |

### 4.2 Entitas: Bundle

| Status | Definisi |
|---|---|
| `DRAFT` | Wadah kumpulan data dibuat dan sedang dalam proses seleksi. |
| `LOCKED` | Pengelompokan ditutup, fisik masuk map, dan Surat Pengantar Bundle terbit. |
| `IN_MANIFEST` | Bundle ditarik dan dikunci ke dalam Manifest pengiriman. |

### 4.3 Entitas: Manifest

| Status | Definisi |
|---|---|
| `DRAFT` | Wadah manifest dibuat untuk mengumpulkan bundle siap kirim. |
| `LOCKED` | Pengiriman ditutup, kardus disegel, dan Surat Pengantar Manifest terbit. |
| `SENT` | Bukti Tanda Terima dari Kantor Pusat diunggah sebagai penanda selesainya pengiriman logistik. |

---

## 5. Functional Requirements: Happy Path

### Fase 1: Penerimaan Data
**Role:** `PENGINPUT`

- **Aksi:** Menerima dokumen permohonan fisik dan menginput data awalnya.
- **Transisi Status:** Permohonan menjadi `SUBMITTED`.

### Fase 2: Verifikasi & Pengelompokan
**Role:** `PENELITI`

- **Aksi:**
  - Membuat Bundle baru (`DRAFT`).
  - Memverifikasi kesesuaian berkas dan mencentang permohonan untuk dimasukkan ke dalam Bundle (Permohonan menjadi `BUNDLED`).
- **Dokumen Fisik:**
  - Mencetak Surat Pengantar Bundle.
  - Khusus untuk permohonan **Mutasi Sebagian**, Peneliti juga wajib mencetak **Kertas Kerja** layanan tersebut.
  - Menyusun fisik ke dalam map dan mengunci Bundle.
- **Transisi Status:** Bundle menjadi `LOCKED`.

### Fase 3: Digitalisasi
**Role:** `PENGARSIP`

- **Aksi:** Memindai fisik dokumen dari dalam map menjadi Arsip Digital (PDF) dan mengunggahnya.
- **Transisi Status:** Permohonan menjadi `ARCHIVED`.

### Fase 4: Logistik
**Role:** `PENGIRIM`

- **Aksi:**
  - Membuat Manifest (`DRAFT`).
  - Memasukkan Bundle ke Manifest (Bundle menjadi `IN_MANIFEST`).
  - Mencetak Surat Pengantar Manifest, menyegel kardus, lalu mengunci Manifest (`LOCKED`).
  - Setelah fisik tiba, mengunggah Bukti Tanda Terima.
- **Transisi Status:** Manifest menjadi `SENT` (Terminal logistik).

### Fase 5: Pemantauan
**Role:** `PEMANTAU`

- **Aksi:**
  - Mengecek *dashboard* permohonan yang fisiknya telah tiba di pusat.
  - Saat produk layanan riil telah terbit, menandai permohonan tersebut di sistem.
- **Transisi Status (Akhir):** Permohonan menjadi `COMPLETED`.

---

## 6. Exception Handling & Edge Cases

### 6.1 Fase 1 (`PENGINPUT`)

**Anomali: Salah Input Data**
- Dapat melakukan *Update* data selama status permohonan `SUBMITTED` dan belum terikat `bundleId`.
- *Hard Delete* dilarang mutlak.

### 6.2 Fase 2 (`PENELITI`)

**Anomali: Berkas Tidak Valid / Kurang Syarat**
- Peneliti menekan tombol "Minta Revisi".
- Status Permohonan turun menjadi `REVISION`.
- Berkas fisik dikembalikan ke `PENGINPUT` untuk diperbaiki atau menunggu kelengkapan dari wajib pajak.

**Anomali: Kapasitas Map Penuh**
- Selama Bundle `DRAFT`, Peneliti memakai fitur "Keluarkan dari Bundle" (status kembali `SUBMITTED`) agar berkas tidak tercetak di Surat Pengantar/Kertas Kerja dan bisa diikutkan di map berikutnya.

**Anomali: Race Condition (Tabrakan)**
- Dicegah secara teknis via *database transaction* di sisi *server*.

### 6.3 Fase 3 (`PENGARSIP`)

**Anomali: Dokumen Fisik Cacat Saat Pemindaian**
- Pengarsip menggunakan fitur "Unbundle".
- Relasi Bundle diputus, status permohonan turun ke `SUBMITTED`.
- Fisik dikembalikan ke `PENELITI` untuk dikaji ulang (kemudian diturunkan ke `REVISION` jika perlu perbaikan wajib pajak).

### 6.4 Fase 4 (`PENGIRIM`)

**Anomali: Map Tertinggal di Laci**
- Fitur "Revisi Manifest" digunakan untuk membuka gembok Manifest kembali ke `DRAFT`, melepas map yang tertinggal dari daftar elektronik, lalu dikunci ulang.

**Anomali: Map Hilang Saat Transit**
- Pengirim tetap menyelesaikan Manifest ke `SENT` (karena kargo utama tiba).
- Lalu menggunakan fitur "Laporkan Bundle Hilang" untuk memutus `manifestId` dari map yang hilang.
- Status map tersebut kembali `LOCKED` untuk dilacak.

### 6.5 Fase 5 (`PEMANTAU`)

**Anomali: Human Error (Salah Klik Selesai)**
- Fitur "Batal Selesai" (*Rollback*) digunakan untuk mengubah status `COMPLETED` kembali ke `ARCHIVED`.
- Nilai perubahan sebelum dan sesudah wajib terekam ke *Audit Log*.

---

## 7. System Automation & Non-Functional Requirements

### 7.1 Otomatisasi Sistem (Backend)

**Aturan Tutup Buku Tahunan:**
- Sistem menjalankan *Scheduled Job/Cron Job* setiap akhir tahun anggaran.
- Seluruh data berstatus `REVISION` yang tidak di-`UPDATE` oleh Penginput hingga pergantian tahun akan dipaksa berubah menjadi `REJECTED`.
- Tombol *update* dikunci secara permanen, mengharuskan wajib pajak melakukan registrasi permohonan baru.

### 7.2 Larangan Hard Delete

- *Hard Delete* dilarang mutlak pada data Permohonan (berlaku pada Fase 1, terkait penanganan kesalahan input data oleh Penginput).

### 7.3 Audit Log

- Pada aksi "Batal Selesai" (*Rollback*) di Fase 5, nilai perubahan sebelum dan sesudah status wajib terekam ke *Audit Log*.

### 7.4 Database Transaction

- Penanganan *Race Condition* (Tabrakan) pada Fase 2 dicegah secara teknis via *database transaction* di sisi *server*.

---

## 8. Glossary / Kamus Istilah

| Istilah | Penjelasan (berdasarkan Blueprint) |
|---|---|
| **Permohonan** | Salah satu dari tiga entitas utama sistem; dapat berjumlah banyak di dalam satu Bundle. |
| **Bundle** | Salah satu dari tiga entitas utama sistem; wadah kumpulan Permohonan; dapat berjumlah banyak di dalam satu Manifest. |
| **Manifest** | Salah satu dari tiga entitas utama sistem; wadah pengiriman yang dapat berisi banyak Bundle. |
| **State Machine** | Mekanisme yang menggambarkan keadaan aktual objek (bukan aktivitas pengguna), di mana setiap entitas memiliki status independen. |
| **Mutasi Sebagian** | Salah satu dari 6 jenis permohonan layanan pajak daerah; memerlukan pencetakan Kertas Kerja khusus pada Fase 2. |
| **Mutasi Habis Update** | Salah satu dari 6 jenis permohonan layanan pajak daerah. |
| **Mutasi Habis Reguler** | Salah satu dari 6 jenis permohonan layanan pajak daerah. |
| **Objek Pajak Baru** | Salah satu dari 6 jenis permohonan layanan pajak daerah. |
| **Pembetulan** | Salah satu dari 6 jenis permohonan layanan pajak daerah. |
| **Pengaktifan** | Salah satu dari 6 jenis permohonan layanan pajak daerah. |
| **Kertas Kerja** | Dokumen fisik yang wajib dicetak oleh Peneliti khusus untuk permohonan jenis Mutasi Sebagian pada Fase 2. |
| **Surat Pengantar Bundle** | Dokumen fisik yang dicetak oleh Peneliti saat mengelompokkan Permohonan ke dalam Bundle (Fase 2). |
| **Surat Pengantar Manifest** | Dokumen fisik yang dicetak oleh Pengirim saat mengunci Manifest (Fase 4). |
| **Arsip Digital (PDF)** | Hasil pemindaian dokumen fisik oleh Pengarsip pada Fase 3, yang menjadikan Permohonan berstatus `ARCHIVED`. |
| **Bukti Tanda Terima** | Dokumen yang diunggah oleh Pengirim setelah fisik tiba di Kantor Pusat, menandai Manifest menjadi `SENT`. |
| **bundleId** | Atribut pengikat pada Permohonan; jika belum terikat, Permohonan masih dapat di-*update* oleh Penginput selama status `SUBMITTED`. |
| **manifestId** | Atribut pengikat pada Bundle terhadap Manifest; diputus pada fitur "Laporkan Bundle Hilang". |
| **Hard Delete** | Tindakan penghapusan permanen data; dilarang mutlak dalam sistem (Fase 1). |
| **Keluarkan dari Bundle** | Fitur bagi Peneliti untuk mengembalikan status Permohonan ke `SUBMITTED` saat Bundle masih `DRAFT`. |
| **Minta Revisi** | Fitur bagi Peneliti untuk menurunkan status Permohonan menjadi `REVISION`. |
| **Unbundle** | Fitur bagi Pengarsip untuk memutus relasi Bundle dan menurunkan status Permohonan ke `SUBMITTED` akibat dokumen fisik cacat saat pemindaian. |
| **Revisi Manifest** | Fitur bagi Pengirim untuk membuka kembali Manifest dari `LOCKED` ke `DRAFT` guna melepas Bundle yang tertinggal. |
| **Laporkan Bundle Hilang** | Fitur bagi Pengirim untuk memutus `manifestId` dari Bundle yang hilang saat transit, mengembalikan status Bundle ke `LOCKED`. |
| **Batal Selesai (Rollback)** | Fitur bagi Pemantau untuk mengubah status Permohonan dari `COMPLETED` kembali ke `ARCHIVED`, dengan nilai perubahan terekam di Audit Log. |
| **Scheduled Job / Cron Job** | Mekanisme otomatis backend yang dijalankan setiap akhir tahun anggaran untuk menerapkan Aturan Tutup Buku Tahunan. |
| **Aturan Tutup Buku Tahunan** | Aturan sistem yang memaksa seluruh Permohonan berstatus `REVISION` yang tidak di-*update* hingga pergantian tahun menjadi `REJECTED` secara otomatis. |
| **Race Condition (Tabrakan)** | Kondisi anomali teknis pada Fase 2 yang dicegah melalui *database transaction* di sisi server. |
| **Audit Log** | Catatan sistem yang merekam nilai perubahan sebelum dan sesudah pada aksi "Batal Selesai" (Rollback). |
