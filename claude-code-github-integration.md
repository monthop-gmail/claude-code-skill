# Claude Code GitHub Integration Guide

Claude Code มี GitHub integration ในตัวหลายแบบ นอกเหนือจากการใช้ `gh` CLI โดยตรง

---

## 1. PR Status ใน Footer

Claude Code จะแสดงลิงก์ PR พร้อมสถานะสีอัตโนมัติในแถบด้านล่าง:

| สี     | สถานะ            |
| ------ | ---------------- |
| เขียว  | Approved         |
| เหลือง | Pending Review   |
| แดง    | Changes Requested |
| เทา    | Draft            |
| ม่วง   | Merged           |

- อัปเดตทุก 60 วินาที
- กด `Ctrl+click` เพื่อเปิด PR ในเบราว์เซอร์
- ต้องมี `gh` CLI ติดตั้งและ authenticate แล้ว

---

## 2. GitHub App (`@claude` ใน PR/Issue)

### ติดตั้ง

```bash
claude
/install-github-app
```

### การใช้งาน

พิมพ์ใน PR comment หรือ Issue:

```
@claude implement this feature based on the issue description
@claude fix the TypeError in the user dashboard component
@claude how should I implement user authentication for this endpoint?
```

Claude จะตอบ, สร้าง PR, หรือ review code ให้อัตโนมัติ

### รองรับ

- AWS Bedrock และ Google Vertex AI สำหรับ self-hosted
- Custom prompts และ automation workflows
- ใช้ `CLAUDE.md` ของโปรเจกต์เป็นแนวทาง

---

## 3. Code Review อัตโนมัติ (Teams/Enterprise)

### วิธีใช้

พิมพ์ใน PR comment:

```
@claude review
```

### ความสามารถ

- รัน auto เมื่อสร้าง PR, push ใหม่, หรือสั่งด้วย `@claude review`
- วิเคราะห์โค้ดแบบขนาน หลาย agent พร้อมกัน
- ตรวจจับ:
  - Logic errors
  - Security vulnerabilities
  - Edge case issues
  - Subtle regressions
- โพสต์ inline comment ตรงจุดที่มีปัญหา
- ระบุ severity: **Normal**, **Nit**, **Pre-existing**
- มี collapsible reasoning อธิบายเหตุผลแต่ละจุด

### ปรับแต่งด้วย Guidance Files

- `CLAUDE.md` — กฎระดับโปรเจกต์ที่ใช้กับทุก review
- `REVIEW.md` — กฎเฉพาะสำหรับ code review

---

## 4. Session เชื่อมกับ PR

```bash
# ต่อ session จาก PR number
claude --from-pr 123

# ต่อ session จาก PR URL
claude --from-pr https://github.com/owner/repo/pull/123
```

Session ที่สร้าง PR ผ่าน `gh pr create` จะเชื่อมกับ PR นั้นอัตโนมัติ

---

## 5. GitHub Actions

ใช้ Claude เป็น GitHub Action ใน CI/CD:

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request:
    types: [opened, synchronize]
jobs:
  claude:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### ความสามารถ

- ตอบ `@claude` mentions อัตโนมัติ
- รันตาม scheduled triggers ได้
- รองรับ custom automation workflows
- ทำงานกับทุก GitHub event

---

## 6. Worktree สำหรับทำงานขนาน

```bash
# แยก worktree ทำงานคนละ feature
claude --worktree feature-auth
claude --worktree bugfix-123
```

แต่ละ worktree เป็น isolated copy ของ repo ทำงานแยกกันได้โดยไม่กระทบกัน

---

## 7. Session Management กับ Git

| คำสั่ง / ฟีเจอร์        | รายละเอียด                          |
| ----------------------- | ----------------------------------- |
| `claude --resume`       | เปิด session picker เลือก session   |
| `claude --continue`     | ต่อ session ล่าสุด                  |
| `claude --from-pr 123`  | ต่อ session ที่เชื่อมกับ PR          |
| `/resume`               | เปิด session picker (ในเซสชัน)      |
| `/rename`               | ตั้งชื่อ session                     |
| กรอง branch             | กด `B` ใน session picker            |

---

## 8. CLAUDE.md สำหรับ GitHub Context

สร้างไฟล์ `CLAUDE.md` หรือ `.claude/CLAUDE.md` ในโปรเจกต์เพื่อกำหนด:

- Coding standards ของโปรเจกต์
- เกณฑ์ code review
- Preferred patterns และ conventions
- Security requirements

ไฟล์นี้จะถูกใช้อัตโนมัติกับ:

- การสร้าง PR
- Code review findings
- GitHub Actions responses

---

## สรุป

| ฟีเจอร์             | วิธีใช้                          | รายละเอียด                              |
| -------------------- | -------------------------------- | --------------------------------------- |
| **PR Creation**      | บอก Claude "create a pr"         | สร้าง PR อัตโนมัติ                      |
| **PR Linking**       | `claude --from-pr 123`           | ต่อ session จาก PR                      |
| **Code Review**      | `@claude review` ใน PR           | วิเคราะห์โค้ดแบบ multi-agent            |
| **GitHub Actions**   | `/install-github-app`            | ตอบ @claude ใน PR/Issue อัตโนมัติ       |
| **PR Status**        | แสดงใน footer อัตโนมัติ           | สถานะ review แบบ real-time              |
| **Session Resuming** | `/resume`                        | หา session เดิม กรองตาม branch ได้       |
| **Worktrees**        | `claude --worktree feature-name` | ทำงานขนานแบบ isolated                   |
