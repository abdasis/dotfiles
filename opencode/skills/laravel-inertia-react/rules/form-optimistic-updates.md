---
section: forms
priority: high
description: Menerapkan optimistic updates untuk update UI instan sebelum server merespons, dengan rollback otomatis jika request gagal
keywords: [optimistic, optimistic update, instant ui, rollback, useForm, router, useHttp, Form component]
---

# Optimistic Updates

Optimistic updates memungkinkan UI di-update **langsung** tanpa menunggu response dari server — misalnya incrementing like counter, toggle bookmark, atau menambah item ke list. Inertia menerapkan perubahan secara instan ketika request masih in-flight, dan **otomatis rollback** jika request gagal.

Gunakan pola ini untuk aksi yang cepat, sering dilakukan user, dan hampir selalu sukses di server. Jangan gunakan untuk aksi yang butuh konfirmasi server (pembayaran, operasi destructive, dsb).

## Bad Example

```tsx
// Anti-pattern: nunggu response server baru update UI, bikin terasa lambat
import { router } from "@inertiajs/react";
import { useState } from "react";

function LikeButton({ post }: { post: Post }) {
  const [likes, setLikes] = useState(post.likes);
  const [loading, setLoading] = useState(false);

  const handleLike = () => {
    setLoading(true);
    router.post(`/posts/${post.id}/like`, {}, {
      onSuccess: () => {
        setLikes(likes + 1); // user harus nunggu dulu
        setLoading(false);
      },
      onError: () => setLoading(false),
    });
  };

  return (
    <button onClick={handleLike} disabled={loading}>
      {likes} likes
    </button>
  );
}
```

Masalah:
- User harus nunggu roundtrip ke server sebelum melihat perubahan
- State lokal duplikat dengan props dari server (rawan out-of-sync)
- Tidak ada rollback otomatis kalau request gagal
- Loading state bikin UI terasa lambat untuk aksi sederhana

## Good Example

### 1. Router Visits dengan `.optimistic()`

```tsx
import { router } from "@inertiajs/react";
import { Post } from "@/types/post";

interface Props {
  post: Post;
}

function LikeButton({ post }: Props) {
  const handleLike = () => {
    router
      .optimistic((props) => ({
        post: {
          ...props.post,
          likes: props.post.likes + 1,
        },
      }))
      .post(`/posts/${post.id}/like`);
  };

  return <button onClick={handleLike}>{post.likes} likes</button>;
}
```

Callback menerima **current page props** dan return **partial update** yang akan di-merge. UI langsung re-render dengan nilai baru sebelum request dikirim.

### 2. Form Component dengan prop `optimistic`

```tsx
import { Form } from "@inertiajs/react";
import { Todo } from "@/types/todo";

interface Props {
  todos: Todo[];
}

export default function TodoList({ todos }: Props) {
  return (
    <>
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.name}</li>
        ))}
      </ul>

      <Form
        action="/todos"
        method="post"
        optimistic={(props, data) => ({
          todos: [
            ...props.todos,
            { id: Date.now(), name: data.name, done: false },
          ],
        })}
      >
        <input type="text" name="name" />
        <button type="submit">Add Todo</button>
      </Form>
    </>
  );
}
```

Karena `<Form>` mengelola input data internal, callback menerima **dua argumen**: `props` (page props) dan `data` (form data).

### 3. `useForm` dengan `.optimistic()`

```tsx
import { useForm } from "@inertiajs/react";
import { FormEvent } from "react";
import { Post } from "@/types/post";

interface Props {
  posts: Post[];
}

export default function CreatePost({ posts }: Props) {
  const { data, setData, optimistic, post, processing, errors } = useForm({
    title: "",
  });

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();

    optimistic((props) => ({
      posts: [...props.posts, { id: Date.now(), title: data.title }],
    }));

    post("/posts", {
      onSuccess: () => setData("title", ""),
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={data.title}
        onChange={(e) => setData("title", e.target.value)}
      />
      {errors.title && <p className="text-red-500">{errors.title}</p>}
      <button type="submit" disabled={processing}>
        Save
      </button>
    </form>
  );
}
```

### 4. Inline Option dalam Visit Options

Callback juga bisa diletakkan langsung di visit options tanpa chaining:

```tsx
router.post(
  `/posts/${post.id}/like`,
  {},
  {
    optimistic: (props) => ({
      post: { ...props.post, likes: props.post.likes + 1 },
    }),
  }
);
```

### 5. `useHttp` untuk HTTP Requests

Untuk request yang tidak memakai Inertia props (misal API call murni), `useHttp` menerima callback yang beroperasi pada **form data**-nya sendiri, bukan page props:

```tsx
import { useHttp } from "@inertiajs/react";

function LikeCounter() {
  const { data, optimistic, post } = useHttp({
    likes: 0,
  });

  const handleLike = () => {
    optimistic((currentData) => ({
      likes: currentData.likes + 1,
    }));
    post("/api/likes");
  };

  return <button onClick={handleLike}>{data.likes} likes</button>;
}
```

Jika request gagal, form data otomatis dikembalikan ke nilai sebelum request.

## Cara Kerja Optimistic Updates

Ketika optimistic update diterapkan, Inertia melakukan langkah berikut:

1. **Snapshot** — Inertia membandingkan nilai return callback dengan current props, lalu menyimpan snapshot hanya untuk key yang berubah
2. **Merge** — Return value callback di-shallow-merge ke current data
3. **Request dikirim** — Request dilanjutkan ke server seperti biasa
4. **On success** — Response server menggantikan data optimistic
5. **On failure** — Hanya key yang di-snapshot yang di-rollback ke nilai asli

Callback **harus** return partial object (hanya key yang ingin diupdate), bukan full state.

## Automatic Rollback

Inertia otomatis rollback optimistic state pada beberapa kondisi:

- **Validation errors (422)** — state di-rollback, validation errors di-preserve
- **Server errors** — request gagal karena reason lain, original state dipulihkan
- **Interrupted visits** — request baru yang menginterupsi request in-flight, state sebelumnya dipulihkan dulu sebelum optimistic update baru diterapkan

Tidak perlu handle rollback manual — cukup percaya mekanisme Inertia.

## Concurrent Updates

Beberapa optimistic request bisa in-flight bersamaan. Inertia **tracks** prop mana yang di-touch setiap optimistic update, dan response server **tidak akan** overwrite sebuah prop sampai optimistic request terakhir yang memodifikasinya selesai.

Artinya aman untuk klik tombol like berkali-kali tanpa khawatir race condition di UI.

## Kapan Pakai dan Kapan Tidak

### Pakai untuk:
- Like, bookmark, favorite, vote (aksi cepat & sering)
- Toggle state (mark as read, pin/unpin, archive)
- Menambah item ke list yang sederhana (comment, todo)
- Counter increment/decrement
- Reorder list (drag-and-drop)

### Jangan pakai untuk:
- Pembayaran atau transaksi finansial
- Operasi destructive (delete permanent) tanpa konfirmasi
- Aksi yang butuh generated ID dari server untuk operasi lanjutan
- Form dengan validasi kompleks yang sering gagal
- Aksi yang butuh user lihat hasil server sebelum lanjut

## Pola Lengkap Like Button

```tsx
// resources/js/types/post.d.ts
export interface Post {
  id: number;
  title: string;
  likes: number;
  liked_by_user: boolean;
}
```

```tsx
// resources/js/Components/LikeButton.tsx
import { router } from "@inertiajs/react";
import { Post } from "@/types/post";

interface Props {
  post: Post;
}

export default function LikeButton({ post }: Props) {
  const handleToggle = () => {
    router
      .optimistic((props) => ({
        post: {
          ...props.post,
          liked_by_user: !props.post.liked_by_user,
          likes: props.post.liked_by_user
            ? props.post.likes - 1
            : props.post.likes + 1,
        },
      }))
      .post(`/posts/${post.id}/toggle-like`, {}, {
        preserveScroll: true,
      });
  };

  return (
    <button
      onClick={handleToggle}
      className={post.liked_by_user ? "text-red-500" : "text-gray-500"}
    >
      {post.liked_by_user ? "Unlike" : "Like"} ({post.likes})
    </button>
  );
}
```

Controller Laravel cukup return redirect (Inertia akan auto-refresh props):

```php
public function toggleLike(Post $post)
{
    $user = auth()->user();

    if ($post->likedBy($user)) {
        $post->likes()->where('user_id', $user->id)->delete();
    } else {
        $post->likes()->create(['user_id' => $user->id]);
    }

    return back();
}
```

## Tips

- Selalu return **partial object**, jangan full props
- Gunakan spread operator untuk preserve field yang tidak berubah: `{ ...props.post, likes: ... }`
- Untuk list operations, pakai `Date.now()` atau temporary ID untuk item baru — server akan menggantinya dengan ID asli saat response
- Kombinasikan dengan `preserveScroll: true` agar user tidak kehilangan posisi scroll
- Test edge case: request gagal, validation error, concurrent clicks

## Referensi

- [Inertia.js Optimistic Updates](https://inertiajs.com/optimistic-updates)
- [Inertia.js useHttp](https://inertiajs.com/http-requests)
