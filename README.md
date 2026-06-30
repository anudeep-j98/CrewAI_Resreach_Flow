# Guide Generator Flow — Project Overview

An AI-powered pipeline built with CrewAI Flows that turns mixed research sources (YouTube, web pages, arXiv papers, local documents) into a single summarized document

---

## High-Level Idea

Learning a new framework or tool usually means jumping between videos, docs, blogs, and papers. This project automates that research and writing workflow:

1. You provide sources — URLs, arXiv links, and/or local file paths.
2. Research Crew — A manager agent delegates to specialist agents, each equipped with tools for their source type. They produce one large research report.
3. Writing Crew — A technical writer drafts a structured guide; a content editor reviews and polishes it.
4. Output — A markdown getting-started guide saved to `outputs/getting_started_guide.md`.

The project uses CrewAI Flow to chain these two crews in order, with shared state between steps.

---

## How It Works

### End-to-End Flow


User Input (CLI)
       │
       ▼
┌──────────────────────────────────────────────────────────────┐
│  GuidedGeneratorFlow (CrewAI Flow)                           │
│                                                              │
│  1. receive_user_input     → Validate at least one source    │
│  2. run_research_crew      → Hierarchical Research Crew      │
│  3. run_writing_crew       → Sequential Writing Crew         │
└──────────────────────────────────────────────────────────────┘
       │
       ▼
outputs/getting_started_guide.md


### Flow State (`ResearchFlowState`)

| Field | Purpose |
|-------|---------|
| `youtube_links` | YouTube video/channel URLs |
| `webpage_links` | Docs, blogs, articles |
| `research_paper_links` | arXiv URLs or queries |
| `document_paths` | Local PDF, TXT, MD paths |
| `research_report` | Output from Research Crew |
| `final_guide` | Output from Writing Crew |

### Crew 1: Research Crew (Hierarchical)

- Process: `Process.hierarchical` with `planning=True`
- Manager: `research_manager` — coordinates and compiles the final report
- Specialists (with tools):

| Agent | Tools | Sources |
|-------|-------|---------|
| YouTube Specialist | `YoutubeVideoSearchTool`, `YoutubeChannelSearchTool` | YouTube links |
| Web Specialist | `ScrapeWebsiteTool`, `SeleniumScrapingTool`, `CodeDocsSearchTool` | Web URLs |
| Arxiv Specialist | `ArxivPaperTool` | arXiv papers |
| Document Specialist | `FileReadTool`, `PDFSearchTool`, `TXTSearchTool`, `MDXSearchTool`, `DirectoryReadTool` | Local files |

- Task: `research_compilation` — delegate by source type, merge into one 5000+ word markdown report
- Output: Stored in `flow.state.research_report`

### Crew 2: Writing Crew (Sequential)

- Process: `Process.sequential`
- Agents:
  - Technical Writer — turns the research report into a structured beginner guide
  - Content Editor — reviews clarity, completeness, and formatting
- Tasks:
  1. `write_getting_started_guide` → draft guide from `{research_report}`
  2. `review_and_polish_guide` → final polish; writes to `./outputs/getting_started_guide.md`
- Output: Stored in `flow.state.final_guide` and saved as a file

---

## File Structure
```
guide_generator_flow/
├── pyproject.toml                          # Project config, dependencies, CLI scripts
├── README.md                               # Short run command reference
├── README_output.md                        # Captured terminal output from a sample run
├── readme_aboutProject.md                  # This file — project overview
│
├── outputs/
│   └── getting_started_guide.md            # Final generated guide (from Writing Crew)
│
└── src/
    └── guide_generator_flow/
        ├── main.py                         # Flow definition, CLI input, entry point
        │
        ├── crews/
        │   ├── research_crew/
        │   │   ├── research_crew.py        # Research agents, tools, hierarchical crew
        │   │   └── config/
        │   │       ├── agents.yaml         # Roles/goals for manager + 4 specialists
        │   │       └── tasks.yaml          # Research compilation task + prompts
        │   │
        │   └── writing_crew/
        │       ├── writing_crew.py         # Writer + editor, sequential crew
        │       └── config/
        │           ├── agents.yaml         # Technical writer & content editor
        │           └── tasks.yaml          # Write + review tasks (output_file set here)
        │
        └── tools/
            └── custom_tool.py              # Placeholder custom tool (not wired in yet)
```

---

## Key Files Explained

| File | Role |
|------|------|
| `main.py` | Defines `GuidedGeneratorFlow`, collects CLI input, runs both crews in sequence |
| `research_crew/research_crew.py` | Instantiates research agents, attaches `crewai-tools`, builds hierarchical crew |
| `writing_crew/writing_crew.py` | Builds sequential writer → editor pipeline |
| `*/config/agents.yaml` | Agent roles, goals, backstories (behavior without code changes) |
| `*/config/tasks.yaml` | Task descriptions, expected outputs, template variables |
| `pyproject.toml` | Dependencies: `crewai[google-genai,tools]==1.8.1`, `crewai-tools>=1.8.1` |

---

## How to Run

Prerequisites

- Python 3.10–3.13
- API keys in a `.env` file (e.g. for the configured LLM provider)
- Dependencies installed via `uv`

Run interactively

```uv run python src/guide_generator_flow/main.py```


The CLI prompts for YouTube links, web URLs, arXiv papers, and document paths (all optional, but at least one source type is required).

---

## Architecture Summary

| Layer | Technology | Responsibility |
|-------|------------|----------------|
| Orchestration | CrewAI Flow (`@start`, `@listen`) | Pipeline control and shared state |
| Research | Hierarchical Crew + planning | Multi-source gathering and synthesis |
| Writing | Sequential Crew | Draft → edit → save markdown |
| Configuration | YAML | Agent/task behavior externalized from Python |
| Tools | `crewai-tools` | YouTube, web scrape, arXiv, file/PDF search |

---

## Design Choices

- Flow over single crew — Research and writing are separate phases with clear handoff via `research_report`.
- Hierarchical research — A manager delegates only to specialists whose source type was provided.
- Sequential writing — Writer produces content first; editor refines without parallel conflict.
- YAML-driven prompts — Task structure and agent personas live in config files for easy tuning.
- Beginner-first output — Writing tasks enforce structure (Introduction → Prerequisites → First Project → Troubleshooting, etc.).

---

## Possible Extensions

- Wire `custom_tool.py` into an agent
- Add non-interactive input (JSON/CLI args)
- Support additional source types (GitHub repos, RSS, etc.)
- Add a `plot` flow visualization entry point (referenced in `pyproject.toml`)


---

### Quick summary

| Aspect | Detail |
|--------|--------|
| Purpose | Auto-generate beginner getting-started guides from mixed sources |
| Pattern | CrewAI Flow → Research Crew → Writing Crew |
| Input | YouTube, web, arXiv, local documents (CLI) |
| Output | `outputs/getting_started_guide.md` |
