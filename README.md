# 🚀 Nuxt Dashboard

เทมเพลต Admin Dashboard แบบ Full-Stack ที่สร้างด้วย **Nuxt 4**, **Nuxt UI** และ **Prisma** เชื่อมต่อกับ **Microsoft SQL Server** มาพร้อมระบบ Authentication ที่ปลอดภัย, ระบบควบคุมสิทธิ์ตามบทบาท (RBAC) และโครงสร้าง Layered Architecture ที่สะอาด ขยายตัวได้ง่าย

---

## ✨ คุณสมบัติหลัก

- 🔐 **ระบบเข้าสู่ระบบที่ปลอดภัย** — Login ด้วย Username/Password พร้อมการเข้ารหัสแบบ AES ฝั่ง Client และ Bcrypt Hash ฝั่ง Server
- 👥 **ควบคุมสิทธิ์ตามบทบาท (RBAC)** — 3 บทบาท: `ADMIN`, `EDITOR`, `USER`
- 📋 **จัดการผู้ใช้ (Users)** — CRUD ครบครัน พร้อม Pagination, ค้นหา, และกรองข้อมูลฝั่ง Server
- 🧑‍🤝‍🧑 **จัดการลูกค้า (Customers)** — CRUD ครบครัน รูปแบบเดียวกับโมดูล Users
- 🏗️ **โครงสร้างแบบ Layered** — API → Service → Repository → Database
- 🧪 **ระบบ Unit Testing** — Vitest พร้อม Mocking แยก Folder `tests/` เรียบร้อย
- 🎨 **ธีมสำหรับงาน Admin** — โทนสี Indigo/Slate เน้นความชัดเจนและใช้งานซ้ำๆ ได้เร็ว ฟอนต์ Outfit + Public Sans
- 🤝 **รองรับภาษาลาว** — ฟอนต์ Noto Sans Lao สำหรับ UI และข้อความในระบบ

---

## 🛠️ เทคโนโลยีที่ใช้

| ส่วน | เทคโนโลยี |
|---|---|
| Frontend | Nuxt 4 + Vue 3 |
| UI Library | @nuxt/ui (Tailwind CSS v4) |
| ORM | Prisma v7 |
| ฐานข้อมูล | Microsoft SQL Server |
| Authentication | nuxt-auth-utils |
| Hashing รหัสผ่าน | bcryptjs |
| เข้ารหัสข้อมูล | crypto-js (AES-256) |
| Validation | Zod |
| Testing | Vitest + happy-dom |
| Package Manager | Bun |

---

## 📁 โครงสร้าง Project

```
dashboard/
├── app/
│   ├── components/         # Vue Components (users/, customers/, ฯลฯ)
│   ├── composables/        # useApi, useApiAction
│   ├── layouts/            # Layout หลัก (Sidebar + Navbar)
│   ├── pages/              # login.vue, users.vue, customers.vue
│   └── utils/              # ฟังก์ชัน Crypto ฝั่ง Frontend
├── server/
│   ├── api/                # REST Endpoints (auth/, users/, customers/)
│   ├── middleware/         # Global Auth Middleware
│   ├── services/           # Business Logic (UserService, CustomerService)
│   └── utils/              # BaseRepository, Repositories, Validation, Crypto, Pagination
├── prisma/
│   ├── schema.prisma       # Database Schema
│   └── seed.ts             # Seed ข้อมูลตั้งต้น (Admin + ตัวอย่าง Customers)
├── tests/
│   └── unit/
│       ├── services/       # user.service.spec.ts
│       └── utils/          # crypto.spec.ts
├── vitest.config.ts
├── prisma.config.ts
└── .env
```

---

## 📏 มาตรฐานทีม (Standards)

ทุกคนและ **AI ทุกตัว** (Antigravity, Claude Code, Cursor, Copilot, Codex ฯลฯ) ต้องทำตามมาตรฐานเดียวกัน:

- [`AGENTS.md`](AGENTS.md) — จุดเริ่มต้นที่ AI อ่านก่อนเขียนโค้ด
- [`docs/standards/coding-standards.md`](docs/standards/coding-standards.md) — สถาปัตยกรรม, API, Service, Frontend, Testing
- [`docs/standards/ui-standards.md`](docs/standards/ui-standards.md) — มาตรฐาน UI/UX

`CLAUDE.md`, `GEMINI.md`, `.cursor/`, `.github/copilot-instructions.md` เป็นเพียงตัวชี้มาที่ไฟล์ข้างบน — **แก้มาตรฐานที่ `docs/standards/` เท่านั้น**

ก่อนส่งงานต้องผ่าน: `bun run lint && bun run typecheck && bun run test`

---

## ⚙️ ความต้องการของระบบ

- [Bun](https://bun.sh) v1.2 ขึ้นไป
- [Node.js](https://nodejs.org) v20 ขึ้นไป
- Microsoft SQL Server (Local หรือ Remote)

---

## 📦 วิธีติดตั้งและใช้งาน

### ขั้นตอนที่ 1: Clone โปรเจกต์

```bash
git clone <your-repo-url>
cd dashboard
```

### ขั้นตอนที่ 2: ติดตั้ง Dependencies

```bash
bun install
```

### ขั้นตอนที่ 3: ตั้งค่า Environment Variables

คัดลอกไฟล์ `.env.example` และกรอกข้อมูลให้ครบ:

```bash
copy .env.example .env
```

แก้ไขไฟล์ `.env` ดังนี้:

```env
# การเชื่อมต่อ SQL Server
DB_SERVER=ip-ของ-sql-server
DB_PORT=1433
DB_DATABASE=db_nuxt_template
DB_USER=sa
DB_PASSWORD=รหัสผ่าน-ของคุณ
DATABASE_URL="sqlserver://ip-ของ-sql-server:1433;database=db_nuxt_template;user=sa;password=รหัสผ่าน;trustServerCertificate=true;encrypt=false"

# Session Secret (ต้องมีความยาวอย่างน้อย 32 ตัวอักษร)
NUXT_SESSION_PASSWORD=สตริงสุ่มของคุณอย่างน้อย32ตัวอักษร

# AES Encryption Key (ต้องเป็นค่าเดียวกันทั้ง Frontend และ Backend)
SECRET_KEY=สตริงลับสำหรับเข้ารหัส
```

> ⚠️ **สำคัญมาก**: `SECRET_KEY` ต้องมีค่าเดียวกันทั้งในไฟล์ `app/utils/crypto.ts` และ `server/utils/crypto.ts` และห้ามเด็ดขาดที่จะ Commit ค่านี้ขึ้น Git หรือโพสต์ไว้ที่สาธารณะ

### ขั้นตอนที่ 4: สร้างตารางในฐานข้อมูล

```bash
bunx prisma db push
```

### ขั้นตอนที่ 5: เพิ่มข้อมูลตั้งต้น (Seed)

สร้าง Admin ค่าเริ่มต้นและข้อมูลตัวอย่าง Customers:

```bash
bun prisma/seed.ts
```

ข้อมูล Admin ค่าเริ่มต้น:
- **Username**: `admin`
- **Password**: `password123`

> 💡 แนะนำให้เปลี่ยนรหัสผ่านทันทีหลังจาก Login ครั้งแรก

### ขั้นตอนที่ 6: เริ่ม Development Server

```bash
bun run dev
```

เปิด [http://localhost:3000](http://localhost:3000) ในเบราว์เซอร์

---

## 🧪 การรัน Unit Tests

```bash
# รัน Test ทั้งหมดครั้งเดียว
bun run test

# รัน Test แบบ Watch Mode (รันอัตโนมัติเมื่อโค้ดเปลี่ยน)
bun run test --watch

# เปิด Vitest UI Dashboard
bun run test:ui
```

---

## 👥 สิทธิ์การใช้งานตามบทบาท

| การกระทำ | ADMIN | EDITOR | USER |
|---|:---:|:---:|:---:|
| ดูรายชื่อผู้ใช้ | ✅ | ✅ | ✅ |
| สร้าง / แก้ไขผู้ใช้ | ✅ | ❌ | ❌ |
| ลบผู้ใช้ | ✅ | ❌ | ❌ |
| ดูรายชื่อลูกค้า | ✅ | ✅ | ✅ |
| สร้าง / แก้ไขลูกค้า | ✅ | ✅ | ❌ |
| ลบลูกค้า | ✅ | ❌ | ❌ |

---

## 🔑 ระบบความปลอดภัยของรหัสผ่าน

```
ฝั่ง Frontend                     ฝั่ง Backend
─────────                          ───────
ผู้ใช้กรอกรหัสผ่าน
         │
  เข้ารหัสด้วย AES (SECRET_KEY)
         │
   ─── ส่งผ่าน HTTP POST ──►   ถอดรหัส AES (SECRET_KEY)
                                         │
                                   Hash ด้วย Bcrypt
                                         │
                                   บันทึกลงฐานข้อมูล
```

เมื่อ Login ระบบจะถอดรหัส AES ก่อน แล้วนำไปเปรียบเทียบกับ Bcrypt Hash ในฐานข้อมูล โดยยังรองรับรหัสผ่านแบบเก่า (Plain Text) เพื่อความสะดวกในการ Migration

---

## 📝 คำสั่งที่ใช้บ่อย

| คำสั่ง | คำอธิบาย |
|---|---|
| `bun run dev` | เริ่ม Development Server |
| `bun run build` | Build สำหรับ Production |
| `bun run preview` | ดูตัวอย่าง Production Build |
| `bun run test` | รัน Unit Tests |
| `bun run test:ui` | เปิด Vitest UI |
| `bun run lint` | ตรวจสอบโค้ดด้วย ESLint |
| `bun run typecheck` | ตรวจสอบ TypeScript Types |
| `bunx prisma db push` | Sync Schema ไปยังฐานข้อมูล |
| `bunx prisma studio` | เปิด Prisma Studio (GUI สำหรับดูข้อมูล) |
| `bun prisma/seed.ts` | เพิ่มข้อมูลตั้งต้น |
