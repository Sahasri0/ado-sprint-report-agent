# Sprint Report Agent

An AI-powered sprint report generator for Azure DevOps teams, built as a [Claude Code](https://docs.anthropic.com/en/docs/claude-code) custom slash command. Generate comprehensive, beautifully formatted sprint reports with a single command — no Python, no API keys, no setup scripts.

```
/sprint-report MyProject MyTeam
```

That's it. Claude fetches your sprint data from Azure DevOps, analyzes it, and generates a professional HTML report.

![Sprint Report Sample](examples/sample-report-preview.png)

---

## Features

- **Zero-config setup** — Just copy the command file and go. No Python environment, no dependencies, no `.env` files.
- **Live ADO data** — Fetches iterations, work items, capacities, PRs, and comments directly from Azure DevOps via MCP.
- **AI-powered analysis** — Claude analyzes sprint health, individual performance, risks, and team dynamics — not just raw numbers.
- **Beautiful HTML reports** — Clean, professional reports with KPI dashboards, color-coded status badges, and progress bars. Email-ready.
- **Smart metrics**:
  - Distinguishes **committed work** vs **mid-sprint additions** (bonus work is praised, not penalized)
  - **Defect RCA classification** — separates requirement-gap defects from real code defects
  - **PR activity** — shows actual code shipped per team member
  - **Wait gap detection** — identifies delays from work item comments and state transitions
  - **Carryover tracking** — highlights committed items that didn't complete
- **Per-member narratives** — AI-generated strengths, concerns, and status for each team member
- **Discussion synthesis** — Summarizes key decisions and collaboration from work item comments into a readable narrative

## Report Sections

| Section | What it shows |
|---------|--------------|
| KPI Dashboard | Velocity, commitment rate, bonus SP, defects resolved |
| Defect Analysis | RCA breakdown (requirement gap vs code defect) with detail table |
| Individual Performance | Stories, SP, tasks, hours, completion %, bonus — per member |
| Member Details | AI narrative, committed items, bonus items, tasks with status icons |
| Code Contributions | PRs merged per team member |
| Key Discussions | AI-synthesized narrative of sprint discussions and decisions |
| Risks & Recommendations | Side-by-side actionable insights |
| Team Dynamics | Separate analysis for development and testing teams |

---

## Prerequisites

1. **Claude Code CLI** — [Install Claude Code](https://docs.anthropic.com/en/docs/claude-code/getting-started)
2. **Azure DevOps MCP Server** — An MCP server that connects Claude Code to your Azure DevOps instance
3. **Azure DevOps PAT** — A Personal Access Token with the right scopes

### About Claude API / AI Usage

> **You do NOT need a separate Claude API key.** This tool runs inside Claude Code, which handles all AI interactions natively. Claude itself analyzes your sprint data and generates the report — there's no separate `CLAUDE_API_KEY` or Anthropic SDK involved. If you can run Claude Code, you can run this tool.

### Setting up the Azure DevOps PAT

You need a Personal Access Token (PAT) from Azure DevOps to authenticate API calls.

**How to create a PAT:**

1. Go to your Azure DevOps organization: `https://dev.azure.com/your-org`
2. Click your profile icon (top right) → **Personal access tokens**
3. Click **+ New Token**
4. Configure:
   - **Name**: `sprint-report-agent` (or anything you like)
   - **Organization**: Select your org
   - **Expiration**: Set as needed (max 1 year)
   - **Scopes** — select **Custom defined**, then enable:

| Scope | Access | Why it's needed |
|-------|--------|-----------------|
| **Work Items** | Read | Fetch sprint work items, states, story points |
| **Project and Team** | Read | List projects, teams, iterations |
| **Code** | Read | Fetch pull request activity |
| **Identity** | Read | Resolve team member identities |

5. Click **Create** and **copy the token** — you won't see it again.

> **Security tip**: Use the minimum scopes above. Never grant Full access. Store the PAT securely — never commit it to version control.

> **On-prem TFS/Azure DevOps Server**: If your org uses on-prem TFS, your URL will look like `https://tfs.yourcompany.com/tfs/YourCollection`. PAT creation is under the same profile menu. NTLM auth may also be available depending on your MCP server.

### Setting up the Azure DevOps MCP Server

Add an Azure DevOps MCP server to your Claude Code configuration. You can use any MCP server that provides Azure DevOps tools.

**Option A: Add via Claude Code settings**

In your Claude Code settings (`~/.claude/settings.json` or project-level `.claude/settings.json`):

```json
{
  "mcpServers": {
    "azure-devops": {
      "command": "npx",
      "args": ["-y", "azure-devops-mcp-server"],
      "env": {
        "ADO_ORG_URL": "https://dev.azure.com/your-org",
        "ADO_PAT": "your-personal-access-token-here"
      }
    }
  }
}
```

Replace:
- `https://dev.azure.com/your-org` with your actual Azure DevOps URL
- `your-personal-access-token-here` with the PAT you created above

> **Note**: The exact MCP server package name may vary. Check the [MCP server registry](https://mcp.so) for available Azure DevOps MCP servers. The slash command works with any ADO MCP server that provides the standard work item, iteration, and pull request tools.

**Option B: If your organization already has an ADO MCP server configured** in Claude Code, you're good to go — no additional setup needed.

### Verify your setup

Run Claude Code and try:

```
List all projects in my Azure DevOps organization
```

If Claude can list your projects, the MCP server is working and you're ready to generate reports.

---

## Installation

### Quick start (30 seconds)

1. **Copy the command file** into your project (or home directory):

```bash
# Option A: Project-level (only works in this repo)
mkdir -p .claude/commands
cp path/to/sprint-report.md .claude/commands/

# Option B: User-level (works everywhere)
mkdir -p ~/.claude/commands
cp path/to/sprint-report.md ~/.claude/commands/
```

2. **Run it**:

```
/sprint-report YourProject YourTeam
```

### From this repo

```bash
git clone https://github.com/YOUR_USERNAME/sprint-report-agent.git
cd sprint-report-agent

# Copy to your project
cp -r .claude/commands/sprint-report.md /path/to/your/project/.claude/commands/

# Or copy to user-level (available in all projects)
cp .claude/commands/sprint-report.md ~/.claude/commands/
```

---

## Usage

### Generate a report for the current sprint

```
/sprint-report MyProject MyTeam
```

### Generate a report for a specific sprint

```
/sprint-report MyProject MyTeam --sprint "Sprint 26.06"
```

### Team names with spaces

```
/sprint-report MyProject "My Team Name"
```

### What happens

1. Claude fetches the current (or specified) sprint iteration from Azure DevOps
2. Pulls all work items, capacities, PRs, and comments for that sprint
3. Classifies committed vs mid-sprint work, defect RCA, completion rates
4. Generates AI-powered analysis: health assessment, member narratives, risks, recommendations
5. Produces a self-contained HTML report file

### Output

The command generates an HTML file named `report_<team>_<sprint>.html` in your working directory. Open it in any browser — it's fully self-contained with inline CSS, no external dependencies.

---

## How It Works

This tool is a **Claude Code custom slash command** — a markdown prompt file that instructs Claude on how to:

1. **Collect data** using Azure DevOps MCP tools (iterations, work items, PRs, comments)
2. **Classify and compute metrics** (committed vs bonus work, completion rates, defect RCA)
3. **Analyze** the sprint using Claude's intelligence (no separate API call — Claude IS the analyzer)
4. **Generate** a professional HTML report

```
┌─────────────────────────────────────────────────────────┐
│                    Claude Code CLI                        │
│                                                           │
│  /sprint-report MyProject MyTeam                          │
│       │                                                   │
│       ▼                                                   │
│  ┌─────────────┐    ┌──────────────┐    ┌──────────────┐ │
│  │  ADO MCP     │───▶│  Claude AI    │───▶│  HTML Report  │ │
│  │  Server      │    │  Analysis     │    │  Generator    │ │
│  │              │    │              │    │              │ │
│  │ - Iterations │    │ - Health     │    │ - KPIs       │ │
│  │ - Work Items │    │ - Narratives │    │ - Tables     │ │
│  │ - PRs        │    │ - Risks      │    │ - Charts     │ │
│  │ - Comments   │    │ - Recommend  │    │ - Styling    │ │
│  └─────────────┘    └──────────────┘    └──────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Why a slash command instead of a Python app?

| | Python App | Slash Command |
|---|---|---|
| **Setup** | pip install, .env, API keys | Copy one file |
| **Dependencies** | Python, requests, anthropic, jinja2 | None |
| **Claude API key** | Separate `CLAUDE_API_KEY` required ($$$) | **No API key needed** — runs inside Claude Code |
| **ADO auth** | PAT config in .env + code changes | MCP server handles it (one-time setup) |
| **Maintenance** | Update packages, fix breaking changes | Just a prompt file — edit markdown |
| **Customization** | Edit Python code | Edit a markdown file |
| **Portability** | Clone repo, install deps, configure | Copy one `.md` file to any machine |

---

## Customization

The slash command is just a markdown file — edit it to customize:

- **Add/remove report sections** — Edit the "Output: Generate HTML Report" section
- **Change analysis guidelines** — Modify the "Analysis Guidelines" section
- **Adjust colors/styling** — Update the "Design System" section
- **Add custom fields** — Add your ADO custom fields to the data collection steps
- **Change scope-change tag** — Update `"Adopted after Sprint Lock"` to match your team's tag

### Common customizations

**Different scope-change tag:**
Find `"Adopted after Sprint Lock"` in the command and replace with your tag.

**Additional custom fields (e.g., custom defect fields):**
Add field names to the `wit_get_work_items_batch_by_ids` fields list in Step 3.

**Different work item types:**
Modify the "Work Item Types" classification in the command.

---

## Sample Report

Open [`examples/sample-report.html`](examples/sample-report.html) in your browser to see what the output looks like.

The sample shows:
- KPI dashboard with velocity, commitment, bonus SP, and defects
- Individual performance table with status badges
- Risk and recommendation cards

---

## Troubleshooting

### "No MCP tools found for Azure DevOps"
You need an Azure DevOps MCP server configured. See [Prerequisites](#prerequisites).

### "No current iteration found"
The team might not have a sprint configured as "current" in ADO. Use `--sprint` to specify one explicitly.

### "Work items batch returned empty"
Check that the sprint actually has work items assigned. The iteration might exist but have no items on the board.

### Report is missing PR data
The MCP server needs access to Git repositories in your ADO project. Ensure the PAT (or auth) has `Code (Read)` scope.

---

## Contributing

Contributions are welcome! This is a community tool — if you improve the analysis logic, report design, or add support for new data sources, please open a PR.

### Ideas for contributions
- Jira MCP support (adapt the command for Jira instead of ADO)
- GitHub Projects support
- Sprint-over-sprint trend analysis
- PDF export integration
- Slack/Teams notification integration
- Dark mode report template

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

## Author

Built by [Dimmula Sahasri](https://github.com/YOUR_USERNAME) as an exploration of AI-powered developer tools using Claude Code.

If this helped your team, give it a star and share it with other scrum masters and engineering managers!
