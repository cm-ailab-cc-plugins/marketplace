# Team Marketplace 實作計畫

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 建立 CMoney AILab 團隊的 Claude Code Plugin Marketplace 基礎設施，包含 index repo 和管理工具 plugin-mp。

**Architecture:** 採用 marketplace index + 獨立 plugin repo 架構。marketplace repo 提供 plugin 索引、CI 驗證和 scaffold 範本。plugin-mp 以 SKILL.md 形式提供所有管理操作（發佈、搜尋、更新等），是第一個自舉 plugin。

**Tech Stack:** Bash/jq（驗證腳本）、GitHub Actions（CI）、Claude Code Plugin System（SKILL.md）、gh CLI

**Spec:** `docs/2026-04-02-team-marketplace-design.md`

---

## 檔案結構

### marketplace repo (`cm-ailab-cc-plugins/marketplace`)

```
marketplace/
├── .claude-plugin/
│   └── marketplace.json              ← Plugin 目錄索引
├── .github/
│   ├── CODEOWNERS                    ← 審核者設定
│   └── workflows/
│       └── validate.yml              ← CI 自動驗證
├── scripts/
│   └── validate-plugin.sh            ← 驗證腳本（CI 使用）
├── tests/
│   ├── run-tests.sh                  ← 測試執行器
│   └── fixtures/                     ← 測試用 mock plugin 結構
│       ├── valid-skill/
│       ├── valid-hook/
│       ├── invalid-missing-fields/
│       ├── invalid-type-mismatch/
│       └── invalid-naming/
├── templates/
│   ├── skill/
│   ├── agent/
│   ├── hook/
│   ├── mcp-server/
│   └── mixed/
└── docs/
    └── (existing design doc)
```

### plugin-mp repo (`cm-ailab-cc-plugins/plugin-mp`)

```
plugin-mp/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── setup/
│   │   └── SKILL.md
│   ├── validate/
│   │   └── SKILL.md
│   ├── list/
│   │   └── SKILL.md
│   ├── search/
│   │   └── SKILL.md
│   ├── my-plugins/
│   │   └── SKILL.md
│   ├── publish/
│   │   └── SKILL.md
│   ├── update/
│   │   └── SKILL.md
│   └── deprecate/
│       └── SKILL.md
└── README.md
```

---

## Phase 1: Marketplace Repo 基礎建設

### Task 1: marketplace.json 初始化

**Files:**
- Create: `.claude-plugin/marketplace.json`

- [ ] **Step 1: 建立目錄**

```bash
mkdir -p .claude-plugin
```

- [ ] **Step 2: 寫入 marketplace.json**

```json
{
  "name": "cm-ailab-cc-plugins",
  "owner": {
    "name": "CMoney AILab",
    "email": "ailab@cmoney.com.tw"
  },
  "metadata": {
    "description": "CMoney AILab 團隊共用的 Claude Code Plugin Marketplace",
    "version": "1.0.0"
  },
  "plugins": []
}
```

- [ ] **Step 3: 驗證 JSON 格式**

Run: `jq . .claude-plugin/marketplace.json`
Expected: 格式化的 JSON 輸出，無錯誤

- [ ] **Step 4: Commit**

```bash
git add .claude-plugin/marketplace.json
git commit -m "feat: 初始化 marketplace.json 索引"
```

---

### Task 2: Plugin scaffold 範本

**Files:**
- Create: `templates/skill/**`, `templates/agent/**`, `templates/hook/**`, `templates/mcp-server/**`, `templates/mixed/**`

- [ ] **Step 1: 建立 skill 範本**

`templates/skill/.claude-plugin/plugin.json`:
```json
{
  "name": "__PLUGIN_NAME__",
  "version": "1.0.0",
  "type": "skill",
  "description": "__DESCRIPTION__",
  "keywords": ["__KEYWORD1__", "__KEYWORD2__"],
  "author": {
    "name": "__AUTHOR_NAME__",
    "github": "__GITHUB_USER__"
  }
}
```

`templates/skill/skills/__PLUGIN_NAME__/SKILL.md`:
```markdown
---
name: __PLUGIN_NAME__
description: __DESCRIPTION__
---

# __PLUGIN_NAME__

在此撰寫 skill 的指令內容。
```

`templates/skill/README.md`:
```markdown
# plugin-__PLUGIN_NAME__

__DESCRIPTION__

## 安裝

```
/plugin install __PLUGIN_NAME__@cm-ailab-cc-plugins
```

## 使用

```
/__PLUGIN_NAME__
```
```

- [ ] **Step 2: 建立 agent 範本**

`templates/agent/.claude-plugin/plugin.json`:
```json
{
  "name": "__PLUGIN_NAME__",
  "version": "1.0.0",
  "type": "agent",
  "description": "__DESCRIPTION__",
  "keywords": ["__KEYWORD1__", "__KEYWORD2__"],
  "author": {
    "name": "__AUTHOR_NAME__",
    "github": "__GITHUB_USER__"
  }
}
```

`templates/agent/agents/__PLUGIN_NAME__.md`:
```markdown
---
name: __PLUGIN_NAME__
description: __DESCRIPTION__
tools:
  - Read
  - Grep
  - Glob
  - Bash
---

# __PLUGIN_NAME__

在此撰寫 agent 的指令內容。
```

`templates/agent/README.md`:（同 skill 範本格式）

- [ ] **Step 3: 建立 hook 範本**

`templates/hook/.claude-plugin/plugin.json`:
```json
{
  "name": "__PLUGIN_NAME__",
  "version": "1.0.0",
  "type": "hook",
  "description": "__DESCRIPTION__",
  "keywords": ["__KEYWORD1__", "__KEYWORD2__"],
  "author": {
    "name": "__AUTHOR_NAME__",
    "github": "__GITHUB_USER__"
  }
}
```

`templates/hook/hooks/hooks.json`:
```json
{
  "hooks": {
    "__EVENT_TYPE__": [
      {
        "matcher": "",
        "command": "echo 'hook triggered'"
      }
    ]
  }
}
```

`templates/hook/README.md`:（同格式）

- [ ] **Step 4: 建立 mcp-server 範本**

`templates/mcp-server/.claude-plugin/plugin.json`:
```json
{
  "name": "__PLUGIN_NAME__",
  "version": "1.0.0",
  "type": "mcp",
  "description": "__DESCRIPTION__",
  "keywords": ["__KEYWORD1__", "__KEYWORD2__"],
  "author": {
    "name": "__AUTHOR_NAME__",
    "github": "__GITHUB_USER__"
  }
}
```

`templates/mcp-server/mcp-servers/config.json`:
```json
{
  "__PLUGIN_NAME__": {
    "command": "__COMMAND__",
    "args": []
  }
}
```

- [ ] **Step 5: 建立 mixed 範本**

`templates/mixed/.claude-plugin/plugin.json`:
```json
{
  "name": "__PLUGIN_NAME__",
  "version": "1.0.0",
  "type": "mixed",
  "description": "__DESCRIPTION__",
  "keywords": ["__KEYWORD1__", "__KEYWORD2__"],
  "author": {
    "name": "__AUTHOR_NAME__",
    "github": "__GITHUB_USER__"
  }
}
```

`templates/mixed/skills/__PLUGIN_NAME__/SKILL.md`:（同 skill 範本）
`templates/mixed/agents/__PLUGIN_NAME__.md`:（同 agent 範本）
`templates/mixed/README.md`:（同格式）

- [ ] **Step 6: Commit**

```bash
git add templates/
git commit -m "feat: 新增 plugin scaffold 範本（skill/agent/hook/mcp/mixed）"
```

---

### Task 3: 驗證腳本 — 核心驗證 (TDD)

**Files:**
- Create: `scripts/validate-plugin.sh`
- Create: `tests/run-tests.sh`
- Create: `tests/fixtures/**`

- [ ] **Step 1: 建立測試 fixtures — valid-skill**

`tests/fixtures/valid-skill/.claude-plugin/plugin.json`:
```json
{
  "name": "test-skill",
  "version": "1.0.0",
  "type": "skill",
  "description": "A valid test skill plugin for validation testing",
  "keywords": ["test", "validation"],
  "author": {
    "name": "Test User",
    "github": "test-user"
  }
}
```

`tests/fixtures/valid-skill/skills/test-skill/SKILL.md`:
```markdown
---
name: test-skill
description: A test skill
---

Test content.
```

`tests/fixtures/valid-skill/README.md`:
```markdown
# test-skill
A valid test skill.
```

- [ ] **Step 2: 建立測試 fixtures — invalid 案例**

`tests/fixtures/invalid-missing-fields/.claude-plugin/plugin.json`:
```json
{
  "name": "bad-plugin",
  "version": "1.0.0"
}
```

`tests/fixtures/invalid-type-mismatch/.claude-plugin/plugin.json`:
```json
{
  "name": "type-mismatch",
  "version": "1.0.0",
  "type": "skill",
  "description": "Claims to be skill but has hooks",
  "keywords": ["test", "mismatch"],
  "author": {
    "name": "Test",
    "github": "test"
  }
}
```

`tests/fixtures/invalid-type-mismatch/hooks/hooks.json`:
```json
{ "hooks": {} }
```

`tests/fixtures/invalid-naming/.claude-plugin/plugin.json`:
```json
{
  "name": "wrong-name",
  "version": "not-semver",
  "type": "invalid-type",
  "description": "Bad",
  "keywords": ["only-one"],
  "author": {
    "name": "Test",
    "github": "test"
  }
}
```

- [ ] **Step 3: 建立測試 fixture — valid-hook**

`tests/fixtures/valid-hook/.claude-plugin/plugin.json`:
```json
{
  "name": "test-hook",
  "version": "2.1.0",
  "type": "hook",
  "description": "A valid test hook plugin for validation testing",
  "keywords": ["test", "hook"],
  "author": {
    "name": "Test User",
    "github": "test-user"
  }
}
```

`tests/fixtures/valid-hook/hooks/hooks.json`:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "command": "echo 'file written'"
      }
    ]
  }
}
```

- [ ] **Step 4: 撰寫測試執行器**

`tests/run-tests.sh`:
```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
ROOT_DIR="$(cd "$SCRIPT_DIR/.." && pwd)"
VALIDATE="$ROOT_DIR/scripts/validate-plugin.sh"
FIXTURES="$SCRIPT_DIR/fixtures"

PASS=0
FAIL=0
TOTAL=0

run_test() {
  local test_name="$1"
  local expected="$2"  # "pass" or "fail"
  local plugin_dir="$3"
  local repo_name="$4"
  ((TOTAL++))

  local result
  if bash "$VALIDATE" "$plugin_dir" "$repo_name" > /dev/null 2>&1; then
    result="pass"
  else
    result="fail"
  fi

  if [[ "$result" == "$expected" ]]; then
    echo "  ✓ $test_name"
    ((PASS++))
  else
    echo "  ✗ $test_name (expected $expected, got $result)"
    ((FAIL++))
  fi
}

echo "=== validate-plugin.sh tests ==="
echo ""

echo "--- Structure validation ---"
run_test "valid skill plugin passes" "pass" "$FIXTURES/valid-skill" "plugin-test-skill"
run_test "valid hook plugin passes" "pass" "$FIXTURES/valid-hook" "plugin-test-hook"
run_test "missing required fields fails" "fail" "$FIXTURES/invalid-missing-fields" "plugin-bad-plugin"
run_test "invalid naming/version/type fails" "fail" "$FIXTURES/invalid-naming" "plugin-wrong-name"

echo ""
echo "--- Content & cross validation ---"
run_test "type mismatch (skill with hooks) fails" "fail" "$FIXTURES/invalid-type-mismatch" "plugin-type-mismatch"
run_test "wrong repo name fails" "fail" "$FIXTURES/valid-skill" "plugin-wrong-repo-name"

echo ""
echo "=== Results: $PASS/$TOTAL passed, $FAIL failed ==="

if (( FAIL > 0 )); then
  exit 1
fi
```

- [ ] **Step 5: 執行測試確認全部失敗**

Run: `chmod +x tests/run-tests.sh && bash tests/run-tests.sh`
Expected: 所有測試失敗（驗證腳本尚未存在）

- [ ] **Step 6: 撰寫驗證腳本**

`scripts/validate-plugin.sh`:
```bash
#!/usr/bin/env bash
set -euo pipefail

# Usage: validate-plugin.sh <plugin-dir> <repo-name> [<expected-ref>]
# Exit 0 = pass, Exit 1 = fail

PLUGIN_DIR="${1:?Usage: validate-plugin.sh <plugin-dir> <repo-name> [<expected-ref>]}"
REPO_NAME="${2:?Usage: validate-plugin.sh <plugin-dir> <repo-name> [<expected-ref>]}"
EXPECTED_REF="${3:-}"

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

ERRORS=0
WARNINGS=0

error() { echo -e "${RED}✗ $1${NC}"; ((ERRORS++)) || true; }
pass()  { echo -e "${GREEN}✓ $1${NC}"; }
warn()  { echo -e "${YELLOW}⚠ $1${NC}"; ((WARNINGS++)) || true; }

# ── 結構驗證 ──

PLUGIN_JSON="$PLUGIN_DIR/.claude-plugin/plugin.json"

if [[ ! -f "$PLUGIN_JSON" ]]; then
  error "plugin.json not found at .claude-plugin/plugin.json"
  echo -e "\n${RED}Validation failed with $ERRORS error(s)${NC}"
  exit 1
fi
pass "plugin.json exists"

# 必填欄位
for field in name version type description; do
  val=$(jq -r ".$field // empty" "$PLUGIN_JSON" 2>/dev/null)
  if [[ -z "$val" ]]; then
    error "Missing required field: $field"
  else
    pass "Field '$field' exists: $val"
  fi
done

# keywords (>= 2)
KW_COUNT=$(jq '.keywords | length' "$PLUGIN_JSON" 2>/dev/null || echo 0)
if (( KW_COUNT < 2 )); then
  error "keywords must have at least 2 entries (found $KW_COUNT)"
else
  pass "keywords has $KW_COUNT entries"
fi

# author.github
AUTHOR_GH=$(jq -r '.author.github // empty' "$PLUGIN_JSON" 2>/dev/null)
if [[ -z "$AUTHOR_GH" ]]; then
  error "Missing required field: author.github"
else
  pass "author.github: $AUTHOR_GH"
fi

# type 合法值
PTYPE=$(jq -r '.type // empty' "$PLUGIN_JSON" 2>/dev/null)
case "$PTYPE" in
  skill|agent|hook|mcp|mixed) pass "Type '$PTYPE' is valid" ;;
  "") ;; # already reported as missing
  *) error "Invalid type: '$PTYPE' (must be skill|agent|hook|mcp|mixed)" ;;
esac

# version semver
VERSION=$(jq -r '.version // empty' "$PLUGIN_JSON" 2>/dev/null)
if [[ -n "$VERSION" ]]; then
  if [[ "$VERSION" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
    pass "Version '$VERSION' is valid semver"
  else
    error "Version '$VERSION' is not valid semver (expected x.y.z)"
  fi
fi

# ── 命名驗證 ──

PNAME=$(jq -r '.name // empty' "$PLUGIN_JSON" 2>/dev/null)
if [[ -n "$PNAME" ]]; then
  EXPECTED_REPO="plugin-$PNAME"
  if [[ "$REPO_NAME" == "$EXPECTED_REPO" ]]; then
    pass "Repo name '$REPO_NAME' matches plugin-$PNAME"
  else
    error "Repo name '$REPO_NAME' should be '$EXPECTED_REPO'"
  fi
fi

# ── 內容驗證（依 type）──

if [[ -n "$PTYPE" ]]; then
  case "$PTYPE" in
    skill)
      SKILL_COUNT=$(find "$PLUGIN_DIR/skills" -name "SKILL.md" 2>/dev/null | wc -l | tr -d ' ')
      if (( SKILL_COUNT == 0 )); then
        error "type=skill but no SKILL.md found under skills/"
      else
        pass "Found $SKILL_COUNT SKILL.md file(s)"
        while IFS= read -r sf; do
          if head -1 "$sf" | grep -q '^---'; then
            if grep -q '^description:' "$sf"; then
              pass "SKILL.md has description frontmatter: $sf"
            else
              error "SKILL.md missing description in frontmatter: $sf"
            fi
          else
            error "SKILL.md missing frontmatter (---): $sf"
          fi
        done < <(find "$PLUGIN_DIR/skills" -name "SKILL.md" 2>/dev/null)
      fi
      ;;
    hook)
      if [[ -f "$PLUGIN_DIR/hooks/hooks.json" ]]; then
        if jq . "$PLUGIN_DIR/hooks/hooks.json" > /dev/null 2>&1; then
          pass "hooks.json is valid JSON"
        else
          error "hooks.json is not valid JSON"
        fi
      else
        error "type=hook but hooks/hooks.json not found"
      fi
      ;;
    agent)
      AGENT_COUNT=$(find "$PLUGIN_DIR/agents" -name "*.md" 2>/dev/null | wc -l | tr -d ' ')
      if (( AGENT_COUNT == 0 )); then
        error "type=agent but no .md files found under agents/"
      else
        pass "Found $AGENT_COUNT agent definition(s)"
        while IFS= read -r af; do
          if head -1 "$af" | grep -q '^---'; then
            pass "Agent has frontmatter: $af"
          else
            error "Agent missing YAML frontmatter: $af"
          fi
        done < <(find "$PLUGIN_DIR/agents" -name "*.md" 2>/dev/null)
      fi
      ;;
    mcp)
      MCP_FOUND=0
      for mf in "$PLUGIN_DIR/mcp-servers"/*.json; do
        [[ -f "$mf" ]] || continue
        ((MCP_FOUND++))
        if jq -e '.. | .command? // empty' "$mf" > /dev/null 2>&1; then
          pass "MCP config has command field: $mf"
        else
          error "MCP config missing command field: $mf"
        fi
      done
      if (( MCP_FOUND == 0 )); then
        error "type=mcp but no JSON files found under mcp-servers/"
      fi
      ;;
    mixed)
      CONTENT_TYPES=0
      [[ -d "$PLUGIN_DIR/skills" ]] && (( $(find "$PLUGIN_DIR/skills" -name "SKILL.md" 2>/dev/null | wc -l) > 0 )) && ((CONTENT_TYPES++)) || true
      [[ -d "$PLUGIN_DIR/agents" ]] && (( $(find "$PLUGIN_DIR/agents" -name "*.md" 2>/dev/null | wc -l) > 0 )) && ((CONTENT_TYPES++)) || true
      [[ -f "$PLUGIN_DIR/hooks/hooks.json" ]] && ((CONTENT_TYPES++)) || true
      [[ -d "$PLUGIN_DIR/mcp-servers" ]] && (( $(find "$PLUGIN_DIR/mcp-servers" -name "*.json" 2>/dev/null | wc -l) > 0 )) && ((CONTENT_TYPES++)) || true
      if (( CONTENT_TYPES < 2 )); then
        error "type=mixed but only $CONTENT_TYPES content type(s) found (need >= 2)"
      else
        pass "type=mixed with $CONTENT_TYPES content types"
      fi
      ;;
  esac
fi

# ── 交叉驗證 ──

if [[ -n "$PTYPE" && "$PTYPE" != "hook" && "$PTYPE" != "mixed" ]]; then
  if [[ -f "$PLUGIN_DIR/hooks/hooks.json" ]]; then
    error "type=$PTYPE but hooks/hooks.json exists (should be type=hook or mixed)"
  fi
fi

if [[ -n "$PTYPE" && "$PTYPE" != "mcp" && "$PTYPE" != "mixed" ]]; then
  MCP_EXISTS=$(find "$PLUGIN_DIR/mcp-servers" -name "*.json" 2>/dev/null | wc -l | tr -d ' ')
  if (( MCP_EXISTS > 0 )); then
    error "type=$PTYPE but mcp-servers/ contains config (should be type=mcp or mixed)"
  fi
fi

# ── 版本一致性（僅在提供 ref 時）──

if [[ -n "$EXPECTED_REF" && -n "$VERSION" ]]; then
  EXPECTED_VERSION="${EXPECTED_REF#v}"
  if [[ "$VERSION" == "$EXPECTED_VERSION" ]]; then
    pass "Version '$VERSION' matches ref '$EXPECTED_REF'"
  else
    error "Version '$VERSION' does not match ref '$EXPECTED_REF' (expected $EXPECTED_VERSION)"
  fi
fi

# ── 品質檢查（警告）──

DESC=$(jq -r '.description // empty' "$PLUGIN_JSON" 2>/dev/null)
if [[ -n "$DESC" && ${#DESC} -lt 10 ]]; then
  warn "description is very short (${#DESC} chars) — consider adding more detail"
fi

if [[ ! -f "$PLUGIN_DIR/README.md" ]]; then
  warn "No README.md found — consider adding one"
fi

# ── 結果 ──

echo ""
if (( ERRORS > 0 )); then
  echo -e "${RED}Validation FAILED with $ERRORS error(s), $WARNINGS warning(s)${NC}"
  exit 1
else
  echo -e "${GREEN}Validation PASSED with $WARNINGS warning(s)${NC}"
  exit 0
fi
```

- [ ] **Step 7: 執行測試確認全部通過**

Run: `chmod +x scripts/validate-plugin.sh && bash tests/run-tests.sh`
Expected: `6/6 passed, 0 failed`

- [ ] **Step 8: Commit**

```bash
git add scripts/validate-plugin.sh tests/
git commit -m "feat: 新增 plugin 驗證腳本與測試"
```

---

### Task 4: CI Workflow + CODEOWNERS

**Files:**
- Create: `.github/workflows/validate.yml`
- Create: `.github/CODEOWNERS`

- [ ] **Step 1: 建立 CI Workflow**

`.github/workflows/validate.yml`:
```yaml
name: Validate Plugin PR

on:
  pull_request:
    paths:
      - '.claude-plugin/marketplace.json'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout marketplace
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get base marketplace.json
        run: |
          git show origin/${{ github.base_ref }}:.claude-plugin/marketplace.json \
            > /tmp/base-marketplace.json 2>/dev/null \
            || echo '{"plugins":[]}' > /tmp/base-marketplace.json

      - name: Find changed plugins
        id: diff
        run: |
          # 比較 base 和 PR 版本，找出新增或修改的 plugin
          jq -r '.plugins[] | "\(.name)|\(.source.repo // "")|\(.source.ref // "")"' \
            .claude-plugin/marketplace.json | sort > /tmp/curr.txt

          jq -r '.plugins[] | "\(.name)|\(.source.repo // "")|\(.source.ref // "")"' \
            /tmp/base-marketplace.json | sort > /tmp/base.txt 2>/dev/null || touch /tmp/base.txt

          CHANGED=$(comm -23 /tmp/curr.txt /tmp/base.txt)
          if [[ -z "$CHANGED" ]]; then
            echo "No plugin changes detected"
            echo "changed=false" >> "$GITHUB_OUTPUT"
          else
            echo "Changed plugins:"
            echo "$CHANGED"
            echo "$CHANGED" > /tmp/changed.txt
            echo "changed=true" >> "$GITHUB_OUTPUT"
          fi

      - name: Validate changed plugins
        if: steps.diff.outputs.changed == 'true'
        run: |
          FAILED=0
          while IFS='|' read -r name repo ref; do
            echo ""
            echo "=========================================="
            echo "Validating: $name ($repo@$ref)"
            echo "=========================================="

            # Clone plugin repo at specified ref
            TMPDIR=$(mktemp -d)
            if ! git clone --depth 1 --branch "$ref" \
              "https://github.com/${repo}.git" "$TMPDIR" 2>/dev/null; then
              echo "::error::Failed to clone $repo at ref $ref"
              ((FAILED++)) || true
              continue
            fi

            # Run validation
            if ! bash scripts/validate-plugin.sh "$TMPDIR" "plugin-$name" "$ref"; then
              ((FAILED++)) || true
            fi

            rm -rf "$TMPDIR"
          done < /tmp/changed.txt

          if (( FAILED > 0 )); then
            echo ""
            echo "::error::$FAILED plugin(s) failed validation"
            exit 1
          fi

      - name: Validate marketplace.json structure
        run: |
          # 驗證 marketplace.json 本身的格式
          JSON=".claude-plugin/marketplace.json"

          # 必要欄位
          jq -e '.name' "$JSON" > /dev/null || { echo "::error::Missing 'name'"; exit 1; }
          jq -e '.owner.name' "$JSON" > /dev/null || { echo "::error::Missing 'owner.name'"; exit 1; }
          jq -e '.plugins | type == "array"' "$JSON" > /dev/null || { echo "::error::'plugins' must be array"; exit 1; }

          # 每個 plugin entry 必要欄位
          ISSUES=$(jq -r '.plugins[] | select(
            (.name | length) == 0 or
            (.source.repo | length) == 0 or
            (.source.ref | length) == 0 or
            (.description | length) == 0
          ) | .name // "unnamed"' "$JSON")

          if [[ -n "$ISSUES" ]]; then
            echo "::error::Plugin entries with incomplete fields: $ISSUES"
            exit 1
          fi

          echo "✓ marketplace.json structure is valid"
```

- [ ] **Step 2: 建立 CODEOWNERS**

`.github/CODEOWNERS`:
```
# marketplace.json 變更需要指定審核者
.claude-plugin/marketplace.json @cm-ailab-cc-plugins/reviewers
```

- [ ] **Step 3: 驗證 workflow YAML 語法**

Run: `python3 -c "import yaml; yaml.safe_load(open('.github/workflows/validate.yml'))" && echo "YAML valid"`
Expected: `YAML valid`

- [ ] **Step 4: Commit**

```bash
git add .github/
git commit -m "feat: 新增 CI 驗證 workflow 和 CODEOWNERS"
```

---

## Phase 2: plugin-mp 管理工具

### Task 5: plugin-mp 基礎結構

**Files:**
- Create: `../plugin-mp/.claude-plugin/plugin.json`
- Create: `../plugin-mp/README.md`

- [ ] **Step 1: 建立 plugin-mp 目錄與 git repo**

```bash
mkdir -p ../plugin-mp/.claude-plugin
cd ../plugin-mp
git init
```

- [ ] **Step 2: 寫入 plugin.json**

`../plugin-mp/.claude-plugin/plugin.json`:
```json
{
  "name": "mp",
  "version": "1.0.0",
  "type": "skill",
  "description": "Team Marketplace 管理工具 — 發佈、搜尋、安裝、更新 plugin",
  "keywords": ["marketplace", "management", "plugin", "team"],
  "author": {
    "name": "CMoney AILab",
    "github": "ailabcmoney"
  }
}
```

- [ ] **Step 3: 寫入 README.md**

`../plugin-mp/README.md`:
```markdown
# plugin-mp

Team Marketplace 管理工具 — CMoney AILab 團隊的 Claude Code Plugin 管理中心。

## 安裝

```
/plugin install mp@cm-ailab-cc-plugins
```

## 可用命令

| 命令 | 功能 |
|------|------|
| `/mp:setup` | 環境檢查與設定 |
| `/mp:validate` | 本地驗證 plugin 結構 |
| `/mp:list` | 列出所有可用 plugin |
| `/mp:search <keyword>` | 搜尋 plugin |
| `/mp:my-plugins` | 列出自己發佈的 plugin |
| `/mp:publish` | 發佈新 plugin |
| `/mp:update <name>` | 更新 plugin 版本 |
| `/mp:deprecate <name>` | 標記 plugin 為棄用 |
```

- [ ] **Step 4: Commit**

```bash
cd ../plugin-mp
git add -A
git commit -m "feat: 初始化 plugin-mp 基礎結構"
```

---

### Task 6: /mp:setup skill

**Files:**
- Create: `../plugin-mp/skills/setup/SKILL.md`

- [ ] **Step 1: 撰寫 SKILL.md**

`../plugin-mp/skills/setup/SKILL.md`:
```markdown
---
name: setup
description: 檢查並設定 Team Marketplace 開發環境
---

# Marketplace 環境設定

執行以下檢查，逐項回報狀態：

## 檢查清單

### 1. gh CLI
```bash
which gh
```
- 有 → ✓ gh CLI 已安裝
- 無 → ✗ 請安裝：`brew install gh`

### 2. gh 認證狀態
```bash
gh auth status
```
- 已登入 → ✓ 已認證為 <username>
- 未登入 → ✗ 請執行：`gh auth login`

### 3. Org 成員資格
```bash
gh api /user/memberships/orgs/cm-ailab-cc-plugins --jq '.state' 2>/dev/null
```
- active → ✓ 已加入 cm-ailab-cc-plugins
- 無結果/錯誤 → ✗ 請聯繫管理員取得邀請

### 4. Marketplace 已加入
確認 marketplace 已加入 Claude Code：
```bash
cat ~/.claude/plugins/installed_plugins.json 2>/dev/null | grep -o 'cm-ailab-cc-plugins' | head -1
```
- 找到 → ✓ Marketplace 已設定
- 未找到 → ✗ 請執行：`/plugin marketplace add cm-ailab-cc-plugins/marketplace`

### 5. mp plugin 已安裝
（如果你正在執行此命令，代表已安裝）
✓ mp plugin 已安裝

### 6. GITHUB_TOKEN（背景更新用，可選）
```bash
echo "${GITHUB_TOKEN:+已設定}"
```
- 已設定 → ✓ GITHUB_TOKEN 已設定
- 空 → ⚠ 未設定（選填 — 僅背景自動更新需要）
  - 設定方式：在 shell profile 加入 `export GITHUB_TOKEN=$(gh auth token)`

## 輸出格式

逐項列出檢查結果，最後給出總結：
- 全部通過：「🎉 環境正常！可以開始使用 marketplace。」
- 有失敗項：列出修復步驟，引導使用者逐一處理
```

- [ ] **Step 2: 驗證 SKILL.md frontmatter**

Run: `head -4 ../plugin-mp/skills/setup/SKILL.md`
Expected: 開頭為 `---`，包含 `name: setup` 和 `description:`

- [ ] **Step 3: Commit**

```bash
cd ../plugin-mp
git add skills/setup/
git commit -m "feat: 新增 /mp:setup skill"
```

---

### Task 7: /mp:validate skill

**Files:**
- Create: `../plugin-mp/skills/validate/SKILL.md`

- [ ] **Step 1: 撰寫 SKILL.md**

`../plugin-mp/skills/validate/SKILL.md`:
```markdown
---
name: validate
description: 本地驗證 plugin 結構是否符合 marketplace 規範
---

# 驗證 Plugin 結構

對當前目錄（或指定目錄）的 plugin 進行結構驗證，等同 CI 的檢查項目。

## 使用方式

- `/mp:validate` — 驗證當前目錄
- `/mp:validate <path>` — 驗證指定目錄
- 「幫我檢查這個 plugin 結構對不對」

## 驗證流程

### 1. 定位 plugin.json

讀取 `<dir>/.claude-plugin/plugin.json`。如果不存在，回報錯誤並停止。

### 2. 結構驗證（必過）

檢查以下必填欄位，逐項回報 ✓ 或 ✗：

| 欄位 | 規則 |
|------|------|
| `name` | 非空字串 |
| `version` | semver 格式 `x.y.z` |
| `type` | 為 `skill`、`agent`、`hook`、`mcp`、`mixed` 之一 |
| `description` | 非空字串 |
| `keywords` | 陣列，至少 2 個元素 |
| `author.github` | 非空字串 |

### 3. 命名驗證

如果當前目錄名以 `plugin-` 開頭，驗證：
- 目錄名 == `plugin-` + `plugin.json.name`

### 4. 內容驗證（依 type）

| type | 檢查 |
|------|------|
| `skill` | `skills/` 下至少一個 `SKILL.md`，且有 frontmatter `description` |
| `agent` | `agents/` 下至少一個 `.md`，且有 YAML frontmatter |
| `hook` | `hooks/hooks.json` 存在且為合法 JSON |
| `mcp` | `mcp-servers/` 下有 JSON 且含 `command` 欄位 |
| `mixed` | 至少包含兩種以上類型的實際內容 |

### 5. 交叉驗證

- type 不含 `hook`/`mixed` → 不得有 `hooks/hooks.json`
- type 不含 `mcp`/`mixed` → 不得有 `mcp-servers/*.json`

### 6. 品質提示（警告，不阻擋）

- description 少於 10 字 → 建議補充
- 無 README.md → 建議新增
- keywords 與現有 plugin 高度重複 → 提醒

## 輸出格式

```
=== Plugin 驗證: <name> ===

結構驗證:
  ✓ plugin.json 存在
  ✓ name: query-mongo
  ✓ version: 1.0.0 (valid semver)
  ✓ type: skill
  ✓ description: ...
  ✓ keywords: [mongodb, query] (2 個)
  ✓ author.github: Nero-2307

內容驗證:
  ✓ 找到 1 個 SKILL.md
  ✓ SKILL.md 有 description frontmatter

品質提示:
  ⚠ 建議補上 README.md

結果: ✓ 通過 (0 errors, 1 warning)
```
```

- [ ] **Step 2: Commit**

```bash
cd ../plugin-mp
git add skills/validate/
git commit -m "feat: 新增 /mp:validate skill"
```

---

### Task 8: /mp:list + /mp:search + /mp:my-plugins skills

**Files:**
- Create: `../plugin-mp/skills/list/SKILL.md`
- Create: `../plugin-mp/skills/search/SKILL.md`
- Create: `../plugin-mp/skills/my-plugins/SKILL.md`

- [ ] **Step 1: 撰寫 /mp:list**

`../plugin-mp/skills/list/SKILL.md`:
```markdown
---
name: list
description: 列出 Team Marketplace 中所有可用的 plugin
---

# 列出所有 Plugin

## 使用方式

- `/mp:list` — 列出所有 plugin
- `/mp:list --all` — 包含已棄用的 plugin
- 「團隊有哪些 plugin 可以用？」

## 執行步驟

### 1. 取得 marketplace.json

```bash
gh api repos/cm-ailab-cc-plugins/marketplace/contents/.claude-plugin/marketplace.json \
  --jq '.content' | base64 -d
```

如果失敗，嘗試本地讀取：
```bash
cat ~/CMoney/cm-ailab-cc-plugins/marketplace/.claude-plugin/marketplace.json
```

### 2. 解析並格式化

讀取 `plugins` 陣列，過濾掉 `deprecated: true` 的項目（除非使用 `--all`）。

### 3. 輸出格式

以表格形式顯示：

```
📦 Team Marketplace — N 個 plugin

| 名稱 | 描述 | 版本 | 作者 |
|------|------|------|------|
| mp | Marketplace 管理工具 | v1.0.0 | ailabcmoney |
| query-mongo | 以自然語言查詢 MongoDB | v1.2.0 | Nero-2307 |

安裝方式: /plugin install <name>@cm-ailab-cc-plugins
```

如果有 deprecated plugin 且未使用 `--all`：
```
（另有 N 個已棄用 plugin，使用 /mp:list --all 檢視）
```
```

- [ ] **Step 2: 撰寫 /mp:search**

`../plugin-mp/skills/search/SKILL.md`:
```markdown
---
name: search
description: 以關鍵字搜尋 Team Marketplace 中的 plugin
---

# 搜尋 Plugin

## 使用方式

- `/mp:search <keyword>` — 以關鍵字搜尋
- 「有沒有人做過 MongoDB 相關的 plugin？」
- 「找找看有沒有跟 code review 相關的工具」

## 執行步驟

### 1. 取得 marketplace.json

```bash
gh api repos/cm-ailab-cc-plugins/marketplace/contents/.claude-plugin/marketplace.json \
  --jq '.content' | base64 -d
```

### 2. 搜尋匹配

從使用者的輸入中提取搜尋意圖，在以下欄位中比對：
- `name` — 名稱包含關鍵字
- `description` — 描述包含關鍵字
- `keywords` — 標籤完全或部分匹配

進行語意匹配，不僅限於字面比對。例如使用者說「資料庫」，也應匹配 `database`、`mongo`、`sql` 等相關詞。

### 3. 排除已棄用

預設排除 `deprecated: true` 的 plugin。使用者明確要求才顯示。

### 4. 輸出格式

```
🔍 搜尋 "mongodb" — 找到 N 筆結果

1. query-mongo (v1.2.0) by Nero-2307
   以自然語言查詢 MongoDB，自動生成並執行 query
   Keywords: mongodb, query, database
   安裝: /plugin install query-mongo@cm-ailab-cc-plugins

（無結果時）
🔍 搜尋 "xxx" — 未找到匹配的 plugin
```
```

- [ ] **Step 3: 撰寫 /mp:my-plugins**

`../plugin-mp/skills/my-plugins/SKILL.md`:
```markdown
---
name: my-plugins
description: 列出自己發佈的 plugin
---

# 我的 Plugin

## 使用方式

- `/mp:my-plugins` — 列出自己的 plugin
- 「我發佈過哪些 plugin？」

## 執行步驟

### 1. 取得當前使用者

```bash
gh api /user --jq '.login'
```

### 2. 取得 marketplace.json

```bash
gh api repos/cm-ailab-cc-plugins/marketplace/contents/.claude-plugin/marketplace.json \
  --jq '.content' | base64 -d
```

### 3. 取得每個 plugin 的作者

對於 marketplace.json 中的每個 plugin，取得其 plugin.json 查看 `author.github`：

```bash
gh api repos/cm-ailab-cc-plugins/plugin-<name>/contents/.claude-plugin/plugin.json \
  --jq '.content' | base64 -d | jq -r '.author.github'
```

篩選 `author.github` 等於當前使用者的 plugin。

### 4. 輸出格式

```
👤 <username> 的 Plugin — N 個

| 名稱 | 描述 | 版本 | 狀態 |
|------|------|------|------|
| query-mongo | 以自然語言查詢 MongoDB | v1.2.0 | ✓ 啟用中 |
| old-tool | 舊工具 | v0.1.0 | ⚠ 已棄用 |

（無結果時）
你尚未發佈任何 plugin。使用 /mp:publish 開始！
```
```

- [ ] **Step 4: Commit**

```bash
cd ../plugin-mp
git add skills/list/ skills/search/ skills/my-plugins/
git commit -m "feat: 新增 /mp:list、/mp:search、/mp:my-plugins skills"
```

---

### Task 9: /mp:publish skill

**Files:**
- Create: `../plugin-mp/skills/publish/SKILL.md`

- [ ] **Step 1: 撰寫 SKILL.md**

`../plugin-mp/skills/publish/SKILL.md`:
```markdown
---
name: publish
description: 引導式發佈新 plugin 到 Team Marketplace
---

# 發佈新 Plugin

透過對話引導使用者完成 plugin 發佈的全流程。

## 使用方式

- `/mp:publish` — 啟動發佈流程
- 「我想發佈一個新的 plugin」
- 「幫我建一個 plugin 分享給團隊」

## 發佈流程

### Step 1: 了解內容

詢問使用者：
1. 這個 plugin 做什麼？（功能描述）
2. 已經有開發好的檔案嗎？還是要從零開始？
3. 如果已有檔案，請告知檔案位置

### Step 2: 建議命名

根據描述，提供 2-3 個命名選項。命名規範：
- 格式：`<動作>-<對象>`
- 全英文小寫，`-` 分隔
- 範例：`query-mongo`、`review-code`、`gen-report`、`check-style`

讓使用者選擇或自訂。

### Step 3: 確認 type

根據內容判斷 type 並解釋：

| 類型 | 條件 | 安全等級 |
|------|------|---------|
| `skill` | 僅 SKILL.md（斜線命令） | 低風險 |
| `agent` | Agent 定義（.md） | 低風險 |
| `hook` | hooks.json（事件觸發） | 高風險 — 安裝需手動確認 |
| `mcp` | MCP Server 設定 | 高風險 — 安裝需手動確認 |
| `mixed` | 以上多種組合 | 依最高風險 |

### Step 4: 檢查重複

```bash
MARKETPLACE=$(gh api repos/cm-ailab-cc-plugins/marketplace/contents/.claude-plugin/marketplace.json \
  --jq '.content' | base64 -d)
echo "$MARKETPLACE" | jq '.plugins[].name'
```

比對名稱和 keywords，如有類似 plugin 告知使用者，確認是否繼續。

### Step 5: 完善描述和 Keywords

- 根據功能草擬描述（建議 >= 10 個字，說明具體功能）
- 建議 keywords（至少 2 個），參考現有 plugin 的 keyword 風格
- 讓使用者確認或修改

### Step 6: 確認資訊

展示完整 plugin 資訊，等待使用者確認：

```
📦 即將發佈的 Plugin:

  名稱:     query-mongo
  類型:     skill
  描述:     以自然語言查詢 MongoDB，自動生成並執行 query
  Keywords: mongodb, query, database
  作者:     Nero-2307
  版本:     1.0.0

確認發佈？(Y/n)
```

### Step 7: 執行

確認後依序執行：

#### 7.1 取得作者資訊
```bash
GITHUB_USER=$(gh api /user --jq '.login')
GITHUB_NAME=$(gh api /user --jq '.name // .login')
```

#### 7.2 建立 GitHub Repo
```bash
cd ~/CMoney/cm-ailab-cc-plugins
gh repo create cm-ailab-cc-plugins/plugin-<name> --private --clone
cd plugin-<name>
```

#### 7.3 建立 Plugin 結構

依 type 建立對應結構。以 skill 為例：

```
plugin-<name>/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── <name>/
│       └── SKILL.md
└── README.md
```

`.claude-plugin/plugin.json`:
```json
{
  "name": "<name>",
  "version": "1.0.0",
  "type": "<type>",
  "description": "<description>",
  "keywords": [<keywords>],
  "author": {
    "name": "<GITHUB_NAME>",
    "github": "<GITHUB_USER>"
  }
}
```

對其他 type 則建立對應目錄和檔案（agents/*.md、hooks/hooks.json、mcp-servers/config.json）。

#### 7.4 填入使用者內容

如果使用者已有開發好的檔案，將其複製到正確位置。
如果是新建，生成基礎範本內容。

#### 7.5 提交並推送
```bash
git add -A
git commit -m "feat: 初始化 plugin-<name>"
git tag v1.0.0
git push origin main
git push origin v1.0.0
```

#### 7.6 更新 Marketplace

```bash
cd ~/CMoney/cm-ailab-cc-plugins/marketplace
git checkout main
git pull origin main
git checkout -b add-plugin-<name>
```

在 `.claude-plugin/marketplace.json` 的 `plugins` 陣列末尾加入：
```json
{
  "name": "<name>",
  "source": {
    "source": "github",
    "repo": "cm-ailab-cc-plugins/plugin-<name>",
    "ref": "v1.0.0"
  },
  "description": "<description>",
  "keywords": [<keywords>]
}
```

```bash
git add .claude-plugin/marketplace.json
git commit -m "feat: 新增 plugin <name>"
git push origin add-plugin-<name>

gh pr create \
  --repo cm-ailab-cc-plugins/marketplace \
  --title "Add plugin: <name>" \
  --body "## 新增 Plugin

- **名稱**: <name>
- **類型**: <type>
- **描述**: <description>
- **作者**: <GITHUB_USER>
- **Repo**: cm-ailab-cc-plugins/plugin-<name>
- **版本**: v1.0.0"
```

#### 7.7 回報結果

```
✅ Plugin 發佈流程完成！

📦 Plugin repo: https://github.com/cm-ailab-cc-plugins/plugin-<name>
📋 Marketplace PR: <PR URL>

下一步：
1. CI 會自動驗證 plugin 結構
2. 等待 reviewer 審核
3. 合併後全員可用
```

切回 main branch：
```bash
git checkout main
```
```

- [ ] **Step 2: Commit**

```bash
cd ../plugin-mp
git add skills/publish/
git commit -m "feat: 新增 /mp:publish skill"
```

---

### Task 10: /mp:update skill

**Files:**
- Create: `../plugin-mp/skills/update/SKILL.md`

- [ ] **Step 1: 撰寫 SKILL.md**

`../plugin-mp/skills/update/SKILL.md`:
```markdown
---
name: update
description: 更新已發佈的 plugin 版本
---

# 更新 Plugin

## 使用方式

- `/mp:update <name>` — 更新指定 plugin
- `/mp:update` — 自動偵測當前目錄的 plugin
- 「更新我的 query-mongo plugin」

## 更新流程

### Step 1: 確認目標 Plugin

如果使用者指定了名稱，在 marketplace.json 中查找。
如果未指定，檢查當前目錄是否為 plugin repo（有 `.claude-plugin/plugin.json`）。

讀取目前版本：
```bash
jq -r '.version' .claude-plugin/plugin.json
```

### Step 2: 確認版本變更

詢問版本 bump 類型：
- **patch** (x.y.Z) — bug fix、小修正
- **minor** (x.Y.0) — 新功能、向後相容
- **major** (X.0.0) — 破壞性變更

根據 git log 建議 bump 類型：
```bash
git log --oneline $(git describe --tags --abbrev=0)..HEAD
```

### Step 3: 確認變更內容

展示自上次 tag 以來的變更摘要，讓使用者確認。

### Step 4: 執行

#### 4.1 更新版本號
使用 jq 更新 `.claude-plugin/plugin.json` 的 `version` 欄位：
```bash
NEW_VERSION="<new-version>"
jq ".version = \"$NEW_VERSION\"" .claude-plugin/plugin.json > tmp.json && mv tmp.json .claude-plugin/plugin.json
```

#### 4.2 提交並推送
```bash
git add .claude-plugin/plugin.json
git commit -m "chore: bump version to $NEW_VERSION"
git tag "v$NEW_VERSION"
git push origin main
git push origin "v$NEW_VERSION"
```

#### 4.3 更新 Marketplace

```bash
cd ~/CMoney/cm-ailab-cc-plugins/marketplace
git checkout main
git pull origin main
git checkout -b update-plugin-<name>-v$NEW_VERSION
```

更新 `.claude-plugin/marketplace.json` 中對應 plugin 的 `ref` 為 `v$NEW_VERSION`。

```bash
git add .claude-plugin/marketplace.json
git commit -m "feat: 更新 plugin <name> 至 v$NEW_VERSION"
git push origin update-plugin-<name>-v$NEW_VERSION

gh pr create \
  --repo cm-ailab-cc-plugins/marketplace \
  --title "Update plugin: <name> → v$NEW_VERSION" \
  --body "## 更新 Plugin

- **名稱**: <name>
- **版本**: <old-version> → v$NEW_VERSION
- **變更摘要**: <changes summary>"
```

#### 4.4 回報結果

```
✅ Plugin 更新完成！

📦 新版本: v<NEW_VERSION>
📋 Marketplace PR: <PR URL>
```

切回 main：
```bash
git checkout main
```
```

- [ ] **Step 2: Commit**

```bash
cd ../plugin-mp
git add skills/update/
git commit -m "feat: 新增 /mp:update skill"
```

---

### Task 11: /mp:deprecate skill

**Files:**
- Create: `../plugin-mp/skills/deprecate/SKILL.md`

- [ ] **Step 1: 撰寫 SKILL.md**

`../plugin-mp/skills/deprecate/SKILL.md`:
```markdown
---
name: deprecate
description: 標記 plugin 為棄用
---

# 棄用 Plugin

## 使用方式

- `/mp:deprecate <name>` — 棄用指定 plugin
- 「我想淘汰 old-tool 這個 plugin」

## 棄用流程

### Step 1: 確認目標

從 marketplace.json 中查找指定 plugin。如果找不到，回報錯誤。

### Step 2: 詢問替代方案

- 「是否有替代的 plugin？」
- 如果有，記錄 `replacement` 欄位

### Step 3: 確認

```
⚠ 即將棄用:

  名稱:   old-tool
  替代:   new-tool（選填）

棄用後的影響:
  - /mp:search 預設不顯示（加 --all 才顯示）
  - /mp:list 顯示但標記為已棄用
  - 已安裝的使用者：繼續運作，啟動時顯示警告
  - 新安裝：允許但顯示警告

確認棄用？(Y/n)
```

### Step 4: 執行

```bash
cd ~/CMoney/cm-ailab-cc-plugins/marketplace
git checkout main
git pull origin main
git checkout -b deprecate-plugin-<name>
```

在 `.claude-plugin/marketplace.json` 中對應的 plugin entry 加入：
```json
{
  "name": "<name>",
  "deprecated": true,
  "replacement": "<replacement-name>",
  ...
}
```

（`replacement` 僅在有替代方案時加入）

```bash
git add .claude-plugin/marketplace.json
git commit -m "feat: 棄用 plugin <name>"
git push origin deprecate-plugin-<name>

gh pr create \
  --repo cm-ailab-cc-plugins/marketplace \
  --title "Deprecate plugin: <name>" \
  --body "## 棄用 Plugin

- **名稱**: <name>
- **替代**: <replacement or '無'>
- **理由**: <user's reason>"
```

### Step 5: 回報結果

```
✅ 棄用 PR 已建立！

📋 PR: <PR URL>
合併後 <name> 將被標記為已棄用。
```
```

- [ ] **Step 2: Commit**

```bash
cd ../plugin-mp
git add skills/deprecate/
git commit -m "feat: 新增 /mp:deprecate skill"
```

---

## Phase 3: 整合與驗證

### Task 12: 建立 plugin-mp GitHub Repo

**Files:** 無新增檔案（Git/GitHub 操作）

- [ ] **Step 1: 建立遠端 repo**

```bash
cd ~/CMoney/cm-ailab-cc-plugins/plugin-mp
gh repo create cm-ailab-cc-plugins/plugin-mp --private --source=. --push
```

- [ ] **Step 2: 建立 v1.0.0 tag 並推送**

```bash
git tag v1.0.0
git push origin v1.0.0
```

- [ ] **Step 3: 驗證遠端 repo**

```bash
gh repo view cm-ailab-cc-plugins/plugin-mp --json name,url
```
Expected: 顯示 repo 資訊

---

### Task 13: 註冊 plugin-mp 到 marketplace 並驗證

**Files:**
- Modify: `.claude-plugin/marketplace.json`

- [ ] **Step 1: 更新 marketplace.json**

在 `plugins` 陣列中加入 mp plugin：

```json
{
  "name": "cm-ailab-cc-plugins",
  "owner": {
    "name": "CMoney AILab",
    "email": "ailab@cmoney.com.tw"
  },
  "metadata": {
    "description": "CMoney AILab 團隊共用的 Claude Code Plugin Marketplace",
    "version": "1.0.0"
  },
  "plugins": [
    {
      "name": "mp",
      "source": {
        "source": "github",
        "repo": "cm-ailab-cc-plugins/plugin-mp",
        "ref": "v1.0.0"
      },
      "description": "Team Marketplace 管理工具 — 發佈、搜尋、安裝、更新 plugin",
      "keywords": ["marketplace", "management", "plugin", "team"]
    }
  ]
}
```

- [ ] **Step 2: 用驗證腳本檢查 plugin-mp**

```bash
cd ~/CMoney/cm-ailab-cc-plugins/marketplace
bash scripts/validate-plugin.sh ../plugin-mp plugin-mp v1.0.0
```

Expected: `Validation PASSED`

- [ ] **Step 3: Commit marketplace 變更**

```bash
git add .claude-plugin/marketplace.json
git commit -m "feat: 註冊 plugin-mp 為第一個 marketplace plugin"
```

- [ ] **Step 4: 推送 marketplace repo**

```bash
git push origin main
```

- [ ] **Step 5: 端到端驗證**

1. 確認 marketplace.json 格式正確：
   ```bash
   jq . .claude-plugin/marketplace.json
   ```

2. 確認 plugin-mp repo 可透過 gh 存取：
   ```bash
   gh api repos/cm-ailab-cc-plugins/plugin-mp/git/ref/tags/v1.0.0 --jq '.ref'
   ```
   Expected: `refs/tags/v1.0.0`

3. 確認驗證腳本測試全部通過：
   ```bash
   bash tests/run-tests.sh
   ```
   Expected: 全部 PASS
