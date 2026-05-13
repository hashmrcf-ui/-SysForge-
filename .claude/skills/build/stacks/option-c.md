# الخيار C — منصة SaaS كاملة

## متى تختاره
- ستبيع النظام لعدة شركات
- كل شركة بحسابها وبياناتها المنفصلة
- تريد نظام اشتراكات ودفع
- هدفك منتج SaaS حقيقي

## مفهوم Multi-Tenant
كل شركة = "tenant" — لها بياناتها المعزولة تماماً داخل نفس النظام.

```
شركة A → tenant_id: 1 → بياناتها معزولة
شركة B → tenant_id: 2 → بياناتها معزولة
شركة C → tenant_id: 3 → بياناتها معزولة
```

## التقنيات

| الطبقة | التقنية | السبب |
|---|---|---|
| الواجهة | Next.js 14 + Subdomain routing | `company-a.myapp.com` |
| الـ UI | shadcn/ui + Tailwind | |
| الخادم | FastAPI | |
| قاعدة البيانات | PostgreSQL + Row-Level Security | عزل بيانات حقيقي |
| وكيل AI | LangChain + Multi-agent | وكيل منفصل لكل tenant |
| المصادقة | Clerk (يدعم multi-tenant) | الأسهل لـ SaaS |
| الاشتراكات | Stripe | معيار الصناعة |
| البريد | Resend | |
| النشر | Vercel + Railway + Supabase | |

## هيكل DB (Multi-Tenant)

```sql
-- كل جدول يحتوي tenant_id
tenants (id, name, slug, plan, stripe_customer_id, created_at)
users (id, tenant_id, name, email, role, created_at)

-- كل جدول آخر
invoices (id, tenant_id, ...) -- tenant_id في كل جدول
clients (id, tenant_id, ...)
products (id, tenant_id, ...)
```

## هيكل المشروع

```
saas-platform/
├── frontend/
│   ├── app/
│   │   ├── (marketing)/          ← الصفحة التسويقية
│   │   │   ├── page.tsx          ← الـ landing page
│   │   │   ├── pricing/
│   │   │   └── about/
│   │   ├── (auth)/               ← تسجيل دخول + إنشاء حساب
│   │   ├── (app)/                ← لوحة التحكم (بعد الدخول)
│   │   │   ├── layout.tsx
│   │   │   └── [module]/
│   │   └── api/
│   │       └── webhooks/stripe/  ← Stripe webhooks
│   └── middleware.ts             ← Tenant detection بالـ subdomain
│
├── backend/
│   ├── routers/
│   │   ├── tenants.py            ← إدارة الشركات
│   │   ├── auth.py
│   │   ├── [module].py
│   │   ├── ai.py
│   │   └── billing.py            ← Stripe integration
│   ├── middleware/
│   │   └── tenant.py             ← Tenant isolation middleware
│   └── ...
│
├── database/
│   └── rls_policies.sql          ← Row Level Security policies
│
└── docker-compose.yml
```

## خطوات إضافية (SaaS فقط)

```
1. إعداد Stripe + خطط الاشتراك
2. Subdomain routing في Next.js
3. Row Level Security في PostgreSQL
4. Tenant onboarding flow (تسجيل شركة جديدة)
5. لوحة تحكم Super Admin (أنت تدير كل الشركات)
6. إيميلات تلقائية (ترحيب، فاتورة، إلخ)
7. نظام الخطط (مجاني / احترافي / مؤسسي)
```

## متغيرات البيئة الإضافية

```env
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
RESEND_API_KEY=re_...
NEXT_PUBLIC_APP_URL=https://myapp.com
```

## عدد الجلسات المتوقعة: 15-20 جلسة

## تحذير مهم
Option C مناسب فقط إذا كان هدفك بيع النظام لعدة شركات.
إذا كانت الشركة واحدة — اختر A أو B. Option C أعقد بكثير مما تحتاج.
