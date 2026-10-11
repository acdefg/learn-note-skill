# learn-note skill — 移植包

把 `learn-note` 这个 Claude Code skill 装到另一台机器上。

**这个包是什么**：一个 skill 的源码 + Obsidian 知识库的骨架模板。**不含**任何已写好的笔记内容——那些在 vault 里，要么同步过去，要么在新机器上从零开始写。

**这个包是脱敏版**：所有路径都是占位符，不含任何个人目录、用户名或具体研究领域的痕迹。

---

## 装什么、装到哪

| 内容 | 装到 |
|---|---|
| `skills/learn-note/SKILL.md` | `<家目录>/.claude/skills/learn-note/SKILL.md` |
| `templates/TAXONOMY.md` | `<vault>/study_note/cc_notes/TAXONOMY.md`（**新 vault 才需要**） |
| `templates/INDEX.md` | `<vault>/study_note/cc_notes/INDEX.md`（**新 vault 才需要**） |

- Windows 家目录：`C:\Users\<用户名>`
- macOS / Linux 家目录：`/Users/<用户名>` 或 `/home/<用户名>`

> Claude Code 也认 `~/.agents/skills/`（跨 runtime 别名）。用 `.claude/skills/` 就行。

---

## 步骤

### 1. 装 skill

**Windows (PowerShell)**
```powershell
$src = "解压后的目录"                    # 例：C:\Users\me\Downloads\learn-note-skill
$dst = "$env:USERPROFILE\.claude\skills\learn-note"
New-Item -ItemType Directory -Force -Path $dst | Out-Null
Copy-Item "$src\skills\learn-note\SKILL.md" "$dst\SKILL.md" -Force
```

**macOS / Linux / Git Bash**
```bash
SRC=~/Downloads/learn-note-skill          # 改成解压后的目录
mkdir -p ~/.claude/skills/learn-note
cp "$SRC/skills/learn-note/SKILL.md" ~/.claude/skills/learn-note/SKILL.md
```

### 2. 填上本机的 vault 路径

`SKILL.md` 里笔记根目录写的是占位符 `<VAULT>`（**只有这一处需要填**）：

```
<VAULT>/study_note/cc_notes/
```

改成你本机的真实路径即可，例如 `/home/me/notes/study_note/cc_notes/`。改这一行就够——下文提到 `cc_notes\` 的地方都指它。

跑这段（把 `NEW_ROOT` 改成本机路径）：

```bash
python - <<'PY'
import pathlib
NEW_ROOT = r"/home/me/notes/study_note/cc_notes"     # ← 唯一要改的地方
p = pathlib.Path.home() / ".claude" / "skills" / "learn-note" / "SKILL.md"

# newline="" 在两个方向都关掉换行符转换。少了它，Windows 上读→写会把全文的
# LF 静默改成 CRLF——内容看着一模一样，但每一行都变了。
with open(p, "r", encoding="utf-8", newline="") as f:
    s = f.read()

before = s
# 替换串结尾不带斜杠——SKILL.md 里两处写法不同，带斜杠只会换掉一处：
#   <VAULT>/study_note/cc_notes/      ← 根目录那一行（带斜杠）
#   cd "<VAULT>/study_note/cc_notes"  ← 标签统计命令（不带）
s = s.replace("<VAULT>/study_note/cc_notes", NEW_ROOT)
assert s != before, "没匹配上——确认该行还是 <VAULT> 占位符"

with open(p, "w", encoding="utf-8", newline="") as f:
    f.write(s)
print("剩余占位符：", s.count("<VAULT>"))
PY
```

跑完 `剩余占位符： 0` 才算对。若还剩 1 个，就是替换串写错了（多半是自己加了结尾斜杠）。

### 3. 验证

在新机器的 Claude Code 里敲：

```
/learn-note
```

能触发（而不是报"未知命令"）就装好了。然后随便丢一个问答让它整理，看它**报告**有没有落到你设的 `cc_notes` 路径下。

> 若没触发：确认文件在 `<家目录>/.claude/skills/learn-note/SKILL.md`（**目录名和文件名都要对**），然后**重启 Claude Code**——技能列表在启动时加载。

### 4. （可选）新 vault 从零开始

如果新机器上没有 `cc_notes/`，建出来并把模板放进去：

```bash
VAULT="改成你的 vault 路径"
mkdir -p "$VAULT/study_note/cc_notes"
cp templates/TAXONOMY.md "$VAULT/study_note/cc_notes/TAXONOMY.md"
cp templates/INDEX.md    "$VAULT/study_note/cc_notes/INDEX.md"
```

- **`TAXONOMY.md` 必须放**——skill 把它当作**分类的唯一权威**，每次写笔记前都读它。没有它，skill 会拒绝即兴造文件名，等于没法归类。
- 模板里的分类表是**格式示例**（`<领域A>` 这种），第一次用时换成你自己的领域，把示例行删掉。领域是慢慢长出来的，别一次造一堆。
- `INDEX.md` 里是空的 `## 待填` 占位，第一条笔记写进去时会被替换成正式分类。

> **如果新机器的 vault 和旧机器是同一个**（比如放在同步盘里），跳过这一步，`TAXONOMY.md` / `INDEX.md` 已经在里面了。

---

## 这个 skill 会做什么

触发后它会把一段问答/PDF/博客整理成 Obsidian vault 里的**一个条目**，规则包括：

- 双层结构：速查卡（≤5 行）+ 正文 + 自测题
- 归类以 `TAXONOMY.md` 为唯一权威，归类决定必须报告给你
- 标签纪律：先 grep 已有词表再建新标签，新标签必须报告
- `INDEX.md` 的条目数与标签表**实算**，不累加
- 改文件名前后跑双链死链检查

完整的规则在 `skills/learn-note/SKILL.md` 里，那就是权威版本。

---

## 没包含的东西

| 没包含 | 说明 |
|---|---|
| 已写好的笔记 | 在 vault 里，不在这个包里 |
| 你的领域分类 | 被脱敏了。想延续原来的分类，自己把旧的 `TAXONOMY.md` 拷过去 |
| Obsidian 本体 | 需要另外装；`[[双链]]` 和标签面板靠它 |
| 其他 skill | 本包只带 `learn-note` 一个 |
