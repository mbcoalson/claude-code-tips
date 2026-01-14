# Security Self-Review Checklist

Use this checklist to verify the security of this repository yourself. All items should pass.

## Quick Checks (5 minutes)

- [ ] **No hardcoded secrets**: `grep -r "password\|api_key\|secret" . --include="*.sh" --include="*.js"` returns only false positives
- [ ] **No eval usage**: `grep -r "eval" . --include="*.sh" --include="*.js"` returns no results
- [ ] **Input validation present**: Check that `clone-conversation.sh` validates UUIDs with regex (line 336)
- [ ] **Variables quoted**: Shell scripts use `"$variable"` not `$variable`
- [ ] **Safe error handling**: Scripts use `set -euo pipefail`

## Shell Script Security (10 minutes)

### context-bar.sh
- [ ] Uses `jq -r` to safely parse JSON (lines 27-28)
- [ ] All git commands use `-C "$cwd"` with quotes (lines 35, 38, 42, 64, 89)
- [ ] No command substitution with user input
- [ ] Uses `basename` to sanitize directory names (line 29)

### clone-conversation.sh & half-clone-conversation.sh
- [ ] UUID validated with strict regex pattern (line 336)
- [ ] Uses `set -euo pipefail` (line 22)
- [ ] All path operations use variables in quotes
- [ ] `convert_path_to_dirname` uses safe sed replacements (line 67)
- [ ] No arbitrary file operations outside ~/.claude

### backup-cli.sh & restore-cli.sh
- [ ] SHA256 hash verification before operations (line 70)
- [ ] No user input used directly in commands
- [ ] Fails safely if files don't exist

## JavaScript Security (5 minutes)

### patch-cli.js
- [ ] No `eval()` or `Function()` usage
- [ ] Reads files from explicit paths only (line 87-92)
- [ ] Uses `fs.readFileSync` not `require()` for data
- [ ] Regex replacements are safe (no ReDoS patterns)

### generate-toc.js
- [ ] Only reads README.md (line 21)
- [ ] No user input processing
- [ ] Safe regex patterns (lines 25-26)

## Skills & Commands (5 minutes)

### reddit-fetch (skills/reddit-fetch/SKILL.md)
- [ ] Provides instructions, doesn't execute code
- [ ] Uses tmux (controlled by Claude, approved by user)
- [ ] No direct command construction from URLs

### gha.md (commands/gha.md)
- [ ] Uses `gh` CLI (official GitHub tool)
- [ ] `$ARGUMENTS` passed to gh commands (gh handles URL parsing safely)
- [ ] No shell command construction from user input

### handoff.md, clone.md, half-clone.md
- [ ] Call reviewed shell scripts
- [ ] No additional security concerns

## System Prompt Patches (5 minutes)

- [ ] Patches are text files (*.find.txt, *.replace.txt)
- [ ] No executable code in patches
- [ ] patch-cli.js verifies hash before patching (line 70)
- [ ] Backup created automatically before patching (line 84)
- [ ] Restore script available

## Manual Testing (10 minutes)

### Test 1: Command injection in context-bar.sh
```bash
cat << 'JSON' | bash scripts/context-bar.sh
{
  "model": {"display_name": "Test"},
  "cwd": "$(echo MALICIOUS)",
  "transcript_path": "",
  "context_window": {"context_window_size": 200000}
}
JSON
```
- [ ] Output shows `$(echo MALICIOUS)` as literal text, not executed

### Test 2: UUID validation in clone script
```bash
bash scripts/clone-conversation.sh "../../../etc/passwd"
```
- [ ] Script exits with "Invalid session ID format" error

### Test 3: Path validation in clone script
```bash
bash scripts/clone-conversation.sh "d96c899d-7501-4e81-a31b-e0095bb3b501" "../../.."
```
- [ ] Script safely handles the path (no directory traversal)

### Test 4: Check for backdoors after tests
```bash
ls /tmp/backdoor 2>/dev/null && echo "FAIL: Backdoor created" || echo "PASS: No backdoors"
```
- [ ] Reports "PASS: No backdoors"

## Additional Verifications

- [ ] `.gitignore` excludes sensitive files (check `.gitignore` exists)
- [ ] No `.env` files with secrets committed
- [ ] Docker container setup documented (see `container/` directory)
- [ ] Scripts have clear comments explaining what they do
- [ ] README warns about reviewing commands before approval

## Red Flags (Should NOT Find Any)

If you find any of these, investigate immediately:
- ❌ `eval` statements
- ❌ Unquoted variables in shell scripts: `$var` instead of `"$var"`
- ❌ `source` or `.` with user-controlled paths
- ❌ Command substitution with external input: `$(user_input)`
- ❌ Direct use of `curl | sh` or similar patterns
- ❌ Hardcoded passwords, API keys, or tokens
- ❌ Weak input validation or no validation
- ❌ SQL queries (this repo doesn't use databases, so shouldn't exist)

## Results

All checks should pass. If any fail, please:
1. Open an issue on GitHub
2. Do not use the affected script until fixed
3. Check SECURITY.md for mitigation strategies

---

**Last Updated:** January 14, 2026  
**Review Checklist Version:** 1.0
