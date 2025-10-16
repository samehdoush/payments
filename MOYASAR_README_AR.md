# إضافة بوابة الدفع Moyasar

## نظرة عامة

تم إضافة بوابة الدفع **Moyasar** بنجاح إلى حزمة Nafezly Payments. Moyasar هي بوابة دفع سعودية تدعم:

- **الدفع بالبطاقات الائتمانية** (Visa, Mastercard, Mada)
- **Apple Pay**
- **STC Pay**

## الملفات المضافة

### 1. فئة الدفع الرئيسية
📁 `src/Classes/MoyasarPayment.php`

تحتوي على:
- وظيفة `pay()` لإنشاء طلب الدفع
- وظيفة `verify()` للتحقق من الدفع
- دعم طرق الدفع المتعددة
- تحويل تلقائي للمبالغ إلى أصغر وحدة عملة

### 2. واجهة HTML
📁 `resources/views/html/moyasar.blade.php`

صفحة دفع كاملة تحتوي على:
- تصميم عصري ومتجاوب
- تحميل مكتبة Moyasar JavaScript
- نموذج الدفع التفاعلي
- معالجة الأحداث (النجاح، الفشل)
- عرض المبلغ والعملة

### 3. ملف الإعدادات
📁 `config/nafezly-payments.php`

تم إضافة:
```php
#MOYASAR
'MOYASAR_SECRET_KEY'=>env('MOYASAR_SECRET_KEY'),
'MOYASAR_PUBLISHABLE_KEY'=>env('MOYASAR_PUBLISHABLE_KEY'),
'MOYASAR_CURRENCY'=>env('MOYASAR_CURRENCY','SAR'),
```

### 4. Service Provider
📁 `src/NafezlyPaymentsServiceProvider.php`

تم إضافة:
- استيراد فئة MoyasarPayment
- تسجيل الفئة في حاوية Laravel

### 5. التوثيق والأمثلة
📁 `examples/MOYASAR_INTEGRATION.md` - دليل التكامل الشامل بالإنجليزية
📁 `examples/moyasar_example.php` - أمثلة عملية للاستخدام

## خطوات التفعيل

### 1. إضافة مفاتيح API إلى ملف .env

```env
# Moyasar Payment Gateway
MOYASAR_SECRET_KEY=sk_test_xxxxxxxxxxxxxxxxxxxxxxxxxx
MOYASAR_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxxxxxxxx
MOYASAR_CURRENCY=SAR
```

**ملاحظة مهمة:** Moyasar يستخدم نوعين من المفاتيح فقط:
- **Secret Key** (`sk_test_xxx` أو `sk_live_xxx`): يُستخدم لجميع عمليات Backend API (التحقق، الاستعلام، إلخ)
- **Publishable Key** (`pk_test_xxx` أو `pk_live_xxx`): يُستخدم فقط في نموذج الدفع بالـ Frontend
- **لا تشارك Secret Key أبدًا في كود Frontend!**

### 2. الحصول على مفاتيح API

1. سجل في [لوحة تحكم Moyasar](https://dashboard.moyasar.com/register/new)
2. انتقل إلى الإعدادات > API Keys
3. ستجد مفتاحين:
   - **Secret Key**: للاستخدام في Backend (يبدأ بـ `sk_test_` أو `sk_live_`)
   - **Publishable Key**: للاستخدام في Frontend (يبدأ بـ `pk_test_` أو `pk_live_`)
4. للاختبار، استخدم مفاتيح Test
5. للإنتاج، استخدم مفاتيح Live

### 3. إعداد المسارات

```php
// routes/web.php
Route::get('/payment/verify/{payment}', [PaymentController::class, 'verify'])
    ->name('verify-payment');
```

## الاستخدام الأساسي

### إنشاء عملية دفع

```php
use Nafezly\Payments\Classes\MoyasarPayment;

$payment = new MoyasarPayment();

$result = $payment
    ->setAmount(100)                    // المبلغ بالريال السعودي
    ->setUserFirstName('أحمد')          // الاسم الأول
    ->setUserLastName('محمد')            // الاسم الأخير
    ->setUserEmail('ahmed@example.com') // البريد الإلكتروني
    ->setUserPhone('966501234567')      // رقم الجوال
    ->pay();

// عرض نموذج الدفع
return view('payment', ['html' => $result['html']]);
```

### التحقق من الدفع

```php
public function verify(Request $request)
{
    $payment = new MoyasarPayment();
    $result = $payment->verify($request);
    
    if ($result['success']) {
        // نجحت العملية
        $paymentId = $result['payment_id'];
        $data = $result['process_data'];
        
        // احفظ في قاعدة البيانات، أرسل بريد تأكيد، إلخ.
        
        return redirect('/success');
    } else {
        // فشلت العملية
        return redirect('/failed');
    }
}
```

## طرق الدفع المختلفة

### 1. البطاقات الائتمانية فقط
```php
$result = $payment
    ->setAmount(100)
    ->setSource('creditcard')
    ->pay();
```

### 2. STC Pay فقط
```php
$result = $payment
    ->setAmount(100)
    ->setSource('stcpay')
    ->pay();
```

### 3. Apple Pay فقط
```php
$result = $payment
    ->setAmount(100)
    ->setSource('applepay')
    ->pay();
```

### 4. جميع الطرق (افتراضي)
```php
$result = $payment
    ->setAmount(100)
    ->pay(); // لا تحدد source
```

## هيكل الاستجابة

### استجابة ناجحة
```php
[
    'success' => true,
    'payment_id' => 'abc123...',
    'message' => 'تمت العملية بنجاح',
    'process_data' => [
        'id' => 'abc123...',
        'status' => 'paid',
        'amount' => 10000, // بالهللة (100 ريال)
        'currency' => 'SAR',
        // ... بيانات أخرى
    ]
]
```

### استجابة فاشلة
```php
[
    'success' => false,
    'payment_id' => 'abc123...',
    'message' => 'فشلت العملية',
    'process_data' => [/* تفاصيل الخطأ */]
]
```

## تحويل العملات

يتم تحويل المبلغ تلقائيًا إلى أصغر وحدة عملة:

- **SAR**: 1 ريال = 100 هللة
- **KWD**: 1 دينار = 1000 فلس
- **USD**: 1 دولار = 100 سنت

مثال:
```php
->setAmount(100)  // 100 ريال
// يتم تحويله تلقائيًا إلى 10000 هللة في API
```

## الاختبار

### بطاقات الاختبار

**دفع ناجح:**
- رقم البطاقة: `4111 1111 1111 1111`
- CVV: أي 3 أرقام
- تاريخ الانتهاء: أي تاريخ مستقبلي

**دفع فاشل:**
- رقم البطاقة: `4000 0000 0000 0002`

**يتطلب 3D Secure:**
- رقم البطاقة: `4000 0000 0000 3063`

### اختبار STC Pay
- رقم الجوال: `0500000001`
- رمز OTP: `1234`

## الميزات الإضافية

### 1. معرف دفع مخصص
```php
$payment
    ->setPaymentId('ORDER-2024-00123')
    ->setAmount(100)
    ->pay();
```

### 2. إضافة معلومات المستخدم
```php
$payment
    ->setUserId(123)
    ->setUserFirstName('أحمد')
    ->setUserLastName('السعيد')
    ->setUserEmail('ahmed@example.com')
    ->setUserPhone('966501234567')
    ->pay();
```

### 3. عملات مختلفة
```php
$payment
    ->setAmount(50)
    ->setCurrency('USD')
    ->pay();
```

## ملاحظات الأمان

1. ✅ **لا تشارك مفتاح API السري أبدًا**
2. ✅ **تحقق دائمًا من الدفع على السيرفر**
3. ✅ **استخدم HTTPS في الإنتاج**
4. ✅ **تحقق من صحة callback URL**
5. ✅ **سجل جميع المعاملات**

## استكشاف الأخطاء

### نموذج الدفع لا يظهر
- تحقق من صحة مفتاح Publishable Key
- تأكد من تحميل ملفات Moyasar CSS و JS
- افحص console المتصفح

### فشل التحقق من الدفع
- تحقق من صلاحيات مفتاح API
- تأكد من تمرير payment ID بشكل صحيح
- تحقق من قدرة السيرفر على إرسال طلبات HTTPS

### عدم تطابق المبلغ
- المبالغ تُحول تلقائيًا لأصغر وحدة
- تحقق دائمًا من المبلغ بعد الدفع

## الدعم الفني

**Moyasar:**
- الموقع: [https://moyasar.com](https://moyasar.com)
- الدعم: [care@moyasar.com](mailto:care@moyasar.com)
- هاتف: 800 1111 848
- التوثيق: [https://docs.moyasar.com](https://docs.moyasar.com)

## الخلاصة

تم تكامل Moyasar بنجاح مع حزمة Nafezly Payments باستخدام:

✅ Form Integration (وليس API المباشر)  
✅ دعم جميع طرق الدفع (بطاقات، Apple Pay، STC Pay)  
✅ واجهة HTML كاملة مع تصميم متجاوب  
✅ معالجة تلقائية للمبالغ والعملات  
✅ توثيق شامل وأمثلة عملية  
✅ متوافق مع بنية البكج الحالية  

**التكامل جاهز للاستخدام! 🎉**
