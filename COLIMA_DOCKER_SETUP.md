# วิธีการติดตั้งและใช้งาน Docker ผ่าน Colima (แทน Docker Desktop)

เอกสารนี้อธิบายขั้นตอนการติดตั้ง Colima และ Docker command-line บน macOS เพื่อให้สามารถรัน container ได้โดยไม่ต้องติดตั้ง Docker Desktop

## ทำไมต้องใช้ Colima?

Colima เป็นเครื่องมือที่ช่วยสร้างและจัดการ Virtual Machine (VM) ที่มี Containerd และ containerd runtime ทำให้คุณสามารถใช้ `docker` command-line เพื่อสั่งงาน container ได้ ซึ่งเป็นทางเลือกที่มีน้ำหนักเบาและใช้ทรัพยากรน้อยกว่า Docker Desktop

---

## ขั้นตอนการติดตั้ง (Installation)

### 1. ติดตั้ง Homebrew

หากคุณยังไม่มี Homebrew ให้ติดตั้งก่อนโดยใช้คำสั่งนี้ใน Terminal:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 2. ติดตั้ง Colima และ Docker CLI

ใช้ Homebrew เพื่อติดตั้ง Colima และ command-line tool ของ Docker (ซึ่งจะไม่ได้ติดตั้ง Docker Desktop):

```bash
brew install colima
brew install docker
```

- `colima`: คือตัวจัดการ Virtual Machine ที่จะรัน Docker daemon
- `docker`: คือ Docker client (CLI) ที่เราใช้พิมพ์คำสั่ง `docker ...`

---

## ขั้นตอนการใช้งาน (Usage)

### 1. เริ่มต้นการทำงานของ Colima

ในการใช้งานครั้งแรก หรือหลังจากที่เครื่องถูกรีสตาร์ท คุณต้องสั่งให้ Colima สร้างและเปิด VM ขึ้นมาก่อน:

```bash
colima start
```

คำสั่งนี้จะดาวน์โหลด image ที่จำเป็นและตั้งค่า VM ให้พร้อมใช้งาน คุณต้องรันคำสั่งนี้เพียงครั้งเดียวเพื่อเริ่มต้น VM หลังจากนั้น VM จะทำงานอยู่เบื้องหลัง

### 2. ตรวจสอบว่า Docker พร้อมใช้งาน

หลังจาก `colima start` ทำงานเสร็จสิ้น ลองทดสอบว่า Docker CLI สามารถเชื่อมต่อกับ Docker daemon ที่รันใน Colima ได้หรือไม่:

```bash
docker ps
```

หากคำสั่งนี้ทำงานได้โดยไม่แสดงข้อผิดพลาด (แม้ว่าจะยังไม่มี container ใดๆ แสดงขึ้นมา) หมายความว่าการติดตั้งสำเร็จแล้ว

### 3. นำมาใช้กับโปรเจคนี้

เมื่อ Colima ทำงานแล้ว คุณสามารถใช้คำสั่ง `docker` หรือ `docker-compose` (ที่รวมอยู่ใน pnpm script) ได้ตามปกติ:

```bash
# อยู่ที่ root directory ของโปรเจค anyvet-microchip-api

# สั่งให้ Docker สร้างและรัน container ของฐานข้อมูล PostgreSQL
pnpm db:up

# ตรวจสอบว่า container ของโปรเจค (anyvet-microchip-api-db) กำลังทำงานอยู่
docker ps
```

จากนั้นคุณก็สามารถรันแอปพลิเคชันด้วยคำสั่ง `pnpm dev` ได้ตามปกติ

---

## การจัดการ Colima และ Container

สิ่งสำคัญคือต้องแยกให้ออกระหว่าง "การหยุด Container" และ "การหยุด Colima VM"

### 1. การหยุด Container (เช่น Database) - ใช้เมื่อพักการพัฒนา

ถ้าคุณต้องการหยุดแค่บาง service ที่รันอยู่ (เช่น หยุดฐานข้อมูล) แต่ยังต้องการให้ Docker daemon (Colima) พร้อมใช้งานอยู่เบื้องหลัง ให้ใช้คำสั่ง `docker` ตามปกติ

สำหรับโปรเจคนี้ มี script เตรียมไว้ให้แล้ว:

```bash
# หยุดและลบ container ของฐานข้อมูลที่เกี่ยวกับโปรเจคนี้
pnpm db:down
```

คำสั่งนี้เป็นทางลัดของ `docker compose down` ครับ **วิธีนี้เป็นวิธีที่แนะนำเมื่อต้องการหยุดพักการพัฒนาชั่วคราว** เพราะ Colima VM ยังคงรันอยู่ ทำให้การสั่ง `pnpm db:up` ครั้งต่อไปจะทำงานได้รวดเร็วมาก

### 2. การหยุด Colima VM ทั้งหมด - ใช้เมื่อเลิกใช้งาน Docker

ถ้าคุณเลิกใช้งาน Docker สำหรับวันนั้นแล้ว และต้องการคืนทรัพยากร (RAM, CPU) ทั้งหมดที่ Colima VM ใช้ไปให้กับเครื่อง Mac ของคุณ ให้ใช้คำสั่ง:

```bash
# หยุดการทำงานของ Colima VM ทั้งหมด
colima stop
```

**ข้อควรจำ:** หลังจากรัน `colima stop` แล้ว ถ้าจะกลับมาใช้ Docker อีกครั้ง คุณจะต้องรัน `colima start` เพื่อเปิด VM ขึ้นมาใหม่ ซึ่งอาจจะใช้เวลาสักครู่

### คำสั่งที่มีประโยชน์อื่นๆ

- **ดูสถานะของ Colima:**
  ```bash
  colima status
  ```

- **ลบ Colima VM เพื่อเริ่มต้นใหม่ทั้งหมด:**
  (หากการตั้งค่ามีปัญหาและต้องการล้างทั้งหมด)
  ```bash
  colima delete
  ```
  หลังจากลบแล้ว คุณจะต้องรัน `colima start` ใหม่อีกครั้ง
