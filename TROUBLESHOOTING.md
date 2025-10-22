# 🔧 دليل استكشاف الأخطاء وإصلاحها - ChatSphere

## 🚨 مشكلة: لا يمكن استلام الرسائل

### الخطوة 1: استخدام أداة التشخيص

1. **ادخل إلى التطبيق** كـ admin
2. **اذهب إلى قائمة المستخدم** (أيقونة المستخدم في الأعلى)
3. **اختر "Webhook Diagnostics"**
4. **اختر الـ instance** المطلوب من القائمة المنسدلة

### الخطوة 2: فحص حالة النظام

في صفحة التشخيص، تحقق من:

#### ✅ **Instance Status**
- **Configured**: يجب أن يكون أخضر
- **Active**: يجب أن يكون أخضر  
- **Has App Secret**: يجب أن يكون أخضر (إذا كنت تستخدمه)
- **Has Verify Token**: يجب أن يكون أخضر

#### ⚠️ **إذا كان أي مؤشر أحمر:**
- اذهب إلى **Settings** وأصلح الإعدادات
- تأكد من أن الـ instance نشط
- تحقق من الـ access token و phone number ID

### الخطوة 3: اختبار إرسال رسالة

1. **اذهب إلى تبويب "Test Message"**
2. **أدخل رقم الهاتف** (مع رمز الدولة)
3. **أدخل رسالة تجريبية**
4. **اضغط "Send Test Message"**

#### ✅ **إذا نجح الاختبار:**
- المشكلة في استلام الرسائل، وليس في الإرسال
- تحقق من إعدادات الـ webhook في Meta

#### ❌ **إذا فشل الاختبار:**
- تحقق من إعدادات الـ instance
- تأكد من صحة الـ access token
- تحقق من أن الـ phone number ID صحيح

### الخطوة 4: فحص الأحداث الأخيرة

1. **اذهب إلى تبويب "Recent Events"**
2. **ابحث عن أحداث webhook حديثة**

#### ✅ **إذا وجدت أحداث:**
- تحقق من الـ response status
- إذا كان 200، المشكلة في معالجة الرسائل
- إذا كان خطأ، اتبع رسالة الخطأ

#### ❌ **إذا لم تجد أحداث:**
- المشكلة في إعدادات الـ webhook في Meta
- تحقق من الـ webhook URL
- تأكد من أن الـ webhook مُفعل في Meta

### الخطوة 5: تشخيص الـ Webhook

1. **اذهب إلى تبويب "Debug Webhook"**
2. **أدخل payload تجريبي:**

```json
{
  "entry": [
    {
      "changes": [
        {
          "value": {
            "messages": [
              {
                "from": "1234567890",
                "type": "text",
                "text": {
                  "body": "Hello World"
                }
              }
            ]
          }
        }
      ]
    }
  ]
}
```

3. **اضغط "Debug Payload"**
4. **تحقق من النتائج في console**

## 🔍 تشخيص متقدم

### فحص Logs في الخادم

ابحث عن هذه الرسائل في console الخادم:

```
🚀 Webhook POST received for instance: {instanceId}
✅ Instance found: {name} (Active: {isActive})
🔐 Verifying webhook signature...
✅ Webhook signature verified
🔄 Parsing incoming events...
📨 Parsed {count} events
💬 Processing event from: {phone}
💾 Saving message to database...
✅ Message saved with ID: {messageId}
📡 Broadcasting message to WebSocket clients...
✅ Message broadcasted
🎉 Webhook processing completed in {duration}ms
```

### فحص قاعدة البيانات

```sql
-- فحص الـ instances
SELECT id, name, is_active, webhook_behavior FROM whatsapp_instances;

-- فحص المحادثات
SELECT id, phone, display_name, instance_id FROM conversations ORDER BY last_at DESC LIMIT 10;

-- فحص الرسائل
SELECT m.id, m.direction, m.body, m.created_at, c.phone 
FROM messages m 
JOIN conversations c ON m.conversation_id = c.id 
ORDER BY m.created_at DESC LIMIT 20;

-- فحص webhook events
SELECT id, instance_id, created_at, response 
FROM webhook_events 
ORDER BY created_at DESC LIMIT 10;
```

## 🛠️ الحلول الشائعة

### 1. **مشكلة: "Instance not found"**
**الحل:**
- تحقق من أن الـ instance ID صحيح
- تأكد من أن الـ instance موجود في قاعدة البيانات

### 2. **مشكلة: "Invalid signature"**
**الحل:**
- تحقق من الـ app secret في Settings
- تأكد من أن الـ signature header موجود
- أو أزل الـ app secret إذا لم تكن تحتاجه

### 3. **مشكلة: "No events parsed from payload"**
**الحل:**
- تحقق من هيكل الـ payload من Meta
- تأكد من وجود `entry` و `changes` و `messages`
- استخدم أداة Debug Webhook لفحص الـ payload

### 4. **مشكلة: الرسائل لا تظهر في التطبيق**
**الحل:**
- تحقق من اتصال WebSocket
- تأكد من أن الرسائل محفوظة في قاعدة البيانات
- تحقق من console المتصفح للأخطاء

### 5. **مشكلة: الـ webhook لا يستقبل طلبات**
**الحل:**
- تحقق من الـ webhook URL في Meta
- تأكد من أن الخادم متاح من الإنترنت
- تحقق من firewall settings

## 📞 طلب المساعدة

إذا لم تحل المشكلة، قدم هذه المعلومات:

1. **نتائج صفحة التشخيص** (screenshot)
2. **رسائل الخطأ** من console الخادم
3. **معلومات الـ instance** (بدون بيانات حساسة)
4. **خطوات إعادة الإنتاج**

---

## 🎯 نصائح مهمة

1. **استخدم أداة التشخيص أولاً** قبل البحث عن حلول أخرى
2. **تحقق من Logs** في console الخادم
3. **اختبر الإرسال** قبل التركيز على الاستلام
4. **تحقق من إعدادات Meta** إذا لم تصل webhook events
5. **استخدم Debug Webhook** لفهم هيكل البيانات

هذا الدليل يجب أن يساعدك في حل معظم مشاكل استلام الرسائل. إذا استمرت المشكلة، استخدم المعلومات المجمعة لطلب المساعدة.
