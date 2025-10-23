# surveyking-array-exception
Array Index Out of Bounds Exception in Answer Submission

## Summary

A critical array index validation issue has been identified in the SurveyKing application's answer submission service. The vulnerability allows unauthenticated users to trigger an `IndexOutOfBoundsException` by submitting answers to an empty survey (a survey with no questions), causing service disruption and potential information disclosure through error messages.


## Vulnerability Details

### Description

The `AnswerServiceImpl.saveAnswer()` method attempts to access the first element of a question list without validating that the list contains any elements. When a user submits an answer to a survey that has no questions defined, the application crashes with an `IndexOutOfBoundsException`.

### Affected Code Location

**File:** `server/rdbms/src/main/java/cn/surveyking/server/impl/AnswerServiceImpl.java`
**Line:** 113
**Method:** `saveAnswer(AnswerRequest request)`

### Vulnerable Code

```java
public AnswerView saveAnswer(AnswerRequest request) {
    // ... previous code ...

    ProjectView project = projectService.getProject(answer.getProjectId());
    List<SurveySchema> flatQuestionSchema = SchemaHelper.flatQuestions(project.getSurvey());

    // ❌ VULNERABILITY: No validation that flatQuestionSchema is not empty
    SurveySchema.QuestionType questionType = flatQuestionSchema.get(0).getType();

    // ... rest of method ...
}
```

### Root Cause Analysis

1. **Missing Validation:** The code assumes `flatQuestionSchema` always contains at least one element
2. **Unsafe Array Access:** Direct use of `.get(0)` without bounds checking
3. **No Business Logic Validation:** Empty surveys can be created and saved to the database
4. **Public API Exposure:** The vulnerable endpoint `/api/public/saveAnswer` requires no authentication

---

## Impact Assessment

### Technical Impact

| Impact Category | Severity | Description |
|----------------|----------|-------------|
| **Availability** | High | Service crashes on every answer submission to empty surveys |
| **Confidentiality** | Low | Stack traces may leak internal implementation details |
| **Integrity** | None | No data corruption occurs |
| **Authentication** | None | Vulnerability exploitable by anonymous users |

### Business Impact

- **Service Disruption:** All users attempting to answer affected surveys experience failures
- **User Experience:** Poor user experience with cryptic error messages
- **Database Transactions:** Transaction rollbacks may affect concurrent operations
- **Operational Overhead:** Increased support tickets and error log analysis

### CVSS v3.1 Score Breakdown

**Base Score:** 5.3 (Medium)
**Vector String:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:L`

| Metric | Value | Explanation |
|--------|-------|-------------|
| Attack Vector (AV) | Network (N) | Exploitable via public HTTP API |
| Attack Complexity (AC) | Low (L) | No special conditions required |
| Privileges Required (PR) | None (N) | No authentication needed |
| User Interaction (UI) | None (N) | Fully automated attack |
| Scope (S) | Unchanged (U) | Impact limited to vulnerable component |
| Confidentiality (C) | None (N) | No data disclosure |
| Integrity (I) | None (N) | No data modification |
| Availability (A) | Low (L) | Partial DoS for specific survey |


## Proof of Concept (PoC)

```http
POST /api/project/save HTTP/1.1
Host: target.com
Content-Type: application/json
Authorization: Bearer <admin-token>

{
  "name": "Empty Survey PoC",
  "survey": {
    "id": "empty-poc",
    "title": "Empty Survey",
    "pages": []
  }
}
```

<img width="1500" height="118" alt="image" src="https://github.com/user-attachments/assets/da86ea3e-3ce2-4f3a-adf9-0f084bb01c8c" />

<img width="1075" height="369" alt="image" src="https://github.com/user-attachments/assets/ad637de8-18f3-41f2-933b-e58f463189c6" />

<img width="1510" height="944" alt="image" src="https://github.com/user-attachments/assets/c9bc3a59-d8e1-46e6-98c3-b76f37fd9dab" />


<img width="1124" height="428" alt="image" src="https://github.com/user-attachments/assets/e8a50f54-c6e9-4901-8c89-306a9423c656" />


