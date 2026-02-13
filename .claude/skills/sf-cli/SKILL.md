---
name: Salesforce CLI Platform Reference
description: Dense reference for sf CLI commands — auth, scratch orgs, deploy/retrieve, testing, data ops, and CI/CD patterns. Optimized for quick lookup.
when_to_use: When working with Salesforce DX projects, deploying metadata, running Apex tests, managing orgs, querying data, or setting up CI/CD pipelines with the Salesforce CLI.
version: "62.0"
---

# Salesforce CLI (`sf`) — Platform Reference

All commands use the modern `sf` CLI. Legacy `sfdx` aliases are deprecated.

---

## Project Structure — `sfdx-project.json`

```jsonc
{
  // Required. Defines where source lives and optional package info.
  "packageDirectories": [
    {
      "path": "force-app",          // Relative path to metadata source directory
      "default": true,              // One directory must be default
      "package": "MyPackage",       // Optional — unlocked/managed package name
      "versionName": "v1.0",        // Optional — human-readable version label
      "versionNumber": "1.0.0.NEXT" // Optional — MAJOR.MINOR.PATCH.BUILD
    }
  ],

  // Optional. Registered namespace prefix for the org/package.
  "namespace": "",

  // Optional. Login URL for auth. Use https://test.salesforce.com for sandboxes.
  "sfdcLoginUrl": "https://login.salesforce.com",

  // Optional. API version for source operations. Match your target org.
  "sourceApiVersion": "62.0"
}
```

| Field                | Required | Purpose                                                                 |
|----------------------|----------|-------------------------------------------------------------------------|
| `packageDirectories` | Yes      | Array of source directories. At least one must have `"default": true`.  |
| `namespace`          | No       | Namespace prefix. Empty string for unmanaged projects.                  |
| `sfdcLoginUrl`       | No       | Default login endpoint. Overridable per command with `--instance-url`.  |
| `sourceApiVersion`   | No       | Default API version for all source commands. Overridable with `--api-version`. |

---

## Auth Commands

| Command | Auth Method | Use Case | Key Flags |
|---------|-------------|----------|-----------|
| `sf org login web` | Browser OAuth | Interactive dev, first-time setup | `--alias`, `--instance-url`, `--set-default`, `--set-default-dev-hub` |
| `sf org login jwt` | JWT Bearer token | CI/CD, headless automation | `--username`, `--jwt-key-file`, `--client-id`, `--instance-url`, `--set-default`, `--alias` |
| `sf org login sfdx-url` | SFDX auth URL file | CI/CD, portable auth | `--sfdx-url-file`, `--alias`, `--set-default` |
| `sf org login device` | Device code flow | Headless envs without key files | `--alias`, `--instance-url`, `--set-default` |
| `sf org display` | — | Show current org auth info | `--target-org`, `--verbose` |
| `sf org list` | — | List all authenticated orgs | `--all`, `--clean`, `--skip-connection-status` |
| `sf org logout` | — | Remove stored auth for an org | `--target-org`, `--no-prompt`, `--all` |

---

## Scratch Org Commands

| Command | Purpose | Key Flags |
|---------|---------|-----------|
| `sf org create scratch` | Create a scratch org from a definition file | `--definition-file config/project-scratch-def.json`, `--alias`, `--duration-days` (1–30, default 7), `--set-default`, `--edition` (developer, enterprise, group, professional), `--target-dev-hub` |
| `sf org open` | Open org in browser | `--target-org`, `--path` (specific page), `--url-only` (print URL without opening) |
| `sf org list --all` | List all orgs (including expired scratch orgs) | `--all`, `--clean` (remove stale entries), `--skip-connection-status` |
| `sf org delete scratch` | Delete a scratch org | `--target-org`, `--no-prompt` |
| `sf config set target-org` | Set the default org for commands | `sf config set target-org=myAlias`, `--global` |

---

## Deploy / Retrieve Commands

Cross-reference the **metadata-api** skill for `package.xml` structure and wildcard details.

| Command | Purpose | Key Flags |
|---------|---------|-----------|
| `sf project deploy start` | Deploy source to an org | `--source-dir`, `--metadata`, `--manifest` (package.xml), `--target-org`, `--test-level`, `--tests`, `--dry-run`, `--wait`, `--async`, `--ignore-conflicts` |
| `sf project deploy report` | Check status of async deploy | `--job-id`, `--target-org`, `--wait` |
| `sf project deploy cancel` | Cancel an in-progress deploy | `--job-id`, `--target-org`, `--wait` |
| `sf project retrieve start` | Retrieve source from an org | `--source-dir`, `--metadata`, `--manifest`, `--target-org`, `--wait`, `--ignore-conflicts`, `--package-name` |
| `sf project retrieve report` | Check status of async retrieve | `--job-id`, `--target-org`, `--wait` |

---

## Testing Commands

| Command | Purpose | Key Flags |
|---------|---------|-----------|
| `sf apex run test --tests ClassName.methodName` | Run a specific test method | `--target-org`, `--wait`, `--result-format` |
| `sf apex run test --suite MySuite` | Run a test suite | `--target-org`, `--wait` |
| `sf apex run test --test-level RunLocalTests` | Run all local tests (excludes managed package tests) | `--target-org`, `--wait` |
| `sf apex run test --code-coverage --result-format human` | Run tests and display coverage | `--output-dir` (for junit/json output) |
| `sf apex run test --wait 10` | Wait N minutes for async test results | Combine with any of the above |

**Test Levels** (used with `--test-level` on deploy or test commands):

| Level | Behavior |
|-------|----------|
| `NoTestRun` | No tests. Only valid for non-production deploys. |
| `RunSpecifiedTests` | Run only tests named with `--tests`. Must cover deployed Apex. |
| `RunLocalTests` | All tests in the org except managed package tests. |
| `RunAllTestsInOrg` | Every test, including managed package tests. |

---

## Data Operations

| Command | Purpose | Key Flags |
|---------|---------|-----------|
| `sf data query --query "SELECT Id, Name FROM Account LIMIT 10"` | Inline SOQL query | `--target-org`, `--result-format` (human, csv, json), `--bulk` (use Bulk API), `--wait` |
| `sf data query --file query.soql` | Run SOQL from file | Same as above |
| `sf data export tree --query "SELECT Id, Name FROM Account" --plan` | Export records with relationship plan file | `--target-org`, `--output-dir`, `--plan` (generates plan JSON) |
| `sf data import tree --plan plan.json` | Import records from a plan file | `--target-org` |
| `sf data create record --sobject Account --values "Name='Test'"` | Create a single record | `--target-org`, `--use-tooling-api` |
| `sf data update record --sobject Account --record-id 001xx... --values "Name='Updated'"` | Update a record by ID | `--target-org`, `--where` (alternative to `--record-id`) |
| `sf data delete record --sobject Account --record-id 001xx...` | Delete a record by ID | `--target-org`, `--where` |

---

## Anonymous Apex

```bash
# From file
sf apex run --file script.apex --target-org myOrg

# From stdin
echo "System.debug('hi');" | sf apex run --target-org myOrg
```

---

## CI/CD Patterns

### JWT Auth (headless)

1. Create a **Connected App** in the target org (Enable OAuth, select `api` and `refresh_token` scopes, upload a digital certificate).
2. Generate or obtain `server.key` (private key matching the certificate).
3. Auth in CI:
   ```bash
   sf org login jwt \
     --username user@example.com \
     --jwt-key-file assets/server.key \
     --client-id <connected_app_consumer_key> \
     --instance-url https://login.salesforce.com \
     --set-default \
     --alias ci-org
   ```

### Dry-Run Deployment

Validate without persisting changes. Runs tests if `--test-level` is specified.

```bash
sf project deploy start \
  --source-dir force-app \
  --dry-run \
  --test-level RunLocalTests \
  --wait 30
```

### Package Version Creation

```bash
sf package version create \
  --package MyPackage \
  --installation-key-bypass \
  --wait 20 \
  --code-coverage \
  --definition-file config/project-scratch-def.json
```

---

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `No default org set` | No `target-org` configured and no `--target-org` flag passed | Run `sf config set target-org=<alias>` or pass `--target-org` explicitly |
| `The org cannot be found` | Alias/username not in local auth store, or org was deleted | Re-authenticate with `sf org login web --alias <alias>` or check `sf org list` |
| `INVALID_SESSION_ID` | Access token expired or revoked | Re-authenticate. For JWT: check certificate expiration and connected app permissions |
| `Deploy failed — test coverage below 75%` | Org-wide Apex coverage under 75% after deploy | Write more tests. Check coverage with `sf apex run test --code-coverage`. All triggers must have >0% coverage |
| `Shape/feature not enabled in scratch org` | Scratch org definition requests a feature not available or misspelled | Verify feature name in `project-scratch-def.json`. Check Salesforce docs for valid `features` and `settings` values |
| `Socket timeout` | Network interruption or long-running operation exceeded default timeout | Increase `--wait` value. Check network/proxy. For deploys, use `--async` and poll with `deploy report` |
| `ECONNRESET` | Connection dropped by Salesforce or intermediary proxy | Retry the command. If persistent, check VPN/proxy/firewall. Use `SF_LOG_LEVEL=debug` for diagnostics |
| `Metadata API request failed` | Malformed package.xml, invalid metadata type, or API version mismatch | Validate `package.xml` types against the metadata coverage report. Ensure `sourceApiVersion` matches org capability |
