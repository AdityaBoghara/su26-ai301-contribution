# Contribution 1: general.hideFunSettings does not deactivate fun features

**Contribution Number:** 1

**Student:** Aditya Boghara

**Issue:** https://github.com/jxn-30/better-moodle/issues/876

**Status:** Phase III Complete

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
https://github.com/AdityaBoghara/better-moodle/tree/fix-hide-fun-settings
```

---

# Solution Approach

## Analysis

The issue is that `hideFunSettings` was only hiding fun-related settings in the UI after features had already been loaded. This meant that “fun” was not part of the core feature model, and fun features could still be instantiated even when the setting was enabled.

The root problem was that the source of truth lived at the setting/UI level instead of the feature-loading level.

## Proposed Solution

Move the “fun” classification into feature metadata. Allow features to declare tags directly through `Feature.register(...)`, expose a way to read those tags, and use that metadata in the loader to skip fun-tagged features when `settings.general.hideFunSettings` is enabled.

## Implementation Plan

Using the adapted **UMPIRE framework**:

### Understand

The goal is to prevent fun features from loading when `hideFunSettings` is enabled, instead of only hiding their settings in the modal.

### Match

The codebase already has a clean core structure with `Feature`, `FeatureGroup`, and `imports.ts` controlling registration and loading. There was already a concept of “fun” at the setting-tag level, so the new work follows that pattern while moving it to the feature level.

### Plan

1. Modify `src/_lib/Feature.tsx` so `Feature.register(...)` accepts tags.
2. Add feature-level tag accessors such as `tags` and `hasTag(...)`.
3. Update `src/_lib/imports.ts` so the loader checks `settings.general.hideFunSettings` before instantiating tagged features.
4. Tag known fun features such as `googlyEyes` and `christmasCountdown`.
5. Update `src/features/general/index.ts` so `hideFunSettings` is reload-based, since skipped features require a fresh load to return.

### Implement

Implemented locally in the core feature model, loader path, and tagged feature files.

### Review

Self-review checklist:

- [x] Follows the project’s feature/group architecture
- [x] Keeps feature metadata in the core model
- [x] Avoids coupling the loader directly to `general/index.ts`
- [x] Preserves existing settings behavior where possible
- [x] Makes reload behavior explicit for `hideFunSettings`

### Evaluate

Verify the solution by building and type-checking the project, then confirming in Moodle that fun features are hidden by default and reappear only after disabling **Hide fun settings** and reloading.

## Testing Strategy

### Unit Tests

1. **Feature registration accepts tags and stores them correctly**
2. **`hasTag("fun")` returns `true` only for tagged features**
3. **The loader skips fun-tagged features when `settings.general.hideFunSettings` is `true`**

### Integration Tests

1. **Tagged features are not instantiated when fun settings are hidden**
2. **Tagged features appear again after disabling Hide fun settings and reloading**

### Manual Testing

Manual runtime verification is still incomplete. We were able to implement the fix and type-check it locally, but we were not able to fully run and validate the userscript end-to-end in the target Moodle environment.

Expected manual testing flow:

1. Install the built userscript.
2. Log in to Moodle with a normal account.
3. Confirm that **Hide fun settings** is visible under General Settings.
4. Confirm that fun features are hidden by default.
5. Disable **Hide fun settings** and save the changes.
6. Reload the page.
7. Confirm that the fun features reappear.

## Implementation Notes

### Week [3] Progress

Implemented the core change by moving the “fun” classification into `Feature.register(...)` metadata. Added feature-level tag accessors and updated the loader so fun-tagged features can be skipped before instantiation. Tagged the known fun features and updated `hideFunSettings` to require a reload.

### Week [4] Progress

Follow-up work should focus on runtime verification in Moodle and adding automated coverage for feature metadata and loader behavior.

## Code Changes

### Files Modified

- `src/_lib/Feature.tsx`
- `src/_lib/imports.ts`
- `src/features/general/index.ts`
- `src/features/general/googlyEyes.tsx`
- `src/features/navbarMarquee/christmasCountdown.tsx`

### Key Commits

https://github.com/jxn-30/better-moodle/commit/c95aca9e1996b3969c41e0caa3b854d812f225cd

### Approach Decisions

Feature-level metadata was selected because it makes the feature itself the source of truth and allows the loader to decide early whether a feature should exist at all. This is cleaner than loading the feature and only hiding its settings in the UI.

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
