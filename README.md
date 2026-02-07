# HOSPITAL SYSTEM ENTITIES & CARDINALITIES

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

# PATIENT LOGIN & APPOINTMENT WORKFLOW

---

## **Workflow Overview**

This document outlines the complete end-to-end workflow for a patient logging into the hospital system, booking an appointment, receiving treatment, getting a prescription, and paying invoices.

https://mermaid.live/edit#pako:eNqtVs1y2zYQfhUMD7nU8Z_sWNHU6cAUJTORKFainDh1RwOTsIgxRSgEaMeVfey503tneukD9JnyBH2ELhcUSUuZHDr1eDQEsLvf4sO3wK6sUEbc6ljzjC1jEnSvUgJ_9Kcr658_f_-L-EwLnmpyIZTQipxLtRSaJeQ9v4YJfmX9TF6-fEPOwN6OpVS8NrnJ5IIMhNJgZKKeoa0NtgM5Fym5FzomzoKJZM-PZcrJC8BT6l5mUeVjo093RXMdQyIihIRk-v11tvdmkochV-omT354MsbdwvjRk4_EAZBJLO-Jk2Uy66C9m96xRETEznhUhGKJqmAck1ozzCVXj6SHybKoIqLLVHwtWSND89vDAH0wrynj94qMlkW-ymRwsEvOpLwldLmUItULMMP5w120Lnavm4sKV1vr1YyrMBMmIK4clSuwMymAi42c-pjTeUGFM3DsoLOFXjmco6kLphiQ3sGhsOuEk64MtcwM3PUDmSx5CMTph8rTRc-3BQhPeKhLD_ICXepAk0TWcG_R6V2hmowzzQn1_ZHrBUPHC9CP1TnORLRDloZU_I4QoPhE07gUHK6pMOZRnvBoxjSMNNO5OvUdr-t6_Qr9HaIPCnSZ3ohsgZoC9Y35QqQRz8ikyY35HaDTEJy-_PZ3k0OQhOaYyfrkaZaJu8ZpDNHVA9eSG4BVeaJrKbNnx04CseCVt4feo1VXsHkqlTBnMeafcgGCwMGQR2VhrCvB_I6qevABfLqMvka1IYmcEns09AdO4HQraB-hfwTfPk95Vnhvpe56FyPXdjbYGtUlNK737SqVc0X8sTOxx64fuCOv8hsj1qQooIbOSZdrUFBZP8vGytek0FTJhoJwXRT4Rhqp1Fv1MsEUgpUd8_CWOCyMSTfL5-vbo7razO2jZXj7nO6g3vS0lnZztzM3cIboHkFgzPOTfij2odicr5OcVZJe8Og0GE-dcnsxA62GD-B36k0Hgyr9KSZ-URAt1JKncA0XiaMTXsPrzItZTByXujzMOAoOkpgBpTFL63vtAoO-h6A0ioiWpDzpeg-fcgbXaJF_ngo9W2ZwB5lME5HymZaAuEFxUCqS7JFRrom8IZjOI_nw_xHWo4PJVxijk4nb9xrq_oAbvGxc2QG7LeTZUBlGgb07nzXPUnjV_DJiFeUSo3wsopRLZH0Iqj6FNYLPHlQVA1dG4_rJPBMJMDff4Ow9IlD6n0r4Y-ncDEipmSxe7QrbkK_Wp4xxm8VOehxWvytr4eW5zEFlcPGs56rd49yQZbf5cmMj1DQA1N56JcsHDAMVRb_1yFLTB9DuChis3s0h17GMnhchNa-3C5lnLA35I6FOQ1v0smJtgd6nrjeZjqlXblms_UDO8k7AY3Dq0aFZq7jqiQS4sBMmFptbNOiltn2QNteQQe-bGYymwWzUm_kj-50TbMullycJoQuZb71JJZoNdAFI_5sgNp2cm8NbQJEItDJUbgY1vRA9r_XW1ESlNZ-63T2fjgOX1jcR7ZXO5bD_bFhOmmaDulsyGPOQQ90hzotndUjOoZGUWV121PQdtGg8vvzxK3HSqLPdpJkeBOiDyNBVKuxjoRvBMNYOdL0isjo6y_mOteCg32JorQqIKwsazuIZ7sBnxG8YFMKVdZU-gduSpR-lXKw9M5nPY6tzAw0ljHLkrHitM1ab8KKvsIsztDqHrX2MYXVW1merc9zePWm_ar96fdhq78P_q-Md6wGmj3cPD05O2gcHR-3X7aNW62nH-gVRj3ZbrdZh-2R_v7V_3GofHu1YcKJAz9D08tjSP_0Lxy_IgQ 
---

## **PHASE 1: PATIENT AUTHENTICATION & DASHBOARD**

### Step 1: Patient Access
- Patient visits hospital website
- Selects hospital from available list

### Step 2: Login
- Patient enters **email/phone** and **password**
- System validates credentials against `PATIENT` table

**Outcome:**
- ✅ **Success:** Load patient dashboard with all options
- ❌ **Failure:** Show error message, prompt retry

### Step 3: Dashboard Display
Patient sees main menu:
1. **Book Appointment**
2. **View Past Appointments**
3. **View Prescriptions**
4. **View Invoices & Payments**

---

## **PHASE 2: APPOINTMENT BOOKING**

### Step 4: Select "Book Appointment"

### Step 5: View Available Doctors
- Display doctors by specialty (e.g., Cardiology, Neurology)
- Filter by hospital
- Show available time slots

### Step 6: Make Selection
- Patient picks **doctor** + **available slot**
- Enters **reason for visit** (chief complaint)

### Step 7: Create Appointment Record
```
INSERT INTO APPOINTMENT (
  appointment_id,
  hospital_id,
  doctor_id,
  patient_id,
  scheduled_at,
  status,
  reason,
  created_at
)
VALUES (
  NEXT_ID,
  patient.hospital_id,
  selected_doctor_id,
  patient.patient_id,
  selected_slot,
  'PENDING',
  'Chest pain / Fever / etc.',
  NOW()
)
```

### Step 8: Confirmation
- Send appointment confirmation via **email/SMS**
- Display on patient dashboard with appointment ID

---

## **PHASE 3: CONSULTATION & DIAGNOSIS**

### Step 9: Appointment Date Arrives
- Patient arrives at hospital at scheduled time
- Staff updates appointment status tracking

### Step 10: Doctor Consultation
- Doctor examines patient
- Documents clinical findings in notes
- Makes diagnosis

### Step 11: Medication Decision
**Decision Point:** Does patient need medication?

#### **Path A: No Medication**
- Doctor updates `APPOINTMENT.status = 'COMPLETED'`
- Generate consultation invoice (flat fee)
- Skip to **PHASE 5: BILLING**

#### **Path B: Medication Required**
- Proceed to **Step 12**

---

## **PHASE 4: PRESCRIPTION & DRUG FULFILLMENT**

### Step 12: Doctor Issues Prescription
```
INSERT INTO PRESCRIPTION (
  prescription_id,
  appointment_id,
  doctor_id,
  patient_id,
  issued_at,
  notes,
  is_complete
)
VALUES (
  NEXT_ID,
  appointment.appointment_id,
  doctor.doctor_id,
  patient.patient_id,
  NOW(),
  'Take with food. Avoid dairy.',
  FALSE
)
```

### Step 13: For Each Drug in Prescription

**Decision Point:** Is drug in hospital stock?

#### **Path A: Hospital Stock Available**

**Step 13a:**
```
INSERT INTO PRESCRIPTION_ITEM (
  item_id,
  prescription_id,
  drug_id,
  quantity,
  dosage,
  instructions,
  is_hospital_med,
  pharmacy_id,
  fulfilled_at
)
VALUES (
  NEXT_ID,
  prescription.prescription_id,
  amoxicillin_id,
  30,
  '500mg',
  '1 capsule 3x daily for 10 days',
  TRUE,
  NULL,
  NOW()
)
```

**Step 13b:** Dispense from inventory
```
UPDATE HOSPITAL_DRUG_STOCK
SET qty_on_hand = qty_on_hand - 30
WHERE drug_id = amoxicillin_id
  AND hospital_id = patient.hospital_id
```

**Step 13c:** Add to invoice line items
```
INSERT INTO INVOICE_ITEM (
  invoice_item_id,
  invoice_id,
  description,
  quantity,
  unit_price,
  line_total,
  drug_id
)
VALUES (
  NEXT_ID,
  invoice.invoice_id,
  'Amoxicillin 30×500mg',
  30,
  0.50,
  15.00,
  amoxicillin_id
)
```

---

#### **Path B: Hospital Out of Stock**

**Step 13d:**
```
INSERT INTO PRESCRIPTION_ITEM (
  item_id,
  prescription_id,
  drug_id,
  quantity,
  dosage,
  instructions,
  is_hospital_med,
  pharmacy_id,
  fulfilled_at
)
VALUES (
  NEXT_ID,
  prescription.prescription_id,
  ibuprofen_id,
  20,
  '200mg',
  '1-2 tablets every 6 hours as needed',
  FALSE,
  partner_pharmacy_id,
  NULL
)
```

**Step 13e:** Patient gets prescription
- Print physical or digital prescription
- Redirect to external pharmacy (assigned or patient's choice)
- Patient buys from pharmacy and pays pharmacy directly OR hospital bills for it

**Step 13f (Optional tracking):** Record in invoice
```
INSERT INTO INVOICE_ITEM (
  invoice_item_id,
  invoice_id,
  description,
  quantity,
  unit_price,
  line_total,
  drug_id,
  pharmacy_id
)
VALUES (
  NEXT_ID,
  invoice.invoice_id,
  'Ibuprofen 20×200mg (Pharmacy)',
  20,
  1.50,
  30.00,
  ibuprofen_id,
  partner_pharmacy_id
)
```

---

### Step 14: Mark Prescription Complete
```
UPDATE APPOINTMENT
SET status = 'COMPLETED'
WHERE appointment_id = consultation.appointment_id
```

---

## **PHASE 5: BILLING & INVOICING**

### Step 15: Generate Invoice
```
INSERT INTO INVOICE (
  invoice_id,
  hospital_id,
  patient_id,
  appointment_id,
  total_amount,
  status,
  issued_at,
  due_at
)
VALUES (
  NEXT_ID,
  hospital.hospital_id,
  patient.patient_id,
  appointment.appointment_id,
  45.00,  -- 15 (meds) + 30 (pharmacy) + consultation_fee
  'PENDING',
  NOW(),
  DATE_ADD(NOW(), INTERVAL 14 DAY)
)
```

**Invoice Contents:**
- Consultation fee: $10–50 (varies by hospital)
- In-house medications: cost × quantity
- Pharmacy-sourced medications: cost + markup
- Any lab/imaging fees: if applicable

### Step 16: Patient Receives Invoice
- Send via **email**
- Display on **patient dashboard**
- Show itemized breakdown

---

## **PHASE 6: PAYMENT**

### Step 17: Patient Views Invoice on Dashboard

### Step 18: Patient Selects Payment Method

#### **Option A: Insurance**
```
INSERT INTO PAYMENT (
  payment_id,
  invoice_id,
  amount,
  method,
  insurance_provider,
  insurance_claim_ref,
  paid_at,
  notes
)
VALUES (
  NEXT_ID,
  invoice.invoice_id,
  45.00,
  'INSURANCE',
  'Blue Cross Insurance',
  'CLM-2025-001234',
  NOW(),
  'Claim filed; awaiting reimbursement'
)
```
- Hospital files claim with insurance company
- Claim reference tracked
- Possible co-pay from patient

#### **Option B: Out of Pocket (Full Amount)**
```
INSERT INTO PAYMENT (
  payment_id,
  invoice_id,
  amount,
  method,
  insurance_provider,
  paid_at
)
VALUES (
  NEXT_ID,
  invoice.invoice_id,
  45.00,
  'OUT_OF_POCKET',
  NULL,
  NOW()
)
```
- Patient pays 100% of bill immediately

#### **Option C: Cash**
```
INSERT INTO PAYMENT (
  payment_id,
  invoice_id,
  amount,
  method,
  paid_at
)
VALUES (
  NEXT_ID,
  invoice.invoice_id,
  45.00,
  'CASH',
  NOW()
)
```
- On-site payment at hospital counter
- Immediate receipt issued

### Step 19: Update Invoice Status
```
UPDATE INVOICE
SET status = 
  CASE 
    WHEN (SELECT SUM(amount) FROM PAYMENT WHERE invoice_id = invoice.invoice_id) >= total_amount
      THEN 'PAID'
    WHEN (SELECT SUM(amount) FROM PAYMENT WHERE invoice_id = invoice.invoice_id) > 0
      THEN 'PARTIAL'
    ELSE 'PENDING'
  END
WHERE invoice_id = invoice.invoice_id
```

---

## **PHASE 7: PATIENT DASHBOARD SUMMARY**

### Step 20: Patient Views Final Records
On **patient dashboard**, patient can now see:

1. **Appointment History**
   - Date, doctor, reason, status (COMPLETED)

2. **Prescriptions**
   - List of all past prescriptions
   - Items with fulfillment status
   - Hospital vs. pharmacy-sourced breakdown

3. **Invoices & Payments**
   - Invoice ID, total, status
   - Itemized breakdown (consultation, meds, lab work)
   - Payment details (method, insurance, amount, date)

4. **Download Options**
   - Receipt (PDF)
   - Prescription (for pharmacy or record-keeping)
   - Insurance claim documentation

---

## **Data Entities Involved**

| Phase | Entities Created/Updated |
|-------|--------------------------|
| **Phase 1: Auth** | Patient (login validation) |
| **Phase 2: Booking** | Appointment (new) |
| **Phase 3: Consultation** | Appointment (status update) |
| **Phase 4: Prescription** | Prescription, PrescriptionItem, HospitalDrugStock (inventory), InvoiceItem (optional) |
| **Phase 5: Billing** | Invoice, InvoiceItem |
| **Phase 6: Payment** | Payment |
| **Phase 7: Summary** | All (read-only display) |

---

## **Key Business Rules**

1. **Prescription without drugs:** Not allowed. Every prescription must have ≥1 item.
2. **Hospital meds first:** Doctors check hospital stock before prescribing external pharmacy drugs.
3. **Stock management:** When hospital dispenses, decrement `HospitalDrugStock.qty_on_hand`.
4. **Partial payments:** Invoices can accept multiple payments. Status changes from PENDING → PARTIAL → PAID.
5. **Insurance claims:** Only one claim per invoice (though multiple payments possible).
6. **Pharmacy routing:** If assigned in `PrescriptionItem.pharmacy_id`, patient directed there; otherwise, patient's choice.

---

## **Error Handling**

| Scenario | Action |
|----------|--------|
| **Invalid login credentials** | Reject, prompt retry up to 3 times, then lock account |
| **No doctors available** | Show message; allow offline booking request |
| **Drug out of stock at all pharmacies** | Flag as *call pharmacy for availability* |
| **Partial payment outstanding** | Send reminder email on due date |
| **Insurance claim rejected** | Mark as OVERDUE; send notification to patient |

---


