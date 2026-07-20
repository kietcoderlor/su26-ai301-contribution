# Cycle 2 — ExternalDNS Issue #5151

## Status

Cycle 2 Phase II Complete — Reproduce & Plan.

## Selected Issue

**Project:** ExternalDNS
**Repository:** https://github.com/kubernetes-sigs/external-dns
**Issue:** https://github.com/kubernetes-sigs/external-dns/issues/5151
**Fork:** https://github.com/kietcoderlor/external-dns
**Working Branch:** https://github.com/kietcoderlor/external-dns/tree/fix-5151-dotted-dnsname

---

## Problem Summary

I selected issue #5151, where ExternalDNS creates malformed DNS records when a `DNSEndpoint` `dnsName` contains dots in the hostname portion, such as:

```text
name-192.168.0.1.example.com
```

The expected behavior is that ExternalDNS preserves the full dotted DNS name. The reported behavior suggests that the name is split incorrectly when TXT ownership records are generated with a TXT suffix.

---

# Phase II: Reproduce & Plan

## Environment Setup

I set up the project locally using **Windows + WSL Ubuntu + Cursor**.

**Environment:**

* OS: Windows + WSL Ubuntu
* Local repo path: `~/codepath/external-dns`
* Branch: `fix-5151-dotted-dnsname`
* Fork: https://github.com/kietcoderlor/external-dns
* Upstream: https://github.com/kubernetes-sigs/external-dns
* Editor: Cursor

**Setup path used:**

I followed the normal open-source setup path for a Go repository: fork the upstream project, clone the fork in WSL, add the upstream remote, create a working branch, inspect the relevant packages, and run targeted Go tests.

**Setup steps completed:**

1. Forked `kubernetes-sigs/external-dns`.
2. Cloned my fork into WSL at `~/codepath/external-dns`.
3. Added the upstream remote.
4. Created the working branch `fix-5151-dotted-dnsname`.
5. Opened the repository in Cursor using the WSL environment.
6. Inspected the DNSEndpoint, TXT registry, mapper, and RFC2136 provider code paths.
7. Added a targeted failing regression test in `registry/mapper/mapper_test.go`.

**Setup challenges and how I resolved them:**

1. **Windows vs. Linux commands:**
   I initially tried to run Linux commands such as `sudo apt update` inside Windows PowerShell, which failed because `sudo` is only available inside WSL. I resolved this by entering Ubuntu/WSL first, then running the Linux setup commands there.

2. **Clone/network issue:**
   The first clone attempt failed with an RPC / early EOF error. I resolved this by removing the incomplete clone and cloning again from inside WSL.

3. **GitHub push authentication:**
   Pushing with a GitHub password failed because GitHub no longer supports password authentication for Git operations. I resolved this by using GitHub authentication through `gh auth login` / Git credential setup, then pushed the branch to my fork.

4. **Go toolchain:**
   The repository uses Go modules and requires a newer Go toolchain than the default Ubuntu package may provide. I checked `go.mod`, confirmed the required Go version, and used targeted tests instead of running the entire test suite immediately.

---

## Branch Link

https://github.com/kietcoderlor/external-dns/tree/fix-5151-dotted-dnsname

---

## Reproduction Process

### Steps to Reproduce

1. Open the mapper implementation:

```text
registry/mapper/mapper.go
```

2. Inspect `AffixNameMapper.ToTXTName`.

3. Notice that the function splits DNS names using:

```go
strings.SplitN(dns, ".", 2)
```

4. Open the mapper test file:

```text
registry/mapper/mapper_test.go
```

5. Add a regression test case to `TestAffixNameMapper_ToTXTName` with this input:

```text
DNS name: name-192.168.0.1.example.com
TXT suffix: -txtSuffix
Record type: A
```

6. Run the targeted test:

```bash
CGO_ENABLED=0 go test ./registry/mapper -run TestAffixNameMapper_ToTXTName -v
```

7. Observe that the new regression test fails on the current implementation.

---

## Expected Behavior

When a DNS name contains dots in the hostname portion, the TXT ownership mapper should preserve the full DNS name and apply the TXT suffix to the intended full name.

For this input:

```text
DNS name: name-192.168.0.1.example.com
TXT suffix: -txtSuffix
```

The expected TXT record name is:

```text
name-192.168.0.1.example.com-txtsuffix
```

This means the dotted DNS name is preserved before the TXT suffix is applied.

---

## Actual Behavior

The current implementation splits the DNS name at the first dot and applies the TXT suffix after only the first label.

The failing test produced:

```diff
Expected:
name-192.168.0.1.example.com-txtsuffix

Actual:
name-192-txtsuffix.168.0.1.example.com
```

This confirms that `AffixNameMapper.ToTXTName` applies the suffix too early. Instead of preserving `name-192.168.0.1.example.com`, it treats `name-192` as the first part and `168.0.1.example.com` as the remaining domain.

This reproduces the malformed TXT ownership record behavior described in ExternalDNS issue #5151.

---

## Reproduction Evidence

I added a targeted failing regression test in:

```text
registry/mapper/mapper_test.go
```

The failing test case is named:

```text
suffix with dotted hostname (#5151)
```

The test command was:

```bash
CGO_ENABLED=0 go test ./registry/mapper -run TestAffixNameMapper_ToTXTName -v
```

The failure shows the exact mismatch between expected and actual TXT record name:

```diff
-name-192.168.0.1.example.com-txtsuffix
+name-192-txtsuffix.168.0.1.example.com
```

This gives a repeatable, source-level reproduction without needing a full Kubernetes + BIND environment.

---

## Specific Files and Functions Involved

### Primary root-cause candidate

* `registry/mapper/mapper.go`

  * `AffixNameMapper.ToTXTName`
  * `AffixNameMapper.ToEndpointName`

### Test file

* `registry/mapper/mapper_test.go`

  * `TestAffixNameMapper_ToTXTName`

### Related files investigated

* `registry/txt/registry.go`
* `registry/txt/registry_test.go`
* `provider/rfc2136/rfc2136.go`
* `provider/rfc2136/rfc2136_test.go`
* `source/crd.go`
* `source/crd_test.go`
* `endpoint/endpoint.go`

---

## Investigation Depth

To avoid guessing, I inspected the code path from the reported resource to DNS record generation:

1. **DNSEndpoint CRD source:**
   `source/crd.go` appears to pass `dnsName` through unchanged, so the CRD source is unlikely to be the root cause.

2. **Endpoint model:**
   `endpoint/endpoint.go` stores the full `DNSName` and does not appear to truncate the dotted name during endpoint creation.

3. **TXT registry mapper:**
   `registry/mapper/mapper.go` is the strongest root-cause candidate because `ToTXTName` uses `strings.SplitN(dns, ".", 2)`, which explains why a dotted hostname gets split after `name-192`.

4. **RFC2136 provider:**
   `provider/rfc2136/rfc2136.go` is relevant to the reported provider path, but the current strongest reproduction is in the TXT mapper. RFC2136 should be tested later as a regression guard to confirm full FQDN behavior.

This investigation found a concrete "match" example in the codebase: the existing mapper tests already cover TXT prefix/suffix behavior, but they do not cover dotted hostname cases. I added the dotted hostname case in the same test style.

---

## Solution Approach

### Understand

ExternalDNS creates malformed TXT ownership record names when a DNS name contains dots in the hostname portion. The current TXT mapper treats everything before the first dot as the host label and everything after it as the domain. This breaks names like:

```text
name-192.168.0.1.example.com
```

### Match

The relevant existing pattern is in:

```text
registry/mapper/mapper_test.go
```

Existing tests already cover normal TXT prefix and suffix behavior. My reproduction adds a missing edge case to the same test table: a DNS name with a dotted hostname portion.

### Plan

1. Keep the failing regression test in `registry/mapper/mapper_test.go`.
2. Update `AffixNameMapper.ToTXTName` in `registry/mapper/mapper.go` so TXT suffixes are applied without incorrectly splitting the DNS name at the first dot.
3. Check whether `AffixNameMapper.ToEndpointName` also needs to be updated for symmetric round-trip behavior.
4. Add or update TXT registry tests in `registry/txt/registry_test.go` if needed.
5. Optionally add an RFC2136 provider regression test in `provider/rfc2136/rfc2136_test.go` to confirm the full FQDN is preserved.
6. Run targeted tests:

   * `go test ./registry/mapper`
   * `go test ./registry/txt`
   * `go test ./provider/rfc2136`
7. Keep the PR focused on mapper / TXT registry behavior unless maintainers request provider-level changes.

### Implement

Implementation will happen in Phase III on this branch:

https://github.com/kietcoderlor/external-dns/tree/fix-5151-dotted-dnsname

### Review

Before opening a PR, I will review:

* ExternalDNS contribution guidelines.
* Existing mapper and registry test patterns.
* The final diff to ensure no unrelated provider, CRD, or Kubernetes behavior is changed.

### Evaluate

The fix will be considered successful when:

* The new dotted DNS name regression test passes.
* Existing mapper tests still pass.
* TXT registry tests pass.
* RFC2136 targeted tests pass or remain unaffected.
* Existing behavior for normal DNS names remains unchanged.

---

## Phase II Status

Phase II Complete.

* [x] Repository set up in WSL.
* [x] Working branch created and linked.
* [x] Code path investigated.
* [x] Root-cause hypothesis identified in `registry/mapper/mapper.go`.
* [x] Failing regression test added in `registry/mapper/mapper_test.go`.
* [x] Expected vs. actual behavior documented clearly.
* [x] Reproduction steps documented.
* [x] Implementation plan documented.
* [ ] Fix implementation will happen in Phase III.
