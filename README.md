#  Report Builder สร้าง QR Code โดยใช้ QRCoder.dll

เอกสารนี้อธิบายวิธีการสร้าง **QR Code** ภายในรายงาน โดยใช้ **Microsoft Report Builder**   
และ **QRCoder.dll** โดยใช้งานร่วมกับ  **SQL Server Reporting Services (SSRS)**  
หรือ **Power BI Report Server** 

---
## คุณสมบัติ
- สร้าง QR Code ภายในรายงาน (RDL)
- ไม่เชื่อมต่ออินเทอร์เน็ต
- ไม่ส่งข้อมูลออกภายนอกระบบ


### ความปลอดภัย
* โปรเจกต์นี้ทำงานภายในระบบรายงานเท่านั้น
ไม่มีการส่งข้อมูลออกภายนอก (No external transmission)

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

- ใช้งานเชิงพาณิชย์หรือภายในองค์กรได้
- ใช้งานในระบบออฟไลน์ / ระบบปิด
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

## 🧩 อธิบายโค้ด

### 1. ประกาศฟังก์ชัน

```vb
Public Function GenerateQR(ByVal data As String) As Byte()
```

* สร้างฟังก์ชันชื่อ `GenerateQR`
* รับข้อมูลเป็น `String`
* ส่งค่ากลับเป็น `Byte()` (Binary Image)

**เหตุผลที่ใช้ `Byte()`**

* Image ใน SSRS (Source = Database) ต้องใช้ข้อมูลรูปแบบ Byte Array
* รองรับ MIME type เช่น `image/png`

**ข้อมูลที่ส่งเข้าได้**

* ข้อความทั่วไป
* ตัวเลข
* URL
* JSON / String ยาว
* Part No, Lot No, Serial No

---

### 2. สร้าง QRCodeGenerator

```vb
Dim qrGenerator = New QRCoder.QRCodeGenerator()
```

* เป็นอ็อบเจ็กต์หลักของ QRCoder
* ใช้แปลงข้อความ → โครงสร้าง QR Code
* ทำงานทั้งหมดใน Memory

---

### 3. สร้าง QRCodeData

```vb
Dim qrCodeData = qrGenerator.CreateQrCode(
    data,
    QRCoder.QRCodeGenerator.ECCLevel.Q
)
```

#### 🔹 `data`

ข้อความที่ต้องการเข้ารหัสเป็น QR Code

ตัวอย่าง:

```text
INV-2026-0001
https://intranet/report?id=123
{"Lot":"A01","Qty":100}
```

#### 🔹 `ECCLevel` (Error Correction Level)

ระดับความสามารถในการกู้คืนข้อมูลเมื่อ QR เสียหาย

| ค่า | ความหมาย | กู้คืนได้ประมาณ | เหมาะกับ          |
| --- | -------- | --------------- | ----------------- |
| L   | Low      | ~7%             | ข้อมูลยาวมาก      |
| M   | Medium   | ~15%            | ใช้งานทั่วไป      |
| Q   | Quartile | ~25%            | งานโรงงาน / พิมพ์ |
| H   | High     | ~30%            | ฉลาก เสี่ยงเลอะ   |

> ค่า `Q` เป็นค่าที่แนะนำ

---

### 4. แปลงเป็น BitmapByteQRCode

```vb
Dim qrCode = New QRCoder.BitmapByteQRCode(qrCodeData)
```

* แปลง QRCodeData → รูปภาพ Bitmap
* เตรียมส่งออกเป็น Byte Array
* เหมาะกับ SSRS / Report Builder

---

### 5. สร้างรูปและส่งกลับ

```vb
Return qrCode.GetGraphic(20)
```

#### 🔹 ค่า `20` คืออะไร?

คือ **Pixels per Module (Scale)**
กำหนดขนาดของ QR Code

| ค่า | ขนาด    | การใช้งาน    |
| --- | ------- | ------------ |
| 5   | เล็กมาก | หน้าจอ       |
| 10  | เล็ก    |              |
| 20  | มาตรฐาน | รายงาน / PDF |
| 30  | ใหญ่    | งานพิมพ์     |
| 40+ | ใหญ่มาก | ป้าย         |

---

## 🔧 ตัวอย่างการปรับแต่งเพิ่มเติม

### ตรวจสอบค่าว่าง (แนะนำ)

```vb
If String.IsNullOrEmpty(data) Then
    Return Nothing
End If
```

---

### เปลี่ยนระดับ Error Correction

```vb
qrGenerator.CreateQrCode(data, QRCoder.QRCodeGenerator.ECCLevel.H)
```

---

### เปลี่ยนขนาด QR Code

```vb
Return qrCode.GetGraphic(30)
```

---

## ✅ Flow การทำงาน

```text
String Data
   ↓
QRCodeGenerator
   ↓
QRCodeData
   ↓
BitmapByteQRCode
   ↓
Byte()
   ↓
SSRS Image
```

---

### 🖼️ การใช้งานใน SSRS / Report Builder

1. เพิ่ม **Image** ลงในรายงาน
2. ตั้งค่า:

   * Source = `Database`
   * MIME Type = `image/png`
3. Expression:

   ```text
   =Code.GenerateQR(Fields!YourDataField.Value)
   ```

<img width="1914" height="1015" alt="image" src="https://github.com/user-attachments/assets/e5342715-b3d2-4e2a-a052-7b3efb1dd35a" />


<img width="571" height="526" alt="image" src="https://github.com/user-attachments/assets/e98f5c61-e4e9-410e-accb-f76c2e04bf0e" />


<img width="543" height="612" alt="image" src="https://github.com/user-attachments/assets/6f29aef9-5dcc-4d8d-9475-5f34e44c2663" />


---
### ตัวอย่างรายงาน
<img width="1070" height="616" alt="image" src="https://github.com/user-attachments/assets/7a68fd70-cd3a-4572-ba37-28e9e6dcbc2d" />

---
### อัพโหลดขึ้น SSRS
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

---
## การปรับตำแหน่งของ QR Code
- ก่อนปรับ Padding
<img width="376" height="154" alt="image" src="https://github.com/user-attachments/assets/0d591610-ad9b-4127-af31-e66baa00e8ec" />

<img width="839" height="640" alt="image" src="https://github.com/user-attachments/assets/ca92383a-c796-4bd0-816b-91ac6d507b4f" />


- ปรับ Padding
<img width="1173" height="791" alt="image" src="https://github.com/user-attachments/assets/87b5d8af-083e-4901-8ad1-7f8071249cfc" />

<img width="574" height="528" alt="image" src="https://github.com/user-attachments/assets/d30ebed7-b554-403f-8a46-4c3d7da5785e" />

<img width="571" height="527" alt="image" src="https://github.com/user-attachments/assets/368ad154-ee9f-4f86-b9d5-7d31e250911f" />


- หลังปรับ Padding ให้เหมาะสม
<img width="854" height="623" alt="image" src="https://github.com/user-attachments/assets/9e9d7033-2cb4-4441-bbf8-6dc161bee1be" />

