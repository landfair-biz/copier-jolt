# AI Development Rules

## 1. Understand the Existing Project First

- Treat the existing repository as the source of truth.
- Before making changes, inspect the repository structure, README, package/configuration files, build scripts, and existing architecture.
- Identify the project's language, framework, runtime, package manager, build system, test framework, and development workflow before modifying anything.
- Do not assume the project uses React, TypeScript, Tailwind, Vite, Next.js, Node.js, Python, or any other specific technology unless the repository already does.
- Preserve the existing architecture and conventions whenever reasonably possible.

## 2. Make Minimal, Targeted Changes

- Modify only what is necessary to implement the requested feature or fix.
- Do not rewrite, migrate, or modernize unrelated parts of the application.
- Do not replace the project's framework, build system, package manager, UI library, or architecture unless explicitly requested.
- Do not create duplicate implementations when an existing utility, component, service, or abstraction can be reused.
- Before creating a new file, check whether an existing file should be extended instead.

## 3. Preserve Existing Functionality

- Existing behavior should continue to work unless the requested change intentionally modifies it.
- Do not remove features, routes, commands, configuration, integrations, or dependencies without a clear reason.
- Be especially careful with authentication, authorization, persistence, APIs, database logic, networking, configuration, and deployment code.
- Never replace working functionality with mock data, placeholder implementations, or simplified examples unless explicitly requested.

## 4. Follow Existing Conventions

- Match the project's existing naming, formatting, file organization, patterns, and coding style.
- Use the project's existing dependency manager and scripts.
- Prefer existing dependencies and utilities over adding new ones.
- If a library already exists in the project for a particular task, use it instead of introducing another library.
- Follow existing error-handling, logging, validation, testing, and configuration patterns.

## 5. Dependencies

- Keep dependencies minimal.
- Do not add a dependency if the functionality can reasonably be implemented using the existing stack or standard library.
- Before adding a dependency, inspect the existing dependency configuration.
- Use the project's existing package manager and lockfile.
- Never change package managers without explicit instruction.
- Do not upgrade unrelated dependencies merely because newer versions are available.

## 6. Configuration and Environment

- Do not hardcode secrets, API keys, passwords, tokens, credentials, or private endpoints.
- Preserve existing environment-variable conventions.
- Do not modify `.env`, credentials, deployment secrets, or other sensitive configuration unless explicitly requested.
- When configuration is required, update the appropriate example/template configuration when one exists.
- Never commit secrets or generated credentials.

## 7. Backend and API Changes

- Reuse existing API patterns and services whenever possible.
- Preserve existing API contracts unless the requested change requires a breaking change.
- Validate user-controlled input at appropriate boundaries.
- Handle errors explicitly and consistently with the existing application.
- Do not silently swallow exceptions or return misleading success responses.
- Do not expose internal errors, secrets, credentials, or sensitive implementation details to clients.

## 8. Frontend Changes

- If the project has a frontend, use the project's existing frontend framework and component system.
- Reuse existing components and styling patterns before creating new ones.
- Do not introduce a new CSS framework, component library, icon library, or state-management library unless necessary.
- Maintain responsive behavior and accessibility consistent with the existing application.
- Do not redesign unrelated portions of the UI when implementing a specific feature.

## 9. Database and Data

- Inspect the existing database schema, migrations, models, and data-access patterns before making database changes.
- Prefer migrations over destructive schema changes.
- Never delete or reset production data as part of a development task.
- Preserve backwards compatibility when reasonably possible.
- Do not invent database fields, tables, or relationships without understanding the existing data model.

## 10. Testing and Verification

- Inspect existing tests before modifying behavior.
- Add or update tests when the project already has a testing framework and the change warrants it.
- Run the project's existing validation, linting, type-checking, and test commands when practical.
- Fix errors introduced by your changes before considering the task complete.
- Do not disable tests, lint rules, type checking, or security checks merely to make a build pass.
- If a test cannot be run, clearly identify what could not be verified.

## 11. Git and Repository Hygiene

- Do not delete or overwrite user work.
- Do not reset, rebase, force-push, or rewrite Git history unless explicitly requested.
- Do not modify `.gitignore` unless necessary.
- Do not commit generated build artifacts unless the repository already tracks them.
- Keep changes focused and reviewable.

## 12. Security

- Treat security-sensitive code conservatively.
- Never weaken authentication, authorization, encryption, validation, sandboxing, firewall rules, or access controls to make functionality work.
- Do not disable security protections as a workaround.
- Follow the project's existing security model.
- Call out security implications when a requested change affects them.

## 13. Documentation

- Update documentation when a change affects installation, configuration, commands, APIs, user-facing behavior, or developer workflows.
- Prefer updating existing documentation over creating duplicate documentation.
- Keep documentation consistent with the actual implementation.

## 14. When Requirements Are Ambiguous

- Do not make large architectural assumptions.
- Infer intent from the existing codebase before asking questions.
- If multiple approaches are possible, prefer the one that best matches the existing architecture.
- Ask for clarification when proceeding would require a significant architectural, security, data, or compatibility decision.

## 15. General Rule

- Existing code is presumed intentional unless there is evidence otherwise.
- Preserve before replacing.
- Reuse before creating.
- Fix before rewriting.
- Verify before declaring the task complete.