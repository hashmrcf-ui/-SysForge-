# بلوبرينت — نظام المبيعات ونقاط البيع (POS)

## نظرة عامة
نظام مبيعات متكامل يدير المنتجات، الطلبات، المخزون، ونقاط البيع — مع وكيل AI يحلل الأداء ويقترح التحسينات.

---

## الوحدات الأساسية

### 1. إدارة المنتجات والمخزون
**ما تحتويه:**
- كتالوج المنتجات (اسم، صورة، سعر، فئة)
- تتبع الكميات في المخزن
- تنبيهات نفاد المخزون

**جداول DB:**
```sql
categories (id, name, parent_id)
products (id, name, sku, barcode, category_id, price, cost, image_url)
inventory (id, product_id, quantity, min_quantity, location)
inventory_logs (id, product_id, change, reason, date)
```

---

### 2. نقطة البيع (POS)
**ما تحتويه:**
- واجهة سريعة للبيع (كليك + باركود)
- سلة المشتريات
- طرق الدفع (كاش، بطاقة، آجل)
- طباعة إيصال

**صفحات Next.js:**
- `/pos` — شاشة البيع الرئيسية (full screen)
- `/pos/receipt/[id]` — الإيصال للطباعة

---

### 3. الطلبات والفواتير
**ما تحتويه:**
- تاريخ جميع الطلبات
- حالات الطلب: pending / confirmed / shipped / delivered / returned
- فاتورة PDF لكل طلب

**جداول DB:**
```sql
orders (id, client_id, date, status, total, discount, tax, payment_method)
order_items (id, order_id, product_id, qty, unit_price, total)
```

---

### 4. إدارة العملاء والولاء
**ما تحتويه:**
- قاعدة بيانات العملاء
- تاريخ المشتريات
- نقاط الولاء (اختياري)

---

### 5. التقارير والتحليلات
- أكثر المنتجات مبيعاً
- تقرير المبيعات اليومي / الأسبوعي / الشهري
- هامش الربح لكل منتج
- أداء المبيعات مقارنة الأهداف

---

### 6. وكيل AI — مدير المبيعات الذكي

**مهامه:**
- "ما المنتجات التي ستنفد خلال 7 أيام؟"
- تحليل ساعات الذروة واقتراح توزيع الموظفين
- اكتشاف المنتجات الراكدة + اقتراح خصومات
- توقع مبيعات الشهر القادم
- مقارنة أداء الفروع (إذا متعدد)

---

## هيكل المشروع

```
sales-system/
├── frontend/
│   ├── app/
│   │   ├── dashboard/
│   │   ├── products/
│   │   ├── pos/               ← واجهة POS
│   │   ├── orders/
│   │   ├── clients/
│   │   ├── inventory/
│   │   ├── reports/
│   │   └── ai-advisor/
│   └── components/
│       ├── POSCart.tsx
│       ├── BarcodeScanner.tsx
│       └── SalesChart.tsx
│
├── backend/
│   ├── routers/
│   │   ├── products.py
│   │   ├── orders.py
│   │   ├── inventory.py
│   │   └── ai.py
│   └── main.py
│
└── database/
```

---

## ترتيب البناء

```
1. هيكل + Docker + DB
2. المنتجات والفئات
3. واجهة POS
4. الطلبات والفواتير
5. المخزون والتنبيهات
6. العملاء
7. وكيل AI
8. التقارير
9. المصادقة + الأدوار (كاشير / مدير)
```
