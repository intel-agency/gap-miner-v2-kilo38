# Validation

All changes must be validated.

- Changes should be validated as they are implemented.
- All changes must be validated before committing.

## Steps

The following steps must be run as part of validation:

- build
- scan
- test

A validation script must be maintained to run these steps automatically (i.e. `validation.sh`, `validation.ps1`, etc.).

- It should mirror exactly what is run in the CI/CD pipeline.
- Update the local and CI/CD copies to keep them in sync with any changes.

## Missing Validation Script

If an agent needs to run validation and the expected script (e.g. `validation.ps1`, `validation.sh`) does not exist:

1. **Create the script** before proceeding with any validation. Write it at the repository root with the platform-appropriate extension (`.ps1` for Windows, `.sh` for Unix).
2. **Implement the three steps** — `build`, `scan`, `test` — in the order listed. Each step must fail fast (non-zero exit) on error so the script stops immediately.
3. **Make it executable** (`chmod +x validation.sh` on Unix; on Windows ensure the execution policy allows it).
4. **Commit the script** as its own change before running it, so CI/CD picks it up on the same branch.
5. **Mirror CI/CD** — inspect any existing pipeline configuration (e.g. `.github/workflows/`, `azure-pipelines.yml`) and ensure the script commands match what CI runs. If no CI config exists, choose sensible defaults for the project's language/framework and document the choices in a comment at the top of the script.

## Testing

An automated test suite must be maintained.

- Test results and coverage reports should be generated automatically.
- Test Coverage levels must be maintained as new code is added.
- Test coverage level must be > 85% at all times.

### Test Driven Development (TDD)

When implementing new features, TDD should be used.

- Implement failing tests to cover the required functionality.
- Implement changes to make the tests pass.
- Iterate creating tests and implementing changes to make them pass until the required functionality is implemented.
