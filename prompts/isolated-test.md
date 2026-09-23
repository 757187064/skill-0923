---
description: Test the isolated-test subagent (verifies no inherited system prompt)
argument-hint: "[task]"
---
Use the `subagent` tool in single mode to invoke the agent named `isolated-test`.

Pass this task to it:

> ${@:-Prove that you are an isolated subagent and that your final answer starts with ISOLATED:}

Then show me the subagent's final answer exactly as returned.
