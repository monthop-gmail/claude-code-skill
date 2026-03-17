# Claude Code Agent Teams Guide

Agent Teams คือฟีเจอร์ experimental ที่ให้ Claude Code หลายตัวทำงานร่วมกันเป็นทีม โดยแต่ละตัวมี context window แยกกัน สื่อสารกันได้ และแบ่งงานกันทำแบบขนาน

---

## ชื่อเรียก Claude แต่ละบทบาท

| ชื่อเรียก              | คือใคร                                                       |
| ---------------------- | ------------------------------------------------------------ |
| **Main / Team Lead**   | Claude ตัวหลักที่ผู้ใช้คุยด้วยโดยตรง สั่งงาน ประสานงาน        |
| **Subagent**           | ลูกน้องเฉพาะกิจที่ Lead สร้างขึ้น ทำเสร็จแล้วรายงานผลกลับ     |
| **Teammate**           | สมาชิกทีมที่คุยกันเองได้ แชร์งานกันได้ (ในโหมด Teams)          |

> ทุกตัวเป็น Claude เหมือนกัน แค่บทบาทต่างกัน

---

## สถาปัตยกรรม

| ส่วนประกอบ    | หน้าที่                                                    |
| ------------- | ---------------------------------------------------------- |
| **Team Lead** | เซสชันหลักที่สร้างทีม, สั่งงาน, ประสานงาน                  |
| **Teammates** | Claude Code แต่ละตัวที่ทำงานอิสระตาม task ที่ได้รับ          |
| **Task List** | รายการงานที่แชร์ร่วมกัน teammate claim งานเองได้             |
| **Mailbox**   | ระบบส่งข้อความระหว่าง agent                                 |

> **Agent Teams vs Subagents:** Subagent รายงานผลกลับไปที่ agent หลักอย่างเดียว แต่ Teammates แชร์ task list, claim งานเอง, และคุยกันเองได้โดยตรง

---

## Agents (Subagents) vs Teams — ต่างกันอย่างไร?

|                        | **Agents / Subagents**                          | **Agent Teams**                                   |
| ---------------------- | ----------------------------------------------- | ------------------------------------------------- |
| **คืออะไร**            | ลูกน้องชั่วคราวที่ Claude หลักสร้างขึ้นทำงานเฉพาะกิจ | ทีมงานที่มีบทบาทชัดเจน สื่อสารกันเองได้            |
| **อายุ**               | เกิดตอนแชท จบแล้วหายไป                           | อยู่ตลอด session, ใช้ซ้ำได้                         |
| **การสื่อสาร**          | รายงานผลกลับไปที่ Claude หลักอย่างเดียว           | คุยกันเองได้ระหว่าง teammates + แชร์ task list      |
| **การจัดการงาน**        | Claude หลักสั่งและรอผล                            | Teammates claim งานเอง, มี dependencies ได้         |
| **Context window**     | แยกคนละตัว แต่ไม่แชร์กัน                          | แยกคนละตัว แต่แชร์ผ่าน messages และ task list       |
| **UI ที่เห็น**          | แถบสถานะด้านล่าง กดเข้าไปดูได้                    | แยก pane ใน tmux หรือดูใน process เดียวกัน          |
| **เปรียบเทียบ**         | จ้าง freelance ทำงานชิ้นเดียว                     | ทีมประจำที่ประสานงานกันเอง                          |
| **เหมาะกับ**           | งานอิสระที่แยกกันทำได้ เช่น ค้นหา, วิเคราะห์       | งานซับซ้อนที่ต้องประสานกันหลายมุม เช่น review, debug |
| **ค่าใช้จ่าย**          | ต่ำกว่า (จบเร็ว)                                  | สูงกว่า (หลาย instance ทำงานพร้อมกัน)               |

### ตัวอย่างเปรียบเทียบ

**แบบ Agents (เฉพาะกิจ)** — Claude หลักสั่ง 3 agents แยกกัน แต่ละตัวทำเสร็จแล้วส่งผลกลับ:
```text
สร้าง agent 3 ตัว review PR #42: ตัวแรกดู security, ตัวสองดู performance, ตัวสามดู test coverage
```

**แบบ Teams (ทีมประจำ)** — Teammates คุยกันเอง แชร์ task list และ claim งานเอง:
```text
Create an agent team to review PR #42. Spawn three reviewers:
- security-reviewer focused on vulnerabilities
- perf-reviewer checking performance impact
- test-reviewer validating test coverage
Have them discuss findings with each other before reporting.
```

---

## เปิดใช้งาน

Agent Teams ปิดอยู่โดย default ต้องเปิดเอง:

### วิธีที่ 1: settings.json

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### วิธีที่ 2: Environment Variable

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

### ข้อกำหนด

- Claude Code **v2.1.32** ขึ้นไป (ตรวจด้วย `claude --version`)

---

## Display Modes

ควบคุมวิธีแสดงผล teammate ด้วย `--teammate-mode`:

```bash
claude --teammate-mode auto        # default: auto detect
claude --teammate-mode in-process  # ทุกอย่างใน terminal เดียว
claude --teammate-mode tmux        # แยก pane ใน tmux
```

หรือตั้งค่าถาวรใน `settings.json`:

```json
{
  "teammateMode": "tmux"
}
```

### เปรียบเทียบ Display Modes

| Mode           | รายละเอียด                                      | ต้องการ             |
| -------------- | ----------------------------------------------- | ------------------- |
| **auto**       | อยู่ใน tmux ใช้ split-pane, ไม่อยู่ใช้ in-process | ไม่มี               |
| **in-process** | ทุก teammate อยู่ใน terminal เดียว                | ไม่มี               |
| **tmux**       | แต่ละ teammate ได้ pane ของตัวเอง                 | tmux หรือ iTerm2    |

---

## Tmux Integration

### ติดตั้ง tmux

```bash
# macOS
brew install tmux

# Debian / Ubuntu
sudo apt-get install tmux

# Fedora / RHEL
sudo dnf install tmux

# ตรวจสอบ
which tmux
```

### วิธีใช้งาน

```bash
# เริ่ม tmux session ก่อน
tmux new -s claude-team

# จากนั้นรัน Claude Code ด้วย teammate-mode tmux
claude --teammate-mode tmux
```

### พฤติกรรมของ Split-Pane Mode

- แต่ละ teammate ได้ pane ของตัวเอง
- เห็น output ของทุก teammate พร้อมกัน
- **คลิกเข้าไปใน pane** เพื่อคุยกับ teammate ตัวนั้นโดยตรง
- ทุก pane มี terminal view เต็มรูปแบบ

### iTerm2 (macOS)

ถ้าใช้ iTerm2 แทน tmux:
1. ติดตั้ง [`it2` CLI](https://github.com/mkusaka/it2)
2. เปิด Python API: **iTerm2 → Settings → General → Magic → Enable Python API**
3. ใช้ `tmux -CC` เป็น entry point

### ทำความสะอาด Orphaned Sessions

ถ้า tmux session ค้างหลังจากทีมจบงาน:

```bash
# ดู session ทั้งหมด
tmux ls

# ลบ session ที่ค้าง
tmux kill-session -t <session-name>
```

### ข้อจำกัดของ Split-Pane Mode

- ไม่รองรับ VS Code integrated terminal, Windows Terminal, Ghostty
- ทำงานได้ดีที่สุดบน macOS

---

## การโต้ตอบกับ Teammates

### In-Process Mode

| ปุ่ม           | ทำอะไร                        |
| -------------- | ----------------------------- |
| `Shift+Down`   | สลับดู teammate ตัวถัดไป       |
| `Enter`        | ดู session เต็มของ teammate    |
| `Escape`       | หยุด turn ปัจจุบันของ teammate |
| `Ctrl+T`       | เปิด/ปิด task list            |
| พิมพ์ข้อความ    | ส่งข้อความไปยัง teammate ปัจจุบัน |

### Split-Pane Mode (tmux)

- **คลิกเข้า pane** ของ teammate ที่ต้องการ
- พิมพ์ข้อความส่งได้เลย
- เห็น output ของทุกตัวพร้อมกัน

### สั่ง Lead ให้จัดการ

```text
Ask the security teammate to review the authentication module
Ask the researcher teammate to shut down
```

---

## การสื่อสารระหว่าง Agents

| รูปแบบ        | รายละเอียด                                    |
| ------------- | --------------------------------------------- |
| **message**   | ส่งถึง teammate เฉพาะตัว                       |
| **broadcast** | ส่งถึงทุก teammate พร้อมกัน (ใช้ token เยอะ)    |

- ข้อความส่งถึงผู้รับอัตโนมัติ (lead ไม่ต้อง poll)
- แจ้งเตือนอัตโนมัติเมื่อ teammate ว่าง (idle)
- Task list แชร์ร่วมกันทุกตัว

---

## Task Management

- **Lead กำหนดงาน** ให้ teammate แต่ละตัว
- **Self-claim:** teammate หยิบงานถัดไปที่ว่างเอง
- **Dependencies:** task ที่มี dependency จะถูก block จนกว่า prerequisite เสร็จ
- **File locking:** ป้องกัน race condition เมื่อหลายตัว claim พร้อมกัน

---

## Plan Approval Mode

สำหรับงานที่มีความเสี่ยง สั่งให้ teammate วางแผนก่อนลงมือ:

```text
Spawn an architect teammate to refactor the authentication module.
Require plan approval before they make any changes.
```

ขั้นตอน:
1. Teammate ทำงานแบบ read-only วางแผน
2. ส่งแผนให้ lead อนุมัติ
3. Lead อนุมัติ หรือ reject พร้อมเหตุผล
4. ถ้า reject → teammate แก้ไขแล้วส่งใหม่
5. อนุมัติแล้ว → teammate เริ่มลงมือทำ

กำหนดเกณฑ์ได้:
```text
Only approve plans that include test coverage
Reject plans that modify database schema
```

---

## Quality Gates ด้วย Hooks

ใช้ hooks บังคับกฎเมื่อ teammate ทำงานเสร็จ:

```json
{
  "hooks": {
    "TeammateIdle": [
      {
        "type": "command",
        "command": "~/.claude/hooks/validate-teammates.sh"
      }
    ]
  }
}
```

| Hook Event        | ทำอะไร                                               |
| ----------------- | ---------------------------------------------------- |
| **TeammateIdle**  | รันเมื่อ teammate จะว่าง, exit code 2 = สั่งทำต่อ      |
| **TaskCompleted** | รันเมื่อจะ mark task เสร็จ, exit code 2 = ยังไม่ให้เสร็จ |

---

## ตัวอย่างการใช้งาน

### Parallel Code Review

```text
Create an agent team to review PR #142. Spawn three reviewers:
- One focused on security implications
- One checking performance impact
- One validating test coverage
Have them each review and report findings.
```

### Debugging แบบแข่งกันหาสาเหตุ

```text
Users report the app exits after one message instead of staying connected.
Spawn 5 agent teammates to investigate different hypotheses. Have them talk to
each other to try to disprove each other's theories, like a scientific debate.
Update the findings doc with whatever consensus emerges.
```

### Design Exploration

```text
I'm designing a CLI tool that helps developers track TODO comments across
their codebase. Create an agent team to explore this from different angles:
one teammate on UX, one on technical architecture, one playing devil's advocate.
```

### Feature Implementation

```text
Create a team with 4 teammates to refactor these modules in parallel.
Use Sonnet for each teammate.
```

---

## ค่าใช้จ่ายและ Token

### ทำไมใช้ token เยอะ?

- แต่ละ teammate มี context window แยก = Claude instance แยก
- Token scale ตามจำนวน teammate
- ประมาณ **7x มากกว่า** single session (เมื่อใช้ plan mode)

### ตัวอย่างค่าใช้จ่าย

```
Single session:                ~$0.55 (6 นาที)
Agent team (5 teammates, 15 นาที): ~$2.75-5.50
```

### วิธีประหยัด

1. **ทีมเล็กพอ** — 3-5 teammates, 5-6 tasks ต่อตัว
2. **เลือก model ให้เหมาะ** — ใช้ Sonnet สำหรับงานประสาน, Opus สำหรับงานซับซ้อน
3. **Spawn prompt กระชับ** — teammate โหลด CLAUDE.md อัตโนมัติอยู่แล้ว
4. **ปิดทีมเมื่อเสร็จ** — teammate ที่ว่างยังกิน token อยู่ (`Clean up the team`)
5. **ลด MCP servers** — ในเซสชันทีม
6. **CLAUDE.md กระชับ** — ไม่เกิน 500 บรรทัด

---

## เมื่อไหร่ควรใช้ / ไม่ควรใช้

### ควรใช้

- Research และ review ที่แยกมุมมองได้ชัด
- สร้าง feature ใหม่หลาย module พร้อมกัน
- Debug ด้วย competing hypotheses
- Cross-layer coordination (frontend / backend / infra)

### ไม่ควรใช้

- งาน routine ธรรมดา (single session ถูกกว่า)
- งานที่ต้องทำเรียงลำดับ (ไม่ได้ประโยชน์จาก parallelism)
- แก้ไขไฟล์เดียวกัน (coordination overhead สูง)

---

## ข้อจำกัดที่ทราบ

| ข้อจำกัด                             | วิธีแก้                                      |
| ------------------------------------ | -------------------------------------------- |
| `/resume` ไม่ restore in-process teammates | บอก lead ให้ spawn teammate ใหม่             |
| Task status อาจ lag                   | ตรวจสอบเองแล้วสั่ง update                      |
| Shutdown ช้า                          | รอให้ teammate เสร็จ request ปัจจุบัน           |
| 1 ทีมต่อ session                      | ปิดทีมเก่าก่อนสร้างใหม่                        |
| ไม่มี nested teams                    | เฉพาะ lead เท่านั้นที่จัดการทีมได้               |
| Lead เปลี่ยนไม่ได้                     | วางแผนล่วงหน้า                                |
| Permission ตั้งตอน spawn              | เปลี่ยน mode ของ teammate ทีหลังได้             |

---

## ที่เก็บข้อมูล

```
~/.claude/teams/{team-name}/config.json    # Team config + รายชื่อสมาชิก
~/.claude/tasks/{team-name}/               # Task list
```

---

## Custom Slash Commands — ไม่ต้องพิมพ์ยาวทุกครั้ง

ถ้าต้องสั่งแบบเดิมบ่อยๆ เช่น:

```text
สร้าง agent 3 ตัว review PR #42: ตัวแรกดู security, ตัวสองดู performance, ตัวสามดู test coverage
```

**ไม่ต้องพิมพ์ใหม่ทุกครั้ง!** สร้าง Custom Slash Command ไว้ใช้ซ้ำได้:

### วิธีสร้าง

1. สร้างโฟลเดอร์ `.claude/commands/` ใน project
2. สร้างไฟล์ `.md` หนึ่งไฟล์ต่อหนึ่งคำสั่ง
3. ใช้ `$ARGUMENTS` เป็นตัวแปรรับค่าจากผู้ใช้

### ตัวอย่าง: `/review-pr`

ไฟล์ `.claude/commands/review-pr.md`:

```markdown
สร้าง agent 3 ตัว review PR #$ARGUMENTS พร้อมกัน:

1. **security-reviewer** — ตรวจช่องโหว่ security
2. **performance-reviewer** — ตรวจปัญหา performance
3. **test-coverage-reviewer** — ตรวจ test coverage gaps

แต่ละตัวให้ fetch PR diff ด้วย `gh pr diff` แล้วรายงานผลแบบ structured
```

### วิธีใช้

```text
/review-pr 42
```

แค่นี้เลย! `$ARGUMENTS` จะถูกแทนที่ด้วย `42` ให้อัตโนมัติ

### ระดับของ Commands

| ที่เก็บ | ใช้ได้กับ |
| ------- | --------- |
| `.claude/commands/` (ใน project) | project นี้เท่านั้น แชร์กับทีมผ่าน git ได้ |
| `~/.claude/commands/` (home) | ทุก project ของเราเอง |

---

## Custom Subagents — กำหนด Agent ไว้ล่วงหน้า

นอกจาก Slash Commands แล้ว ยังสร้าง **Subagent สำเร็จรูป** ไว้ใน `.claude/agents/` ได้ ซึ่งกำหนดได้ทั้ง model, tools, permissions และ prompt เต็มรูปแบบ

### วิธีสร้าง

สร้างไฟล์ `.md` ใน `.claude/agents/` เป็น Markdown + YAML frontmatter:

ไฟล์ `.claude/agents/security-reviewer.md`:

```markdown
---
name: security-reviewer
description: Security review specialist. ตรวจช่องโหว่ security ใน code changes.
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a senior security engineer reviewing code for vulnerabilities.

When invoked:
1. Run git diff to see recent changes
2. Analyze for OWASP Top 10 vulnerabilities
3. Check for sensitive data exposure, injection, auth issues
4. Report findings with severity level and file/line references
```

### Frontmatter ที่กำหนดได้

| Field              | คืออะไร                              | ตัวอย่าง                          |
| ------------------ | ------------------------------------ | --------------------------------- |
| `name`             | ชื่อ agent (ต้องมี)                   | `security-reviewer`               |
| `description`      | อธิบายว่าใช้เมื่อไหร่ (ต้องมี)         | `"Security review specialist..."` |
| `tools`            | tools ที่อนุญาต                       | `Read, Grep, Glob, Bash`         |
| `disallowedTools`  | tools ที่ห้ามใช้                       | `Write, Edit`                     |
| `model`            | model ที่ใช้                          | `sonnet`, `opus`, `haiku`         |
| `permissionMode`   | ระดับ permission                      | `default`, `plan`, `dontAsk`      |
| `maxTurns`         | จำนวน turns สูงสุด                    | `10`                              |
| `isolation`        | ทำงานใน git worktree แยก              | `worktree`                        |
| `background`       | รันเป็น background task เสมอ           | `true`                            |

### วิธีใช้

Claude จับคู่จาก `description` อัตโนมัติ หรือสั่งตรงๆ ก็ได้:

```text
# อัตโนมัติ — Claude เลือก agent ที่เหมาะเอง
Review this code for security issues

# สั่งตรง — ระบุชื่อ agent
Use the security-reviewer agent to check this PR
```

### ที่เก็บ Subagent Files

| ที่เก็บ                     | ใช้ได้กับ                                  |
| --------------------------- | ------------------------------------------ |
| `.claude/agents/` (ใน project) | project นี้เท่านั้น แชร์กับทีมผ่าน git ได้    |
| `~/.claude/agents/` (home)     | ทุก project ของเราเอง                       |

### Subagents vs Slash Commands vs Teams

|                    | **Subagents** (`.claude/agents/`) | **Slash Commands** (`.claude/commands/`) | **Agent Teams**              |
| ------------------ | --------------------------------- | ---------------------------------------- | ---------------------------- |
| **คืออะไร**        | AI agent แยกตัว มี context เป็นของตัวเอง | prompt ที่รันใน conversation หลัก           | กลุ่ม agent ทำงานพร้อมกัน     |
| **กำหนดได้**       | model, tools, permissions, prompt | แค่ prompt                                | สร้างจาก subagents หลายตัว    |
| **Context**        | แยก (เริ่มใหม่ทุกครั้ง)             | ใช้ร่วมกับแชทหลัก                          | แต่ละตัวแยก แต่คุยกันได้       |
| **เหมาะกับ**       | งาน self-contained (review, debug) | คำสั่งสั้นๆ ที่ต้องการ context แชท           | งานซับซ้อนที่ต้องประสานหลายมุม |

---

## ตัวอย่างจริง: ติดตามงาน Odoo บน GitHub

### สถานการณ์

ต้องตรวจสอบ issues/PRs ใน GitHub repos ที่เกี่ยวกับ Odoo ของ `monthop-gmail` เป็นประจำ ว่าเรื่องราวไปถึงไหน ต้องทำอะไรต่อ

### Repos ที่เกี่ยวข้อง

**Primary:**

| Repo | รายละเอียด |
| ---- | ---------- |
| `odoo-addons` | Accsumana Odoo Addons - Thai Accounting meta-module |
| `odoo-19-migration-guide` | คู่มือ migration Odoo 18 → 19 |
| `lawform-odoo` | Legal Forms — แบบฟอร์มศาลสำหรับ Odoo 18 |
| `webkul-addons` | Webkul Marketplace addons (v15, v18, v19) |

**Related (modules/localization):**

| Repo | รายละเอียด |
| ---- | ---------- |
| `l10n-thailand` | Thai localization |
| `server-ux` | Server UX modules |
| `partner-contact` | Partner and Contact addons |

### วิธีที่เลือกใช้: Slash Command + Subagent

เลือกใช้ **Slash Command** เรียก **Subagent** เพราะ:
- สั่งสั้น (`/check-odoo`) — ไม่ต้องพิมพ์ยาวทุกครั้ง
- Subagent ทำงานแยก context — ไม่รบกวนแชทหลัก
- กำหนด model, tools, prompt ไว้ครบ — ผลลัพธ์สม่ำเสมอ
- Agent Team overkill สำหรับงานนี้ — เพราะเป็นงาน read-only ตัวเดียวทำได้

### ไฟล์ที่สร้าง

**1. Subagent** — `.claude/agents/odoo-tracker.md`

```markdown
---
name: odoo-tracker
description: ตรวจสอบสถานะ issues/PRs ใน Odoo repos ของ monthop-gmail
tools: Read, Grep, Glob, Bash
model: sonnet
---

ตรวจสอบทุก repo ที่เกี่ยวกับ Odoo แล้วสรุปภาพรวม
พร้อมแนะนำ action ถัดไปเรียงตามความเร่งด่วน
```

**2. Slash Command** — `.claude/commands/check-odoo.md`

```markdown
ใช้ odoo-tracker agent ตรวจสอบสถานะ issues/PRs
ถ้ามี argument ให้กรองเฉพาะ repo นั้น: $ARGUMENTS
```

### วิธีใช้

```text
/check-odoo              # ดูทุก repo
/check-odoo lawform-odoo # ดูเฉพาะ repo นี้
```

### เมื่อไหร่ควรเปลี่ยนไปใช้ Agent Team?

ถ้าในอนาคต repo เยอะขึ้นมากหรืองานซับซ้อนขึ้น เช่น:
- ต้องการ agent ตัวนึงดู issues, อีกตัวดู PRs, อีกตัวสรุปภาพรวม
- ต้องการให้ agents ถกกันว่าควร prioritize อะไรก่อน

ค่อยยกระดับเป็น Agent Team ได้ทีหลัง

---

## Quick Start

```bash
# 1. เปิด experimental flag
# เพิ่มใน ~/.claude/settings.json:
# "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" }

# 2. ตรวจ version
claude --version  # ต้อง v2.1.32+

# 3. (ถ้าใช้ tmux) เริ่ม tmux session
tmux new -s my-team

# 4. รัน Claude Code
claude --teammate-mode tmux

# 5. สั่งสร้างทีม
# "Create an agent team to review PR #42 with 3 reviewers..."
# หรือถ้าสร้าง slash command ไว้แล้ว:
# /review-pr 42
```
