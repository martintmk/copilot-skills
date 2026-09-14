# Focused provider CLI recipes

These are operation candidates, not cached authorization. Validate installed
help, provider metadata, credentials and repository scope before binding them.
Use placeholders only after resolving their values; serialize write payloads
to files. All hosting calls remain MCP or official provider CLI operations with
`direct_http: false`. Public documentation and installed CLI source can explain
an operation, but are not permission to call the SDK or raw HTTP directly.

## Check tools without installing them

In the PowerShell probe process, disable implicit extension installation:

```powershell
$env:AZURE_EXTENSION_USE_DYNAMIC_INSTALL = 'no'
```

Check `gh --version`, `az version`, and
`az extension show --name azure-devops` before extension commands. A missing
extension must not trigger its interactive installer merely from `--help`.
After explicit setup approval, `az extension add --name azure-devops` is the
official installation route. Do not change global defaults or credentials.

## GitHub

Use separate identity reads: `gh api --hostname <host> user` for the poster and
`gh api --hostname <host> users/<target>` for the target. Retain stable IDs and
provider type, then read `repos/<owner>/<repo>` for its immutable ID/access.
Review delivery and read-back use the shared delivery contract.

For paginated arrays, choose one supported output form:

- `gh api --paginate --slurp <endpoint>` returns an outer array of pages to
  parse and flatten locally.
- `gh api --paginate --jq <filter> <endpoint>` applies the filter to each page;
  do not treat its concatenated output as one JSON array.

GitHub CLI 2.74.2 rejects combining `--slurp` with `--jq` or `--template`.
Preserve all pages and source fields needed for identity/history even if the
display is compact. Read the complete PR reviews list, issue timeline and
current requested reviewers separately: one does not replace the others.
The requested-reviewer removal API accepts user logins, **not a request-cycle
condition**; the queue's race and acknowledgment gates remain mandatory.

## Azure DevOps identity and repository

Use `az repos show --repository <name-or-id> --organization <org-url>
--project <project>`; `--id` is not its repository selector. Once resolved,
carry the exact organization URL, project ID and repository ID on every call.

The official creator resolver provides a focused identity path:

```text
az repos pr list --organization <org-url> --project <project-id> --repository <repo-id> --creator <confirmed-target-upn> --status all --top 1
az repos pr list --organization <org-url> --project <project-id> --repository <repo-id> --creator me --status all --top 1
```

In the Azure DevOps extension 1.0.8, `--creator me` resolves
`ConnectionData.authenticated_user.id`; a non-GUID UPN resolves through the
provider identity service. Confirm that behavior for the installed version,
then retain each result's `createdBy.id`. These are two independent resolutions,
not an email/display-name comparison. An empty result does **not** resolve an
identity: use another proven identity operation or report the gap, without
searching unconfigured repositories.

When MCP exposes a documented `createdByMe` filter, the same exact-repository
sample plus a full PR read can independently resolve its posting actor. Do not
assume the MCP and CLI credentials match. All chosen write routes must resolve
to the same approved actor.

For provider classification and descriptor normalization, validate the Graph
`users` and `storageKeys` resources through `az devops invoke`:

| Resource | Route parameters | Required evidence |
| --- | --- | --- |
| `graph / users` | `userDescriptor=<provider-returned-descriptor>` | subject kind, origin and member/service classification |
| `graph / storageKeys` | `subjectDescriptor=<same-descriptor>` | storage-key GUID matching the `IdentityRef.id` |
| `core / projectCollections` | scoped list, then `collectionId=<returned-id>` | immutable hosting collection ID and its kind |

Do not decode descriptor text and substitute its embedded identifier for the
storage key. A project-collection ID is not an Accounts account ID, Entra tenant
ID or identity-domain ID; retain the kind of the scoped identifier actually read.
An entitlement-service permission denial is not permission to retry that
protected data through another interface.

## Azure DevOps invocation and history

Use **both** `--area` and `--resource`, with the repository-scoped route values,
an explicit API version and `--http-method GET` for probes:

```text
az devops invoke --organization <org-url> --area git --resource pullRequestThreads --route-parameters project=<project-id> repositoryId=<repo-id> pullRequestId=<pr-id> --api-version 7.1 --http-method GET
```

Do not use argument-free `az devops invoke` for preflight discovery: it walks
many service locations and can stall on unrelated services. In extension
1.0.8, providing an area without a resource can fail with a `NoneType.lower`
error. Some resource-area and resource names also differ; use installed
metadata/docs rather than guessing names such as `location` or `profile`.
The same extension's API-version parser accepts `7.1-preview` but rejects
`7.1-preview.1`; verify the resource supports the selected version instead of
blindly rewriting every version string.

Useful validated Git resource names include `pullRequests`,
`pullRequestIterations`, `pullRequestThreads`, `items`, and
`pullRequestStatuses`. Preserve pagination/continuation metadata and select
CI evidence by the pinned iteration, not the first status in the response.
For effective permission evidence, discover the Git Repositories security
namespace and read the exact repository token with
`az devops security permission show`; never probe by casting a vote.

Raw `pullRequestThreads` retains system properties and identity dictionaries
that a compact MCP projection may omit. Resolve every identity reference
through the thread's dictionary, not its displayed comment text:

| Observed thread type | Evidence it can establish |
| --- | --- |
| `ReviewersUpdate` | added/removed identities, initiator and server event time |
| `VoteUpdate` | recorded vote value, voter and server event time |
| `ResetMultipleVotes` | a reset and the identities actually recorded; an example-voter list is not necessarily the complete voter set |
| `RefUpdate` plus iterations | revision changes, not an individual same-head request generation |

An assignment immediately followed by that user's vote can be self-initiated
membership, not a pending request. A historical nonzero vote followed by a
reset proves a prior review despite a current zero vote.

**Transport-complete is not history-complete.** The threads List API has no
`top`/`skip` parameters; do not invent them for the raw route. Account for
continuation metadata actually returned. Even a fully received list can contain
deleted threads/comments or omit the event semantics needed for re-requests.
Current `isFlagged`, membership or vote values do not establish when a same-head
request began. Prove the generation/time mapping and prior-review absence from
an authoritative contract/history; otherwise keep those facts `unknown` and
stop the applicable work under the state machine.

Once this semantic gap is established, do not spend the remaining budget
enumerating more APIs or claiming the provider can never supply the evidence.
Report the exact unsupported fact and retain the successful bindings.
