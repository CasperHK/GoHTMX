# VaporStack ⚡

**VaporStack** is a modern, high-concurrency, full-stack application template designed for developers who want the structure and ergonomics of an enterprise framework without the bloat of heavy Single Page Application (SPA) runtimes. 

It combines **Goravel** (Go), **HTMX**, and **Alpine.js** to deliver lightning-fast server-rendered HTML with seamless micro-interactivity and zero client-side build complexity.

---

## 🌟 The VaporStack Philosophy

Modern web development has largely defaulted to heavyweight stacks (like Next.js or React) even for standard forms, dashboards, and database-driven CRUD apps [00:01:18]. VaporStack takes a radically efficient approach:

1. **HTML Over the Wire:** The backend renders finished HTML fragments; HTMX swaps them directly into the DOM [00:02:52]. No client-side JSON serialization, no duplicate application state, and no hydration mismatches [00:03:11].
2. **Enterprise Structure:** Powered by **Goravel**, providing a clean, Laravel-inspired architecture [1.1.2] with a robust ORM, migrations, and service providers out of the box [1.1.2, 1.1.5].
3. **Micro-Interactivity Without Bloat:** **Alpine.js** handles transient client-side UI state (modals, dropdowns, tabs) instantly in the browser without requiring a server round-trip [00:08:02].
4. **Single Binary Deployment:** Goravel compiles down to a single static binary with no external runtimes required on your production server [00:04:32].

---

## 🏗️ Architecture & Technology Stack

* **Backend / MVC & Routing:** [Goravel](https://goravel.dev) (Go) — Modular, enterprise-grade Go framework [1.1.2].
* **Server-Driven Interactions:** [HTMX 2.x](https://htmx.org) — Attributes-driven AJAX requests and DOM swapping [00:01:32, 00:02:52].
* **Local UI State / Reactivity:** [Alpine.js](https://alpinejs.dev) — Lightweight declarative JavaScript for local DOM behavior.
* **Database & Persistence:** PostgreSQL via Goravel ORM.

---

## 📁 Project Directory Structure

```text
vaporstack/
├── app/
│   ├── http/
│   │   ├── controllers/      # Handles business logic and returns HTML views/partials
│   │   └── middleware/       # Custom HTTP middleware (Auth, Logging, etc.)
│   └── models/               # Database models and associations
├── database/
│   └── migrations/           # Database schema migrations [1.1.2]
├── resources/
│   └── views/                # Go html/template layouts and HTMX partial fragments
├── routes/
│   ├── web.go                # HTTP routes for server-rendered HTML & HTMX endpoints
│   └── api.go                # Optional JSON API endpoints if needed
├── config/                   # Framework configuration files (database, app, logging)
├── .env.example              # Environment variables template
├── go.mod
└── main.go                   # Application entry point

```

---

## 🚀 Getting Started

### Prerequisites

Ensure you have the following installed on your machine:

* Go (1.22 or higher)
* PostgreSQL
* [Goravel CLI](https://goravel.dev) (optional, for running artisan commands)

### 1. Clone & Setup Repository

```bash
git clone [https://github.com/your-username/vaporstack.git](https://github.com/your-username/vaporstack.git)
cd vaporstack

```

### 2. Configure Environment Variables

Copy the example environment file and update your database credentials:

```bash
cp .env.example .env

```

Edit `.env`:

```env
APP_NAME=VaporStack
APP_ENV=local
APP_DEBUG=true
APP_PORT=8080

DB_CONNECTION=postgres
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=vaporstack_db
DB_USERNAME=postgres
DB_PASSWORD=secret

```

### 3. Run Database Migrations

```bash
go run main.go artisan migrate

```

### 4. Start the Development Server

```bash
go run main.go

```

Navigate to `http://localhost:8080` in your browser.

---

## 💡 Code Examples

### 1. Defining an HTMX Route (`routes/web.go`)

```go
package routes

import (
	"[github.com/goravel/framework/facades](https://github.com/goravel/framework/facades)"
	"vaporstack/app/http/controllers"
)

func Web() {
	itemController := controllers.NewItemController()

	// HTMX post request endpoint
	facades.Route().Post("/items", itemController.Store)
}

```

### 2. Returning an HTML Partial (`app/http/controllers/item_controller.go`)

```go
package controllers

import (
	"[github.com/goravel/framework/contracts/http](https://github.com/goravel/framework/contracts/http)"
)

type ItemController struct{}

func NewItemController() *ItemController {
	return &ItemController{}
}

func (c *ItemController) Store(ctx http.Context) http.Response {
	name := ctx.Request().Input("name")

	// Save to database using Goravel ORM...

	// Return a small HTML fragment for HTMX to swap into the DOM
	return ctx.Response().View().Make("partials/item_row.tmpl", map[string]any{
		"Name": name,
	})
}

```

### 3. Combining Alpine.js & HTMX (`resources/views/welcome.tmpl`)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>VaporStack App</title>
    <!-- HTMX & Alpine CDN -->
    <script src="[https://unpkg.com/htmx.org@2.0.4](https://unpkg.com/htmx.org@2.0.4)"></script>
    <script defer src="[https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js](https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js)"></script>
</head>
<body class="bg-slate-50 p-8">

    <!-- Alpine.js managing local modal state (Zero server round-trip) -->
    <div x-data="{ openModal: false }" class="mb-6">
        <button @click="openModal = true" class="bg-indigo-600 text-white px-4 py-2 rounded-lg shadow hover:bg-indigo-700 transition">
            Open Settings Modal
        </button>

        <div x-show="openModal" class="fixed inset-0 bg-slate-900/50 flex items-center justify-center p-4">
            <div @click.outside="openModal = false" class="bg-white p-6 rounded-xl shadow-xl max-w-md w-full">
                <h3 class="text-lg font-bold text-slate-800">Local UI State (Alpine)</h3>
                <p class="text-slate-600 text-sm mt-1">Managed entirely in the browser without server latency.</p>
                <button @click="openModal = false" class="mt-4 bg-slate-200 text-slate-700 px-4 py-2 rounded-lg text-sm font-medium">Close</button>
            </div>
        </div>
    </div>

    <!-- HTMX handling server-driven form submission -->
    <form hx-post="/items" hx-target="#item-list" hx-swap="beforeend" class="flex gap-2 max-w-lg mb-6">
        <input type="text" name="name" placeholder="Enter item name..." class="border border-slate-300 p-2 rounded-lg flex-1 focus:ring-2 focus:ring-indigo-500 outline-none" required>
        <button type="submit" class="bg-emerald-600 text-white px-4 py-2 rounded-lg font-medium hover:bg-emerald-700 transition">Add Item</button>
    </form>

    <!-- Target container where HTMX injects server-rendered HTML fragments -->
    <ul id="item-list" class="space-y-2 max-w-lg">
        <!-- Rendered items appear here dynamically -->
    </ul>

</body>
</html>

```

---

## 📦 Deployment

VaporStack compiles into a single, highly optimized static binary.

1. **Build the binary:**
```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o vaporstack-app main.go

```


2. **Transfer to your server:** Copy the binary, your `.env` file, and the `resources/views` directory to your production server.
3. **Run with process manager (e.g., systemd):**
```bash
./vaporstack-app

```



---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE).
