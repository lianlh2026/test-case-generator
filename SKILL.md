---
name: test-case-generator
description: Generate comprehensive system test cases from requirements documents. Use this skill when the user needs to create test cases from requirements, design test scenarios, or perform test coverage analysis. Supports requirement clarification workflow, full test case generation, and coverage auditing.
---

# System Test Case Generator

## Overview

This skill enables systematic test case generation from requirements documents. It follows a professional QA workflow: requirement analysis, clarification, test case design, and coverage auditing. Test cases are written directly to the project's Excel file in the "Test Case" sheet.

## When to Use This Skill

Invoke this skill when:
- User provides a requirements document and requests test cases
- User wants to generate test scenarios from specifications
- User needs test coverage analysis for existing requirements
- User asks to design test cases with professional QA standards

## Workflow

### Phase 1: Requirement Analysis & Clarification

**DO NOT generate test cases immediately.** First, analyze requirements and output clarification questions.

#### Step 1.1: Extract Requirement Points

Read the requirements document and extract requirement points with IDs:
- Format: `REQ-001`, `REQ-002`, `REQ-003`, etc.
- Each requirement should be atomic and testable
- Group by functional areas if applicable

#### Step 1.2: Generate Clarification Questions

Output a "Clarification Question List" sorted by priority (High → Low), covering:

| Category | Example Questions |
|----------|-------------------|
| **Business Rules** | What are the exact business logic conditions? What happens when multiple rules conflict? |
| **Field Constraints** | What are the min/max lengths? Required vs optional? Data types? Format patterns? |
| **State Transitions** | What are valid state transitions? Can states be skipped? What triggers each transition? |
| **Permissions** | Who can perform each action? What are the role hierarchies? Any data-level restrictions? |
| **Exception Handling** | What errors should be caught? How to display errors? Retry mechanisms? |
| **Third-party Dependencies** | What external services? Timeout handling? Fallback behavior? |
| **Non-functional Metrics** | Response time requirements? Concurrent user limits? Availability targets? |

Output format:
```
## 澄清问题清单

### 高优先级
1. [业务规则] REQ-001: ...
2. [字段约束] REQ-002: ...

### 中优先级
...

### 低优先级
...

---
请回答以上问题后，我将生成完整测试用例。
```

**IMPORTANT**: Wait for user's answers before proceeding to Phase 2.

If user wants to skip clarification, proceed directly to Phase 2.

### Phase 2: Test Case Generation

#### Step 2.1: Design Test Cases Per Requirement

For each requirement point (REQ-xxx), design test cases covering:

| Test Type | Description | Priority |
|-----------|-------------|----------|
| **Normal Flow** | Happy path scenarios | H |
| **Boundary Values** | Min, max, edge cases | H |
| **Exception/Error** | Invalid inputs, error conditions | H |
| **Permission/Role** | Access control, role-based scenarios | M |
| **Data Validation** | Field validation, format checks | H |
| **Concurrency/Idempotency** | Race conditions, repeated requests | M |
| **Compatibility/Usability** | Browser, device, accessibility | L |

#### Step 2.2: Test Case Format

Each test case MUST contain:

| Field | Description | Example |
|-------|-------------|---------|
| **用例编号** | Format: TC-REQ-NNN-NNN | TC-REQ-001-001 |
| **测试项** | High-level test area | 用户登录功能 |
| **标题** | Specific test scenario | 正确账号密码登录成功 |
| **重要级别** | H (High), M (Medium), L (Low) | H |
| **预置条件** | Required setup | 用户已注册，账号状态正常 |
| **输入** | Test data | 账号: test@example.com, 密码: Test@123 |
| **操作步骤** | Step-by-step actions | 1. 打开登录页面\n2. 输入账号\n3. 输入密码\n4. 点击登录 |
| **预期结果** | Expected outcome | 登录成功，跳转到首页 |
| **备注** | Additional notes | 无 |

#### Step 2.3: Output Format

Output in **TSV (Tab-Separated Values)** format:

```
用例编号	测试项	标题	重要级别	预置条件	输入	操作步骤	预期结果	备注
TC-REQ-001-001	用户登录	正确账号密码登录成功	H	用户已注册，账号状态正常	账号: test@example.com\n密码: Test@123	1. 打开登录页面\n2. 输入账号\n3. 输入密码\n4. 点击登录	登录成功，跳转到首页	无
TC-REQ-001-002	用户登录	账号为空登录失败	H	用户已注册	账号: 空\n密码: Test@123	1. 打开登录页面\n2. 密码框输入密码\n3. 点击登录	提示"请输入账号"	无
```

**Note**: Use `\n` for multi-line content within cells.

#### Step 2.4: Write to Excel File

Write test cases to the project's Excel file in the "Test Case" sheet:

1. First, save TSV content to a temporary file
2. Execute the script to write to the project's Excel file:
   ```
   python scripts/generate_excel.py "<project_path>/测试用例.xlsx" --from-file test-cases.tsv
   ```
3. The script will:
   - **Create** the Excel file if it doesn't exist
   - **Create** the "Test Case" sheet if it doesn't exist
   - **Replace** existing content in the sheet (default mode)
   - Or **append** to existing content with `--mode append`

**Script Options:**
```
python scripts/generate_excel.py <excel_file> --from-file <tsv_file>
python scripts/generate_excel.py <excel_file> --from-file <tsv_file> --sheet "Test Case"
python scripts/generate_excel.py <excel_file> --from-file <tsv_file> --mode append
```

**Excel Formatting:**
- Professional header styling (blue background, white text)
- Optimized column widths for readability
- Text wrapping for multi-step content
- Frozen header row for easy navigation
- Auto-filter for sorting and filtering

### Phase 3: Coverage Audit

After generating test cases, perform coverage analysis:

#### Step 3.1: Summary by Requirement

```
## 覆盖性审计

### 按需求点统计
| 需求点 | 用例数量 | 正向 | 反向 | 边界 | 权限 | 异常 |
|--------|----------|------|------|------|------|------|
| REQ-001 | 8 | 2 | 2 | 2 | 1 | 1 |
| REQ-002 | 6 | 1 | 2 | 1 | 1 | 1 |
```

#### Step 3.2: Identify Missing Scenarios

List at least **15 potential missing scenarios**:

```
### 可能遗漏的场景
1. [REQ-001] 弱密码场景 - 用户使用简单密码注册
2. [REQ-001] 会话超时场景 - 登录后长时间无操作
3. [REQ-002] 并发创建场景 - 多用户同时创建相同数据
...
15. [REQ-xxx] xxx场景
```

#### Step 3.3: Supplementary Test Cases

For high-risk missing scenarios, provide additional test cases:

```
### 补充用例
用例编号	测试项	标题	重要级别	预置条件	输入	操作步骤	预期结果	备注
TC-REQ-001-S01	用户登录	弱密码拒绝登录	H	用户已注册，密码为123456	账号: test@example.com\n密码: 123456	1. 打开登录页面\n2. 输入账号密码\n3. 点击登录	提示"密码过于简单，请修改密码"	安全策略
```

## Test Design Patterns

### Boundary Value Analysis
```
For numeric fields:
- Min - 1 (invalid)
- Min (valid)
- Min + 1 (valid)
- Normal value (valid)
- Max - 1 (valid)
- Max (valid)
- Max + 1 (invalid)

For string fields:
- Empty string
- 1 character
- Min length
- Max length
- Max + 1
- Special characters
- Unicode characters
```

### State Transition Testing
```
For each state machine:
- Valid transitions (all paths)
- Invalid transitions (blocked paths)
- Self-transitions
- Skip transitions (if allowed)
- Concurrent transitions
```

### Error Handling Patterns
```
- Network timeout
- Service unavailable
- Invalid response format
- Empty response
- Malformed data
- Permission denied
- Resource not found
- Duplicate conflict
```

## Usage Example

**User Input:**
> 根据《订单系统需求文档v1.0.docx》生成测试用例

**AI Response:**
1. Read and analyze requirements document
2. Extract requirement points (REQ-001, REQ-002...)
3. Output clarification questions
4. Wait for user confirmation
5. Generate TSV test cases
6. Write to `<project_path>/测试用例.xlsx` in "Test Case" sheet
7. Perform coverage audit
8. Provide supplementary cases

## Output Files

Write test cases to the project's Excel file:
- `<project_path>/测试用例.xlsx` - Test cases written to "Test Case" sheet
- `coverage-audit.md` - Coverage analysis report

### Excel Writing Script

Use the bundled script to write to Excel file:

```bash
# Basic usage - writes to "Test Case" sheet
python scripts/generate_excel.py "<project_path>/测试用例.xlsx" --from-file test-cases.tsv

# Specify custom sheet name
python scripts/generate_excel.py "<excel_file>" --from-file <tsv_file> --sheet "CustomSheet"

# Append mode - add to existing content
python scripts/generate_excel.py "<excel_file>" --from-file <tsv_file> --mode append
```

**Script Features:**
1. Opens existing Excel file or creates new one
2. Finds or creates the "Test Case" sheet
3. Writes formatted test cases with:
   - Styled header row (blue background, white text)
   - Auto column widths
   - Text wrapping for multi-line content
   - Frozen header row
   - Auto-filter enabled
4. Auto-installs openpyxl if needed
