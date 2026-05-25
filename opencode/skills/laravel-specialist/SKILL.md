---
name: laravel-specialist
description: Build and configure Laravel 10+ applications using Service Action Pattern. Controller hanya terima HTTP request, validasi wajib Form Request, business logic di Service/Action, read data via Repository+Interface. Use when creating Laravel models, APIs, queues, Livewire components, or writing Pest/PHPUnit tests.
metadata:
  author: https://github.com/Jeffallan (adapted for abdasis/claude-config)
  version: "1.2.0"
  domain: backend
  triggers: Laravel, Eloquent, PHP framework, Laravel API, Artisan, Blade templates, Laravel queues, Livewire, Laravel testing, Sanctum, Horizon
  role: specialist
  scope: implementation
  output-format: code
---

# Laravel Specialist

Senior Laravel specialist dengan deep expertise di Laravel 10+, Eloquent ORM, PHP 8.2+, dan **Service Action Pattern**.

## Arsitektur: Service Action Pattern

```
HTTP Request
    |
    v
[Form Request]  <-- validasi WAJIB di sini, TIDAK BOLEH inline di controller
    |
    v
[Controller]    <-- hanya terima request, panggil Service atau Action, return response
    |
    +-- logic sederhana --> [Action]      <-- single DB operation via Model
    |
    +-- logic kompleks  --> [Service]
                               |
                               v
                           [Action]      <-- Service WAJIB panggil Action, TIDAK BOLEH query langsung
                               |
                               v
                           [Model]       <-- Eloquent query ada di sini

Read data:
[Controller/Service] --> [Repository Interface] --> [Repository Implementation] --> [Model]
```

### Aturan Keras Tiap Layer

| Layer | Boleh | DILARANG |
|-------|-------|---------|
| **Controller** | Terima request, panggil Form Request, panggil Service/Action, return response | Query model langsung, validasi inline, business logic |
| **Form Request** | Validasi input, authorize | - |
| **Service** | Orchestrate business logic kompleks, panggil Action | Query DB langsung, menggunakan Model langsung |
| **Action** | Satu operasi DB tunggal via Model | Lebih dari satu query DB, business logic kompleks |
| **Repository** | Read data via Model (query, filter, paginate) | Write/mutate data |
| **Interface** | Kontrak repository | - |

## Core Workflow

1. **Analyse requirements** - Identifikasi models, relationships, APIs, dan queue needs
2. **Design architecture** - Tentukan layer: perlu Service atau Action langsung? Read via Repository?
3. **Implement models** - Eloquent models dengan relationships, scopes, casts
4. **Build Form Requests** - Satu Form Request per endpoint untuk validasi
5. **Build Actions** - Satu Action per operasi DB (CreatePost, UpdatePost, DeletePost)
6. **Build Repositories** - Repository + Interface untuk semua read data
7. **Build Services** - Hanya jika logic kompleks yang orchestrate beberapa Actions
8. **Build Controllers** - Tipis, hanya delegasi ke Service/Action
9. **Test thoroughly** - Feature + unit tests; `php artisan test` (target >85% coverage)

## Reference Guide

| Topic | Reference | Load When |
|-------|-----------|-----------|
| Eloquent ORM | `references/eloquent.md` | Models, relationships, scopes, query optimization |
| Routing & APIs | `references/routing.md` | Routes, controllers, middleware, API resources |
| Queue System | `references/queues.md` | Jobs, workers, Horizon, failed jobs, batching |
| Livewire | `references/livewire.md` | Components, wire:model, actions, real-time |
| Testing | `references/testing.md` | Feature tests, factories, mocking, Pest PHP |

## Constraints

### MUST DO
- Validasi WAJIB menggunakan Form Request — tidak boleh inline di controller
- Controller TIDAK BOLEH query model langsung
- Service TIDAK BOLEH query DB langsung — wajib panggil Action
- Action adalah single DB operation menggunakan Model
- Repository + Interface untuk semua operasi read data
- Logic sederhana: Controller panggil Action langsung
- Logic kompleks: Controller panggil Service, Service panggil Action
- Use PHP 8.2+ features (readonly, enums, typed properties)
- Type hint semua method parameters dan return types
- Eager loading untuk hindari N+1
- **SETIAP return response WAJIB menggunakan Resource** — tidak boleh return array, model, atau collection mentah. Sesuaikan dengan konteks project:
  - **Deteksi konteks:** cek apakah project menggunakan Inertia (`use Inertia\Inertia`) atau pure API (`use Illuminate\Http\JsonResponse`) sebelum implementasi
  - **API / JSON response:**
    - Single: `PostResource::make($model)` atau `new PostResource($model)`
    - Collection: `PostResource::collection($collection)`
    - Setelah create (HTTP 201): `(new PostResource($post))->response()->setStatusCode(201)`
    - Delete/no-content (HTTP 204): `response()->json(null, 204)` — satu-satunya pengecualian tanpa Resource
  - **Inertia response:**
    - Single: `Inertia::render('Post/Show', ['post' => PostResource::make($post)])`
    - Collection: `Inertia::render('Post/Index', ['posts' => PostResource::collection($posts)])`
    - Resource TETAP WAJIB dipakai sebagai data yang dikirim ke Inertia — jangan kirim model mentah
  - **Resource WAJIB dibuat** setiap kali ada entity baru yang punya controller — jangan tunda atau skip
- Queue long-running tasks
- Write comprehensive tests (>85% coverage)
- Gunakan service containers dan dependency injection
- Follow PSR-12 coding standards
- Explore existing codebase patterns BEFORE implementing
- Use `php -l` untuk syntax check setiap file baru

### MUST NOT DO
- Validasi inline di controller (`$request->validate([...])`)
- Query model langsung di controller (`Post::where(...)` di controller)
- Query DB langsung di service (`DB::table(...)` atau `Post::create(...)` di service)
- Lebih dari satu DB operation dalam satu Action
- Logika bisnis di controller
- Raw queries tanpa protection (SQL injection)
- Skip eager loading
- Simpan sensitive data tanpa enkripsi
- Hardcode configuration values
- Gunakan deprecated Laravel features
- Ignore queue failures
- Return array mentah dari controller: `return response()->json(['data' => $post])` — WAJIB pakai Resource
- Return model langsung: `return $post` — WAJIB pakai Resource
- Return collection mentah: `return $posts` — WAJIB pakai `Resource::collection()`
- Kirim model mentah ke Inertia: `Inertia::render('Page', ['post' => $post])` — WAJIB pakai Resource
- Kirim array manual ke Inertia: `Inertia::render('Page', ['post' => $post->toArray()])` — WAJIB pakai Resource

### PROJECT-SPECIFIC RULES (from CLAUDE.md)
- JANGAN auto-run `vendor/bin/pint` — beritahu user untuk menjalankan sendiri
- JANGAN auto-run `bun dev`, `bun run build`, atau `composer run dev`
- JANGAN auto-run migration — beritahu user: `php artisan migrate`
- Boleh: `php -l`, `php artisan test`, `php artisan route:list`, `php artisan tinker`

## Code Templates

### Form Request

```php
<?php

declare(strict_types=1);

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

final class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', Post::class);
    }

    public function rules(): array
    {
        return [
            'title'  => ['required', 'string', 'max:255'],
            'body'   => ['required', 'string'],
            'status' => ['required', \Illuminate\Validation\Rule::enum(PostStatus::class)],
        ];
    }
}
```

### Repository Interface

```php
<?php

declare(strict_types=1);

namespace App\Repositories\Contracts;

use App\Models\Post;
use Illuminate\Contracts\Pagination\LengthAwarePaginator;

interface PostRepositoryInterface
{
    public function findById(int $id): ?Post;

    public function findPublished(int $perPage = 15): LengthAwarePaginator;

    public function findByAuthor(int $userId, int $perPage = 15): LengthAwarePaginator;
}
```

### Repository Implementation

```php
<?php

declare(strict_types=1);

namespace App\Repositories;

use App\Models\Post;
use App\Repositories\Contracts\PostRepositoryInterface;
use Illuminate\Contracts\Pagination\LengthAwarePaginator;

final class PostRepository implements PostRepositoryInterface
{
    public function findById(int $id): ?Post
    {
        return Post::with(['author', 'comments'])->find($id);
    }

    public function findPublished(int $perPage = 15): LengthAwarePaginator
    {
        return Post::with('author')
            ->published()
            ->latest('published_at')
            ->paginate($perPage);
    }

    public function findByAuthor(int $userId, int $perPage = 15): LengthAwarePaginator
    {
        return Post::with('author')
            ->where('user_id', $userId)
            ->latest()
            ->paginate($perPage);
    }
}
```

### Bind Interface di ServiceProvider

```php
// App\Providers\AppServiceProvider::register()
$this->app->bind(
    \App\Repositories\Contracts\PostRepositoryInterface::class,
    \App\Repositories\PostRepository::class,
);
```

### Action (single DB operation)

```php
<?php

declare(strict_types=1);

namespace App\Actions\Post;

use App\Models\Post;
use App\Models\User;

final class CreatePostAction
{
    public function handle(User $author, array $data): Post
    {
        return $author->posts()->create([
            'title'  => $data['title'],
            'body'   => $data['body'],
            'status' => $data['status'],
        ]);
    }
}
```

```php
<?php

declare(strict_types=1);

namespace App\Actions\Post;

use App\Models\Post;

final class PublishPostAction
{
    public function handle(Post $post): Post
    {
        $post->update([
            'status'       => PostStatus::Published,
            'published_at' => now(),
        ]);

        return $post->refresh();
    }
}
```

### Service (complex business logic — orchestrate Actions)

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Actions\Post\CreatePostAction;
use App\Actions\Post\NotifySubscribersAction;
use App\Models\Post;
use App\Models\User;

final class PostService
{
    public function __construct(
        private readonly CreatePostAction        $createPost,
        private readonly NotifySubscribersAction $notifySubscribers,
    ) {}

    public function createAndNotify(User $author, array $data): Post
    {
        $post = $this->createPost->handle($author, $data);

        if ($data['status'] === PostStatus::Published->value) {
            $this->notifySubscribers->handle($post);
        }

        return $post;
    }
}
```

### Controller (thin — hanya delegasi)

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers\Api;

use App\Actions\Post\DeletePostAction;
use App\Http\Requests\StorePostRequest;
use App\Http\Requests\UpdatePostRequest;
use App\Http\Resources\PostResource;
use App\Repositories\Contracts\PostRepositoryInterface;
use App\Services\PostService;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Resources\Json\AnonymousResourceCollection;

final class PostController
{
    public function __construct(
        private readonly PostRepositoryInterface $posts,
        private readonly PostService             $postService,
        private readonly DeletePostAction        $deletePost,
    ) {}

    public function index(): AnonymousResourceCollection
    {
        return PostResource::collection(
            $this->posts->findPublished()
        );
    }

    public function show(int $id): PostResource
    {
        $post = $this->posts->findById($id);

        abort_if($post === null, 404);

        return new PostResource($post);
    }

    // Logic kompleks -> panggil Service
    public function store(StorePostRequest $request): JsonResponse
    {
        $post = $this->postService->createAndNotify(
            $request->user(),
            $request->validated(),
        );

        return (new PostResource($post))
            ->response()
            ->setStatusCode(201);
    }

    // Logic sederhana -> panggil Action langsung
    public function update(UpdatePostRequest $request, int $id): PostResource
    {
        $post = $this->posts->findById($id);

        abort_if($post === null, 404);

        $post = $this->updatePost->handle($post, $request->validated());

        return PostResource::make($post);
    }

    public function destroy(int $id): JsonResponse
    {
        $post = $this->posts->findById($id);

        abort_if($post === null, 404);

        $this->deletePost->handle($post);

        return response()->json(null, 204); // satu-satunya pengecualian: no-content response
    }
}
```

### Eloquent Model

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\SoftDeletes;

final class Post extends Model
{
    use HasFactory, SoftDeletes;

    protected $fillable = ['title', 'body', 'status', 'user_id', 'published_at'];

    protected $casts = [
        'status'       => PostStatus::class,
        'published_at' => 'immutable_datetime',
    ];

    public function author(): BelongsTo
    {
        return $this->belongsTo(User::class, 'user_id');
    }

    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }

    public function scopePublished(Builder $query): Builder
    {
        return $query->where('status', PostStatus::Published);
    }
}
```

### Migration

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('posts', function (Blueprint $table): void {
            $table->id();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->string('title');
            $table->text('body');
            $table->string('status')->default('draft');
            $table->timestamp('published_at')->nullable();
            $table->softDeletes();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('posts');
    }
};
```

### API Resource

Resource adalah layer transformasi WAJIB antara model Eloquent dan response HTTP/Inertia.
DILARANG return model atau array mentah dari controller.

```php
<?php

declare(strict_types=1);

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

final class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'           => $this->id,
            'title'        => $this->title,
            'body'         => $this->body,
            'status'       => $this->status->value,
            'published_at' => $this->published_at?->toIso8601String(),
            'created_at'   => $this->created_at->toIso8601String(),
            // Relasi: gunakan whenLoaded() agar tidak trigger N+1 jika tidak di-eager load
            'author'       => new UserResource($this->whenLoaded('author')),
            'comments'     => CommentResource::collection($this->whenLoaded('comments')),
            // Field kondisional berdasarkan role/context
            'secret_note'  => $this->when($request->user()?->isAdmin(), $this->secret_note),
        ];
    }
}
```

**Penggunaan di Controller:**
```php
// Single object
return PostResource::make($post);

// Collection / paginated
return PostResource::collection($this->posts->findPublished());

// Setelah create — HTTP 201
return (new PostResource($post))->response()->setStatusCode(201);

// Setelah delete — HTTP 204 (satu-satunya pengecualian tanpa Resource)
return response()->json(null, 204);

// Inertia — Resource tetap WAJIB sebagai data
return Inertia::render('Post/Index', [
    'posts' => PostResource::collection($posts),
    'post'  => PostResource::make($post),
]);
```

### Queued Job

```php
<?php

declare(strict_types=1);

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

final class PublishPost implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 3;
    public int $backoff = 60;

    public function __construct(
        private readonly Post $post,
    ) {}

    public function handle(PublishPostAction $action): void
    {
        $action->handle($this->post);
    }

    public function failed(\Throwable $e): void
    {
        logger()->error('PublishPost failed', ['post' => $this->post->id, 'error' => $e->getMessage()]);
    }
}
```

### Feature Test (Pest)

```php
<?php

use App\Models\Post;
use App\Models\User;

it('returns published posts', function (): void {
    $user = User::factory()->create();
    Post::factory()->published()->count(3)->for($user, 'author')->create();

    $this->getJson('/api/posts')
        ->assertOk()
        ->assertJsonCount(3, 'data');
});

it('creates a post via service action pattern', function (): void {
    $user = User::factory()->create();

    $this->actingAs($user)
        ->postJson('/api/posts', [
            'title'  => 'Hello World',
            'body'   => 'Content here',
            'status' => 'draft',
        ])
        ->assertCreated()
        ->assertJsonPath('data.title', 'Hello World');

    $this->assertDatabaseHas('posts', ['title' => 'Hello World', 'user_id' => $user->id]);
});

it('validates request via form request — rejects missing title', function (): void {
    $user = User::factory()->create();

    $this->actingAs($user)
        ->postJson('/api/posts', ['body' => 'no title'])
        ->assertUnprocessable()
        ->assertJsonValidationErrors(['title']);
});
```

## Struktur Folder

```
app/
├── Actions/
│   └── Post/
│       ├── CreatePostAction.php
│       ├── UpdatePostAction.php
│       ├── DeletePostAction.php
│       └── PublishPostAction.php
├── Services/
│   └── PostService.php
├── Repositories/
│   ├── Contracts/
│   │   └── PostRepositoryInterface.php
│   └── PostRepository.php
├── Http/
│   ├── Controllers/
│   │   └── Api/
│   │       └── PostController.php
│   ├── Requests/
│   │   ├── StorePostRequest.php
│   │   └── UpdatePostRequest.php
│   └── Resources/
│       └── PostResource.php
└── Models/
    └── Post.php
```

## Decision Tree: Pilih Layer yang Tepat

```
Butuh tulis ke DB?
├── Ya, satu operasi sederhana
│   └── Buat Action -> Controller panggil langsung
├── Ya, beberapa operasi / ada kondisi bisnis
│   └── Buat Service yang panggil beberapa Action
└── Tidak, hanya baca data
    └── Buat Repository + Interface -> Controller/Service gunakan Repository
```

## Validation Checkpoints

| Stage | Command | Expected Result |
|-------|---------|-----------------|
| After migration | `php artisan migrate:status` | All migrations show `Ran` |
| After routing | `php artisan route:list --path=api` | New routes appear with correct verbs |
| After job dispatch | `php artisan queue:work --once` | Job processes without exception |
| After implementation | `php artisan test --coverage` | >85% coverage, 0 failures |
| Before PR | Beritahu user jalankan: `./vendor/bin/pint` | PSR-12 linting passes |

## Knowledge Reference

Laravel 10+, Eloquent ORM, PHP 8.2+, Service Action Pattern, Repository Pattern, Form Requests, API resources, Sanctum/Passport, queues, Horizon, Livewire, Inertia, Octane, Pest/PHPUnit, Redis, broadcasting, events/listeners, notifications, task scheduling
