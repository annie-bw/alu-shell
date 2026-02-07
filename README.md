# FINAL HOSPITAL SYSTEM ENTITIES & CARDINALITIES

---

## 1. HOSPITAL
**Primary Key:** `hospital_id`

| Field | Type | Constraint | Notes |
|-------|------|-----------|-------|
| hospital_id | INTEGER | PK | Unique identifier |
| name | VARCHAR(255) | NOT NULL | Hospital name |
| address | VARCHAR(500) | NOT NULL | Street address |
| phone | VARCHAR(20) | NOT NULL | Contact number |
| email | VARCHAR(100) | NULLABLE | Email address |
| city | VARCHAR(100) | NULLABLE | City location |

---

## 2. DOCTOR
**Primary Key:** `doctor_id` | **Foreign Key:** `hospital_id`

| Field | Type | Constraint | Notes |
|-------|------|-----------|-------|
| doctor_id | INTEGER | PK | Unique identifier |
| hospital_id | INTEGER | FK → Hospital | Which hospital employs this doctor |
| first_name | VARCHAR(100) | NOT NULL | Doctor's first name |
| last_name | VARCHAR(100) | NOT NULL | Doctor's last name |
| specialty | VARCHAR(100) | NOT NULL | e.g., Cardiology, Neurology |
| license_no | VARCHAR(50) | NOT NULL, UNIQUE | Medical license number |
| email | VARCHAR(100) | NOT NULL | Email for contact/booking |
| phone | VARCHAR(20) | NULLABLE | Phone number |

---

## 3. PATIENT
**Primary Key:** `patient_id` | **Foreign Key:** `hospital_id`

| Field | Type | Constraint | Notes |
|-------|------|-----------|-------|
| patient_id | INTEGER | PK | Unique identifier |
| hospital_id | INTEGER | FK → Hospital | Patient registered at this hospital |
| first_name | VARCHAR(100) | NOT NULL | Patient's first name |
| last_name | VARCHAR(100) | NOT NULL | Patient's last name |
| email | VARCHAR(100) | NOT NULL, UNIQUE | Email for login |
| phone | VARCHAR(20) | NOT NULL | Contact number |
| date_of_birth | DATE | NOT NULL | For age calculation & records |
| mrn | VARCHAR(50) | NOT NULL, UNIQUE | Medical Record Number |
| address | VARCHAR(500) | NULLABLE | Home address |
| gender | ENUM('M','F','Other') | NULLABLE | Gender |

---

## 4. APPOINTMENT
**Primary Key:** `appointment_id` | **Foreign Keys:** `hospital_id`, `doctor_id`, `patient_id`

| Field | Type | Constraint | Notes |
|-------|------|-----------|-------|
| appointment_id | INTEGER | PK | Unique identifier |
| hospital_id | INTEGER | FK → Hospital | Which hospital hosts the appointment |
| doctor_id | INTEGER | FK → Doctor | Which doctor is consulted |
| patient_id | INTEGER | FK → Patient | Which patient is attending |
| scheduled_at | DATETIME | NOT NULL | Date & time of consultation |
| status | ENUM | DEFAULT 'PENDING' | PENDING, COMPLETED, CANCELLED, NO_SHOW |
| reason | TEXT | NOT NULL | Chief complaint (e.g., "Chest pain") |
| notes | TEXT | NULLABLE | Doctor's notes during visit |
| created_at | DATETIME | DEFAULT NOW() | When appointment was booked |

---

## 5. PRESCRIPTION
**Primary Key:** `prescription_id` | **Foreign Keys:** `appointment_id`, `doctor_id`, `patient_id`

| Field | Type | Constraint | Notes |
|-------|------|-----------|-------|
| prescription_id | INTEGER | PK | Unique identifier |
| appointment_id | INTEGER | FK → Appointment | Which visit triggered this |
| doctor_id | INTEGER | FK → Doctor | Which doctor issued it |
| patient_id | INTEGER | FK → Patient | Which patient receives it |
| issued_at | DATETIME | NOT NULL | When prescription was written |
| notes | TEXT | NULLABLE | Doctor's instructions (e.g., "Take with food") |
| is_complete | BOOLEAN | DEFAULT FALSE | Whether all items have been fulfilled |

---

## 6. PRESCRIPTION_ITEM
**Primary Key:** `item_id` | **Foreign Keys:** `prescription_id`, `drug_id`, `pharmacy_id`

| Field | Type | Constraint | Notes |
|-------|------|-----------|-------|
| item_id | INTEGER | PK | Unique identifier |
| prescription_id | INTEGER | FK → Prescription | Which prescription this item belongs to |
| drug_id | INTEGER | FK → Drug | Which drug to dispense |
| quantity | INTEGER | NOT NULL | Number of units (tablets, capsules, bottles) |
| dosage | VARCHAR(100) | NOT NULL | e.g., "500mg", "10ml" |
| instructions | TEXT | NOT NULL | How to take it (e.g., "2 tablets 3x daily for 7 days") |
| is_hospital_med | BOOLEAN | DEFAULT FALSE | TRUE = available in hospital stock; FALSE = patient buys from pharmacy |
| pharmacy_id | INTEGER | FK → Pharmacy (NULLABLE) | If not hospital med, which pharmacy to fulfill from |
| fulfilled_at | DATETIME | NULLABLE | When drug was actually dispensed |
| fulfilled_by_pharmacy_id | INTEGER | NULLABLE | Which pharmacy actually dispensed it (for tracking) |

---

## 7. DRUG
**Primary Key:** `drug_id`

| Field | Type | Constraint | Notes |
|-------|------|-----------|-------|
| drug_id | INTEGER | PK | Unique identifier |
| name | VARCHAR(200) | NOT NULL | Generic drug name (e.g., "Amoxicillin") |
| brand | VARCHAR(200) | NULLABLE | Brand name (e.g., "Augmentin") |
| unit_price | DECIMAL(10,2) | NOT NULL | Standard unit price |
| description | TEXT | NULLABLE | Drug description & warnings |
| created_at | DATETIME | DEFAULT NOW() | When drug was added to catalog |

---

## 8. HOSPITAL_DRUG_STOCK
**Primary Key:** `stock_id` | **Foreign Keys:** `drug_id`, `hospital_id`

| Field | Type | Constraint | Notes |
|-------|------|-----------|-------|
| stock_id | INTEGER | PK | Unique identifier |
| drug_id | INTEGER | FK → Drug | Which drug |
| hospital_id | INTEGER | FK → Hospital | Which hospital stocks it |
| qty_on_hand | INTEGER | NOT NULL, DEFAULT 0 | Current inventory count |
| reorder_level | INTEGER | NOT NULL | Alert threshold for reordering |
| last_updated | DATETIME | DEFAULT NOW() | Last time stock was checked |

**Composite Unique Index:** (drug_id, hospital_id) — each drug can appear once per hospital.

---

## 9. PHARMACY
**Primary Key:** `pharmacy_id`

| Field | Type | Constraint | Notes |
|-------|------|-----------|-------|
| pharmacy_id | INTEGER | PK | Unique identifier |
| name | VARCHAR(255) | NOT NULL | Pharmacy name |
| address | VARCHAR(500) | NOT NULL | Street address |
| phone | VARCHAR(20) | NOT NULL | Contact number |
| email | VARCHAR(100) | NULLABLE | Email address |
| city | VARCHAR(100) | NULLABLE | City location |
| is_partner | BOOLEAN | DEFAULT FALSE | Whether hospital has a partnership agreement |

---

## 10. PHARMACY_DRUG_STOCK
**Primary Key:** `pharmacy_stock_id` | **Foreign Keys:** `drug_id`, `pharmacy_id`

| Field | Type | Constraint | Notes |
|-------|------|-----------|-------|
| pharmacy_stock_id | INTEGER | PK | Unique identifier |
| drug_id | INTEGER | FK → Drug | Which drug |
| pharmacy_id | INTEGER | FK → Pharmacy | Which pharmacy stocks it |
| qty_on_hand | INTEGER | NOT NULL, DEFAULT 0 | Current inventory |
| price | DECIMAL(10,2) | NOT NULL | Pharmacy's selling price (may differ from standard) |
| last_updated | DATETIME | DEFAULT NOW() | Last time stock was checked |

**Composite Unique Index:** (drug_id, pharmacy_id) — each drug can appear once per pharmacy.

---

## 11. INVOICE
**Primary Key:** `invoice_id` | **Foreign Keys:** `hospital_id`, `patient_id`, `appointment_id`

| Field | Type | Constraint | Notes |
|-------|------|-----------|-------|
| invoice_id | INTEGER | PK | Unique identifier |
| hospital_id | INTEGER | FK → Hospital | Issuing hospital |
| patient_id | INTEGER | FK → Patient | Patient being billed |
| appointment_id | INTEGER | FK → Appointment (NULLABLE) | Related appointment (optional) |
| total_amount | DECIMAL(12,2) | NOT NULL | Total bill amount |
| status | ENUM | DEFAULT 'PENDING' | PENDING, PARTIAL, PAID, OVERDUE |
| issued_at | DATETIME | NOT NULL | When invoice was created |
| due_at | DATETIME | NULLABLE | Payment due date |

---

## 12. INVOICE_ITEM
**Primary Key:** `invoice_item_id` | **Foreign Keys:** `invoice_id`, `drug_id`, `pharmacy_id`

| Field | Type | Constraint | Notes |
|-------|------|-----------|-------|
| invoice_item_id | INTEGER | PK | Unique identifier |
| invoice_id | INTEGER | FK → Invoice | Which invoice this line belongs to |
| description | VARCHAR(500) | NOT NULL | Item description (e.g., "Consultation", "Amoxicillin 30x500mg") |
| quantity | INTEGER | NOT NULL | Quantity of items |
| unit_price | DECIMAL(10,2) | NOT NULL | Price per unit |
| line_total | DECIMAL(12,2) | NOT NULL | quantity × unit_price |
| drug_id | INTEGER | FK → Drug (NULLABLE) | If it's a drug item, link to drug |
| pharmacy_id | INTEGER | FK → Pharmacy (NULLABLE) | If pharmacy-sourced, which pharmacy |
| created_at | DATETIME | DEFAULT NOW() | When item was added |

---

## 13. PAYMENT
**Primary Key:** `payment_id` | **Foreign Key:** `invoice_id`

| Field | Type | Constraint | Notes |
|-------|------|-----------|-------|
| payment_id | INTEGER | PK | Unique identifier |
| invoice_id | INTEGER | FK → Invoice | Which invoice is being paid |
| amount | DECIMAL(12,2) | NOT NULL | Payment amount (can be partial) |
| method | ENUM | NOT NULL | CASH, INSURANCE, OUT_OF_POCKET |
| insurance_provider | VARCHAR(200) | NULLABLE | Insurance company name (if method='INSURANCE') |
| insurance_claim_ref | VARCHAR(100) | NULLABLE | Claim reference number |
| paid_at | DATETIME | NOT NULL | When payment was received |
| notes | TEXT | NULLABLE | Payment notes/reference |

---

## 14. STAFF
**Primary Key:** `staff_id` | **Foreign Key:** `hospital_id`

| Field | Type | Constraint | Notes |
|-------|------|-----------|-------|
| staff_id | INTEGER | PK | Unique identifier |
| hospital_id | INTEGER | FK → Hospital | Which hospital they work at |
| first_name | VARCHAR(100) | NOT NULL | Staff first name |
| last_name | VARCHAR(100) | NOT NULL | Staff last name |
| role | VARCHAR(100) | NOT NULL | e.g., Pharmacist, Nurse, Admin |
| email | VARCHAR(100) | NOT NULL | Email for login/contact |
| phone | VARCHAR(20) | NULLABLE | Phone number |

---

---

# CARDINALITIES (Plain Text Format)

## Hospital → Others
- **Hospital to Doctor:** One hospital employs zero or many doctors
- **Hospital to Patient:** One hospital registers zero or many patients
- **Hospital to Appointment:** One hospital hosts zero or many appointments
- **Hospital to Staff:** One hospital employs zero or many staff
- **Hospital to Invoice:** One hospital issues zero or many invoices
- **Hospital to Hospital Drug Stock:** One hospital maintains zero or many drug stock records

## Doctor
- **Doctor to Appointment:** One doctor conducts zero or many appointments
- **Doctor to Prescription:** One doctor issues zero or many prescriptions

## Patient
- **Patient to Appointment:** One patient books zero or many appointments
- **Patient to Prescription:** One patient receives zero or many prescriptions
- **Patient to Invoice:** One patient is billed for zero or many invoices

## Appointment
- **Appointment to Prescription:** One appointment generates zero or one prescription (because not all visits require medication)

## Prescription
- **Prescription to Prescription Item:** One prescription contains one or many prescription items (always at least one item)
- **Prescription to Drug:** Linked indirectly through Prescription Item (one drug appears in zero or many prescription items across all prescriptions)

## Prescription Item
- **Prescription Item to Drug:** One drug is prescribed in zero or many prescription items (across all patients)
- **Prescription Item to Pharmacy:** One pharmacy may fulfill zero or many prescription items (if the item is not a hospital medicine)

## Invoice
- **Invoice to Invoice Item:** One invoice contains one or many invoice items (always at least one item)
- **Invoice to Payment:** One invoice receives one or many payments (to support partial payments)

## Invoice Item
- **Invoice Item to Drug:** One drug is billed in zero or many invoice items
- **Invoice Item to Pharmacy:** One pharmacy sources zero or many invoice items (if the item is pharmacy-sourced)

## Drug
- **Drug to Hospital Drug Stock:** One drug has exactly one stock record per hospital (composite key)
- **Drug to Pharmacy Drug Stock:** One drug has exactly one stock record per pharmacy (composite key)

## Pharmacy
- **Pharmacy to Pharmacy Drug Stock:** One pharmacy maintains zero or many drug stock records

---

SUMMARY

- **Hospital:** 1 to many (Doctor, Patient, Staff, Appointment, Invoice, HospitalDrugStock)
- **Doctor:** 1 to many (Appointment, Prescription)
- **Patient:** 1 to many (Appointment, Prescription, Invoice)
- **Appointment:** 1 to zero or one (Prescription); 1 to many (Appointment)
- **Prescription:** 1 to many (PrescriptionItem)
- **PrescriptionItem:** many to 1 (Prescription, Drug, Pharmacy)
- **Invoice:** 1 to many (InvoiceItem, Payment)
- **InvoiceItem:** many to 1 (Invoice, Drug, Pharmacy)
- **Drug:** 1 to many (PrescriptionItem, InvoiceItem, HospitalDrugStock, PharmacyDrugStock)
- **Pharmacy:** 1 to many (PrescriptionItem, InvoiceItem, PharmacyDrugStock)
