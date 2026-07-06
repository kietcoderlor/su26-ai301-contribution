# Cycle 2 - ExternalDNS Issue #5151

## Status

Cycle 2 Phase II Complete - Reproduce & Plan.

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

The expected behavior is that ExternalDNS preserves the full DNS name. The reported behavior suggests that part of the name is split or transformed incorrectly, especially when TXT ownership records are created with a TXT suffix.

---

## Why I Chose This Issue

I chose this issue because it is a real bug in a widely used Kubernetes project and is more technical than my first contribution. It gives me practice with Go, Kubernetes-style resources, DNS record generation, provider behavior, and test-driven debugging.

The issue includes a concrete example and expected behavior, which makes it possible to write a focused reproduction test instead of relying only on manual testing with Kubernetes and BIND.

---

# Phase II: Reproduce & Plan

## Environment Setup

I set up the project locally using WSL Ubuntu and Cursor.

**Environment:**

* OS: Windows + WSL Ubuntu
* Local repo path: `~/codepath/external-dns`
* Branch: `fix-5151-dotted-dnsname`
* Fork: https://github.com/kietcoderlor/external-dns
* Upstream: https://github.com/kubernetes-sigs/external-dns
* Editor: Cursor

**Setup completed:**

1. Forked `kubernetes-sigs/external-dns`.
2. Cloned my fork into WSL.
3. Added the upstream remote.
4. Created the working branch `fix-5151-dotted-dnsname`.
5. Opened the project in Cursor.
6. Inspected the DNSEndpoint, TXT registry, and RFC2136 provider code paths.
7. Added a targeted failing regression test in `registry/mapper/mapper_test.go`.

---

## Branch Link

https://github.com/kietcoderlor/external-dns/tree/fix-5151-dotted-dnsname

---

## Investigation Summary

I investigated the ExternalDNS code path for issue #5151 and found that the bug is unlikely to be in the `DNSEndpoint` CRD source. The CRD source appears to pass `dnsName` through unchanged.

The strongest root-cause candidate is the TXT registry name mapper:

```text
registry/mapper/mapper.go
```

Specifically, `AffixNameMapper.ToTXTName` uses `strings.SplitN(dns, ".", 2)`. This splits the DNS name at the first dot, which breaks names where the hostname portion itself contains dots, such as:

```text
name-192.168.0.1.example.com
```

---

## Steps to Reproduce

1. Open the mapper implementation:

```text
registry/mapper/mapper.go
```

2. Inspect `AffixNameMapper.ToTXTName`.

3. Observe that the function splits DNS names using:

```go
strings.SplitN(dns, ".", 2)
```

4. Add a regression test in:

```text
registry/mapper/mapper_test.go
```

5. Use this test input:

```text
DNS name: name-192.168.0.1.example.com
TXT suffix: -txtSuffix
Record type: A
```

6. Run the targeted test:

```bash
CGO_ENABLED=0 go test ./registry/mapper -run TestAffixNameMapper_ToTXTName -v
```

7. The new regression test fails on the current implementation.

---

## Reproduction Evidence

The failing test shows that the current implementation applies the TXT suffix too early.

**Expected:**

```text
name-192.168.0.1.example.com-txtsuffix
```

**Actual:**

```text
name-192-txtsuffix.168.0.1.example.com
```

This confirms that `AffixNameMapper.ToTXTName` currently splits the DNS name at the first dot and applies the TXT suffix to only the first label. This reproduces the malformed TXT ownership record behavior described in issue #5151.

---

## Relevant Files

### Primary files

* `registry/mapper/mapper.go`
* `registry/mapper/mapper_test.go`

### Related files

* `registry/txt/registry.go`
* `registry/txt/registry_test.go`
* `provider/rfc2136/rfc2136.go`
* `provider/rfc2136/rfc2136_test.go`
* `source/crd.go`
* `source/crd_test.go`
* `endpoint/endpoint.go`

---

## Solution Approach

### Understand

ExternalDNS creates malformed TXT ownership record names when a DNS name contains dots in the hostname portion. The current TXT mapper treats everything before the first dot as the host label and everything after it as the domain, which breaks names like:

```text
name-192.168.0.1.example.com
```

### Match

The existing mapper tests in `registry/mapper/mapper_test.go` already cover normal TXT prefix/suffix behavior. However, they do not cover dotted hostname cases. I added a failing regression test that follows the existing test style and captures this missing edge case.

### Plan

1. Keep the failing regression test in `registry/mapper/mapper_test.go`.
2. Update `AffixNameMapper.ToTXTName` in `registry/mapper/mapper.go` so TXT suffixes are applied without incorrectly splitting the DNS name at the first dot.
3. Check whether `AffixNameMapper.ToEndpointName` also needs to be updated for symmetric round-trip behavior.
4. Add or update TXT registry tests in `registry/txt/registry_test.go` if needed.
5. Run targeted tests:

   * `go test ./registry/mapper`
   * `go test ./registry/txt`
   * `go test ./provider/rfc2136`
6. Keep the PR focused on mapper / TXT registry behavior unless maintainers request provider-level changes.

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
* [x] Working branch created.
* [x] Code path investigated.
* [x] Likely root cause identified in `registry/mapper/mapper.go`.
* [x] Failing regression test added in `registry/mapper/mapper_test.go`.
* [x] Reproduction steps documented.
* [x] Implementation plan documented.
* [ ] Fix implementation will happen in Phase III.
