เมื่อฉันปิดที่ทำงานแล้ว Workerman ก็ปิดตัวเองไปทำไม?

Workerman มีโหมดการเริ่มต้น 2 แบบ คือ debug mode และ daemon mode

การรัน ```php xxx.php start``` คือการเข้าสู่ debug mode ที่ใช้สำหรับการแก้ไขปัญหาและเมื่อที่ทำงานปิด Workerman ก็จะปิดตามไปด้วย

การรัน ```php xxx.php start -d``` คือการเข้าสู่ daemon mode โหมดที่ทำให้ Workerman ไม่รับผลกระทบจากการปิดที่ทำงาน

หากต้องการให้ Workerman ไม่รับผลกระทบจากการปิดที่ทำงาน สามารถใช้งานดาวน์โหมดเริ่มต้นได้