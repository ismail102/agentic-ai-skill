# Claude Skills Collection

A collection of custom [Agent Skills](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills) for Claude — reusable, self-contained instruction packages that teach Claude specialized workflows. Upload a skill once, and Claude automatically applies it whenever your request matches, or you can invoke it directly as a slash command (e.g., `/write-resume`).

Skills in this repo follow the [Agent Skills open standard](https://code.claude.com/docs/en/skills), so they work across Claude.ai (web/desktop/mobile), Claude Code, and the Claude API.

---

## What is a skill?

A skill is a folder containing, at minimum, a `SKILL.md` file with YAML frontmatter (`name` and `description`) followed by Markdown instructions. Skills can also bundle supporting material:

```
skill-name/
├── SKILL.md            # required — frontmatter + instructions
├── assets/             # optional — templates, source files used in output
├── references/         # optional — docs Claude loads only when needed
└── scripts/            # optional — executable code Claude can run
```

Claude uses **progressive disclosure**: it always sees each skill's name and description, reads the full `SKILL.md` only when the skill is relevant, and opens bundled files only when the instructions point to them. This keeps skills cheap in context while allowing them to be arbitrarily deep.

Two rules that trip people up:

- The `name` in frontmatter must be lowercase letters, numbers, and hyphens only (max 64 chars), and it should match the folder name.
- The `description` (max 200 chars for Claude.ai upload) is the **triggering mechanism** — it must say both *what the skill does* and *when to use it*, including the phrases a user would actually type.

---

## Installation

### Claude.ai (web / desktop / mobile)

1. Download a skill from this repo (each skill is a folder; zip it so the skill folder is the single top-level entry, e.g., `write-resume.zip` containing `write-resume/SKILL.md`, ...).
2. In Claude, go to **Settings → Capabilities → Skills** (make sure **Code execution and file creation** is enabled — skills require it).
3. Upload the `.zip` (or `.skill`) file.
4. Toggle the skill on. It's now active in all your chats.

Notes:
- Custom skills are private to your account. On Team/Enterprise plans, sharing must be enabled by an org owner (**Organization settings → Skills**), after which you can share skills with colleagues or the whole org.
- To update a skill, delete it and re-upload the new zip.
- Official guide: [Use skills in Claude](https://support.claude.com/en/articles/12512180-use-skills-in-claude)

### Claude Code

Copy the skill folder into your skills directory:

```bash
# Project-scoped (shared with anyone using the repo)
mkdir -p .claude/skills
cp -r write-resume .claude/skills/

# Or user-scoped (available in all your projects)
mkdir -p ~/.claude/skills
cp -r write-resume ~/.claude/skills/
```

The skill becomes available as `/write-resume`, and Claude can also load it automatically when a task matches its description. Docs: [Extend Claude with skills](https://code.claude.com/docs/en/skills).

### Claude API

Upload the zipped skill via the Skills API and reference its `skill_id` in the `container` parameter of a Messages request (requires the code-execution tool; up to 8 skills per request). Docs: [Using Agent Skills with the API](https://platform.claude.com/docs/en/build-with-claude/skills-guide).

---

## How to use a skill

Once installed, there are two ways to invoke any skill:

1. **Slash command** — type `/skill-name` followed by your input:
   ```
   /write-resume for https://example.com/careers/job-posting-123
   ```
2. **Natural language** — just describe the task; Claude matches it to the skill's description automatically:
   ```
   Tailor my resume for this job description: [pasted JD]
   ```

If a skill isn't firing when you expect, ask Claude directly: *"When would you use the `skill-name` skill?"* — its answer tells you how it interprets the description, which is what you'd edit to fix triggering.

---

## Skills in this repo

> **A note on personalization:** several of these skills bundle *my* documents (resume, cover letter template) as assets. To use them for yourself, replace the files in each skill's `assets/` folder with your own and adjust any personal facts referenced in `SKILL.md` (see [Adapting these skills](#adapting-these-skills-for-yourself)).

### 📄 `write-resume`

Tailors a LaTeX resume to a specific job description without inventing credentials. Produces paste-ready LaTeX for targeted sections, a change-rationale table, and — most importantly — a **Gaps list** of JD requirements the resume doesn't support, so keyword matching never turns into fabrication.

**Input:** just a job description (pasted text or URL). The resume is bundled.

**Example prompts:**
```
/write-resume for https://company.com/careers/ml-research-intern
```
```
Tailor my resume for this JD: [paste the full job description]
```
```
Which keywords am I missing for this posting? [paste JD]
```

### ✉️ `write-cover-letter`

Writes a complete, compilable one-page LaTeX (moderncv) cover letter from a job description. Uses the bundled resume as the **fact source** and the bundled cover letter as a **style/structure template only** — facts in an old letter never silently migrate into a new one. Outputs a `.tex` file, a section-to-JD mapping table, and a gaps/flags list.

**Input:** just a job description (pasted text or URL).

**Example prompts:**
```
/write-cover-letter for https://company.com/careers/ai-lab-intern
```
```
Write a cover letter for this position: [paste JD]
```
```
I just tailored my resume for the Workato role — now draft the matching cover letter.
```

### 📋 `cv-to-resume`

Converts a full academic CV into a tailored one-page resume for a specific job, in a fixed LaTeX format (paracol layout, teal accent).

**Input:** your CV (upload or paste) + the job description.

**Example prompts:**
```
Here's my CV and a job description — create a one-page resume for this position.
```
```
/cv-to-resume — CV attached, JD: [paste]
```

### ✍️ `humanizer`

Edits text to remove telltale signs of AI-generated writing — inflated symbolism, promotional filler, em-dash overuse, rule-of-three flourishes, vague attributions, and other patterns from Wikipedia's "Signs of AI writing" guide.

**Input:** any draft text.

**Example prompts:**
```
Humanize this paragraph: [paste text]
```
```
This abstract sounds AI-written — clean it up.
```

### 🔬 `research-paper-method`

Reviews, critiques, or structures research papers using a problem → prior work → limitation → proposal framework. Domain-agnostic (ML, systems, biology, social science, ...).

**Input:** a draft, an idea, or a paper to review.

**Example prompts:**
```
Review this introduction using the problem/prior-work/limitation framing: [paste]
```
```
I have a raw idea — help me frame it as a research contribution.
```
```
How does my method differ from baselines X, Y, and Z as a paper contribution?
```

---

## Adapting these skills for yourself

The document skills (`write-resume`, `write-cover-letter`, `cv-to-resume`) are personalized. To fork them:

1. **Replace the assets.** Swap the `.tex` files in `assets/` for your own resume/letter. Keep the filenames or update the paths referenced in `SKILL.md`.
2. **Update personal facts in `SKILL.md`.** These skills name specific anchor facts (e.g., "the most recent entries are the X and Y papers") used for freshness checks, plus known landmines (numbers or titles to verify). Rewrite those for your documents.
3. **Keep the grounding rule.** The core design of these skills is that *every claim must trace to your source documents* — that's what makes the output safe to send to employers. Don't delete the Gaps-list requirement; it's the most useful part of the output.
4. Re-zip and re-upload.

---

## Creating new skills

The fastest way: ask Claude to build one for you. Describe the workflow in a conversation (or perform it once manually), then say:

```
Turn this workflow into a skill I can reuse. Include the reference files I uploaded as assets.
```

Guidelines that make skills work well (learned the hard way):

- **Make the description pushy and specific.** List the actual phrases users will type. Claude under-triggers vague descriptions.
- **One skill, one job.** A skill that does resume tailoring *and* cover letters *and* interview prep triggers unreliably. Split it.
- **Push detail into `references/`.** Keep `SKILL.md` under ~500 lines; move exhaustive format specs, command inventories, and edge-case docs into reference files that `SKILL.md` points to.
- **Bundle templates as `assets/`, not as prose.** A real `.tex`/`.docx` template beats a description of one.
- **Explain *why*, not just *what*.** Instructions that state the reasoning ("fabricated bullets waste interview slots") generalize better than bare rules.
- **State the output format exactly.** If the skill should produce a file, a table, and a checklist, spell that structure out.
- **Test with 5 phrasings.** Try direct invocation, casual phrasing, and 2–3 unrelated requests (the skill should stay quiet on those). Fix the description based on misses.

Official authoring guide: [How to create custom skills](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills).

---

## Repo structure

```
.
├── README.md
├── write-resume/
│   ├── SKILL.md
│   ├── assets/
│   └── references/
├── write-cover-letter/
│   ├── SKILL.md
│   ├── assets/
│   └── references/
├── cv-to-resume/
│   └── SKILL.md ...
├── humanizer/
│   └── SKILL.md ...
└── research-paper-method/
    └── SKILL.md ...
```

Each folder is independently installable — zip any one folder and upload it.

## Contributing

Forks and adaptations welcome. If you build a variant (different resume template, different language, a new domain), open a PR or an issue. Please keep the grounding/anti-fabrication rules intact in any document-writing skill — they're the point.

## License

MIT — do whatever you like, no warranty. Replace personal documents in `assets/` with your own before use.
