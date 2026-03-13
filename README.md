# Python Backend Plugin for Claude Code

A Claude Code plugin that makes Python/FastAPI development faster and safer. It adds slash commands, an AI security auditor, automatic code formatting, and Python best-practice guidance directly into your Claude Code sessions.

---

## What's Included

| Component | What it does |
|---|---|
| `/generate-endpoint` | Scaffolds a full FastAPI endpoint with schemas, service layer, and tests |
| `/generate-test` | Writes a complete pytest test file for any existing module |
| `/review-module` | Reviews a Python file for bugs, style issues, and bad patterns |
| `@security-auditor` | AI agent that audits your code for security vulnerabilities |
| Hooks | Auto-formats, type-checks, runs tests, and scans for secrets automatically |
| Skill | Keeps Claude aligned to Python best practices whenever it writes `.py` files |

---

## Installation

### Step 1 — Make sure you have Claude Code installed

```bash
claude --version
# Should be 1.5.0 or higher
```

### Step 2 — Load the plugin

Claude Code loads local plugins using the `--plugin-dir` flag when you start a session:

```bash
claude --plugin-dir "A:/Projects/Plugins/python_plugin"
```

> Replace the path with wherever the `python_plugin` folder lives on your machine.

The plugin is active for that session. All commands, hooks, and the security auditor are available immediately.

---

### Optional: Avoid retyping the flag every time

**Shell alias (recommended for personal use)**

Add this to your shell profile (`.bashrc`, `.zshrc`, or PowerShell profile):

```bash
alias claude-py='claude --plugin-dir "A:/Projects/Plugins/python_plugin"'
```

Then just run `claude-py` instead of `claude` to get a session with the plugin loaded.

---

### Optional: Install permanently via a local marketplace

If you want `claude` itself to always load this plugin, you can set up a local marketplace.

**1. Create a marketplace index file** at `A:/Projects/Plugins/.claude-plugin/marketplace.json`:

```json
{
  "name": "Local Plugins",
  "plugins": [
    {
      "name": "python-backend-plugin",
      "description": "FastAPI scaffolding, security auditing, and Python best practices",
      "source": "./python_plugin"
    }
  ]
}
```

**2. Register the marketplace** (run this once inside a Claude Code session):

```
/plugin marketplace add "A:/Projects/Plugins"
```

**3. Install from it:**

```
/plugin install python-backend-plugin@Local-Plugins
```

After this, the plugin loads automatically in every session without the `--plugin-dir` flag.

---

## Using the Commands

### `/generate-endpoint` — Create a FastAPI endpoint

Generates a router, Pydantic schemas, a service class, and a pytest test file all at once.

**Syntax:**
```
/generate-endpoint <resource> <method>
```

**Examples:**
```
/generate-endpoint user GET
/generate-endpoint invoice POST
/generate-endpoint product DELETE
```

Each command creates 4 files:
- `app/routers/<resource>.py` — the FastAPI route
- `app/schemas/<resource>.py` — Pydantic request/response models
- `app/services/<resource>_service.py` — business logic (no FastAPI imports)
- `tests/test_<resource>.py` — pytest tests

---

### `/generate-test` — Write tests for an existing file

Reads your module and generates a full pytest test file covering happy paths, edge cases, and error cases.

**Syntax:**
```
/generate-test <filepath>
```

**Examples:**
```
/generate-test app/services/user_service.py
/generate-test app/routers/invoice.py
/generate-test app/utils/email.py
```

Output is saved to `tests/test_<module_name>.py`.

---

### `/review-module` — Review a Python file

Checks a module for common Python mistakes and style issues.

**Syntax:**
```
/review-module
/review-module <filepath>
```

**Examples:**
```
/review-module
/review-module app/services/user_service.py
```

It checks for things like:
- Missing type hints
- Mutable default arguments (`def foo(items=[])`)
- Bare `except:` that silently swallow errors
- Hardcoded credentials
- Blocking calls inside `async` functions
- Functions that are too long

Output looks like:
```
## Module Review: user_service.py

### ✅ Passed
### ⚠️ Warnings
### ❌ Issues
### 📋 Summary
```

---

## Using the Security Auditor

The `@security-auditor` is a specialized AI agent you can call anytime to audit your code for security vulnerabilities.

**How to use it:**

In your Claude Code chat, just mention it:
```
@security-auditor please audit app/routers/auth.py
```
or
```
@security-auditor check the whole app/ directory before I open a PR
```

It checks for:
- Hardcoded secrets and API keys
- SQL injection and shell injection risks
- Unsafe use of `eval()`, `pickle`, or `yaml.load()`
- FastAPI routes missing authentication
- Overly permissive CORS settings
- Vulnerable package versions in `requirements.txt`

Output is a report organized by severity: Critical, High, Medium, and Low.

---

## Automatic Hooks (Runs in the Background)

Once the plugin is installed, these run automatically — you don't need to do anything.

| Hook | When it runs | What it does |
|---|---|---|
| **Auto-format** | After Claude writes any `.py` file | Runs `ruff format` and `ruff check --fix` |
| **Run tests** | After Claude edits a file in `app/` | Runs the matching test file with pytest |
| **Type check** | After Claude writes to `app/` | Runs `mypy` and shows any type errors |
| **Secret scan** | Before any `git commit` | Scans staged files for hardcoded secrets |
| **Block rm -rf** | Before dangerous bash commands | Blocks `rm -rf` targeting root or home directories |
| **Pip version warning** | Before `pip install` | Warns if you install a package without pinning a version |
| **Session checklist** | When your Claude Code session ends | Prints a reminder checklist |

> **Prerequisites for hooks:** The hooks require `ruff`, `mypy`, and `pytest` to be installed in your Python environment. Install them with:
> ```bash
> pip install ruff mypy pytest pytest-asyncio pytest-mock httpx
> ```

---

## Recommended Project Structure

The plugin works best when your project follows this layout:

```
app/
├── main.py               # FastAPI app setup
├── routers/              # One file per resource (users.py, invoices.py)
├── services/             # Business logic — no FastAPI imports here
├── schemas/              # Pydantic models
├── models/               # Database models
├── dependencies/         # Shared Depends() functions (auth, db)
└── utils/                # Helper functions
tests/
├── conftest.py           # Shared test fixtures
└── test_<module>.py      # One test file per module
```

---

## Quick Start Example

Here's a typical workflow using the plugin:

```
# 1. Scaffold a new endpoint
/generate-endpoint order POST

# 2. Review the generated code
/review-module app/routers/order.py

# 3. Security check before committing
@security-auditor audit app/routers/order.py

# 4. Commit — the secret scan hook runs automatically
git commit -m "add order endpoint"
```

---

## Troubleshooting

**Plugin not found during install**
Make sure you're pointing to the folder that contains the `.claude-plugin/plugin.json` file, not a parent folder.

**Hooks not running**
Check that `ruff`, `mypy`, and `pytest` are installed and available in your terminal's PATH.

**`@security-auditor` not responding**
Make sure the plugin is installed (`claude plugin list`) and you're using `@security-auditor` at the start of your message.
# Claude_local_plugin
