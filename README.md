# Contribution 1: general.hideFunSettings does not deactivate fun features

**Contribution Number:** 1

**Student:** Aditya Boghara

**Issue:** https://github.com/jxn-30/better-moodle/issues/876

**Status:** Phase II Complete

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

````md
# Contribution 1: general.hideFunSettings does not deactivate fun features

**Contribution Number:** 1

**Student:** Aditya Boghara

**Issue:** https://github.com/jxn-30/better-moodle/issues/876

**Status:** Phase II Complete

---

## Reproduction Process

### Environment Setup

I set up the Better-Moodle project locally by following the repository’s contribution instructions. I cloned the repository, enabled Yarn through Corepack, used the Node version specified in `.nvmrc`, and installed the project dependencies with Yarn.

Commands used:

```bash
git clone https://github.com/jxn-30/better-moodle.git
cd better-moodle
corepack enable
nvm use
yarn install
````

After setting up the project, I created a working branch for this issue:

```bash
git checkout -b fix-hide-fun-settings
```

I also built the userscript using the UzL config:

```bash
yarn build uzl
```

The build generated a userscript in the `dist/` folder. I installed the generated userscript in Tampermonkey to check that it loaded in the browser. Tampermonkey showed that the script was enabled and targeted:

```text
https://moodle.uni-luebeck.de/*
```

One challenge I faced was that full browser testing required access to the supported Moodle instance. Since the UzL userscript runs on `moodle.uni-luebeck.de`, I was able to install and inspect the userscript in Tampermonkey, but I could not fully test the logged-in Moodle workflow without valid Moodle credentials.

Another challenge I faced was linting. Running the lint command showed import sorting warnings in the file I edited. I resolved those by reordering the imports in `src/features/navbarMarquee/christmasCountdown.tsx`. Linting also reported TypeScript ESLint errors in unrelated files such as `SliderSetting.tsx` and `i18n.ts`. Since those files were outside the scope of this issue and were not caused by my change, I documented them separately instead of modifying unrelated code.

Commands used for linting and formatting:

```bash
yarn run lint:fix
yarn run prettier:write
```

### Steps to Reproduce

1. Open the Better-Moodle source code.

2. Locate the Christmas countdown feature file:

```text
src/features/navbarMarquee/christmasCountdown.tsx
```

3. Review the Christmas countdown setting:

```ts
const enabled = new BooleanSetting('enabled', true)
    .addAlias('general.christmasCountdown')
    .addTag('fun');
```

This shows that the Christmas countdown is a fun feature and is enabled by default.

4. Locate the general settings file where `hideFunSettings` is defined.

5. Review the `hideFunSettings` setting:

```ts
const hideFunSettings = new BooleanSetting('hideFunSettings', true).onInput(
    updateFunSettingsHiddenState
);
```

This shows that `hideFunSettings` is also enabled by default.

6. Check what `hideFunSettings` does. It only toggles a CSS class on the document body:

```ts
document.body.classList.toggle(
    settingsStyle.hideFunSettings,
    hideFunSettings.value
);
```

7. Go back to the Christmas countdown feature and review the original `reload()` function. Before the fix, the countdown only checked whether its own setting was enabled:

```ts
if (enabled.value) {
    [[, countdownSpanClone]] = marquee.add(countdownSpan);
    updateCountdown();
}
```

8. Observed result: `general.hideFunSettings` hides fun settings from the settings UI, but the Christmas countdown can still run because the feature logic does not check `hideFunSettings.value`.

9. Expected result: When `general.hideFunSettings` is enabled, fun features such as the Christmas countdown should not continue appearing to the user.

### Branch Link

Working branch in my fork:

```text
[Add your direct branch link here]
```

Example format:

```text
https://github.com/AdityaBoghara/better-moodle/tree/fix-hide-fun-settings
```

---

## Solution Approach

### Implementation Plan

I used an UMPIRE-style approach to understand, reproduce, plan, implement, review, and evaluate the fix.

#### U — Understand

I first reviewed the GitHub issue and identified the main problem: `general.hideFunSettings` hides fun settings from the settings page, but it does not deactivate the actual fun features. The issue specifically mentions `navbarMarquee.christmasCountdown.enabled` as an example of a fun feature that is enabled by default.

The root problem was that hiding the setting did not affect runtime behavior.

#### M — Map

I mapped the related code areas:

* `src/features/general/index.ts`

  * Contains the `hideFunSettings` setting.
  * Handles hiding fun settings in the UI by toggling a CSS class.

* `src/features/navbarMarquee/christmasCountdown.tsx`

  * Contains the Christmas countdown feature.
  * Defines the countdown setting as a fun setting using `.addTag('fun')`.
  * Adds the countdown to the navbar when `enabled.value` is true.

This helped me identify that the countdown feature was only checking its own enabled setting and not checking the global `hideFunSettings` value.

#### P — Plan

My plan was to reuse the existing `hideFunSettings` setting rather than creating a duplicate setting. To do this, I planned to:

1. Export the existing `hideFunSettings` setting from the general feature group.
2. Import that setting into the Christmas countdown feature.
3. Update the countdown activation condition so it only runs when:

   * the Christmas countdown is enabled, and
   * fun settings are not hidden.
4. Add an input listener so the countdown updates immediately when `hideFunSettings` changes.
5. Run linting, formatting, and build commands to verify the change.

#### I — Implement

I updated the general settings file by exporting the existing setting:

```ts
export const hideFunSettings = new BooleanSetting('hideFunSettings', true).onInput(
    updateFunSettingsHiddenState
);
```

Then I imported it into the Christmas countdown feature:

```ts
import { hideFunSettings } from '../general';
```

I updated the countdown condition from:

```ts
if (enabled.value) {
```

to:

```ts
if (enabled.value && !hideFunSettings.value) {
```

I also added an input listener:

```ts
hideFunSettings.onInput(reload);
```

This ensures that when the user changes `general.hideFunSettings`, the Christmas countdown reloads and respects the updated setting.

#### R — Review

After implementing the fix, I reviewed the change to make sure I was not creating a duplicate setting object. I reused the existing `hideFunSettings` setting by exporting and importing it.

I also checked that the fix was limited to the issue scope. The change updates the Christmas countdown behavior, which is the specific enabled-by-default fun feature mentioned in the issue.

I ran formatting and linting commands:

```bash
yarn run lint:fix
yarn run prettier:write
```

I resolved import sorting warnings in the changed file.

#### E — Evaluate

The expected behavior after the fix is:

* If `navbarMarquee.christmasCountdown.enabled` is true and `general.hideFunSettings` is false, the Christmas countdown can appear.
* If `general.hideFunSettings` is true, the Christmas countdown should not appear even if the countdown setting itself is enabled.
* If the user changes `hideFunSettings`, the countdown reloads immediately because of the new input listener.

This solution improves the user experience by making the Christmas countdown respect the global fun settings preference instead of only hiding the setting from the UI.

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
