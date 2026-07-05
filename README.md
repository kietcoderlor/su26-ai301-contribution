## Cycle 2 — ExternalDNS Issue #5151

### Status

Cycle 2 Phase I Complete. Starting Phase II: Reproduce & Plan.

### Selected Issue

**Project:** ExternalDNS
**Repository:** https://github.com/kubernetes-sigs/external-dns
**Issue:** https://github.com/kubernetes-sigs/external-dns/issues/5151
**Fork:** https://github.com/kietcoderlor/external-dns
**Working Branch:** https://github.com/kietcoderlor/external-dns/tree/fix-5151-dotted-dnsname

### Problem Summary

I selected issue #5151, where ExternalDNS creates malformed DNS records when a `DNSEndpoint` `dnsName` contains dots in the hostname portion, such as `name-192.168.0.1.example.com`.

The expected behavior is that ExternalDNS preserves the full DNS name. The reported behavior suggests that part of the name is split or transformed incorrectly, especially when TXT ownership records are created with a TXT suffix.

### Why I Chose This Issue

I chose this issue because it is a real bug in a widely used Kubernetes project and is more technical than my first contribution. It gives me practice with Go, Kubernetes-style resources, DNS record generation, provider behavior, and test-driven debugging.

The issue includes a concrete example and expected behavior, which makes it possible to write a focused reproduction test instead of relying only on manual testing with Kubernetes and BIND.

---

## Cycle 2 — Phase II: Reproduce & Plan

### Investigation Summary

I investigated the ExternalDNS code path for issue #5151 and found that the bug is unlikely to be in the `DNSEndpoint` CRD source. The CRD source appears to pass `dnsName` through unchanged.

The most likely issue is in the TXT registry name mapper:

* `registry/mapper/mapper.go`
* `AffixNameMapper.ToTXTName`
* `AffixNameMapper.ToEndpointName`

The mapper currently splits DNS names using `strings.SplitN(dns, ".", 2)`. This works for normal hostnames, but it can break names where the hostname portion itself contains dots, such as `name-192.168.0.1.example.com`.

### Relevant Files

#### DNSEndpoint / CRD source

* `apis/v1alpha1/dnsendpoint.go`
* `source/crd.go`
* `source/crd_test.go`

#### Endpoint model and planning

* `endpoint/endpoint.go`
* `plan/plan.go`
* `controller/controller.go`

#### TXT registry and name mapping

* `registry/mapper/mapper.go`
* `registry/mapper/mapper_test.go`
* `registry/txt/registry.go`
* `registry/txt/registry_test.go`

#### RFC2136 provider

* `provider/rfc2136/rfc2136.go`
* `provider/rfc2136/rfc2136_test.go`

### Reproduction Plan

I plan to reproduce the bug with targeted unit tests instead of setting up a full Kubernetes + BIND environment first.

Planned test cases:

1. Add a mapper test for a dotted DNS name:

   * Input: `name-192.168.0.1.example.com`
   * TXT suffix: `-txtSuffix`
   * Expected TXT name: the suffix should apply to the full DNS name, not only the first segment before the first dot.

2. Add a TXT registry regression test:

   * Verify that TXT ownership records preserve dotted DNS names correctly.

3. Add or extend an RFC2136 provider test:

   * Verify that the DNS UPDATE message still contains the full FQDN.

4. Optionally add a CRD source test:

   * Confirm that `source/crd.go` passes the dotted `dnsName` through unchanged.

### Targeted Test Commands

Initial targeted tests:

```bash
CGO_ENABLED=0 go test ./registry/mapper -run 'ToTXTName|ToEndpointName' -v
CGO_ENABLED=0 go test ./registry/txt -run 'Suffix|GenerateTXT' -v
CGO_ENABLED=0 go test ./source -run TestCRDSource -v
CGO_ENABLED=0 go test ./provider/rfc2136 -run 'ApplyChanges|AddRecord' -v
CGO_ENABLED=0 go test ./endpoint -run 'NewEndpoint' -v
```

### Implementation Plan

1. Add a failing unit test in `registry/mapper/mapper_test.go` for dotted DNS names with TXT suffixes.
2. Confirm the current behavior incorrectly splits the name at the first dot.
3. Update `AffixNameMapper.ToTXTName` so suffixes are applied to the intended full DNS name behavior.
4. Update `AffixNameMapper.ToEndpointName` if needed to keep TXT name round-tripping symmetric.
5. Add or update TXT registry tests to cover the ownership record path.
6. Add an RFC2136 regression test if needed to confirm the provider keeps the full FQDN.
7. Run targeted tests before opening a PR.
8. Keep the PR focused on mapper/registry behavior unless maintainers request provider-level changes.

### Current Findings

* CRD source does not appear to modify `dnsName`.
* RFC2136 appears to build a full FQDN in the DNS UPDATE message.
* The strongest root-cause candidate is TXT name mapping in `registry/mapper/mapper.go`.
* Full BIND reproduction may not be necessary for the first fix if a focused unit test clearly reproduces the mapper behavior.

### Phase II Status

In progress. Repository is cloned in WSL, branch is created, and initial code investigation is complete.
