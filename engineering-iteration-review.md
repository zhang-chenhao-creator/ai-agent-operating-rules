# Engineering Iteration Review: Browser Control Rules for AI Agents

## 1. Starting Point

The original problem was practical: an AI agent needed to operate a real browser through an extension control channel. The goal was to make browser control efficient without causing confusion, accidental actions, or sensitive-data exposure.

The first tension was not technical implementation. It was rule design:

- The agent needs enough checks to avoid operating on the wrong page or account.
- Repeating full checks before every action slows down simple tasks.
- Sensitive operations need stronger boundaries than ordinary page navigation.
- Failures should be diagnosable without turning every normal action into a troubleshooting routine.

The real question became:

> How should browser-control rules handle conflicts between user preference, safety, efficiency, and failure recovery?

## 2. First Approach: Fixed Preflight Checks

The first model was a fixed preflight checklist:

```text
1. Check the extension connection.
2. Confirm the controlled tab.
3. Confirm the target URL and title.
4. Check authentication state.
5. Check for popups, CAPTCHAs, or permission prompts.
6. Confirm the target button or input.
7. Confirm the sensitive-data boundary.
```

This was complete, but too heavy as a default.

It reduced risk, but it also made low-risk actions unnecessarily slow. Opening a page or clicking a normal navigation item should not require a full diagnostic pass.

Engineering conclusion:

> A fixed preflight checklist is useful as a troubleshooting reference, but too rigid as the default execution model.

## 3. Second Approach: Skip Checks After Success

The next idea was to skip repeated checks once the browser control path had already worked.

This was directionally correct. The expensive parts of browser automation are often page loading, authentication state, rendering, anti-abuse flows, and external site behavior, not a few local checks.

But the simple version was unsafe:

- A working page on one site does not prove another site is controllable.
- A correct tab earlier does not prove the same tab is still the target.
- A page title may be correct before an unexpected redirect or popup.
- A low-risk action can be followed by a high-risk action.

Engineering conclusion:

> Successful control should create reusable trusted state, but that state needs expiration rules.

## 4. Third Approach: Trusted State and Risk Levels

The rule evolved into a stronger pattern:

```text
Reuse trusted state for low-risk actions.
Reconfirm necessary state for high-risk actions.
Diagnose only after failure.
```

This was a major improvement because it separated normal execution from risk handling.

Low-risk actions include ordinary navigation, normal clicks, and reading public page structure such as title or URL. High-risk actions include operations that may change account state, create costs, reveal sensitive data, or affect external systems.

However, listing examples became fragile. Downloading a file may be harmless in one context and sensitive in another. Submitting a form may be trivial or irreversible depending on the target.

Engineering conclusion:

> The rule should define risk by impact, not by an exhaustive list of actions.

## 5. Final Model: Four Decision Categories

The final model avoids a single linear priority list. Browser control is divided into four decision categories:

```text
Routing / Risk / Efficiency / Failure Handling
```

This is more accurate than saying "preference always comes first" or "safety always comes first."

Different categories answer different questions:

- **Routing** answers: which browser-control environment should be used?
- **Risk** answers: does this action need a gate before proceeding?
- **Efficiency** answers: can the agent reuse verified state?
- **Failure Handling** answers: what should happen when control breaks?

This avoids rule conflict. User preference matters most when selecting the browser route. Risk matters most before sensitive or externally impactful actions. Efficiency matters during low-risk continuation. Failure handling matters only after a control failure.

## 6. The Cache Invalidation Lesson

The most important refinement was adding expiration conditions for trusted state.

The earlier statement was:

```text
Trusted state can be reused.
```

That is incomplete. In engineering terms, this creates a cache without invalidation rules.

The final principle is:

```text
Trusted state can be reused, but it expires when the site changes, the controlled tab changes, the page redirects unexpectedly, a login challenge appears, a CAPTCHA appears, a permission dialog appears, or the next action becomes high risk.
```

This preserves speed while preventing stale assumptions from being reused in unsafe contexts.

## 7. Final Rule Reference

The final public rule set is:

```md
# Browser Control Principles

Browser control is governed by four decision categories: Routing, Risk, Efficiency, and Failure Handling.

- Routing: use the user's real browser extension control channel when browser state, login sessions, or existing tabs matter. For Windows users controlling Microsoft Edge through Codex, `@chrome` should route through the `chrome@openai-bundled` Node extension backend. Do not decide that browser control is unavailable only because the current tool list does not expose an explicit `chrome.*` tool; first initialize the extension backend and run a lightweight connection check such as `browser.user.openTabs()`. Do not silently fall back to an isolated browser when the user expects their real browser environment.
- Risk: before actions that may change account state, create costs, expose sensitive information, or affect external systems, confirm the necessary state. Sensitive data should only be read or repeated after explicit user authorization.
- Efficiency: trusted state can be reused, but it expires when the site changes, the controlled tab changes, the page redirects unexpectedly, a login challenge appears, a CAPTCHA appears, a permission dialog appears, or the next action becomes high risk. Low-risk actions should not require heavy preflight checks.
- Failure Handling: only run layered diagnosis after connection or control fails, and report whether the block appears to be plugin loading, backend initialization, Edge runtime state, extension response, Native Host, page loading, login or anti-abuse flow, element targeting, or permission boundary.
```

## 8. Human-Agent Co-Design

A useful part of this iteration was that the rule set was not designed upfront.

It emerged through human-agent co-design. The human side kept challenging whether the rules were too long, whether they would slow execution, whether priorities were being oversimplified, and whether the model would still work when state changed. The agent side kept turning those challenges into explicit tradeoffs, candidate rules, and shorter decision models.

This changed the work from "asking an agent to write a rule" into "building a control framework with an agent."

The important lesson is that agent collaboration can be used not only to execute tasks, but also to construct the operating rules for future tasks. The human provides judgment, taste, and risk sensitivity; the agent helps externalize, compress, and test the structure.

## 9. Transferable Engineering Lessons

This case is small, but the pattern transfers well to other AI agent workflows:

- Do not start by listing rules. Start by identifying conflicts.
- Avoid forcing every situation into a single priority ladder.
- Separate routing decisions, risk gates, efficiency strategy, and failure handling.
- Make low-risk paths fast and high-risk paths explicit.
- If state can be reused, define when it expires.
- Normal execution should stay simple; abnormal execution should be diagnosable.
- Rules should guide judgment, not create ritual.

## 10. Summary

The final result is not a large safety framework. It is a compact engineering model for one practical problem: how an AI agent should control a real browser without being slow, confused, or unsafe.

The main takeaway:

> Good operational rules do not hard-code every situation. They help an agent make consistent, low-cost, low-risk decisions in new situations.
