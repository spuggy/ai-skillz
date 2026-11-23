---
name: gherkin-spec
description: Turn a loose feature request into a Gherkin .feature spec in specs/<area>/ that says WHAT the system does, never HOW, and finish with the decisions the user must confirm. Use when the user describes a feature to spec, asks for a spec or .feature file, or says "spec this". Writes the spec only, never step code or the implementation.
---

# Gherkin spec writer

Turns a loose requirement ("let me fix an account's name after adding it") into an executable specification: a Gherkin `.feature` file that says WHAT the system does, never HOW. The spec is the contract a human approves and an agent implements against. This skill writes the spec only. It never writes step code or the implementation.

## 1. Learn the domain before writing

If there is a codebase, read enough of it to use its real language and spot the risks:
- AGENTS.md, CLAUDE.md, PLAN.md, README and any docs on security, privacy or permissions
- the models and choices the feature touches (for example the account types, transaction statuses and category sources)
- existing specs in `specs/` and existing tests, so the wording matches
- anything the feature does that the codebase has never done before (a first outbound HTTP call, a new import format, a new public endpoint, the first data sent to an LLM). Call these out, because they are where the risk is.

Use the product's words (account, statement, transaction, category, rule), not the code's (amount_pence, dedupe_key, AccountCreate).

## 2. Work out the rules

From the request, list:
- **who** acts (the user, an agent acting for them, an outside system such as a bank export)
- **the capability**, in one sentence, and why it matters to the user
- **the rules**, grouped as:
  1. the main path
  2. missing, empty or odd input
  3. failure of anything outside the system (files malformed, a service down, slow, returning junk)
  4. behaviour over time (what is stored, what happens when things change later, re-imports)
  5. safety: untrusted input, outbound calls, injection, private addresses, and **what may be sent to a model or third party** (check the project's rules)
  6. permissions: who may and may not do it
  7. boundaries: exact limits (£40.00 vs £40.01, ±3 days vs 4 days)
- **what is out of scope**: say so explicitly.

## 3. Write the feature file

Structure:
```gherkin
Feature: <capability>
  <2-3 lines: what and why, from the user's point of view>

  Out of scope: <things deliberately excluded>

  Background:
    Given <shared setup, named people and things>

  Rule: <a rule in plain words>

    Scenario: <one behaviour>
      Given <context>
      When <one action>
      Then <an observable outcome>
```

Writing rules:
- **Domain language only.** No URLs of the app itself, CSS selectors, button ids, HTTP status codes, table, class or function names. Ask: would this line still be true if the app were rewritten with a different framework? If not, rewrite it.
- **One When per scenario.** If you need two, it is two scenarios.
- **Then lines are observable outcomes** (what a person sees or can do), with concrete values ("'Joint current' has a balance of £812.34", not "shows the correct balance").
- **Use concrete, named examples**: people (Alex), accounts, payees and amounts. Use `example.org` / "Example Bank" style names for anything outside, and never real bank data.
- **Describe the outside world as Givens** ("Given the statement opens at £1,000.00", "Given the model is not responding", "Given Example Card Co exports purchases as positive") so the tests can fake it and never touch the real network or real files.
- **Use Scenario Outline + Examples** for families of cases and boundaries.
- **Tag `@ui` sparingly**: only for the one or two scenarios that must prove the user interface works. Everything else should be runnable through the API.
- Keep each scenario short: 3-6 lines.

Bad (a UI script, coupled to the implementation):
```gherkin
When I click "#edit-account" and type "Joint current" into "input[name=name]"
Then the API returns 200 and .account-name shows "Joint current"
```
Good:
```gherkin
When Alex renames "Joint curent" to "Joint current"
Then the accounts list shows "Joint current"
```

## 4. Report the decisions, don't hide them

After the feature file, always give a short list headed **"Decisions I made that you should confirm"**. These are every rule you guessed rather than were told: snapshot or live data, refused vs "not found", case sensitivity, what happens on failure, timing, privacy. Also list any **open questions** that would add or change scenarios. Never present a guess as settled. The human's answers are the actual spec-writing.

## 5. Self-check before handing over

- [ ] Every Then line has a concrete, checkable value
- [ ] No implementation detail leaks into any line
- [ ] Main path, failures, safety, permissions and boundaries are each covered or listed as out of scope
- [ ] Anything outside the system is set up with a Given, so it can be faked
- [ ] Each scenario would fail today if the feature didn't exist
- [ ] The decisions list is included

## Output

- Save to `specs/<area>/<feature_name>.feature` (for example `specs/accounts/editing_an_account.feature`), or wherever the user names.
- Do not write step functions, actions or implementation code. Once the human approves the spec, the implementing agent does that.
- If the user asks for changes, edit the feature file and update the decisions list to match.
