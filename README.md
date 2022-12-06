# KGE-model
Knowledge Graph Embeddings 3 โมเดล ได้แก่ TransE, ComplEx และ RotatE ในการหาความสัมพันธ์ของคำศัพท์ระหว่างชุดข้อมูลกราฟความรู้ใน Conceptnet กับ เป้าหมายพัฒนาอย่างยั่งยืน (SDGs: Sustainable Development Goals)

# วิธีการทำ
การหาความสัมพันธ์ของคำศัพท์ในงานนี้ ผู้ศึกษาใช้ชุดข้อมูลจาก ConceptNet โดยเลือกข้อมูลบางส่วนเท่านั้น และใช้ชุดข้อมูลกราฟความรู้เกี่ยวกับ Sustainable Development Goals ที่ผู้ศึกษาได้สร้างขึ้นมาเอง เพื่อนำมาใช้สำหรับสร้างชุดข้อมูลใหม่ที่มีคำศัพท์เกี่ยวข้องกันจากทั้ง 2 ชุดข้อมูลด้วยกระบวนการ Natural Language Processing แล้วใช้เป็นชุดข้อมูลสำหรับศึกษาความความสัมพันธ์สำหรับการเรียนรู้ในโมเดล Knowledge Graph Embeddings เพื่อใช้ในการทำนายความพันธ์ในชุดข้อมูลและคำนวณคะแนนความสัมพันธ์ โดยที่ใช้ฐานข้อมูลสำหรับเก็บกราฟความรู้ใน Neo4j บน Docker

## 1. การรัน Neo4j บน Docker
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

## 2. การเตรียมข้อมูลด้วย NLP
โหลดข้อมูล 2 ชุดคือ “assertions.csv” ซึ่งเป็นไฟล์ข้อมูลทั้งหมดของตัว ConceptNet ประกอบด้วยคอลัมน์ 

     “The URI of the whole edge” (URI) , 
     “The relation expressed by the edge” (relation), 
     “The node at the start of the edge” (start_node), 
     “The node at the end of the edge” (end _node), 
     “A JSON structure of additional information” 
     
ซึ่งทำการเตรียมข้อมูลและเลือกข้อมูลมาใช้เพียง 3 คอลัมน์ ได้แก่ “start_node”, “relation”  และ “end _node” โดยทำความสะอาดข้อมูลโดยเลือกเฉพาะภาษาอังกฤษ และตัดอักขระที่ไม่ต้องการ เช่น / œ é à æ เป็นต้น รวมถึงเลือก relation ที่เหมาะสมได้แก่ 

    “RelatedTo”, “ HasA”, “HasProperty”, “IsA”, “PartOf”, “SimilarTo”, “UsedFor”,  “InstanceOf” และ “DefinedAs” โดยเลือกข้อมูลมาบางส่วน จึงมีข้อมูลทั้งหมด 38,411 แถว

และชุดข้อมูล “sdg.csv” เป็นชุดข้อมูลกราฟความรู้เกี่ยวกับ Sustainable Development Goals (SDGs) ที่ได้จัดทำผ่าน CD-OAM ประกอบด้วยคอลัมน์ “subject” (start_node), “predicated” (relation) และ “object” (end _node) ซึ่งมีข้อมูลทั้งหมด 2,978 แถว 

ผู้ศึกษาต้องการสร้างชุดข้อมูลที่คำศัพท์ในชุดข้อมูล “ConceptNet” เกี่ยวข้องเป้าหมายทั้ง 17 ข้อของชุดข้อมูล “SDGs” โดยเริ่มดังนี้

    - ติดตั้งแพ็กเกจ Pykeen
    - นำเข้าโมดูลหรือแพ็กเกจwได้จากไฟล์ requirements_NLP.txt 
    - ทำความสะอาดในรูปแบบที่ต้องการ เช่น “No_Poverty” เป็น “No Poverty” รวมถึงลบข้อมูลที่ไม่ต้องการ เช่น “Nd5f024d488114f5f9492e0469195152f”
    - ใช้ NLP ในการทำความสะอาดข้อมูลในรูปแบบที่ต้องการและตัดประโยคเป็นคำศัพท์ทีละคำในทั้ง 2 ชุดข้อมูล โดยมีการทำ word tokenize, ลบ stop words ภาษาอังกฤษ และการทำ PorterStemmer ซึ่งจะได้ข้อมูลคำศัพท์ในลักษณะรากศัพท์ เช่น “Manufacturing employment” เป็น “Manufacturing” และ “employment” 

    โดยสามารถดูโค้ดเพิ่มเติมใน NLP data.ipynb 
    
โดยผู้ศึกษาต้องการสร้างคลังคำศัพท์ในชุดข้อมูล “SDGs” ด้วยการรวมข้อมูลทั้งใน “start_node” และ “end _node” แล้วตัดคำศัพท์ที่ซ้ำออก จะได้คลังคำศัพท์ที่สมบูรณ์ทั้งหมด 382 คำ ต่อมาทำการ map คำศัพท์กับชุดข้อมูล“ConceptNet” ซึ่งได้ข้อมูลทั้งหมด 36,159 แถว 

ผู้ศึกษาสร้างชุดข้อมูลใหม่ขึ้นมา “dataset” คือการนำชุดข้อมูล “SDGs” ที่ผ่านการทำความสะอาดข้อมูลไม่รวมการทำ NLP และชุดข้อมูลที่ทำการ map และไม่ผ่านการทำ NLP เพื่อรับคำศัพท์ใหม่ๆที่มีความเกี่ยวข้องกัน โดยมารวมกันและตั้งชื่อคอลัมน์ให้เป็น “start_node”, “relation” และ “end _node” ซึ่งมีข้อมูลทั้งหมด 39,054 entities 37,456 triples  และ 17 relations

## 3. การหาความสัมพันธ์ของชุดข้อมูล
ผู้ศึกษาทำการศึกษาเกี่ยวกับความสัมพันธ์ชุดข้อมูลที่คำศัพท์ในชุดข้อมูล “ConceptNet” เกี่ยวข้องเป้าหมายในแต่ละข้อของชุดข้อมูล “SDGs” โดยผู้ศึกษาเลือกหัวข้อมูลในเป้าหมายที่ 3, 6, 11, 12 และ 13 โดยการสร้างคลังคำศัพท์ในเป้าหมายที่สนใจและทำการ map คำศัพท์กับชุดข้อมูล “ConceptNet” จะได้คำศัพท์ที่สัมพันธ์กัน และหาคำศัพท์ที่เกี่ยวข้อง
ด้วยการใช้ “FreqDist” จาก nltk

## 4. การโหลดชุดข้อมูลเข้าสู่ Neo4j 
หลังจากเชื่อม docker เข้ากับ Neo4j ผ่าน Neo4j  Desktop โดยสร้างโปรเจกต์และทำการเชื่อมต่อกับ Neo4j  Browser 
ขั้นต่อมาคือ การนำข้อมูลเข้าสู่ Neo4j โดยการใช้ภาษา Python และภาษา Cypher ในการสร้าง node และ relations โดยเริ่มต้นดังนี้

    - ติดตั้งแพ็กเกจ neo4j
    - นำเข้าโมดูลหรือแพ็กเกจของ neo4j
    - ตั้งค่าเพื่อเชื่อมกับตัวฐานข้อมูล neo4j-docker
    - ใช้คำสั่ง pandas ในการเรียกชุดข้อมูล “dataset” ซึ่งเป็นไฟล์ CSV 
    - ใช้คำสั่ง pandas ดึงข้อมูลทีละแถว และใช้คำสั่งภาษา Cypher สร้างคิวรี โดยใช้คำสั่ง CREATE  ในการสร้างโหนด และ MATCH เหมือนคำสั่ง SELECT  ในการดึงข้อมูล และใช้คำสั่ง CREATE เพื่อสร้าง relations ระหว่างโหนด 
    - ใช้คำสั่ง session ในการรันคิวรี เพื่อสร้าง GraphDatabase 

## 5. การสร้างโมเดล Knowledge Graph Embeddings
หาโมเดลที่มีผลลัพธ์ดีที่สุดเพื่อใช้ในการหาความสัมพันธ์ของข้อมูล ได้แก่ โมเดล TransE
โมเดล ComplEx  และโมเดล RotatE ด้วยการใช้ “Pipeline” ของ Pykeen สำหรับการทำโมเดลและวัดผลประสิทธิภาพโมเดลด้วยเมทริกซ์ได้แก่ “Hit@K”, “Mean Rank” และ “Mean Reciprocal Rank” โดยเริ่มต้นดังนี้

    - ติดตั้งแพ็กเกจ Pykeen
    - นำเข้าโมดูลหรือแพ็กเกจได้จากไฟล์ requirements_model.txt
    - เตรียมชุดข้อมูล โดยนำเข้าชุดข้อมูล “dataset” แล้วทำการแบ่งข้อมูลเป็น training, validation และ testing ด้วย “TriplesFactory” เป็น [.8, .1, .1] 
    - สร้าง Pipeline ของแต่ละโมเดล โดยผู้ศึกษากำหนดพารามิเตอร์ต่าง ๆ ดังนี้ โดยเรียกใช้พารามิเตอร์ได้แก่ training=training,  testing=testing, validation=validation, model=(TransE/ ComplEx/ RotatE),loss='softplus', model_kwargs=dict(embedding_dim=20),
    optimizer='Adam', optimizer_kwargs=dict(lr=0.01),training_kwargs=dict (num_epochs=60, use_tqdm_batch=False), valuation_kwargs=dict(batch_size =128,use_tqdm=False),evaluator='RankBasedEvaluator' และ random_seed=1,
    โดยทำการเปลี่ยนเฉพาะพารามิเตอร์โมเดล
    - ประเมินประสิทธิภาพแต่ละโมเดลด้วยเมทริกซ์ที่กำหนดไว้ และกำหนดให้ค่า k=10
    - บันทึกโมเดลที่มีค่าดีที่สุดในรูปแบบไฟล์ pkl
    
    โดยสามารถดูโค้ดเพิ่มเติมใน model KGE.ipynb 
    
## 6. คำนวณคะแนนความสัมพันธ์ระหว่างชุดข้อมูล
หลังจากทำการเทรนโมเดลเพื่อหาโมเดลที่เหมาะสม ให้เรียกใช้งานโมเดลนั้นเพื่อหาคำนวณคะแนนชุดข้อมูล testing โดยใช้ “predict.get_all_prediction_df” ของ Pykeen ซึ่งจะเลือกข้อมูลที่มีคะแนนสูงสุด 3 อันดับแรก จากการกำหนดค่า k=3 โดยจะเป็นการทำนายความสัมพันธ์เพิ่มในชุดข้อมูล




    

