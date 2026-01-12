# AI Agent Policy

## Overview

AI coding agents (such as Claude Code, GitHub Copilot, Cursor, Cline, and similar tools) are permitted on the HTCF, subject to the policies below. These tools can automate coding tasks, generate scripts, and interact with files and the command line on behalf of users.

## Execution Requirements

**AI agents must run in interactive sessions or batch jobs.** Agents are prohibited from running on login nodes.

Agents may spawn subprocesses, make network requests, and perform file operations in ways that are unpredictable and resource-intensive. This type of activity violates the [login node policy](../policies.md#login-node-policy) and may impact other users.

### Interactive Sessions

For interactive agent use (development, debugging, exploration):

```bash
$ srun --mem=8G --cpus-per-task=1 -J interactive -p interactive --pty /bin/bash -l
```

Then start your agent within the session.

### Batch Jobs

For automated agent workflows:

```bash
#!/bin/bash

#SBATCH --job-name=ai_agent
#SBATCH --cpus-per-task=1
#SBATCH --mem=8G
#SBATCH --time=4:00:00
#SBATCH --output=agent_%j.out

# Set up your agent environment
# ...

# Run agent task
your-agent-command
```

!!! warning "Time Limits Required"
    Always specify a `--time` limit for agent batch jobs. Agents can enter loops or take longer than expected. A time limit ensures runaway processes are terminated.

## Storage

Agents should perform all work on `/scratch`. Results can be copied to `/lts` after review.

!!! tip "Review Before Archiving"
    Review agent-generated files before copying to long-term storage. Agents may create unexpected files or directory structures.

## Data Security

Per [WUSTL policies regarding data security](http://wustl.edu/policies/compolicy.html), do not process sensitive, protected, or proprietary data through AI agents that transmit data to external services.

This includes:

- Protected Health Information (PHI/HIPAA)
- Personally Identifiable Information (PII)
- Proprietary research data not cleared for external transmission
- Any data subject to data use agreements restricting third-party access

If your agent uses an external API (most do), assume that your prompts and file contents may be transmitted to that service.

## User Responsibility

Users are responsible for all actions performed by their AI agents, including:

- Files created, modified, or deleted
- Resources consumed
- Any policy violations

Treat agent actions as your own. Review agent output and changes before finalizing work.

## Recommended Practices

1. **Start with interactive sessions** - Test agent behavior before running in batch
2. **Use version control** - Commit work before running agents so changes can be reviewed or reverted
3. **Set resource limits** - Specify `--time`, `--mem`, and `--cpus-per-task` to bound agent resource usage
4. **Keep credentials secure** - Use environment variables for API keys; never hardcode them in scripts
5. **Monitor agent sessions** - Check on long-running agents periodically

## Example Use Cases

### Appropriate Uses

- Code development and debugging assistance
- Generating and refactoring scripts
- Automated testing workflows
- Documentation generation
- Data pipeline development (with non-sensitive data)

### Prohibited Uses

- Processing sensitive data through external AI services
- Running agents on login nodes
- Unattended agents without time limits
- Agents with unsupervised write access to shared directories

## Questions

Contact HTCF support if you have questions about whether a specific AI agent workflow is appropriate.
