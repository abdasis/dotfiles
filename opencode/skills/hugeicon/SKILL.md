---
name: hugeicon
description: Use when adding or replacing icons using HugeIcons in React or React Native projects. Handles installation, imports, component usage, icon switching with showAlt/altIcon, sizing, color, and strokeWidth. Triggers on tasks involving HugeIcons, @hugeicons/react, @hugeicons/core-free-icons, or icon swapping patterns.
metadata:
  author: local
  version: "1.1.0"
  domain: frontend
  triggers: hugeicon, HugeiconsIcon, @hugeicons/react, icon, icons, icon switching, icon toggle
  role: specialist
  scope: implementation
  output-format: code
  related-skills: react-expert, typescript-pro, laravel-inertia-react
---

# HugeIcon Skill

Spesialis integrasi HugeIcons ke dalam project React dan React Native. Panduan ini mencakup instalasi, import, penggunaan komponen `HugeiconsIcon`, dan pola-pola umum.

## Instalasi

```bash
npm install @hugeicons/react @hugeicons/core-free-icons
# atau
bun add @hugeicons/react @hugeicons/core-free-icons
```

- `@hugeicons/react` — wrapper komponen React untuk rendering icon
- `@hugeicons/core-free-icons` — 5.100+ icon gratis (Stroke Rounded style saja)

Untuk style tambahan (Solid, Duotone, Twotone, dll), gunakan paket pro:
```bash
npm install @hugeicons-pro/core-solid-rounded
```

## Penggunaan Dasar

```tsx
import { HugeiconsIcon } from '@hugeicons/react';
import { Notification03Icon } from '@hugeicons/core-free-icons';

export function NotificationBell() {
  return (
    <HugeiconsIcon
      icon={Notification03Icon}
      size={24}
      color="currentColor"
      strokeWidth={1.5}
    />
  );
}
```

## Props Lengkap `HugeiconsIcon`

| Prop          | Tipe             | Default         | Keterangan                                      |
|---------------|------------------|-----------------|-------------------------------------------------|
| `icon`        | `IconSvgObject`  | **required**    | Icon utama yang ditampilkan                     |
| `altIcon`     | `IconSvgObject`  | —               | Icon alternatif untuk state switching           |
| `showAlt`     | `boolean`        | `false`         | Jika `true`, tampilkan `altIcon`                |
| `size`        | `number`         | `24`            | Ukuran icon dalam pixel (width & height)        |
| `color`       | `string`         | `currentColor`  | Warna icon, mendukung semua format CSS          |
| `strokeWidth` | `number`         | `1.5`           | Ketebalan stroke untuk icon tipe outlined       |
| `className`   | `string`         | —               | Tambahan CSS class                              |

## Pola-Pola Umum

### 1. Icon dengan State Toggle (showAlt)

Gunakan `altIcon` + `showAlt` untuk swap icon berdasarkan state — tanpa conditional rendering.

```tsx
import { HugeiconsIcon } from '@hugeicons/react';
import { Sun03Icon, Moon02Icon } from '@hugeicons/core-free-icons';

interface ThemeToggleProps {
  isDark: boolean;
  onToggle: () => void;
}

export function ThemeToggle({ isDark, onToggle }: ThemeToggleProps) {
  return (
    <button onClick={onToggle}>
      <HugeiconsIcon
        icon={Sun03Icon}
        altIcon={Moon02Icon}
        showAlt={isDark}
        size={20}
        color="currentColor"
      />
    </button>
  );
}
```

### 2. Favorite / Bookmark Toggle

```tsx
import { HugeiconsIcon } from '@hugeicons/react';
import { HeartIcon } from '@hugeicons/core-free-icons';

interface FavoriteButtonProps {
  isFavorited: boolean;
  onToggle: () => void;
}

export function FavoriteButton({ isFavorited, onToggle }: FavoriteButtonProps) {
  return (
    <button
      onClick={onToggle}
      className={isFavorited ? 'text-red-500' : 'text-gray-400'}
    >
      <HugeiconsIcon
        icon={HeartIcon}
        size={24}
        strokeWidth={isFavorited ? 0 : 1.5}
        color="currentColor"
      />
    </button>
  );
}
```

### 3. Search Input dengan Clear Button

```tsx
import { useState } from 'react';
import { HugeiconsIcon } from '@hugeicons/react';
import { Search01Icon, Cancel01Icon } from '@hugeicons/core-free-icons';

export function SearchInput() {
  const [query, setQuery] = useState('');

  return (
    <div className="relative flex items-center">
      <HugeiconsIcon
        icon={Search01Icon}
        altIcon={Cancel01Icon}
        showAlt={query.length > 0}
        size={18}
        color="currentColor"
        className="absolute left-3 text-gray-400"
      />
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Cari..."
        className="pl-9 pr-4 py-2 border rounded-lg"
      />
    </div>
  );
}
```

### 4. Bottom Navigation dengan Active State

```tsx
import { HugeiconsIcon } from '@hugeicons/react';
import { Home01Icon, Search01Icon, UserIcon } from '@hugeicons/core-free-icons';

const navItems = [
  { label: 'Beranda', icon: Home01Icon, path: '/' },
  { label: 'Cari', icon: Search01Icon, path: '/search' },
  { label: 'Profil', icon: UserIcon, path: '/profile' },
];

interface BottomNavProps {
  currentPath: string;
}

export function BottomNav({ currentPath }: BottomNavProps) {
  return (
    <nav className="flex justify-around py-3 border-t">
      {navItems.map((item) => {
        const isActive = currentPath === item.path;
        return (
          <a
            key={item.path}
            href={item.path}
            className={isActive ? 'text-blue-600' : 'text-gray-400'}
          >
            <HugeiconsIcon
              icon={item.icon}
              size={24}
              strokeWidth={isActive ? 2 : 1.5}
              color="currentColor"
            />
            <span className="text-xs">{item.label}</span>
          </a>
        );
      })}
    </nav>
  );
}
```

### 5. Icon di dalam Button / Link

Gunakan `color="currentColor"` agar icon mewarisi warna teks dari parent element.

```tsx
import { HugeiconsIcon } from '@hugeicons/react';
import { Add01Icon } from '@hugeicons/core-free-icons';

export function AddButton({ onClick }: { onClick: () => void }) {
  return (
    <button
      onClick={onClick}
      className="flex items-center gap-2 text-blue-600 hover:text-blue-800"
    >
      <HugeiconsIcon icon={Add01Icon} size={16} color="currentColor" />
      Tambah Data
    </button>
  );
}
```

## Cara Menemukan Nama Icon

1. Gunakan MCP tool `mcp__hugeicons__search_icons` untuk mencari icon
2. Verifikasi keberadaan file: `ls node_modules/@hugeicons/core-free-icons/dist/esm/<NamaIcon>.js`
3. Nama import menggunakan format: **PascalCase + suffix style + "Icon"**
   - Contoh: `home-01` (stroke rounded) → `Home01Icon`
   - Contoh: `notification-03` → `Notification03Icon`
   - Contoh: `user-circle` → `UserCircleIcon`

**WAJIB verifikasi** dengan cek file di `dist/esm/` sebelum menggunakan icon baru. Jangan asumsikan nama icon valid tanpa verifikasi.

## Icon yang Sudah Diverifikasi Valid (Proyek Ini)

Icon-icon berikut sudah terkonfirmasi ada di `@hugeicons/core-free-icons` dan digunakan di project:

```
Activity01Icon, Add01Icon, Alert01Icon, Alert02Icon, AlertCircleIcon,
ArrowDown01Icon, ArrowExpandIcon, ArrowLeft01Icon, ArrowReloadHorizontalIcon,
ArrowRight01Icon, ArrowRight02Icon, ArrowTurnBackwardIcon, ArrowUp01Icon,
ArrowUpDownIcon, ArrowUpRight01Icon, Attachment01Icon, Attachment02Icon,
AttachmentIcon, BarChartIcon, BookOpen01Icon, Briefcase01Icon, Building01Icon,
Building02Icon, Building03Icon, Calculator01Icon, Calendar01Icon, Calendar03Icon,
CallIcon, CallRinging01Icon, Cancel01Icon, ChartUpIcon, ChatQuestion01Icon,
CheckListIcon, CheckmarkCircle01Icon, CheckmarkCircle02Icon, CheckmarkSquare01Icon,
Clock01Icon, CloudIcon, CloudServerIcon, Coins01Icon, Comment01Icon,
ContactBookIcon, ContactIcon, ContractsIcon, Copy01Icon, CopyLinkIcon,
CreditCardIcon, DashboardSquare01Icon, Database01Icon, Delete01Icon, Delete02Icon,
DocumentValidationIcon, Dollar01Icon, DollarCircleIcon, Download01Icon,
DragDropIcon, DragDropVerticalIcon, DropboxIcon, DropperIcon, EarthIcon,
Edit01Icon, Edit02Icon, EraserIcon, EyeIcon, File01Icon, File02Icon,
FileAttachmentIcon, FileAudioIcon, FileEditIcon, FileRemoveIcon, FileValidationIcon,
FileVideoIcon, FileZipIcon, FilterIcon, FireIcon, FloppyDiskIcon,
Folder01Icon, FolderLibraryIcon, FolderOpenIcon, Globe02Icon, GlobeIcon,
GoogleDocIcon, HeadphonesIcon, Image01Icon, InformationCircleIcon, Invoice01Icon,
Invoice02Icon, Invoice03Icon, KanbanIcon, LanguageCircleIcon, LaptopIcon,
Layers01Icon, LayoutGridIcon, LayoutTwoRowIcon, LeftToRightListBulletIcon,
LeftToRightListNumberIcon, Link01Icon, LinkSquare01Icon, ListViewIcon, Loading01Icon,
Loading03Icon, Location01Icon, LockIcon, LockPasswordIcon, Logout01Icon, Mail01Icon,
MailReply01Icon, MailSend01Icon, MapPinIcon, MapsIcon, MapsLocation01Icon,
Maximize01Icon, Menu01Icon, Menu04Icon, Message01Icon, Message02Icon,
MessageAdd01Icon, MessageMultiple02Icon, MessageQuestionIcon, MinusSignIcon,
Moon01Icon, MoreHorizontalIcon, MoreVerticalIcon, Note01Icon, Notification01Icon,
Notification03Icon, PackageIcon, PaintBoardIcon, PencilEdit01Icon, PencilEdit02Icon,
PencilIcon, PercentIcon, Pin02Icon, PinIcon, PinOffIcon, PlayIcon,
PlusSignIcon, PresentationOnlineIcon, Refresh01Icon, RefreshIcon, Rotate01Icon,
RotateLeft01Icon, ScanIcon, Search01Icon, SecurityCheckIcon, SecurityLockIcon,
Settings01Icon, Share01Icon, Shield01Icon, ShieldUserIcon, SidebarRightIcon,
SignatureIcon, SmartPhone01Icon, SourceCodeIcon, StarIcon, StickyNote01Icon,
StopCircleIcon, StopIcon, Sun01Icon, Table01Icon, TableIcon, Tag01Icon,
Target01Icon, Task01Icon, TextBoldIcon, TextItalicIcon, ThumbsDownIcon,
ThumbsUpIcon, Tick01Icon, Tick02Icon, Ticket01Icon, Timer01Icon, Upload01Icon,
Upload04Icon, User02Icon, UserAdd01Icon, UserCheck01Icon, UserGroupIcon,
UserIcon, UserMultiple02Icon, ViewIcon, ViewOffIcon, Wallet01Icon,
ZapIcon, ZoomInAreaIcon
```

## Icon yang TIDAK ADA — Gunakan Penggantinya

Icon-icon berikut **tidak tersedia** di `@hugeicons/core-free-icons`. Gunakan pengganti yang sudah diverifikasi:

| Icon Salah (JANGAN PAKAI) | Pengganti yang Benar |
|---|---|
| `ActivityIcon` | `Activity01Icon` |
| `ArrowMoveIcon` | `DragDropIcon` |
| `Bell01Icon` | `Notification01Icon` |
| `CalendarIcon` | `Calendar01Icon` |
| `Call01Icon` | `CallRinging01Icon` |
| `CheckIcon` | `Tick01Icon` |
| `ContractIcon` | `ContractsIcon` |
| `CopyPlusIcon` | `Copy01Icon` |
| `CreditCardIcon` (standalone) | `Wallet01Icon` |
| `DollarSignIcon` | `Dollar01Icon` |
| `DownloadIcon` | `Download01Icon` |
| `EyeOffIcon` | `ViewOffIcon` |
| `FileCheckmarkIcon` | `FileValidationIcon` |
| `FileIcon` | `File01Icon` |
| `FileSignatureIcon` | `SignatureIcon` |
| `FileSpreadsheetIcon` | `File01Icon` |
| `FileText` (tanpa Icon) | `File02Icon` |
| `FileTextIcon` | `File01Icon` |
| `Globe01Icon` | `Globe02Icon` |
| `GridScanIcon` | `ScanIcon` |
| `GripVerticalIcon` | `DragDropVerticalIcon` |
| `HorizontalRuleIcon` | `MinusSignIcon` |
| `ImageIcon` | `Image01Icon` |
| `LayoutIcon` | `LayoutGridIcon` |
| `LinkIcon` | `Link01Icon` |
| `ListIcon` | `LeftToRightListBulletIcon` |
| `LogoutIcon` | `Logout01Icon` |
| `MailIcon` | `Mail01Icon` |
| `MessageSquare01Icon` | `Message01Icon` |
| `PlusIcon` | `PlusSignIcon` |
| `Receipt01Icon` | `Invoice01Icon` |
| `ReceiptText` (tanpa Icon) | `Invoice01Icon` |
| `SendIcon` | `MailSend01Icon` |
| `TargetIcon` | `Target01Icon` |
| `TaskListSquareIcon` | `CheckListIcon` |
| `Trash2Icon` | `Delete01Icon` |
| `TrendingUp01Icon` | `ChartUpIcon` |
| `UserCheckIcon` | `UserCheck01Icon` |
| `UserHugeIcon` | `UserIcon` (itu alias, bukan icon terpisah) |
| `XIcon` | `Cancel01Icon` |

## Aturan Penggunaan

### WAJIB
- Selalu import `HugeiconsIcon` dari `@hugeicons/react`
- Selalu import icon name dari `@hugeicons/core-free-icons` (atau paket pro sesuai style)
- Gunakan `color="currentColor"` untuk icon yang harus mengikuti warna teks parent
- Gunakan `showAlt` + `altIcon` untuk toggle — jangan conditional rendering `{condition ? <Icon A/> : <Icon B/>}`
- Gunakan TypeScript dengan tipe yang tepat
- **Verifikasi** nama icon baru dengan cek file `dist/esm/<NamaIcon>.js` sebelum digunakan

### DILARANG
- Jangan gunakan `<img>` atau `<svg>` raw untuk icon HugeIcons
- Jangan hardcode warna hex jika icon berada di dalam elemen yang sudah punya warna teks
- Jangan install paket pro jika icon yang dibutuhkan tersedia di paket free
- Jangan gunakan icon dari daftar "Icon yang TIDAK ADA" di atas
