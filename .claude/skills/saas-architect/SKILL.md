---
name: sysarch
description: SysForge Architect — تصميم معمارية الأنظمة. يرسم Domain Model, Multi-tenant, Auth, Permissions, AI Agent, API Contract قبل أي كود. Outputs architecture.md. Triggers on "/sysarch", "/saas-architect", "design architecture", "صمم المعمارية", called by /sysforge in PHASE 4.5.
---

# SysArch — معمارية الأنظمة

## الهدف

تصميم **كامل** للمعمارية الأساسية قبل كتابة حرف واحد من الكود.
النتيجة: ملف `architecture.md` مكتمل + موافقة المتدرب.

**قاعدة:** لا كود حتى تُوقَّع المعمارية.

---

## متى تُستخدم

- تلقائياً من `/build` في PHASE 4.5 (بعد اختيار A/B/C)
- يدوياً: `/saas-architect` على أي مشروع قائم

---

## الخطوات

### الخطوة 1 — Domain Model (نموذج المجال)

**اقرأ أولاً:** `context.md` للمشروع لتعرف نوع النظام والمستخدمين.

**ثم حدد الكيانات الأساسية بناءً على نوع النظام:**

```
للمحاسبة:    Tenant, User, Client, Invoice, Payment, Account, JournalEntry
للمستشفى:    Tenant, User, Patient, Doctor, Appointment, MedicalRecord, Prescription
للمبيعات:    Tenant, User, Customer, Product, Order, Payment, Inventory
للـ HR:      Tenant, User, Employee, Department, Attendance, Leave, Payroll
```

**ارسم العلاقات بشكل نصي:**

```
Tenant (1) ──── (∞) User
Tenant (1) ──── (∞) [كيانات النظام]
User    (1) ──── (∞) [الإجراءات]
```

**اسأل:** "هل هذه الكيانات تغطي احتياجاتك؟ ما الذي ينقص؟"

---

### الخطوة 2 — Multi-Tenant Strategy

**عرض الخيارات الثلاثة وانتظر الاختيار:**

```
كيف نعزل بيانات كل شركة؟

A) Row-Level Security (RLS) ← الأبسط والأسرع
   كل سجل يحتوي tenant_id
   PostgreSQL يمنع تلقائياً الوصول بين الشركات
   المناسب: Option A و B من /build
   
B) Schema per Tenant ← عزل أعلى
   كل شركة لها schema منفصلة في نفس DB
   أعقد في الإدارة لكن عزل أفضل
   المناسب: Option C (SaaS كبير)

C) Database per Tenant ← العزل الكامل
   كل شركة DB مستقلة
   الأغلى والأصعب
   المناسب: متطلبات امتثال عالية (بنوك، مستشفيات كبيرة)
```

**التوصية الافتراضية:** A (RLS) لكل المستويات — إلا إذا كان هناك سبب قانوني واضح.

---

### الخطوة 3 — Authentication & Permission Model

**حدد مستويات الوصول:**

```
المستوى 1: Super Admin (أنت — صاحب المنصة)
   → يرى كل شيء، يدير كل الشركات

المستوى 2: Tenant Admin (مدير الشركة)
   → يرى بيانات شركته فقط
   → يضيف/يحذف مستخدمين في شركته

المستوى 3: User (موظف)
   → يرى ما خُصص له فقط
   → لا يدير إعدادات الشركة

المستوى 4: Viewer (مشاهد) ← اختياري
   → قراءة فقط، لا تعديل
```

**رسم Authentication Flow:**

```
المستخدم يفتح التطبيق
    ↓
يُدخل email + password
    ↓
Backend يتحقق + يُولّد JWT Token
    ↓
Token يحتوي: { user_id, tenant_id, role }
    ↓
كل request يرسل Token في Header
    ↓
Backend يستخرج tenant_id ويُفلتر البيانات تلقائياً
```

**اسأل:** "هل تحتاج Social Login (Google/Microsoft)؟ هل تحتاج 2FA؟"

---

### الخطوة 4 — AI Agent Architecture

**حدد دور وأدوات الوكيل بناءً على PHASE 1:**

```
نوع الوكيل: [من إجابة Phase 1 سؤال 2]

إذا كان مساعداً:
   tools: [get_summary, answer_question, generate_report]
   context: يقرأ بيانات tenant الحالي فقط

إذا كان محللاً:
   tools: [analyze_data, detect_anomaly, forecast, alert]
   triggers: يعمل تلقائياً على جدول زمني أو عند حدث

إذا كان وكيلاً كاملاً:
   tools: [read_data, write_data, send_notification, make_decision]
   ⚠️ يحتاج human-in-the-loop للقرارات الحساسة
```

**رسم تكامل الـ AI:**

```
المستخدم يسأل سؤالاً
    ↓
Next.js يرسل للـ AI endpoint في FastAPI
    ↓
FastAPI يحضّر context من DB (بيانات tenant فقط)
    ↓
Claude/GPT يُحلل + يُجيب
    ↓
الإجابة ترجع للمستخدم
    ↓
(اختياري) تُحفظ في سجل المحادثة
```

---

### الخطوة 5 — API Contract

**حدد الـ endpoints الرئيسية:**

```
Authentication:
POST   /api/auth/login
POST   /api/auth/logout
POST   /api/auth/refresh
GET    /api/auth/me

Tenant Management:
GET    /api/tenants                    [Super Admin فقط]
POST   /api/tenants                    [Super Admin فقط]
GET    /api/tenants/{id}

Users:
GET    /api/users                      [Tenant Admin]
POST   /api/users
PUT    /api/users/{id}
DELETE /api/users/{id}

[كيانات النظام — حسب النوع]:
GET    /api/{entity}                   [list + filter]
POST   /api/{entity}                   [create]
GET    /api/{entity}/{id}              [detail]
PUT    /api/{entity}/{id}              [update]
DELETE /api/{entity}/{id}              [delete]

AI Agent:
POST   /api/ai/chat                    [محادثة]
POST   /api/ai/analyze                 [تحليل بيانات]
GET    /api/ai/alerts                  [التنبيهات التلقائية]

Webhooks (SaaS فقط):
POST   /api/webhooks/stripe            [أحداث الدفع]
```

**قاعدة لكل endpoint:**
- يحتوي `tenant_id` من الـ JWT تلقائياً — لا يُرسل من المستخدم
- يُرجع خطأ 403 إذا حاول مستخدم الوصول لبيانات tenant آخر

---

### الخطوة 6 — State & Events

```
كيف تتواصل الأجزاء؟

للإصدار A/B (بسيط):
   → HTTP requests مباشرة
   → Polling كل X ثوانٍ للتحديثات

للإصدار C (SaaS كامل):
   → Webhooks للأحداث المهمة
   → WebSocket للتحديثات الفورية (اختياري)
   → Queue (Redis/Celery) للمهام الثقيلة
```

---

### الخطوة 7 — Scalability Checkpoints

**حدد نقاط الضعف قبل البناء:**

```
⚠️ نقاط تحتاج انتباهاً:

1. [N+1 Query] — كل استدعاء API يجب أن يكون query واحدة
2. [AI Latency] — الـ AI بطيء، استخدم streaming للمستخدم
3. [File Upload] — الصور والمرفقات تخزن في S3 مش DB
4. [Reports] — التقارير الثقيلة تُولَّد في background
5. [Multi-tenant] — كل index في DB يجب يحتوي tenant_id
```

---

### الخطوة 8 — إنشاء architecture.md

**بعد الموافقة على كل القرارات، أنشئ الملف:**

```markdown
# معمارية النظام — [اسم المشروع]

**تاريخ التصميم:** [DATE]
**آخر تحديث:** [DATE]

## نموذج المجال (Domain Model)
[الكيانات والعلاقات]

## استراتيجية Multi-Tenant
[الخيار المختار + السبب]

## نموذج الصلاحيات
[المستويات الأربعة + ما يملكه كل مستوى]

## Authentication Flow
[رسم التسلسل]

## معمارية وكيل AI
[النوع + الأدوات + التكامل]

## API Contract الأساسي
[الـ endpoints الرئيسية]

## State & Events
[كيف تتواصل الأجزاء]

## نقاط الحذر (Scalability)
[القائمة]

## القرارات المؤجلة
[ما لم نقرره بعد + لماذا]
```

احفظه في: `{project-folder}/architecture.md`
أضف إشارة له في `context.md`.

---

### الخطوة 9 — الموافقة النهائية

```
✅ معمارية [اسم النظام] جاهزة للمراجعة.

Domain Model:       [X] كيان
Multi-Tenant:       [الخيار المختار]
Auth:               [JWT / Session]
AI Agent:           [النوع]
API endpoints:      [العدد] endpoint

هل تعتمد هذه المعمارية؟
نعم → ننتقل لـ PHASE 5 (devil على المعمارية)
تعديل → أخبرني ما تريد تغييره
```

**لا تنتقل لـ PHASE 5 إلا بعد "نعم" صريح.**

---

## قواعد لا تُكسر

1. **لا تكتب كوداً في هذه المرحلة** — فقط تصميم وقرارات
2. **لا تفترض tenant strategy** — اسأل دائماً
3. **لا تنسَ AI Agent** — حتى لو بسيط، حدد تكامله
4. **لا تتجاوز الموافقة** — architecture.md يُنشأ قبل Phase 5
5. **وثّق كل قرار + سببه** — المتدرب سيحتاجه لاحقاً

---

## مثال سريع (نظام محاسبة — Option B)

```
Domain: Tenant, User, Client, Invoice, InvoiceItem, Payment, Account, JournalEntry
Multi-Tenant: RLS (Row-Level Security) — tenant_id في كل جدول
Auth: JWT Token { user_id, tenant_id, role } — انتهاء الصلاحية 24 ساعة
Roles: super_admin, tenant_admin, accountant, viewer
AI: محلل — يقرأ البيانات، ينبه على الشذوذات، يولّد تقارير
API: 35 endpoint — REST — /api/v1/{resource}
Events: HTTP polling كل دقيقة للتنبيهات — WebSocket لاحقاً
```
