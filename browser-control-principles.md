# Browser Control Principles

Browser control is governed by four decision categories: **Routing**, **Risk**, **Efficiency**, and **Failure Handling**.

## Routing

Use the user's real browser extension control channel when browser state, login sessions, or existing tabs matter.

Tool names may not match the actual browser in use. If a tool is named after one browser but the user's configured workflow routes through another real browser extension channel, follow the configured user environment instead of switching to a generic fallback browser.

Do not silently fall back to an in-app or isolated browser when the user explicitly expects control of their real browser environment.

## Risk

Before actions that may change account state, create costs, expose sensitive information, or affect external systems, confirm the necessary state.

Sensitive data should only be read or repeated after explicit user authorization. Examples include financial records, access secrets, personal account details, payment flows, deletion actions, submissions, file transfers, and messages sent to external recipients.

## Efficiency

Trusted state can be reused, but it expires when the site changes, the controlled tab changes, the page redirects unexpectedly, a login challenge appears, a CAPTCHA appears, a permission dialog appears, or the next action becomes high risk.

Low-risk actions should not require heavy preflight checks. When state appears to drift, perform the smallest useful confirmation, such as checking the control channel, target tab, URL/title, or target element.

## Failure Handling

Only run layered diagnosis after connection or control fails.

When failure occurs, report the layer where it appears to be blocked:

- extension connection
- browser state
- page loading
- login or anti-abuse flow
- element targeting
- permission boundary
