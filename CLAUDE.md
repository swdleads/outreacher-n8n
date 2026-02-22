# N8N Workflow Builder

This project uses Claude Code to build production-ready N8N workflows. You have access to the **n8n-mcp** MCP server for node discovery, validation, and workflow management, and **n8n-skills** for expert guidance on building workflows correctly.

## Setup

### n8n-mcp Server

Add the MCP server to connect Claude to your N8N instance:

```bash
claude mcp add n8n-mcp \
  -e MCP_MODE=stdio \
  -e LOG_LEVEL=error \
  -e DISABLE_CONSOLE_OUTPUT=true \
  -e N8N_API_URL=<YOUR_N8N_CLOUD_URL> \
  -e N8N_API_KEY=<YOUR_API_KEY> \
  -- npx n8n-mcp
```

Or use the project `.mcp.json` — copy `.mcp.json.example` to `.mcp.json` and fill in your credentials.

Verify with: `claude mcp list` or `/mcp` in a conversation.

### n8n-skills Plugin

Install the skills that teach Claude n8n workflow patterns:

```
/plugin install czlonkowski/n8n-skills
```

This adds 7 skills that auto-activate based on context: Expression Syntax, MCP Tools Expert, Workflow Patterns, Validation Expert, Node Configuration, Code JavaScript, and Code Python.

---

## Workflow Building Process

When the user asks you to build a workflow, follow these steps in order:

### 1. Understand
- Clarify the user's requirements before building anything
- Identify: trigger type, data sources, actions, expected output, edge cases
- Ask about error notification preferences (Slack, email, etc.)

### 2. Research
- Use `search_nodes` to find appropriate nodes for the task
- Use `get_node` (with mode: info, docs, or full) to understand node capabilities and required properties
- Use `search_templates` to find similar existing workflows that can serve as a starting point
- Use `get_template` to examine promising templates in detail

### 3. Design
- Choose the right architectural pattern:
  - **Webhook** — event-driven, real-time processing
  - **Scheduled** — recurring tasks, polling, batch processing
  - **HTTP API** — request/response integrations
  - **Database** — CRUD operations, data sync
  - **AI** — LLM chains, agents with tools, RAG pipelines
- Plan the node sequence and data flow before building
- Identify where error handling and branching are needed

### 4. Build
- Create the workflow via `n8n_create_workflow`
- Add and configure nodes incrementally with `n8n_update_partial_workflow`
- Use `validate_node` to check individual node configurations as you go

### 5. Validate
- Run `n8n_validate_workflow` on the complete workflow
- Use `n8n_autofix_workflow` to automatically fix common issues
- Fix any remaining validation errors manually

### 6. Test
- Use `n8n_test_workflow` to execute the workflow with sample data
- Verify the output matches expectations
- Check error paths work correctly

### 7. Document
- Set a clear, descriptive workflow name
- Add a workflow description explaining its purpose
- Add Sticky Note nodes to document complex logic sections
- Name every node descriptively (not "HTTP Request" but "Fetch User Profile")

---

## MCP Tools Quick Reference

### Node Discovery
| Tool | Purpose |
|------|---------|
| `search_nodes` | Find nodes by keyword |
| `get_node` | Get node details — modes: `info`, `docs`, `search_properties`, `versions` |

### Validation
| Tool | Purpose |
|------|---------|
| `validate_node` | Check individual node config — profiles: `runtime`, `ai-friendly`, `strict` |
| `n8n_validate_workflow` | Validate a complete workflow by ID |
| `n8n_autofix_workflow` | Auto-fix common workflow issues |

### Workflow Management
| Tool | Purpose |
|------|---------|
| `n8n_create_workflow` | Create a new workflow |
| `n8n_update_partial_workflow` | Incremental updates (17 operation types incl. `activateWorkflow`) |
| `n8n_test_workflow` | Execute workflow for testing |
| `n8n_deploy_template` | Deploy a template to the n8n instance |
| `n8n_workflow_versions` | Version history and rollback |
| `n8n_executions` | View and manage workflow executions |

### Templates & Guides
| Tool | Purpose |
|------|---------|
| `search_templates` | Search by keyword, nodes, task, or metadata |
| `get_template` | Get full template details |
| `ai_agents_guide` | Guidance for building AI agent workflows |
| `tools_documentation` | Meta-docs for all available MCP tools |

---

## Best Practices

### Error Handling
- Every workflow MUST have error handling — no exceptions
- Create a companion Error Workflow using the Error Trigger node for alerts (Slack, email, etc.)
- Enable **Retry on Fail** on nodes that call external APIs (3-5 retries with exponential backoff)
- Use **Continue on error output** for batch processing so one failure doesn't stop the entire workflow
- Never leave error output paths unconnected

### Naming & Organization
- Workflow names must clearly convey purpose (e.g., "Sync Stripe Payments to Airtable" not "My Workflow")
- Prefix sub-workflows: `Sub - Validate Email`, `Module - Format Date`
- Name every node descriptively — avoid defaults like "HTTP Request" or "IF"
- Add Sticky Note nodes to explain complex logic sections

### Modularity
- Refactor when a workflow exceeds ~30 nodes — break into sub-workflows
- Extract reusable logic into shared sub-workflows
- Don't duplicate the same logic across multiple workflows

### Node Selection
- Always prefer a dedicated built-in node over HTTP Request (e.g., use the Slack node, not an HTTP Request to Slack's API)
- Use **Set** node for data transformation and field mapping
- Use **IF** / **Switch** for branching logic
- Use **Code** node sparingly — prefer no-code nodes for 95% of cases
- When a Code node is needed, prefer JavaScript over Python (Python in n8n has no external library support)

### Expressions
- **Critical:** Webhook data lives under `$json.body`, not directly on `$json`
- Use proper `{{ }}` expression syntax
- Test expressions with sample data before relying on them

### Code Node Rules (when needed)
- Always return data in `[{json: {...}}]` format
- Use `$helpers.httpRequest()` for HTTP calls within Code nodes
- Use n8n's built-in DateTime handling
- Keep code simple — if it's getting complex, consider breaking into multiple nodes

### Security
- Never hardcode credentials — always use n8n's credential system
- Never modify production workflows directly — create a copy first
- Export backups of important workflows before making AI-driven changes
- Validate and test all changes before activating in production
- Sanitize user inputs in webhook-triggered workflows

---

## Quality Checklist

Before considering a workflow complete, verify ALL of the following:

- [ ] Error handling is in place (Error Workflow or error output paths)
- [ ] All nodes have descriptive names
- [ ] Workflow has a clear name and description
- [ ] Complex sections have Sticky Note documentation
- [ ] `n8n_validate_workflow` passes with no errors
- [ ] `n8n_test_workflow` executes successfully
- [ ] Credentials use n8n's credential system (nothing hardcoded)
- [ ] Workflow is idempotent where applicable (safe to re-run)

---

## Skills Reference

The n8n-skills plugin provides 7 skills that activate automatically based on your query:

| Skill | When It Activates |
|-------|-------------------|
| **n8n MCP Tools Expert** | Any MCP tool usage (highest priority) |
| **n8n Workflow Patterns** | Designing workflow architecture |
| **n8n Node Configuration** | Setting up node properties and connections |
| **n8n Expression Syntax** | Writing `{{ }}` expressions, accessing `$json`/`$node` variables |
| **n8n Validation Expert** | Interpreting and fixing validation errors |
| **n8n Code JavaScript** | Writing JavaScript in Code nodes |
| **n8n Code Python** | Writing Python in Code nodes |

For complex workflows, multiple skills compose automatically. The typical flow:
**Workflow Patterns** (architecture) → **MCP Tools Expert** (node discovery) → **Node Configuration** (setup) → **Expression Syntax** (data mapping) → **Validation Expert** (verification)
