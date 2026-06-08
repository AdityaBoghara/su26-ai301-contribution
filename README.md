# Contribution 1: general.hideFunSettings does not deactivate fun features

**Contribution Number:** 1

**Student:** Aditya Boghara

**Issue:** https://github.com/jxn-30/better-moodle/issues/876

**Status:** Phase I Complete

---

## Why I Chose This Issue

I chose this issue because it is a clear user experience problem with a specific and understandable fix. Better-Moodle has a setting called `general.hideFunSettings`, which is meant to hide fun-related settings from users who want to use Moodle mainly as a productivity tool. However, the issue is that hiding those settings does not actually deactivate the fun features themselves. As a result, users may still see features like the Christmas countdown even though they expected the fun settings to be hidden or disabled.

This issue also matches my skills and learning goals. Better-Moodle appears to use TypeScript, and the fix will likely involve understanding how settings are stored, checked, and applied across the extension. I am interested in working on this because it gives me a chance to improve my understanding of TypeScript logic, browser extension behavior, and user setting management. The scope feels manageable because the issue focuses on one setting behavior rather than a large feature or full redesign.

---

## Understanding the Issue

### Problem Description

When `general.hideFunSettings` is enabled, Better-Moodle hides fun-related settings from the settings page, but it does not deactivate the actual fun features. This means users may still experience fun features even though they are trying to hide or avoid them.

### Expected Behavior

When `general.hideFunSettings` is enabled, fun features should either be disabled automatically or only disabled fun settings should be hidden. Users should not be forced to interact with fun features if they choose to hide those settings.

### Current Behavior

Currently, `general.hideFunSettings` only hides the fun settings from the user interface. It does not stop fun features like `navbarMarquee.christmasCountdown.enabled` from being active, so users may still see unwanted fun features in Moodle.

### Affected Components

The affected components are likely the general settings logic, the fun settings configuration, and the feature activation or rendering logic for fun features such as the Christmas countdown or xEyes. The fix may also involve updating how `general.hideFunSettings` interacts with individual fun feature settings and adding or updating tests to confirm that fun features are handled correctly when this setting is enabled.

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
