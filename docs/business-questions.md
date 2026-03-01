# Business Questions Catalog

This catalog maps the **17 business questions** the system can answer to their corresponding API endpoints and MCP tools.

## Skills & Expertise

### 1. Employee skill profile
**Question:** "What skills does employee `{employee_id}` have, at what proficiency, and with what confidence?"
**API:** `GET /tm/employees/{employee_id}/skills`
**MCP Tool:** `get_employee_skills`
**Response:** skill name, proficiency (0-5), confidence (0-100), source, last_updated_at

### 2. Evidence behind a skill level
**Question:** "Why do we think `{employee_id}` is proficient in `{skill}`?"
**API:** `GET /tm/employees/{employee_id}/skills/{skill_id}/evidence`
**MCP Tool:** `get_skill_evidence`
**Response:** evidence_type, title, issuer_or_system, evidence_date, signal_strength, url_or_ref, notes

### 3. Top experts for a skill
**Question:** "Who are the top experts in `{skill}`?"
**API:** `GET /tm/skills/{skill_id}/experts?min_proficiency=4&limit=20`
**MCP Tool:** `get_top_experts`
**Ranking:** proficiency desc → confidence desc → most recent update

### 4. Skill coverage and distribution
**Question:** "How many employees have `{skill}` at proficiency >= N?"
**API:** `GET /tm/skills/{skill_id}/coverage?min_proficiency=3`
**MCP Tool:** `get_skill_coverage`
**Response:** count of employees, distribution histogram by proficiency 0-5

### 5. Multi-skill search (AND)
**Question:** "Who has **both** `{skill A}` and `{skill B}` at proficiency >= N?"
**API:** `GET /tm/talent/search?skills=python,sql&min_proficiency=3`
**MCP Tool:** `search_talent`
**Response:** employee list with per-skill detail

### 6. Evidence-backed candidates
**Question:** "Who has `{skill}` at >= N *and* strong evidence (signal_strength >= M)?"
**API:** `GET /tm/skills/{skill_id}/candidates?min_proficiency=3&min_evidence_strength=4`
**MCP Tool:** `get_evidence_backed_candidates`
**Response:** employee_id, proficiency, confidence, plus strongest evidence items

### 7. Stale skills / recency checks
**Question:** "Which employees have `{skill}` but haven't had it validated recently?"
**API:** `GET /tm/skills/{skill_id}/stale?older_than_days=365`
**MCP Tool:** `get_stale_skills`
**Response:** employee_id, proficiency, confidence, last_updated_at

### 8. Employee's strongest skills
**Question:** "What are the top 10 skills for `{employee_id}`?"
**API:** `GET /tm/employees/{employee_id}/top-skills?limit=10`
**MCP Tool:** `get_top_skills`
**Ranking:** proficiency desc → confidence desc → evidence strength / recency

### 9. Skill co-occurrence
**Question:** "For employees strong in `{skill}`, what other skills commonly co-occur?"
**API:** `GET /tm/skills/{skill_id}/cooccurring?min_proficiency=3&top=20`
**MCP Tool:** `get_cooccurring_skills`
**Response:** skill list with co-occurrence counts

### 10. Evidence inventory
**Question:** "What evidence exists for `{employee_id}` across all skills?"
**API:** `GET /tm/employees/{employee_id}/evidence`
**MCP Tool:** `get_evidence_inventory`
**Response:** all evidence rows, sortable by date/strength/type

### 11. Skill taxonomy browsing
**Question:** "What skills exist in the catalog and how are they categorized?"
**API:** `GET /tm/skills?category=technical&search=python`
**MCP Tool:** `browse_skills`
**Response:** skill_id, name, category, description

### 12. Org skill summaries
**Question:** "What are the top skills in org unit `{org_unit_id}`?"
**API:** `GET /tm/orgs/{org_unit_id}/skills/summary`
**MCP Tool:** `get_org_skill_summary`
**Response:** skill counts, top experts per skill

**Follow-up:** "Who in my team has `{skill}` at >= N?"
**API:** `GET /tm/orgs/{org_unit_id}/skills/{skill_id}/experts?min_proficiency=3`
**MCP Tool:** `get_org_skill_experts`

### 13. Employee search by name
**Question:** "Find employees whose name matches `{name}`"
**API:** `GET /tm/employees/search?name=smith&limit=20`
**MCP Tool:** `search_employees`
**Response:** employee_id, full_name, org_unit_id

---

## Attrition Prediction

### 14. Individual attrition risk
**Question:** "What is the attrition risk for employee `{employee_id}`?"
**API:** `GET /tm/attrition/employees/{employee_id}`
**MCP Tool:** `get_employee_attrition_risk`
**Response:** probability (0-1), risk_level (low/medium/high/critical), factor breakdown with weights

### 15. Paginated attrition risk list
**Question:** "Show me attrition predictions for all employees, sorted by risk"
**API:** `GET /tm/attrition/employees?limit=50&offset=0&min_risk=medium&sort=risk_desc`
**MCP Tool:** `get_attrition_risks`
**Response:** paginated list with employee_id, probability, risk_level

### 16. High-risk employees
**Question:** "Which employees have attrition probability above `{threshold}`?"
**API:** `GET /tm/attrition/high-risk?threshold=0.25&limit=50`
**MCP Tool:** `get_high_risk_employees`
**Response:** employees sorted by probability descending, with risk_level

### 17. Org-level attrition summary
**Question:** "What is the attrition risk profile for org unit `{org_unit_id}`?"
**API:** `GET /tm/attrition/orgs/{org_unit_id}/summary?top_risk_limit=5`
**MCP Tool:** `get_org_attrition_summary`
**Response:** total employees, average probability, risk distribution (low/medium/high/critical counts), top-N riskiest employees

---

## Tool-to-Question Quick Reference

| MCP Tool | Question # | Category |
|----------|-----------|----------|
| `get_employee_skills` | 1 | Employee |
| `get_skill_evidence` | 2 | Employee |
| `get_top_skills` | 8 | Employee |
| `get_evidence_inventory` | 10 | Employee |
| `search_employees` | 13 | Employee |
| `browse_skills` | 11 | Skill |
| `get_top_experts` | 3 | Skill |
| `get_skill_coverage` | 4 | Skill |
| `get_evidence_backed_candidates` | 6 | Skill |
| `get_stale_skills` | 7 | Skill |
| `get_cooccurring_skills` | 9 | Skill |
| `search_talent` | 5 | Skill |
| `get_org_skill_summary` | 12 | Org |
| `get_org_skill_experts` | 12 | Org |
| `get_employee_attrition_risk` | 14 | Attrition |
| `get_attrition_risks` | 15 | Attrition |
| `get_high_risk_employees` | 16 | Attrition |
| `get_org_attrition_summary` | 17 | Attrition |
