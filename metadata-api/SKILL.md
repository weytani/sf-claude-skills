---
name: Salesforce Metadata API Reference
description: Dense platform reference for Salesforce Metadata API — package.xml structure, metadata types, source vs MDAPI format, deploy/retrieve commands, destructive changes, and common errors. Optimized for quick lookup.
when_to_use: When working with Salesforce metadata deployments, retrievals, package.xml files, destructive changes, or troubleshooting deployment errors.
version: "62.0"
---

# Salesforce Metadata API Reference (v62.0)

## package.xml Anatomy

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- XML declaration — required first line, UTF-8 encoding -->

<Package xmlns="http://soap.sforce.com/2006/04/metadata">
<!-- Root element. xmlns is mandatory and never changes. -->

    <types>
        <!-- Each <types> block groups members of a single metadata type -->
        <members>MyApexClass</members>
        <members>AnotherClass</members>
        <!-- Use <members>*</members> to include ALL components of this type (if wildcard supported) -->
        <name>ApexClass</name>
        <!-- <name> MUST appear once per <types> block, AFTER <members> -->
    </types>

    <types>
        <members>AccountTrigger</members>
        <name>ApexTrigger</name>
    </types>

    <types>
        <members>myLwcComponent</members>
        <name>LightningComponentBundle</name>
    </types>

    <types>
        <members>Account</members>
        <members>Contact</members>
        <name>CustomObject</name>
        <!-- Standard objects use API name (Account). Custom objects use MyObj__c -->
    </types>

    <types>
        <members>Account.My_Custom_Field__c</members>
        <members>Contact.Email_Opt_In__c</members>
        <name>CustomField</name>
        <!-- CustomField members use Object.FieldName format -->
    </types>

    <version>62.0</version>
    <!-- API version. Must match target org capabilities. -->

</Package>
```

**Key rules:**
- One `<name>` per `<types>` block, always after `<members>`.
- `<members>*</members>` retrieves all — but not all types support wildcard.
- `<version>` appears once, outside `<types>`, as the last child of `<Package>`.
- Child metadata (CustomField, ValidationRule, etc.) uses `ParentObject.ComponentName` format.

---

## Common Metadata Types

| Metadata Type | Directory (Source Format) | File Suffix | Wildcard (`*`) |
|---|---|---|---|
| ApexClass | `classes/` | `.cls` | Yes |
| ApexTrigger | `triggers/` | `.trigger` | Yes |
| ApexPage | `pages/` | `.page` | Yes |
| ApexComponent | `components/` | `.component` | Yes |
| LightningComponentBundle | `lwc/` | (directory) | Yes |
| AuraDefinitionBundle | `aura/` | (directory) | Yes |
| CustomObject | `objects/` | `.object-meta.xml` | Yes |
| CustomField | `objects/<Obj>/fields/` | `.field-meta.xml` | Yes |
| CustomTab | `tabs/` | `.tab-meta.xml` | Yes |
| Layout | `layouts/` | `.layout-meta.xml` | Yes |
| Profile | `profiles/` | `.profile-meta.xml` | No |
| PermissionSet | `permissionsets/` | `.permissionset-meta.xml` | Yes |
| FlowDefinition | `flowDefinitions/` | `.flowDefinition-meta.xml` | Yes |
| Flow | `flows/` | `.flow-meta.xml` | Yes |
| CustomMetadata | `customMetadata/` | `.md-meta.xml` | Yes |
| CustomLabel | `labels/` | `.labels-meta.xml` | Yes |
| StaticResource | `staticresources/` | `.resource-meta.xml` | Yes |
| RemoteSiteSetting | `remoteSiteSettings/` | `.remoteSite-meta.xml` | Yes |
| NamedCredential | `namedCredentials/` | `.namedCredential-meta.xml` | Yes |
| ConnectedApp | `connectedApps/` | `.connectedApp-meta.xml` | Yes |
| ApexTestSuite | `testSuites/` | `.testSuite-meta.xml` | Yes |
| AssignmentRules | `assignmentRules/` | `.assignmentRules-meta.xml` | No |
| AutoResponseRules | `autoResponseRules/` | `.autoResponseRules-meta.xml` | No |
| WorkflowRule | `workflows/` | `.workflow-meta.xml` | Yes |
| ValidationRule | `objects/<Obj>/validationRules/` | `.validationRule-meta.xml` | Yes |
| RecordType | `objects/<Obj>/recordTypes/` | `.recordType-meta.xml` | Yes |
| ListView | `objects/<Obj>/listViews/` | `.listView-meta.xml` | Yes |
| CompactLayout | `objects/<Obj>/compactLayouts/` | `.compactLayout-meta.xml` | Yes |
| WebLink | `objects/<Obj>/webLinks/` | `.webLink-meta.xml` | Yes |
| HomePageLayout | `homePageLayouts/` | `.homePageLayout-meta.xml` | Yes |

**Notes:**
- Child types (CustomField, ValidationRule, RecordType, ListView, CompactLayout, WebLink) use `ParentObject.ComponentName` in package.xml.
- Profile does NOT support wildcard — you must list each profile by name.
- AssignmentRules and AutoResponseRules do not support wildcard.

---

## Source Format vs MDAPI Format

| Aspect | Source Format | MDAPI Format |
|---|---|---|
| Directory structure | `force-app/main/default/` with decomposed subdirs | Flat `src/` or `unpackaged/` directory |
| Decomposition | Objects decomposed into fields, rules, etc. in subdirs | Single monolithic `.object` XML file per object |
| Tracking | Source tracking per org (scratch/sandbox) | No built-in source tracking |
| CLI commands | `sf project deploy start`, `sf project retrieve start` | `sf project deploy start --metadata-dir` |
| Config file | `sfdx-project.json` | `package.xml` at root of deploy dir |
| Use cases | Day-to-day development, scratch orgs, version control | CI/CD pipelines, unlocked/managed packages, org migrations |
| VCS friendliness | High — small decomposed files, clean diffs | Low — large monolithic files, noisy diffs |
| Convert between | `sf project convert mdapi` (MDAPI -> source) | `sf project convert source` (source -> MDAPI) |

---

## Deployment Commands

### `sf project deploy start`

| Flag | Description |
|---|---|
| `--source-dir`, `-d` | Path to source files to deploy (source format). Repeatable. |
| `--metadata`, `-m` | Metadata component names (e.g., `ApexClass:MyClass`). Repeatable. |
| `--manifest`, `-x` | Path to `package.xml` manifest file. |
| `--metadata-dir` | Path to MDAPI-format directory (root must contain `package.xml`). |
| `--target-org`, `-o` | Alias or username of the target org. |
| `--test-level`, `-l` | `NoTestRun`, `RunSpecifiedTests`, `RunLocalTests`, `RunAllTestsInOrg` |
| `--tests`, `-t` | Specific test classes to run (requires `--test-level RunSpecifiedTests`). Repeatable. |
| `--dry-run` | Validate only — does not deploy. Equivalent to `checkOnly` in MDAPI. |
| `--wait`, `-w` | Minutes to wait for deploy to finish (default: 33). `0` = async. |
| `--async` | Run deploy asynchronously. Returns job ID immediately. |
| `--ignore-errors` | Ignore deploy errors and allow partial success. |
| `--ignore-warnings` | Ignore deploy warnings. |

**Examples:**
```bash
# Deploy source directory
sf project deploy start -d force-app/main/default -o myOrg

# Deploy specific metadata
sf project deploy start -m ApexClass:MyClass -m ApexTrigger:MyTrigger -o myOrg

# Deploy via manifest with tests
sf project deploy start -x manifest/package.xml -o myOrg -l RunSpecifiedTests -t MyClassTest

# Validate only (dry run)
sf project deploy start -x manifest/package.xml -o myOrg --dry-run -l RunLocalTests

# Quick deploy after successful validation (uses job ID from dry run)
sf project deploy quick --job-id <jobId> -o myOrg
```

### `sf project deploy report`

```bash
# Check status of async deploy
sf project deploy report --job-id <jobId> -o myOrg
```

### `sf project deploy cancel`

```bash
# Cancel an in-progress deploy
sf project deploy cancel --job-id <jobId> -o myOrg
```

---

## Retrieval Commands

### `sf project retrieve start`

| Flag | Description |
|---|---|
| `--source-dir`, `-d` | Retrieve source into this directory path. Repeatable. |
| `--metadata`, `-m` | Metadata component names (e.g., `ApexClass:MyClass`). Repeatable. |
| `--manifest`, `-x` | Path to `package.xml` manifest. |
| `--target-org`, `-o` | Alias or username of the source org. |
| `--package-name`, `-n` | Name of installed package to retrieve. Repeatable. |
| `--output-dir`, `-r` | Directory to store retrieved MDAPI-format files. |
| `--api-version`, `-a` | Override API version for retrieval. |

**Examples:**
```bash
# Retrieve by manifest
sf project retrieve start -x manifest/package.xml -o myOrg

# Retrieve specific components
sf project retrieve start -m ApexClass:MyClass -m CustomObject:Account -o myOrg

# Retrieve into MDAPI format directory
sf project retrieve start -x manifest/package.xml -o myOrg -r mdapi-output/

# Retrieve installed package
sf project retrieve start -n "My Managed Package" -o myOrg
```

---

## Destructive Changes

### Pre-Destructive vs Post-Destructive

| Aspect | `destructiveChangesPre.xml` | `destructiveChangesPost.xml` |
|---|---|---|
| Execution order | Deletions happen **before** deploy | Deletions happen **after** deploy |
| Use when | Removing something that blocks the deploy (e.g., field referenced by deleted code) | Removing something that the deploy makes obsolete |
| Typical use | Remove fields/objects before deploying code that no longer references them | Clean up old components after new code is deployed |

### Requirements

1. An **empty `package.xml`** is required alongside destructive manifests (even if deploying nothing new).
2. Use `--metadata-dir` to deploy the directory containing both files.

### Full Example

**Directory structure:**
```
destructive-deploy/
  package.xml                    # Empty — required
  destructiveChangesPost.xml     # Components to delete after deploy
```

**`package.xml`** (empty — required even when only deleting):
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
    <version>62.0</version>
</Package>
```

**`destructiveChangesPost.xml`**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
    <types>
        <members>ObsoleteClass</members>
        <members>DeprecatedHelper</members>
        <name>ApexClass</name>
    </types>
    <types>
        <members>Account.Old_Field__c</members>
        <name>CustomField</name>
    </types>
    <types>
        <members>Old_Object__c</members>
        <name>CustomObject</name>
    </types>
    <version>62.0</version>
</Package>
```

**Deploy command:**
```bash
# Deploy destructive changes
sf project deploy start --metadata-dir destructive-deploy/ -o myOrg -w 30

# Validate first (recommended)
sf project deploy start --metadata-dir destructive-deploy/ -o myOrg --dry-run -w 30
```

**Pre-destructive example** (deletions run first):
```bash
# Directory contains: package.xml + destructiveChangesPre.xml
# Use when removing a field that existing code no longer references
# (deploy removes the code, but the field must go first to avoid dependency errors)
sf project deploy start --metadata-dir pre-destructive-deploy/ -o myOrg
```

---

## Common Deployment Errors

| Error | Cause | Fix |
|---|---|---|
| `Cannot delete metadata component` | Component is referenced by another component, or deletion not allowed for that type. | Remove references first. Use `destructiveChangesPre.xml` to sequence deletions. Check if the type supports deletion. |
| `This component has an error that prevents deployment` | Apex compilation error, LWC template error, or invalid XML in the component being deployed. | Fix syntax/compilation errors locally. Run `sf apex run test` to surface failures before deploying. |
| `Code coverage below 75%` | Org-wide Apex code coverage is under 75% (required for production deploys). | Write more tests. Use `--test-level RunLocalTests` to verify coverage. Check coverage with `sf apex get test -i <testRunId>`. |
| `Test failure` | One or more Apex test methods failed during the deployment validation. | Run tests locally first. Use `--tests` to isolate failures. Check test data setup and org-specific config. |
| `Dependent class is invalid` | A class referenced by the deploying code has a compilation error in the target org. | Fix or deploy the dependent class first. Deploy both classes together. |
| `Entity of type X named Y not found` | Referencing a metadata component (field, object, class) that doesn't exist in the target org. | Deploy the missing dependency first. Check API names for typos. Verify the component exists in the target. |
| `API version mismatch` | Component's API version exceeds the org's supported version, or package.xml version is incompatible. | Lower the `<version>` in package.xml or the component's `-meta.xml`. Check org's max API version in Setup > Apex Settings. |
| `Namespace prefix mismatch` | Deploying components with a namespace prefix to an org with a different or no namespace. | Remove namespace prefixes for unmanaged deployments. Ensure org namespace matches. |
| `DUPLICATE_VALUE` | Attempting to create a metadata component that already exists (e.g., duplicate field API name, duplicate record type). | Rename the new component or delete the existing one first. Use `retrieve` to check current state. |
| `UNKNOWN_EXCEPTION: An unexpected error occurred` | Salesforce platform error — transient or caused by corrupt metadata. | Retry the deployment. If persistent, open a Salesforce support case. Try deploying smaller subsets to isolate. |

---

## Quick Reference: Test Levels for Production Deploy

| Test Level | When to Use |
|---|---|
| `NoTestRun` | **Not allowed** for production. Sandbox/scratch only. |
| `RunSpecifiedTests` | Deploy with targeted tests. Must cover 75% of deployed code. |
| `RunLocalTests` | Runs all local tests (excludes managed package tests). Default for production. |
| `RunAllTestsInOrg` | Runs everything including managed package tests. Slowest — rarely needed. |

## Metadata API Limits (v62.0)

| Limit | Value |
|---|---|
| Max file size per component | 5 MB |
| Max total deploy/retrieve size (unzipped) | 400 MB |
| Max components per deploy | 10,000 |
| Deploy timeout (default) | 33 minutes (`--wait` overrides) |
| Max retrieve size (zipped) | 39 MB |
| Long-running deploy timeout | 24 hours (async) |
