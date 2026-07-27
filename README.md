# Week 8 Update — ExternalDNS Pivot Decision

## Status

Cycle 2 ExternalDNS issue paused. I am switching to a backup issue after Phase III investigation and mentor feedback.

## Issue

**Project:** ExternalDNS  
**Issue:** https://github.com/kubernetes-sigs/external-dns/issues/5151  
**Fork:** https://github.com/kietcoderlor/external-dns  
**Branch:** https://github.com/kietcoderlor/external-dns/tree/fix-5151-dotted-dnsname  

---

## What I Worked On This Week

This week, I reviewed the Phase III blocker for ExternalDNS issue #5151 and asked for mentor/peer feedback in CodePath Slack.

In Phase II / early Phase III, I added a local failing regression test for a dotted DNS name case involving `--txt-suffix`.

The reported current behavior creates a TXT record like:

```text
name-192-txtSuffix.168.0.1.example.com
```

The issue reporter expected a TXT record like:

```text
name-192.168.0.1.example.com-txtSuffix
```

My regression test was based on the reporter's expected behavior. However, after reviewing the existing mapper implementation, tests, and documentation, I found that the current ExternalDNS behavior appears to apply `--txt-suffix` to the first DNS label / host portion rather than the full FQDN.

---

## Blocker / Scope Finding

The main blocker is that the reporter's expected behavior may conflict with existing `--txt-suffix` semantics in ExternalDNS.

Changing `AffixNameMapper.ToTXTName` to append the suffix to the full FQDN would likely require a broader design decision from maintainers and could break existing mapper tests and documented behavior.

Because of this, I paused production code changes instead of forcing a fix that might not match the project's intended behavior.

---

## Mentor / Peer Feedback

I asked for feedback in CodePath Slack. The feedback was that this issue may not be a good candidate to complete because:

1. The reporter's expected behavior appears to conflict with current mapper semantics.
2. The behavior may involve DNS semantics beyond a small bug fix.
3. The maintainer discussion suggests that this issue may no longer be intended for outside contribution.

Based on this feedback, I decided to pivot away from ExternalDNS #5151 and switch to a backup issue.

---

## Artifacts

| Item | Link |
|---|---|
| ExternalDNS issue | https://github.com/kubernetes-sigs/external-dns/issues/5151 |
| Working branch | https://github.com/kietcoderlor/external-dns/tree/fix-5151-dotted-dnsname |
| Reproduction test commit | https://github.com/kietcoderlor/external-dns/tree/fix-5151-dotted-dnsname |
| Contribution README | https://github.com/kietcoderlor/su26-ai301-contribution |

---

## What I Learned

- How to trace a reported open-source bug into the relevant code path.
- How to write a targeted regression test for a suspected behavior mismatch.
- How to compare a reporter's expected behavior against existing tests and documentation.
- How to recognize when an issue requires maintainer/product clarification instead of a quick code change.
- How to make a responsible open-source decision to pivot when an issue is no longer a good fit.

---

## Next Steps

1. Stop active work on ExternalDNS #5151.
2. Select a backup issue for the remaining weeks.
3. Start the new issue from Phase I / Phase II.
4. Focus on an issue with clearer scope, clearer expected behavior, and a realistic path to a PR.

## Week 8 Status

Week 8 progress submitted.

- [x] ExternalDNS investigation documented.
- [x] Blocker explained clearly.
- [x] Mentor / peer feedback incorporated.
- [x] Pivot decision documented.
- [ ] Backup issue selected.
- [ ] New issue Phase I / Phase II started.
