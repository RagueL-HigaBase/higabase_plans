# SonarQube Quality Workflow

Last reviewed: 2026-10-01

This document defines the canonical workflow for reviewing and remediating SonarQube findings in `higa_systems_react`.

## Purpose

SonarQube is an additional quality-control layer. It helps identify reliability, maintainability, security, accessibility, duplication, and code-quality concerns.

SonarQube findings are inputs to engineering review, not automatic instructions to rewrite the application.

The existing application architecture, business behavior, Phoenix template conventions, established UI/UX, localization patterns, and project documentation remain authoritative unless an intentional project decision changes them.

## Core rule

Never change code only to make SonarQube green.

Every finding must first be understood in the context of the actual Higa implementation. A remediation is accepted only when it improves the code without unintentionally changing established behavior, architecture, styling, or template conventions.

## Working model

SonarQube remediation is handled as a controlled engineering workflow.

For each finding:

1. Identify the exact SonarQube issue, rule, severity, file, and affected code.
2. Read the current repository version of the affected file.
3. Inspect enough surrounding and related code to understand the implementation rather than treating the reported line in isolation.
4. Determine whether the finding is:
   - a real defect or quality problem;
   - a safe semantic or structural improvement;
   - generated/vendor/template code that requires special handling;
   - an intentional implementation;
   - a false positive or a finding whose proposed remediation would conflict with project rules.
5. Choose the smallest correct fix.
6. Preserve existing business behavior and UI behavior unless the defect itself requires a deliberate behavior change.
7. Preserve Phoenix conventions and existing project architecture. Do not redesign or refactor unrelated code as part of a Sonar fix.
8. Run the project's normal verification after an appropriate batch of changes.
9. Run SonarQube analysis again and confirm the relevant finding is resolved without introducing new problems.
10. Continue with the next finding only after the previous change is understood and verified.

## Change boundaries

A SonarQube fix should normally be minimal and local.

Allowed when justified:
- correcting invalid HTML/CSS/TypeScript;
- improving accessibility semantics;
- removing genuine duplicate or contradictory declarations;
- replacing unsafe or unreliable constructs with equivalent safe constructs;
- simplifying code when behavior remains unchanged;
- correcting an actual security issue;
- adding narrowly scoped configuration or exclusions when technically justified.

Not allowed merely to satisfy SonarQube:
- changing business rules;
- redesigning screens or navigation;
- changing API contracts;
- changing persisted data models;
- replacing established Phoenix patterns without a separate architectural decision;
- broad refactoring unrelated to the finding;
- hiding valid findings with exclusions, `aria-hidden`, presentation roles, comments, or rule suppression when the underlying issue can be correctly fixed;
- modifying generated/vendor/template assets mechanically without first determining their ownership and regeneration path.

If a finding requires crossing one of these boundaries, stop treating it as a routine Sonar remediation. It becomes a separate engineering decision and must be reviewed in the relevant canonical project documentation before implementation.

## Phoenix and template code

The project intentionally uses Phoenix template structures and styling conventions. SonarQube must not be used as a reason to progressively replace the template with unrelated custom patterns.

When a finding occurs inside Phoenix-derived code:

1. determine whether the code is project-owned or effectively vendor/generated template code;
2. determine whether the reported construct is actually invalid or unsafe;
3. prefer a standards-compliant fix that preserves Phoenix classes, appearance, and behavior;
4. avoid large edits to bundled theme assets when a source-level or configuration-level solution is more appropriate;
5. document exclusions or accepted exceptions when they are necessary and intentional.

## Accessibility findings

Accessibility findings are treated as functional quality findings, not cosmetic warnings.

Prefer correct semantic HTML and ARIA usage over suppressing the rule. Existing Phoenix classes may remain unchanged when semantic elements can be corrected without affecting visual behavior.

For example, a data table should use appropriate header semantics rather than being marked as presentation-only simply to silence an accessibility rule.

## Security findings

Security findings receive explicit review even when SonarQube reports low severity.

A finding must not be dismissed solely because the current Quality Gate passes. Determine the actual exposure, data flow, trust boundary, and runtime behavior.

Community Build limitations must also be considered: a passing SonarQube analysis does not mean that the application has received complete security analysis.

Secrets and SonarQube tokens must never be committed to the repository or documentation.

## Generated, vendor, and large template assets

Large generated or vendor-derived files can produce many findings that do not have the same ownership characteristics as application source code.

Before editing such a file, determine:
- where it originates;
- whether it is generated;
- whether a source file exists elsewhere;
- whether an upstream/template update would overwrite the change;
- whether the file should be analyzed at all.

Exclusions are acceptable only when there is a documented technical reason. They must not be used simply to reduce the issue count.

## False positives and accepted findings

A SonarQube issue may be marked false-positive or accepted only after the implementation has been inspected and there is a concrete technical reason not to change it.

For any significant exception, record:
- the Sonar rule or category;
- the affected area;
- why remediation is inappropriate;
- why the current implementation is safe or intentional;
- whether the decision should be revisited later.

Do not create documentation entries for trivial resolved findings. Documentation is for durable decisions, not for duplicating SonarQube's issue history.

## Verification

Sonar remediation does not replace the project's existing verification.

After an appropriate change or small batch:

1. review the diff for unintended changes;
2. run the normal frontend verification/build used by the repository;
3. run any relevant tests or checks for the affected area;
4. perform a focused UI/runtime check when the change can affect rendering or interaction;
5. rerun SonarQube analysis;
6. confirm that the intended finding disappeared or changed as expected;
7. inspect whether the remediation introduced new findings.

A green Quality Gate alone is not sufficient evidence that a change is correct.

## Repository workflow

The repository version is the working source for remote review and remediation. Before relying on it, local work intended to be included must be committed and pushed.

When remediation is performed directly against the repository:
- inspect the current `main` version before editing;
- change only the files required for the reviewed finding;
- use focused commit messages;
- do not combine unrelated cleanup with Sonar remediation;
- synchronize the local checkout with the resulting repository commit before continuing local development.

Uncommitted local changes are outside this remote workflow and must not be assumed to exist in the repository.

## Documentation policy

Do not turn this file into a line-by-line Sonar issue log.

Update this document only when the remediation process itself changes or when a durable exception/policy is introduced, such as:
- a justified analysis exclusion;
- a rule intentionally disabled or customized;
- a recurring false-positive pattern;
- a generated-code policy;
- a Quality Gate or Quality Profile decision;
- a remediation that requires an architectural exception.

Routine fixes belong in Git history and SonarQube history.

## Definition of done for a Sonar remediation

A finding is considered remediated when:

- the reported problem has been understood rather than mechanically suppressed;
- the smallest appropriate correction has been made;
- established Higa architecture and Phoenix behavior remain intact unless an intentional change was separately approved;
- relevant build/tests/runtime checks pass;
- a subsequent SonarQube analysis confirms the expected result;
- no secret, scanner cache, token, or local Sonar artifact has been committed;
- any durable exception or configuration decision has been documented.

## Local Sonar artifacts

Local scanner output is not application source and must remain outside version control.

The repository `.gitignore` should continue to exclude local scanner artifacts such as:

```text
.scannerwork/
.sonar_lock
```

SonarQube authentication tokens are local secrets and must never be stored in tracked files.