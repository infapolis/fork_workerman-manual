# ความต้องการของสภาพแวดล้อม

## ผู้ใช้ Windows
ตั้งแต่เวอร์ชัน 3.5.3 เป็นต้นไป Workerman สามารถสนับสนุนระบบปฏิบัติการ Linux และ Windows พร้อมกันได้

1. ต้องใช้ PHP >= 5.4 และตั้งค่าตัวแปรสภาพแวดล้อมของ PHP ไว้ให้พร้อมใช้งาน
2. Workerman เวอร์ชันของ Windows ไม่ขึ้นอยู่กับส่วนขยายใด ๆ
3. การติดตั้งและข้อจำกัดในการใช้งานสามารถดูได้จาก [ที่นี่](https://www.workerman.net/windows)
4. เนื่องจาก Workerman มีข้อจำกัดในการใช้งานในระบบปฏิบัติการ Windows ดังนั้นแนะนำให้ใช้งานระบบปฏิบัติการ Linux เป็นอย่างยิ่งเมื่ออยู่ในสภาพแวดล้อมที่เป็นทางการ และระบบปฏิบัติการ Windows เหมาะสำหรับการพัฒนาเท่านั้น

``` ==== หน้าเว็บนี้นี้เห็นได้เฉพาะผู้ใช้ระบบปฏิบัติการ Linux เท่านั้น ผู้ใช้ระบบปฏิบัติการ Windows โปรดละเว้น ====```

## ผู้ใช้ Linux (รวมถึง Mac OS)
ผู้ใช้ระบบปฏิบัติการ Linux สามารถใช้ Workerman เวอร์ชัน Linux เท่านั้น

1. ติดตั้ง PHP >= 5.4 และติดตั้งส่วนขยาย pcntl และ posix
2. แนะนำให้ติดตั้งส่วนขยาย event แต่ไม่จำเป็นต้องทำ (โปรดทราบว่าส่วนขยาย event ต้องใช้ PHP >= 5.4)

### สแควริปต์ตรวจสอบสภาพแวดล้อมของ Linux
ผู้ใช้ระบบปฏิบัติการ Linux สามารถรันสคริปต์ตรวจสอบสภาพแวดล้อมดังต่อไปนี้เพื่อตรวจสอบว่าสภาพแวดล้อมบนเครื่องสามารถทำงานได้ตามที่ Workerman ต้องการหรือไม่

```curl -Ss https://www.workerman.net/check | php```

หากสคริปต์แสดงข้อความ "ok" ทั้งหมด แปลว่าสภาพแวดล้อมสำหรับการทำงานของ Workerman พร้อมใช้งาน

(โปรดทราบ: สคริปต์ตรวจสอบไม่ได้ตรวจสอบส่วนขยาย event หากมีการเชื่อมต่อพร้อมกันมากกว่า 1024 โปรดแนะนำให้ติดตั้งส่วนขยาย event สำหรับวิธีการติดตั้ง โปรดดูในส่วนถัดไป)

## คำอธิบายอย่างละเอียด

### เกี่ยวกับ PHP-CLI

Workerman ทำงานบนโหมด [PHP command line (PHP-CLI)](https://php.net/manual/zh/features.commandline.php) PHP-CLI แยกจาก PHP-FPM หรือ Apache MOD-PHP และไม่มีความขัดแย้งหรือล้มเหลวซึ่งกันและกัน และทำงานอย่างอิสระ

### เกี่ยวกับส่วนขยายที่ Workerman ต้องการ

1. [ส่วนขยาย pcntl](https://cn2.php.net/manual/zh/book.pcntl.php)
   ส่วนขยาย pcntl เป็นส่วนขยายที่สำคัญสำหรับการควบคุมกระบวนการในระบบปฏิบัติการ Linux และ Workerman ใช้คุณสมบัติต่าง ๆ เช่น [การสร้างกระบวนการ](https://cn2.php.net/manual/zh/function.pcntl-fork.php) ควบคุมสัญญาณ (https://cn2.php.net/manual/zh/function.pcntl-signal.php) การตั้งนาฬิกา (https://cn2.php.net/manual/zh/function.pcntl-alarm.php) การทำงานของกระบวนการสถานะการดูแล (https://cn2.php.net/manual/zh/function.pcntl-waitpid.php) ข้อมูลเฉพาะการดูแลดังต่อไปนี้ ส่วนขยายนี้ไม่รองรับบนระบบปฏิบัติการ Windows

2. [ส่วนขยาย posix](https://cn2.php.net/manual/zh/book.posix.php)
   ส่วนขยาย posix ทำให้ PHP สามารถเรียกใช้ส่วนต่อประธานจากระบบผ่านทาง [มาตรฐาน POSIX](https://baike.baidu.com/view/209573.htm) Workerman ใช้ส่วนต่อประธานในการทำงานเช่นเดียวกันกับการทำกระบวนการที่ปกติควบการของกลุ่มผู้ใช้งาน Workerman ส่วนขยายนี้ไม่รองรับบนระบบปฏิบัติการ Windows

3. [ส่วนขยาย Event](https://php.net/manual/zh/book.event.php) หรือ [ส่วนขยาย libevent](https://cn2.php.net/manual/en/book.libevent.php)
   ส่วนขยาย Event ทำให้ PHP สามารถใช้งานระบบจัดการเหตุการณ์ขั้นสูงเช่น [Epoll](https://baike.baidu.com/view/1385104.htm) และ Kqueue ทำให้ได้ใช้ระบบขั้นสูง เมื่อมีการเชื่อมต่อกับจำนวนมาก ส่วนมากจะเป็นส่วนสำคัญในการใช้งานที่เกี่ยวข้องกับการเชื่อมต่อทางระยะยาวที่มีจำนวนมาก ส่วนขยาย libevent (หรือ ส่วนขยาย event) ไม่จำเป็นต้องใช้งาน หากทำการตั้งค่าไว้แล้วการใช้งานส่วนขยายงี้โดยปริยายจะนำไปใช้งานผ่านกับการจัดกำลังประมวลผลที่ทำมาจาก PHP

# วิธีการติดตั้งส่วนขยาย

โปรดดูที่มาตรฐานการติดตั้งส่วนขยายในส่วนเสริม [การติดตั้งส่วนขยาย](../appendices/install-extension.md)