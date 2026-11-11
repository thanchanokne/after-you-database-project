================================================================================
  AFTER YOU DESSERT CAFE - RELATIONAL DATABASE SYSTEM DESIGN
  Enterprise Relational Database Architecture & System Design Documentation
================================================================================

[PROJECT OVERVIEW]
Project Title : After You Dessert Cafe Database Architecture & System Design
Domain        : Food & Beverage (F&B), POS, Table Management & Recipe Inventory
Course        : ITDS221 Database Management System, Mahidol University
Methodology   : Conceptual Modeling (Chen EER) -> 8-Step Transform Mapping -> 3NF Normalization

บทสรุปโครงการ:
การวิเคราะห์และออกแบบสถาปัตยกรรมฐานข้อมูลเชิงสัมพันธ์ (Relational Database Design)
แบบครบวงจรสำหรับธุรกิจร้านขนมหวานหลายสาขา โดยใช้กรณีศึกษาจริงของ After You 
Dessert Cafe (มากกว่า 44 สาขาทั่วประเทศ) เพื่อสร้างโครงสร้างข้อมูลที่รองรับกระบวนการ
ทำงานจริงของหน้าร้าน ได้แก่ ระบบจัดการสาขาและสถานะโต๊ะ, ระบบขายหน้าร้าน (POS) 
แบบ Pay-First และระบบตัดสต็อกวัตถุดิบตามสูตรเมนู (BOM) พร้อมการทำ Normalization
จนถึงระดับ Third Normal Form (3NF)

--------------------------------------------------------------------------------
1. SCOPE & BUSINESS PROCESSES (ขอบเขตและระบบงานที่ออกแบบ)
--------------------------------------------------------------------------------

1. ระบบจัดการสาขาและโต๊ะ (Branch & Table Management):
   - จัดเก็บข้อมูลสาขา ที่อยู่ และหมายเลขโทรศัพท์ติดต่อ (รองรับหลายเบอร์ต่อสาขา)
   - จัดการสถานะโต๊ะรองรับลูกค้าแบบ Real-time: "ว่าง", "กำลังใช้งาน", "กำลังทำความสะอาด"
   - จัดสรรและควบคุมการปฏิบัติงานของพนักงานประจำแต่ละสาขา

2. ระบบสั่งซื้อหน้าร้าน (In-Store POS & Pay-First Ordering):
   - รองรับกระบวนการสั่งซื้อแบบ Pay First (ชำระเงินก่อนรับบริการ)
   - จำแนกบทบาทหน้าที่เฉพาะของพนักงานผ่าน Specialization Hierarchy:
     * Cashier : บันทึกออเดอร์, ประจำเคาน์เตอร์ตามกะ (cashierCounterNo),
                 คุมลิ้นชักเงิน (cashDrawerID), และเป้าหมายยอดขายประจำวัน (DailyTarget)
     * Waiter  : ดูแลโซนให้บริการ (ServiceZone), โควตารองรับโต๊ะสูงสุด (maxTable),
                 และยอดทิป (TipAmount)
   - เชื่อมโยงคำสั่งซื้อ (Order) กับธุรกรรมการชำระเงิน (Payment) แบบ 1:1 Transaction

3. ระบบจัดการเมนูและคลังวัตถุดิบ (Menu & Recipe Inventory Tracking):
   - บริหารจัดการรายการเมนูของหวาน/เครื่องดื่ม และราคาจำหน่าย
   - จัดเก็บข้อมูลคลังวัตถุดิบเพื่อรองรับการตรวจรับและเบิกใช้ตามหลัก FIFO
   - ออกแบบโครงสร้างสูตรเมนู (Recipe / Bill of Materials) ผ่านความสัมพันธ์แบบ M:N
     ระหว่าง Menu Item กับ Ingredient เพื่อตัดยอดสต็อกอัตโนมัติเมื่อเกิดคำสั่งซื้อ

--------------------------------------------------------------------------------
2. DATA MODELING & ARCHITECTURE (การออกแบบแบบจำลองข้อมูล)
--------------------------------------------------------------------------------

A. Conceptual Data Modeling (Peter Chen EER Diagram):
   - Disjoint Specialization ('d') : Employee (Superclass) -> Cashier, Waiter (Subclasses)
     โดยใช้ EmployeeType เป็นตัวจำแนก
   - Multi-Valued Attribute        : BranchTelephone บน Entity Branch
   - Composite Attribute           : EmpName ประกอบด้วย FName และ LName
   - Derived Attributes            : TotalAmount (Order), PaymentAmount (Payment), TipAmount (Waiter)
   - Structural Constraints        : กำหนด Cardinality Ratio และ Participation Constraints
                                     (min, max) ครบถ้วนทุกความสัมพันธ์

B. Logical Database Design (8-Step Transform Mapping):
   - Step 1 (Regular Entities)       : สร้างตาราง Employee, Branch, Table, Order,
                                       MenuItem, Ingredient
   - Step 2 (Weak Entities)          : ไม่มี Weak Entity ในระบบ
   - Step 3 (1:1 Relationships)      : Order <-> Payment (นำ OrderID ไปเป็น FK ใน Payment)
   - Step 4 (1:N Relationships)      : Branch -> Table, Branch -> Employee,
                                       Cashier -> Order, Table -> Order
   - Step 5 (M:N Relationships)      : Waiter_serve_Table, Order_has_MenuItem,
                                       MenuItem_uses_Igd
   - Step 6 (Multi-valued Attribute) : แตก BranchTelephone เป็นตารางแยก
   - Step 7 (n-ary Relationships)    : ไม่มีในระบบ
   - Step 8 (Specialization/Gen.)    : ใช้แนวทาง Option 8A แยกตาราง Superclass และ
                                       Subclasses (Employee, Waiter, Cashier)

--------------------------------------------------------------------------------
3. RELATIONAL SCHEMA SPECIFICATION (สรุปโครงสร้างตารางและความสัมพันธ์)
--------------------------------------------------------------------------------

1. Employee
   - PK : EmpID
   - FK : BranchID -> Branch(BranchID)
   - Attributes: EmpID, FName, LName, Phone, Salary, HireDate, BranchID

2. Waiter (Subclass of Employee)
   - PK, FK : EmpID -> Employee(EmpID)
   - Attributes: EmpID, maxTable, ServiceZone

3. Cashier (Subclass of Employee)
   - PK : (EmpID, cashierCounterNo)  [Composite Primary Key]
   - FK : EmpID -> Employee(EmpID)
   - Attributes: EmpID, cashierCounterNo, DailyTarget, cashDrawerID

4. Branch
   - PK : BranchID
   - Attributes: BranchID, BranchAddress

5. BranchTelephone (Multi-Valued Table of Branch)
   - PK, FK : (BranchID, BranchTelephone)
   - Attributes: BranchID, BranchTelephone

6. Table
   - PK : TableID
   - FK : BranchID -> Branch(BranchID)
   - Attributes: TableID, TableStatus, TableCapacity, BranchID

7. Waiter_serve_Table (M:N Associative Table)
   - PK, FK : (TableID, EmpID)
   - Attributes: TableID, EmpID

8. Order
   - PK : OrderID
   - FK : (cashierCounterNo, EmpID) -> Cashier, TableID -> Table(TableID)
   - Attributes: OrderID, OrderDate, cashierCounterNo, EmpID, TableID

9. Payment (1:1 with Order)
   - PK : PaymentID
   - FK : OrderID -> Order(OrderID)
   - Attributes: PaymentID, PaymentMethod, PaymentDate, OrderID

10. MenuItem
    - PK : MenuID
    - Attributes: MenuID, MenuName, Price

11. Order_has_MenuItem (M:N Associative Table)
    - PK, FK : (OrderID, MenuID)
    - Attributes: OrderID, MenuID

12. Ingredient
    - PK : IngredientID
    - Attributes: IngredientID, Description, IgdName

13. MenuItem_uses_Igd (M:N Recipe / BOM Table)
    - PK, FK : (MenuID, IngredientID)
    - Attributes: MenuID, IngredientID

--------------------------------------------------------------------------------
4. NORMALIZATION AUDIT: 3NF (การตรวจสอบความถูกต้องระดับนอร์มัลฟอร์มที่ 3)
--------------------------------------------------------------------------------

ทุกตารางความสัมพันธ์ผ่านการวิเคราะห์ Functional Dependencies (FD) จนถึงระดับ 3NF
โดยมีกรณีศึกษาการตรวจสอบโครงสร้างตาราง Cashier ดังนี้:

- First Normal Form (1NF):
  * ทุกคอลัมน์จัดเก็บค่าเดี่ยว (Atomic Value) ปราศจาก Repeating Groups หรือ Nested List
  * กำหนด Composite Primary Key ร่วมอย่างชัดเจน: (EmpID, cashierCounterNo)

- Second Normal Form (2NF):
  * ผ่านเกณฑ์ 1NF
  * ขจัด Partial Functional Dependency โดย Non-key attributes ทั้งหมด
    (DailyTarget, cashDrawerID) พึ่งพาคีย์หลักร่วมทั้งสองตัวแบบ Full Functional Dependency:
    - DailyTarget ขึ้นกับทั้งรหัสพนักงานและเคาน์เตอร์กะนั้นๆ
    - cashDrawerID ขึ้นกับทั้งรหัสพนักงานและลิ้นชักประจำเคาน์เตอร์นั้นๆ
    - ไม่มี Non-key attribute ใดพึ่งพาเพียงส่วนหนึ่งส่วนใดของคีย์หลักร่วม

- Third Normal Form (3NF):
  * ผ่านเกณฑ์ 2NF
  * ขจัด Transitive Dependency: ไม่มี Non-key attribute ใดกำหนดค่าให้แก่กันเอง
    (No Non-key -> Non-key dependencies) แอตทริบิวต์ทั้งหมดขึ้นตรงต่อ Primary Key เท่านั้น

--------------------------------------------------------------------------------
5. REPOSITORY STRUCTURE (โครงสร้างไฟล์ในโปรเจกต์)
--------------------------------------------------------------------------------

├── README.md                                    # รายละเอียดโปรเจกต์ฉบับหลักสำหรับ GitHub
├── Readme.txt                                   # เอกสารสรุปเนื้อหาทางเทคนิค (Plain-text)
├── Report.pdf                                   # รายงานวิชาการฉบับสมบูรณ์ (จำนวน 29 หน้า)
└── Organized_By_Section/                        # ไดอะแกรมและแผนภาพสถาปัตยกรรม
    ├── Section4_ER_EER_Diagram_Chen.jpg         # แผนภาพ Peter Chen ER/EER Diagram ฉบับเต็ม
    ├── Section5_Relation_Schema_Full.png        # แผนผัง Relational Schema รวมทุกตาราง
    ├── Section5_Step1_Regular_Entity.jpg        # Transform Mapping Step 1: Regular Entity
    ├── Section5_Step3_1_to_1_Relationship.jpg   # Transform Mapping Step 3: 1:1 Relationship
    ├── Section5_Step4_1_to_N_Relationship.jpg   # Transform Mapping Step 4: 1:N Relationship
    ├── Section5_Step5_M_to_N_Relationship.jpg   # Transform Mapping Step 5: M:N Relationship
    ├── Section5_Step6_Multi_Valued_Attribute.jpg # Transform Mapping Step 6: Multi-Valued
    ├── Section5_Step8_Specialization_Generalization.jpg # Transform Mapping Step 8: Hierarchy
    └── Section6_Third_Normal_Form_3NF.jpg       # แผนผังแสดงการพิสูจน์คุณสมบัติระดับ 3NF

--------------------------------------------------------------------------------
6. TECHNICAL COMPETENCIES DEMONSTRATED (ทักษะทางเทคนิคที่แสดงในผลงานนี้)
--------------------------------------------------------------------------------
- Database Engineering & Architecture
- Conceptual & Logical Data Modeling (Peter Chen EER, Relational Schema)
- 8-Step Transform Mapping Methodology
- Database Normalization (1NF, 2NF, 3NF Functional Dependency Analysis)
- Business Domain & System Analysis (POS, F&B Operations, Inventory BOM)
- Referential Integrity & Constraint Enforcement
================================================================================
