# UI/UX Specification — Architax
## Addendum UI/UX dari `ARC_PRD.md` dan `ARC_TECHNICAL_SPEC.md`

> **Dokumen ini adalah turunan UI/UX murni dari PRD final dan Technical Spec final.** Tidak ada halaman, fitur, tombol aksi, atau role baru yang ditambahkan di luar yang sudah didefinisikan pada kedua dokumen sumber tersebut.

| Atribut | Keterangan |
|---|---|
| **Dokumen Sumber** | `ARC_PRD.md` (Final), `ARC_TECHNICAL_SPEC.md` (Final) |
| **Status** | Final |
| **Konsumen** | AI Coding Assistant (Cursor / Claude Code) & Tim Frontend Engineering |
| **Tanggal** | 22 Juni 2026 |

---

## ⚠️ Catatan Penting: Sumber Kebenaran Token Visual

Selama penyusunan dokumen ini ditemukan **`UI_DESIGN_SYSTEM.md` v2.0** — sebuah design system internal Architax yang sudah ada dan berstatus *"single source of truth for the visual language of ARCHITAX"*. Dokumen tersebut mendefinisikan token visual (warna, spacing, radius, shadow, motion, komponen) secara lengkap dan eksplisit menyatakan nilai-nilainya *"should be treated as production-ready defaults... never overridden ad hoc."*

Beberapa nilai token visual yang diminta dalam brief penyusunan dokumen ini **berbeda** dari `UI_DESIGN_SYSTEM.md`, contoh:

| Aspek | Diminta di Brief | `UI_DESIGN_SYSTEM.md` (v2.0, sumber resmi) |
|---|---|---|
| Warna primary | Navy blue `#1e3a8a` | Violet/purple `#6C5CE0` (`color-primary`) |
| Sidebar gradient | Light blue-lavender `#dbeafe → #e0e7ff` | Cyan-violet `#6FD8E0 → #B6A4F0` (`color-bg-sidebar-start/end`) |
| Lebar sidebar | 240px (collapsed 64px) | 264px (collapsed 72px) |
| Tinggi header | 64px | 76px |
| Radius card | 8–12px | 16px |
| Warna semantik | Hex generik (Tailwind default) | Hex terkalibrasi khusus dengan varian `-tint` (`color-success`, `color-warning`, `color-danger`, `color-info`) |

**Keputusan:** Dokumen `ARC_UI_SPEC.md` ini **mengikuti `UI_DESIGN_SYSTEM.md` v2.0 sebagai sumber kebenaran tunggal untuk seluruh token visual** (warna, tipografi, spacing, radius, shadow, motion, anatomi komponen), karena dokumen tersebut secara eksplisit memposisikan dirinya sebagai *single source of truth* yang sudah disetujui dan dipelihara oleh tim Design & Frontend Engineering. Nilai dari brief (navy blue, dimensi spesifik di atas) **tidak digunakan** untuk menghindari dua sistem token yang saling bertentangan berjalan paralel dalam satu produk.

Pemetaan status entitas ke kategori semantik (REVISION=warning, SUBMITTED=info, COMPLETED=success, REJECTED=danger, dst.) tetap mengikuti maksud brief — hanya **nilai hex literalnya** yang digantikan oleh token semantik resmi (`color-warning`, `color-info`, `color-success`, `color-danger`) dari `UI_DESIGN_SYSTEM.md`. Lihat Bab 1.4 untuk pemetaan lengkap.

Seluruh struktur halaman, role, status, dan aturan State-Driven UI pada dokumen ini tetap murni diturunkan dari `ARC_PRD.md` dan `ARC_TECHNICAL_SPEC.md` tanpa perubahan substansi bisnis.

---

## ⚠️ Catatan Revisi v2: Kalibrasi Visual terhadap Screenshot Referensi

Revisi ini dipicu oleh perbandingan piksel langsung antara draf v1 dokumen ini dan sebuah screenshot referensi (dashboard "Home" bergaya kartu favorit + daftar tugas). Sample warna RGB diambil pada titik-titik kunci (header, canvas, panel konten, ubin warna) untuk memverifikasi token v1 terhadap referensi tersebut. Tiga ketidaksesuaian struktural ditemukan dan diperbaiki di revisi ini; **tidak ada halaman, fitur, tombol aksi, atau role baru** yang ditambahkan — seluruh perubahan murni di lapisan presentasi/visual dari data dan aksi yang sudah didefinisikan di `ARC_PRD.md` / `ARC_TECHNICAL_SPEC.md` dan draf v1.

| # | Temuan | v1 (sebelum) | v2 (revisi ini) |
|---|---|---|---|
| 1 | Header/navbar di referensi **tidak punya warna terpisah** dari canvas — sample piksel di area header dan area konten sama-sama `#EEF0F2` | `--color-bg-navbar: #FFFFFF` (putih solid, beda dari canvas) | `--color-bg-navbar` disetel identik dengan `--color-bg-canvas` (`#EEF0F2`) — header menyatu visual dengan konten, tidak ada garis batas |
| 2 | Setiap section di Home pada referensi **tidak dibungkus card putih** — tidak ada border/shadow, konten duduk langsung di atas canvas, hanya garis pemisah tipis di bawah judul | Tiap section Home dibungkus `<Card>` shadcn: putih, border 1px, shadow Level 1 | Diperkenalkan `<DashboardPanel>` baru (§3.5) — *card-less*, transparan, hanya `border-b` tipis di bawah header |
| 3 | Pola 4-zona Home di referensi **berbeda jenis komponen per zona** (panel daftar, ubin warna solid, panel daftar lain, panel feed) — bukan 4 kartu angka besar yang seragam | Bab 4: seluruh kartu Home (semua role) berpola sama — "angka besar + subtitle + 1 link" | Bab 4 ditulis ulang: 4 zona dengan tipe komponen berbeda (`<ListRowsPanel>`, `<QuickAccessTilesPanel>`, `<ActivityFeedPanel>`) — **data & rute yang ditampilkan tetap persis sama** dengan v1 (status, count, link target identik), hanya bentuk presentasinya yang berubah |

Token semantik (`color-primary`, `color-success`, dst.) dari `UI_DESIGN_SYSTEM.md` v2.0 tetap dipertahankan sebagai sumber kebenaran warna kategori (lihat catatan addendum di atas) — revisi ini **tidak mengganti palet semantik**, hanya mengoreksi token layout (`bg-navbar`, `bg-canvas`) dan struktur komponen Home agar sesuai bukti visual referensi.

---

## 1. Design System & Global Rules

### 1.1 Sumber Token

Seluruh token CSS custom property / Tailwind theme extension diimplementasikan persis sebagaimana didefinisikan di `UI_DESIGN_SYSTEM.md` Bab 3–8. Ringkasan token yang dipakai berulang di seluruh spesifikasi ini:

```css
/* Core Palette — diambil dari UI_DESIGN_SYSTEM.md §3.1 */
--color-primary: #6C5CE0;
--color-primary-hover: #5B4BCB;
--color-primary-tint: #EDE9FB;
--color-secondary: #34C7B8;

--color-bg-canvas: #EEF0F2;
--color-bg-surface: #FFFFFF;
--color-bg-sidebar-start: #6FD8E0;
--color-bg-sidebar-end: #B6A4F0;
/* color-bg-navbar SENGAJA disetel identik dengan color-bg-canvas (v2 — lihat Catatan
   Revisi v2 di atas). Header tidak boleh punya warna terpisah dari main content;
   keduanya satu bidang visual tanpa garis batas, sesuai screenshot referensi. */
--color-bg-navbar: #EEF0F2;

--color-hover: #F6F5FC;
--color-active: #EDE9FB;
--color-border: #E7E6F0;
--color-divider: #EFEEF6;

--color-text-primary: #211E33;
--color-text-secondary: #8B87A0;
--color-text-placeholder: #B5B1C6;
--color-text-disabled: #C7C4D6;

--color-success: #2FBE8F;
--color-success-tint: #E2F8F0;
--color-warning: #F0A93B;
--color-warning-tint: #FDF1DC;
--color-danger: #F0507F;
--color-danger-tint: #FDE6ED;
--color-info: #6C5CE0;
--color-info-tint: #EDE9FB;
--color-focus-ring: rgba(108, 92, 224, 0.35);
```

**Catatan v2:** tidak ada token warna baru yang ditambahkan. Komponen baru di v2 (count badge, ubin akses cepat — lihat §3.5) **memakai ulang token semantik di atas** (`color-danger`, `color-warning`, `color-info`, `color-success`, `color-primary`, `color-secondary`), hanya konteks pemakaiannya yang baru.

```css
/* Typography — UI_DESIGN_SYSTEM.md §4 */
font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
```

Tailwind config disetel agar `font-sans` default memakai Inter, dan seluruh palet warna di atas didaftarkan sebagai extend pada `tailwind.config.ts` (bukan hex literal di JSX), konsisten dengan instruksi `UI_DESIGN_SYSTEM.md` §24 bahwa nilai *"never [dihardcode] inside individual screens or components."*

### 1.2 shadcn/ui sebagai Lapisan Komponen

Seluruh komponen dasar (`Button`, `Table`, `Dialog`, `Form`, `Badge`, `Card`, sidebar primitives) dibangun di atas shadcn/ui sesuai instruksi tech stack, dengan `theme` shadcn di-override agar memetakan ke token `UI_DESIGN_SYSTEM.md` (radius, warna, shadow) — bukan default shadcn neutral/slate. Contoh pemetaan:

| shadcn token | Nilai Architax |
|---|---|
| `--radius` | `16px` (untuk Card/Modal), `8px` (untuk Button/Input) — lihat detail per-komponen di Bab 6 |
| `--primary` | `#6C5CE0` |
| `--destructive` | `#F0507F` |
| `--muted` | `#F6F5FC` (`color-hover`) |
| `--border` | `#E7E6F0` |

### 1.3 Ikon: Lucide React

Seluruh ikon memakai Lucide React, stroke-based outline, stroke-width 1.75px pada ukuran default, mengikuti `UI_DESIGN_SYSTEM.md` §16. Ukuran ikon: 16px (konteks dense seperti aksi tabel), 18–20px (default — nav item, tombol, field form), 24px (empty state, ikon status besar pada kartu dashboard).

### 1.4 State-Driven Badges — Pemetaan Status ke Token Semantik

Mengikuti maksud brief (status mendapat warna semantik konsisten), nilai hex digantikan token resmi `UI_DESIGN_SYSTEM.md`:

**Permohonan** (`StatusPermohonan`, sesuai PRD Bab 4.1 / Tech Spec Bab 2):

| Status | Kategori Semantik | Token Badge |
|---|---|---|
| `SUBMITTED` | Info | `bg-[--color-info-tint] text-[--color-info]` |
| `REVISION` | Warning | `bg-[--color-warning-tint] text-[--color-warning]` |
| `BUNDLED` | Info (varian netral-primer, karena tidak ada token "purple" terpisah di `UI_DESIGN_SYSTEM.md`; `color-primary` dipakai sebagai pembeda visual dari `SUBMITTED`) | `bg-[--color-primary-tint] text-[--color-primary]` |
| `ARCHIVED` | Secondary (token `color-secondary`, sebagai pembeda kategori "dalam proses lanjutan" dari `BUNDLED`/`SUBMITTED`) | `bg-[--color-secondary]/10 text-[--color-secondary]` |
| `COMPLETED` | Success | `bg-[--color-success-tint] text-[--color-success]` |
| `REJECTED` | Danger | `bg-[--color-danger-tint] text-[--color-danger]` |

**Bundle** (`StatusBundle`, PRD Bab 4.2):

| Status | Kategori Semantik | Token Badge |
|---|---|---|
| `DRAFT` | Warning | `bg-[--color-warning-tint] text-[--color-warning]` |
| `LOCKED` | Info | `bg-[--color-info-tint] text-[--color-info]` |
| `IN_MANIFEST` | Info (varian primer, pembeda dari `LOCKED`) | `bg-[--color-primary-tint] text-[--color-primary]` |

**Manifest** (`StatusManifest`, PRD Bab 4.3):

| Status | Kategori Semantik | Token Badge |
|---|---|---|
| `DRAFT` | Warning | `bg-[--color-warning-tint] text-[--color-warning]` |
| `LOCKED` | Info | `bg-[--color-info-tint] text-[--color-info]` |
| `SENT` | Success | `bg-[--color-success-tint] text-[--color-success]` |

Implementasi komponen `<Badge>` mengikuti anatomi `UI_DESIGN_SYSTEM.md` §15: pill shape (radius 999px), padding 4px vertikal / 10px horizontal, teks 12px/600, background memakai varian `-tint`, teks/ikon memakai warna semantik penuh — **tidak pernah** warna saturasi penuh sebagai fill besar.

**Pengecualian v2 — `<QuickAccessTile>` (lihat §3.5):** satu-satunya komponen yang sengaja memakai warna semantik **saturasi penuh** sebagai fill background adalah ubin akses cepat pada zona kedua Halaman Home, mengikuti bukti visual referensi (ubin Favorites memakai fill warna solid, bukan tint). Pengecualian ini dibatasi ketat pada komponen tersebut saja — seluruh `<Badge>`, alert inline, dan komponen lain di luar `<QuickAccessTile>` tetap wajib memakai varian `-tint` sesuai aturan di atas.

```tsx
// components/ui/status-badge.tsx
import { Badge } from "@/components/ui/badge";
import { cn } from "@/lib/utils";

type PermohonanStatus = "SUBMITTED" | "REVISION" | "BUNDLED" | "ARCHIVED" | "COMPLETED" | "REJECTED";
type BundleStatus = "DRAFT" | "LOCKED" | "IN_MANIFEST";
type ManifestStatus = "DRAFT" | "LOCKED" | "SENT";

const PERMOHONAN_STATUS_STYLE: Record<PermohonanStatus, string> = {
  SUBMITTED: "bg-[--color-info-tint] text-[--color-info]",
  REVISION: "bg-[--color-warning-tint] text-[--color-warning]",
  BUNDLED: "bg-[--color-primary-tint] text-[--color-primary]",
  ARCHIVED: "bg-[--color-secondary]/10 text-[--color-secondary]",
  COMPLETED: "bg-[--color-success-tint] text-[--color-success]",
  REJECTED: "bg-[--color-danger-tint] text-[--color-danger]",
};

const PERMOHONAN_STATUS_LABEL: Record<PermohonanStatus, string> = {
  SUBMITTED: "Submitted",
  REVISION: "Revisi",
  BUNDLED: "Bundled",
  ARCHIVED: "Archived",
  COMPLETED: "Selesai",
  REJECTED: "Ditolak",
};

export function PermohonanStatusBadge({ status }: { status: PermohonanStatus }) {
  return (
    <Badge className={cn("rounded-full px-2.5 py-1 text-xs font-semibold", PERMOHONAN_STATUS_STYLE[status])}>
      {PERMOHONAN_STATUS_LABEL[status]}
    </Badge>
  );
}
```

(Pola yang sama diterapkan untuk `BundleStatusBadge` dan `ManifestStatusBadge` dengan tabel pemetaan di atas.)

### 1.5 Layout Global

Mengikuti `UI_DESIGN_SYSTEM.md` §5 dan §9–10:

- **Sidebar:** fixed left, 264px expanded / 72px collapsed (icon-only), background gradient diagonal 135° `color-bg-sidebar-start → color-bg-sidebar-end`, logo Architax di atas, item menu dengan ikon Lucide 20px.
- **Top Navigation (Header):** fixed top, tinggi 76px, background **identik dengan `color-bg-canvas`** (v2 — bukan putih terpisah; lihat Catatan Revisi v2), tidak ada border atau shadow pemisah dari konten di bawahnya, berisi judul halaman (Display style, 30px/700) di kiri, dan klaster utilitas (search → notifikasi → avatar) di kanan dengan gap 24px. Hanya elemen di dalamnya (search bar, avatar) yang punya bidang putih `color-bg-surface` — bukan bar header itu sendiri.
- **Main Content:** margin-left menyesuaikan lebar sidebar aktif, margin-top 76px, padding 24px, background `color-bg-canvas` (`#EEF0F2`) — satu bidang visual yang sama dengan header di atasnya.
- **Application shell:** max-width 1600px (center pada layar ultra-wide), shell duduk di atas page canvas dengan shadow Level 2-equivalent.

---

## 2. Halaman Login

**Tujuan Halaman:** Autentikasi user sebelum mengakses dashboard sesuai role (Tech Spec Bab 10 — NextAuth.js Credentials Provider).

**Deskripsi Visual:**
- `<Card>` terpusat secara vertikal & horizontal di atas `color-bg-canvas`, radius 16px, shadow Level 1, border 1px `color-border`, max-width ±400px.
- Logo Architax di bagian atas card, margin-bottom 24px.
- Form berisi dua `<Input>` (email, password) memakai anatomi `UI_DESIGN_SYSTEM.md` §13 (tinggi 40px, radius 8px, border `color-border`, focus state border `color-primary` + glow `color-focus-ring`).
- Tombol "Masuk" — `<Button>` variant **Primary** (background `color-primary`, full width), satu-satunya tombol primary pada halaman ini sesuai prinsip "satu primary action per layar".
- Link teks "Lupa password?" di bawah tombol, style Ghost-text, warna `color-primary`, 13px/600.

**Validasi Error:**
- Border input menjadi `color-danger`.
- Teks helper error 12px, warna `color-danger`, muncul 4px di bawah field, diawali ikon peringatan kecil (Lucide `AlertCircle`, 14px) — sesuai `UI_DESIGN_SYSTEM.md` §13 baris "Validation — error".

```tsx
// app/(auth)/login/page.tsx
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";
import { signIn } from "next-auth/react";
import { Card, CardContent, CardHeader } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { Label } from "@/components/ui/label";

const loginSchema = z.object({
  email: z.string().email("Format email tidak valid."),
  password: z.string().min(1, "Password wajib diisi."),
});

type LoginInput = z.infer<typeof loginSchema>;

export default function LoginPage() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<LoginInput>({ resolver: zodResolver(loginSchema) });

  const onSubmit = async (data: LoginInput) => {
    await signIn("credentials", { ...data, callbackUrl: "/" });
  };

  return (
    <div className="flex min-h-screen items-center justify-center bg-[--color-bg-canvas]">
      <Card className="w-full max-w-[400px] rounded-2xl border border-[--color-border] shadow-sm">
        <CardHeader className="flex flex-col items-center gap-2 pb-2">
          <img src="/logo-architax.svg" alt="Architax" className="h-10" />
        </CardHeader>
        <CardContent>
          <form onSubmit={handleSubmit(onSubmit)} className="flex flex-col gap-5">
            <div>
              <Label htmlFor="email">Email</Label>
              <Input id="email" type="email" {...register("email")} className="mt-2 h-10 rounded-lg" />
              {errors.email && (
                <p className="mt-1 text-xs text-[--color-danger]">{errors.email.message}</p>
              )}
            </div>
            <div>
              <Label htmlFor="password">Password</Label>
              <Input id="password" type="password" {...register("password")} className="mt-2 h-10 rounded-lg" />
              {errors.password && (
                <p className="mt-1 text-xs text-[--color-danger]">{errors.password.message}</p>
              )}
            </div>
            <Button
              type="submit"
              disabled={isSubmitting}
              className="h-10 w-full rounded-lg bg-[--color-primary] font-semibold text-white hover:bg-[--color-primary-hover]"
            >
              {isSubmitting ? "Memproses..." : "Masuk"}
            </Button>
            <a href="/forgot-password" className="text-center text-[13px] font-semibold text-[--color-primary]">
              Lupa password?
            </a>
          </form>
        </CardContent>
      </Card>
    </div>
  );
}
```

---

## 3. Layout Template (Shared Across All Roles)

Sesuai instruksi brief poin 5: **semua role memiliki layout yang sama persis** — Sidebar + Header + Main Content dengan token visual identik. Hanya **isi menu** sidebar dan **isi grid Home** yang berbeda per role (Bab 4), bukan struktur atau warnanya.

### 3.1 `app/(dashboard)/layout.tsx`

```tsx
// app/(dashboard)/layout.tsx
import { Sidebar } from "@/components/layout/Sidebar";
import { Header } from "@/components/layout/Header";
import { auth } from "@/lib/auth";
import { redirect } from "next/navigation";

export default async function DashboardLayout({ children }: { children: React.ReactNode }) {
  const session = await auth();
  if (!session) redirect("/login");

  return (
    <div className="min-h-screen bg-[--color-bg-canvas]">
      <Sidebar role={session.user.role} />
      <div className="ml-[264px] flex min-h-screen flex-col transition-[margin] duration-200 ease-in-out peer-data-[collapsed=true]:ml-[72px]">
        <Header user={session.user} />
        <main className="flex-1 p-6">{children}</main>
      </div>
    </div>
  );
}
```

### 3.2 `components/layout/Sidebar.tsx`

Navigasi berbasis role. Menu **"Home" wajib selalu berada paling atas** (instruksi brief poin 6), diikuti menu sesuai halaman-halaman yang berwenang diakses role tersebut (lihat pemetaan lengkap di Bab 5). Mengikuti anatomi `UI_DESIGN_SYSTEM.md` §9: item aktif memakai pill putih solid (radius 10px, shadow Level 1), item non-aktif memakai hover overlay translucent putih.

```tsx
// components/layout/Sidebar.tsx
"use client";

import { useState } from "react";
import Link from "next/link";
import { usePathname } from "next/navigation";
import { cn } from "@/lib/utils";
import {
  Home, FileInput, FolderOpen, ScanLine, Truck, Gauge, ChevronLeft,
} from "lucide-react";
import type { Role } from "@prisma/client";

interface NavItem {
  label: string;
  href: string;
  icon: React.ElementType;
}

// Pemetaan menu per role — mengikuti SATU-SATUNYA halaman yang berwenang diakses
// masing-masing role sesuai PRD Bab 3 & Tech Spec Bab 1.3 / Bab 10.3.
const ROLE_NAV_MAP: Record<Role, NavItem[]> = {
  PENGINPUT: [
    { label: "Home", href: "/", icon: Home },
    { label: "Permohonan", href: "/permohonan", icon: FileInput },
  ],
  PENELITI: [
    { label: "Home", href: "/", icon: Home },
    { label: "Permohonan", href: "/permohonan", icon: FileInput },
    { label: "Bundle", href: "/bundle", icon: FolderOpen },
  ],
  PENGARSIP: [
    { label: "Home", href: "/", icon: Home },
    { label: "Bundle", href: "/bundle", icon: FolderOpen },
    { label: "Arsip", href: "/arsip", icon: ScanLine },
  ],
  PENGIRIM: [
    { label: "Home", href: "/", icon: Home },
    { label: "Bundle", href: "/bundle", icon: FolderOpen },
    { label: "Manifest", href: "/manifest", icon: Truck },
  ],
  PEMANTAU: [
    { label: "Home", href: "/", icon: Home },
    { label: "Pemantauan", href: "/pemantauan", icon: Gauge },
  ],
};

export function Sidebar({ role }: { role: Role }) {
  const pathname = usePathname();
  const [collapsed, setCollapsed] = useState(false);
  const items = ROLE_NAV_MAP[role];

  return (
    <aside
      data-collapsed={collapsed}
      className={cn(
        "peer fixed left-0 top-0 z-30 flex h-screen flex-col px-4 pt-6 transition-[width] duration-200 ease-in-out",
        "bg-[linear-gradient(135deg,_var(--color-bg-sidebar-start),_var(--color-bg-sidebar-end))]",
        collapsed ? "w-[72px]" : "w-[264px]"
      )}
      role="navigation"
    >
      <div className="mb-7 flex items-center justify-between px-2">
        {!collapsed && <img src="/logo-architax-white.svg" alt="Architax" className="h-7" />}
        <button
          onClick={() => setCollapsed((c) => !c)}
          aria-label={collapsed ? "Perluas sidebar" : "Ciutkan sidebar"}
          className="rounded-md p-1 hover:bg-white/35"
        >
          <ChevronLeft className={cn("h-4 w-4 transition-transform", collapsed && "rotate-180")} />
        </button>
      </div>

      <nav className="flex flex-col gap-1">
        {items.map((item) => {
          const isActive = pathname === item.href || pathname.startsWith(item.href + "/");
          const Icon = item.icon;
          return (
            <Link
              key={item.href}
              href={item.href}
              aria-current={isActive ? "page" : undefined}
              className={cn(
                "flex h-10 items-center gap-3 rounded-[10px] px-3 text-sm transition-colors",
                isActive
                  ? "bg-[--color-bg-surface] font-semibold text-[--color-primary] shadow-sm"
                  : "font-medium text-[--color-text-primary] hover:bg-white/35"
              )}
            >
              <Icon className="h-5 w-5 shrink-0" strokeWidth={1.75} />
              {!collapsed && <span>{item.label}</span>}
            </Link>
          );
        })}
      </nav>
    </aside>
  );
}
```

### 3.3 `components/layout/Header.tsx`

```tsx
// components/layout/Header.tsx
"use client";

import { usePathname } from "next/navigation";
import { Bell, Search } from "lucide-react";
import { Avatar, AvatarFallback } from "@/components/ui/avatar";
import { Badge } from "@/components/ui/badge";

const PAGE_TITLE_MAP: Record<string, string> = {
  "/": "Home",
  "/permohonan": "Permohonan",
  "/bundle": "Bundle",
  "/arsip": "Arsip",
  "/manifest": "Manifest",
  "/pemantauan": "Pemantauan",
};

export function Header({ user }: { user: { name: string; role: string } }) {
  const pathname = usePathname();
  const title =
    Object.entries(PAGE_TITLE_MAP).find(([path]) => pathname === path || pathname.startsWith(path + "/"))?.[1] ??
    "Architax";

  return (
    // v2: bg-[--color-bg-navbar] kini identik dengan bg-[--color-bg-canvas] (lihat §1.1) —
    // tidak ada border/shadow pemisah, header menyatu visual dengan main content.
    <header className="sticky top-0 z-20 flex h-[76px] items-center justify-between bg-[--color-bg-navbar] px-6">
      <h1 className="text-[30px] font-bold leading-tight text-[--color-text-primary]">{title}</h1>

      <div className="flex items-center gap-6">
        <div className="flex h-10 w-[360px] items-center gap-2 rounded-full bg-[--color-bg-surface] px-4 focus-within:w-[480px] focus-within:ring-[3px] focus-within:ring-[--color-focus-ring] transition-all">
          <Search className="h-4 w-4 text-[--color-text-placeholder]" />
          <input
            placeholder="Cari..."
            className="w-full bg-transparent text-sm text-[--color-text-primary] placeholder:text-[--color-text-placeholder] focus:outline-none"
          />
        </div>

        <button aria-label="Notifikasi" className="relative">
          <Bell className="h-5 w-5 text-[--color-text-secondary]" strokeWidth={1.75} />
        </button>

        <div className="flex items-center gap-3">
          <Avatar className="h-9 w-9">
            <AvatarFallback className="bg-[--color-primary] text-sm font-semibold text-white">
              {user.name.slice(0, 2).toUpperCase()}
            </AvatarFallback>
          </Avatar>
          <div className="flex flex-col leading-tight">
            <span className="text-sm font-semibold text-[--color-text-primary]">{user.name}</span>
            <Badge className="w-fit rounded-full bg-[--color-primary-tint] px-2 py-0 text-[11px] font-semibold text-[--color-primary]">
              {user.role}
            </Badge>
          </div>
        </div>
      </div>
    </header>
  );
}
```

### 3.4 `components/layout/DashboardGrid.tsx`

Container untuk 4 zona (2×2 layout) yang dipakai di halaman Home seluruh role (Bab 4), gap 24px sesuai `UI_DESIGN_SYSTEM.md` §11 "Spacing in grid". **v2:** masing-masing zona kini diisi `<DashboardPanel>` (§3.5) yang heterogen bentuknya (daftar / ubin / feed) — bukan 4 `<Card>` seragam seperti v1; kode container ini sendiri tidak berubah karena sudah generik.

```tsx
// components/layout/DashboardGrid.tsx
export function DashboardGrid({ children }: { children: React.ReactNode }) {
  return (
    <div className="grid grid-cols-1 gap-6 md:grid-cols-2">{children}</div>
  );
}
```

### 3.5 Komponen Panel Dashboard — *Card-less* (v2)

Empat komponen berikut menggantikan pola "kartu angka besar seragam" dari v1 (lihat Catatan Revisi v2). Prinsip bersama: **tidak ada `<Card>` pembungkus** — tidak ada `bg-surface`, `border`, atau `shadow`; setiap panel duduk langsung di atas `color-bg-canvas`, hanya dipisahkan oleh `border-b border-[--color-divider]` tipis di bawah header-nya, sesuai bukti piksel pada screenshot referensi. Data, status, dan rute (`href`) yang ditampilkan oleh tiap panel **identik dengan yang sudah didefinisikan di v1** — lihat pemetaan per-role di Bab 4.

#### 3.5.1 `<DashboardPanel>` — wrapper header (badge + link + menu opsi)

Header generik dipakai oleh seluruh 4 zona: judul + count badge (opsional, memakai token semantik §1.4) + 1–2 link teks (`color-primary`, 13px/600 — rute yang sama dengan v1) + ikon menu titik-tiga (`MoreHorizontal`, opsional, **murni kosmetik**: dropdown berisi opsi pengurutan client-side seperti "Terbaru"/"Terlama" pada data yang sudah ditampilkan — bukan aksi atau halaman baru).

```tsx
// components/dashboard/DashboardPanel.tsx
"use client";

import Link from "next/link";
import { MoreHorizontal } from "lucide-react";
import {
  DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu";
import { cn } from "@/lib/utils";

interface DashboardPanelProps {
  title: string;
  countBadge?: { value: number; tone?: "danger" | "warning" | "info" | "success" };
  links?: { label: string; href: string }[];
  sortOptions?: string[]; // opsional, kosmetik — lihat catatan di atas
  children: React.ReactNode;
}

const BADGE_TONE: Record<string, string> = {
  danger: "bg-[--color-danger] text-white",
  warning: "bg-[--color-warning] text-white",
  info: "bg-[--color-info] text-white",
  success: "bg-[--color-success] text-white",
};

export function DashboardPanel({ title, countBadge, links = [], sortOptions, children }: DashboardPanelProps) {
  return (
    <section className="flex flex-col">
      <div className="flex items-center justify-between border-b border-[--color-divider] pb-3">
        <div className="flex items-center gap-2">
          <h2 className="text-lg font-semibold text-[--color-text-primary]">{title}</h2>
          {countBadge && (
            <span
              className={cn(
                "flex h-5 min-w-5 items-center justify-center rounded-full px-1.5 text-[11px] font-bold",
                BADGE_TONE[countBadge.tone ?? "danger"]
              )}
            >
              {countBadge.value}
            </span>
          )}
        </div>
        <div className="flex items-center gap-4">
          {links.map((l) => (
            <Link key={l.href} href={l.href} className="text-[13px] font-semibold text-[--color-primary] hover:underline">
              {l.label}
            </Link>
          ))}
          {sortOptions && sortOptions.length > 0 && (
            <DropdownMenu>
              <DropdownMenuTrigger aria-label="Opsi tampilan" className="rounded-md p-1 hover:bg-[--color-hover]">
                <MoreHorizontal className="h-4 w-4 text-[--color-text-secondary]" strokeWidth={1.75} />
              </DropdownMenuTrigger>
              <DropdownMenuContent align="end">
                {sortOptions.map((opt) => (
                  <DropdownMenuItem key={opt}>{opt}</DropdownMenuItem>
                ))}
              </DropdownMenuContent>
            </DropdownMenu>
          )}
        </div>
      </div>
      <div className="pt-4">{children}</div>
    </section>
  );
}
```

#### 3.5.2 `<ListRowsPanel>` — daftar baris (dipakai Zona A & Zona C di Bab 4)

Padanan visual dari "Tasks Due" dan "Recent Projects" pada referensi: baris-baris item dipisah `divide-y` tipis (bukan border tabel penuh), tanpa zebra-striping. Memakai `<PermohonanStatusBadge>` / `<BundleStatusBadge>` / `<ManifestStatusBadge>` yang sudah ada (§1.4) untuk kolom badge.

```tsx
// components/dashboard/ListRowsPanel.tsx
import { type LucideIcon } from "lucide-react";
import { cn } from "@/lib/utils";

interface ListRow {
  id: string;
  title: string;
  meta: string; // contoh: "Tanggal Submit: 29 Jan" atau "Jenis: Mutasi Sebagian"
  badgeLabel?: string;
  badgeClassName?: string; // token dari §1.4, mis. PERMOHONAN_STATUS_STYLE[status]
}

export function ListRowsPanel({ rows, icon: Icon }: { rows: ListRow[]; icon?: LucideIcon }) {
  return (
    <div className="flex flex-col divide-y divide-[--color-divider]">
      {rows.map((row) => (
        <div key={row.id} className="flex items-center justify-between py-3">
          <div className="flex items-center gap-3">
            {Icon && <Icon className="h-4 w-4 shrink-0 text-[--color-text-secondary]" strokeWidth={1.75} />}
            <div className="flex flex-col">
              <span className="text-sm font-medium text-[--color-text-primary]">{row.title}</span>
              <span className="text-xs text-[--color-text-secondary]">{row.meta}</span>
            </div>
          </div>
          {row.badgeLabel && (
            <span className={cn("rounded-full px-2.5 py-1 text-xs font-semibold", row.badgeClassName)}>
              {row.badgeLabel}
            </span>
          )}
        </div>
      ))}
      {rows.length === 0 && (
        <p className="py-6 text-center text-sm text-[--color-text-secondary]">Tidak ada data.</p>
      )}
    </div>
  );
}
```

#### 3.5.3 `<QuickAccessTilesPanel>` — ubin warna solid (dipakai Zona B di Bab 4)

Padanan visual dari "Favorites" pada referensi — **satu-satunya tempat** warna semantik dipakai sebagai fill solid penuh (lihat pengecualian §1.4). Tiap ubin adalah pintasan navigasi ke rute yang **sudah ada** di v1 (link "Lihat Semua" per status/kategori), bukan halaman baru.

```tsx
// components/dashboard/QuickAccessTilesPanel.tsx
import Link from "next/link";
import { type LucideIcon } from "lucide-react";
import { cn } from "@/lib/utils";

interface Tile {
  label: string;
  sublabel: string;
  href: string;
  icon: LucideIcon;
  tone: "primary" | "secondary" | "success" | "warning" | "danger" | "info";
}

const TILE_BG: Record<Tile["tone"], string> = {
  primary: "bg-[--color-primary]",
  secondary: "bg-[--color-secondary]",
  success: "bg-[--color-success]",
  warning: "bg-[--color-warning]",
  danger: "bg-[--color-danger]",
  info: "bg-[--color-info]",
};

export function QuickAccessTilesPanel({ tiles }: { tiles: Tile[] }) {
  return (
    <div className="grid grid-cols-2 gap-4 sm:grid-cols-4">
      {tiles.map((tile) => {
        const Icon = tile.icon;
        return (
          <Link key={tile.href} href={tile.href} className="flex flex-col items-start gap-2">
            <div className={cn("flex h-20 w-20 items-center justify-center rounded-2xl", TILE_BG[tile.tone])}>
              <Icon className="h-7 w-7 text-white" strokeWidth={1.75} />
            </div>
            <div className="flex flex-col">
              <span className="text-sm font-semibold text-[--color-text-primary]">{tile.label}</span>
              <span className="text-xs text-[--color-text-secondary]">{tile.sublabel}</span>
            </div>
          </Link>
        );
      })}
    </div>
  );
}
```

#### 3.5.4 `<ActivityFeedPanel>` — feed aktivitas (dipakai Zona D di Bab 4)

Padanan visual dari "Message Team" pada referensi. **Catatan ketat:** Architax tidak memiliki fitur chat/messaging di PRD manapun — komponen ini **bukan** chat. Ia adalah tampilan *read-only* dari entri Audit Log yang sudah wajib dicatat sistem untuk setiap transisi status (Tech Spec Bab 7, dirujuk berulang di Bab 6 dokumen ini — mis. Modal Batal Selesai §6.6). Tidak ada input field, tidak ada tombol kirim, tidak ada aksi baru — murni menyurfacekan data yang sudah ada.

```tsx
// components/dashboard/ActivityFeedPanel.tsx
interface ActivityEntry {
  id: string;
  actorName: string;
  description: string; // contoh: "memindahkan Permohonan #1234 ke status REVISION"
  timeAgo: string; // contoh: "5 menit lalu"
}

export function ActivityFeedPanel({ entries }: { entries: ActivityEntry[] }) {
  return (
    <div className="flex flex-col gap-4">
      {entries.map((e) => (
        <div key={e.id} className="flex flex-col gap-0.5">
          <span className="text-xs text-[--color-text-secondary]">{e.timeAgo}</span>
          <p className="text-sm text-[--color-text-primary]">
            <span className="font-semibold">{e.actorName}</span> {e.description}
          </p>
        </div>
      ))}
      {entries.length === 0 && (
        <p className="text-sm text-[--color-text-secondary]">Belum ada aktivitas.</p>
      )}
    </div>
  );
}
```

---

## 4. Spesifikasi Halaman Home per Role — 4 Zona Dashboard (v2)

**Perubahan dari v1 (lihat Catatan Revisi v2 di awal dokumen):** pola "4 kartu angka besar seragam" digantikan 4 zona dengan jenis komponen berbeda, disusun via `<DashboardGrid>` (Bab 3.4) dan dibangun dari 4 komponen `<DashboardPanel>` turunan di Bab 3.5. Posisi & jenis komponen zona **sama persis di seluruh role** (memenuhi instruksi brief poin 5: "semua role memiliki layout yang sama persis"); yang berbeda per role hanyalah data, status, dan rute yang ditampilkan — seluruhnya **identik dengan yang sudah didefinisikan di v1**, tidak ada metrik atau rute baru.

- **Zona A (top-left):** `<ListRowsPanel>` — daftar item paling aktif/mendesak untuk role tersebut, dengan header `<DashboardPanel>` berisi count badge + link "Lihat Semua" (padanan visual "Tasks Due" pada referensi).
- **Zona B (top-right):** `<QuickAccessTilesPanel>` — ubin warna solid untuk metrik/kategori sekunder yang di v1 berupa kartu angka saja, kini jadi pintasan navigasi (padanan visual "Favorites").
- **Zona C (bottom-left):** `<ListRowsPanel>` kedua — daftar item terkait lainnya (padanan visual "Recent Projects").
- **Zona D (bottom-right):** `<ActivityFeedPanel>` — feed Audit Log *read-only* terkait domain role tersebut (padanan visual "Message Team", **bukan** fitur chat — lihat catatan ketat di §3.5.4).

### 4.1 Role: PENGINPUT — Halaman Home

**Zona A — "Permohonan Revisi" (`<ListRowsPanel>`)**
- Header: count badge `tone="warning"` = jumlah Permohonan berstatus `REVISION` milik Penginput; link "Lihat Semua" → `/permohonan?status=REVISION` (sama dengan v1 Grid 1).
- Baris: hingga 5 Permohonan `REVISION` terbaru — nama pemohon (title), "Jenis · Tanggal Submit" (meta), `<PermohonanStatusBadge status="REVISION">`.

**Zona B — "Ringkasan Layanan & Status" (`<QuickAccessTilesPanel>`)**
- Memuat ubin yang di v1 adalah Grid 2 ("Jenis Layanan Terinput") + Grid 4 ("Permohonan Rejected"), kini sebagai pintasan navigasi:

| Ubin | Sublabel | Ikon | Tone | Href |
|---|---|---|---|---|
| Mutasi Sebagian | `{count} permohonan` | `ArrowLeftRight` | primary | `/permohonan?jenis=MUTASI_SEBAGIAN` |
| Mutasi Habis Update | `{count} permohonan` | `RefreshCcw` | info | `/permohonan?jenis=MUTASI_HABIS_UPDATE` |
| Mutasi Habis Reguler | `{count} permohonan` | `Repeat` | secondary | `/permohonan?jenis=MUTASI_HABIS_REGULER` |
| Objek Pajak Baru | `{count} permohonan` | `FilePlus` | success | `/permohonan?jenis=OBJEK_PAJAK_BARU` |
| Pembetulan | `{count} permohonan` | `Edit3` | warning | `/permohonan?jenis=PEMBETULAN` |
| Pengaktifan | `{count} permohonan` | `Power` | info | `/permohonan?jenis=PENGAKTIFAN` |
| Permohonan Rejected | `{count} kedaluwarsa` | `AlertTriangle` | danger | `/permohonan?status=REJECTED` |

(Tone 6 ubin pertama murni variasi visual non-semantik — `JenisPermohonan` tidak punya kategori semantik di §1.4. Tone ubin "Permohonan Rejected" mengikuti token resmi status `REJECTED` = danger, sesuai §1.4.)

**Zona C — "Permohonan Terkini" (`<ListRowsPanel>`)**
- Sama persis dengan v1 Grid 3: 5 Permohonan terakhir yang diinput Penginput — nama pemohon (title), "Jenis · Tanggal Submit" (meta), `<PermohonanStatusBadge>`. Link "Lihat Semua" → `/permohonan`.

**Zona D — "Aktivitas Terkini" (`<ActivityFeedPanel>`)**
- Entri Audit Log yang menyangkut Permohonan milik Penginput tersebut (mis. transisi ke `REVISION` oleh Peneliti, atau ke `REJECTED` oleh Cron Job) — read-only, tanpa aksi.

```tsx
// app/(dashboard)/page.tsx — varian PENGINPUT, Zona A (v2)
<DashboardPanel
  title="Permohonan Revisi"
  countBadge={{ value: revisionCount, tone: "warning" }}
  links={[{ label: "Lihat Semua", href: "/permohonan?status=REVISION" }]}
>
  <ListRowsPanel
    rows={revisionItems.map((p) => ({
      id: p.id,
      title: p.namaPemohon,
      meta: `${p.jenisPermohonan} · ${formatDate(p.tanggalSubmit)}`,
      badgeLabel: "Revisi",
      badgeClassName: PERMOHONAN_STATUS_STYLE.REVISION,
    }))}
  />
</DashboardPanel>
```

### 4.2 Role: PENELITI — Halaman Home

**Zona A — "Permohonan Menunggu Verifikasi" (`<ListRowsPanel>`)**
- Header: count badge `tone="info"` = jumlah Permohonan `SUBMITTED` tanpa `bundleId` (sama dengan v1 Grid 2); link "Lihat Semua" → `/permohonan?status=SUBMITTED`.
- Baris: hingga 5 Permohonan `SUBMITTED` terbaru — nama pemohon, "Jenis · Tanggal Submit", `<PermohonanStatusBadge status="SUBMITTED">`.

**Zona B — "Ringkasan Bundle" (`<QuickAccessTilesPanel>`)**
- Ubin dari v1 Grid 3 ("Bundle Terkunci Bulan Ini", tone primary, ikon `Lock`, sublabel `{count} bulan ini · {trend}`, href `/bundle?status=LOCKED`) dan v1 Grid 4 ("Permohonan Perlu Kertas Kerja", tone warning, ikon `FileText`, sublabel `{count} Mutasi Sebagian`, href `/permohonan?jenis=MUTASI_SEBAGIAN`).

**Zona C — "Bundle DRAFT" (`<ListRowsPanel>`)**
- Header: count badge `tone="warning"` = jumlah Bundle `DRAFT` (sama dengan v1 Grid 1); link "Lihat Semua" → `/bundle?status=DRAFT`.
- Baris: hingga 5 Bundle `DRAFT` — ID Bundle (title), "Jumlah Permohonan · Tanggal Dibuat" (meta), `<BundleStatusBadge status="DRAFT">`.

**Zona D — "Aktivitas Terkini" (`<ActivityFeedPanel>`)**
- Entri Audit Log terkait Bundle/Permohonan yang ditangani Peneliti (mis. "Minta Revisi" dikirim, Permohonan dimasukkan ke Bundle).

### 4.3 Role: PENGARSIP — Halaman Home

**Zona A — "Bundle Siap Pindai" (`<ListRowsPanel>`)**
- Header: count badge `tone="info"` = jumlah Bundle `LOCKED` yang Permohonan-nya belum `ARCHIVED` (sama dengan v1 Grid 1); link "Lihat Semua" → `/bundle?status=LOCKED`.
- Baris: hingga 5 Bundle — ID Bundle, "Jumlah Permohonan · Tanggal Dikunci", `<BundleStatusBadge status="LOCKED">`.

**Zona B — "Ringkasan Arsip" (`<QuickAccessTilesPanel>`)**
- Ubin dari v1 Grid 2 ("Arsip Digital Selesai", tone success, ikon `CheckCircle2`, sublabel `{count} bulan ini · {trend}`) dan v1 Grid 4 ("Total Arsip Digital", tone secondary, ikon `Archive`, sublabel `{count} total sistem`), keduanya tanpa href aksi (murni indikator, sama seperti v1 yang tidak mendefinisikan link untuk kedua grid ini).

**Zona C — "Dokumen Cacat (Unbundle)" (`<ListRowsPanel>`)**
- Header: count badge `tone="danger"` = jumlah Permohonan hasil *Unbundle* (sama dengan v1 Grid 3); link "Lihat Semua" → `/permohonan?status=SUBMITTED`.
- Baris: hingga 5 Permohonan hasil unbundle — nama pemohon, "NOP · Tanggal Unbundle", `<PermohonanStatusBadge status="SUBMITTED">`.

**Zona D — "Aktivitas Terkini" (`<ActivityFeedPanel>`)**
- Entri Audit Log terkait Bundle/Arsip (mis. "Selesai Pindai" dipicu, "Unbundle" dipicu).

### 4.4 Role: PENGIRIM — Halaman Home

**Zona A — "Bundle Siap Kirim" (`<ListRowsPanel>`)**
- Header: count badge `tone="info"` = jumlah Bundle `LOCKED` tanpa `manifestId` (sama dengan v1 Grid 2); link "Lihat Semua" → `/bundle?status=LOCKED`.
- Baris: hingga 5 Bundle — ID Bundle, "Jumlah Permohonan · Tanggal Dikunci", `<BundleStatusBadge status="LOCKED">`.

**Zona B — "Ringkasan Manifest" (`<QuickAccessTilesPanel>`)**
- Ubin dari v1 Grid 3 ("Manifest Terkirim", tone success, ikon `Truck`, sublabel `{count} bulan ini · {trend}`) dan v1 Grid 4 ("Bundle Hilang", tone danger, ikon `AlertTriangle`, sublabel `{count} perlu dilacak`, href `/bundle?status=LOCKED`).

**Zona C — "Manifest DRAFT" (`<ListRowsPanel>`)**
- Header: count badge `tone="warning"` = jumlah Manifest `DRAFT` (sama dengan v1 Grid 1); link "Lihat Semua" → `/manifest?status=DRAFT`.
- Baris: hingga 5 Manifest `DRAFT` — ID Manifest, "Jumlah Bundle · Tanggal Dibuat", `<ManifestStatusBadge status="DRAFT">`.

**Zona D — "Aktivitas Terkini" (`<ActivityFeedPanel>`)**
- Entri Audit Log terkait Manifest/Bundle (mis. "Kunci & Cetak" Manifest, "Laporkan Hilang" dipicu).

### 4.5 Role: PEMANTAU — Halaman Home

**Zona A — "Permohonan ARCHIVED" (`<ListRowsPanel>`)**
- Header: count badge `tone="info"` = jumlah Permohonan `ARCHIVED` (sama dengan v1 Grid 1); link "Lihat Semua" → `/pemantauan`.
- Baris: hingga 5 Permohonan `ARCHIVED` — nama pemohon, "Jenis · {jumlah hari menunggu} hari" (meta), `<PermohonanStatusBadge status="ARCHIVED">`.

**Zona B — "Ringkasan Penyelesaian" (`<QuickAccessTilesPanel>`)**
- Ubin dari v1 Grid 2 ("Permohonan Selesai", tone success, ikon `CheckCircle2`, sublabel `{count} bulan ini · {trend}`) dan v1 Grid 4 ("Total Manifest SENT", tone primary, ikon `PackageCheck`, sublabel `{count} total sistem`).

**Zona C — "Bottleneck (>30 hari)" (`<ListRowsPanel>`)**
- Header: count badge `tone="warning"` = jumlah Permohonan `ARCHIVED` >30 hari belum `COMPLETED` (sama dengan v1 Grid 3); link "Lihat Semua" → `/pemantauan?filter=bottleneck`.
- Baris: hingga 5 Permohonan bottleneck — nama pemohon, "{jumlah hari menunggu} hari menunggu" (meta, ditonjolkan `color-warning` jika >30 hari), `<PermohonanStatusBadge status="ARCHIVED">`.

**Zona D — "Aktivitas Terkini" (`<ActivityFeedPanel>`)**
- Entri Audit Log terkait transisi `COMPLETED`/rollback (mis. "Tandai Selesai" dan "Batal Selesai" — selaras dengan kewajiban pencatatan Audit Log di Modal §6.6).

---


## 5. Spesifikasi Halaman per Role (Page-by-Page Breakdown)

Pemetaan halaman mengikuti struktur folder Tech Spec Bab 1.3, dibatasi ketat pada halaman yang berwenang diakses tiap role (Tech Spec Bab 10.3 `ROUTE_ROLE_MAP`).

### 5.1 Role: PENGINPUT

#### Halaman List Permohonan (`/permohonan`)
- **Tujuan Halaman:** Menampilkan seluruh Permohonan milik Penginput yang bersangkutan, dengan fokus pada status `SUBMITTED` dan `REVISION` yang masih dapat ditindaklanjuti.
- **Komponen Utama:** `<DataTable>` (Header Row + Body Row, density Comfortable default), `<Tabs>` underline-style untuk filter cepat (Semua / Submitted / Revision), `<Button>` Primary "Permohonan Baru" di kanan-atas page header.
- **Kolom Data:** NOP, Nama Pemohon, Jenis Permohonan, Tanggal Submit, Status (`<PermohonanStatusBadge>`), kolom aksi (ikon overflow `⋯`, ghost, muncul saat row hover).
- **Aksi / Tombol yang Muncul:**
  - Status `SUBMITTED` & belum punya `bundleId` → tombol **"Update"** muncul di kolom aksi (PRD 6.1: Penginput dapat update selama `SUBMITTED` dan belum terikat `bundleId`).
  - Status `REVISION` → tombol **"Update"** muncul (PRD 6.2: berkas dikembalikan ke Penginput untuk diperbaiki).
  - Status `BUNDLED`, `ARCHIVED`, `COMPLETED`, `REJECTED` → **tidak ada tombol aksi** (di luar kewenangan Penginput pada status tersebut).

#### Halaman Form Permohonan Baru (`/permohonan/permohonan-baru`)
- **Tujuan Halaman:** Menginput data awal Permohonan (Fase 1 PRD).
- **Komponen Utama:** `<Form>` (React Hook Form + Zod, sesuai Tech Spec Bab 11.3), `<Card>` pembungkus form, `<Select>` untuk Jenis Permohonan (6 opsi enum), `<Input>` untuk field lain.
- **Kolom Data / Field:** NOP (`<Input>`, validasi regex 18 digit), Nama Pemohon (`<Input>`), Jenis Permohonan (`<Select>` — Mutasi Sebagian, Mutasi Habis Update, Mutasi Habis Reguler, Objek Pajak Baru, Pembetulan, Pengaktifan), Luas Bumi (`<Input type="number">`), Luas Bangunan (`<Input type="number">`). Field `tanggalSubmit` **tidak ditampilkan sebagai input** — auto-generated di server (Tech Spec Bab 11.3).
- **Aksi / Tombol yang Muncul:** `<Button>` Primary "Simpan Permohonan" (selalu muncul, men-submit form via Server Action `createPermohonanAction`); `<Button>` Secondary "Batal" di sebelah kiri tombol primary.

#### Halaman Detail/Update Permohonan (`/permohonan/[id]`)
- **Tujuan Halaman:** Melihat detail dan memperbarui data Permohonan yang dikembalikan untuk revisi atau yang masih `SUBMITTED` dan belum terikat Bundle.
- **Komponen Utama:** `<Card>` detail (read-only fields jika status tidak mengizinkan edit), `<Form>` editable jika status `SUBMITTED` (tanpa `bundleId`) atau `REVISION`.
- **Kolom Data / Field:** Sama seperti form Permohonan Baru, plus `<PermohonanStatusBadge>` di header card.
- **Aksi / Tombol yang Muncul:**
  - Status `REVISION` atau (`SUBMITTED` AND `bundleId == null`) → tombol **"Simpan Perubahan"** (Primary) aktif.
  - Status lain → seluruh field render read-only, **tidak ada tombol Simpan**.

### 5.2 Role: PENELITI

#### Halaman List Permohonan (`/permohonan`)
- **Tujuan Halaman:** Menyeleksi Permohonan berstatus `SUBMITTED` untuk dimasukkan ke Bundle.
- **Komponen Utama:** `<DataTable>` dengan kolom checkbox di paling kiri (untuk seleksi multi-row), filter default `status=SUBMITTED`.
- **Kolom Data:** Checkbox, NOP, Nama Pemohon, Jenis Permohonan, Tanggal Submit, Status.
- **Aksi / Tombol yang Muncul:**
  - Status `SUBMITTED` → tombol baris **"Minta Revisi"** (Secondary/Ghost, ikon `XCircle`) dan toolbar bulk-action **"Masukkan ke Bundle"** (Primary, aktif setelah ≥1 row dicentang dan Bundle tujuan berstatus `DRAFT`).

#### Halaman List Bundle (`/bundle`)
- **Tujuan Halaman:** Memantau seluruh Bundle yang sedang dikerjakan/dikunci.
- **Komponen Utama:** `<DataTable>`, `<Tabs>` filter (Draft / Locked / In Manifest), `<Button>` Primary "Bundle Baru" di page header.
- **Kolom Data:** ID Bundle, Jumlah Permohonan di dalamnya, Status (`<BundleStatusBadge>`), Tanggal Dibuat, kolom aksi.
- **Aksi / Tombol yang Muncul:** Klik baris → navigasi ke Halaman Detail Bundle. Tidak ada aksi inline di tabel ini selain navigasi (aksi state-transition dilakukan di halaman detail, sesuai Bab 7).

#### Halaman Detail Bundle & Seleksi (`/bundle/[id]`)
- **Tujuan Halaman:** Mengelola anggota Bundle (Fase 2 PRD): mencentang Permohonan masuk, mengeluarkan, dan mengunci Bundle.
- **Komponen Utama:** `<Card>` info Bundle (status, jumlah anggota), `<DataTable>` daftar Permohonan anggota dengan checkbox, panel "Tambah Permohonan" (`<DataTable>` terpisah berisi Permohonan `SUBMITTED` yang belum terikat Bundle manapun).
- **Kolom Data:** Sama seperti tabel Permohonan, plus indikator "Mutasi Sebagian" pada baris yang memerlukan Kertas Kerja (Tech Spec Bab 6.3).
- **Aksi / Tombol yang Muncul (state-driven, lihat Bab 7.2):**
  - Bundle `DRAFT` → tombol **"Kunci & Cetak"** (Primary, men-trigger cetak Surat Pengantar Bundle + Kertas Kerja jika ada Permohonan Mutasi Sebagian, lalu transisi Bundle ke `LOCKED`), **"Tambah Permohonan"**, dan per-baris **"Keluarkan dari Bundle"** (Ghost, ikon `LogOut`).
  - Bundle `LOCKED`/`IN_MANIFEST` → tabel anggota menjadi read-only; tidak ada tombol modifikasi keanggotaan dari halaman ini (perubahan keanggotaan pada status ini hanya melalui fitur "Unbundle" oleh PENGARSIP, di luar kewenangan PENELITI).
  - Tombol **"Minta Revisi"** muncul per-baris Permohonan anggota selama Bundle `DRAFT`.

### 5.3 Role: PENGARSIP

#### Halaman List Bundle (`/bundle`)
- **Tujuan Halaman:** Melihat Bundle yang siap dipindai.
- **Komponen Utama:** `<DataTable>`, filter default `status=LOCKED`.
- **Kolom Data:** ID Bundle, Jumlah Permohonan, Status, Tanggal Dikunci, kolom aksi.
- **Aksi / Tombol yang Muncul:** Klik baris (status `LOCKED`) → navigasi ke Halaman Detail Arsip.

#### Halaman Detail Arsip (`/arsip/[id]`)
- **Tujuan Halaman:** Memindai dan mengunggah dokumen fisik menjadi Arsip Digital (Fase 3 PRD).
- **Komponen Utama:** `<Card>` info Bundle, area upload PDF (drag-and-drop, memanggil Route Handler `app/api/upload/route.ts` — Tech Spec Bab 6.5), `<DataTable>` daftar Permohonan dalam Bundle dengan status upload per-item.
- **Kolom Data:** NOP, Nama Pemohon, Jenis Permohonan, status upload Arsip Digital (sudah/belum), kolom aksi per baris.
- **Aksi / Tombol yang Muncul:**
  - Bundle `LOCKED` → tombol **"Unbundle"** (Danger, per-baris Permohonan, jika dokumen fisik cacat saat pemindaian — PRD 6.3) dan tombol **"Selesai Pindai"** (Primary, aktif setelah seluruh Permohonan dalam Bundle memiliki `arsipDigitalUrl`, men-transisi seluruh Permohonan anggota ke `ARCHIVED`).

### 5.4 Role: PENGIRIM

#### Halaman List Bundle (`/bundle`)
- **Tujuan Halaman:** Melihat Bundle berstatus `LOCKED` yang siap dimasukkan ke Manifest.
- **Komponen Utama:** `<DataTable>`, filter default `status=LOCKED`, kolom checkbox untuk seleksi multi-Bundle ke Manifest.
- **Kolom Data:** ID Bundle, Jumlah Permohonan, Status, Tanggal Dikunci.
- **Aksi / Tombol yang Muncul:** Toolbar bulk-action **"Masukkan ke Manifest"** (Primary, aktif setelah ≥1 Bundle dicentang).

#### Halaman List Manifest (`/manifest`)
- **Tujuan Halaman:** Memantau seluruh Manifest.
- **Komponen Utama:** `<DataTable>`, `<Tabs>` filter (Draft / Locked / Sent), `<Button>` Primary "Manifest Baru".
- **Kolom Data:** ID Manifest, Jumlah Bundle, Status (`<ManifestStatusBadge>`), Tanggal Dibuat, kolom aksi.
- **Aksi / Tombol yang Muncul:** Klik baris → navigasi ke Halaman Detail Manifest.

#### Halaman Detail Manifest (`/manifest/[id]`)
- **Tujuan Halaman:** Mengelola anggota Manifest dan logistik pengiriman (Fase 4 PRD).
- **Komponen Utama:** `<Card>` info Manifest, `<DataTable>` daftar Bundle anggota dengan checkbox, area upload Bukti Tanda Terima (PDF).
- **Kolom Data:** ID Bundle, Jumlah Permohonan di dalamnya, Status Bundle.
- **Aksi / Tombol yang Muncul (state-driven, lihat Bab 7.3):**
  - Manifest `DRAFT` → tombol **"Kunci & Cetak"** (Primary, mencetak Surat Pengantar Manifest, transisi ke `LOCKED`), **"Tambah Bundle"**.
  - Manifest `LOCKED` → tombol **"Revisi Manifest"** (Secondary, membuka kembali ke `DRAFT` — PRD 6.4) dan area **"Upload Bukti Terima"** (mengunggah Bukti Tanda Terima, transisi ke `SENT`).
  - Per-baris Bundle anggota saat Manifest `LOCKED` (atau setelah `SENT`, karena PRD 6.4 menyatakan pelaporan hilang tetap terjadi meski Manifest sudah `SENT`) → tombol **"Laporkan Hilang"** (Danger, memutus `manifestId` Bundle tersebut, mengembalikan Bundle ke status `LOCKED`).

### 5.5 Role: PEMANTAU

#### Halaman Pemantauan (`/pemantauan`)
- **Tujuan Halaman:** Memantau dashboard Permohonan yang fisiknya telah tiba di pusat, dan menandai selesai saat produk layanan terbit (Fase 5 PRD).
- **Komponen Utama:** `<Card>` Stats untuk ringkasan bottleneck (>30 hari `ARCHIVED` belum `COMPLETED`), `<DataTable>` utama daftar Permohonan berstatus `ARCHIVED` (dan opsional toggle untuk melihat `COMPLETED`).
- **Kolom Data:** NOP, Nama Pemohon, Jenis Permohonan, Tanggal `ARCHIVED`, Jumlah Hari Menunggu, Status, kolom aksi.
- **Aksi / Tombol yang Muncul:**
  - Status `ARCHIVED` → tombol **"Tandai Selesai"** (Primary, per-baris, transisi ke `COMPLETED`).
  - Status `COMPLETED` → tombol **"Batal Selesai"** (Danger/Secondary, per-baris, memicu Modal konfirmasi Rollback — Bab 6, transisi kembali ke `ARCHIVED` dengan pencatatan Audit Log wajib).

---

## 6. Komponen Spesial & Modal (Dialog)

Seluruh modal memakai komponen `<Dialog>` shadcn/ui di atas anatomi `UI_DESIGN_SYSTEM.md` §15/§19: radius 16px, shadow level "Modal" (`0 20px 48px rgba(33,30,51,0.18)`), animasi masuk 200ms ease-out (backdrop fade + panel scale 95%→100%), animasi keluar 150ms ease-in.

### 6.1 Modal "Minta Revisi"
- **Trigger:** Tombol "Minta Revisi" pada Permohonan `SUBMITTED` (PENELITI).
- **Isi:** `<Textarea>` **wajib diisi** (alasan revisi), label "Alasan Revisi", validasi Zod `min(1)` pada submit.
- **Tombol:** Primary "Kirim Permintaan Revisi" (disabled hingga textarea terisi), Secondary "Batal".

### 6.2 Modal "Keluarkan dari Bundle"
- **Trigger:** Tombol "Keluarkan dari Bundle" pada baris Permohonan di Halaman Detail Bundle (Bundle `DRAFT`).
- **Isi:** Konfirmasi teks singkat + daftar (Simplified List Row) Permohonan yang akan dikeluarkan dari Bundle.
- **Tombol:** Primary "Konfirmasi Keluarkan", Secondary "Batal".

### 6.3 Modal "Unbundle"
- **Trigger:** Tombol "Unbundle" pada Halaman Detail Arsip (PENGARSIP) saat dokumen fisik cacat.
- **Isi:** Konfirmasi keras — Alert inline (`color-danger` left-border 4px, `-tint` background) berisi peringatan bahwa data akan kembali ke status `SUBMITTED` dan fisik dikembalikan ke PENELITI untuk dikaji ulang (PRD 6.3).
- **Tombol:** Danger "Ya, Unbundle Dokumen Ini", Secondary "Batal".

### 6.4 Modal "Revisi Manifest"
- **Trigger:** Tombol "Revisi Manifest" pada Halaman Detail Manifest (Manifest `LOCKED`).
- **Isi:** Konfirmasi + daftar (Simplified List Row) Bundle yang akan dilepas dari Manifest setelah Manifest dibuka kembali ke `DRAFT`.
- **Tombol:** Primary "Konfirmasi Buka Kunci", Secondary "Batal".

### 6.5 Modal "Laporkan Bundle Hilang"
- **Trigger:** Tombol "Laporkan Hilang" pada baris Bundle di Halaman Detail Manifest.
- **Isi:** Konfirmasi **sangat keras** — Alert inline `color-danger` dengan peringatan eksplisit bahwa tindakan ini akan tercatat sebagai insiden dan dapat ditelusuri melalui audit. Menampilkan ID Bundle yang akan dilaporkan hilang.
- **Tombol:** Danger "Ya, Laporkan Hilang", Secondary "Batal".

### 6.6 Modal "Batal Selesai (Rollback)"
- **Trigger:** Tombol "Batal Selesai" pada Permohonan `COMPLETED` (PEMANTAU).
- **Isi:** Konfirmasi dengan peringatan eksplisit: *"Tindakan ini akan dicatat ke Audit Log — nilai sebelum dan sesudah perubahan akan terekam permanen"* (PRD 6.5 / Tech Spec Bab 7).
- **Tombol:** Danger "Ya, Batalkan Status Selesai", Secondary "Batal".

```tsx
// components/modals/BatalSelesaiModal.tsx
"use client";

import {
  Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter,
} from "@/components/ui/dialog";
import { Button } from "@/components/ui/button";
import { AlertTriangle } from "lucide-react";
import { batalSelesaiAction } from "@/lib/actions/permohonan.actions";

interface Props {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  permohonanId: string;
  actorId: string;
}

export function BatalSelesaiModal({ open, onOpenChange, permohonanId, actorId }: Props) {
  const handleConfirm = async () => {
    await batalSelesaiAction(permohonanId, actorId, "PEMANTAU");
    onOpenChange(false);
  };

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className="rounded-2xl">
        <DialogHeader>
          <DialogTitle>Batalkan Status Selesai?</DialogTitle>
        </DialogHeader>

        <div className="flex gap-3 rounded-lg border-l-4 border-[--color-danger] bg-[--color-danger-tint] p-4">
          <AlertTriangle className="h-5 w-5 shrink-0 text-[--color-danger]" strokeWidth={1.75} />
          <p className="text-sm text-[--color-text-primary]">
            Tindakan ini akan dicatat ke Audit Log — nilai sebelum dan sesudah perubahan akan terekam permanen.
          </p>
        </div>

        <DialogFooter className="mt-4 gap-2">
          <Button variant="outline" onClick={() => onOpenChange(false)} className="rounded-lg">
            Batal
          </Button>
          <Button
            onClick={handleConfirm}
            className="rounded-lg bg-[--color-danger] text-white hover:opacity-90"
          >
            Ya, Batalkan Status Selesai
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}
```

---

## 7. State-Driven UI Rules (Tombol & Aksi Berdasarkan Status)

Tombol dan aksi **hanya muncul jika status entitas mengizinkan**, mengikuti `validateTransition()` pada Tech Spec Bab 12. Tabel berikut adalah representasi UI dari `TRANSITION_RULES` tersebut.

### 7.1 Permohonan

| Status | Tombol yang Muncul | Role |
|---|---|---|
| `SUBMITTED` (belum punya `bundleId`) | "Update" | PENGINPUT |
| `SUBMITTED` | "Minta Revisi", "Masukkan ke Bundle" | PENELITI |
| `REVISION` | "Update" | PENGINPUT |
| `BUNDLED` (Bundle masih `DRAFT`) | "Keluarkan dari Bundle" | PENELITI |
| `BUNDLED` (Bundle `LOCKED`) | "Unbundle" | PENGARSIP |
| `ARCHIVED` | "Tandai Selesai" | PEMANTAU |
| `COMPLETED` | "Batal Selesai" | PEMANTAU |
| `REJECTED` | *(tidak ada — status terminal)* | — |

### 7.2 Bundle

| Status | Tombol yang Muncul | Role |
|---|---|---|
| `DRAFT` | "Kunci & Cetak", "Tambah Permohonan", "Keluarkan dari Bundle" (per-baris anggota) | PENELITI |
| `LOCKED` | "Masukkan ke Manifest" | PENGIRIM |
| `LOCKED` | "Unbundle" (per-baris anggota Permohonan) | PENGARSIP |
| `IN_MANIFEST` | "Laporkan Hilang" | PENGIRIM |

**Catatan khusus "Laporkan Hilang" (PRD 6.4 — Map Hilang Saat Transit):** tombol ini tetap muncul pada baris Bundle berstatus `IN_MANIFEST` **terlepas dari status Manifest induknya** — termasuk ketika Manifest induk sudah bertransisi ke `SENT`. PRD secara eksplisit menyatakan Pengirim *"tetap menyelesaikan Manifest ke SENT... lalu menggunakan fitur Laporkan Bundle Hilang"*, sehingga Halaman Detail Manifest (Bab 5.4) tidak mengunci tombol ini hanya karena Manifest sudah berstatus terminal `SENT` — yang berstatus terminal pada skenario ini adalah alur normal logistik, bukan kemungkinan pelaporan kehilangan itu sendiri.

### 7.3 Manifest

| Status | Tombol yang Muncul | Role |
|---|---|---|
| `DRAFT` | "Kunci & Cetak", "Tambah Bundle" | PENGIRIM |
| `LOCKED` | "Revisi Manifest", "Upload Bukti Terima" | PENGIRIM |
| `SENT` | *(tidak ada — status terminal logistik)* | — |

### 7.4 Implementasi: Helper Komponen `StateDrivenActions`

Komponen ini membaca status entitas dan role user aktif (dari `useSession()` NextAuth) untuk merender tombol secara kondisional, sehingga satu sumber kebenaran (tabel di atas) konsisten di seluruh halaman.

```tsx
// components/state-driven/PermohonanActions.tsx
"use client";

import { useSession } from "next-auth/react";
import { Button } from "@/components/ui/button";

interface Props {
  status: "SUBMITTED" | "REVISION" | "BUNDLED" | "ARCHIVED" | "COMPLETED" | "REJECTED";
  hasBundleId: boolean;
  bundleStatus?: "DRAFT" | "LOCKED" | "IN_MANIFEST";
  onUpdate?: () => void;
  onMintaRevisi?: () => void;
  onMasukkanKeBundle?: () => void;
  onKeluarkanDariBundle?: () => void;
  onUnbundle?: () => void;
  onTandaiSelesai?: () => void;
  onBatalSelesai?: () => void;
}

export function PermohonanActions(props: Props) {
  const { data: session } = useSession();
  const role = session?.user.role;
  const { status, hasBundleId, bundleStatus } = props;

  if (role === "PENGINPUT" && (status === "REVISION" || (status === "SUBMITTED" && !hasBundleId))) {
    return <Button onClick={props.onUpdate}>Update</Button>;
  }

  if (role === "PENELITI" && status === "SUBMITTED") {
    return (
      <div className="flex gap-2">
        <Button variant="outline" onClick={props.onMintaRevisi}>Minta Revisi</Button>
        <Button onClick={props.onMasukkanKeBundle}>Masukkan ke Bundle</Button>
      </div>
    );
  }

  if (role === "PENELITI" && status === "BUNDLED" && bundleStatus === "DRAFT") {
    return <Button variant="outline" onClick={props.onKeluarkanDariBundle}>Keluarkan dari Bundle</Button>;
  }

  if (role === "PENGARSIP" && status === "BUNDLED" && bundleStatus === "LOCKED") {
    return <Button variant="destructive" onClick={props.onUnbundle}>Unbundle</Button>;
  }

  if (role === "PEMANTAU" && status === "ARCHIVED") {
    return <Button onClick={props.onTandaiSelesai}>Tandai Selesai</Button>;
  }

  if (role === "PEMANTAU" && status === "COMPLETED") {
    return <Button variant="destructive" onClick={props.onBatalSelesai}>Batal Selesai</Button>;
  }

  return null; // REJECTED atau kombinasi role/status yang tidak diizinkan: tidak ada tombol
}
```

**Alasan pola ini:** Memusatkan logika "tombol mana yang boleh tampil" pada satu komponen per entitas mencegah duplikasi aturan di banyak halaman, dan memastikan UI selalu konsisten dengan `validateTransition()` di backend (Tech Spec Bab 12) — jika backend menolak suatu transisi, UI yang benar seharusnya memang tidak pernah menampilkan tombol untuk transisi tersebut sejak awal.
