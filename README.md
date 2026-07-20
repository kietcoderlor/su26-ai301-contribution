# Cycle 2 — Phase III: Build Progress

## Status

Cycle 2 Phase III In Progress — implementation paused pending maintainer clarification.

## Issue

**Project:** ExternalDNS
**Issue:** https://github.com/kubernetes-sigs/external-dns/issues/5151
**Fork:** https://github.com/kietcoderlor/external-dns
**Branch:** https://github.com/kietcoderlor/external-dns/tree/fix-5151-dotted-dnsname

---

## Implementation Progress

In Phase II, I added a targeted failing regression test for a dotted DNS name with a TXT suffix.

Test input:

```text
DNS name: name-192.168.0.1.example.com
TXT suffix: -txtSuffix
Record type: A
```

The test expected:

```text
name-192.168.0.1.example.com-txtsuffix
```

The current mapper produced:

```text
a-name-192-txtsuffix.168.0.1.example.com
```

This confirmed that the current TXT mapper applies the suffix after the first DNS label instead of applying it to the full dotted name.

---

## Current Blocker

During Phase III, I reviewed the existing mapper tests and documentation before changing production code. I found that my Phase II expected behavior may conflict with the current `--txt-suffix` semantics in ExternalDNS.

Current behavior appears to treat `--txt-suffix` as applying to the host portion / first DNS label. Existing tests seem to encode this behavior. Changing `AffixNameMapper.ToTXTName` to append the suffix to the full FQDN may break existing mapper tests and documented behavior.

Because of this, I paused implementation instead of changing `registry/mapper/mapper.go` immediately.

---

## Code Changes So Far

| Item                             | Link                                                                                  |
| -------------------------------- | -------------------------------------------------------------------------             |
| Branch                           | https://github.com/kietcoderlor/external-dns/tree/fix-5151-dotted-dnsname             |
| Issue                            | https://github.com/kubernetes-sigs/external-dns/issues/5151                           |
| Reproduction test commit         | (https://github.com/kietcoderlor/external-dns/tree/fix-5151-dotted-dnsname)           |
| Maintainer clarification comment | (https://github.com/kubernetes-sigs/external-dns/issues/5151#issuecomment-5018310108) |

Files involved:

* `registry/mapper/mapper.go`
* `registry/mapper/mapper_test.go`

No production code fix has been committed yet because the intended `--txt-suffix` behavior needs maintainer clarification.

---

## Testing Strategy

I ran the targeted mapper test:

```bash
CGO_ENABLED=0 go test ./registry/mapper -run TestAffixNameMapper_ToTXTName -v
```

Result:

* Existing mapper suffix/prefix cases pass.
* The new dotted hostname regression test fails.
* The failure demonstrates a behavior mismatch, but the intended expected behavior needs maintainer confirmation.

I also checked mapper round-trip behavior and found that existing `TestToEndpointNameNewTXT` cases pass. This suggests the current prefix/suffix mapper behavior is internally consistent, which is why changing it without maintainer guidance could be risky.

---

## Maintainer Clarification Needed

I am asking maintainers to clarify which behavior is intended when `--txt-suffix` is used with a `dnsName` containing dotted hostname segments:

1. Keep the current first-label suffix behavior.
2. Append the suffix to the full FQDN.
3. Apply the suffix to the zone-relative name, which may require zone/domain context in the mapper.

Once maintainers clarify the expected behavior, I will update the test and implementation plan accordingly.

---

## Challenges Faced

1. The original Phase II plan assumed that the suffix should be appended to the full DNS name.
2. Phase III investigation showed that this assumption may conflict with existing ExternalDNS mapper tests and documentation.
3. The issue may require a product/design clarification before a safe code change can be made.
4. I paused implementation to avoid introducing a breaking change.

---

## Next Steps

1. Post or track the maintainer clarification comment on issue #5151.
2. Wait for maintainer guidance on the intended `--txt-suffix` semantics.
3. Adjust the failing test expectation if needed.
4. Implement the smallest safe fix once the intended behavior is confirmed.
5. Run targeted mapper and TXT registry tests.
6. Continue toward a Phase IV PR if the maintainers confirm the desired behavior.

---

## Phase III Progress Checklist

* [x] Failing regression test added.
* [x] Existing mapper tests reviewed.
* [x] Potential semantics conflict identified.
* [x] Implementation paused to avoid breaking existing behavior.
* [x] Maintainer clarification comment drafted.
* [ ] Maintainer clarification received.
* [ ] Production fix implemented.
* [ ] Targeted tests passing.
* [ ] PR opened.
