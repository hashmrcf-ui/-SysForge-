# بلوبرينت — نظام المحاسبة والفواتير

## نظرة عامة
نظام محاسبة متكامل مع وكيل ذكاء اصطناعي يحلل البيانات المالية، ينبّه على الشذوذات، ويولّد التقارير تلقائياً.

---

## الوحدات الأساسية (بالترتيب)

### 1. إدارة العملاء والموردين
**ما تحتويه:**
- بيانات العملاء (اسم، رقم ضريبي، عنوان، بريد)
- بيانات الموردين
- تاريخ المعاملات لكل طرف

**جداول DB:**
```sql
clients (id, name, tax_number, email, phone, address, created_at)
suppliers (id, name, tax_number, email, phone, address, created_at)
```

**صفحات Next.js:**
- `/clients` — قائمة العملاء + بحث + تصفية
- `/clients/[id]` — ملف العميل + تاريخ الفواتير
- `/suppliers` — نفس البنية

---

### 2. الفواتير والمدفوعات
**ما تحتويه:**
- إنشاء فاتورة بنود متعددة
- حالات الفاتورة: مسودة / مُرسلة / مدفوعة / متأخرة
- تتبع المدفوعات الجزئية

**جداول DB:**
```sql
invoices (id, client_id, number, date, due_date, status, total, tax, notes)
invoice_items (id, invoice_id, description, qty, unit_price, total)
payments (id, invoice_id, amount, date, method, reference)
```

**صفحات Next.js:**
- `/invoices` — قائمة الفواتير مع فلتر الحالة
- `/invoices/new` — إنشاء فاتورة جديدة
- `/invoices/[id]` — عرض + طباعة + تسجيل دفعة

---

### 3. المصروفات والمشتريات
**ما تحتويه:**
- تسجيل المصروفات اليومية
- ربط بالموردين
- تصنيف المصروفات (إيجار، رواتب، مواد)

**جداول DB:**
```sql
expense_categories (id, name, code)
expenses (id, supplier_id, category_id, amount, date, description, receipt_url)
```

---

### 4. الحسابات والأستاذ العام
**ما تحتويه:**
- شجرة الحسابات (Chart of Accounts)
- قيود يومية
- ميزان المراجعة

**جداول DB:**
```sql
accounts (id, code, name, type, parent_id)
journal_entries (id, date, description, reference)
journal_lines (id, entry_id, account_id, debit, credit)
```

---

### 5. التقارير المالية
**ما تحتويه:**
- قائمة الدخل (شهري / ربعي / سنوي)
- الميزانية العمومية
- تقرير التدفق النقدي
- تقرير الضريبة

**صفحات Next.js:**
- `/reports/income` — قائمة الدخل
- `/reports/balance` — الميزانية
- `/reports/tax` — التقرير الضريبي

---

### 6. وكيل الذكاء الاصطناعي — المحاسب الذكي

**الاسم المقترح:** "أمين" أو "CFO AI"

**مهامه:**
- تحليل الإيرادات vs المصروفات وتنبيه الانحرافات
- اكتشاف الفواتير المتأخرة وإرسال تذكيرات
- توقع التدفق النقدي للشهر القادم
- الإجابة على أسئلة مثل: "كم ربحنا هذا الربع؟"
- اقتراح تصنيف المصروف تلقائياً

**FastAPI endpoints:**
```
POST /ai/analyze-month      → تحليل شهر محدد
POST /ai/forecast-cashflow  → توقع 30 يوم قادم
POST /ai/chat               → محادثة حرة مع المحاسب
GET  /ai/alerts             → التنبيهات التلقائية
```

---

## هيكل المشروع المقترح (Option B)

```
accounting-system/
├── frontend/                    ← Next.js
│   ├── app/
│   │   ├── (auth)/login/
│   │   ├── dashboard/           ← ملخص سريع
│   │   ├── clients/
│   │   ├── invoices/
│   │   ├── expenses/
│   │   ├── reports/
│   │   └── ai-assistant/        ← واجهة المحاسب الذكي
│   ├── components/
│   │   ├── InvoiceTable.tsx
│   │   ├── ReportChart.tsx
│   │   └── AIChat.tsx
│   └── lib/
│       └── api.ts               ← استدعاءات FastAPI
│
├── backend/                     ← FastAPI
│   ├── routers/
│   │   ├── clients.py
│   │   ├── invoices.py
│   │   ├── expenses.py
│   │   └── ai.py
│   ├── models/                  ← SQLAlchemy models
│   ├── schemas/                 ← Pydantic schemas
│   └── main.py
│
├── database/
│   └── migrations/              ← Alembic migrations
│
└── docker-compose.yml           ← تشغيل محلي بأمر واحد
```

---

## ترتيب البناء المثالي

```
1. هيكل المشروع + Docker
2. قاعدة البيانات (جداول + migrations)
3. FastAPI: clients + suppliers
4. Next.js: صفحات العملاء
5. FastAPI: invoices + payments
6. Next.js: صفحات الفواتير + الطباعة
7. FastAPI: expenses
8. Next.js: صفحات المصروفات
9. FastAPI: AI endpoints
10. Next.js: واجهة المحاسب الذكي
11. التقارير
12. المصادقة + الصلاحيات
```

---

## التكاملات المحتملة
- **PDF:** خدمة طباعة الفواتير (WeasyPrint أو Puppeteer)
- **البريد:** إرسال الفواتير بالبريد (SendGrid / Resend)
- **الضريبة:** ربط بنظام ضريبة القيمة المضافة المحلي
