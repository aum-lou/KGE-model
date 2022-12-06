# KGE-model
Knowledge Graph Embeddings 3 โมเดล ได้แก่ TransE, ComplEx และ RotatE ในการหาความสัมพันธ์ของคำศัพท์ระหว่างชุดข้อมูลกราฟความรู้ใน Conceptnet กับ เป้าหมายพัฒนาอย่างยั่งยืน (SDGs: Sustainable Development Goals)

# วิธีการทำ
การหาความสัมพันธ์ของคำศัพท์ในงานนี้ ผู้ศึกษาใช้ชุดข้อมูลจาก ConceptNet โดยเลือกข้อมูลบางส่วนเท่านั้น และใช้ชุดข้อมูลกราฟความรู้เกี่ยวกับ Sustainable Development Goals ที่ผู้ศึกษาได้สร้างขึ้นมาเอง เพื่อนำมาใช้สำหรับสร้างชุดข้อมูลใหม่ที่มีคำศัพท์เกี่ยวข้องกันจากทั้ง 2 ชุดข้อมูลด้วยกระบวนการ Natural Language Processing แล้วใช้เป็นชุดข้อมูลสำหรับศึกษาความความสัมพันธ์สำหรับการเรียนรู้ในโมเดล Knowledge Graph Embeddings เพื่อใช้ในการทำนายความพันธ์ในชุดข้อมูลและคำนวณคะแนนความสัมพันธ์ โดยที่ใช้ฐานข้อมูลสำหรับเก็บกราฟความรู้ใน Neo4j บน Docker

##1. การรัน Neo4j บน Docker
    เริ่มต้นที่การติดตั้ง  โดยระบบจัดการฐานข้อมูลเชิงกราฟ Neo4j นั้นมีหลายรูปแบบให้เลือกใช้ ทั้งแบบที่เก็บค่าบริการสำหรับการนำไปใช้งานในเชิงพาณิชย์ และแบบที่ไม่เก็บค่าบริการ เพื่อนำไปใช้งานในเชิงการศึกษาและวิจัย สำหรับโปรแกรม Neo4j สามารถดาวน์โหลดได้ที่ Neo4j Dowload Center ซึ่งมีให้ทั้งระบบปฎิบัติการ Windows และ Mac OS X  ผู้ใช้สามารถเลือกดาวน์โหลดทั้งแบบ Neo4j Desktop หรือ Neo4j sever ในที่นี้ผู้ศึกษาเลือกดาวน์โหลด Neo4j Community Sever ไม่จำกัดระยะเวลาในการใช้งาน ขณะที่  Desktop version จะให้ใช้ฟรี 30 วัน แล้วก็เลือกตาม OS ที่ใช้งานอยู่ได้เลย
    
    เมื่อโหลดเสร็จเรียบร้อยแล้ว ก็ให้แตกไฟล์และวางไฟล์ไว้ในตำแหน่งที่ต้องการ directory ระดับบนสุดจะเรียกว่า Neo4j_HOME
    
    ในที่นี้ จะนำไปไว้ที่ Desktop ในโฟล์เดอร์ชื่อ neo4j-community-3.3.2 หลังจากนั้นให้ทำการเปิด command prompt แบบ Admin (เปิดหน้า terminal/shell ขึ้นมาเพื่อใช้ Neo4j) ให้ไปที่ตำแหน่งไฟล์ของ neo4j ที่แตกไว้ให้ทำการไปยังโฟล์เดอร์ Bin    

C:\Users\Desktop\neo4j-community-3.3.2\bin
ให้ทำการเรียกไฟล์    neo4j.bat  ทำการติดตั้ง service  ดังนี้
C:\Users\Desktop\neo4j-community-3.3.2\bin\ neo4j.bat install-service
หลังจากนั้นให้ทำการรัน service
C:\Users\Desktop\neo4j-community-3.3.2\bin\ neo4j.bat install-service

    เมื่อทำการติดตั้งเรียบร้อยแล้วจะเห็นคำว่า Neo4j is ready อยู่แสดงว่าใช้งานได้แล้ว การเรียกใช้งานโปรแกรมสามารถเรียกผ่านเบราว์เซอร์จาก Port 7474 โดยพิมพ์ http://localhost:7474/browser/ ให้ทำการ login ด้วย username : neo4j และ password : neo4j  ในครั้งแรก หลังจากนั้นทำการเปลี่ยนรหัสผ่าน
    
    Expose Ports ตามค่าเริ่มต้น Docker image จะแสดง Ports 3 Ports สำหรับการ Remote Access ได้แก่
7474 for HTTP (สำหรับเข้าสู่หน้า Neo4j Browser แบบ HTTP)
7473 for HTTPS (สำหรับเข้าสู่หน้า Neo4j Browser แบบ HTTPS)
7687 for Bolt (สำหรับ Neo4j Database Connection)

    โดยเวลาติดตั้ง Neo4j จะต้อง Expose Port อย่างน้อย 2 Ports จากทั้งหมด 3 Ports เพื่อ Remote Access คือ 7474 และ 7687 เนื่องจากไม่ได้เปิด HTTPS ถ้าจะทำ HTTPS ต้องทำเพิ่มเติมเอง ซึ่งตาม Default แล้ว Docker Image นี้ไม่ได้มี HTTPS มาให้
กำหนด Env Config 
กำหนด Default Username/Password สำหรับ Login เข้า Database
--env NEO4J_AUTH=<DATA_BASE_USERNAME>/<DATA_BASE_PASSWORD>

ถ้าไม่ต้องการให้มี Username/Password ก็ให้กำหนดเป็น
--env NEO4J_AUTH=none

กำหนด Configuration ต่าง ๆ สำหรับ Neo4j Browser (เป็น Tool สำหรับ Query Data บน Browser คล้าย ๆ phpMyAdmin)
--env NEO4J_dbms_connector_https_advertised__address="<SERVER_IP>:7473"
--env NEO4J_dbms_connector_http_advertised__address="<SERVER_IP>:7474"
--env NEO4J_dbms_connector_bolt_advertised__address="<SERVER_IP>:7687"

    ถ้าไม่กำหนด เวลาเปิด Neo4j Browser ด้วย IP อื่น ๆ จะทำให้ ไม่สามารถ Login หรือ Query data จาก Database ได้ เนื่องจาก Connection จะ Default เป็น localhost
Run Docker with Neo4j เป็นการดึงและเรียกใช้ Neo4j ภายใน Docker container โดยใช้หนึ่งใน images ที่ให้มานั้น ซึ่งใช้เวลาไม่กี่ขั้นตอน ซึ่งจะต้องรันคำสั่ง docker run ด้วย neo4j image และระบุตัวเลือกหรือเวอร์ชันที่ต้องการพร้อมกับสิ่งนั้น โดยให้ดูที่ตัวเลือกไม่กี่ตัวที่ใช้ได้กับคำสั่ง docker run


