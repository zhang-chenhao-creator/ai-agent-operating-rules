# AI Agent Browser Control Rule Design

This repository is a small engineering case study about designing browser-control rules for an AI agent operating through a real user browser extension channel.

It is not a browser automation tutorial. The focus is the decision model behind safe and efficient agent behavior when browser state, user preferences, sensitive information, and failure handling can conflict.

## Scope

The concrete environment behind this case is:

- Windows users
- Codex as the AI agent runtime
- Microsoft Edge as the user's real browser
- `chrome@openai-bundled` as the Codex browser-extension control channel

In this setup, `@chrome` should be treated as the Codex extension route for controlling Edge, not as an instruction to switch to Google Chrome or an isolated in-app browser.

## Core Model

The final rule set is organized around four decision categories:

- **Routing**: choose the correct browser-control channel and respect the user's actual browser environment.
- **Risk**: add gates before actions that may expose sensitive data, change account state, create costs, or affect external systems.
- **Efficiency**: reuse trusted state when it is still valid, and avoid heavy preflight checks for low-risk actions.
- **Failure Handling**: diagnose only when control fails, and report the layer where the failure occurred.

## Files

- [`browser-control-principles.md`](browser-control-principles.md): the concise public rule reference.
- [`engineering-iteration-review.md`](engineering-iteration-review.md): the engineering review of how the rule evolved from fixed preflight checks into a scenario-based decision model.

## Why This Matters

AI agents that control a real browser are not only executing commands. They are operating inside a live user environment with logged-in sessions, changing pages, possible popups, sensitive account data, and external side effects.

A good rule set should not force a long checklist before every click. It should help the agent decide when to move fast, when to stop, when to ask for confirmation, and how to recover when the control path breaks.

## Origin

This case study started as a spontaneous idea during an ordinary class session: if an AI agent is going to control a real browser, its behavior should be governed by clear operating rules rather than ad hoc prompts.

The useful part was not the idea itself, but turning it into a concrete artifact in the same session: a concise rule reference, an iteration review, and a reusable decision model.

The key lesson:

> Good operational rules do not hard-code every situation. They help an agent make consistent, low-cost, low-risk decisions in new situations.
