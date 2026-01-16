GitHub Repo: https://agentic-project-management.dev/docs/

```bash
npm install agentic-pm

cd <project-dir>

apm init
```

## Chat 1: Setup Agent
Prompt: `/apm-1-initiate-setup`
After working through project description, goals, this agent will create an `Implementation_Plan.md` and will output a *bootstrap prompt* for the manager agent. You can close this chat window when this is complete.
## Chat 2: Manager Agent
Prompt: `/apm-2-initiate-manager <bootstrap-prompt>`
This agent will confirm their understanding of the project and initialize the memory system `.apm/Memory/` and will output a *task assignment prompt* for the task agent
## Chat 3: Task Agent
Prompt `/apm-3-initiate-implementation <task-assignment-prompt>`
This agent will perform the task, update the relevant memory log, and output the final task report in the format
```
Task: [Task ID]
Memory Log: [path]
[Brief 1-2 sentence summary]
```
## Workflow
Copy output from task agent to manager agent for a new prompt for a new task.


## Tips & Takeaways
1. if you use budget model for Task Agent, REMEMBER TO SWITCH BACK for Manager Agent.
2. remember to start each new chat with the appropriate `/apm` instructions.
3. if the task is sufficiently modular, use alternative models to perform the task (outside of GitHub Copilot) and copy the output. then, update the final task report in the required format to feed back to manager agent.
4. spend time and care with Setup and Manager Agents.
5. review steps to ensure that each one is efficient in terms of token consumption (i.e. context + output is reasonable).

This consumes tokens VERY quickly. If the planning is all you need, use apm for that and then carefully ensure that you are copying the relevant files to perform the task outside of the IDE.