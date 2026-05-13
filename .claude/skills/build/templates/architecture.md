# معمارية النظام — [اسم المشروع]

**تاريخ التصميم:** [DATE]
**آخر تحديث:** [DATE]
**المصمم:** Claude + [اسمك]
**الحالة:** مسودة / معتمدة ✅

---

## 1. نموذج المجال (Domain Model)

### الكيانات الأساسية

| الكيان | الوصف | العلاقات |
|---|---|---|
| Tenant | شركة تستخدم النظام | يملك كل شيء |
| User | مستخدم في شركة | ينتمي لـ Tenant |
| [كيان 3] | [وصف] | [علاقة] |
| [كيان 4] | [وصف] | [علاقة] |

### رسم العلاقات

```
Tenant (1) ──── (∞) User
Tenant (1) ──── (∞) [كيان رئيسي]
User    (1) ──── (∞) [إجراءات المستخدم]
```

### الحقول الإلزامية في كل جدول

```sql
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
tenant_id   UUID NOT NULL REFERENCES tenants(id)
created_at  TIMESTAMP DEFAULT NOW()
updated_at  TIMESTAMP DEFAULT NOW()
```

---

## 2. استراتيجية Multi-Tenant

**الخيار المختار:** [RLS / Schema per Tenant / DB per Tenant]

**السبب:** [لماذا هذا الخيار]

### تطبيق RLS (إذا كان الخيار A)

```sql
-- سياسة العزل الأساسية
ALTER TABLE [table_name] ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON [table_name]
  USING (tenant_id = current_setting('app.tenant_id')::UUID);
```

### كيف يعمل في الكود

```python
# FastAPI — إعداد tenant_id تلقائياً من JWT
async def get_db(token: str = Depends(get_token)):
    tenant_id = decode_jwt(token)["tenant_id"]
    await db.execute(f"SET app.tenant_id = '{tenant_id}'")
    return db
```

---

## 3. نموذج الصلاحيات

### المستويات

| المستوى | الدور | ما يملكه |
|---|---|---|
| 1 | super_admin | كل شيء — يدير كل الشركات |
| 2 | tenant_admin | شركته فقط — يدير المستخدمين |
| 3 | user | ما خُصص له — لا يدير |
| 4 | viewer | قراءة فقط |

### جدول الصلاحيات

| الإجراء | super_admin | tenant_admin | user | viewer |
|---|---|---|---|---|
| إنشاء tenant | ✅ | ❌ | ❌ | ❌ |
| إدارة users | ✅ | ✅ | ❌ | ❌ |
| [إجراء النظام] | ✅ | ✅ | ✅ | ❌ |
| عرض البيانات | ✅ | ✅ | ✅ | ✅ |

---

## 4. Authentication Flow

### تسلسل الدخول

```
1. المستخدم → POST /api/auth/login { email, password }
2. Backend → يتحقق من البيانات
3. Backend → يُولّد JWT { user_id, tenant_id, role, exp: 24h }
4. Backend → يُرجع { access_token, refresh_token }
5. Frontend → يحفظ في localStorage / httpOnly cookie
6. كل request → Authorization: Bearer {token}
7. Backend → يتحقق من التوقيع + يستخرج tenant_id
8. Backend → يُعين tenant_id في DB session
```

### JWT Structure

```json
{
  "sub": "user_id",
  "tenant_id": "tenant_uuid",
  "role": "tenant_admin",
  "iat": 1234567890,
  "exp": 1234654290
}
```

### Refresh Token Strategy

```
access_token:  مدته 24 ساعة
refresh_token: مدته 30 يوم
عند انتهاء access_token → POST /api/auth/refresh
```

---

## 5. معمارية وكيل AI

### النوع

**[مساعد / محلل / وكيل كامل]**

### الأدوات (Tools)

```python
tools = [
    {
        "name": "get_summary",
        "description": "يُلخص بيانات الفترة المحددة",
        "parameters": { "period": "month|quarter|year" }
    },
    {
        "name": "detect_anomaly",
        "description": "يكتشف الشذوذات في البيانات",
        "parameters": { "threshold": "float" }
    },
    # أضف أدوات أخرى هنا
]
```

### تدفق الوكيل

```
المستخدم ← يسأل سؤالاً
    ↓
Next.js ← POST /api/ai/chat { messages, context }
    ↓
FastAPI ← يُحضّر بيانات tenant من DB
    ↓
Claude API ← system_prompt + tools + data
    ↓
Claude ← يختار tool أو يُجيب مباشرة
    ↓
FastAPI ← ينفذ tool + يُرجع النتيجة
    ↓
المستخدم ← يرى الإجابة (streaming)
```

### System Prompt الأساسي

```
أنت مساعد ذكي لنظام [اسم النظام].
مهمتك: [وصف المهمة]
قواعدك:
- تعمل فقط على بيانات الشركة الحالية
- لا تُعدّل بيانات إلا بتأكيد صريح
- أجب دائماً بالعربية
- إذا لم تكن متأكداً، اطلب توضيحاً
```

---

## 6. API Contract

### Base URL
```
Development:  http://localhost:8000/api/v1
Production:   https://api.[domain].com/v1
```

### Authentication Header
```
Authorization: Bearer {jwt_token}
Content-Type:  application/json
```

### Response Format (ثابت لكل endpoint)

```json
{
  "success": true,
  "data": { ... },
  "message": "عملية ناجحة",
  "errors": null,
  "meta": {
    "page": 1,
    "total": 100,
    "per_page": 20
  }
}
```

### الـ Endpoints الرئيسية

```
Auth:
POST   /auth/login
POST   /auth/logout
POST   /auth/refresh
GET    /auth/me

Users:
GET    /users
POST   /users
GET    /users/{id}
PUT    /users/{id}
DELETE /users/{id}

[كيانات النظام]:
GET    /{entity}
POST   /{entity}
GET    /{entity}/{id}
PUT    /{entity}/{id}
DELETE /{entity}/{id}

AI:
POST   /ai/chat
POST   /ai/analyze
GET    /ai/alerts
```

---

## 7. State & Events

**استراتيجية التواصل بين الأجزاء:**

```
[الخيار المختار: HTTP Polling / WebSocket / Queue]

السبب: [لماذا]

التطبيق:
- التحديثات العادية: HTTP polling كل [X] ثوانٍ
- التنبيهات الفورية: [WebSocket / Push Notification]
- المهام الثقيلة (تقارير كبيرة): [Background Job]
```

---

## 8. نقاط الحذر (Scalability)

| الخطر | المكان | الحل |
|---|---|---|
| N+1 Query | كل endpoint يجلب بيانات مرتبطة | JOIN أو eager loading |
| AI Latency | واجهة المستخدم تتجمد | Streaming response |
| File Storage | رفع الملفات للـ DB | S3 / Cloudflare R2 |
| Heavy Reports | طلبات التقارير الكبيرة | Background generation |
| Missing tenant_id | أي index بدون tenant_id | Composite index إلزامي |

---

## 9. القرارات المؤجلة

| القرار | السبب | متى نقرره |
|---|---|---|
| [قرار 1] | [لماذا مؤجل] | [متى] |
| [قرار 2] | [لماذا مؤجل] | [متى] |

---

## الموافقة

- [ ] Domain Model معتمد
- [ ] Multi-tenant strategy محددة
- [ ] Auth flow واضح
- [ ] AI Agent دوره محدد
- [ ] API contract مكتوب
- [ ] Scalability risks معروفة

**تاريخ الاعتماد:** [DATE]
