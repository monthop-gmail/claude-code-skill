# Claude Code Skill

รวมคู่มือการใช้งาน Claude Code ภาษาไทย สำหรับทีมพัฒนา — เรียนรู้ เพิ่มเติม แชร์กันต่อได้

## คู่มือ

| หัวข้อ | เนื้อหา |
| ------ | ------- |
| [Agent Teams](claude-code-agent-teams.md) | Agents, Subagents, Teams, Custom Slash Commands, Custom Subagents พร้อมตัวอย่างจริง |
| [GitHub Integration](claude-code-github-integration.md) | PR status, @claude review, GitHub Actions, Worktrees, Session management |

## ตัวอย่าง Slash Commands & Subagents

โปรเจกต์นี้มีตัวอย่างพร้อมใช้ใน `.claude/`:

```
.claude/
├── agents/
│   └── odoo-tracker.md       # Subagent: ตรวจสอบ issues/PRs ใน Odoo repos
└── commands/
    ├── check-odoo.md          # /check-odoo — เรียก odoo-tracker agent
    └── review-pr.md           # /review-pr 42 — review PR 3 มุม (security, performance, test)
```

## วิธีใช้

1. Clone repo นี้
2. เปิด Claude Code ใน directory นี้
3. ลองสั่ง `/check-odoo` หรือ `/review-pr <number>`

## ร่วมเพิ่มเติม

เพิ่มคู่มือใหม่ได้เลย — สร้างไฟล์ `.md` แล้วเพิ่มลิงก์ในตารางด้านบน
