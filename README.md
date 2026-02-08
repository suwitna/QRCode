#  Microsoft Report Builder (15 or Higher) with QRCoder.dll / SSRS
การสร้าง QR Code โดยใช้ QRCoder.dll และ Microsoft Report Builder (เวอร์ชัน 15.1 ขึ้นไป)

เอกสารนี้อธิบายวิธีการผสานการทำงานของ **QRCoder.dll** เข้ากับ  
**Microsoft Report Builder (v15.1 ขึ้นไป)** ซึ่งใช้งานร่วมกับ  
**SQL Server Reporting Services (SSRS)** หรือ **Power BI Report Server**  
เพื่อสร้าง **QR Code** ภายในรายงานแบบ **Paginated Report**

---

## 📦 ไลบรารีที่ใช้

- **QRCoder**
  - เวอร์ชันที่แนะนำ: `1.3.6` ขึ้นไป  
  - เวอร์ชันล่าสุด: `1.7.0`
  - ภาษา: C#
  - ที่มา: https://www.nuget.org/api/v2/package/QRCoder/1.7.0

<img width="461" height="265" alt="image" src="https://github.com/user-attachments/assets/7db1df73-92dd-4bd5-b4d4-823404a01ea3" />

---

## 📄 ไลเซนส์ (License)

QRCoder ใช้ **MIT License**

คุณสามารถ:
- ใช้งานเชิงพาณิชย์หรือภายในองค์กรได้
- ใช้งานในระบบออฟไลน์ / ระบบปิด (เช่น โรงงาน)
- แก้ไขและแจกจ่ายไฟล์ DLL ได้

⚠️ เงื่อนไขเดียวที่ต้องปฏิบัติตาม:
- ต้องเก็บข้อความ Copyright และ License ของ MIT ไว้ในเอกสารหรือซอร์สโค้ด

ไม่มีการตรวจสอบไลเซนส์ออนไลน์  
ไม่มีการเชื่อมต่อกลับไปยัง NuGet หรืออินเทอร์เน็ตใด ๆ

---

## 🛠️ สิ่งที่ต้องเตรียม (Prerequisites)

### 1. QRCoder.dll
- ดาวน์โหลดแพ็กเกจจาก NuGet (`.nupkg`)
- แตกไฟล์และเลือก DLL จากโฟลเดอร์:
  - `net35` หรือ
  - `net40`  
(แนะนำสำหรับ SSRS / Report Builder)

### 2. สิทธิ์การเข้าถึงระบบ
- ต้องมีสิทธิ์ **Administrator**
- สำหรับการคัดลอกไฟล์ DLL ไปยัง:
  - Report Server
  - หรือ Global Assembly Cache (GAC)

---

## 🚀 ขั้นตอนการติดตั้ง (Implementation Steps)

### 1. ติดตั้ง QRCoder.dll

คัดลอกไฟล์ `QRCoder.dll` ไปยัง:

- **เครื่องออกแบบรายงาน**
```vb
- C:\Program Files (x86)\Microsoft SQL Server Reporting Services\Report Builder\PrivateAssemblies
``` 
- **เครื่อง Report Server**
```vb
- C:\Program Files\Microsoft SQL Server\Reporting Services\SSRS\ReportServer\bin
```

<img width="684" height="219" alt="image" src="https://github.com/user-attachments/assets/b3d17378-3037-48c9-89b8-57a8e8cc0372" />


(ตำแหน่งอาจแตกต่างตามเวอร์ชัน SSRS)

---

### 2. เพิ่ม Reference ใน Report Builder

1. เปิด Report Builder
2. ไปที่ **Report Properties**
3. เลือกแท็บ **References**
4. เพิ่ม Reference ไปยัง `QRCoder.dll`

<img width="648" height="580" alt="image" src="https://github.com/user-attachments/assets/5281ca42-04f4-4f24-be03-19b02ebcd4b0" />
<img width="792" height="280" alt="image" src="https://github.com/user-attachments/assets/e5abc82c-4341-4b35-b52a-8cd93b7e7e10" />

---

### 3. เพิ่ม Custom Code (VB.NET)

ไปที่ **Report Properties > Code**  
เพิ่มโค้ดดังนี้:

```vb
Public Function GenerateQR(ByVal data As String) As Byte()
  Dim qrGenerator = New QRCoder.QRCodeGenerator()
  Dim qrCodeData = qrGenerator.CreateQrCode(data, QRCoder.QRCodeGenerator.ECCLevel.Q)
  Dim qrCode = New QRCoder.BitmapByteQRCode(qrCodeData)
  Return qrCode.GetGraphic(20)
End Function
```
<img width="828" height="578" alt="image" src="https://github.com/user-attachments/assets/9e543ea3-3c40-4fc5-9bb6-fbb75f3024ad" />

---

### 4. แสดงผลเป็นรูปภาพ

1. เพิ่ม Image ลงในรายงาน
ตั้งค่า:
```vb
Source = Database
MIME type = image/png
ใส่ Expression:
```
<img width="1914" height="1015" alt="image" src="https://github.com/user-attachments/assets/e5342715-b3d2-4e2a-a052-7b3efb1dd35a" />


<img width="571" height="526" alt="image" src="https://github.com/user-attachments/assets/e98f5c61-e4e9-410e-accb-f76c2e04bf0e" />

```vb
=Code.GenerateQR(Fields!YourDataField.Value)
```

<img width="543" height="612" alt="image" src="https://github.com/user-attachments/assets/6f29aef9-5dcc-4d8d-9475-5f34e44c2663" />

---
### 5. ตัวอย่างรายงาน
<img width="1070" height="616" alt="image" src="https://github.com/user-attachments/assets/7a68fd70-cd3a-4572-ba37-28e9e6dcbc2d" />

---
### 6. อัพโหลดขึ้น SSRS
<img width="726" height="466" alt="image" src="https://github.com/user-attachments/assets/c6ecdcd9-cd4c-426b-a259-e34cf3447c33" />




```vb
=Code.GenerateQR("ภาพที่คุณแนบมามีปัญหาที่เรียกว่า Missing Alignment Pattern ครับ ซึ่งเป็นเหตุผลหลักที่ทำให้เครื่องสแกนอ่านค่าไม่ได้")
```
<img width="300" height="300" alt="image" src="https://github.com/user-attachments/assets/bcaf29bb-e281-41a2-aaa5-13f85017daba" />




```vb
=Code.GenerateQR("M03-M04-M05")
```
<img width="300" height="300" alt="image" src="https://github.com/user-attachments/assets/89527f7e-12b1-4148-a9ca-d4391a0485c6" />




```vb
=Code.GenerateQR("https://www.senior-thailand.com")
```
<img width="300" height="300" alt="image" src="https://github.com/user-attachments/assets/c3070ceb-c670-4074-b6cb-7142842ac457" />

