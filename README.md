# 🔐 Context-Aware Virtual Private Database (VPD) Demo

This document provides a complete step-by-step demonstration of:

- Role-Based Access Control (RBAC)
- Row-Level Security (Virtual Private Database behavior)
- Location-Based Access Control
- Write Auditing (INSERT / UPDATE / DELETE)
- Separation of Duties (Auditor role)

---

# 🧭 DEMONSTRATION FLOW

We will demonstrate:

1. Initial Setup Verification
2. Virtual Private Database (RLS)
3. Location-Based Access Restriction
4. Auditing Proof
5. Policy Enforcement Blocking

---

# 1️⃣ INITIAL VERIFICATION

## Login as Database Administrator

```bash
psql -U postgres -d corporate_db
```

Verify tables exist:

```sql
\dt
```

Reset audit logs for clean demonstration:

```sql
TRUNCATE audit_logs;
SELECT * FROM audit_logs;
```

Expected: No rows.

Exit:

```sql
\q
```

---

# 2️⃣ VIRTUAL PRIVATE DATABASE (ROW-LEVEL SECURITY)

## 🔹 Case A: HR Access

Login:

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

Query employees:

```sql
SELECT emp_id, emp_name, department
FROM employees
ORDER BY emp_id;
```

Expected Result:
✔ Only HR department employees visible.

Explanation:
RLS filters rows dynamically based on role and department.

Exit:

```sql
\q
```

---

## 🔹 Case B: Manager Access

Login:

```bash
psql -U app_manager -d corporate_db
```

Bind context:

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

Expected Result:
✔ Only Sales department employees visible.

Explanation:
Same query, different output → Virtual Private Database behavior.

Exit:

```sql
\q
```

---

## 🔹 Case C: Employee Access

Login:

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

Expected Result:
✔ Only Alice's row visible.

Exit:

```sql
\q
```

---

# 3️⃣ LOCATION-BASED ACCESS CONTROL

## 🔹 Case A: Internal Modification (Allowed)

Login:

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

Attempt update:

```sql
UPDATE employees
SET salary = salary + 500
WHERE department = 'Sales';
```

Expected:
✔ Update successful.

Exit:

```sql
\q
```

---

## 🔹 Case B: External Modification (Blocked)

Login:

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

Attempt update:

```sql
UPDATE employees
SET salary = salary + 500
WHERE department = 'Sales';
```

Expected:
❌ Permission denied due to location-based RLS policy.

Exit:

```sql
\q
```

Explanation:
Location attribute is evaluated dynamically inside RLS policy.

---

# 4️⃣ AUDITING DEMONSTRATION

## 🔹 Step 1: Perform Valid Modification

Login as HR:

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

Execute:

```sql
UPDATE employees
SET salary = salary + 1000
WHERE department = 'HR';
```

Exit:

```sql
\q
```

---

## 🔹 Step 2: Auditor Verification

Login as Auditor:

```bash
psql -U app_auditor -d corporate_db
```

Check audit logs:

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

✔ UPDATE entries recorded  
✔ Correct user_id logged  
✔ Timestamp present  
✔ One entry per modified row  

Explanation:
Triggers log every successful INSERT / UPDATE / DELETE.

Exit:

```sql
\q
```

---

# 5️⃣ POLICY ENFORCEMENT TEST (Unauthorized Modification)

Login as Employee:

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

Attempt unauthorized update:

```sql
UPDATE employees
SET salary = 999999;
```

Expected:
❌ Blocked by RLS.

Explanation:
Authorization occurs before execution.
Triggers do not run when RLS blocks access.

Exit:

```sql
\q
```

---

# 🔐 SECURITY FEATURES DEMONSTRATED

✔ Authentication via DB Roles  
✔ Authorization via GRANT statements  
✔ Context-aware filtering via RLS  
✔ Department-based data isolation  
✔ Location-based modification control  
✔ Trigger-based write auditing  
✔ Separation of duties (Auditor role)  

---

# 🎯 FINAL STATEMENT

This system enforces access control entirely at the database layer using:

- PostgreSQL Role-Based Access
- Row-Level Security (VPD simulation)
- Session-bound contextual attributes
- Location-aware policy enforcement
- Trigger-based auditing

Security decisions are evaluated dynamically at query execution time, independent of application logic.

---
