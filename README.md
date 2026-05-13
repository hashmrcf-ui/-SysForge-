# 🏗️ SysForge — هندسة الأنظمة

> **Systems Engineering for Claude Code**  
> من الفكرة إلى التسليم — منظومة بناء SaaS احترافية للمتدربين

---

## ما هو SysForge؟

**SysForge** هي منظومة مهارات متكاملة لـ [Claude Code](https://claude.ai/code) تُمكّن أي متدرب من بناء أنظمة أعمال SaaS كاملة — محاسبة، مستشفيات، مبيعات، موارد بشرية — مدمجة مع وكيل ذكاء اصطناعي.

```
/sysforge نظام محاسبة  ←  يبدأ رحلة البناء الكاملة تلقائياً
```

---

## الأوامر الأربعة

| الأمر | الدور | متى تستخدمه |
|---|---|---|
| `/sysforge` | القائد الرئيسي — 10 مراحل | البداية |
| `/sysarch` | معمارية الأنظمة — Domain Model, Auth, API | PHASE 4.5 |
| `/syscore` | أساس الأنظمة — DB+RLS, Auth, Middleware | PHASE 7A |
| `/sysweb` | بنّاء الويب — Next.js + FastAPI + AI | PHASE 7B |

---

## رحلة البناء — 10 مراحل

```
PHASE 0   IDENTIFY    اختيار نوع النظام
PHASE 1   ALIGN       5 أسئلة لتوضيح النية
PHASE 2   COORDINATE  إنشاء ذاكرة المشروع (context.md)
PHASE 3   SCOUT       أفضل 3 مصادر مفتوحة للنظام
PHASE 4   BURST       اختيار معمارية A/B/C

PHASE 4.5 ARCHITECT ← /sysarch
          ├── Domain Model (الكيانات والعلاقات)
          ├── Multi-tenant Strategy (RLS/Schema/DB)
          ├── Auth + Permission Flow (JWT + 4 أدوار)
          ├── AI Agent Architecture (النوع + الأدوات)
          └── API Contract (كل الـ endpoints)

PHASE 5   DEVIL       هجوم على الخطة والمعمارية
PHASE 6   PROTOTYPE   5 شاشات HTML + موافقة التصميم

PHASE 7A  CORE ENGINE ← /syscore
          ├── Layer 1: DB Foundation (tenants + users + RLS)
          ├── Layer 2: Auth System (JWT endpoints)
          ├── Layer 3: Multi-tenant Middleware
          ├── Layer 4: Permission Guard
          ├── Layer 5: Base API Structure
          └── Layer 6: Frontend Shell

PHASE 7B  FEATURES    ← /sysweb
          وحدات النظام فوق الأساس (3 ملفات لكل feature)

PHASE 8   VERIFY      devil على الكود الحقيقي
PHASE 9   SHIP        نشر + calibrate
```

---

## الأنظمة المدعومة

| النظام | الوحدات الجاهزة |
|---|---|
| 📊 محاسبة | فواتير، عملاء، مصروفات، أستاذ عام، تقارير |
| 🏥 مستشفى | مرضى، أطباء، مواعيد، سجلات طبية |
| 🛒 مبيعات + POS | منتجات، طلبات، مخزون، نقطة بيع |
| 👥 موارد بشرية | موظفون، حضور، إجازات، رواتب |

---

## التثبيت

### عالمي (يعمل في كل مشروع)

```bash
# استنسخ المستودع
git clone https://github.com/hashmrcf-ui/-SysForge-.git

# انسخ المهارات للمجلد العالمي
cp -r ./.claude/skills/* ~/.claude/skills/
```

**Windows:**
```powershell
git clone https://github.com/hashmrcf-ui/-SysForge-.git
Copy-Item -Recurse .\.claude\skills\* "$env:USERPROFILE\.claude\skills\"
```

### على مستوى المشروع

```bash
# انسخ مجلد .claude إلى جذر مشروعك
cp -r ./.claude ./your-project/
```

---

## الاستخدام

```bash
# في Claude Code — ابدأ مشروع جديد
/sysforge نظام محاسبة

# أو معمارية مباشرة
/sysarch

# أو بناء الأساس مباشرة
/syscore

# أو بناء الويب مباشرة
/sysweb
```

---

## هيكل المستودع

```
SysForge/
├── .claude/
│   └── skills/
│       ├── build/              ← /sysforge (القائد الرئيسي)
│       │   ├── SKILL.md
│       │   ├── systems/        ← blueprints للأنظمة
│       │   │   ├── accounting.md
│       │   │   ├── hospital.md
│       │   │   ├── sales.md
│       │   │   └── hr.md
│       │   ├── stacks/         ← خيارات المعمارية
│       │   │   ├── option-a.md
│       │   │   ├── option-b.md
│       │   │   └── option-c.md
│       │   ├── sources/        ← مصادر مفتوحة
│       │   │   └── open-source.md
│       │   └── templates/      ← قوالب الملفات
│       │       ├── context.md
│       │       ├── session-log.md
│       │       └── architecture.md
│       ├── saas-architect/     ← /sysarch
│       │   └── SKILL.md
│       ├── saas-core/          ← /syscore
│       │   └── SKILL.md
│       └── next-skill/         ← /sysweb
│           └── SKILL.md
└── README.md
```

---

## المتطلبات

- [Claude Code](https://claude.ai/code) مثبّت
- Node.js 18+
- Python 3.11+
- PostgreSQL (أو Supabase للنشر السريع)

---

## المبدأ الأساسي

```
Architecture → Core Engine → Features
     ↓               ↓           ↓
  ما نبني       الأساس      فوق الأساس
```

**بعد اكتمال SysCore — كل feature جديدة = 3 ملفات فقط.**

---

## الترخيص

MIT License — استخدم، عدّل، وشارك.

---

<div align="center">

**🏗️ SysForge — ابنِ أنظمة تدوم**

</div>
