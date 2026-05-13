---
name: syscore
description: SysForge Core — أساس الأنظمة. يبني الطبقة الأساسية الكاملة: DB+RLS, Auth, Multi-tenant, Permissions, Base API, Frontend Shell. بعد اكتماله كل feature = 3 ملفات فقط. Triggers on "/syscore", "/saas-core", "build core engine", "ابني الأساس", called by /sysforge in PHASE 7A.
---

# SysCore — أساس الأنظمة

## الهدف

بناء **الطبقة الأساسية الكاملة** التي تجلس فوقها كل features المشروع.

**القاعدة الذهبية:** بعد اكتمال Core Engine، أي feature جديدة = 3 ملفات فقط.

---

## قبل البدء

**اقرأ أولاً:**
- `context.md` — لمعرفة نوع النظام والمعمارية المختارة
- `architecture.md` — لمعرفة Domain Model, Multi-tenant strategy, Auth flow

**تحقق من الخيار المختار:**
- Option A → استخدم أوامر الـ `Next.js API Routes`
- Option B/C → استخدم أوامر الـ `FastAPI`

---

## الطبقة 1 — DB Foundation (قاعدة البيانات)

### الخطوة: أنشئ الجداول الأساسية

**Option B/C — FastAPI + PostgreSQL:**

```sql
-- migrations/001_core_foundation.sql

-- جدول الشركات (Tenants)
CREATE TABLE tenants (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name        VARCHAR(255) NOT NULL,
  slug        VARCHAR(100) UNIQUE NOT NULL,  -- اسم النطاق الفرعي
  plan        VARCHAR(50) DEFAULT 'free',
  is_active   BOOLEAN DEFAULT true,
  settings    JSONB DEFAULT '{}',
  created_at  TIMESTAMP DEFAULT NOW(),
  updated_at  TIMESTAMP DEFAULT NOW()
);

-- جدول المستخدمين
CREATE TABLE users (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id       UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  email           VARCHAR(255) NOT NULL,
  hashed_password VARCHAR(255) NOT NULL,
  name            VARCHAR(255),
  role            VARCHAR(50) DEFAULT 'user',
  is_active       BOOLEAN DEFAULT true,
  last_login      TIMESTAMP,
  created_at      TIMESTAMP DEFAULT NOW(),
  updated_at      TIMESTAMP DEFAULT NOW(),
  UNIQUE(tenant_id, email)  -- نفس الإيميل مسموح في شركات مختلفة
);

-- فهرس للأداء
CREATE INDEX idx_users_tenant_id ON users(tenant_id);
CREATE INDEX idx_users_email ON users(email);

-- دالة تحديث updated_at تلقائياً
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_tenants_updated_at
  BEFORE UPDATE ON tenants
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_users_updated_at
  BEFORE UPDATE ON users
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

### RLS Policies (Row-Level Security)

```sql
-- تفعيل RLS على كل جدول يحتوي tenant_id
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- سياسة العزل: كل مستخدم يرى بيانات شركته فقط
CREATE POLICY tenant_isolation_users ON users
  USING (tenant_id = NULLIF(current_setting('app.tenant_id', true), '')::UUID);

-- Super Admin يرى الكل (بدون RLS)
CREATE POLICY super_admin_bypass ON users
  USING (current_setting('app.role', true) = 'super_admin');
```

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

model Tenant {
  id        String   @id @default(cuid())
  name      String
  slug      String   @unique
  plan      String   @default("free")
  isActive  Boolean  @default(true)
  settings  Json     @default("{}")
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  users     User[]
}

model User {
  id             String   @id @default(cuid())
  tenantId       String
  tenant         Tenant   @relation(fields: [tenantId], references: [id])
  email          String
  hashedPassword String
  name           String?
  role           String   @default("user")
  isActive       Boolean  @default(true)
  lastLogin      DateTime?
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt

  @@unique([tenantId, email])
  @@index([tenantId])
}
```

---

## الطبقة 2 — Auth System (نظام المصادقة)

**Option B/C — FastAPI:**

```python
# backend/core/security.py
from datetime import datetime, timedelta
from typing import Optional
import jwt
from passlib.context import CryptContext
from fastapi import HTTPException, status

SECRET_KEY = "your-super-secret-key"  # من .env
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 1440   # 24 ساعة
REFRESH_TOKEN_EXPIRE_DAYS = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)

def create_access_token(user_id: str, tenant_id: str, role: str) -> str:
    payload = {
        "sub": user_id,
        "tenant_id": tenant_id,
        "role": role,
        "type": "access",
        "exp": datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

def create_refresh_token(user_id: str) -> str:
    payload = {
        "sub": user_id,
        "type": "refresh",
        "exp": datetime.utcnow() + timedelta(days=REFRESH_TOKEN_EXPIRE_DAYS)
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

def decode_token(token: str) -> dict:
    try:
        return jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="انتهت صلاحية الجلسة")
    except jwt.JWTError:
        raise HTTPException(status_code=401, detail="توكن غير صالح")
```

```python
# backend/routers/auth.py
from fastapi import APIRouter, Depends, HTTPException
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel
from core.security import (
    verify_password, create_access_token,
    create_refresh_token, decode_token
)
from core.database import get_db

router = APIRouter(prefix="/auth", tags=["Auth"])
security = HTTPBearer()

class LoginRequest(BaseModel):
    email: str
    password: str

class TokenResponse(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str = "bearer"

@router.post("/login", response_model=TokenResponse)
async def login(data: LoginRequest, db=Depends(get_db)):
    # ابحث عن المستخدم
    user = await db.fetchrow(
        "SELECT * FROM users WHERE email = $1 AND is_active = true",
        data.email
    )
    if not user or not verify_password(data.password, user["hashed_password"]):
        raise HTTPException(status_code=401, detail="بيانات الدخول غير صحيحة")

    # حدّث آخر تسجيل دخول
    await db.execute(
        "UPDATE users SET last_login = NOW() WHERE id = $1",
        user["id"]
    )

    return TokenResponse(
        access_token=create_access_token(
            str(user["id"]), str(user["tenant_id"]), user["role"]
        ),
        refresh_token=create_refresh_token(str(user["id"]))
    )

@router.post("/refresh")
async def refresh_token(credentials: HTTPAuthorizationCredentials = Depends(security)):
    payload = decode_token(credentials.credentials)
    if payload.get("type") != "refresh":
        raise HTTPException(status_code=401, detail="توكن غير صالح")

    user = await db.fetchrow("SELECT * FROM users WHERE id = $1", payload["sub"])
    if not user:
        raise HTTPException(status_code=401, detail="المستخدم غير موجود")

    return {
        "access_token": create_access_token(
            str(user["id"]), str(user["tenant_id"]), user["role"]
        )
    }

@router.get("/me")
async def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security), db=Depends(get_db)):
    payload = decode_token(credentials.credentials)
    user = await db.fetchrow(
        "SELECT id, email, name, role, tenant_id, created_at FROM users WHERE id = $1",
        payload["sub"]
    )
    if not user:
        raise HTTPException(status_code=404, detail="المستخدم غير موجود")
    return dict(user)

@router.post("/logout")
async def logout():
    # في JWT لا يوجد logout حقيقي — نترك الـ token ينتهي
    # لو أردت blacklist: استخدم Redis
    return {"message": "تم تسجيل الخروج"}
```

---

## الطبقة 3 — Multi-Tenant Middleware

**Option B/C — FastAPI:**

```python
# backend/middleware/tenant.py
from fastapi import Request, HTTPException
from starlette.middleware.base import BaseHTTPMiddleware
from core.security import decode_token
import re

# المسارات التي لا تحتاج tenant_id
PUBLIC_PATHS = ["/auth/login", "/auth/refresh", "/health", "/docs", "/openapi.json"]

class TenantMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        # تجاوز المسارات العامة
        if any(request.url.path.startswith(p) for p in PUBLIC_PATHS):
            return await call_next(request)

        # استخرج الـ token
        auth_header = request.headers.get("Authorization")
        if not auth_header or not auth_header.startswith("Bearer "):
            raise HTTPException(status_code=401, detail="مطلوب تسجيل الدخول")

        token = auth_header.split(" ")[1]
        payload = decode_token(token)

        # أضف tenant_id و role للـ request state
        request.state.tenant_id = payload["tenant_id"]
        request.state.user_id = payload["sub"]
        request.state.role = payload["role"]

        # عيّن tenant_id في DB session (لـ RLS)
        response = await call_next(request)
        return response


# backend/core/database.py
import asyncpg
from fastapi import Request

DATABASE_URL = "postgresql://user:password@localhost:5432/mydb"
pool = None

async def create_pool():
    global pool
    pool = await asyncpg.create_pool(DATABASE_URL)

async def get_db(request: Request):
    async with pool.acquire() as conn:
        # عيّن tenant_id لـ RLS تلقائياً
        tenant_id = getattr(request.state, "tenant_id", None)
        role = getattr(request.state, "role", "user")

        if tenant_id:
            await conn.execute(f"SET app.tenant_id = '{tenant_id}'")
        await conn.execute(f"SET app.role = '{role}'")

        yield conn
```

---

## الطبقة 4 — Permission Guard

**FastAPI — Decorators:**

```python
# backend/core/permissions.py
from functools import wraps
from fastapi import HTTPException, Request, Depends
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

ROLE_HIERARCHY = {
    "super_admin": 4,
    "tenant_admin": 3,
    "user": 2,
    "viewer": 1
}

def require_role(*allowed_roles: str):
    """Decorator: يمنع الوصول إذا لم يكن الدور مسموحاً به"""
    async def dependency(request: Request):
        user_role = getattr(request.state, "role", "viewer")
        if user_role not in allowed_roles and user_role != "super_admin":
            raise HTTPException(
                status_code=403,
                detail=f"غير مسموح — يحتاج: {', '.join(allowed_roles)}"
            )
        return user_role
    return Depends(dependency)

# الاستخدام:
# @router.post("/users")
# async def create_user(role=require_role("tenant_admin", "super_admin")):
#     ...
```

**Next.js — Hook:**

```typescript
// src/hooks/usePermission.ts
import { useSession } from "next-auth/react"

type Role = "super_admin" | "tenant_admin" | "user" | "viewer"

const ROLE_HIERARCHY: Record<Role, number> = {
  super_admin: 4,
  tenant_admin: 3,
  user: 2,
  viewer: 1
}

export function usePermission() {
  const { data: session } = useSession()
  const userRole = (session?.user?.role as Role) || "viewer"

  return {
    can: (minRole: Role) =>
      ROLE_HIERARCHY[userRole] >= ROLE_HIERARCHY[minRole],
    role: userRole,
    isAdmin: userRole === "super_admin",
    isTenantAdmin: ["super_admin", "tenant_admin"].includes(userRole),
  }
}

// الاستخدام:
// const { can } = usePermission()
// {can("tenant_admin") && <Button>إضافة مستخدم</Button>}
```

---

## الطبقة 5 — Base API Structure

**FastAPI — main.py:**

```python
# backend/main.py
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import JSONResponse
from middleware.tenant import TenantMiddleware
from core.database import create_pool
from routers import auth

app = FastAPI(
    title="[اسم النظام] API",
    version="1.0.0",
    docs_url="/docs"
)

# CORS
app.add_middleware(CORSMiddleware, allow_origins=["http://localhost:3000"], allow_credentials=True, allow_methods=["*"], allow_headers=["*"])

# Tenant Middleware
app.add_middleware(TenantMiddleware)

# Base Response Format
def success_response(data=None, message="تمت العملية بنجاح", meta=None):
    return {"success": True, "data": data, "message": message, "errors": None, "meta": meta}

def error_response(message: str, errors=None, code: int = 400):
    return JSONResponse(
        status_code=code,
        content={"success": False, "data": None, "message": message, "errors": errors}
    )

# Global Error Handler
@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    return error_response("حدث خطأ غير متوقع", str(exc), 500)

# Startup
@app.on_event("startup")
async def startup():
    await create_pool()

# Health Check
@app.get("/health")
async def health():
    return {"status": "ok", "version": "1.0.0"}

# Routers
app.include_router(auth.router, prefix="/api/v1")
# أضف روuters الوحدات هنا لاحقاً
```

---

## الطبقة 6 — Frontend Shell

**Next.js — API Client:**

```typescript
// src/lib/api-client.ts
const API_URL = process.env.NEXT_PUBLIC_API_URL || "http://localhost:8000/api/v1"

class ApiClient {
  private getToken(): string | null {
    if (typeof window === "undefined") return null
    return localStorage.getItem("access_token")
  }

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const token = this.getToken()
    const response = await fetch(`${API_URL}${endpoint}`, {
      ...options,
      headers: {
        "Content-Type": "application/json",
        ...(token ? { Authorization: `Bearer ${token}` } : {}),
        ...options.headers,
      },
    })

    // تجديد الـ token تلقائياً إذا انتهى
    if (response.status === 401) {
      await this.refreshToken()
      return this.request(endpoint, options)
    }

    const data = await response.json()
    if (!data.success) throw new Error(data.message)
    return data.data
  }

  private async refreshToken() {
    const refresh = localStorage.getItem("refresh_token")
    if (!refresh) { window.location.href = "/login"; return }
    const res = await fetch(`${API_URL}/auth/refresh`, {
      method: "POST",
      headers: { Authorization: `Bearer ${refresh}` }
    })
    const data = await res.json()
    if (data.access_token) {
      localStorage.setItem("access_token", data.access_token)
    } else {
      window.location.href = "/login"
    }
  }

  get<T>(endpoint: string) { return this.request<T>(endpoint) }
  post<T>(endpoint: string, body: unknown) {
    return this.request<T>(endpoint, { method: "POST", body: JSON.stringify(body) })
  }
  put<T>(endpoint: string, body: unknown) {
    return this.request<T>(endpoint, { method: "PUT", body: JSON.stringify(body) })
  }
  delete<T>(endpoint: string) {
    return this.request<T>(endpoint, { method: "DELETE" })
  }
}

export const api = new ApiClient()
```

**Next.js — Protected Layout:**

```typescript
// src/app/(dashboard)/layout.tsx
import { redirect } from "next/navigation"
import { getServerSession } from "next-auth"
import { Sidebar } from "@/components/layout/Sidebar"
import { Header } from "@/components/layout/Header"

export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const session = await getServerSession()
  if (!session) redirect("/login")

  return (
    <div className="flex h-screen overflow-hidden bg-background" dir="rtl">
      <Sidebar />
      <div className="flex flex-col flex-1 overflow-hidden">
        <Header user={session.user} />
        <main className="flex-1 overflow-y-auto p-6">{children}</main>
      </div>
    </div>
  )
}
```

---

## Checklist الاكتمال

بعد بناء كل طبقة، تحقق:

```
الطبقة 1 — DB Foundation:
□ جدول tenants يعمل
□ جدول users يعمل
□ RLS مُفعَّل ويعزل البيانات
□ Migration نظيف وقابل للتراجع

الطبقة 2 — Auth System:
□ POST /auth/login يُرجع access + refresh token
□ GET /auth/me يُرجع بيانات المستخدم الصحيحة
□ POST /auth/refresh يُجدد الـ token
□ JWT يحتوي tenant_id و role

الطبقة 3 — Multi-Tenant Middleware:
□ طلب بدون token → 401
□ طلب بـ token صالح → tenant_id يُعين تلقائياً
□ مستخدم A لا يرى بيانات مستخدم B (اختبر يدوياً)

الطبقة 4 — Permission Guard:
□ viewer لا يستطيع POST
□ user لا يستطيع إدارة المستخدمين
□ tenant_admin لا يرى tenants أخرى
□ super_admin يرى كل شيء

الطبقة 5 — Base API:
□ /health يرد بـ 200
□ كل response بنفس الشكل { success, data, message }
□ الأخطاء تُرجع رسائل عربية واضحة
□ CORS يسمح للـ frontend

الطبقة 6 — Frontend Shell:
□ صفحة Login تعمل + تحفظ الـ token
□ Protected routes تُعيد توجيه غير المسجلين
□ Sidebar + Header يظهران بشكل صحيح
□ RTL صحيح في جميع الصفحات
□ API client يُرسل الـ token تلقائياً
```

---

## بعد اكتمال Core Engine

```
✅ Core Engine جاهز — الآن كل feature جديدة = 3 ملفات:

backend/models/[feature].py      ← تعريف الجدول
backend/routers/[feature].py     ← CRUD endpoints (5 endpoints)
frontend/app/[feature]/page.tsx  ← الصفحة الكاملة

لا تحتاج إعادة كتابة Auth أو Tenant أو Permissions في كل مرة.
```

**انتقل لـ PHASE 7B — BUILD LOOP (Features).**
