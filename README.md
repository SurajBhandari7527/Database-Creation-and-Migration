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
<img width="1263" height="877" alt="Screenshot 2026-08-10 213205" src="https://github.com/user-attachments/assets/9483f5fb-1df3-4916-910b-6b212f6f74dd" />


[📥 Download (PDF)]
[ER (1).pdf](https://github.com/user-attachments/files/30904996/ER.1.pdf)

Here is a detailed, technical documentation of your hospital database project. It maps each business problem to its corresponding architectural and code-level solution based on your SQL script.

---


## **3. Detailed Problem-to-Solution Mapping**

### **Problem 1: Lack of Unique Identifiers**
* **The Problem:** In the legacy Excel file, records lacked guaranteed unique primary identifiers, causing ambiguity across patient visits, appointments, and doctor assignments.
* **The Solution:** Every created table includes a surrogate primary key enforced with `AUTO_INCREMENT` and `PRIMARY KEY` constraints.

```sql
-- Example: Primary keys added across all tables
CREATE TABLE Patient (
    PatientID INT AUTO_INCREMENT PRIMARY KEY,
    ...
);

CREATE TABLE Department (
    DepartmentID INT AUTO_INCREMENT PRIMARY KEY,
    ...
);
```

---

### **Problem 2: Disconnected Relationships & Broken Referential Integrity**
* **The Problem:** Excel listed appointments without enforceable links to valid doctors, patients, or departments, allowing orphaned records and invalid assignments.
* **The Solution:** **Foreign Key Constraints (`FOREIGN KEY`)** were added to establish strict relational linkages and prevent orphaned data entries.

```sql
-- Example: Enforcing dependencies between tables
CREATE TABLE Doctor (
    ...
    DepartmentID INT NOT NULL, 
    FOREIGN KEY (DepartmentID) REFERENCES Department(DepartmentID)
);

CREATE TABLE Appointment (
    ...
    DoctorID INT,
    PatientID INT,
    FOREIGN KEY (DoctorID) REFERENCES Doctor(DoctorID),
    FOREIGN KEY (PatientID) REFERENCES Patient(PatientID)
);
```

---

### **Problem 3: Invalid/Ambiguous Data Entries & Inconsistent Formats**
* **The Problem:** Legacy data contained unstandardized values (e.g., non-standard gender designations, unformatted dates as strings, and missing foreign keys).
* **The Solution:** 
  1. **`ENUM` Types:** Used to constrain column domain values (e.g., `Gender ENUM('M','F','O')`, `Status ENUM('Scheduled','Completed','Cancelled')`, and `Paid ENUM('0','1')`).
  2. **`STR_TO_DATE()` Formatting:** Converts string dates from Excel into native SQL `DATE` (`%d-%m-%Y`) and `DATETIME` (`%d-%m-%Y %H:%i`) formats during data migration.
  3. **Sanitization Logic:** Migration queries filter out empty/invalid foreign keys using `WHERE` filters.

```sql
-- Migration Query Example: Cleaning dates and filtering empty values
INSERT INTO Appointment (AppointmentID, AppointmentTime, Status, DoctorID, PatientID)
SELECT 
    `Appointments.AppointmentID`,
    STR_TO_DATE(`Appointments.AppointmentTime`, '%d-%m-%Y %H:%i'),
    `Appointments.Status`,
    `Appointments.DoctorID`,
    `Appointments.PatientID` 
FROM hospital_data_10000_rows
WHERE `Appointments.DoctorID` <> '' AND `Appointments.PatientID` <> '';
```

---

### **Problem 4: Unregulated Scheduling (Past Dates & Double-Booking)**
* **The Problem:** Appointments were occasionally scheduled in the past or double-booked for doctors at the same time slot.
* **The Solution:** A `BEFORE INSERT` database trigger (`valid_AppoinmentTime`) was deployed to inspect new insertions before commit:
  * **Rule 1:** Rejects appointments where `AppointmentTime < NOW()`.
  * **Rule 2:** Checks if an appointment already exists at the requested time slot.
  * **Enforcement:** Uses `SIGNAL SQLSTATE '45000'` to raise an error and abort the insertion.

```sql
DELIMITER $$
CREATE TRIGGER valid_AppoinmentTime
BEFORE INSERT ON Appointment
FOR EACH ROW
BEGIN
    -- Prevent appointments in the past
    IF NEW.AppointmentTime < NOW() THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'You cannot book an appointment in the past';
    END IF;

    -- Prevent double-booking slot collision
    IF EXISTS (SELECT 1 FROM Appointment WHERE AppointmentTime = NEW.AppointmentTime) THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'You cannot book an appointment at the same time as another appointment';
    END IF;
END $$
DELIMITER ;
```

---

### **Problem 5: Open Access to Sensitive Patient Data**
* **The Problem:** All medical personnel had unrestricted visibility into all patient details across the hospital, breaching data privacy protocols.
* **The Solution:** A secure Stored Procedure (`patient_info`) was created to act as a **Role-Based Access Control (RBAC)** abstraction layer:
  1. **Authentication:** Validates the doctor's credentials (`user_name` and `password`) against `doctor_credentials`.
  2. **General Doctors (`Role <> 'Senior'`):** Can view medical, billing, and lab details **only** for patients explicitly assigned to them (`a.doctorid = doct_id`).
  3. **Senior Doctors (`Role = 'Senior'`):** Can view medical records for **all patients across their assigned department**.

```sql
DELIMITER $$
CREATE PROCEDURE patient_info(IN doct_name VARCHAR(20), IN pass VARCHAR(20))
BEGIN
    DECLARE doctor_role VARCHAR(15);
    DECLARE doctor_depart VARCHAR(15);
    DECLARE doct_id INT;
    
    -- Authenticate user & retrieve role metadata
    SELECT Role, departmentid, doctor_id 
    INTO doctor_role, doctor_depart, doct_id 
    FROM doctor_credentials dc 
    LEFT JOIN doctor d ON dc.doctor_id = d.DoctorID 
    WHERE user_name = doct_name;
    
    IF EXISTS (SELECT 1 FROM doctor_credentials WHERE doctor_id = doct_id AND password = pass) THEN
        -- Non-Senior: Access limited to own patients only
        IF doctor_role <> 'Senior' THEN
            SELECT ... 
            FROM (SELECT * FROM Appointment WHERE doctorid = doct_id) a 
            LEFT JOIN doctor d ON a.doctorid = d.doctorid  
            LEFT JOIN patient p ON a.patientid = p.patientid 
            LEFT JOIN prescription pr ON a.AppointmentID = pr.appointmentid 
            LEFT JOIN bill b ON a.AppointmentID = b.AppointmentID 
            LEFT JOIN labreport lr ON a.appointmentid = lr.appointmentid;
        END IF;

        -- Senior: Access expanded to entire assigned department
        IF doctor_role = 'Senior' THEN
            SELECT ... 
            FROM Appointment a 
            RIGHT JOIN (SELECT * FROM doctor WHERE departmentid = doctor_depart) d ON a.doctorid = d.doctorid  
            LEFT JOIN patient p ON a.patientid = p.patientid 
            LEFT JOIN prescription pr ON a.AppointmentID = pr.appointmentid 
            LEFT JOIN bill b ON a.AppointmentID = b.AppointmentID 
            LEFT JOIN labreport lr ON a.appointmentid = lr.appointmentid;
        END IF;
    ELSE 
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Doctor credentials are incorrect';
    END IF;
END $$
DELIMITER ;
```

---

### **Problem 6: Disconnected Reporting & Departmental Financial Summaries**
* **The Problem:** The legacy setup lacked mechanisms to calculate departmental revenue metrics or aggregate monthly hospital billing summaries.
* **The Solution:** A Stored Procedure (`monthly_reports`) was implemented to perform complex multi-table joins (`Department` $\rightarrow$ `Doctor` $\rightarrow$ `Appointment` $\rightarrow$ `Bill`) and aggregate monthly revenue for a given fiscal year.

#### **Key Features:**
* Filters billing data by target year (`YEAR(billdate) = dateyear`).
* Aggregates revenue using `SUM(b.amount)`.
* Groups results by `DepartmentID` and `Month`.
* Uses a `CASE` statement to sort months chronologically (January to December) rather than alphabetically.

```sql
DELIMITER $$
CREATE PROCEDURE monthly_reports(IN dateyear INT)
BEGIN
    SELECT 
        dep.DepartmentID,
        b.month_name, 
        b.year, 
        SUM(b.amount) AS 'Revenue'
    FROM Department dep 
    LEFT JOIN Doctor d ON dep.DepartmentID = d.DepartmentID
    LEFT JOIN Appointment a ON a.DoctorID = d.DoctorID 
    RIGHT JOIN (
        SELECT *, MONTHNAME(billdate) AS month_name, dateyear AS year 
        FROM Bill 
        WHERE YEAR(billdate) = dateyear and paid=1
    ) b ON b.AppointmentID = a.AppointmentID
    GROUP BY d.DepartmentID, b.month_name
    ORDER BY dep.DepartmentID, 
        CASE b.month_name
            WHEN 'January'   THEN 1
            WHEN 'February'  THEN 2
            WHEN 'March'     THEN 3
            WHEN 'April'     THEN 4
            WHEN 'May'       THEN 5
            WHEN 'June'      THEN 6
            WHEN 'July'      THEN 7
            WHEN 'August'    THEN 8
            WHEN 'September' THEN 9
            WHEN 'October'   THEN 10
            WHEN 'November'  THEN 11
            WHEN 'December'  THEN 12
        END;
END $$
DELIMITER ;
```

---

## **4. Data Migration Summary Pipeline**

The migration script uses the following order to honor key dependency constraints:

```
[Legacy Excel Staging Table: hospital_data_10000_rows]
       │
       ├──► 1. Department (Independent Master Entity)
       │
       ├──► 2. Patient (Independent Master Entity)
       │
       ├──► 3. Doctor (Dependent on Department)
       │
       ├──► 4. Appointment (Dependent on Doctor & Patient)
       │
       ├───┬──► 5. Prescription (Dependent on Appointment)
       ├───┼──► 6. Bill (Dependent on Appointment)
       └───┴──► 7. LabReport (Dependent on Appointment)
```

---

## **5. Summary Table**

| # | Problem Description | Root Cause | Solution Implemented |
|---|---|---|---|
| **1** | Missing unique keys | Flat file structure | Added `AUTO_INCREMENT PRIMARY KEY` on all tables |
| **2** | Unlinked relationships | Spreadsheet structure | Enforced `FOREIGN KEY` constraints across dependent entities |
| **3** | Invalid / messy data | Manual data entry | Applied `ENUM` validations and `STR_TO_DATE()` functions during migration |
| **4** | Double-booking & past dates | Unregulated booking | Implemented `BEFORE INSERT` trigger (`valid_AppoinmentTime`) |
| **5** | Patient data leaks | Universal data access | Built RBAC procedure (`patient_info`) enforcing role/department boundaries |
| **6** | No revenue tracking | Disconnected files | Created dynamic financial report procedure (`monthly_reports`) |
