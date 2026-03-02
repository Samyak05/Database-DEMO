# 🔐 Corporate VPD Demonstration Script (Final Evaluation Flow)

This project demonstrates:

✔ Role-Based Access Control (RBAC)  
✔ Virtual Private Database (PostgreSQL RLS)  
✔ Context-Aware Security using session variables  
✔ Location-Based Write Restriction  
✔ Row-Level Audit Logging  
✔ Separation of Duties (Auditor Role)  

This script is designed for a 10–12 minute live evaluation.

------------------------------------------------------------
🟢 PART 1 — System Setup (1 minute)
------------------------------------------------------------

Login as superuser:

```bash
psql -U postgres -d corporate_db
```

Show tables:

```sql
\dt
```

Say:
"This is the corporate database schema including business tables and audit infrastructure."

Clear old logs:

```sql
TRUNCATE audit_logs;
SELECT * FROM audit_logs;
```

Expected:
0 rows

Say:
"The audit log is clean before demonstration."

Exit:

```sql
\q
```

------------------------------------------------------------
🟢 PART 2 — Virtual Private Database (RLS) (3 minutes)
------------------------------------------------------------

🔹 Step 1: HR Access

```bash
psql -U app_hr -d corporate_db
```

Bind session context:

```sql
SET app.username = 'hr1';
SET app.role = 'HR';
SET app.department = 'HR';
SET app.location = 'Internal';
```

Run:

```sql
SELECT emp_id, emp_name, department
FROM employees
ORDER BY emp_id;
```

Expected:
Only HR department rows.

Say:
"Even though this table contains multiple departments, RLS dynamically filters rows based on role and department. This simulates a Virtual Private Database."

Exit.

---

🔹 Step 2: Manager Access

```bash
psql -U app_manager -d corporate_db
```

Bind:

```sql
SET app.username = 'manager1';
SET app.role = 'Manager';
SET app.department = 'Sales';
SET app.location = 'Internal';
```

Run:

```sql
SELECT emp_id, emp_name, department
FROM employees
ORDER BY emp_id;
```

Expected:
Only Sales department employees.

Say:
"Same query, different result. Security enforcement occurs at database layer."

Exit.

---

🔹 Step 3: Employee Self Access

```bash
psql -U app_employee -d corporate_db
```

Bind:

```sql
SET app.username = 'Alice';
SET app.role = 'Employee';
SET app.department = 'Sales';
SET app.location = 'Internal';
```

Run:

```sql
SELECT * FROM employees;
```

Expected:
Only Alice’s row visible.

Say:
"Employee can access only their own record."

Exit.

------------------------------------------------------------
🟢 PART 3 — Location-Based Enforcement (3 minutes)
------------------------------------------------------------

🔹 Case 1: Manager Internal Update (Allowed)

```bash
psql -U app_manager -d corporate_db
```

Bind:

```sql
SET app.username = 'manager1';
SET app.role = 'Manager';
SET app.department = 'Sales';
SET app.location = 'Internal';
```

Run:

```sql
BEGIN;

UPDATE employees
SET salary = salary + 500
WHERE department = 'Sales';

COMMIT;
```

Expected:
UPDATE 3

Say:
"Internal location allows modification as defined in UPDATE policy."

Exit.

---

🔹 Case 2: Manager External Update (Blocked)

Login again:

```bash
psql -U app_manager -d corporate_db
```

Bind:

```sql
SET app.username = 'manager1';
SET app.role = 'Manager';
SET app.department = 'Sales';
SET app.location = 'External';
```

Run:

```sql
UPDATE employees
SET salary = salary + 500
WHERE department = 'Sales';
```

Expected:
Blocked (no rows updated or permission denied via RLS).

Say:
"Location is evaluated dynamically inside RLS policy. External access prevents modification."

Exit.

------------------------------------------------------------
🟢 PART 4 — Auditing & Forensic Proof (3–4 minutes)
------------------------------------------------------------

🔹 Step 1: Perform Valid HR Update

```bash
psql -U app_hr -d corporate_db
```

Bind:

```sql
SET app.username = 'hr1';
SET app.role = 'HR';
SET app.department = 'HR';
SET app.location = 'Internal';
```

Run:

```sql
UPDATE employees
SET salary = salary + 1000
WHERE department = 'HR';
```

Exit.

---

🔹 Step 2: Auditor Verifies Logs

```bash
psql -U app_auditor -d corporate_db
```

Run:

```sql
SELECT audit_id,
       user_id,
       action,
       table_name,
       result,
       access_time
FROM audit_logs
ORDER BY audit_id;
```

Expected:
Entries for:
- Manager UPDATE (Internal)
- HR UPDATE

Say:
"Each row modification generates a separate audit entry. This ensures fine-grained accountability."

---

🔹 Step 3: Prove Manager Cannot View Logs

```bash
psql -U app_manager -d corporate_db
```

Run:

```sql
SELECT * FROM audit_logs;
```

Expected:
Permission denied.

Say:
"Operational users cannot inspect audit records. This enforces separation of duties."

Exit.

------------------------------------------------------------
🟢 PART 5 — Unauthorized Attempt Demonstration
------------------------------------------------------------

Login as employee:

```bash
psql -U app_employee -d corporate_db
```

Bind:

```sql
SET app.username = 'Alice';
SET app.role = 'Employee';
SET app.department = 'Sales';
SET app.location = 'Internal';
```

Run:

```sql
UPDATE employees
SET salary = 999999;
```

Expected:
Blocked by RLS.

Say:
"Modification blocked by policy. Since operation did not succeed, no audit entry is created. This highlights current auditing scope — successful writes only."

------------------------------------------------------------
🎯 FINAL CONCLUSION (Say This Clearly)
------------------------------------------------------------

"This system enforces context-aware access control using session-bound attributes including role, department, and location. Row-level security ensures department-level and user-level data isolation, while trigger-based auditing ensures accountability. Enforcement occurs entirely at the database layer, independent of application logic."

------------------------------------------------------------
✅ What This Demonstrates
------------------------------------------------------------

✔ Role-Based Access Control  
✔ Virtual Private Database behavior (RLS)  
✔ Dynamic context binding  
✔ Location-based enforcement  
✔ Row-level auditing  
✔ Separation of operational and audit roles  
✔ Secure database-layer enforcement  

------------------------------------------------------------
END OF DEMONSTRATION
------------------------------------------------------------
