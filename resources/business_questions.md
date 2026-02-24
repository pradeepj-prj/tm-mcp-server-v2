# Talent Management (TM) Skills DB — Business Questions (Schema 1)

This file lists **business questions** your TM Skills DB can answer and how they can be exposed as **API responses**.  
Assumption: Employee master/org data is in a separate HR DB. The TM DB stores only **skills, proficiency, confidence, and evidence**.

---

## 1) Employee skill profile
**Question:** “What skills does employee `{employee_id}` have, at what proficiency, and with what confidence?”  
**API idea:** `GET /tm/employees/{employee_id}/skills`  
**Response fields:** skill name, proficiency (0–5), confidence (0–100), source, last_updated_at.

**Common add-on:** include top evidence per skill (see next question).

---

## 2) Evidence behind an employee’s skill level
**Question:** “Why do we think `{employee_id}` is proficient in `{skill}`?”  
**API idea:** `GET /tm/employees/{employee_id}/skills/{skill_id}/evidence`  
**Response fields:** evidence_type, title, issuer_or_system, evidence_date, signal_strength, url_or_ref, notes.

---

## 3) Top experts for a given skill
**Question:** “Who are the top experts in `{skill}`?”  
**API idea:** `GET /tm/skills/{skill_id}/experts?min_proficiency=4&limit=20`  
**Ranking logic:** proficiency desc → confidence desc → most recent update.

**Response fields:** employee_id, proficiency, confidence, last_updated_at (+ optional employee attributes from HR DB via join).

---

## 4) Skill coverage and distribution
**Question:** “How many employees have `{skill}` at proficiency ≥ N?”  
**API idea:** `GET /tm/skills/{skill_id}/coverage?min_proficiency=3`  
**Response fields:** count of employees, distribution histogram by proficiency 0–5.

---

## 5) Multi-skill search (“AND”)
**Question:** “Who has **both** `{skill A}` and `{skill B}` at proficiency ≥ N?”  
**API idea:** `GET /tm/talent/search?skills=python,sql&min_proficiency=3`  
**Response fields:** employee_id list, plus match details per skill.

---

## 6) Evidence-backed candidates
**Question:** “Who has `{skill}` at ≥ N *and* strong evidence (signal_strength ≥ M)?”  
**API idea:** `GET /tm/skills/{skill_id}/candidates?min_proficiency=3&min_evidence_strength=4`  
**Response fields:** employee_id, proficiency, confidence, plus summary of strongest evidence item(s).

---

## 7) Stale skills / recency checks
**Question:** “Which employees have `{skill}` but it hasn’t been updated/validated recently?”  
**API idea:** `GET /tm/skills/{skill_id}/stale?older_than_days=365`  
**Response fields:** employee_id, proficiency, confidence, last_updated_at.

*Useful demo angle:* governance / freshness.

---

## 8) Employee’s strongest skills (a “skill passport” view)
**Question:** “What are the top 10 skills for `{employee_id}`?”  
**API idea:** `GET /tm/employees/{employee_id}/top-skills?limit=10`  
**Ranking logic:** proficiency desc → confidence desc → evidence strength / recency.

---

## 9) Skill adjacency via evidence (optional insight)
**Question:** “For employees strong in `{skill}`, what other skills commonly co-occur?”  
**API idea:** `GET /tm/skills/{skill_id}/cooccurring?min_proficiency=3&top=20`  
**Response fields:** skill list with counts.

*(This uses only employee_skill; no need for org data.)*

---

## 10) Evidence inventory
**Question:** “What evidence exists for `{employee_id}` across all skills?”  
**API idea:** `GET /tm/employees/{employee_id}/evidence`  
**Response fields:** evidence rows, sortable by date/strength/type.

---

## 11) Skill taxonomy browsing
**Question:** “What skills exist in the catalog and how are they categorized?”  
**API idea:** `GET /tm/skills?category=technical&search=python`  
**Response fields:** skill_id, name, category, description.

---

## 12) (If you join HR DB) Team/org skill summaries
These require joining HR DB org structure to TM DB skill mappings.

**Question:** “What are the top skills in org unit `{org_unit_id}`?”  
**API idea:** `GET /tm/orgs/{org_unit_id}/skills/summary`  
**Response fields:** skill counts, top experts per skill.

**Question:** “Who in my team matches `{skill}` at ≥ N?”  
**API idea:** `GET /tm/orgs/{org_unit_id}/skills/{skill_id}/experts?min_proficiency=3`

---

## 13) Employee search by name
**Question:** “Find employees whose name matches `{name}`”
**API idea:** `GET /tm/employees/search?name=smith&limit=20`
**Response fields:** employee_id, full_name, org_unit_id — partial match, case-insensitive.

*Useful for looking up employee IDs when you only know a name.*

---

## 14) Individual attrition risk prediction
**Question:** “What is the attrition risk for employee `{employee_id}`?”
**API idea:** `GET /tm/attrition/employees/{employee_id}`
**Response fields:** probability (0–1), risk_level (low/medium/high/critical), factor breakdown with weights.

---

## 15) Paginated attrition risk list
**Question:** “Show me attrition predictions for all employees, sorted by risk”
**API idea:** `GET /tm/attrition/employees?limit=50&offset=0&min_risk=medium&sort=risk_desc`
**Response fields:** paginated list of employee_id, probability, risk_level.

---

## 16) High-risk employees above a threshold
**Question:** “Which employees have attrition probability above `{threshold}`?”
**API idea:** `GET /tm/attrition/high-risk?threshold=0.25&limit=50`
**Response fields:** employees sorted by probability descending, with risk_level.

---

## 17) Org-level attrition summary
**Question:** “What is the attrition risk profile for org unit `{org_unit_id}`?”
**API idea:** `GET /tm/attrition/orgs/{org_unit_id}/summary?top_risk_limit=5`
**Response fields:** total employees, average probability, risk distribution (low/medium/high/critical counts), top-N riskiest employees.

---

## Notes on demo realism
- It’s realistic that only a subset of employees have TM skill records.
- Confidence can be explained as: “strength + recency + source of evidence”.
- Avoid sensitive HR fields (comp, benefits, IDs) in this TM DB.
