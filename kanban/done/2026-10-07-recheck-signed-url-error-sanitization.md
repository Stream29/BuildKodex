# REVIEW READY — Session 558

- **B1 CLOSED on the exact current snapshot. Scoped publisher/CI deployment
  APPROVED within the user's existing authorization.** No remaining static B1
  found in this narrow delta; real host/publication acceptance remains U below.
- Independent read-only reviewer, not implementer; ready for asynchronous
  coordinator reading without waiting for tests, runtime or another worker.
- Session557's original snapshot remains **HOLD**, not retroactively approved.
  This is not default-consumer, IDE-performance or Kodex-release approval.

# Task Tree

- `Independently recheck the signed-URL diagnostic B1 on exact final bytes`()
- `Return the narrow publisher deployment verdict without running code`()

# Details

- Parent: [Fork CI](2026-10-07-implement-fork-package-ci.md).
- BEFORE: [Session557 verdict:1–13](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-recheck-registry-read-and-java21-probe.md#L1),
  `/tmp/kodex-fork-registry-read-review-20261007/`; manifest digest
  `7e78ed6f3d560865327dc5cf5d72f74e4328c1d1b0251c9b9fa25703cd7ddaa3`.
- CURRENT ONLY: [manifest:1–3](file:///tmp/kodex-fork-signed-url-review-20261007/SHA256SUMS#L1);
  independently matched digest
  `3c840e4d5af5430eca94cf2895a00a72c23e9648df31edf07bacf5ed4ef58057`
  and verified **3/3 file hashes**, including a second check:
  - `publisher.py`: `b931567e8432a1760f52cc237e26d9b63a39d3063b8310f0da5d1c2078638532`.
  - `smoke.py`: `6942a1be417e8369343bc30f5ed9bf3f4883e8d016b4f336c26e2b4f2d5e36eb`.
  - `test_publication.py`: `a800ce0067e3823180f01cdc37c993a08a51061485a9839b247deb3ea561d2b5`.
- Loaded AGENTS, associated change/kanban/planning/checklist/ask-user/Gradle,
  document/workspace/toolchain/IDE-collaboration skills, Draft and relevant
  parent authorization. No mutable current Kodex scripts were read.
  Historical context only: dc3 contract/pipeline loose blobs
  `04aad7928920973117bc1925370b5b8992b69b21` /
  `b2db82ae4b1a0d67c2a4eadcc27082c9446a4f89`, decoded to stdout and SHA-1
  checked without Git commands or executing their source.

## R / B1 — transport ownership and confirmed closure

- [NoRedirect:22–37](file:///tmp/kodex-fork-signed-url-review-20261007/publisher.py#L22)
  overrides `http_error_302` and **301/303/307/308 aliases**, raising raw HTTPError
  before inherited parsing/quoting/joining/recursive open. The actual Maven opener
  installs this subclass. CPython replaces the default redirect handler and
  shares HTTP/HTTPS error dispatch: HTTPS/308 cannot bypass these overrides.
  [CPython3.12 urllib.request](https://raw.githubusercontent.com/python/cpython/3.12/Lib/urllib/request.py).
- [Manual block:54–81](file:///tmp/kodex-fork-signed-url-review-20261007/publisher.py#L54)
  saves status/Location and **closes HTTPError before every return/refusal/hop**.
  Before `urljoin`, only raw ASCII `!` through `~` passes: space/TAB/DEL/NUL/
  non-ASCII fail without echo or repair. Malformed brackets/ports fail closed.
  Resolved target requires HTTPS, exact official storage host, default/443,
  no userinfo and no nonempty fragment. Lookalikes/HTTP/other ports/empty userinfo
  fail; relative Maven redirects fail, later storage hops recheck the predicate.
- [Request boundary:47–83](file:///tmp/kodex-fork-signed-url-review-20261007/publisher.py#L47)
  covers construction **inside try**, open and read. ValueError (including Unicode
  encoding subclasses), HTTPException, URLError, OSError and TimeoutError become
  safe RuntimeError **from None**. HTTP status/refusal messages also omit reason,
  body, headers and signed URL. The old query-bearing InvalidURL cannot escape
  these paths into a rendered publisher chain.
  [CPython3.12 HTTP client](https://raw.githubusercontent.com/python/cpython/3.12/Lib/http/client.py).
- HTTPError owns a file-like response, not just metadata; its close reaches the
  response through urllib's wrapper. Ordinary response context management closes
  on return, oversized-body rejection and read failure; urllib closes its
  connection on open failures.
  [HTTPError](https://raw.githubusercontent.com/python/cpython/3.12/Lib/urllib/error.py),
  [response wrapper](https://raw.githubusercontent.com/python/cpython/3.12/Lib/urllib/response.py),
  [wrapper close](https://raw.githubusercontent.com/python/cpython/3.12/Lib/tempfile.py).
- Four requests maximum / three followed GET redirects, each timeout60.
  Basic auth stays on the original Maven request; every storage hop gets fresh
  User-Agent-only recipe headers. PUT follows **no redirect**, including308.
  Initial Maven404 alone means absence; storage404 is a safe read error.
  Other statuses fail through sanitized status handling.
- No inherited requoting remains. Absolute signed `%2F`, `%3D`, `+`, mixed-case
  host and explicit443 preserve the reviewed URL/query spelling, rather than
  decoding and rebuilding its signature.
  [CPython3.12 URL joining](https://raw.githubusercontent.com/python/cpython/3.12/Lib/urllib/parse.py).
- [Publisher:139–196](file:///tmp/kodex-fork-signed-url-review-20261007/publisher.py#L139)
  is unchanged: safe operation, owned artifact path and acknowledged PUT count
  wrap sanitized HTTP transport failures. Preflight prints that safe failure;
  final verification chains it. Pinned contract validation/normalization supplies
  expected paths; signed selectors never become diagnostic `path`. Pinned
  `live_main` uses a public credential-free Git URL, not a signed-URL bypass.
  Acknowledged count does not prove an unacknowledged PUT never reached the server.
- **Static minimal ablation:** reverting to `redirect_request` alone restores
  inherited parsing before the owner's predicate; dropping raw validation permits
  parser cleanup/signature rewriting; moving construction outside try or removing
  HTTPException/from-None handling restores URL-bearing exception responsibility
  gaps. The small transport now owns all three. No generic framework/provider
  or second publication owner is needed.

## B2 — concrete nonblocking diagnostic/test debt

- [Size guard:51–52,82–83](file:///tmp/kodex-fork-signed-url-review-20261007/publisher.py#L51):
  oversized-body ValueError becomes generic “transport failed.” This loses a useful
  size diagnostic, **not** the bound, closure or secrecy: B2 debt, not unsafe PASS.
- [New fixtures:1111–1169](file:///tmp/kodex-fork-signed-url-review-20261007/test_publication.py#L1111)
  exercise loopback HTTP302 and intercept the valid HTTPS hop, asserting exact
  signed spelling and stripped Basic. Malformed IPv6/space/NUL/DEL/TAB exercise
  the actual handler and rendered chain. Protocol injection uses the **actual
  InvalidURL class** in preflight and final read after PUTs, asserting the latter's
  verification path; it is not real TLS or lower-level InvalidURL generation.
- Existing auth/status, unsafe-host, PUT/hop, storage404, partial/different,
  interruption and stale-HEAD assertions retain their meaning; no broad
  exception-swallowing replacement. Credentials are dummy; fixture logging is
  disabled. Real-handler alias statuses, explicit close assertions and constructor/
  read-fault cases would strengthen coverage, not reopen this static B1.
  **Tests were read, not executed by this reviewer.**

## U — controlled rollout, not completed runtime acceptance

- `smoke.py` is independently **byte-identical** to Session557's approved file.
  [Compile21:193–195](file:///tmp/kodex-fork-signed-url-review-20261007/smoke.py#L193),
  [JNI21/FFM25 launchers:225–267](file:///tmp/kodex-fork-signed-url-review-20261007/smoke.py#L225)
  and [commands/receipt:286–305](file:///tmp/kodex-fork-signed-url-review-20261007/smoke.py#L286)
  retain that static approval. Actual three-host Mosaic Java21 JNI, Java25 FFM,
  Native/runtime gates and receipts still precede its writer. Their unexecuted
  state is **U, not another static deployment B1**.
- [Coordinator evidence:133–153](file:///home/stream/ACodeSpace/push/BuildKodex/kanban/done/2026-10-07-implement-fork-package-ci.md#L133)
  records original dc3 MCP **453/453** and Lucene **133/133** exact files,
  including manifest/checksums, no missing/different/read-error and no PUT/DELETE.
  Accept the supplied audit as coherent existing-artifact evidence, not a new
  reviewer audit. Original CI **FAILED** statuses remain; no cleanup/resume needed.
- Changed publisher/tests change recipe hashes. **Do not rerun SDK/Lucene
  publication or overwrite their existing versions with this recipe.**
  Original-contract audits remain valid; later consumer acceptance may use those
  retained binaries without touching them. Do not evade recipe equality or
  misrepresent a new marker as the old exact bundle.
- Mosaic's writer never ran: a fresh version-controlled, reviewed CI run after
  deployment is valid within existing authorization, retaining exact-main,
  ancestry, receipt and remote-byte gates. Partial/different versions fail sealed
  for separately authorized manual remediation, not deletion/overwrite.
- Current-byte offline execution is unverified here; the older reported 74-case
  pass is not rebound to changed bytes. Coordinator owns validation/real rollout.
  Root settings/catalog remain unchanged pending Mosaic and consumer gates;
  no full IDE/model/performance claim.

## Handoff

- **REVIEW READY / scoped deployment APPROVED / old signed-URL B1 CLOSED.**
- Wrote only this owned report with `apply_patch`. No sources/other documents,
  tests (including pytest), Gradle/build/runtime, IDE, device/credential operations,
  Git commands, process controls or network writes. Public primary-source reads
  and local hashing/diffs/object decoding only; no temporary files, services,
  windows or persistent resources created.
