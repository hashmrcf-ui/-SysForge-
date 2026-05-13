---
name: sysweb
description: SysForge Web — بنّاء الويب. يبني Next.js 14 + FastAPI + PostgreSQL + AI Agent. متخصص في أنظمة الأعمال SaaS. Called by /sysforge in PHASE 7B. Triggers on "/sysweb", "/next-skill", "build web system", "ابني بالويب".
---

# SysWeb — بنّاء الويب

## ما هذه المهارة

أداة متخصصة لبناء أنظمة الويب باستخدام:
- **Next.js 14** (App Router) — الواجهة الأمامية
- **FastAPI** (Python) — الخادم الخلفي
- **PostgreSQL** — قاعدة البيانات
- **Claude / GPT-4** — وكيل الذكاء الاصطناعي

تُستدعى من مهارة `/build` تلقائياً، أو مباشرة للمطورين.

---

## بروتوكول البناء

### الخطوة 1 — بيئة العمل

تحقق من توفر المتطلبات:

```bash
node --version    # يجب أن يكون 18+
python --version  # يجب أن يكون 3.11+
docker --version  # اختياري لكن مُفضّل
```

إذا كان شيء ناقصاً، أخبر المتدرب بكيفية تثبيته قبل المتابعة.

---

### الخطوة 2 — إنشاء المشروع

**للخيار A** (نفس المشروع):
```bash
npx create-next-app@latest [project-name] \
  --typescript \
  --tailwind \
  --app \
  --src-dir \
  --import-alias "@/*"

cd [project-name]
npx shadcn@latest init -d
npm install prisma @prisma/client
npm install next-auth @auth/prisma-adapter
npm install @ai-sdk/anthropic ai zod
npx prisma init
```

**للخيار B/C** (منفصل):
```bash
# Frontend
npx create-next-app@latest frontend --typescript --tailwind --app
cd frontend && npx shadcn@latest init -d
npm install axios @tanstack/react-query zustand

# Backend
mkdir backend && cd backend
python -m venv venv
# Windows:
venv\Scripts\activate
pip install fastapi "uvicorn[standard]" sqlalchemy psycopg2-binary
pip install "python-jose[cryptography]" passlib alembic
pip install anthropic langchain-anthropic python-dotenv
pip freeze > requirements.txt
```

---

### الخطوة 3 — هيكل المجلدات المعياري

اعرض الهيكل الكامل قبل إنشاء أي ملف وانتظر الموافقة.

**Option A:**
```
[project]/
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   └── layout.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx          ← sidebar + header
│   │   │   ├── page.tsx            ← dashboard home
│   │   │   └── [module]/
│   │   │       ├── page.tsx        ← list view
│   │   │       ├── [id]/page.tsx   ← detail view
│   │   │       └── new/page.tsx    ← create form
│   │   └── api/
│   │       ├── auth/[...nextauth]/route.ts
│   │       └── [module]/route.ts
│   ├── components/
│   │   ├── ui/                     ← shadcn (auto-generated)
│   │   ├── layout/
│   │   │   ├── Sidebar.tsx
│   │   │   ├── Header.tsx
│   │   │   └── MobileNav.tsx
│   │   └── [module]/
│   │       ├── [Module]Table.tsx
│   │       ├── [Module]Form.tsx
│   │       └── [Module]Card.tsx
│   ├── lib/
│   │   ├── db.ts                   ← Prisma client
│   │   ├── auth.ts                 ← NextAuth config
│   │   └── ai.ts                   ← AI client setup
│   └── types/
│       └── index.ts
├── prisma/
│   └── schema.prisma
└── .env.local
```

---

### الخطوة 4 — قاعدة البيانات

**Option A — Prisma Schema:**

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  role      String   @default("user")
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

// أضف نماذج النظام هنا حسب البلوبرينت المختار
// انظر: systems/[نوع النظام].md
```

**Option B/C — SQLAlchemy Models:**

```python
# backend/models/base.py
from sqlalchemy import Column, String, DateTime, func
from sqlalchemy.dialects.postgresql import UUID
import uuid
from core.database import Base

class TimestampMixin:
    created_at = Column(DateTime, server_default=func.now())
    updated_at = Column(DateTime, onupdate=func.now())

# backend/models/user.py
class User(Base, TimestampMixin):
    __tablename__ = "users"
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    email = Column(String, unique=True, nullable=False)
    name = Column(String)
    role = Column(String, default="user")
    hashed_password = Column(String)
```

---

### الخطوة 5 — وكيل الذكاء الاصطناعي

**إعداد الوكيل الأساسي (يصلح لجميع الخيارات):**

```python
# backend/services/ai_service.py
from anthropic import Anthropic
from typing import List, Dict

client = Anthropic()

SYSTEM_PROMPT = """أنت مساعد ذكي متخصص في [نوع النظام].
مهمتك مساعدة المستخدمين في:
- [مهمة 1]
- [مهمة 2]
- [مهمة 3]

أجب دائماً بالعربية ما لم يطلب المستخدم غير ذلك.
كن دقيقاً ومختصراً."""

async def chat_with_ai(
    messages: List[Dict],
    system_context: str = ""
) -> str:
    response = client.messages.create(
        model="claude-opus-4-5",
        max_tokens=2048,
        system=SYSTEM_PROMPT + "\n\n" + system_context,
        messages=messages
    )
    return response.content[0].text

async def analyze_data(data: dict, query: str) -> str:
    """تحليل البيانات والإجابة على أسئلة محددة"""
    context = f"البيانات المتاحة:\n{data}\n\nالسؤال: {query}"
    return await chat_with_ai(
        [{"role": "user", "content": context}]
    )
```

**FastAPI endpoint:**
```python
# backend/routers/ai.py
from fastapi import APIRouter, Depends
from services.ai_service import chat_with_ai, analyze_data

router = APIRouter(prefix="/ai", tags=["AI"])

@router.post("/chat")
async def chat(messages: list):
    response = await chat_with_ai(messages)
    return {"response": response}

@router.post("/analyze")
async def analyze(data: dict, query: str):
    response = await analyze_data(data, query)
    return {"analysis": response}
```

**Next.js AI Chat component:**
```typescript
// src/components/AIChat.tsx
"use client"
import { useState } from "react"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { ScrollArea } from "@/components/ui/scroll-area"

interface Message {
  role: "user" | "assistant"
  content: string
}

export function AIChat() {
  const [messages, setMessages] = useState<Message[]>([])
  const [input, setInput] = useState("")
  const [loading, setLoading] = useState(false)

  const sendMessage = async () => {
    if (!input.trim()) return
    const userMsg = { role: "user" as const, content: input }
    setMessages(prev => [...prev, userMsg])
    setInput("")
    setLoading(true)

    const res = await fetch("/api/ai/chat", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ messages: [...messages, userMsg] })
    })
    const data = await res.json()
    setMessages(prev => [...prev, { role: "assistant", content: data.response }])
    setLoading(false)
  }

  return (
    <div className="flex flex-col h-full border rounded-lg">
      <ScrollArea className="flex-1 p-4">
        {messages.map((m, i) => (
          <div key={i} className={`mb-3 ${m.role === "user" ? "text-right" : "text-left"}`}>
            <span className={`inline-block p-3 rounded-lg max-w-[80%] ${
              m.role === "user"
                ? "bg-primary text-primary-foreground"
                : "bg-muted"
            }`}>
              {m.content}
            </span>
          </div>
        ))}
        {loading && <div className="text-muted-foreground text-sm">يفكر...</div>}
      </ScrollArea>
      <div className="flex gap-2 p-4 border-t">
        <Input
          value={input}
          onChange={e => setInput(e.target.value)}
          onKeyDown={e => e.key === "Enter" && sendMessage()}
          placeholder="اسأل المساعد الذكي..."
          className="flex-1"
          dir="rtl"
        />
        <Button onClick={sendMessage} disabled={loading}>إرسال</Button>
      </div>
    </div>
  )
}
```

---

### الخطوة 6 — Layout الأساسي (Dashboard)

```typescript
// src/app/(dashboard)/layout.tsx
import { Sidebar } from "@/components/layout/Sidebar"
import { Header } from "@/components/layout/Header"

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <div className="flex h-screen overflow-hidden" dir="rtl">
      <Sidebar />
      <div className="flex flex-col flex-1 overflow-hidden">
        <Header />
        <main className="flex-1 overflow-y-auto p-6 bg-background">
          {children}
        </main>
      </div>
    </div>
  )
}
```

```typescript
// src/components/layout/Sidebar.tsx
"use client"
import Link from "next/link"
import { usePathname } from "next/navigation"
import { cn } from "@/lib/utils"
import {
  LayoutDashboard, Users, FileText,
  Settings, Bot
} from "lucide-react"

const navItems = [
  { href: "/dashboard", icon: LayoutDashboard, label: "الرئيسية" },
  // أضف روابط الوحدات هنا
  { href: "/dashboard/ai-assistant", icon: Bot, label: "المساعد الذكي" },
  { href: "/dashboard/settings", icon: Settings, label: "الإعدادات" },
]

export function Sidebar() {
  const pathname = usePathname()
  return (
    <aside className="w-64 border-l bg-card flex flex-col">
      <div className="p-4 border-b">
        <h1 className="font-bold text-lg">اسم النظام</h1>
      </div>
      <nav className="flex-1 p-2 space-y-1">
        {navItems.map(({ href, icon: Icon, label }) => (
          <Link
            key={href}
            href={href}
            className={cn(
              "flex items-center gap-3 px-3 py-2 rounded-md text-sm transition-colors",
              pathname === href
                ? "bg-primary text-primary-foreground"
                : "hover:bg-muted"
            )}
          >
            <Icon className="w-4 h-4" />
            {label}
          </Link>
        ))}
      </nav>
    </aside>
  )
}
```

---

### الخطوة 7 — Pattern لبناء كل وحدة

اتبع هذا النمط لكل وحدة جديدة:

```
1. Prisma model (أو SQLAlchemy model)     ← تعريف الجدول
2. API endpoint (Next.js route أو FastAPI) ← CRUD endpoints
3. Data table component                    ← قائمة العناصر
4. Form component                          ← إضافة / تعديل
5. Page (list + create + detail)           ← ربط الكل
6. إضافة الرابط للـ Sidebar
```

**مثال تطبيقي — وحدة العملاء:**

```typescript
// 1. Prisma model
model Client {
  id        String   @id @default(cuid())
  name      String
  email     String?
  phone     String?
  address   String?
  createdAt DateTime @default(now())
}

// 2. API Route
// src/app/api/clients/route.ts
import { NextResponse } from "next/server"
import { db } from "@/lib/db"

export async function GET() {
  const clients = await db.client.findMany({ orderBy: { createdAt: "desc" } })
  return NextResponse.json(clients)
}

export async function POST(req: Request) {
  const body = await req.json()
  const client = await db.client.create({ data: body })
  return NextResponse.json(client, { status: 201 })
}
```

---

### الخطوة 8 — التحقق من كل وحدة

بعد بناء كل وحدة، تحقق من:

```
✅ API يرجع بيانات صحيحة
✅ الصفحة تعرض البيانات
✅ النموذج يحفظ البيانات
✅ رسائل الخطأ واضحة للمستخدم
✅ RTL صحيح (النصوص والأزرار من اليمين)
```

---

### الخطوة 9 — النشر

**Option A — Vercel:**
```bash
npm install -g vercel
vercel
# اتبع التعليمات + أضف متغيرات البيئة في لوحة Vercel
```

**Option B/C:**
```bash
# Frontend → Vercel
cd frontend && vercel

# Backend → Railway
# 1. ارفع الكود لـ GitHub
# 2. اربطه بـ Railway
# 3. أضف متغيرات البيئة

# Database → Railway PostgreSQL (مجاني للبداية)
```

---

## قواعد RTL

كل شيء باللغة العربية يجب أن يكون:
- `dir="rtl"` على الـ body أو الـ container الرئيسي
- `text-right` لنصوص العربية
- `space-x-reverse` لـ flex rows
- الـ Sidebar على اليمين

```typescript
// globals.css أو layout.tsx
<html lang="ar" dir="rtl">
```

---

## Checklist قبل الشحن

```
□ كل صفحة تعمل بدون أخطاء في الـ console
□ التحميل لا يتجاوز 3 ثوانٍ
□ النماذج تُظهر رسائل خطأ واضحة
□ الـ RTL صحيح في جميع الصفحات
□ وكيل AI يرد بشكل صحيح
□ يعمل على الموبايل (responsive)
□ لا توجد بيانات حساسة في الكود
□ .env.local موجود في .gitignore
```
