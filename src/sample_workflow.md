## Sample flows
##### A) repoA & repoB—feature branch preview to DEV

Trigger: Open/Update PR on feature/abc in repoA and repoB.

Runs: Each repo’s DEV Preview (PR):

Deploys to DEV using an isolated stack pr-<number>.

Optional smoke tests.

On PR close, destroy preview resources.

Central: not involved (keeps releases clean).

##### B) repoA & repoB—merge to main and cut tag v1.2.3 (release)

Tag: Each repo cuts v1.2.3 on main (manually or via Release-Please).

Notify: Each repo fires release_ready to central with { repo, version }.

Central (one run per repo/set):

DEV job: Triggers deploy_central.yml in repoA & repoB with {env:dev, version:v1.2.3}, waits for both to finish.

QA job: Pauses at qa-global for approval; then triggers QA deploys in both repos and waits.

UAT job: Pauses at uat-global, triggers UAT for both, waits.

PROD job: Pauses at prod-global, triggers PROD for both, waits → done.

Audit: Central run shows the entire chain; each step links to the underlying app-repo run (visible in the action’s summary).

#### Failure, retry, rollback, concurrency

Failure in a repo at a stage: That env job fails; no promotion to next env. Fix repo (config/code), re-run central from that stage using workflow_dispatch with start=qa (for example).

Retry single repo: Re-run that matrix leg by re-running the central job or trigger app repo’s deploy_central.yml manually once.

Rollback: Re-run central with an older tag (no rebuild; DABs deploy is idempotent).

Multiple releases in flight: Allowed (different versions). concurrency: central-promotion-${version} keeps each version serialized while allowing others.

Many repos: Matrices run in parallel per stage. Set fail-fast: false to gather all results even if one fails.

#### #### Guardrails & policy

Tag discipline: Protect main; protect tags v* (only release maintainers can tag).

Global gates: Add required reviewers on qa-global, uat-global, prod-global in central.

Per-repo gates: You can also keep approvals inside app repos (in addition to global) if you want dual control.

Secrets hygiene: Store Databricks creds in the app repo environments; prefer OIDC over PATs.

Resource isolation: Always use DABs var.stack for PR previews (prevents collisions) + optional TTL cleanup.

#### Optional enhancements

Auto-tagging (Release-Please) in each repo to convert merges to versioned tags with changelog.

Manifest in central of repo groups (e.g., engineering vs mlops vs data-science) so you can select groups quickly.

Telemetry: Have each app repo append a deploy_result back to central (simple repository_dispatch with {repo, env, version, status, run_url}) and write it to Delta via a tiny central ingest workflow for dashboards.