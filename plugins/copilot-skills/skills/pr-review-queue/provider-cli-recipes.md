# Focused provider CLI recipes

Load the relevant provider section when binding a CLI route. These are candidates,
not cached authorization: validate installed help/metadata, credentials and scope.
Resolve placeholders and serialize write payloads to files. Keep
`direct_http: false`; reading CLI source/docs does not authorize SDK/raw HTTP.

## Check tools without installing them

Disable implicit Azure extension installation in the probe process:

```powershell
$env:AZURE_EXTENSION_USE_DYNAMIC_INSTALL = 'no'
```

Check `gh --version`, `az version` and `az extension show --name azure-devops`
before extension commands, including `--help`. Install only after explicit setup
approval with `az extension add --name azure-devops`; leave global defaults and
credentials unchanged.

## GitHub

Resolve the poster with `gh api --hostname <host> user`, the target separately
with `gh api --hostname <host> users/<target>`, and repository ID/access from
`repos/<owner>/<repo>`. Retain stable IDs and provider actor type.

For complete paginated arrays, choose one supported form:

- `gh api --paginate --slurp <endpoint>`: outer array of pages; flatten locally.
- `gh api --paginate --jq <filter> <endpoint>`: per-page output, not one JSON array.

CLI 2.74.2 rejects `--slurp` combined with `--jq`/`--template`. Preserve identity
and history fields on every page. Read reviews, issue timeline and current
requested reviewers separately. Reviewer removal accepts logins, **not a
request-cycle condition**; it cannot replace the queue's race safeguards.

## Azure DevOps identity and repository

Use `az repos show --repository <name-or-id> --organization <org-url>
--project <project>`, not `--id`. Carry exact org URL/project ID/repo ID on calls.

Focused identity candidates:

```text
az repos pr list --organization <org-url> --project <project-id> --repository <repo-id> --creator <confirmed-target-upn> --status all --top 1
az repos pr list --organization <org-url> --project <project-id> --repository <repo-id> --creator me --status all --top 1
```

Extension 1.0.8 resolves `me` through `ConnectionData.authenticated_user.id` and
non-GUID UPNs through the identity service. Verify installed behavior and retain
each result's `createdBy.id`; an empty result proves no identity. Use another
proven route or report the gap, never search unconfigured repositories.

A documented MCP `createdByMe` filter plus a full PR read can independently
identify its poster. Do not assume MCP/CLI actors match; all write routes must
resolve the same approved actor.

Validate these `az devops invoke` resources for identity normalization:

| Resource | Route parameters | Required evidence |
| --- | --- | --- |
| `graph / users` | `userDescriptor=<provider-returned-descriptor>` | subject kind, origin and member/service classification |
| `graph / storageKeys` | `subjectDescriptor=<same-descriptor>` | storage-key GUID matching the `IdentityRef.id` |
| `core / projectCollections` | scoped list, then `collectionId=<returned-id>` | immutable hosting collection ID and its kind |

Do not decode descriptors as storage keys or confuse collection IDs with
Accounts, Entra tenant or identity-domain IDs. Retain the actual identifier kind.
A permission denial cannot be retried through another interface to bypass access.

## Azure DevOps invocation and history

Supply **both** area/resource, scoped route values, API version and GET for probes:

```text
az devops invoke --organization <org-url> --area git --resource pullRequestThreads --route-parameters project=<project-id> repositoryId=<repo-id> pullRequestId=<pr-id> --api-version 7.1 --http-method GET
```

Avoid argument-free discovery, which enumerates unrelated services. Extension
1.0.8 can fail with `NoneType.lower` for an area without resource; its parser
accepts `7.1-preview` but rejects `7.1-preview.1`. Verify installed metadata and
supported versions, not guessed names or blanket version rewriting.

Useful Git resources: `pullRequests`, `pullRequestIterations`,
`pullRequestThreads`, `items`, `pullRequestStatuses`. Preserve actual continuation
metadata and select CI for the pinned iteration. For permission evidence, resolve
the Git Repositories security namespace and exact repository token, then use
`az devops security permission show`; never cast a probe vote.

Raw `pullRequestThreads` retains properties/identity dictionaries compact MCP
projections may omit. Resolve identities through dictionaries, not displayed text:

| Observed thread type | Evidence it can establish |
| --- | --- |
| `ReviewersUpdate` | added/removed identities, initiator and server event time |
| `VoteUpdate` | recorded vote value, voter and server event time |
| `ResetMultipleVotes` | a reset and the identities actually recorded; an example-voter list is not necessarily the complete voter set |
| `RefUpdate` plus iterations | revision changes, not an individual same-head request generation |

Assignment followed by that user's vote can be self-initiated membership rather
than a request. A historical nonzero vote then reset still proves prior review.

**Transport-complete is not history-complete.** Threads List has no `top`/`skip`;
honor actual continuation metadata. Deleted/omitted events can leave a fully
received list semantically incomplete. Current `isFlagged`, membership or vote
cannot date a same-head request. Without authoritative generation/time or
prior-review-absence evidence, keep the fact `unknown` and block applicable work.
Once that gap is established, stop discovery, retain successful bindings and
report the missing fact, not a claim that the provider can never supply it.
