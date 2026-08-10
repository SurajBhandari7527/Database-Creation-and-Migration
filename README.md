## **Hospital Database Creation and Data Migration**

#### **Problem Statement: Hospital Database Migration & Automation**

**Background:**
Our hospital has been maintaining all its records—including patient details, doctor rosters, appointments, prescriptions, lab reports, and billing—using Excel file. As our operations have scaled, this method has become inefficient, error-prone, and difficult to manage. We are now transitioning to a relational database system to improve data integrity, performance, and scalability.

---

#### **Problem Description**

We want you to build a robust and well-structured **relational database system** that captures all core functionalities of our hospital. From the data from our current Excel-based system take guide what tables to develop and this data should also be migrated into this new database, ensuring that data integrity and consistency.

The database should also support business rules that govern how appointments are managed, how doctors can access patient data, and how department-wise revenue reports can be generated.

---

#### **Problems You Need to Solve with Your Database Design**

1. **Lack of Unique Identifiers**
   * Currently, we have no guaranteed unique IDs for patients, doctors, departments, or appointments.
   * Introduce something that ensures uniqueness.

2. **Disconnected Relationships**
   * In Excel, appointments are listed, but there is no enforceable link to valid patients or doctors.
   * Implement measures to maintain referential integrity between patients, doctors, departments, and appointments.

3. **Invalid or Ambiguous Data Entries**
   * For example, gender values like "X", appointment statuses like "On Hold", and inconsistent date formats.
   * Allowed values (Gender must be 'M', 'F', or 'O'; Status must be 'Scheduled', 'Completed', or 'Cancelled').

4. **Unregulated Scheduling**
   * Doctors are occasionally double-booked, and appointments are being scheduled in the past.
   * Design and implement measures to automatically prevent invalid appointment entries during insertion.

5. **Open Access to Sensitive Patient Information**
   * All doctors currently see all data, regardless of their role or department.
   * Create limitations that allow access to data based on credentials and roles provided by the firm only senior doctors can view all patients in their department, else doctors can only see details(medication, appointment, reports) of their respective patients.

6. **Disconnected Reporting**
   * There's no way to generate billing or departmental summaries across the hospital.
   * Implement a way that generates **monthly revenue reports** by department.
     
Here is a well-structured, professional draft for your **Solution** section in Markdown. You can directly copy and paste this into your project documentation or GitHub `README.md`.

---

# **Solution & Technical Implementation**

### **1. Solution Overview**
To address the operational challenges, data redundancy, and security risks associated with the spreadsheet-based system, a normalized **Relational Database Management System (RDBMS)** was designed and implemented using **MySQL**. 

The solution transitions the flat, unorganized Excel sheets into a multi-table relational schema with strictly enforced constraints, role-based access security, automated scheduling checks, and analytical reporting views.

---

### **2. Source Data Analysis (Legacy Excel File)**
An initial audit of the legacy Excel workbook was conducted to identify missing relationships, attribute dependencies, and repeating groups. 

#### **Legacy Spreadsheet Attributes:**
Below is the attribute extraction from the legacy system used to derive the baseline entity set:

<img width="2240" height="1260" alt="Departments DepartmentID Departments Name Doctors DoctorID Doctors Name Doctors Specialization Doctors Role Doctors DepartmentID" src="https://github.com/user-attachments/assets/1f1f9b85-3f48-416d-add4-ca3f0c95df92" />


---

### **3. Entity-Relationship (ER) Design**
Based on the identified entities and business requirements, a conceptual and logical **Entity-Relationship (ER) Model** was engineered. 

#### **Key Architecture Features:**
* **Normalization:** Normalized to **3rd Normal Form (3NF)** to eliminate data redundancy and insertion/deletion anomalies.
* **Referential Integrity:** Primary key (PK) and foreign key (FK) constraints established across all operational entities (`Patients`, `Doctors`, `Departments`, `Appointments`, `Prescriptions`, `LabReports`, and `Billing`).
* **Relational Mapping:** Clean 1:N (One-to-Many) and N:M (Many-to-Many) mappings implemented for doctors, appointments, and medical records.

📄 **View the Full Architectural Diagram:**  
[📥 Download / View ER Diagram (PDF)]
[ER (1).pdf](https://github.com/user-attachments/files/30904996/ER.1.pdf)

