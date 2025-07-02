# 🔧 دليل استكشاف الأخطاء وإصلاحها

## مشاكل تسجيل الدخول

### 1. خطأ "Authentication error: API request failed after 3 attempts: Failed to fetch"

**السبب:** مشكلة في الاتصال بـ API أو مسارات غير صحيحة

**الحلول:**

#### أ) تأكد من تشغيل خادم Odoo
```bash
# تأكد من أن Odoo يعمل على المنفذ 8069
curl http://localhost:8069/api/v1/config
```

#### ب) تحقق من المسارات في API
- تأكد من أن ملف `custom_api_controller.py` مثبت في Odoo
- تحقق من أن المسارات تبدأ بـ `/api/v1/` وليس `/api/`

#### ج) اختبار الاتصال
افتح `test-login.html` في المتصفح واختبر الاتصال

### 2. خطأ "CORS" أو "Access-Control-Allow-Origin"

**السبب:** مشكلة في إعدادات CORS

**الحل:**
أضف هذا الكود إلى `custom_api_controller.py`:

```python
from odoo.http import request, Response

# في بداية الكلاس
def _cors_preflight(self):
    """Handle CORS preflight requests"""
    headers = {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type, Authorization, X-Session-ID',
        'Access-Control-Max-Age': '86400'
    }
    return Response(status=200, headers=headers)

@http.route('/api/v1/<path:path>', type='http', auth="public", methods=['OPTIONS'])
def handle_cors_preflight(self, path, **kwargs):
    return self._cors_preflight()
```

### 3. خطأ "Invalid credentials"

**السبب:** بيانات تسجيل الدخول غير صحيحة

**الحلول:**

#### أ) تحقق من بيانات المستخدم
```python
# في Odoo shell
user = env['res.users'].search([('login', '=', 'admin')])
print(user.name, user.login)
```

#### ب) إعادة تعيين كلمة المرور
```python
# في Odoo shell
user = env['res.users'].search([('login', '=', 'admin')])
user.password = 'admin123'
```

### 4. خطأ "Module not found"

**السبب:** الوحدات المطلوبة غير مثبتة

**الحل:**
تأكد من تثبيت الوحدات التالية:
- `website`
- `website_sale`
- `odoo_marketplace` (اختياري)
- `website_auction` (اختياري)

## مشاكل الاتصال

### 1. خطأ "Connection refused"

**الحلول:**
- تأكد من تشغيل خادم Odoo
- تحقق من المنفذ (افتراضي: 8069)
- تأكد من إعدادات الجدار الناري

### 2. خطأ "Timeout"

**الحلول:**
- زيادة مهلة الاتصال في `api.js`
- تحقق من سرعة الإنترنت
- تأكد من عدم وجود مشاكل في الخادم

## مشاكل البيانات

### 1. لا تظهر المنتجات

**الحلول:**
- تأكد من وجود منتجات في قاعدة البيانات
- تحقق من أن المنتجات منشورة على الموقع
- تأكد من صلاحيات الوصول

### 2. خطأ في البحث

**الحلول:**
- تحقق من دالة البحث في `custom_api_controller.py`
- تأكد من صحة معاملات البحث

## إعدادات التطوير

### 1. تفعيل وضع التطوير
```python
# في odoo.conf
debug = True
log_level = debug
```

### 2. مراقبة السجلات
```bash
# مراقبة سجلات Odoo
tail -f /var/log/odoo/odoo.log
```

### 3. اختبار API مباشرة
```bash
# اختبار نقطة نهاية التكوين
curl -X GET http://localhost:8069/api/v1/config

# اختبار تسجيل الدخول
curl -X POST http://localhost:8069/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "admin", "password": "admin"}'
```

## نصائح عامة

1. **استخدم أدوات المطور** في المتصفح لمراقبة طلبات الشبكة
2. **تحقق من وحدة التحكم** للبحث عن أخطاء JavaScript
3. **اختبر API مباشرة** قبل استخدامه في الواجهة الأمامية
4. **احتفظ بنسخة احتياطية** من قاعدة البيانات قبل التعديلات
5. **استخدم بيئة تطوير منفصلة** عن بيئة الإنتاج

## الحصول على المساعدة

إذا استمرت المشكلة:
1. تحقق من سجلات Odoo
2. اختبر API باستخدام Postman أو curl
3. تأكد من إعدادات الخادم
4. راجع وثائق Odoo الرسمية 