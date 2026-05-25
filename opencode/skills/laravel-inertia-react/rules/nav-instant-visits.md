---
section: navigation
priority: high
description: Instant visits allow Inertia to immediately render a target page component while the server request happens in the background
keywords: [instant, visits, component, pageProps, sharedProps, wayfinder, navigation, optimistic]
---

# Instant Visits

Instant visits memungkinkan Inertia langsung merender komponen halaman target tanpa menunggu server merespons. Server request tetap berjalan di background, dan ketika selesai, props asli di-merge ke halaman yang sudah ditampilkan.

Berbeda dengan client-side visits (yang tidak membuat server request sama sekali), instant visits tetap melakukan full server request — tapi pengguna tidak merasakan jeda karena halaman target langsung muncul.

## Bad Example

```tsx
// Anti-pattern: menunggu server sebelum merender halaman
// Pengguna melihat loading state yang tidak perlu untuk navigasi biasa
import { Link } from "@inertiajs/react";

<Link href="/dashboard">Dashboard</Link>
// Halaman baru hanya muncul setelah server merespons
```

## Good Example

### Basic Usage — via Link component

```tsx
// resources/js/Pages/Home.tsx
import { Link } from "@inertiajs/react";

export default function Home() {
  return (
    <Link href="/dashboard" component="Dashboard">
      Dashboard
    </Link>
  );
}
```

### Basic Usage — via router.visit()

```tsx
import { router } from "@inertiajs/react";

router.visit("/dashboard", {
  component: "Dashboard",
});
```

Inertia langsung merender komponen `Dashboard` menggunakan shared props yang tersedia. Props spesifik halaman akan di-merge ketika server merespons.

> **Catatan penting**: Komponen target harus bisa dirender tanpa page-specific props. Gunakan optional chaining (`post?.title`) atau conditional rendering untuk handle props yang belum tersedia.

### Dengan pageProps (Loading State)

```tsx
// Tampilkan placeholder sambil menunggu server
import { Link } from "@inertiajs/react";

export default function PostList({ posts }: { posts: Post[] }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          <Link
            href={`/posts/${post.id}`}
            component="Posts/Show"
            pageProps={{ title: post.title }}
          >
            {post.title}
          </Link>
        </li>
      ))}
    </ul>
  );
}
```

```tsx
// resources/js/Pages/Posts/Show.tsx
interface Props {
  title: string;   // tersedia langsung dari pageProps
  body?: string;   // muncul setelah server merespons — wajib optional
  author?: Author;
}

export default function Show({ title, body, author }: Props) {
  return (
    <article>
      <h1>{title}</h1>
      {body ? <p>{body}</p> : <p className="animate-pulse">Loading...</p>}
      {author && <span>By {author.name}</span>}
    </article>
  );
}
```

### pageProps dengan Callback (Spread Shared Props)

Jika ingin tetap meneruskan shared props sekaligus menambah placeholder:

```tsx
router.visit("/posts/1", {
  component: "Posts/Show",
  pageProps: (currentProps, sharedProps) => ({
    ...sharedProps,
    title: "Loading...",
  }),
});
```

Saat `pageProps` berupa object literal, shared props tidak otomatis diteruskan. Gunakan callback jika ingin keduanya.

### Wayfinder Integration

Jika menggunakan [Wayfinder](https://github.com/laravel/wayfinder), cukup tambahkan prop `instant` — Inertia akan mengambil nama komponen dari definisi route Wayfinder secara otomatis.

```tsx
import { Link } from "@inertiajs/react";
import { show } from "App/Http/Controllers/PostController";

<Link href={show(1)} instant>
  View Post
</Link>
```

Prop `instant` juga tersedia pada komponen `Form`:

```tsx
import { Form } from "@inertiajs/react";
import { store } from "App/Http/Controllers/PostController";

<Form action={store()} instant>
  {/* ... */}
</Form>
```

Jika ada `component` eksplisit, ia selalu mengalahkan `instant`.

#### Konfigurasi Wayfinder

Aktifkan `generate.inertia.component` di `config/wayfinder.php` agar route definition menyertakan nama komponen:

```php
// config/wayfinder.php
return [
    'generate' => [
        'inertia' => [
            'component' => true,
        ],
    ],
];
```

Untuk controller yang merender komponen berbeda secara kondisional, gunakan `withComponent()`:

```tsx
import { show } from "App/Http/Controllers/DashboardController";

router.visit(show.withComponent("Dashboard/Admin"));
```

### Laravel Backend — Shared Props

Laravel adapter otomatis menyertakan metadata `sharedProps` di response, berisi daftar key yang didaftarkan via `Inertia::share()`:

```json
{
  "component": "Dashboard",
  "props": { "auth": { "user": "..." }, "stats": {} },
  "sharedProps": ["auth"]
}
```

Inertia membaca list ini dan meneruskan shared props tersebut ke halaman intermediate.

Untuk menonaktifkan fitur ini (shared props tetap dikirim tapi daftar keynya disembunyikan):

```php
// config/inertia.php
return [
    'expose_shared_prop_keys' => false,
];
```

> Menonaktifkan ini akan mencegah instant visits mengetahui props mana yang bisa diteruskan.

## Why

1. **UX lebih responsif**: Pengguna melihat halaman target langsung tanpa loading blank yang terasa lambat
2. **Tetap server-driven**: Berbeda dengan client-side visits, data selalu fresh dari server — tidak ada stale data
3. **Loading state yang natural**: Dengan `pageProps`, kita bisa menampilkan data yang sudah diketahui (judul post, nama item) sambil menunggu detail lengkap
4. **Wayfinder DX**: Prop `instant` menghilangkan kebutuhan menentukan `component` secara manual — lebih aman dan kurang boilerplate
5. **Backward compatible**: Tidak memerlukan perubahan di server-side controller
