# نظام مفاتيح API - Module API Data

## نظرة عامة

نظام مفاتيح API هو نظام متكامل لإدارة وتأمين الوصول إلى API في وحدة Module API Data. يوفر النظام إدارة شاملة لمفاتيح API مع ميزات أمان متقدمة.

## المميزات الرئيسية

### 🔐 أنواع مفاتيح API
- **مفتاح عام (Public)**: للوصول العام المحدود
- **مفتاح خاص (Private)**: للوصول الكامل
- **مفتاح مؤقت (Temporary)**: مع انتهاء صلاحية تلقائي
- **مفتاح إداري (Admin)**: للوصول الإداري الكامل

### 🛡️ ميزات الأمان
- **تجزئة آمنة**: تخزين مفاتيح API مشفرة
- **تحقق من IP**: تقييد الوصول لعناوين IP محددة
- **تحقق من النقاط النهائية**: تقييد الوصول لنقاط نهائية محددة
- **حد الاستخدام**: تحديد عدد مرات الاستخدام المسموحة
- **انتهاء صلاحية تلقائي**: إلغاء مفاتيح منتهية الصلاحية

### 📊 المراقبة والتتبع
- **تتبع الاستخدام**: تسجيل كل استخدام لمفتاح API
- **إحصائيات مفصلة**: إحصائيات شاملة لاستخدام المفاتيح
- **سجل الأمان**: تسجيل جميع محاولات الوصول
- **إشعارات تلقائية**: إشعارات عند انتهاء صلاحية المفاتيح

## كيفية الاستخدام

### إنشاء مفتاح API جديد

```python
from odoo import api, SUPERUSER_ID

env = api.Environment(cr, SUPERUSER_ID, {})

# إنشاء مفتاح API جديد
api_key = env['api.key'].create({
    'name': 'مفتاح API للتطبيق الخارجي',
    'user_id': user_id,
    'key_type': 'private',
    'permissions': 'write',
    'expires_at': '2024-12-31 23:59:59',
    'max_usage': 1000,
    'allowed_ips': '192.168.1.1\n10.0.0.1',
    'allowed_endpoints': '/api/v1/products\n/api/v1/sellers',
    'description': 'مفتاح API للتطبيق الخارجي'
})
```

### التحقق من صحة مفتاح API

```python
from utils.api_key_validator import ApiKeyValidator

validator = ApiKeyValidator(env)

# التحقق من صحة المفتاح
is_valid, message, api_key = validator.validate_api_key(
    api_key_value='your_api_key_here',
    ip_address='192.168.1.1',
    endpoint='/api/v1/products'
)

if is_valid:
    print("المفتاح صحيح")
else:
    print(f"خطأ: {message}")
```

### الحصول على صلاحيات مفتاح API

```python
permissions = validator.get_api_key_permissions('your_api_key_here')

if permissions:
    print(f"نوع المفتاح: {permissions['key_type']}")
    print(f"الصلاحيات: {permissions['permissions']}")
    print(f"النقاط النهائية المسموحة: {permissions['allowed_endpoints']}")
```

## إدارة مفاتيح API

### تفعيل مفتاح API
```python
api_key.action_activate()
```

### تعليق مفتاح API
```python
api_key.action_suspend()
```

### إلغاء مفتاح API
```python
api_key.action_revoke()
```

### إعادة توليد مفتاح API
```python
api_key.action_regenerate()
```

## المهام الدورية

### التحقق من مفاتيح API منتهية الصلاحية
النظام يحتوي على مهمة دورية تعمل يومياً للتحقق من مفاتيح API منتهية الصلاحية وتحديث حالتها تلقائياً.

```python
# المهمة الدورية
def _cron_check_expired_keys(self):
    expired_keys = self.search([
        ('state', '=', 'active'),
        ('expires_at', '!=', False),
        ('expires_at', '<', fields.Datetime.now())
    ])
    
    for key in expired_keys:
        key.write({'state': 'expired', 'is_active': False})
```

## واجهة المستخدم

### قائمة مفاتيح API
- عرض جميع مفاتيح API مع حالتها
- فلترة حسب النوع والحالة
- بحث متقدم
- تجميع حسب المستخدم والنوع

### نموذج مفتاح API
- عرض تفصيلي لمفتاح API
- إدارة الحالة (تفعيل/تعليق/إلغاء)
- عرض إحصائيات الاستخدام
- إدارة الصلاحيات والقيود

### ويزارد إلغاء مفتاح API
- واجهة سهلة لإلغاء مفاتيح API
- إدخال سبب الإلغاء
- خيار إشعار المستخدم

## الأمان

### تخزين آمن
- مفاتيح API مشفرة باستخدام SHA-256
- لا يتم عرض المفاتيح الأصلية في الواجهة
- تخزين آمن في قاعدة البيانات

### التحقق المتعدد المستويات
1. **التحقق من التجزئة**: مقارنة تجزئة المفتاح
2. **التحقق من الحالة**: التأكد من أن المفتاح نشط
3. **التحقق من انتهاء الصلاحية**: التأكد من عدم انتهاء الصلاحية
4. **التحقق من عدد مرات الاستخدام**: التأكد من عدم تجاوز الحد
5. **التحقق من عنوان IP**: التأكد من أن IP مسموح
6. **التحقق من النقطة النهائية**: التأكد من أن النقطة مسموحة

### تسجيل الأمان
- تسجيل جميع محاولات الوصول
- تسجيل الأخطاء والمحاولات الفاشلة
- تتبع استخدام كل مفتاح API

## الإحصائيات

### إحصائيات مفاتيح API
```python
stats = validator.get_api_key_stats(user_id=user_id)

print(f"إجمالي المفاتيح: {stats['total']}")
print(f"المفاتيح النشطة: {stats['active']}")
print(f"المفاتيح المعلقة: {stats['suspended']}")
print(f"المفاتيح منتهية الصلاحية: {stats['expired']}")
print(f"إجمالي الاستخدام: {stats['total_usage']}")
```

## التكامل مع API

### استخدام في Controllers
```python
from odoo import http
from odoo.http import request
from utils.api_key_validator import ApiKeyValidator

class ApiController(http.Controller):
    
    @http.route('/api/v1/products', type='http', auth='none', methods=['GET'])
    def get_products(self, **kwargs):
        # الحصول على مفتاح API من الطلب
        api_key = request.httprequest.headers.get('Authorization', '').replace('Bearer ', '')
        
        # التحقق من صحة المفتاح
        validator = ApiKeyValidator(request.env)
        is_valid, message, api_key_record = validator.validate_api_key(
            api_key, 
            request.httprequest.remote_addr,
            '/api/v1/products'
        )
        
        if not is_valid:
            return json.dumps({'error': message}), 401
        
        # تسجيل الاستخدام
        validator.log_api_usage(
            api_key, 
            '/api/v1/products', 
            request.httprequest.remote_addr,
            request.httprequest.headers.get('User-Agent'),
            success=True
        )
        
        # تنفيذ الطلب
        products = request.env['module_api_data.product'].search([])
        return json.dumps([{
            'id': p.id,
            'name': p.name,
            'code': p.code,
            'price': p.price
        } for p in products])
```

## الصلاحيات

### صلاحيات المستخدمين
- **المستخدم العادي**: إنشاء وإدارة مفاتيح API الخاصة به
- **مدير النظام**: إدارة جميع مفاتيح API وإنشاء مفاتيح إدارية

### أنواع الصلاحيات
- **قراءة فقط (read)**: الوصول للقراءة فقط
- **قراءة وكتابة (write)**: الوصول للقراءة والكتابة
- **إدارة كاملة (admin)**: الوصول الكامل لجميع الوظائف

## استكشاف الأخطاء

### مشاكل شائعة وحلولها

#### 1. مفتاح API غير صحيح
- **السبب**: المفتاح غير موجود أو غير نشط
- **الحل**: التحقق من صحة المفتاح وحالته

#### 2. عنوان IP غير مسموح
- **السبب**: عنوان IP المستخدم غير مدرج في القائمة المسموحة
- **الحل**: إضافة عنوان IP إلى قائمة العناوين المسموحة

#### 3. النقطة النهائية غير مسموحة
- **السبب**: النقطة النهائية المطلوبة غير مدرجة في القائمة المسموحة
- **الحل**: إضافة النقطة النهائية إلى قائمة النقاط المسموحة

#### 4. انتهاء صلاحية المفتاح
- **السبب**: المفتاح منتهي الصلاحية
- **الحل**: إنشاء مفتاح جديد أو تمديد صلاحية المفتاح الحالي

#### 5. تجاوز حد الاستخدام
- **السبب**: تم استنفاذ عدد مرات الاستخدام المسموحة
- **الحل**: زيادة حد الاستخدام أو إنشاء مفتاح جديد

## التطوير المستقبلي

### ميزات مقترحة
- **JWT Tokens**: دعم JWT tokens للتحقق من الهوية
- **Rate Limiting**: تحديد معدل الطلبات لكل مفتاح
- **Webhook Support**: دعم webhooks للإشعارات
- **API Analytics**: تحليلات متقدمة لاستخدام API
- **Multi-factor Authentication**: دعم المصادقة متعددة العوامل

### تحسينات الأمان
- **Encryption at Rest**: تشفير المفاتيح في قاعدة البيانات
- **Key Rotation**: دوران تلقائي للمفاتيح
- **Audit Trail**: سجل تدقيق مفصل
- **Threat Detection**: كشف التهديدات والهجمات

## الدعم والمساعدة

للمساعدة والدعم التقني، يرجى التواصل مع فريق التطوير أو مراجعة الوثائق التقنية.

---

**ملاحظة**: هذا النظام مصمم لتوفير أمان عالي ومرونة في إدارة مفاتيح API. يرجى اتباع أفضل الممارسات الأمنية عند استخدامه. 
