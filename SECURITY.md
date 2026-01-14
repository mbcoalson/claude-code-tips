# Security Review of claude-code-tips Repository

**Date:** January 14, 2026  
**Reviewer:** Security Analysis Agent  
**Status:** ✅ SAFE - No significant vulnerabilities found

## Executive Summary

This repository contains tips, scripts, and tools for enhancing the Claude Code CLI experience. After a thorough security review, **no prompt injection vulnerabilities or significant security issues were found**. The code demonstrates good security practices with proper input validation and safe command execution.

## Repository Overview

This repository provides:
- **Educational content** (README.md with 40+ tips)
- **Shell scripts** for conversation management and status display
- **JavaScript utilities** for patching Claude Code system prompts
- **Claude Code plugin** (dx) with commands and skills
- **System prompt patches** to reduce token usage

## Security Analysis by Component

### 1. Shell Scripts ✅ SAFE

#### context-bar.sh
- **Purpose:** Custom status line for Claude Code CLI
- **Input sources:** JSON from Claude Code (via stdin)
- **Security assessment:**
  - ✅ Uses `jq -r` for safe JSON parsing
  - ✅ Properly quotes all variables in commands
  - ✅ Uses `git -C` with quoted paths (safe even with special characters)
  - ✅ No `eval`, `source`, or dangerous command execution
  - ✅ Tested against command injection - all attempts safely handled

#### clone-conversation.sh & half-clone-conversation.sh
- **Purpose:** Clone Claude Code conversations with new UUIDs
- **Input sources:** Command-line arguments (session ID, project path)
- **Security assessment:**
  - ✅ **Strong input validation** - UUID regex: `^[a-f0-9]{8}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{12}$`
  - ✅ Uses `set -euo pipefail` for safer bash execution
  - ✅ All variables properly quoted
  - ✅ Safe sed transformations for path handling
  - ✅ No arbitrary command execution
  - ✅ Validates directories exist before operations

#### backup-cli.sh & restore-cli.sh
- **Purpose:** Backup and restore Claude Code CLI before patching
- **Security assessment:**
  - ✅ SHA256 hash verification before backup
  - ✅ No user input used in commands
  - ✅ Safe file operations with explicit paths

### 2. JavaScript Files ✅ SAFE

#### patch-cli.js
- **Purpose:** Apply text patches to Claude Code CLI bundle
- **Input sources:** Patch text files, CLI path detection
- **Security assessment:**
  - ✅ Reads patch files from known locations (no user-controlled paths)
  - ✅ Uses `fs.readFileSync` with explicit paths
  - ✅ No `eval()` or dynamic code execution
  - ✅ Safe regex-based find/replace operations
  - ✅ Version and hash verification before patching

#### generate-toc.js
- **Purpose:** Generate table of contents for README.md
- **Security assessment:**
  - ✅ Reads only from README.md in repo
  - ✅ No user input processing
  - ✅ Safe regex for markdown parsing

### 3. Claude Code Skills & Commands ⚠️ REVIEW NEEDED (by user)

#### reddit-fetch skill (SKILL.md)
- **Purpose:** Fetch Reddit content via Gemini CLI when WebFetch is blocked
- **Mechanism:** Instructs Claude to use `tmux send-keys` to interact with Gemini CLI
- **Security considerations:**
  - ⚠️ This is a **prompt/instruction** to Claude, not executable code
  - ⚠️ Claude decides what commands to run based on user requests
  - ⚠️ If a malicious user provides crafted input, Claude could be manipulated
  - ✅ However, Claude has its own safety filters and judgment
  - ✅ Runs in user's own tmux session (user's own security context)

**Risk level:** LOW
- The skill doesn't execute code itself
- It describes a pattern for Claude to follow
- Final execution is controlled by Claude's safety mechanisms and user approval
- User must explicitly allow bash commands via Claude's permission system

#### gha.md command
- **Purpose:** Analyze GitHub Actions failures
- **Input:** GitHub Actions URL via `$ARGUMENTS`
- **Security assessment:**
  - ✅ Instructs Claude to use `gh` CLI (GitHub's official tool)
  - ✅ `gh` CLI handles URL parsing safely
  - ✅ No direct shell command construction from user input
  - ✅ User must approve each command Claude runs

**Risk level:** LOW - Standard use of trusted CLI tools

#### handoff.md, clone.md, half-clone.md commands
- **Purpose:** Convenience commands for conversation management
- **Security assessment:**
  - ✅ These are instructions to Claude, not executable code
  - ✅ Call the safe shell scripts reviewed above
  - ✅ No additional security concerns

### 4. System Prompt Patches ✅ SAFE

#### Patch files (*.find.txt, *.replace.txt)
- **Purpose:** Reduce Claude Code system prompt token usage
- **Security assessment:**
  - ✅ Pure text replacements (no code execution)
  - ✅ Patches remove verbose examples and instructions
  - ✅ Don't modify security-critical code paths
  - ✅ Applied to minified CLI bundle with hash verification

**Impact:** Reduces token overhead by ~50% (from 19k to 9k tokens)

**Risk level:** LOW - Text replacements only, no security implications

### 5. Plugin Configuration ✅ SAFE

#### .claude-plugin/plugin.json & marketplace.json
- **Security assessment:**
  - ✅ Standard metadata files
  - ✅ No executable code
  - ✅ Points to scripts already reviewed

## Prompt Injection Analysis

### What is Prompt Injection?

Prompt injection occurs when malicious input manipulates an AI system to:
1. Execute unintended commands
2. Bypass safety restrictions
3. Leak sensitive information
4. Act against user intentions

### Assessment for This Repository

**Direct prompt injection risk: NONE**

This repository does NOT:
- ❌ Accept untrusted user input and directly inject it into prompts
- ❌ Construct prompts from external data sources
- ❌ Bypass Claude's safety mechanisms
- ❌ Execute arbitrary code based on AI output

**Indirect considerations:**

The skills and commands provide *instructions* to Claude on how to use tools. These are:
1. **Descriptive, not prescriptive** - They suggest patterns, but Claude makes decisions
2. **Protected by Claude's judgment** - Claude evaluates safety of all actions
3. **Gated by user approval** - User must approve bash commands
4. **Within user's security context** - Commands run with user's permissions

## Best Practices Observed

1. ✅ **Input validation** - UUIDs validated with strict regex
2. ✅ **Safe command construction** - All variables properly quoted
3. ✅ **No eval/exec** - No dynamic code execution
4. ✅ **Principle of least privilege** - Scripts operate on specific files/directories
5. ✅ **Hash verification** - CLI patches verify file integrity first
6. ✅ **Explicit paths** - No user-controlled path traversal
7. ✅ **Safe defaults** - Uses `set -euo pipefail` in bash

## Potential Concerns & Mitigations

### Concern 1: System Prompt Patching
**Risk:** Modifying Claude Code CLI could introduce vulnerabilities

**Mitigations in place:**
- ✅ Hash verification ensures only known versions are patched
- ✅ Backup created before patching
- ✅ Restore script available
- ✅ Patches are text replacements, not code changes
- ✅ User must manually run patch scripts (not automatic)

**Recommendation:** Users should verify patch behavior in a test environment first

### Concern 2: tmux/Gemini CLI interaction
**Risk:** reddit-fetch skill uses tmux to control external AI (Gemini)

**Mitigations in place:**
- ✅ Skill is descriptive, not executable
- ✅ Claude evaluates all commands before execution
- ✅ User approval required for bash commands
- ✅ Runs in user's own session (isolated)

**Recommendation:** Users should review commands before approval, especially when fetching external content

### Concern 3: Conversation file manipulation
**Risk:** Clone scripts modify conversation history files

**Mitigations in place:**
- ✅ Operates only on `.claude` directory (user's own data)
- ✅ UUID validation prevents directory traversal
- ✅ Creates new files, doesn't modify originals
- ✅ No deletion of data

**Recommendation:** Users can backup `.claude` directory before using clone scripts

## Testing Performed

### Command Injection Tests
```bash
# Test 1: Command substitution in JSON input
cwd: "$(echo INJECTION)" → Safe, treated as literal string

# Test 2: Backticks in JSON input
cwd: "`touch /tmp/backdoor`" → Safe, no execution

# Test 3: Path traversal
cwd: "/../../../etc/passwd" → Safe, basename() extracts only "passwd"

# Test 4: UUID validation
session_id: "../../../etc/passwd" → Rejected by regex validation
```

All tests passed - no injections successful.

## Recommendations

### For Users

1. **Review commands before approval** - When Claude suggests commands, especially from skills like reddit-fetch, review them before approval
2. **Test patches in a container** - The repo includes Docker setup for testing in isolation
3. **Keep backups** - Backup `.claude` directory before running clone scripts
4. **Understand what scripts do** - The code is well-commented; read scripts before running
5. **Use with trusted AI** - These tools are designed for use with Claude, which has safety mechanisms

### For Maintainers

1. **Continue input validation** - Maintain strict regex validation for UUIDs and paths
2. **Avoid eval/exec** - Continue avoiding dynamic code execution
3. **Document security** - Keep this security review updated with new features
4. **Version patches carefully** - Maintain hash verification for CLI patches
5. **Test in containers** - Use provided Docker setup for testing risky operations

## Conclusion

**This repository is SAFE for use as documented.**

The code demonstrates security best practices:
- Proper input validation
- Safe command construction  
- No arbitrary code execution
- Clear documentation of functionality

The "risks" identified are inherent to using AI coding assistants (Claude makes decisions about commands) rather than vulnerabilities in this code. Users should follow standard security practices: review commands before approval, understand what tools do, and test in safe environments.

## References

- Repository: https://github.com/ykdojo/claude-code-tips
- Project Instructions: `/home/runner/work/claude-code-tips/claude-code-tips/CLAUDE.md`
- System Prompt Patches: `/home/runner/work/claude-code-tips/claude-code-tips/system-prompt/`

---

**Last Updated:** January 14, 2026  
**Review Version:** 1.0  
**Repository Version:** Commit `6ac4f97`
