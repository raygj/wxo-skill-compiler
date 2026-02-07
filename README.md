# wxo-skill-compiler

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.11%2B-blue)](https://www.python.org/)

**Stop writing boilerplate. Start building skills.**

Converts declarative `SKILL.yaml` → production watsonx Orchestrate MCP servers.

---

## The Problem

Building watsonx Orchestrate skills requires:
- `server.py` (MCP server implementation)
- `openapi.yaml` (API specification)
- `requirements.txt` (dependencies)
- `.env.example` (configuration)
- `agents/agent.yaml` (agent config)
- `README.md` (documentation)

**That's 6+ files of boilerplate for a single skill.**

## The Solution

Write **one declarative file**:

```yaml
# my-skill.yaml
name: vault-auditor
description: "Scan Vault secrets for compliance violations"

tools:
  - name: audit_secrets
    inputs:
      - name: vault_path
        type: string
    implementation: |
      # Your logic here
      secrets = vault_mcp.list(vault_path)
      return analyze(secrets)
```

Compile to **production-ready MCP server**:

```bash
wxo compile my-skill.yaml
# → Generates server.py, openapi.yaml, requirements.txt, README.md, etc.
```

---

## Quick Start

### 1. Install

```bash
git clone https://github.com/YOUR-ORG/wxo-skill-compiler
cd wxo-skill-compiler
pip install -r requirements.txt

# Optional: Install globally
pip install -e .
```

### 2. Create Skill Definition

```bash
cp examples/simple-vault-auditor.skill.yaml my-skill.yaml
# Edit my-skill.yaml
```

### 3. Compile

```bash
python wxo_compile.py my-skill.yaml
# → Generates ./my-skill/ directory with all files
```

### 4. Deploy

```bash
cd my-skill
pip install -r requirements.txt
cp .env.example .env  # Configure credentials
python server.py      # Test locally

# Import to watsonx Orchestrate
orchestrate toolkits import --kind mcp --package-root .
orchestrate agents import -f agents/agent.yaml
```

---

## Features

### ✅ What Gets Generated

- **server.py**: Full MCP server with FastMCP
- **openapi.yaml**: REST API specification
- **requirements.txt**: Python dependencies
- **.env.example**: Configuration template
- **agents/agent.yaml**: watsonx agent config
- **README.md**: Documentation with examples

### 🎯 What You Write

Just the **declarative skill definition**:

```yaml
name: my-skill
description: "What it does"
tools:
  - name: my_tool
    inputs: [...]
    outputs: {...}
    implementation: |
      # Your Python code
```

### 🔧 Customization

Generated files are **editable scaffolds**:
- Compiler generates 80% boilerplate
- You add 20% custom logic
- Re-compile preserves manual edits (future feature)

---

## Skill Format Specification

See [spec/skill_format_v1.yaml](spec/skill_format_v1.yaml) for complete format.

### Minimal Example

```yaml
name: hello-world
description: "My first skill"

tools:
  - name: greet
    inputs:
      - name: user_name
        type: string
    implementation: |
      return f"Hello, {user_name}!"
```

### Full-Featured Example

```yaml
name: vault-secrets-auditor
version: 1.0.0
description: "Comprehensive Vault auditing with risk scoring"

mcp_dependencies:
  - name: vault
    package: hashicorp/vault-mcp-server
    env_vars: [VAULT_ADDR, VAULT_TOKEN]

grounding: |
  You are a security expert specializing in secrets management.
  [... LLM instructions ...]

tools:
  - name: audit_secrets
    inputs:
      - name: scan_targets
        type: array
        description: "Paths to scan"
    outputs:
      type: object
      properties:
        total_secrets: {type: integer}
        findings: {type: array}
    implementation: |
      # Complex multi-step logic
      findings = scan_vault(scan_targets)
      scores = calculate_risk(findings)
      roadmap = generate_plan(scores)
      return {...}
    helpers:
      - name: calculate_risk
        code: |
          def calculate_risk(finding):
              # Helper function logic
              return score

resources:
  - name: compliance-rules
    uri: "vault://rules"
    content:
      PCI-DSS: {...}

examples:
  - name: "Audit production"
    input: {scan_targets: ["secret/prod"]}
    output: {total_secrets: 47, ...}
```

See [examples/](examples/) for more.

---

## Use Cases

### Enterprise Platform Teams
Build reusable skill library:
```bash
skills/
├── terraform-code-gen.skill.yaml
├── vault-auditor.skill.yaml
├── sentinel-policy.skill.yaml
└── Makefile  # make build → compile all
```

### SAs & Solution Engineers
Rapid prototyping for customer demos:
```bash
# Write skill definition in meeting
vim customer-specific-skill.yaml

# Generate working demo
wxo compile customer-specific-skill.yaml
cd customer-specific-skill && python server.py

# Present in same meeting
```

### Open-Source Contributors
Share skills as YAML (not Python servers):
```bash
# Contributor writes
my-skill.skill.yaml

# Consumer compiles for their platform
wxo compile my-skill.skill.yaml
# → Works with watsonx, LangChain, AutoGen, etc. (future)
```

---

## Architecture

```
my-skill.yaml (declarative)
       ↓
wxo-skill-compiler
       ↓
my-skill/
├── server.py          # FastMCP server
├── openapi.yaml       # OpenAPI 3.0 spec
├── requirements.txt   # Python deps
├── .env.example      # Config template
├── agents/
│   └── agent.yaml    # watsonx agent
└── README.md         # Documentation
       ↓
orchestrate toolkits import
       ↓
Production watsonx Orchestrate skill
```

---

## Comparison

| Approach | Files to Write | Lines of Code | Time to First Working Skill |
|----------|---------------|---------------|----------------------------|
| **Manual** | 6+ files | 500+ lines | 4-8 hours |
| **wxo-skill-compiler** | 1 file | ~50 lines | 15 minutes |

---

## Roadmap

- ✅ **v1.0**: Basic compilation (YAML → server.py + openapi.yaml)
- 🟡 **v1.1**: Incremental compilation (preserve manual edits)
- ⏳ **v1.2**: Multi-target compilation (watsonx, LangChain, AutoGen)
- ⏳ **v1.3**: Visual skill builder (web UI)
- ⏳ **v2.0**: Skill marketplace (publish/discover skills)

---

## Contributing

We need:
- **Template improvements** (better generated code)
- **Example skills** (for different use cases)
- **Platform support** (compile for LangChain, AutoGen, etc.)
- **Testing** (validate generated skills work)

See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## Why This Matters

### For IBM
- **Adoption**: Lower barrier to watsonx Orchestrate skills
- **Velocity**: 10x faster skill development
- **Community**: Skill sharing via YAML (not Python servers)

### For Developers
- **DX**: Write what matters (logic), not boilerplate
- **Portability**: YAML skills work across platforms (future)
- **Learning**: Generated code teaches MCP best practices

### For The Ecosystem
- **Open standard**: Declarative skill format (propose to MCP community)
- **Tooling ecosystem**: Linters, validators, IDEs for .skill.yaml
- **Marketplace**: Discoverable, shareable skills

---

## FAQ

**Q: Does this replace manual coding?**  
A: No. Generated code is a **scaffold**. You add custom logic where marked `# TODO`.

**Q: What if I need custom Python beyond the template?**  
A: Edit `server.py` directly. Compiler generates 80%, you customize 20%.

**Q: Can I re-compile without losing edits?**  
A: Not yet (v1.0). v1.1 will support incremental compilation.

**Q: Does this work with non-watsonx platforms?**  
A: Not yet. v1.2 will support LangChain, AutoGen, etc.

**Q: Is the skill format an official standard?**  
A: No, it's our proposal. We hope MCP community adopts it.

---

## License

Apache License 2.0

---

## Acknowledgments

Inspired by:
- Claude Skills (Anthropic) - declarative skill authoring
- OpenAPI Generator - code generation from specs
- Terraform - declarative infrastructure

Built because **we got tired of writing the same 500 lines of boilerplate for every skill**.

---

**Stop writing boilerplate. Start building skills.**

```bash
wxo compile your-brilliant-idea.skill.yaml
```
