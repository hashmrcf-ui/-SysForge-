# بلوبرينت — نظام الموارد البشرية والحضور

## نظرة عامة
نظام HR متكامل يدير الموظفين، الحضور، الرواتب، والإجازات — مع وكيل AI يحلل الأداء والغياب.

---

## الوحدات الأساسية

### 1. إدارة الموظفين
**ما تحتويه:**
- ملف الموظف الكامل
- الهيكل التنظيمي (قسم + مدير مباشر)
- المستندات (عقد، هوية، شهادات)

**جداول DB:**
```sql
departments (id, name, manager_id)
employees (id, number, name, department_id, position, hire_date, salary, status)
documents (id, employee_id, type, file_url, expiry_date)
```

---

### 2. الحضور والانصراف
**ما تحتويه:**
- تسجيل دخول / خروج يدوي أو بالباركود
- حساب ساعات العمل تلقائياً
- تقرير الغياب والتأخر

**جداول DB:**
```sql
attendance (id, employee_id, date, check_in, check_out, hours, status)
```

**صفحات Next.js:**
- `/attendance` — سجل الحضور اليومي
- `/attendance/report` — تقارير الشهر

---

### 3. الإجازات والطلبات
**ما تحتويه:**
- أنواع الإجازات (سنوية، مرضية، طارئة)
- رصيد الإجازات
- مسار الموافقة

**جداول DB:**
```sql
leave_types (id, name, days_per_year, paid)
leave_balances (id, employee_id, leave_type_id, year, used, remaining)
leave_requests (id, employee_id, type_id, from_date, to_date, reason, status)
```

---

### 4. الرواتب والمستحقات
**ما تحتويه:**
- حساب الراتب الشهري
- الخصومات والإضافات
- قسيمة الراتب PDF

**جداول DB:**
```sql
salary_components (id, name, type, calculation_method)
payroll (id, employee_id, month, year, basic, additions, deductions, net)
```

---

### 5. التقييم والأداء
**ما تحتويه:**
- أهداف الموظف الربعية
- تقييم المدير
- تقرير الأداء السنوي

---

### 6. وكيل AI — مدير HR الذكي

**مهامه:**
- "من الموظفون الذين تنتهي عقودهم هذا الشهر؟"
- تحليل نمط الغياب وتنبيه المدير
- توقع احتياجات التوظيف بناءً على النمو
- مقارنة الرواتب بمعدلات السوق
- "ما متوسط أداء قسم المبيعات هذا الربع؟"

---

## ترتيب البناء

```
1. هيكل + DB
2. الموظفون والأقسام
3. الحضور والانصراف
4. الإجازات والطلبات
5. الرواتب
6. التقييم
7. وكيل AI
8. صلاحيات متعددة (موظف / مدير / HR / رئيس)
```
