GHCP Impact Analyzer - Terminal and Agent Workflow

This repository does not use Streamlit and does not require a GitHub PAT in the
Python application. The Python script only compares local Git refs and writes the
runtime files needed by the GHCP/IDE agents.

Run the analyzer
----------------
Open a terminal in this repository and run:

    python impact_analyzer.py

By default, this compares the current working tree/HEAD against the default base
ref found in this order:

    origin/main, origin/master, main, master

To compare a specific branch or ref:

    python impact_analyzer.py --base origin/main --target your-branch-name

To ignore staged, unstaged, and untracked local changes:

    python impact_analyzer.py --committed-only

Generated runtime files
-----------------------
The analyzer writes:

    runtime/impact-report.json
    runtime/trace-agent-input.json
    runtime/trace-agent-prompt.md
    runtime/impacts-facts.json
    runtime/copilot-agent-prompt.md

The key first output for the Trace Agent is:

    runtime/trace-agent-input.json

Run the Trace Agent from GHCP/IDE
---------------------------------
After running `python impact_analyzer.py`, open GHCP or your IDE agent panel and
run the custom agent named:

    trace_impact

Use the prompt in:

    runtime/trace-agent-prompt.md

The Trace Agent should read `runtime/trace-agent-input.json`, follow direct and
indirect step-definition call chains, and write:

    runtime/impacts-facts.json

The Trace Agent is designed to catch cases such as:

    Feature step -> step definition -> page/helper method -> changed method/class

For example, if a step definition calls `performfooter.clicksave()` and the
changed code is inside `clicksave()`, the scenario using that step should be
listed as impacted.

If the Trace Agent prints JSON but does not write the file, save that JSON into:

    runtime/trace-agent-response.json

Then run:

    python impact_analyzer.py --save-trace-response runtime/trace-agent-response.json

Optional RBT subset agent
-------------------------
After `runtime/impacts-facts.json` exists, run the custom agent named:

    copilot_agent_prompt

Use the prompt in:

    runtime/copilot-agent-prompt.md

It should write:

    runtime/copilot-regression-subset.json

If the subset agent prints JSON but does not write the file, save that JSON into:

    runtime/copilot-response.json

Then run:

    python impact_analyzer.py --save-copilot-response runtime/copilot-response.json

Notes
-----
- No Streamlit server is needed.
- No GitHub PAT is required by the Python script.
- `runtime/` is generated locally and intentionally ignored by Git.
