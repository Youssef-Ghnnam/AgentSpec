# AgentSpec
Automating translation of NL or Technical language to agentic instructions for rapid and accessible building of AI agents.
_ _ _ _ _
Agent-of-Agents: Minimal Python scaffold
Windows / Python 3.13 compatible, stdlib-only.

What it does (MVP):
- Reads a user-friendly Agent Intake Form (JSON) -> compiles to AgentSpec.
- Validates with clear and user-facing messages.
- Runs a bounded agent loop with mock tools (search, fetch, summarize, write).
- Produces a brief markdown file and a sources.json (simulated).
- Computes simple QA metrics (confidence proxy, rule checks, self-consistency).

How to run:
1) Save an intake form as agent_form.json (example auto-generated on first run)
2) Optionally, save run inputs as run_input.json (example auto-generated)
3) Run: python agent_scaffold.py
or: python agent_scaffold.py --form agent_form.json --input run_input.json --out out

No external dependencies. All network/file actions are simulated.
Replace mock tools with real ones when ready.
