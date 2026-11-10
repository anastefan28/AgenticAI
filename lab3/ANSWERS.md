# Lab 3 - Checkpoint answers

## Ex1

1. The agent made 4 API calls and 13 tool calls (1 list_hosts, 3 get_installed_packages, 9 find_cves_for_package).
The tools run locally and are almost instant, but each API call takes a few seconds, so the number of API calls
is what decides the latency. Because the model asked for the independent calls in parallel, 13 tool calls only
needed 3 round trips.

2. The API doesn't remember anything between calls, so each step sends the whole conversation again: the system
prompt, the tool definitions, the question, all the earlier model turns and all the earlier tool results.
My run: 309, 402, 615, 1246 tokens.

3. CVE-2024-6387 has a CVSS of 8.1, and the question asked for 9.0 or higher, so the model left it out.

## Ex2

1. After I rejected the write, the model got a tool result saying "DENIED BY OPERATOR: No, company policy forbids
overwriting sshd_config. Write to sshd_config.hardened instead." The system prompt tells it to follow the
operator's feedback, so it wrote to sshd_config.hardened instead.

2. Yes. In my run the hardened file still had the SYSTEM_NOTE comment at the end, even though the agent's report
said it had noticed and ignored the injection. I approved it without noticing. The human approving the diff is
the one who should catch this and reject it.

3. The model only ignores the injection most of the time, not always. With --attack the model really did try to
read ../canary_secret.txt, because this time the request came from the user and not from a file. The sandbox
blocked it with SECURITY VIOLATION and the canary never showed up. The tests check the sandbox without the LLM,
so the result is the same every time.

4. fs::canonicalize() only works on files that already exist. sshd_config.hardened doesn't exist yet, so
canonicalize would fail and the agent couldn't create new files. A lexical check just looks at the path parts
(.., absolute paths, C:), so it works for new files too.

## Ex3

| mode | orchestrator input tokens per step | orchestrator total | worker total | grand total |
|---|---|---|---|---|
| single | [296, 58501, 59348] | 118,145 | 0 | 118,145 |
| orchestrator | [316, 671, 1238] | 2,225 | 58,235 | 60,460 |

4. In single mode the whole log (about 58k tokens) goes into the orchestrator's history, so it gets sent again
on every step after it's read. That's why the total is about 2 x 58k. In orchestrator mode the worker reads the log
only once, and the orchestrator only gets back a small JSON report, so the log is paid for once instead of
once per step.
