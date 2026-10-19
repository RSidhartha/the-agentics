# CrewAI Build Report

**Name:** Sidhartha Reddy Ravula
**Topic I used:** How small teams can use AI agents in data science

## 1. What I changed

I edited `crewai-kit/main.py` to add a fourth agent, a Fact Checker. I added a
`fact_checker` Agent with its own role, goal, and backstory, and a
`fact_check_task` Task that asks it to mark each factual claim in the draft as
VERIFIED, UNCERTAIN, or WRONG and then produce a corrected version. I inserted
both into the `agents` and `tasks` lists of the `Crew`, between the writer and
the editor, and reworded `editing_task` so the editor works on the
fact-checked draft.

I chose this change because the original editor only flags claims with
 and never questions them. I wanted to see whether a dedicated
agent whose only job is skepticism would change the content of the final
explainer, not just its style. It also fit the assignment's requirement of a
visible behavior change, since the extra step shows up in the terminal log and
may alter the final text.

## 2. How CrewAI works

CrewAI models work as a team. An agent is a role-playing worker defined by a
role, a goal, and a backstory, and these are fed to the model as context that
shapes how it behaves. A task is a unit of work: it has a description, an
expected output, and the agent assigned to it. A crew ties agents and tasks
together and decides how they run. In this kit the crew uses
`Process.sequential`, so tasks run in list order and each task receives the
previous task's output as context.

That is why my change worked. When I added `fact_check_task` after
`writing_task` in the `tasks` list, CrewAI automatically passed the writer's
draft to the Fact Checker, and passed the Fact Checker's corrected version to
the editor. I did not write any code to connect them. The order of the list is
the workflow, so the Fact Checker sits between the writer and the editor on
purpose: it needs a draft to check, and the editor should polish the corrected
text rather than the unchecked one. The `{topic}` placeholder in the goals and
descriptions is filled in by `crew.kickoff(inputs={"topic": ...})`, which is
how the same crew runs on any topic. Finally, `main.py` saves only
`result.raw`, which is the output of the last task, to `output/explainer.md`.

## 3. Evidence

Run 1 (`run1_original.txt`, `explainer_run1.md`) used the original
three-agent crew. Its final explainer is about 160 words in four short
paragraphs, below the 250-word target, with only two [source needed] flags. It
keeps every statistic from the research brief, including a 30-50% cut in
notebook-writing time, more than 85% EDA accuracy, a 60% compute saving at a
six-person consultancy, and 2-3 high-risk issues flagged per model. Only the
first two are flagged for a source, and the other two pass through unmarked.

Run 2 (`run2_changed.txt`, `explainer_run2.md`) used the same topic with the
Fact Checker added. Its final explainer is about 580 words, with section
headings, a "Key Take-aways" list, and 15  flags. The tone is
much more cautious. It calls the claimed 10x speed-up "promising but not yet
proven at scale," describes a 30% cloud-cost saving as "illustrative rather
than definitive," says a 40% onboarding figure comes from a single internal
survey with no external validation, and states that a cited NeurIPS 2023 paper
does not exist. In the terminal log, the Fact Checker's table covers 14 claims:
2 WRONG, 7 UNCERTAIN, 3 PARTIALLY VERIFIED, and 2 VERIFIED.

This is not a perfectly controlled comparison. The Researcher produced a
different brief in each run (run 1's statistics do not appear in run 2 at all),
because LLM output varies from run to run even at temperature 0.3. So some of
the difference comes from the research step, not only from the new agent. The
cleanest evidence of the Fact Checker's effect is the verdict table in
`run2_changed.txt`, which can be traced claim by claim to the draft it reviewed.
That table does not appear in `explainer_run2.md`, because `main.py` writes only
the final task's output to the file, and the verdicts belong to the third task,
not the last one.

**Surprise or problem:** The final explainer grew to more than twice the
250-word target. My `fact_check_task` asked for a corrected explainer plus a
verdict list, and neither it nor the editing task repeated the word limit, so
the length constraint was lost after the writing step. The Editor also left a
sentence in the explainer that reads like fact-checking commentary ("a paper
purportedly presented at NeurIPS 2023 ... does not exist") rather than
explainer content. I did not change the code after run 2, so that the two runs
stay comparable. To fix it, I would add "keep the final explainer to 250 words
and leave verdicts out of it" to the task descriptions. Separately, both logs
show a one-time "Invalid response from LLM call - None or empty" error at the
Technical Writer step, which CrewAI retried automatically and recovered from.

## 4. Choosing a framework

I would choose CrewAI for a project like mine because the work is a pipeline of
distinct roles, such as research, write, check, and edit, and CrewAI lets me
describe that as a team in plain language with very little code. Adding my
fourth agent took one agent block, one task block, and two list edits, with no
wiring between steps.

LangGraph would have been harder for this simple pipeline. I would have had to
define a shared state, write a function for each step, and add the edges
between them myself. In exchange, it would have made some things easier. The
graph is explicit, so I could add a conditional edge that sends the draft back
to the writer when the Fact Checker finds a WRONG claim, creating a revision
loop. In CrewAI's sequential process each task runs once, in order, so a loop
like that is not natural. The Agents SDK, by comparison, is built around a
single agent with tools and handoffs, which would suit a project where one
agent decides when to call specialists, but is less natural for a fixed
assembly line.

## 5. Verification and AI use

The kit has no web access, so every fact in the output comes from the model's
memory and could be wrong. I checked two claims from the run 2 output by
searching for them. First, I searched for the NeurIPS 2023 paper "Agent-Based
Fairness Audits." I found no paper with that title. The closest match is
"Fairness Auditing with Multi-Agent Collaboration," which appeared at ECAI 2024
(not NeurIPS 2023), and the NeurIPS 2023 fairness-auditing paper I found,
"Auditing Fairness by Betting," is unrelated to agents. So the Fact Checker was
right to mark that claim and its numbers (12% more violations, under 5% false
positives) as WRONG. Second, I searched for the quoted Stanford AI Index 2024
line about "shorter prototyping cycles" and could not find it, so I treat that
quotation as unverified even though the explainer still presents it in
quotation marks. I treat the Fact Checker's labels as the model's opinion, not
proof, since it is the same kind of model working from the same memory. It
caught one fabricated citation but left others in place.

I used AI assistance (Claude) throughout this assignment. It helped me
understand the instructions, explained how the starter kit works, drafted the
code for the Fact Checker agent and task, drafted the report text, and helped me
read my two run logs and look up the claims above. I edited the report and
verified the work myself. I confirmed the code ran and that all four agents
appeared in the terminal log in the correct order, I compared the explanations
against `main.py` and my run outputs, and I repeated the searches above to check
the claims.