# Technical Specification — Architax
## Addendum Teknis dari `ARC_PRD.md`

> **Dokumen ini adalah turunan teknis murni dari PRD final (`ARC_PRD.md`).**
> Tidak ada fitur bisnis, role, jenis layanan, atau edge case baru yang ditambahkan. Setiap keputusan teknis di bawah ini adalah penerjemahan langsung dari kebutuhan bisnis yang sudah didefinisikan di PRD. Jika ada ambiguitas teknis murni (bukan bisnis), dipilih solusi paling sederhana dan sesuai best practice tech stack yang ditentukan, dengan justifikasi singkat di tiap bab.

| Atribut | Keterangan |
|---|---|
| **Dokumen Sumber** | `ARC_PRD.md` (Final) |
| **Status** | Final |
| **Konsumen** | AI Coding Assistant (Cursor / Claude Code) & Tim Engineering |
| **Tanggal** | 22 Juni 2026 |

---

## 1. Tech Stack & Architecture Overview

### 1.1 Tech Stack Tetap

| Layer | Teknologi |
|---|---|
| Frontend & Backend | Next.js 14+ (App Router) |
| Bahasa | TypeScript (strict mode) |
| Styling | Tailwind CSS + CSS biasa untuk komponen kompleks |
| Database | MongoDB |
| ORM | Prisma |
| API | REST API via Route Handlers (App Router) |
| Auth | NextAuth.js |
| File Storage | Vercel Blob |
| State Management | React Server Components + Client Components |
| Validation | Zod |
| Cron Job | Vercel Cron |

### 1.2 Arsitektur: Server Components vs Client Components

Karena domain Architax bersifat *form-heavy* dan *state-machine-driven* (transisi status Permohonan/Bundle/Manifest yang ketat), pembagian berikut digunakan:

- **Server Components (default)** — digunakan untuk seluruh halaman *read* (dashboard, listing Permohonan/Bundle/Manifest, detail entitas). Data diambil langsung di server lewat Prisma Client tanpa round-trip API tambahan, mengurangi *waterfall* dan menyembunyikan logika query dari klien.
- **Client Components (`"use client"`)** — hanya untuk komponen yang butuh interaktivitas: form input data (Fase 1), checkbox seleksi Permohonan ke Bundle (Fase 2), tombol aksi state-transition ("Minta Revisi", "Unbundle", "Batal Selesai", dsb.), dan komponen upload file.
- **Server Actions** — digunakan untuk seluruh mutasi data (create/update transisi status) alih-alih membuat Route Handler terpisah untuk setiap form, karena Server Actions terintegrasi langsung dengan `useFormState`/`useActionState` untuk validasi Zod dan idempotency (lihat Bab 5).
- **Route Handlers (REST API)** — dipakai khusus untuk: (a) endpoint yang dipanggil Cron Job (Bab 13), (b) endpoint upload file binary (Bab 6), dan (c) endpoint yang perlu dikonsumsi non-browser client di masa depan.

**Alasan:** PRD tidak menyebutkan kebutuhan API publik/eksternal selain proses internal sistem, sehingga Server Actions adalah pilihan paling sederhana dan aman untuk mutasi UI-driven, sementara Route Handlers dicadangkan untuk kasus yang murni teknis (cron, file upload).

### 1.3 Struktur Folder

```
architax/
├── app/
│   ├── (auth)/
│   │   └── login/
│   │       └── page.tsx
│   ├── (dashboard)/
│   │   ├── permohonan/
│   │   │   ├── page.tsx                  # List Permohonan (PENGINPUT)
│   │   │   ├── permohonan-baru/
│   │   │   │   └── page.tsx              # Form input Fase 1
│   │   │   └── [id]/
│   │   │       └── page.tsx              # Detail + update (status SUBMITTED/REVISION)
│   │   ├── bundle/
│   │   │   ├── page.tsx                  # List Bundle (PENELITI)
│   │   │   ├── bundle-baru/
│   │   │   │   └── page.tsx
│   │   │   └── [id]/
│   │   │       └── page.tsx              # Detail Bundle, seleksi Permohonan, lock
│   │   ├── arsip/
│   │   │   └── [id]/
│   │   │       └── page.tsx              # Upload Arsip Digital (PENGARSIP)
│   │   ├── manifest/
│   │   │   ├── page.tsx                  # List Manifest (PENGIRIM)
│   │   │   ├── manifest-baru/
│   │   │   │   └── page.tsx
│   │   │   └── [id]/
│   │   │       └── page.tsx              # Detail, lock, upload bukti terima
│   │   └── pemantauan/
│   │       └── page.tsx                  # Dashboard PEMANTAU
│   └── api/
│       ├── upload/
│       │   └── route.ts                  # Route Handler upload file (Bab 6)
│       └── cron/
│           └── tutup-buku-tahunan/
│               └── route.ts              # Dipanggil Vercel Cron (Bab 13)
├── components/
│   ├── permohonan/
│   │   ├── PermohonanForm.tsx
│   │   └── PermohonanList.tsx
│   ├── bundle/
│   │   └── BundleCard.tsx
│   ├── manifest/
│   │   └── ManifestCard.tsx
│   └── ui/                               # komponen generik (Button, Table, dll.)
├── lib/
│   ├── prisma.ts                         # Prisma Client singleton
│   ├── auth.ts                           # konfigurasi NextAuth.js
│   ├── validations/
│   │   ├── permohonan.schema.ts          # Zod schema
│   │   ├── bundle.schema.ts
│   │   └── manifest.schema.ts
│   ├── state-machine/
│   │   └── validateTransition.ts         # Bab 12
│   ├── audit/
│   │   └── createAuditLog.ts             # Bab 7
│   └── actions/
│       ├── permohonan.actions.ts         # Server Actions
│       ├── bundle.actions.ts
│       └── manifest.actions.ts
├── prisma/
│   └── schema.prisma
├── types/
│   └── index.ts                          # Shared TS types (jika di luar Prisma-generated)
├── middleware.ts                         # Proteksi route berbasis role (Bab 10)
└── next.config.js
```

### 1.4 Alur Data

```
MongoDB
   │
   ▼
Prisma Client (lib/prisma.ts — singleton)
   │
   ▼
Server Actions (lib/actions/*) ──── validasi Zod ──── validateTransition() ──── createAuditLog()
   │                                                                                  │
   ▼                                                                                  ▼
React Server Components (app/**/page.tsx)                                   AuditLog Collection
   │
   ▼
Client Components (interaktivitas, form, tombol aksi)
```

Setiap mutasi yang mengubah status entitas (Permohonan/Bundle/Manifest) **wajib** melewati tiga tahap berurutan di Server Action: (1) validasi Zod terhadap payload, (2) `validateTransition()` untuk memastikan transisi status sah sesuai Bab 12, (3) penulisan ke `AuditLog` jika operasi tersebut tergolong yang wajib dicatat menurut PRD (eksplisit disebutkan untuk "Batal Selesai").

---

## 2. Data Model & Prisma Schema

### 2.1 Pendekatan Relasi di MongoDB (Reference, bukan Embedded)

MongoDB adalah dokumen-oriented dan secara native mendukung *embedding*. Namun PRD secara eksplisit mendefinisikan **state machine independen** untuk Permohonan, Bundle, dan Manifest (Bab 4 PRD) — masing-masing entitas punya siklus hidup, status, dan relasi yang berubah-ubah secara independen (contoh: Permohonan bisa dikeluarkan dari Bundle, Bundle bisa dilepas dari Manifest). Pola akses semacam ini membutuhkan **referensi (foreign-key-like), bukan embedding**, karena:

- Entitas anak (Permohonan) harus bisa di-query, di-update, dan berpindah relasi (`bundleId`) secara independen dari induknya.
- Embedding akan membuat update status satu Permohonan harus menulis ulang seluruh dokumen Bundle/Manifest, dan memutus relasi (Unbundle, Keluarkan dari Bundle) menjadi rumit secara atomik.

Karena itu, seluruh relasi pada schema di bawah menggunakan **Prisma `@relation` dengan `ObjectId` reference**, model standar Prisma untuk mensimulasikan foreign key di MongoDB.

### 2.2 `schema.prisma` Lengkap

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mongodb"
  url      = env("DATABASE_URL")
}

// ============================================================
// ENUMS
// ============================================================

enum Role {
  PENGINPUT
  PENELITI
  PENGARSIP
  PENGIRIM
  PEMANTAU
}

enum JenisPermohonan {
  MUTASI_SEBAGIAN
  MUTASI_HABIS_UPDATE
  MUTASI_HABIS_REGULER
  OBJEK_PAJAK_BARU
  PEMBETULAN
  PENGAKTIFAN
}

enum StatusPermohonan {
  SUBMITTED
  REVISION
  BUNDLED
  ARCHIVED
  COMPLETED
  REJECTED
}

enum StatusBundle {
  DRAFT
  LOCKED
  IN_MANIFEST
}

enum StatusManifest {
  DRAFT
  LOCKED
  SENT
}

// ============================================================
// USER
// ============================================================

model User {
  id        String   @id @default(auto()) @map("_id") @db.ObjectId
  name      String
  email     String   @unique
  role      Role                              // mutually exclusive, lihat Bab 10
  createdAt DateTime @default(now())
  deletedAt DateTime?                          // soft delete, lihat Bab 8

  permohonanDiinput Permohonan[] @relation("PermohonanPenginput")
  auditLogs         AuditLog[]   @relation("AuditLogActor")

  @@map("user")
}

// ============================================================
// PERMOHONAN
// ============================================================

model Permohonan {
  id              String            @id @default(auto()) @map("_id") @db.ObjectId
  nop             String            @unique                      // Nomor Objek Pajak, lihat Bab 11
  namaPemohon     String
  jenisPermohonan JenisPermohonan
  status          StatusPermohonan  @default(SUBMITTED)
  luasBumi        Float?
  luasBangunan    Float?
  tanggalSubmit   DateTime          @default(now())

  // Relasi ke Bundle — nullable & singular karena Permohonan SUBMITTED awal belum punya Bundle.
  // TANPA @unique: lihat penjelasan subbab 2.4 mengapa "1 Permohonan = maksimal 1 Bundle"
  // dijamin oleh tipe field singular ini, bukan oleh unique constraint.
  bundleId        String?           @db.ObjectId
  bundle          Bundle?           @relation(fields: [bundleId], references: [id])

  // Audit trail siapa yang menginput (Fase 1, role PENGINPUT)
  penginputId     String            @db.ObjectId
  penginput       User              @relation("PermohonanPenginput", fields: [penginputId], references: [id])

  // Path dokumen tersimpan (Bab 6)
  arsipDigitalUrl String?
  kertasKerjaUrl  String?

  version         Int               @default(0)                  // optimistic locking, lihat Bab 5 & 9
  createdAt       DateTime          @default(now())
  updatedAt       DateTime          @updatedAt
  deletedAt       DateTime?                                       // soft delete — tapi lihat Bab 6.2 PRD: Hard Delete dilarang mutlak

  @@map("permohonan")
}
```

**Catatan kritikal pada `bundleId`:** field ini bertipe scalar tunggal (singular), bukan array. Ini adalah implementasi langsung dari ketentuan PRD bahwa **satu Permohonan tidak bisa berada di dua Bundle sekaligus** — lihat penjelasan detail di subbab 2.4.

```prisma
// ============================================================
// BUNDLE
// ============================================================

model Bundle {
  id          String        @id @default(auto()) @map("_id") @db.ObjectId
  status      StatusBundle  @default(DRAFT)

  // Surat Pengantar Bundle — dicetak saat Peneliti mengunci Bundle (Fase 2)
  suratPengantarUrl String?

  // Relasi ke Manifest — nullable karena Bundle DRAFT/LOCKED belum tentu masuk Manifest
  manifestId  String?       @db.ObjectId
  manifest    Manifest?     @relation(fields: [manifestId], references: [id])

  // Relasi one-to-many: satu Bundle berisi banyak Permohonan (sisi "many" di model Permohonan)
  permohonanList Permohonan[]

  penelitiId  String        @db.ObjectId                          // role PENELITI yang membuat Bundle

  version     Int           @default(0)
  createdAt   DateTime      @default(now())
  updatedAt   DateTime      @updatedAt

  @@map("bundle")
}

// ============================================================
// MANIFEST
// ============================================================

model Manifest {
  id          String          @id @default(auto()) @map("_id") @db.ObjectId
  status      StatusManifest  @default(DRAFT)

  suratPengantarUrl String?                                       // Surat Pengantar Manifest (Fase 4)
  buktiTerimaUrl    String?                                       // Bukti Tanda Terima (Fase 4)

  // Relasi one-to-many: satu Manifest berisi banyak Bundle
  bundleList  Bundle[]

  pengirimId  String          @db.ObjectId                        // role PENGIRIM yang membuat Manifest

  version     Int             @default(0)
  createdAt   DateTime        @default(now())
  updatedAt   DateTime        @updatedAt

  @@map("manifest")
}

// ============================================================
// AUDIT LOG (immutable, lihat Bab 7)
// ============================================================

model AuditLog {
  id          String   @id @default(auto()) @map("_id") @db.ObjectId
  entityType  String                                              // "Permohonan" | "Bundle" | "Manifest"
  entityId    String   @db.ObjectId
  actorId     String   @db.ObjectId
  actor       User     @relation("AuditLogActor", fields: [actorId], references: [id])
  actorRole   Role
  action      String                                              // contoh: "BATAL_SELESAI", "MINTA_REVISI"
  oldValue    Json
  newValue    Json
  ipAddress   String?
  userAgent   String?
  createdAt   DateTime @default(now())

  // SENGAJA TIDAK ADA updatedAt ATAU deletedAt — lihat Bab 7

  @@map("auditLog")
}
```

### 2.3 Penjelasan Relasi One-to-Many

| Relasi | Sisi "One" | Sisi "Many" | Mekanisme Prisma + MongoDB |
|---|---|---|---|
| Manifest → Bundle | `Manifest.id` | `Bundle.manifestId` | Foreign key reference (`@db.ObjectId` + `@relation`) disimpan di sisi "many" (`Bundle`), persis seperti pola SQL standar. Field `Manifest.bundleList` adalah *virtual relation field* (tidak disimpan fisik di dokumen Manifest) — Prisma menghitungnya saat query lewat `include`. |
| Bundle → Permohonan | `Bundle.id` | `Permohonan.bundleId` | Sama seperti di atas: `Permohonan.bundleId` adalah field fisik berisi `ObjectId` yang menunjuk ke `Bundle.id`. `Bundle.permohonanList` adalah *virtual relation field*. |

### 2.4 Constraint: Permohonan Tidak Bisa Berada di Dua Bundle Sekaligus

PRD menegaskan bahwa Permohonan hanya valid berada dalam **satu** Bundle pada satu waktu (lihat siklus `SUBMITTED → BUNDLED`, dan fitur "Keluarkan dari Bundle"/"Unbundle" yang selalu memutus relasi sebelum Permohonan bisa di-bundle ulang).

**Jaminan struktural (tanpa unique constraint tambahan):** Field `bundleId` pada model `Permohonan` bertipe **scalar tunggal** (`String? @db.ObjectId`), bukan array (`String[]`). Karena satu dokumen Permohonan hanya memiliki **satu** field `bundleId`, secara struktural ia tidak mungkin menunjuk ke dua Bundle sekaligus — bentuk tipe data itu sendiri yang menjamin batasan ini, sehingga tidak diperlukan `@unique` pada field ini (perlu dicatat: `@unique` pada `bundleId` justru akan berarti "satu Bundle hanya boleh dimiliki satu Permohonan", arah constraint yang berbeda dan tidak relevan dengan kebutuhan PRD).

Validasi tambahan tetap dilakukan di level aplikasi (Server Action) sebelum proses "centang permohonan ke Bundle" (Fase 2 PRD): Server Action memeriksa `permohonan.bundleId === null` dan `permohonan.status === "SUBMITTED"` sebelum mengizinkan assignment, sesuai aturan PRD bahwa Penginput hanya bisa update data selama "belum terikat `bundleId`".

```typescript
// lib/actions/bundle.actions.ts
export async function tambahPermohonanKeBundle(bundleId: string, permohonanId: string) {
  const permohonan = await prisma.permohonan.findUniqueOrThrow({ where: { id: permohonanId } });

  if (permohonan.bundleId !== null) {
    throw new Error("Permohonan sudah terikat pada Bundle lain.");
  }
  if (permohonan.status !== "SUBMITTED") {
    throw new Error("Hanya Permohonan berstatus SUBMITTED yang bisa dimasukkan ke Bundle.");
  }

  return prisma.permohonan.update({
    where: { id: permohonanId, version: permohonan.version },
    data: {
      bundleId,
      status: "BUNDLED",
      version: { increment: 1 },
    },
  });
}
```

### 2.5 Ringkasan Embedded vs Reference

| Aspek | Keputusan | Alasan |
|---|---|---|
| Permohonan ↔ Bundle ↔ Manifest | **Reference** (`@relation` + `ObjectId`) | Setiap entitas punya state machine independen (PRD Bab 4) dan relasi yang berubah-ubah (bundling/unbundling, lock/unlock manifest) — embedding akan menyulitkan operasi atomik per-entitas. |
| AuditLog ↔ User/Entity | **Reference** (`actorId`, `entityId` sebagai `ObjectId`) | AuditLog harus tetap valid meski entitas asal berubah status; reference menjaga AuditLog ringan dan tidak terikat fisik pada dokumen entitas yang diaudit. |

---

## 3. Naming Convention

| Kategori | Konvensi | Contoh |
|---|---|---|
| Collection (MongoDB, lewat `@@map`) | camelCase singular | `permohonan`, `bundle`, `manifest`, `user`, `auditLog` |
| Field di MongoDB / Prisma | camelCase | `bundleId`, `manifestId`, `createdAt`, `namaPemohon`, `arsipDigitalUrl` |
| Enum (nama tipe) | PascalCase | `StatusPermohonan`, `StatusBundle`, `JenisPermohonan`, `Role` |
| Enum (nilai/member) | UPPER_SNAKE_CASE | `SUBMITTED`, `LOCKED`, `IN_MANIFEST`, `MUTASI_SEBAGIAN`, `PENGINPUT` |
| Variabel & fungsi TypeScript | camelCase | `bundleId`, `manifestId`, `validateTransition()`, `createAuditLog()` |
| Komponen React | PascalCase | `PermohonanList.tsx`, `BundleCard.tsx`, `ManifestForm.tsx` |
| File Route Next.js (folder App Router) | kebab-case | `permohonan-baru/page.tsx`, `bundle-baru/page.tsx`, `tutup-buku-tahunan/route.ts` |
| Server Action file | kebab/camel sesuai domain + suffix `.actions.ts` | `permohonan.actions.ts`, `bundle.actions.ts` |
| Zod schema file | domain + suffix `.schema.ts` | `permohonan.schema.ts` |

**Alasan:** Konvensi ini mengikuti default idiomatik masing-masing ekosistem — Prisma/MongoDB field memang lazim camelCase, enum value UPPER_SNAKE_CASE adalah konvensi Prisma/TypeScript standar untuk konstanta status (juga konsisten satu-ke-satu dengan penamaan status yang sudah dipakai di PRD, contoh: `SUBMITTED`, `LOCKED`), dan kebab-case untuk folder route adalah konvensi resmi Next.js App Router untuk URL segments.

---

## 4. Cascade Rules & Orphan Handling

> Bab ini menerjemahkan secara teknis tiga anomali yang sudah dijabarkan PRD: "Keluarkan dari Bundle" (PRD 6.2), "Unbundle" (PRD 6.3), dan implikasi struktural saat Bundle/Manifest kehilangan seluruh anggotanya akibat aksi-aksi tersebut. **Tidak ada aturan auto-delete/auto-reset baru di luar yang sudah tersirat dari fitur PRD** — bab ini hanya memastikan konsistensi data saat fitur-fitur tersebut dijalankan berulang kali hingga Bundle/Manifest kosong.

### 4.1 Permohonan di-Unbundle (PRD 6.3) — `bundleId = null`, status kembali `SUBMITTED`

```typescript
// lib/actions/permohonan.actions.ts
import { prisma } from "@/lib/prisma";
import { validateTransition } from "@/lib/state-machine/validateTransition";

export async function unbundlePermohonan(permohonanId: string, actorId: string) {
  const permohonan = await prisma.permohonan.findUniqueOrThrow({
    where: { id: permohonanId },
  });

  validateTransition({
    entity: "Permohonan",
    currentStatus: permohonan.status,
    newStatus: "SUBMITTED",
    role: "PENGARSIP",
  });

  return prisma.permohonan.update({
    where: { id: permohonanId, version: permohonan.version },
    data: {
      bundleId: null,
      status: "SUBMITTED",
      version: { increment: 1 },
    },
  });
}
```

### 4.2 Bundle Menjadi Kosong (0 Permohonan)

Bundle dapat kehilangan seluruh anggotanya melalui dua jalur yang sudah ada di PRD: "Keluarkan dari Bundle" (PRD 6.2, dilakukan berulang oleh Peneliti selama Bundle `DRAFT`) atau "Unbundle" (PRD 6.3, dilakukan oleh Pengarsip). PRD tidak menyebutkan Bundle kosong sebagai status baru — Bundle yang kosong tetap berada pada status `DRAFT` (selama belum dikunci) dan tidak dihapus otomatis, karena PRD tidak pernah menyatakan adanya auto-delete Bundle. Pembersihan Bundle kosong yang tidak terpakai bersifat manual oleh Peneliti, bukan otomatis sistem.

```typescript
// lib/actions/bundle.actions.ts
export async function keluarkanDariBundle(permohonanId: string) {
  const permohonan = await prisma.permohonan.findUniqueOrThrow({
    where: { id: permohonanId },
  });

  if (!permohonan.bundleId) {
    throw new Error("Permohonan tidak terikat pada Bundle manapun.");
  }

  const bundle = await prisma.bundle.findUniqueOrThrow({
    where: { id: permohonan.bundleId },
  });

  if (bundle.status !== "DRAFT") {
    throw new Error('Fitur "Keluarkan dari Bundle" hanya berlaku selama Bundle berstatus DRAFT.');
  }

  // Lepas relasi Permohonan, status kembali SUBMITTED (PRD 6.2)
  await prisma.permohonan.update({
    where: { id: permohonanId, version: permohonan.version },
    data: {
      bundleId: null,
      status: "SUBMITTED",
      version: { increment: 1 },
    },
  });

  // Bundle TIDAK dihapus otomatis meski kosong — tetap DRAFT, menunggu pengisian/penguncian manual.
  return { success: true };
}
```

### 4.3 Manifest Menjadi Kosong (0 Bundle) → Reset ke `DRAFT`

PRD 6.4 secara eksplisit menyebutkan fitur "Revisi Manifest": membuka gembok Manifest dari `LOCKED` kembali ke `DRAFT` untuk melepas Bundle yang tertinggal, lalu dikunci ulang. Aturan reset status Manifest ke `DRAFT` saat dibuka ini **adalah** mekanisme yang dimaksud — bukan aturan tambahan, melainkan implementasi langsung dari "Revisi Manifest":

```typescript
// lib/actions/manifest.actions.ts
export async function revisiManifest(manifestId: string) {
  const manifest = await prisma.manifest.findUniqueOrThrow({
    where: { id: manifestId },
  });

  validateTransition({
    entity: "Manifest",
    currentStatus: manifest.status,
    newStatus: "DRAFT",
    role: "PENGIRIM",
  });

  // Buka gembok Manifest kembali ke DRAFT (PRD 6.4)
  return prisma.manifest.update({
    where: { id: manifestId, version: manifest.version },
    data: {
      status: "DRAFT",
      version: { increment: 1 },
    },
  });
}

export async function lepasBundleDariManifest(manifestId: string, bundleId: string) {
  const bundle = await prisma.bundle.findUniqueOrThrow({ where: { id: bundleId } });

  if (bundle.manifestId !== manifestId) {
    throw new Error("Bundle tidak terdaftar pada Manifest ini.");
  }

  // Melepas map yang tertinggal dari daftar elektronik Manifest (PRD 6.4)
  return prisma.bundle.update({
    where: { id: bundleId, version: bundle.version },
    data: {
      manifestId: null,
      status: "LOCKED",         // Bundle kembali berstatus LOCKED (lepas dari Manifest, fisik tetap dalam map terkunci)
      version: { increment: 1 },
    },
  });
}
```

**Catatan:** PRD tidak menyebutkan adanya proses otomatis ("auto-reset") yang berjalan sendiri ketika Manifest kebetulan menjadi kosong di luar alur "Revisi Manifest". Maka tidak ada Prisma middleware tambahan untuk skenario ini — reset ke `DRAFT` hanya terjadi sebagai bagian eksplisit dari aksi "Revisi Manifest" yang dipicu Pengirim, persis sebagaimana dijabarkan PRD.

---

## 5. Idempotency & Double-Submit Prevention

### 5.1 Optimistic Locking dengan Field `version`

Setiap model yang dapat mengalami mutasi status (`Permohonan`, `Bundle`, `Manifest`) memiliki field `version Int @default(0)` (lihat Bab 2). Setiap operasi update **wajib** menyertakan `version` lama di klausa `where`, dan menaikkan `version` baru di `data`:

```typescript
// Pola umum optimistic locking di seluruh Server Action
async function updateWithOptimisticLock<T>(
  permohonanId: string,
  expectedVersion: number,
  data: Record<string, unknown>
) {
  const result = await prisma.permohonan.updateMany({
    where: {
      id: permohonanId,
      version: expectedVersion,
    },
    data: {
      ...data,
      version: { increment: 1 },
    },
  });

  if (result.count === 0) {
    throw new ConflictError("Konflik data, silakan refresh.");
  }

  return result;
}
```

### 5.2 Custom Error untuk Konflik

```typescript
// lib/errors.ts
export class ConflictError extends Error {
  constructor(message = "Konflik data, silakan refresh.") {
    super(message);
    this.name = "ConflictError";
  }
}
```

### 5.3 Penanganan di Server Action (Double-Submit Prevention)

```typescript
// lib/actions/permohonan.actions.ts
"use server";

import { z } from "zod";
import { prisma } from "@/lib/prisma";
import { ConflictError } from "@/lib/errors";

const mintaRevisiSchema = z.object({
  permohonanId: z.string(),
  version: z.number().int().nonnegative(),
});

export type MintaRevisiState = {
  success: boolean;
  message: string;
};

export async function mintaRevisiAction(
  prevState: MintaRevisiState,
  formData: FormData
): Promise<MintaRevisiState> {
  const parsed = mintaRevisiSchema.safeParse({
    permohonanId: formData.get("permohonanId"),
    version: Number(formData.get("version")),
  });

  if (!parsed.success) {
    return { success: false, message: "Data tidak valid." };
  }

  try {
    const result = await prisma.permohonan.updateMany({
      where: { id: parsed.data.permohonanId, version: parsed.data.version, status: "SUBMITTED" },
      data: { status: "REVISION", version: { increment: 1 } },
    });

    if (result.count === 0) {
      throw new ConflictError();
    }

    return { success: true, message: "Permohonan berhasil diminta revisi." };
  } catch (err) {
    if (err instanceof ConflictError) {
      return { success: false, message: err.message };
    }
    return { success: false, message: "Terjadi kesalahan sistem." };
  }
}
```

### 5.4 Implementasi Frontend: `useActionState` + Disable Button

```tsx
// components/permohonan/MintaRevisiButton.tsx
"use client";

import { useActionState } from "react";
import { mintaRevisiAction, type MintaRevisiState } from "@/lib/actions/permohonan.actions";

const initialState: MintaRevisiState = { success: false, message: "" };

export function MintaRevisiButton({ permohonanId, version }: { permohonanId: string; version: number }) {
  const [state, formAction, isPending] = useActionState(mintaRevisiAction, initialState);

  return (
    <form action={formAction}>
      <input type="hidden" name="permohonanId" value={permohonanId} />
      <input type="hidden" name="version" value={version} />
      <button type="submit" disabled={isPending}>
        {isPending ? "Memproses..." : "Minta Revisi"}
      </button>
      {state.message && <p>{state.message}</p>}
    </form>
  );
}
```

**Alasan:** Kombinasi `version` field + `updateMany` dengan `where` ganda (id + version) adalah pola idempotency standar untuk database tanpa native row-locking seperti MongoDB. Tombol di-`disable` selama `isPending` mencegah double-submit di sisi klien, sementara optimistic locking di server adalah lapisan pertahanan kedua yang menjamin konsistensi meski race condition tetap terjadi (misalnya dua tab browser).

---

## 6. File Storage Strategy

### 6.1 Pilihan: Vercel Blob

**Alasan:** Tech stack yang ditentukan menyebutkan "Vercel Blob atau yang lainnya jika diperlukan", dan karena seluruh aplikasi sudah berjalan di atas Next.js App Router (ekosistem Vercel-native), Vercel Blob dipilih karena integrasinya langsung dengan Route Handler tanpa konfigurasi kredensial cloud terpisah (dibanding S3/Cloudinary yang butuh setup IAM/API key eksternal), sesuai prinsip "pilih solusi paling sederhana" pada constraint dokumen ini.

### 6.2 Path Pattern

```
/{tahun}/{bulan}/{permohonanId}/{jenisDokumen}.pdf
```

Contoh: `/2026/06/665f1a2b3c4d5e6f7a8b9c0d/arsipDigital.pdf`

### 6.3 Jenis Dokumen

| Jenis Dokumen | Sumber PRD | Diunggah oleh |
|---|---|---|
| `arsipDigital` | Fase 3 — hasil pindai dokumen fisik menjadi PDF | PENGARSIP |
| `buktiTerima` | Fase 4 — Bukti Tanda Terima dari Kantor Pusat | PENGIRIM |
| `kertasKerja` | Fase 2 — khusus permohonan Mutasi Sebagian | PENELITI |

### 6.4 Validasi File

- **Magic number, bukan extension** — menggunakan library `file-type` untuk memverifikasi byte signature PDF (`%PDF-`), karena ekstensi `.pdf` pada nama file dapat dipalsukan.
- **Maksimal ukuran:** 10MB.

```bash
npm install file-type
```

### 6.5 Route Handler Upload

```typescript
// app/api/upload/route.ts
import { NextRequest, NextResponse } from "next/server";
import { put } from "@vercel/blob";
import { fileTypeFromBuffer } from "file-type";

const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
const ALLOWED_JENIS = ["arsipDigital", "buktiTerima", "kertasKerja"] as const;

export async function POST(request: NextRequest) {
  const formData = await request.formData();
  const file = formData.get("file") as File | null;
  const permohonanId = formData.get("permohonanId") as string | null;
  const jenisDokumen = formData.get("jenisDokumen") as string | null;

  if (!file || !permohonanId || !jenisDokumen) {
    return NextResponse.json({ error: "Parameter tidak lengkap." }, { status: 400 });
  }

  if (!ALLOWED_JENIS.includes(jenisDokumen as (typeof ALLOWED_JENIS)[number])) {
    return NextResponse.json({ error: "Jenis dokumen tidak valid." }, { status: 400 });
  }

  if (file.size > MAX_FILE_SIZE) {
    return NextResponse.json({ error: "Ukuran file melebihi 10MB." }, { status: 400 });
  }

  const buffer = Buffer.from(await file.arrayBuffer());

  // Validasi magic number — memastikan file benar-benar PDF, bukan hanya berekstensi .pdf
  const detectedType = await fileTypeFromBuffer(buffer);
  if (!detectedType || detectedType.mime !== "application/pdf") {
    return NextResponse.json({ error: "File harus berupa PDF yang valid." }, { status: 400 });
  }

  const now = new Date();
  const tahun = now.getFullYear();
  const bulan = String(now.getMonth() + 1).padStart(2, "0");
  const path = `/${tahun}/${bulan}/${permohonanId}/${jenisDokumen}.pdf`;

  const blob = await put(path, buffer, {
    access: "public",
    contentType: "application/pdf",
  });

  return NextResponse.json({ url: blob.url }, { status: 201 });
}
```

**Alasan validasi 2-tahap (ekstensi + magic number):** Mencegah upaya unggah file berbahaya yang disamarkan sebagai PDF, sesuai praktik validasi file standar untuk sistem yang menyimpan dokumen legal/pemerintahan.

---

## 7. Audit Log Schema & Implementation

### 7.1 Struktur Model `AuditLog` (Immutable)

Model `AuditLog` sudah didefinisikan penuh di Bab 2.2. Ringkasan field wajib:

| Field | Tipe | Keterangan |
|---|---|---|
| `id` | `String @id` | ObjectId MongoDB |
| `entityType` | `String` | `"Permohonan"` \| `"Bundle"` \| `"Manifest"` |
| `entityId` | `String @db.ObjectId` | Referensi ke entitas yang diaudit |
| `actorId` | `String @db.ObjectId` | User yang melakukan aksi |
| `actorRole` | `Role` | Role aktor saat aksi dilakukan |
| `action` | `String` | Nama aksi, contoh `"BATAL_SELESAI"` |
| `oldValue` | `Json` | Snapshot nilai sebelum perubahan |
| `newValue` | `Json` | Snapshot nilai sesudah perubahan |
| `ipAddress` | `String?` | Opsional |
| `userAgent` | `String?` | Opsional |
| `createdAt` | `DateTime @default(now())` | Waktu pencatatan |

**Tegas: tidak ada `updatedAt` maupun `deletedAt`.** AuditLog bersifat *append-only* — sekali ditulis, tidak pernah diubah atau dihapus, sesuai prinsip audit trail yang PRD tuntut pada fitur "Batal Selesai" (PRD 6.5: *"Nilai perubahan sebelum dan sesudah wajib terekam ke Audit Log"*).

### 7.2 Implementasi: Helper Function untuk Insert Audit Log

PRD secara eksplisit hanya menuntut pencatatan Audit Log pada **aksi "Batal Selesai" (Rollback)** di Fase 5 (PEMANTAU). Karena itu, pencatatan dilakukan melalui pemanggilan eksplisit di Server Action terkait — bukan via Prisma middleware global yang mencatat *seluruh* operasi update (yang akan menjadi penambahan scope di luar PRD).

```typescript
// lib/audit/createAuditLog.ts
import { prisma } from "@/lib/prisma";
import type { Role } from "@prisma/client";

interface CreateAuditLogParams {
  entityType: "Permohonan" | "Bundle" | "Manifest";
  entityId: string;
  actorId: string;
  actorRole: Role;
  action: string;
  oldValue: Record<string, unknown>;
  newValue: Record<string, unknown>;
  ipAddress?: string;
  userAgent?: string;
}

export async function createAuditLog(params: CreateAuditLogParams) {
  return prisma.auditLog.create({
    data: {
      entityType: params.entityType,
      entityId: params.entityId,
      actorId: params.actorId,
      actorRole: params.actorRole,
      action: params.action,
      oldValue: params.oldValue,
      newValue: params.newValue,
      ipAddress: params.ipAddress,
      userAgent: params.userAgent,
    },
  });
}
```

### 7.3 Contoh Pemakaian: Fitur "Batal Selesai" (Rollback)

```typescript
// lib/actions/permohonan.actions.ts
"use server";

import { prisma } from "@/lib/prisma";
import { createAuditLog } from "@/lib/audit/createAuditLog";
import { validateTransition } from "@/lib/state-machine/validateTransition";
import { headers } from "next/headers";

export async function batalSelesaiAction(permohonanId: string, actorId: string, actorRole: "PEMANTAU") {
  const permohonan = await prisma.permohonan.findUniqueOrThrow({ where: { id: permohonanId } });

  validateTransition({
    entity: "Permohonan",
    currentStatus: permohonan.status,
    newStatus: "ARCHIVED",
    role: actorRole,
  });

  const oldValue = { status: permohonan.status };
  const newValue = { status: "ARCHIVED" as const };

  const updated = await prisma.permohonan.update({
    where: { id: permohonanId, version: permohonan.version },
    data: { status: "ARCHIVED", version: { increment: 1 } },
  });

  const headersList = await headers();

  await createAuditLog({
    entityType: "Permohonan",
    entityId: permohonanId,
    actorId,
    actorRole,
    action: "BATAL_SELESAI",
    oldValue,
    newValue,
    ipAddress: headersList.get("x-forwarded-for") ?? undefined,
    userAgent: headersList.get("user-agent") ?? undefined,
  });

  return updated;
}
```

**Alasan tidak memakai Prisma middleware global untuk seluruh entitas:** PRD hanya secara eksplisit mensyaratkan audit trail pada satu titik (Batal Selesai). Memasang middleware yang otomatis mencatat *setiap* update di seluruh model akan menjadi keputusan teknis yang melampaui apa yang diminta PRD. Pemanggilan eksplisit `createAuditLog()` pada Server Action terkait adalah pendekatan paling presisi dan sesuai cakupan PRD.

---

## 8. Soft Delete Implementation

### 8.1 Field `deletedAt DateTime?`

PRD 6.1 menyatakan **"*Hard Delete* dilarang mutlak"** pada data Permohonan (Fase 1, PENGINPUT). Larangan hard delete ini diimplementasikan dengan pola soft delete: field `deletedAt DateTime?` (bukan `Boolean isDeleted`), karena `DateTime?` sekaligus menyimpan kapan penghapusan terjadi — informasi yang hilang jika hanya memakai boolean.

```prisma
model Permohonan {
  // ...field lain sudah didefinisikan di Bab 2
  deletedAt DateTime?
}

model User {
  // ...
  deletedAt DateTime?
}
```

**Catatan cakupan:** PRD secara eksplisit menyebutkan larangan Hard Delete hanya pada konteks Permohonan (Fase 1). Field `deletedAt` pada model `User` disertakan sebagai praktik standar manajemen akun (menonaktifkan user tanpa kehilangan riwayat `actorId` di AuditLog), namun **tidak ada mekanisme soft-delete untuk `Bundle` atau `Manifest`** karena PRD tidak mendefinisikan skenario penghapusan untuk kedua entitas tersebut sama sekali — siklus hidup keduanya berhenti di status terminal (`IN_MANIFEST` untuk Bundle yang sudah terkirim, `SENT` untuk Manifest), bukan dihapus.

### 8.2 Query Default Memfilter `deletedAt: null`

```typescript
// lib/prisma.ts
import { PrismaClient } from "@prisma/client";

const prismaClientSingleton = () => {
  return new PrismaClient().$extends({
    query: {
      permohonan: {
        async findMany({ args, query }) {
          args.where = { ...args.where, deletedAt: null };
          return query(args);
        },
        async findUnique({ args, query }) {
          args.where = { ...args.where, deletedAt: null };
          return query(args);
        },
        async findFirst({ args, query }) {
          args.where = { ...args.where, deletedAt: null };
          return query(args);
        },
      },
      user: {
        async findMany({ args, query }) {
          args.where = { ...args.where, deletedAt: null };
          return query(args);
        },
      },
    },
  });
};

declare const globalForPrisma: { prisma?: ReturnType<typeof prismaClientSingleton> };

export const prisma = globalForPrisma.prisma ?? prismaClientSingleton();

if (process.env.NODE_ENV !== "production") {
  (globalForPrisma as { prisma?: typeof prisma }).prisma = prisma;
}
```

### 8.3 Operasi "Delete" yang Sebenarnya Set `deletedAt`

```typescript
// lib/actions/permohonan.actions.ts
export async function softDeletePermohonan(permohonanId: string, version: number) {
  const result = await prisma.permohonan.updateMany({
    where: { id: permohonanId, version, status: "SUBMITTED" }, // hanya boleh selama SUBMITTED, sesuai PRD 6.1
    data: { deletedAt: new Date(), version: { increment: 1 } },
  });

  if (result.count === 0) {
    throw new Error("Konflik data atau Permohonan tidak memenuhi syarat untuk dihapus.");
  }

  return { success: true };
}
```

### 8.4 Pengecualian: `AuditLog` TIDAK Boleh Soft Delete

Model `AuditLog` (Bab 2.2 dan 7.1) **sengaja tidak memiliki field `deletedAt`**. Karena `$extends` di atas hanya didaftarkan untuk model `permohonan` dan `user`, model `auditLog` tidak pernah ikut difilter atau dimodifikasi oleh ekstensi soft-delete ini — konsisten dengan sifat *append-only* dan *immutable* yang dituntut Bab 7.

---

## 9. Concurrency Control

### 9.1 Optimistic Locking sebagai Pengganti `SELECT FOR UPDATE`

MongoDB tidak memiliki mekanisme row-locking native seperti `SELECT FOR UPDATE` pada RDBMS. Field `version Int` (sudah didefinisikan di Bab 2 pada `Permohonan`, `Bundle`, `Manifest`) adalah mekanisme penggantinya: setiap update wajib menyertakan kondisi `version = oldVersion` pada `where`, sebagaimana sudah dijabarkan di Bab 5.

```typescript
// Pola wajib pada SETIAP operasi update status di seluruh Server Action
const result = await prisma.bundle.updateMany({
  where: { id: bundleId, version: currentVersion },
  data: { status: "LOCKED", version: { increment: 1 } },
});

if (result.count === 0) {
  throw new Error("Konflik data, silakan refresh.");
}
```

### 9.2 MongoDB Multi-Document Transaction untuk Operasi Kritis

PRD 6.2 secara eksplisit menyebutkan **Race Condition (Tabrakan) pada Fase 2 dicegah via *database transaction* di sisi server** — yaitu pada momen Peneliti memasukkan Permohonan ke dalam Bundle. Operasi ini melibatkan dua dokumen sekaligus (update `Permohonan.bundleId` + `Permohonan.status`, serta — bila relevan — perubahan pada `Bundle`), sehingga harus dibungkus `Prisma.$transaction()` agar atomik.

```typescript
// lib/actions/bundle.actions.ts
import { prisma } from "@/lib/prisma";

export async function masukkanPermohonanKeBundleTransaksional(
  bundleId: string,
  permohonanId: string,
  permohonanVersion: number
) {
  return prisma.$transaction(async (tx) => {
    // 1. Validasi Bundle masih DRAFT (mencegah masuk ke Bundle yang sudah LOCKED)
    const bundle = await tx.bundle.findUniqueOrThrow({ where: { id: bundleId } });
    if (bundle.status !== "DRAFT") {
      throw new Error("Bundle sudah tidak dalam status DRAFT.");
    }

    // 2. Validasi Permohonan belum terikat Bundle lain & masih SUBMITTED
    const permohonan = await tx.permohonan.findUniqueOrThrow({ where: { id: permohonanId } });
    if (permohonan.bundleId !== null) {
      throw new Error("Permohonan sudah terikat pada Bundle lain.");
    }
    if (permohonan.status !== "SUBMITTED") {
      throw new Error("Permohonan harus berstatus SUBMITTED.");
    }

    // 3. Update atomik dengan optimistic lock di dalam transaksi
    const updateResult = await tx.permohonan.updateMany({
      where: { id: permohonanId, version: permohonanVersion },
      data: {
        bundleId,
        status: "BUNDLED",
        version: { increment: 1 },
      },
    });

    if (updateResult.count === 0) {
      throw new Error("Konflik data, silakan refresh.");
    }

    return tx.permohonan.findUniqueOrThrow({ where: { id: permohonanId } });
  });
}
```

**Alasan:** `Prisma.$transaction()` pada MongoDB memanfaatkan *multi-document ACID transaction* (didukung sejak MongoDB 4.0+ untuk replica set), menjamin bahwa validasi status Bundle, validasi status Permohonan, dan penulisan akhir terjadi sebagai satu unit atomik — sehingga dua Peneliti yang mencoba memasukkan Permohonan yang sama ke dua Bundle berbeda secara bersamaan tidak akan menghasilkan data tidak konsisten (PRD 6.2: *"Race Condition (Tabrakan): Dicegah secara teknis via database transaction di sisi server"*).

---

## 10. Role Exclusivity & Authentication

### 10.1 Satu User = Satu Role (Mutually Exclusive)

PRD Bab 3 memetakan **5 role yang masing-masing bertanggung jawab pada satu fase alur kerja** secara terpisah, tanpa menyebutkan kemungkinan satu user memegang lebih dari satu role. Implementasi: field `role` langsung pada model `User` sebagai enum tunggal (bukan tabel pivot many-to-many), sehingga secara struktural satu user **tidak bisa** memiliki lebih dari satu role.

```prisma
// Sudah didefinisikan di Bab 2.2
enum Role {
  PENGINPUT
  PENELITI
  PENGARSIP
  PENGIRIM
  PEMANTAU
}

model User {
  id    String @id @default(auto()) @map("_id") @db.ObjectId
  email String @unique
  role  Role                          // single scalar field, bukan array atau relasi pivot
  // ...
}
```

**Alasan:** Sebuah scalar field enum tunggal (bukan `Role[]`) secara struktural menjamin eksklusivitas — sama seperti pola pencegahan "1 Permohonan di 2 Bundle" pada Bab 2.4, jaminan datang dari bentuk tipe data itu sendiri, bukan dari constraint tambahan.

### 10.2 Integrasi NextAuth.js

```typescript
// lib/auth.ts
import NextAuth from "next-auth";
import Credentials from "next-auth/providers/credentials";
import { prisma } from "@/lib/prisma";

export const { handlers, auth, signIn, signOut } = NextAuth({
  providers: [
    Credentials({
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" },
      },
      async authorize(credentials) {
        const user = await prisma.user.findUnique({
          where: { email: credentials?.email as string, deletedAt: null },
        });

        if (!user) return null;

        // Verifikasi password (hashing diasumsikan sudah ditangani saat registrasi user)
        // ...

        return {
          id: user.id,
          email: user.email,
          name: user.name,
          role: user.role,
        };
      },
    }),
  ],
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.role = user.role;
      }
      return token;
    },
    async session({ session, token }) {
      if (session.user) {
        session.user.role = token.role as string;
      }
      return session;
    },
  },
  session: { strategy: "jwt" },
});
```

```typescript
// types/next-auth.d.ts — memperluas tipe session NextAuth agar mengandung role
import { Role } from "@prisma/client";

declare module "next-auth" {
  interface Session {
    user: {
      id: string;
      email: string;
      name: string;
      role: Role;
    };
  }
}
```

### 10.3 Middleware Proteksi Route Berdasarkan Role

Pemetaan route ke role mengikuti pembagian fase di PRD Bab 3 dan struktur folder di Bab 1.3:

```typescript
// middleware.ts
import { auth } from "@/lib/auth";
import { NextResponse } from "next/server";

const ROUTE_ROLE_MAP: Record<string, string[]> = {
  "/permohonan": ["PENGINPUT"],
  "/permohonan-baru": ["PENGINPUT"],
  "/bundle": ["PENELITI"],
  "/bundle-baru": ["PENELITI"],
  "/arsip": ["PENGARSIP"],
  "/manifest": ["PENGIRIM"],
  "/manifest-baru": ["PENGIRIM"],
  "/pemantauan": ["PEMANTAU"],
};

export default auth((req) => {
  const { pathname } = req.nextUrl;
  const session = req.auth;

  if (!session) {
    return NextResponse.redirect(new URL("/login", req.url));
  }

  const matchedRoute = Object.keys(ROUTE_ROLE_MAP).find((route) => pathname.startsWith(route));

  if (matchedRoute) {
    const allowedRoles = ROUTE_ROLE_MAP[matchedRoute];
    if (!allowedRoles.includes(session.user.role)) {
      return NextResponse.redirect(new URL("/unauthorized", req.url));
    }
  }

  return NextResponse.next();
});

export const config = {
  matcher: ["/permohonan/:path*", "/bundle/:path*", "/arsip/:path*", "/manifest/:path*", "/pemantauan/:path*"],
};
```

**Alasan pemetaan route-ke-role 1:1:** Mengikuti persis pembagian "Role: PENGINPUT/PENELITI/PENGARSIP/PENGIRIM/PEMANTAU" yang masing-masing bertanggung jawab pada satu fase di PRD Bab 3 — tidak ada role yang memiliki akses lintas-fase menurut PRD, sehingga middleware membatasi akses route secara ketat 1:1 sesuai fase.

---

## 11. Data Validation Rules

### 11.1 Aturan Validasi per Field

| Field | Aturan | Sumber |
|---|---|---|
| `nop` (Nomor Objek Pajak) | Regex `^\d{18}$`, wajib unik | Konvensi NOP 18 digit, constraint unik ditegaskan eksplisit |
| `luasBumi` | number, minimal 0, maksimal 999999 | — |
| `luasBangunan` | number, minimal 0, maksimal 999999 | — |
| `tanggalSubmit` | Auto-generated, tidak bisa di-manual | PRD: Permohonan menjadi `SUBMITTED` saat data awal diinput (Fase 1) |
| `namaPemohon` | Minimal 3 karakter, trim whitespace | — |

### 11.2 Zod Schema — Permohonan

```typescript
// lib/validations/permohonan.schema.ts
import { z } from "zod";

export const jenisPermohonanEnum = z.enum([
  "MUTASI_SEBAGIAN",
  "MUTASI_HABIS_UPDATE",
  "MUTASI_HABIS_REGULER",
  "OBJEK_PAJAK_BARU",
  "PEMBETULAN",
  "PENGAKTIFAN",
]);

export const permohonanSchema = z.object({
  nop: z
    .string()
    .regex(/^\d{18}$/, "NOP harus terdiri dari tepat 18 digit angka."),
  namaPemohon: z
    .string()
    .trim()
    .min(3, "Nama pemohon minimal 3 karakter."),
  jenisPermohonan: jenisPermohonanEnum,
  luasBumi: z
    .number()
    .min(0, "Luas Bumi tidak boleh negatif.")
    .max(999999, "Luas Bumi maksimal 999999.")
    .optional(),
  luasBangunan: z
    .number()
    .min(0, "Luas Bangunan tidak boleh negatif.")
    .max(999999, "Luas Bangunan maksimal 999999.")
    .optional(),
  // tanggalSubmit TIDAK termasuk dalam schema input —
  // di-generate otomatis oleh Prisma @default(now()) di server, tidak menerima nilai dari klien.
});

export type PermohonanInput = z.infer<typeof permohonanSchema>;
```

### 11.3 Validasi 2 Layer: Frontend (React Hook Form + Zod) dan Backend (Server Action)

**Frontend:**

```bash
npm install react-hook-form @hookform/resolvers
```

```tsx
// components/permohonan/PermohonanForm.tsx
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { permohonanSchema, type PermohonanInput } from "@/lib/validations/permohonan.schema";
import { createPermohonanAction } from "@/lib/actions/permohonan.actions";

export function PermohonanForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<PermohonanInput>({
    resolver: zodResolver(permohonanSchema),
  });

  const onSubmit = async (data: PermohonanInput) => {
    await createPermohonanAction(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("nop")} placeholder="Nomor Objek Pajak (18 digit)" />
      {errors.nop && <p>{errors.nop.message}</p>}

      <input {...register("namaPemohon")} placeholder="Nama Pemohon" />
      {errors.namaPemohon && <p>{errors.namaPemohon.message}</p>}

      <select {...register("jenisPermohonan")}>
        <option value="MUTASI_SEBAGIAN">Mutasi Sebagian</option>
        <option value="MUTASI_HABIS_UPDATE">Mutasi Habis Update</option>
        <option value="MUTASI_HABIS_REGULER">Mutasi Habis Reguler</option>
        <option value="OBJEK_PAJAK_BARU">Objek Pajak Baru</option>
        <option value="PEMBETULAN">Pembetulan</option>
        <option value="PENGAKTIFAN">Pengaktifan</option>
      </select>

      <input type="number" {...register("luasBumi", { valueAsNumber: true })} placeholder="Luas Bumi" />
      {errors.luasBumi && <p>{errors.luasBumi.message}</p>}

      <input type="number" {...register("luasBangunan", { valueAsNumber: true })} placeholder="Luas Bangunan" />
      {errors.luasBangunan && <p>{errors.luasBangunan.message}</p>}

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? "Menyimpan..." : "Simpan Permohonan"}
      </button>
    </form>
  );
}
```

**Backend (Server Action — validasi ulang wajib, tidak boleh percaya validasi frontend saja):**

```typescript
// lib/actions/permohonan.actions.ts
"use server";

import { permohonanSchema } from "@/lib/validations/permohonan.schema";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";

export async function createPermohonanAction(input: unknown) {
  const session = await auth();
  if (!session || session.user.role !== "PENGINPUT") {
    throw new Error("Hanya PENGINPUT yang berwenang menginput Permohonan.");
  }

  const parsed = permohonanSchema.safeParse(input);
  if (!parsed.success) {
    throw new Error("Data tidak valid: " + parsed.error.message);
  }

  return prisma.permohonan.create({
    data: {
      ...parsed.data,
      status: "SUBMITTED",        // PRD Fase 1: status awal selalu SUBMITTED
      tanggalSubmit: new Date(),  // auto-generated di server, BUKAN dari input klien
      penginputId: session.user.id,
    },
  });
}
```

**Alasan validasi dilakukan di 2 layer:** Validasi frontend (Zod + React Hook Form) memberi umpan balik instan ke pengguna tanpa round-trip server. Namun karena Server Action dapat dipanggil langsung (bypass UI), validasi backend dengan Zod schema yang **sama persis** wajib dijalankan ulang sebagai sumber kebenaran final — sesuai prinsip "jangan pernah percaya input klien".

---

## 12. State Transition Matrix (Tabel Transisi Status)

### 12.1 Tabel Transisi — Permohonan

| Dari Status | Ke Status | Kondisi | Role |
|---|---|---|---|
| *(baru)* | `SUBMITTED` | Data awal diinput | `PENGINPUT` |
| `SUBMITTED` | `BUNDLED` | Dicentang masuk ke Bundle (Bundle `DRAFT`) | `PENELITI` |
| `SUBMITTED` | `REVISION` | Tombol "Minta Revisi" ditekan | `PENELITI` |
| `BUNDLED` | `SUBMITTED` | Fitur "Keluarkan dari Bundle" (Bundle masih `DRAFT`) | `PENELITI` |
| `BUNDLED` | `SUBMITTED` | Fitur "Unbundle" (dokumen fisik cacat saat pemindaian) | `PENGARSIP` |
| `BUNDLED` | `ARCHIVED` | Dokumen fisik selesai dipindai & diunggah | `PENGARSIP` |
| `REVISION` | `SUBMITTED` | Data diperbaiki/dilengkapi (via Update) | `PENGINPUT` |
| `REVISION` | `REJECTED` | Cron Job tutup buku tahunan (otomatis) | *Sistem* |
| `ARCHIVED` | `COMPLETED` | Produk layanan terbit, ditandai selesai | `PEMANTAU` |
| `COMPLETED` | `ARCHIVED` | Fitur "Batal Selesai" (Rollback) | `PEMANTAU` |

### 12.2 Tabel Transisi — Bundle

| Dari Status | Ke Status | Kondisi | Role |
|---|---|---|---|
| *(baru)* | `DRAFT` | Bundle baru dibuat | `PENELITI` |
| `DRAFT` | `LOCKED` | Surat Pengantar Bundle dicetak, fisik masuk map | `PENELITI` |
| `LOCKED` | `IN_MANIFEST` | Bundle dimasukkan ke Manifest | `PENGIRIM` |
| `IN_MANIFEST` | `LOCKED` | Fitur "Laporkan Bundle Hilang" (map hilang saat transit) | `PENGIRIM` |

### 12.3 Tabel Transisi — Manifest

| Dari Status | Ke Status | Kondisi | Role |
|---|---|---|---|
| *(baru)* | `DRAFT` | Manifest baru dibuat | `PENGIRIM` |
| `DRAFT` | `LOCKED` | Surat Pengantar Manifest dicetak, kardus disegel | `PENGIRIM` |
| `LOCKED` | `SENT` | Bukti Tanda Terima diunggah | `PENGIRIM` |
| `LOCKED` | `DRAFT` | Fitur "Revisi Manifest" (map tertinggal di laci) | `PENGIRIM` |

### 12.4 Implementasi: `validateTransition()`

```typescript
// lib/state-machine/validateTransition.ts
import type { Role } from "@prisma/client";

type Entity = "Permohonan" | "Bundle" | "Manifest";

interface TransitionRule {
  from: string;
  to: string;
  allowedRoles: Role[];
}

// Sumber kebenaran tunggal untuk seluruh transisi yang sah, sesuai Tabel 12.1-12.3
const TRANSITION_RULES: Record<Entity, TransitionRule[]> = {
  Permohonan: [
    { from: "SUBMITTED", to: "BUNDLED", allowedRoles: ["PENELITI"] },
    { from: "SUBMITTED", to: "REVISION", allowedRoles: ["PENELITI"] },
    { from: "BUNDLED", to: "SUBMITTED", allowedRoles: ["PENELITI", "PENGARSIP"] },
    { from: "BUNDLED", to: "ARCHIVED", allowedRoles: ["PENGARSIP"] },
    { from: "REVISION", to: "SUBMITTED", allowedRoles: ["PENGINPUT"] },
    { from: "REVISION", to: "REJECTED", allowedRoles: [] }, // hanya sistem/cron, lihat Bab 13
    { from: "ARCHIVED", to: "COMPLETED", allowedRoles: ["PEMANTAU"] },
    { from: "COMPLETED", to: "ARCHIVED", allowedRoles: ["PEMANTAU"] },
  ],
  Bundle: [
    { from: "DRAFT", to: "LOCKED", allowedRoles: ["PENELITI"] },
    { from: "LOCKED", to: "IN_MANIFEST", allowedRoles: ["PENGIRIM"] },
    { from: "IN_MANIFEST", to: "LOCKED", allowedRoles: ["PENGIRIM"] },
  ],
  Manifest: [
    { from: "DRAFT", to: "LOCKED", allowedRoles: ["PENGIRIM"] },
    { from: "LOCKED", to: "SENT", allowedRoles: ["PENGIRIM"] },
    { from: "LOCKED", to: "DRAFT", allowedRoles: ["PENGIRIM"] },
  ],
};

export class InvalidTransitionError extends Error {
  constructor(message: string) {
    super(message);
    this.name = "InvalidTransitionError";
  }
}

interface ValidateTransitionParams {
  entity: Entity;
  currentStatus: string;
  newStatus: string;
  role: Role | "SYSTEM";
}

export function validateTransition({ entity, currentStatus, newStatus, role }: ValidateTransitionParams): void {
  const rules = TRANSITION_RULES[entity];

  const matchedRule = rules.find((rule) => rule.from === currentStatus && rule.to === newStatus);

  if (!matchedRule) {
    throw new InvalidTransitionError(
      `Transisi ${entity} dari "${currentStatus}" ke "${newStatus}" tidak diizinkan.`
    );
  }

  // Transisi khusus sistem (Cron Job) tidak melalui pengecekan role user
  if (role === "SYSTEM") {
    return;
  }

  if (!matchedRule.allowedRoles.includes(role)) {
    throw new InvalidTransitionError(
      `Role "${role}" tidak berwenang melakukan transisi ${entity} dari "${currentStatus}" ke "${newStatus}".`
    );
  }
}
```

**Alasan:** Sentralisasi seluruh aturan transisi pada satu lookup table (`TRANSITION_RULES`) yang dipanggil sebelum setiap update memastikan tidak ada Server Action yang bisa "lupa" memvalidasi state machine, dan menjadikan tabel ini satu-satunya tempat yang perlu diaudit untuk memverifikasi konsistensi terhadap PRD.

---

## 13. Cron Job Specification

### 13.1 Detail Teknis: Aturan Tutup Buku Tahunan

Sesuai PRD 7.1, Cron Job dijalankan **setiap akhir tahun anggaran**, secara spesifik dijadwalkan pada **1 Januari pukul 00:00**, menggunakan Vercel Cron (sesuai tech stack yang ditentukan).

```json
// vercel.json
{
  "crons": [
    {
      "path": "/api/cron/tutup-buku-tahunan",
      "schedule": "0 0 1 1 *"
    }
  ]
}
```

`0 0 1 1 *` = menit 0, jam 0, tanggal 1, bulan Januari, setiap hari dalam minggu — yaitu tepat 1 Januari pukul 00:00.

### 13.2 Route Handler: Update Massal + Audit Log

```typescript
// app/api/cron/tutup-buku-tahunan/route.ts
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";

export async function GET(request: NextRequest) {
  // Verifikasi request berasal dari Vercel Cron (bukan endpoint publik bebas akses)
  const authHeader = request.headers.get("authorization");
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  // 1. Ambil seluruh Permohonan berstatus REVISION SEBELUM diubah,
  //    untuk keperluan snapshot oldValue/newValue di Audit Log
  const permohonanRevisi = await prisma.permohonan.findMany({
    where: { status: "REVISION", deletedAt: null },
    select: { id: true, status: true },
  });

  if (permohonanRevisi.length === 0) {
    return NextResponse.json({ message: "Tidak ada Permohonan REVISION untuk diproses.", count: 0 });
  }

  // 2. Update massal: REVISION -> REJECTED (PRD 7.1)
  const updateResult = await prisma.permohonan.updateMany({
    where: { status: "REVISION", deletedAt: null },
    data: { status: "REJECTED", version: { increment: 1 } },
  });

  // 3. Batch insert ke AuditLog untuk setiap Permohonan yang berubah
  await prisma.auditLog.createMany({
    data: permohonanRevisi.map((p) => ({
      entityType: "Permohonan" as const,
      entityId: p.id,
      actorId: process.env.SYSTEM_ACTOR_ID as string, // user sistem khusus untuk aksi otomatis
      actorRole: "PEMANTAU" as const, // placeholder teknis: lihat catatan di bawah
      action: "TUTUP_BUKU_TAHUNAN_AUTO_REJECT",
      oldValue: { status: "REVISION" },
      newValue: { status: "REJECTED" },
    })),
  });

  return NextResponse.json({
    message: "Tutup buku tahunan selesai.",
    count: updateResult.count,
  });
}
```

> **Catatan teknis murni (bukan keputusan bisnis baru):** Enum `Role` pada PRD hanya berisi 5 nilai (`PENGINPUT`, `PENELITI`, `PENGARSIP`, `PENGIRIM`, `PEMANTAU`) yang seluruhnya merepresentasikan aktor manusia per fase. PRD 7.1 menyatakan aksi ini dilakukan **otomatis oleh sistem**, bukan oleh salah satu dari 5 role tersebut. Karena field `actorRole` pada `AuditLog` bertipe enum `Role` yang tidak memiliki nilai "SYSTEM", implementasi di atas memerlukan sebuah `User` akun sistem (`SYSTEM_ACTOR_ID`) yang representasi `role`-nya harus dipilih dari salah satu nilai enum yang ada secara teknis administratif untuk keperluan pencatatan — ini murni solusi teknis penyimpanan data, bukan penambahan role atau aktor bisnis baru ke dalam alur kerja PRD.

### 13.3 Validasi Transisi pada Cron Job

```typescript
import { validateTransition } from "@/lib/state-machine/validateTransition";

// Dipanggil per-record sebelum batch update jika validasi individual diperlukan:
permohonanRevisi.forEach((p) => {
  validateTransition({
    entity: "Permohonan",
    currentStatus: p.status,
    newStatus: "REJECTED",
    role: "SYSTEM",
  });
});
```

**Alasan memakai Vercel Cron, bukan node-cron/Inngest:** Aplikasi sudah di-deploy di atas Next.js App Router pada infrastruktur serverless (implikasi dari pemilihan Vercel Blob di Bab 6); `node-cron` membutuhkan proses long-running yang tidak kompatibel dengan model serverless, sementara Inngest menambah dependensi infrastruktur eksternal yang tidak diperlukan untuk satu jadwal tahunan tunggal. Vercel Cron adalah solusi paling sederhana yang native terhadap stack yang sudah dipilih.
