# How to Use Custom Slash Commands & Skills

This guide explains how to use the custom Claude Code setup for building cohesive UIs in the Digital Dirham Wallet application.

---

## Table of Contents

1. [Understanding the Setup](#understanding-the-setup)
2. [How CLAUDE.md Works](#how-claudemd-works)
3. [How Slash Commands Work](#how-slash-commands-work)
4. [Running Slash Commands](#running-slash-commands)
5. [Command Reference](#command-reference)
6. [Example Workflows](#example-workflows)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)

---

## Understanding the Setup

The setup consists of two main parts:

### 1. CLAUDE.md (Always Active)

**Location:** `/wallet/CLAUDE.md`

This file contains base rules that Claude agents **automatically follow** in every conversation. You don't need to invoke it - it's always active when working in the wallet directory.

**What it enforces:**
- Always use dc-ui-toolkit components
- Never hardcode colors or spacing
- Screen wrapper pattern
- Custom hook pattern for screen logic
- Typography system usage
- Design tokens

### 2. Slash Commands (On-Demand)

**Location:** `/.claude/commands/` (at repository root)

These are specialized skills you invoke when needed for specific tasks.

| File | Command | Purpose |
|------|---------|---------|
| `build-ui.md` | `/project:build-ui` | Main orchestrator |
| `build-screen.md` | `/project:build-screen` | Create screens |
| `build-component.md` | `/project:build-component` | Create components |
| `add-to-toolkit.md` | `/project:add-to-toolkit` | Add to toolkit |
| `design-review.md` | `/project:design-review` | Review code |
| `patterns-reference.md` | `/project:patterns-reference` | Quick reference |

> **Note:** Custom slash commands must be placed in `.claude/commands/` at the repository root level for Claude Code to discover them.

---

## How CLAUDE.md Works

The `CLAUDE.md` file is automatically loaded by Claude Code when you start a conversation in the wallet directory.

**You don't need to do anything special** - the rules are applied automatically.

### What Happens Automatically:

1. Claude reads `CLAUDE.md` at conversation start
2. All rules become active constraints
3. Agent follows patterns without you asking
4. Toolkit-first approach is enforced

### Example Without Any Command:

```
You: "Create a login screen"

Claude: (automatically applies CLAUDE.md rules)
- Uses LinearGradient wrapper
- Uses Text from dc-ui-toolkit
- Creates useLoginScreen hook
- Uses theme.palette for colors
- Uses theme.spaces for spacing
```

---

## How Slash Commands Work

Slash commands are invoked when you need **specific guidance** or want to follow a **detailed template**.

### Invoking Commands:

```bash
# In Claude Code, type:
/project:command-name

# Examples:
/project:build-ui
/project:build-screen
/project:build-component
/project:design-review
```

### Command vs CLAUDE.md:

| CLAUDE.md | Slash Commands |
|-----------|----------------|
| Always active | Invoked on demand |
| Base rules | Detailed templates |
| Constraints | Step-by-step guidance |
| Brief | Comprehensive |

---

## Running Slash Commands

### Method 1: Type the Command

In Claude Code chat, simply type:

```
/project:build-screen
```

Then describe what you want:

```
/project:build-screen Create a settings screen with profile info, theme toggle, and logout button
```

### Method 2: Ask Claude to Use It

```
Use /project:build-screen to create a new notifications screen
```

### Method 3: Reference in Context

```
I need a new screen. Follow the /project:build-screen patterns to create a wallet details screen.
```

---

## Command Reference

### `/project:build-ui` - Main Orchestrator

**When to use:** When you're not sure which specific command to use, or for general UI tasks.

**What it does:**
- Analyzes your request
- Routes to appropriate sub-command
- Provides toolkit component inventory
- Enforces design system

**Example:**
```
/project:build-ui I need to add a card showing transaction details
```

---

### `/project:build-screen` - Create Screens

**When to use:** Creating new screens from scratch.

**What it provides:**
- Complete screen template with wrapper
- Custom hook template
- Test file template
- Navigation registration checklist
- Translation key setup

**Example:**
```
/project:build-screen Create a TransactionHistoryScreen that shows a list of past transactions with filtering
```

**Files Created:**
```
src/scenes/TransactionHistory/
├── TransactionHistoryScreen.tsx
├── hooks/useTransactionHistory.ts
├── components/ (if needed)
└── __tests__/TransactionHistoryScreen.test.tsx
```

---

### `/project:build-component` - Create Components

**When to use:** Creating reusable components (not full screens).

**What it provides:**
- Component template with TypeScript
- Props interface pattern
- Themed styling pattern
- Composition patterns
- Performance optimization tips

**Example:**
```
/project:build-component Create a StatusBadge component that shows success/error/pending states
```

---

### `/project:add-to-toolkit` - Add to dc-ui-toolkit

**When to use:** Adding new reusable components to the design system.

**IMPORTANT:** This command will **ask for your approval** before creating files.

**What it provides:**
- Toolkit component template with withTheme HOC
- Category selection guide
- Index file patterns
- Export patterns

**Example:**
```
/project:add-to-toolkit Add a ProgressBar component to the toolkit
```

**Process:**
1. Claude proposes the component
2. Shows you the implementation
3. Asks: "Should I proceed?"
4. You approve or modify
5. Files are created

---

### `/project:design-review` - Review Code

**When to use:** After implementing UI, to check design system compliance.

**What it provides:**
- Component usage review
- Typography review
- Color usage review
- Spacing review
- Screen structure review
- Compliance scoring (X/10)

**Example:**
```
/project:design-review Review the DashboardScreen for design system compliance
```

**Output Format:**
```
## Design Review: DashboardScreen

### Critical Issues (Must Fix)
- [Issue with fix]

### Warnings (Should Fix)
- [Warning with recommendation]

### Compliance Score
| Category | Score |
|----------|-------|
| Typography | 8/10 |
| Colors | 9/10 |
| Overall | 8/10 |
```

---

### `/project:patterns-reference` - Quick Reference

**When to use:** Need quick lookup of patterns, tokens, or conventions.

**What it provides:**
- Import aliases
- Screen wrapper pattern
- Hook pattern
- Typography usage
- Button variants
- Form input patterns
- Design tokens reference

**Example:**
```
/project:patterns-reference Show me the button variants
```

---

## Example Workflows

### Workflow 1: Create a New Feature Screen

```
Step 1: Use build-screen command
> /project:build-screen Create a ProfileSettingsScreen with edit profile, change password, and notification preferences sections

Step 2: Claude creates the screen structure following all patterns

Step 3: Review with design-review
> /project:design-review Review the ProfileSettingsScreen
```

### Workflow 2: Add a Reusable Component

```
Step 1: Start with build-component to see if it should be toolkit or local
> /project:build-component Create a ConfirmationModal for confirming actions

Step 2: If Claude suggests it should go in toolkit:
> /project:add-to-toolkit Add the ConfirmationModal to dc-ui-toolkit

Step 3: Approve when Claude asks
```

### Workflow 3: Quick UI Fix

```
(No command needed - CLAUDE.md rules apply automatically)

> Fix the spacing on the login screen - the buttons are too close together

Claude will automatically:
- Use theme.spaces for spacing
- Follow existing patterns
- Not hardcode values
```

### Workflow 4: Code Review Before PR

```
> /project:design-review Review all files I changed in this PR

Claude reviews and provides:
- Issues to fix
- Compliance scores
- Specific line references
```

---

## Best Practices

### 1. Let CLAUDE.md Handle Basics

Don't invoke commands for simple changes. CLAUDE.md rules automatically apply.

```
# Good - simple request, auto-follows rules
> Add a subtitle to the dashboard header

# Unnecessary - overkill for simple change
> /project:build-ui Add a subtitle to the dashboard header
```

### 2. Use Commands for New Features

Invoke commands when creating something new:

```
# Good - new screen needs full template
> /project:build-screen Create a wallet backup screen

# Good - new component needs patterns
> /project:build-component Create a QR scanner overlay
```

### 3. Always Review Before PR

```
> /project:design-review Review all changes before I submit the PR
```

### 4. Reference Patterns When Stuck

```
> /project:patterns-reference How do I use the AmountInput component?
```

### 5. Ask for Toolkit Approval

Never let Claude auto-add to toolkit. The command enforces this, but be explicit:

```
> Create a component, but ask me before adding to toolkit
```

---

## Troubleshooting

### Command Not Found

If `/project:build-ui` doesn't work:

1. Make sure `.claude/commands/` exists **at the repository root** (not in a subdirectory)
2. Verify the markdown files are present in the root `.claude/commands/` directory
3. Restart Claude Code to reload the commands

### Rules Not Being Followed

If Claude isn't following CLAUDE.md rules:

1. Check `wallet/CLAUDE.md` exists
2. Start a new conversation (CLAUDE.md loads at start)
3. Explicitly mention: "Follow the CLAUDE.md rules"

### Wrong Patterns Being Used

If Claude uses wrong patterns:

1. Use `/project:patterns-reference` to show correct patterns
2. Point to reference files:
   - `src/scenes/DashboardScreen/DashboardScreen.tsx`
   - `src/scenes/Transfer/hooks/useTransferScreen.ts`

---

## File Structure Summary

```
mithril/                          # Repository root
├── .claude/
│   ├── commands/                 # Custom slash commands (MUST be at root)
│   │   ├── build-ui.md          # /project:build-ui
│   │   ├── build-screen.md      # /project:build-screen
│   │   ├── build-component.md   # /project:build-component
│   │   ├── add-to-toolkit.md    # /project:add-to-toolkit
│   │   ├── design-review.md     # /project:design-review
│   │   └── patterns-reference.md # /project:patterns-reference
│   └── how-to-use-setup.md      # This guide
└── wallet/
    ├── CLAUDE.md                # Base rules (always active for wallet dir)
    └── src/
        └── components/
            └── dc-ui-toolkit/   # The design system toolkit
```

---

## Quick Command Cheat Sheet

| Task | Command |
|------|---------|
| Create new screen | `/project:build-screen` |
| Create component | `/project:build-component` |
| Add to toolkit | `/project:add-to-toolkit` |
| Review code | `/project:design-review` |
| Look up patterns | `/project:patterns-reference` |
| General UI task | `/project:build-ui` |
| Simple changes | No command needed (CLAUDE.md applies) |
