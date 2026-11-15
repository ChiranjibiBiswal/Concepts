# Debugging Guide: Resolving CI Security Gate Failures (Trivy, SemVer & Dependency Drift)

> **Quick Summary:** A practical guide to identifying and resolving transitive dependency CVE alerts in CI/CD pipelines caused by package manager mismatches, semantic versioning constraints, and inconsistent dependency resolution.

---

## 1. Issue Overview

During the automated security scanning stage in CI, **Trivy** fails before container images are built with the following report:

```text
Target: package-lock.json
================================================================================
Library  | Vulnerability ID | Severity | Installed Version | Fixed Version
---------|------------------|----------|-------------------|----------------
adm-zip  | CVE-2026-12345   | HIGH     | 0.5.18            | 0.6.1
```

### The Conflict

- **Local environment:** The repository uses `pnpm`. Running `pnpm install` resolves `adm-zip@0.6.1` (patched).
- **Docker environment:** Docker builds use `pnpm install --frozen-lockfile` and complete successfully.
- **CI environment:** The security scan detects `adm-zip@0.5.18` inside an automatically generated `package-lock.json`.

This creates a situation where local development and Docker builds pass, but the CI security gate fails.

---

## 2. Root Cause Analysis

The issue originates from two factors:

1. Transitive dependency version constraints.
2. Package manager differences across environments.

### 2.1 Dependency Chain Analysis

The vulnerable dependency originates from the following dependency chain:

```text
sample-onboarding-service
  └── cassandra-driver@4.10.0
        └── adm-zip: "~0.5.10"
```

The `cassandra-driver` package declares `adm-zip` using the following version constraint:

```json
{
  "adm-zip": "~0.5.10"
}
```

This restricts dependency resolution to the `0.5.x` version range.

### 2.2 SemVer Range Limits (~ vs ^)

Under Semantic Versioning:

#### Tilde (~)

```text
~0.5.10
```

Allows:

```text
>= 0.5.10 AND < 0.6.0
```

#### Caret (^)

```text
^0.5.10
```

Allows:

```text
>= 0.5.10 AND < 0.6.0
```

For versions below `1.0.0`, the caret operator also respects the leftmost non-zero version component.

Consequently, neither `~0.5.10` nor `^0.5.10` allows automatic resolution to `0.6.1`.

The security patch was introduced in `adm-zip@0.6.1`, which falls outside the declared dependency range.

Therefore, npm cannot automatically select the patched version without an explicit override or an updated parent dependency.

---

### 2.3 Environment & Package Manager Mismatch

| Environment | Package Manager | Lockfile Used | Resolved Version |
| --- | --- | --- | --- |
| Local Development | pnpm | pnpm-lock.yaml | 0.6.1 |
| Docker Build | pnpm | pnpm-lock.yaml | 0.6.1 |
| CI Security Scan | npm | Generated package-lock.json | 0.5.18 |

#### What Happens in CI?

1. **Repository Checkout:** Git checks out the repository containing `package.json` and `pnpm-lock.yaml`. No `package-lock.json` is committed.

2. **Default CI Installation:** The shared pipeline executes `npm install` because the project is configured as a Node.js application.

3. **Independent Dependency Resolution:** npm ignores `pnpm-lock.yaml` and resolves dependencies using `package.json` and the dependency constraints.

4. **Lockfile Generation:** npm generates a new, untracked `package-lock.json` containing `adm-zip@0.5.18`.

5. **Trivy Execution:** Trivy scans the generated `package-lock.json`, identifies the vulnerable version, and fails the security gate before Docker packaging begins.

**Important:** npm generates the temporary `package-lock.json` during installation. Trivy subsequently scans that file; Trivy is not responsible for generating it. The real issue is the mismatch between the repository's intended package manager and the package manager actually used in CI.

---

### 2.4 Dependency Tree Verification

To identify the source of transitive dependencies and verify their resolved versions, inspect the complete dependency tree using pnpm.

**Command:**

```bash
pnpm list -r adm-zip --depth Infinity
```

This helps identify:

- Which direct dependency introduces a transitive dependency.
- Whether multiple packages share the same transitive dependency.
- Which versions are actually resolved by pnpm.
- Whether dependency overrides are being applied.

#### Example Findings (Anonymized)

| Dependency Chain           | Resolved Version |
| -------------------------- | ---------------- |
| cassandra-driver → adm-zip | 0.6.1            |

The dependency tree confirms that pnpm resolves the patched `adm-zip` version.

#### Lockfile Verification

The resolved versions can also be checked directly inside `pnpm-lock.yaml`.

**PowerShell command:**

```powershell
Select-String -Path pnpm-lock.yaml -Pattern "adm-zip@"
```

Example output:

```text
adm-zip@0.6.1
```

This provides additional evidence that the expected dependency versions are recorded in the pnpm lockfile.

**Key Observation:**

The dependency tree and lockfile confirm that pnpm resolves the patched versions locally. However, CI independently resolves dependencies using npm, resulting in a different dependency tree.

---

## 3. Solutions

### 3.1 Immediate Fix: Package Manager Overrides

An immediate solution is to configure an explicit package manager override in `package.json`.

> **Important:** An override can fix the detected vulnerable version in the dependency tree, but it does not guarantee compatibility. It is a mitigation, not proof that the parent package and the overridden dependency work correctly together.

**Example:**

```json
{
  "name": "sample-onboarding-service",
  "dependencies": {
    "cassandra-driver": "^4.8.0"
  },
  "overrides": {
    "adm-zip": "0.6.1"
  },
  "pnpm": {
    "overrides": {
      "glob": "^14.0.2",
      "minimatch": "^11.0.1",
      "adm-zip": "0.6.1"
    }
  }
}
```

**How it works:**

- `overrides` is npm's configuration for forcing dependency versions across the dependency tree.
- `pnpm.overrides` is pnpm's configuration for controlling dependency resolution.
- Both configurations can coexist in the same `package.json`.
- npm uses the top-level `overrides` configuration.
- pnpm uses the `pnpm.overrides` configuration.

Since the pnpm lockfile already resolves `adm-zip@0.6.1`, adding it to `pnpm.overrides` is unnecessary unless explicit enforcement is required.

**Important Compatibility Consideration:**

The override forces `adm-zip@0.6.1` even though `cassandra-driver` declares `~0.5.10`.

An override bypasses the original version constraint but does not guarantee compatibility. In real projects, this can fix the vulnerability signal while introducing runtime or integration failures if the parent package depends on older behavior from `adm-zip`.

Therefore, verify that the parent package works correctly with the overridden version through appropriate integration testing.

#### Verification

After applying the npm override, execute:

```bash
npm install
```

Check the resolved dependency:

```bash
npm ls adm-zip
```

Expected output:

```text
sample-onboarding-service
└─┬ cassandra-driver@4.10.0
  └── adm-zip@0.6.1 overridden
```

You can also inspect why the dependency exists:

```bash
npm explain adm-zip
```

This helps confirm the dependency's origin and whether the override is applied.

---

### 3.2 Long-Term Fix: Standardize CI Pipeline Tooling

The permanent solution is to eliminate package manager differences across development, CI, and Docker environments.

Recommended changes:

#### 1. Detect the Package Manager

Configure the shared CI pipeline to identify the package manager using the repository's lockfile.

| Lockfile | Package Manager | Installation Command |
| --- | --- | --- |
| pnpm-lock.yaml | pnpm | pnpm install --frozen-lockfile |
| package-lock.json | npm | npm ci |
| yarn.lock | Yarn | yarn install --immutable (modern Yarn) |

#### 2. Use Deterministic Installation

Avoid running generic `npm install` for repositories that use pnpm.

Use the appropriate installation command based on the package manager.

#### 3. Align Security Scanning

Ensure Trivy scans the dependency lockfile associated with the actual package manager used to build the application.

#### 4. Scan the Final Container Image

Additionally, scan the built container image to identify vulnerabilities in the actual deployed runtime artifacts, including operating system packages and application dependencies.

Lockfile scanning and image scanning complement each other; neither completely replaces the other.

#### 5. Prevent Unexpected Lockfile Generation

Add a CI validation step to detect unexpected lockfiles and prevent accidental dependency resolution using a different package manager.

Recommended guardrails:

- Fail CI if both `pnpm-lock.yaml` and `package-lock.json` exist unexpectedly.
- Fail CI if the lockfile does not match the package manager declared by the repository.
- Validate that the install command and lockfile are aligned before running Trivy or building the image.

---

## 4. Technical Discussion & Q&A

### Q1: Why did local builds pass while CI failed?

**Answer:**

Different package managers were used in different environments.

- Local and Docker builds used pnpm with `pnpm-lock.yaml`, which resolved `adm-zip@0.6.1`.
- CI executed `npm install`, which ignored `pnpm-lock.yaml` and generated a new `package-lock.json`.
- npm resolved `adm-zip@0.5.18` because of the `~0.5.10` constraint.

As a result, Trivy detected the vulnerable version in CI.

### Q2: Why didn't npm automatically install version 0.6.1?

**Answer:**

Because `cassandra-driver` declares `adm-zip` using:

```json
"adm-zip": "~0.5.10"
```

This restricts npm to versions below `0.6.0`.

Since `0.6.1` falls outside the declared range, npm cannot select it automatically without an override or an updated parent dependency.

### Q3: How do dependency overrides compare across package managers?

| Package Manager | Configuration Key | Purpose |
| --- | --- | --- |
| npm (8.3+) | `overrides` | Forces dependency versions across the dependency tree. |
| Yarn | `resolutions` | Forces dependency versions globally or by dependency path. |
| pnpm | `pnpm.overrides` | Controls dependency resolution across the dependency tree. |

Although the purpose is similar, the configuration syntax and resolution behavior differ between package managers.

### Q4: Should package-lock.json be committed alongside pnpm-lock.yaml?

**Answer:**

Generally, no.

Maintaining multiple lockfiles for different package managers introduces additional maintenance overhead and the possibility of dependency drift.

When developers update only the primary lockfile, secondary lockfiles can become outdated and produce inconsistent dependency resolutions.

A repository should generally maintain one authoritative lockfile matching the package manager used by the team and CI pipeline.

### Q5: Does a successful local dependency installation guarantee that CI will pass?

**Answer:**

No.

A successful local installation only confirms that dependencies were resolved successfully in that particular environment.

If CI uses a different package manager, lockfile, or installation command, it can resolve different dependency versions.

Therefore, package manager consistency is essential for reproducible builds and reliable security scanning.

---

## 5. Troubleshooting Commands

Useful commands for investigating dependency resolution and security issues.

| Purpose | Command |
| --- | --- |
| Inspect pnpm dependency tree | `pnpm list -r adm-zip --depth Infinity` |
| Find why a dependency exists | `pnpm why adm-zip` |
| Inspect installed dependency | `pnpm list adm-zip` |
| Check npm dependency tree | `npm ls adm-zip` |
| Find npm dependency origin | `npm explain adm-zip` |
| Verify pnpm lockfile | `pnpm install --frozen-lockfile` |
| Verify npm lockfile | `npm ci` |
| Search pnpm lockfile (PowerShell) | `Select-String -Path pnpm-lock.yaml -Pattern "adm-zip@` |

**Note:** `npm ci` requires an existing, synchronized `package-lock.json` or supported npm shrinkwrap file. It does not generate a missing lockfile. If the repository is pnpm-based, using `npm ci` against a pnpm project is a configuration mismatch and should be treated as such.

---

## 6. Key Takeaways

1. **Deterministic Installs:** CI pipelines should use deterministic installation commands such as `npm ci` or `pnpm install --frozen-lockfile`.

2. **Transitive Dependency Constraints:** Dependencies using tilde (`~`) or caret (`^`) may prevent automatic upgrades across version boundaries.

3. **Package Manager Consistency:** Local development, CI, and Docker builds should use the same package manager and authoritative lockfile.

4. **Dependency Tree Analysis:** Commands such as `pnpm why`, `pnpm list`, and `npm explain` help identify the origin and resolved versions of transitive dependencies.

5. **Overrides:** Package manager overrides can resolve security issues when the parent dependency has not yet updated its version constraint, but compatibility must be tested.

6. **Security Scanning:** Scan the actual dependency lockfile used during installation and the final container image to get better visibility into application and runtime vulnerabilities.

7. **Environment Parity:** A successful local build does not guarantee a successful CI build if dependency resolution differs between environments.

---

## 7. Final Debugging Approach

The recommended troubleshooting sequence for similar CI security gate failures:

```text
        CI Security Gate Failure
                  |
                  v
          Identify CVE Details
                  |
                  v
       Identify Vulnerable Package
                  |
                  v
       Trace Dependency Origin
                  |
                  v
      Inspect Local Dependency Tree
                  |
                  v
       Verify Lockfile Versions
                  |
                  v
      Compare CI and Local Tooling
                  |
                  v
       Identify Resolution Mismatch
                  |
                  v
       Apply Override / Upgrade
                  |
                  v
       Verify Dependency Compatibility
                  |
                  v
       Re-run Security Scan
                  |
                  v
          Validate CI Build
```

**Final Lesson:** Before changing dependencies to fix a security vulnerability, first identify which package manager, lockfile, and install command CI is actually using. Sometimes the vulnerability is not caused by the dependency tree used by the application, but by a different dependency tree generated during the CI process.

**In one sentence:** A CI security gate fails because Trivy is scanning a dependency tree that was generated by the wrong package manager, not because the repository's intended pnpm lockfile was wrong.
