<div dir="rtl">

# اسناد الکترونیکی سازمان پزشکی قانونی
![app](pictures\header_logo_fa.png)

  واحد فن آوری اطلاعات

 # اهداف طرح

 سازمان پزشکی قانونی کشور در نظر دارد که در راستای الکترونیکی کردن سرویسهای خود در تسریع و بهینه سازی ارسال و دریافت اطلاعات و پرونده بیماران تحت بررسی، امکانات و شرایط پیاده سازی تعاملات مربوطه را در بستر اینترنت فراهم کند. 

***در این طرح به تشریح مدلها و سرویسها خواهیم پرداخت.***

 
 # مقدمه
در این طرح یک WebAPI Net Core در شبکه زیرساخت کشور نصب شده و به سازمان پزشکی قانونی کشور سرویسهای مورد نظر ایشان را ارائه می دهد. اپلیکیشن پزشکی قانونی تنها از طریق یک token مشخص که قبلا و به شکل محرمانه به آن سازمان ارائه شده است با این API تماس گرفته و سرویس تنها از طریق این کلید ارائه می شود. 

WebAPI ساخته شده که در این مقاله ساختار و عملکرد آن را تشریح خواهیم کرد، یک واسط بین سازمان پزشکی قانونی کشور و بیمارستانهای دانشگاه علوم پزشکی ایران است که به شکل کاملا سنکرون درخواستها را از سازمان پزشکی قانونی گرفته و به تمام بیمارستانهای تابعه خود ارسال کرده؛ منتظر جواب مانده و نتیجه را به سازمان پزشکی قانونی کشور می فرستد. 

از 5 سرویس ارائه شده، در دو سرویس اولی قاعده بالا رعایت شده است. سرویس Get_AdmissionListBaseAdmissionID که ورودی آن شناسه پذیرش بیمار است و سرویس Get_AdmissionListBasePatientNationalCode  که ورودی آن کد ملی بیمار است؛ در قالب یک آرایه از درخواسته ها به شکل همزمان به لیستی از کلیه بیمارستانها ارسال می شوند. نتیحه هر چه که باشد با گذشت زمان مشخصی به سازمان پزشکی قانونی بر می گردد. 


#
![app](pictures\Presentation1.png)
#


 <h1>
بررسی عملکرد سرویسها
 </h1>

<h2>
سرویس 
Get_AdmissionListBaseAdmissionID
</h2>

دارای دو ورودی شامل admissionId , token است. با مراجعه به سورس برنامه می توان دید که لیست بیمارستانها در فایل Services\SiamURLDataSet\SiamInfo.json ذخیره شده و این لیست از داخل سرور واسط قابل ویرایش کردن است. نام مرکز، کد سیام، URL مرکزکه سرویس Restful Api روی آن نصب شده و کلید گدرواژه استفاده از سرویس هر بیمارستان در آن قرار داده شده است. 
برای سادگی کلید کلیه ارتباط ها فعلا 1234 است تا در عمل از کلیدهای پیچیده غیر قابل حدس استفاده شود. 

قابل توجه اینکه کلیه سرویسهای WebAPI چه در سمت بیمارستانها و چه در سمت سرور واسط از نوع https باید یاشند تا ارتباطات محرمانه و امن باشند. در این پروژه با کمک کد:
<br/>
</div>
<blockquote>
<div class="highlight">
  <code>  ServicePointManager.ServerCertificateValidationCallback +=
                (sender, certificate, chain, sslPolicyErrors) => true; </code>  
</div>
</blockquote>
<div dir="rtl">
<br/>

خاصیت الزام https فعلا از بین برده شده است تا برای تست برنامه نیاز به فعال سازی اکستنشن CORS در مرورگر نداشته باشیم.
در زمان فراخوانی این سرویس یک لیست Thread ایجاد شده که شامل آرایه ای از درخواستها برای تمام بیمارستانهای داخل لیست است. در اینجا به یکباره و همزمان به تعداد بیمارستانهای داخل لیست، درخواست متناظر به سرور های لیست ارسال شده و اگر مرکزی جواب ندهد، در مدل دریافت شده، پرچم Rejected به مقدار true ست می شود که به معنی آن است که بیمارستان مربوطه پاسخی نداده است و اگر پرچم فوق false باشد باید چک کرد که وضعیت پرچم notfound چیست که اگر true باشد یعنی بیمارستان مربوطه پاسخ داده است اما رکوردی پیدا نشده است. در صورتی که هر دوی پرچمها false باشند قطعا بیمارستان مربوطه حداقل یک رکورد برگردانده است و تعداد رکوردهای بیشتر از 1 به معنی آن است که با این آی دی دریافتی بیش از یکبار بیمار پذیرش شده یا تصادفا دو مرکز از یک آی دی پذیرش استفاده کرده اند. 
<br/> 
در مدل دریافتی یک آیتم به نام deliveryTime وجود دارد که شاخصی در مدت زمان پاسخ به میلی تانیه است که سازمان پزشکی قانونی یا مدیران سرور واسط همیشه مانیتور می کنند که درخواستها در چه مدت زمانی در سرور واسط تحلیل و پاسخ داده شده اند. 
<br/>
<br/>
از آنجا که در این فاز از پروژه هنوز WebAPI در سمت بیمارستان وجود ندارد و قرار است تا از طریق تدوین مقررات رگولاتوری استانداردهای سرویس ها به بیمارستانها ابلاغ بشوند، یک پروژه نوشته شد که در مسیر \Siams\Siam_Virtual_1 قرار داده شده است و متدهای آن تا حدودی شبیه متدهای سرور واسط هستند. 
برای اجرای سرویس از مسیر bin\Debug\netcoreapp3.1 فایل Siam_Virtual_1.exe  را اجرا کنید.



 به صورت پیش فرض پورت 5001برای این منظور تخصیص داده می شود. برای تغییر آن به 5003 در هر شبیه ساز بیمارستانی از کد زیر در program.cs استفاده کنید:

<br/>
</div>
<blockquote>
<div class="highlight">
  <code>  public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                    webBuilder.UseUrls("http://localhost:5003", "https://localhost:5004");
                }); </code>  
</div>
</blockquote>
<div dir="rtl">
<br/>

در پوشه VirtulDB دو عدد فایل است که شبیه ساز لیست پذبرش بیمارستان و لیست پرونده الکترونیک هستند. سرور شبیه ساز این لیست ها را خوانده و به درخواستها بر اساس مدل طراحی شده پاسخ می دهد. 
مشابه سرویس واسط اینجا هم یک سرویس تعریف شده است که به عنوان ورودی admissionNumber و token را گرفنه و اگر آیتمی را در VirtualDB\Admisstions.jsonپیدا کند که AdmissionIdآن با ورودی برابر باشد آنرا در خروجی ارسال می کند. 

<br/>
</div>
<blockquote>
<div class="highlight">
  <code>  
  [HttpGet("[action]")]
        public async Task<IActionResult> GetAdmissionsBaseAdmissionNumber(string admissionNumber, string token)
        <br/>
        {
          <br/>
            var checkToken = new CheckToken();
            <br/>
            if (token != checkToken.Token) return Conflict("There is no permission for using this service!");
            <br/>
            var jsonString = new string(await System.IO.File.ReadAllTextAsync(@"VirtualDB\Admisstions.json"));
            <br/>
            var admissions =
                JsonSerializer.Deserialize<List<AdministrationModel>>(jsonString);
                <br/>
            var result = admissions.Where(t => t.AdmissionId == admissionNumber).ToList();
            <br/>
            return Ok(result);<br/>
        }
  </code>  
</div>
</blockquote>
<div dir="rtl">
<br/>
با اجرای فایل اجرایی یک پنجره مشغول شنود فرامین است. شکل زیر:
<br/>
<br/>


</div>

![app](pictures\VirtualAPI_RUN.png)
<br/>
<div dir="rtl">
اگر بخواهیم بیمارستانهای بیشتری را شبیه سازی کنیم باید ابتدا پورت را به عنوان مثال به 6003 و 6004  مطابق کد تغییر داده و کامپایل کنیم. در خروجی پوشه bin را جایی کپی کرده و محتویات json را در پوشه VitualDB تغییر دهیم.  اکنون فایل را اجرا کنیم. به خاطر داشته باشیم که آدرس جدید باید در مسیر ForensicTaskBase\Services\SiamURLDataSet\SiamInfo.json تغییر کند تا API واسط بتواند آنرا در فراخوانی در نظر بگیرد.

<br/>
<br/>
<br/>

</div>

![app](pictures\Presentation2.png)
<br/>

</div>


```javascript
{
  "response": [
    {
      "index": "e887fbff-1249-4efd-b3e9-a8c6410ce47b",
      "siamCode": "0bb37a66-0cbb-627a-cc82-ac4900795ea8",
      "siamName": "فیروزگر",
      "admissionId": "99-87-5444321-99",
      "siamURL": "http://localhost:5003",
      "admissionDate": "1389-01-02",
      "dischargedDate": "1389-01-03",
      "patientNationalCode": "0943260681",
      "patientName": "آرش ابراهیمی",
      "rejected": false,
      "notFound": false
    },
    {
      "index": "00000000-0000-0000-0000-000000000000",
      "siamCode": "68f906ea-0832-4f30-aacf-fe61fd1e5248",
      "siamName": "حضرت رسول اکرم(ص)",
      "admissionId": "99-87-5444321-99",
      "siamURL": "http://localhost:6003",
      "admissionDate": null,
      "dischargedDate": null,
      "patientNationalCode": null,
      "patientName": null,
      "rejected": true,
      "notFound": true
    },
    {
      "index": "00000000-0000-0000-0000-000000000000",
      "siamCode": "f059fbf7-fb46-96df-e096-ffaae328f992",
      "siamName": "قلب وعروق شهید رجائی",
      "admissionId": "99-87-5444321-99",
      "siamURL": "http://localhost:7003",
      "admissionDate": null,
      "dischargedDate": null,
      "patientNationalCode": null,
      "patientName": null,
      "rejected": true,
      "notFound": true
    },
    {
      "index": "00000000-0000-0000-0000-000000000000",
      "siamCode": "09a14242-1cca-0da2-2ee8-0bb544b47d21",
      "siamName": "فیروزآبادی",
      "admissionId": "99-87-5444321-99",
      "siamURL": "http://localhost:8003",
      "admissionDate": null,
      "dischargedDate": null,
      "patientNationalCode": null,
      "patientName": null,
      "rejected": true,
      "notFound": true
    }
  ],
  "deliveryTime": 5120
}
```
<div dir="rtl">
همانطور که مشاهده می کنید زمان پاسخ 5120 میلی ثانیه و فقط فیروزگر جواب داده و فیروزگر یک رکورد را ارسال کرده است.
پاسخ فوق ناشی از درخواست زیر است:
</br>
</br>
</div>
http://localhost:63539/Services/Get_AdmissionListBaseAdmissionID?admissionId=99-87-5444321-99&token=1234
</div>
</br>
</br>
<div dir="rtl">
<h2>
سرویس 
Get_AdmissionListBasePatientNationalCode
</h2>
خیلی شبیه سرویس قبلی است اما کد ملی به عنوان ورودی دریافت شده و نتیجه با ساختاری مشابه json بالا در خروجی ظاهر می شود. 
 نمونه کد زیر:
</div>
http://localhost:63539/Services/Get_AdmissionListBasePatientNationalCode?nationalCode=0943260681&token=1234
<div dir="rtl">
نتیجه زیر را بر می گرداند:
</div>

```javascript
{
  {
  "response": [
    {
      "index": "e887fbff-1249-4efd-b3e9-a8c6410ce47b",
      "siamCode": "0bb37a66-0cbb-627a-cc82-ac4900795ea8",
      "siamName": "فیروزگر",
      "admissionId": "99-87-5444321-99",
      "siamURL": "http://localhost:5003",
      "admissionDate": "1389-01-02",
      "dischargedDate": "1389-01-03",
      "patientNationalCode": "0943260681",
      "patientName": "آرش ابراهیمی",
      "rejected": false,
      "notFound": false
    },
    {
      "index": "52d70576-ba06-418b-bb41-a33da722d64d",
      "siamCode": "0bb37a66-0cbb-627a-cc82-ac4900795ea8",
      "siamName": "فیروزگر",
      "admissionId": "89-87-5444321-99",
      "siamURL": "http://localhost:5003",
      "admissionDate": "1389-01-03",
      "dischargedDate": "1389-01-03",
      "patientNationalCode": "0943260681",
      "patientName": "آرش ابراهیمی",
      "rejected": false,
      "notFound": false
    },
    {
      "index": "00000000-0000-0000-0000-000000000000",
      "siamCode": "68f906ea-0832-4f30-aacf-fe61fd1e5248",
      "siamName": "حضرت رسول اکرم(ص)",
      "admissionId": null,
      "siamURL": "http://localhost:6003",
      "admissionDate": null,
      "dischargedDate": null,
      "patientNationalCode": "0943260681",
      "patientName": null,
      "rejected": true,
      "notFound": true
    },
    {
      "index": "00000000-0000-0000-0000-000000000000",
      "siamCode": "f059fbf7-fb46-96df-e096-ffaae328f992",
      "siamName": "قلب وعروق شهید رجائی",
      "admissionId": null,
      "siamURL": "http://localhost:7003",
      "admissionDate": null,
      "dischargedDate": null,
      "patientNationalCode": "0943260681",
      "patientName": null,
      "rejected": true,
      "notFound": true
    },
    {
      "index": "00000000-0000-0000-0000-000000000000",
      "siamCode": "09a14242-1cca-0da2-2ee8-0bb544b47d21",
      "siamName": "فیروزآبادی",
      "admissionId": null,
      "siamURL": "http://localhost:8003",
      "admissionDate": null,
      "dischargedDate": null,
      "patientNationalCode": "0943260681",
      "patientName": null,
      "rejected": true,
      "notFound": true
    }
  ],
  "deliveryTime": 5635
}
}
```
</div>
<div dir="rtl">
دقت کنیم که بیمار با کد ملی مشخص دوبار در بیمارستان فیروزگر مراجعه داشته و بقیه 3 بیمارستان پاسخ نداده اند.
<br/>
<br/>

</div>

<div dir="rtl">
<h2>
سرویس 
Get_EMR
</h2>
زمانی که با لیست ارسالی فوق و از فیلد index، آی دی مراجعه به عنوان اندیس مشخص شد و کد سیام مرکزی که پرونده بیمار را می خواهیم از آنجا فراخوانی کنیم را در لیست بالا مشخص کردیم، کافی است با کمک این متد، لیست پرونده های آن مراجعه بیمار را از بیمارستان مربوطه مطالبه کنیم. کد دستوری فوق، پرونده با اندیس e887fbff-1249-4efd-b3e9-a8c6410ce47b را از فیروزگر مطالبه کرده و فیروزگر جواب داده است:
<br/>
<br/>
</div>
http://localhost:63539/Services/Get_EMR?admissionUid=e887fbff-1249-4efd-b3e9-a8c6410ce47b&siamCode=0bb37a66-0cbb-627a-cc82-ac4900795ea8&token=1234
<br/>

```javascript
{
  "response": [
    {
      "id": "7539814c-4fef-4a0a-83cd-2498eb961527",
      "admissionUid": "e887fbff-1249-4efd-b3e9-a8c6410ce47b",
      "date": "1399-09-07",
      "time": "16:55",
      "ministryOfHealthEMRCode": 1,
      "ministryOfHealthEMRName": "پذیرش وخلاصه ترخیص",
      "desciption": "",
      "item": [
        {
          "id": "8649ecf3-3c6d-4a13-9046-d22ce161fb2a",
          "type": ".jpg"
        },
        {
          "id": "84192700-9a9e-44e7-9599-6b948e30bd2c",
          "type": ".jpg"
        }
      ]
    },
    {
      "id": "2c9224b7-6179-4373-83c3-0e4c627f2c2b",
      "admissionUid": "e887fbff-1249-4efd-b3e9-a8c6410ce47b",
      "date": "1399-09-07",
      "time": "20:15",
      "ministryOfHealthEMRCode": 5,
      "ministryOfHealthEMRName": "مشاوره ها",
      "desciption": "",
      "item": [
        {
          "id": "0972ed09-6df3-4774-852c-58b6ed90fae3",
          "type": ".jpg"
        }
      ]
    },
    {
      "id": "8a8656aa-13ad-4224-bf6a-974e523ddba8",
      "admissionUid": "e887fbff-1249-4efd-b3e9-a8c6410ce47b",
      "date": "1399-09-08",
      "time": "08:21",
      "ministryOfHealthEMRCode": 21,
      "ministryOfHealthEMRName": "علایم حیاتی",
      "desciption": "",
      "item": [
        {
          "id": "ac890110-7b34-4b2a-83c2-299c96ff8f72",
          "type": ".jpg"
        }
      ]
    }
  ],
  "deliveryTime": 602
}
```

</div>
<div dir=rtl>
فیروزگر سه بسته اطلاعاتی ارسال کرده است که هر بسته می تواند حاوی آدرس چندین فایل باشد. بسته اول از نوع پذیرش و خلاصه ترخیص شامل دو برگه اسکن شده jpg است که آی دی هر برگه در سند ارسالی مشخص است و در زمان درخواست فایل باید به سرور واسط ارسال بشود. مشاوره تک برگی است و علایم حیاتی نیز تک برگی می باشد. نکته اینکه یک آیتم از پرونده می تواند مثلا دارای یک فایل pdf شامل 3 صفحه و یک فایل png از یک عکس از سند و مثلا 83 عکس jpg باشد که عملا سامانه PACS آنها را از دایکام به این فرمت تبدیل کرده و ارسال می کند. از آنجا که این لیست در اختیار سازمان پزشکی قانونی کشور قرار می گیرد، در مرحله درخواست فایل سند بر اساس لیست ارسالی، لیستی از اندیسهای مدارک مورد نظر در فرانت اند سازمان پزشکی قانونی تهیه شده و دقیقا مطابق با همین درخواست، فایلهای مورد نظر سازمان مربوطه آپلود می شوند. 
<br/>
نتیجه ارسال فوق از json شبیه سازی WebAPI فیروزگر تولید شده که در پوشه Siams\Siam_Virtual_1\bin\Debug\netcoreapp3.1\VirtualDB  در فایل EMRList.json قرار داده شده است. 
</div>
<br/>
<br/>
<div dir="rtl">
<h2>
سرویس 
Get_Documnet
</h2>
این سرویس یک فایل را بر می گرداند. ورودی، پارامترهای سرویس قبلی هستند بعلاوه اندیس فایل. فرض بر این است که WebAPI بیمارستان با دریافت این پارامترها از داخل سامانه خود فایل را به سازمان پزشکی قانونی کشور بنا به درخواست ارسال شده می فرستند. سازمان پزشکی قانونی کشور در زمان درخواست می داند که فرمت فایل دریافتی چیست (مثلا .jpg) و آنرا داخل سامانه خود با همان پسوند ذخیره می کند. 
<br/>
به عنوان شبیه سازی، سرویس شبیه ساز به این شکل عمل می کند که اگر در پوشه Siam_Virtual_1\bin\Debug\netcoreapp3.1\VirtualDB\DataFile فایل اندیس درخواستی را پیدا کند آنرا خوانده و به سرور واسط ارسال می کند. سرور واسط نیز بلادرنگ آنرا به سازمان پزشکی قانونی ارسال می کند. 
<br/>
نکته قابل توجه اینکه از آنجا که در زیرساخت، سیستم ها در بستر فیبر نوری کار می کنند، حداقل سرعت شبکه 100مگابیت بر ثانیه است که زمان زیادی در ارسال اطلاعات از بیمارستان به سرور واسط تلف نخواهد شد و اگر اپلیکیشن سازمان پزشکی قانونی نیز در زیرساخت باشد عملا سرعت انتقال اطلاعات کاملا برای کار با سیستم، روان و مناسب است. 
<br/>
<br/>

</div>
<div dir="rtl">
<h2>
سرویس 
GetAdmissionsBasePatientNationalCodeInterval
</h2>
<br>
مورد خطاب این درخواست، یک بیمارستان مشخص از یک کد ملی مشخص در یک بازه زمانی مشخص
است که با فرمتYYYY-MM-DD و با شکل جلالی باید ارسال بشود. به عنوان مثال startDate="1399-01-06" و stopDate="1400-01-01" مراجعات نیمه دوم سال 1399 و اولین روز سال 1400 را مورد نظر قرار می دهد.



