---
section: forms
priority: high
description: Using the Inertia Form component for declarative form submissions without full page reloads
keywords: [form, component, action, method, wayfinder, precognition, validation, slot props, resetOnSuccess, transform]
---

# Form Component

Inertia menyediakan komponen `<Form>` yang berfungsi seperti HTML form klasik, tapi menggunakan Inertia di bawahnya untuk menghindari full page reload. Ini cara paling sederhana untuk membuat form di Inertia tanpa perlu state management manual dengan `useForm`.

Gunakan `<Form>` ketika form kamu cukup sederhana dan tidak butuh kontrol programmatic yang kompleks. Untuk kebutuhan yang lebih advanced (complex state, dynamic fields), gunakan [`useForm`](./form-useform-basic.md).

## Bad Example

```tsx
// Anti-pattern: manual state untuk form sederhana padahal bisa pakai <Form>
import { useState, FormEvent } from "react";
import { router } from "@inertiajs/react";

export default function Create() {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    router.post("/users", { name, email });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <button type="submit">Create</button>
    </form>
  );
}
```

## Good Example

### Basic Usage

```tsx
// resources/js/Pages/Users/Create.tsx
import { Form } from "@inertiajs/react";

export default function Create() {
  return (
    <Form action="/users" method="post">
      <input type="text" name="name" />
      <input type="email" name="email" />
      <button type="submit">Create User</button>
    </Form>
  );
}
```

Seperti HTML form tradisional, tidak perlu `onChange` handler — cukup beri `name` attribute dan komponen `Form` akan otomatis handle submission.

### Nested Data & File Uploads

```tsx
<Form action="/reports" method="post">
  <input type="text" name="name" />
  <textarea name="report[description]"></textarea>
  <input type="text" name="report[tags][]" />
  <input type="file" name="documents" multiple />
  <button type="submit">Create Report</button>
</Form>
```

### Dotted Key Notation

Dotted notation otomatis membentuk nested object:

```tsx
<Form action="/users" method="post">
  <input type="text" name="user.name" />
  <input type="text" name="user.skills[]" />
  <input type="text" name="address.street" />
  <button type="submit">Submit</button>
</Form>
```

Menghasilkan struktur:

```json
{
  "user": { "name": "John", "skills": ["JS"] },
  "address": { "street": "123 Main St" }
}
```

Untuk literal dot (bukan separator), escape dengan backslash: `name="app\.name"`.

### Default Values

Gunakan `defaultValue` dan `defaultChecked` — JANGAN `value` (karena komponen Form tidak controlled):

```tsx
<Form action="/users" method="post">
  <input type="text" name="name" defaultValue="John Doe" />

  <select name="country" defaultValue="uk">
    <option value="us">United States</option>
    <option value="uk">United Kingdom</option>
  </select>

  <input type="checkbox" name="subscribe" value="1" defaultChecked />

  <button type="submit">Submit</button>
</Form>
```

> **Penting**: Untuk checkbox, selalu tambahkan `value="1"` eksplisit. Tanpa value, checkbox ter-check akan submit sebagai `"on"` yang mungkin tidak dikenali sebagai boolean oleh validasi Laravel.

### Wayfinder Integration

Dengan [Wayfinder](https://github.com/laravel/wayfinder), pass Wayfinder object langsung ke `action` — HTTP method dan URL otomatis di-infer:

```tsx
import { Form } from "@inertiajs/react";
import { store } from "App/Http/Controllers/UserController";

export default function Create() {
  return (
    <Form action={store()}>
      <input type="text" name="name" />
      <input type="email" name="email" />
      <button type="submit">Create User</button>
    </Form>
  );
}
```

### Transform Before Submit

Gunakan prop `transform` untuk memodifikasi data sebelum dikirim (misalnya inject user_id atau timestamp):

```tsx
<Form
  action="/posts"
  method="post"
  transform={(data) => ({ ...data, user_id: 123, timestamp: Date.now() })}
>
  <input type="text" name="title" />
  <button type="submit">Create Post</button>
</Form>
```

### Slot Props (Render Props)

`<Form>` mengekspos reactive state dan helper methods via render props — gunakan untuk menampilkan errors, processing state, dan success indicators:

```tsx
<Form action="/users" method="post">
  {({
    errors,
    hasErrors,
    processing,
    progress,
    wasSuccessful,
    recentlySuccessful,
    setError,
    clearErrors,
    resetAndClearErrors,
    defaults,
    isDirty,
    reset,
    submit,
  }) => (
    <>
      <input type="text" name="name" />
      {errors.name && <p className="text-red-500">{errors.name}</p>}

      <button type="submit" disabled={processing}>
        {processing ? "Creating..." : "Create User"}
      </button>

      {wasSuccessful && <p className="text-green-500">User created!</p>}
      {isDirty && <p className="text-sm text-gray-500">Unsaved changes</p>}
    </>
  )}
</Form>
```

Untuk nested error menggunakan dotted notation:

```tsx
<Form action="/users" method="post">
  {({ errors }) => (
    <>
      <input type="text" name="user.name" />
      {errors["user.name"] && <p>{errors["user.name"]}</p>}
    </>
  )}
</Form>
```

### Reset Behavior

```tsx
// Reset semua field setelah sukses
<Form action="/users" method="post" resetOnSuccess>...</Form>

// Reset field spesifik setelah sukses
<Form action="/users" method="post" resetOnSuccess={["name"]}>...</Form>

// Reset setelah error
<Form action="/users" method="post" resetOnError>...</Form>

// Jadikan current values sebagai defaults baru setelah sukses
<Form action="/users" method="post" setDefaultsOnSuccess>...</Form>
```

### Props & Options

Props top-level untuk kontrol form submission; `options` untuk kontrol visit Inertia setelahnya:

```tsx
<Form
  action="/profile"
  method="put"
  errorBag="profile"
  headers={{ "X-Custom-Header": "value" }}
  showProgress={false}
  transform={(data) => ({ ...data, timestamp: Date.now() })}
  invalidateCacheTags={["users", "dashboard"]}
  disableWhileProcessing
  options={{
    preserveScroll: true,
    preserveState: true,
    only: ["users", "flash"],
  }}
>
  <input type="text" name="name" />
  <button type="submit">Update</button>
</Form>
```

`disableWhileProcessing` menambahkan atribut `inert` pada form saat processing. Style dengan Tailwind:

```tsx
<Form
  action="/profile"
  method="put"
  disableWhileProcessing
  className="inert:opacity-50 inert:pointer-events-none"
>
  {/* ... */}
</Form>
```

### Event Callbacks

```tsx
<Form
  action="/users"
  method="post"
  onBefore={(visit) => console.log("before", visit)}
  onStart={() => console.log("start")}
  onProgress={(event) => console.log("progress", event)}
  onSuccess={(page) => console.log("success", page)}
  onError={(errors) => console.log("errors", errors)}
  onFinish={() => console.log("finish")}
  onCancel={() => console.log("cancel")}
>
  {/* ... */}
</Form>
```

### Programmatic Access via Ref

```tsx
import { useRef } from "react";
import { Form } from "@inertiajs/react";

export default function CreateUser() {
  const formRef = useRef<HTMLFormElement>(null);

  const handleExternalSubmit = () => {
    formRef.current?.submit();
  };

  return (
    <>
      <Form ref={formRef} action="/users" method="post">
        <input type="text" name="name" />
      </Form>

      <button onClick={handleExternalSubmit}>Submit from outside</button>
    </>
  );
}
```

### Form Context (Nested Components)

Untuk akses state form dari child component dalam tanpa prop drilling:

```tsx
// resources/js/Components/FormActions.tsx
import { useFormContext } from "@inertiajs/react";

export default function FormActions() {
  const form = useFormContext();

  if (!form) return null;

  return (
    <div className="flex gap-2">
      {form.isDirty && <span className="text-sm">Unsaved changes</span>}
      <button type="button" onClick={() => form.submit()}>Submit</button>
      <button type="button" onClick={() => form.reset()}>Reset</button>
    </div>
  );
}
```

### Precognition (Real-time Validation)

Butuh [Laravel Precognition](https://laravel.com/docs/precognition) di sisi server.

```tsx
<Form action="/users" method="post">
  {({ errors, invalid, valid, validate, validating }) => (
    <>
      <label htmlFor="email">Email:</label>
      <input
        id="email"
        name="email"
        onChange={() => validate("email")}
      />
      {valid("email") && <p className="text-green-500">Valid</p>}
      {invalid("email") && <p className="text-red-500">{errors.email}</p>}

      {validating && <p>Validating...</p>}

      <button type="submit">Create User</button>
    </>
  )}
</Form>
```

Validasi multiple fields sekaligus (useful untuk wizard):

```tsx
<button
  type="button"
  onClick={() =>
    validate({
      only: ["name", "email"],
      onSuccess: () => goToNextStep(),
      onValidationError: () => showErrors(),
    })
  }
>
  Next Step
</button>
```

Options tambahan: `validationTimeout` (default 1500ms), `validateFiles`, `withAllErrors`.

### Laravel Backend Controller

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\StoreUserRequest;
use App\Http\Resources\UserResource;
use App\Models\User;
use Illuminate\Http\RedirectResponse;

class UserController extends Controller
{
    public function store(StoreUserRequest $request): RedirectResponse
    {
        User::create($request->validated());

        return redirect()
            ->route('users.index')
            ->with('success', 'User created successfully.');
    }
}
```

## Kapan Pakai `<Form>` vs `useForm`

| Skenario | Gunakan |
|----------|---------|
| Form CRUD sederhana, field statis | `<Form>` |
| Butuh default values dari props halaman | `<Form>` dengan `defaultValue` |
| File upload sederhana | `<Form>` |
| Dynamic fields (add/remove) | `useForm` |
| Kompleks state transformation | `useForm` |
| Butuh kontrol programmatic penuh | `useForm` |
| Multi-step form dengan conditional logic | `useForm` |

## Why

1. **Less boilerplate**: Tidak perlu `useState` + `onChange` handler untuk setiap input
2. **HTML-native**: Pakai `name` attribute seperti form tradisional — familiar untuk semua developer
3. **Built-in features**: Reset, transform, precognition, slot props — semua tersedia tanpa konfigurasi tambahan
4. **Dotted notation**: Nested data structure dibangun otomatis dari flat input names
5. **Wayfinder ready**: Integrasi mulus dengan type-safe routing Laravel
6. **Form context**: Child components bisa akses form state tanpa prop drilling
7. **Progressive enhancement**: Form tetap berfungsi mirip HTML form biasa, dengan semua keunggulan SPA
