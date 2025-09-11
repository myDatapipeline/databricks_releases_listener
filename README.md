#### Core design
<p>
Repos/project fork a template repo for the workflows

Repos own deployment: Databricks CLI/DABs, workspaces, policies, and secrets live in each repo’s Environments.

Central repo orchestrates: One workflow kicks off the env chain across any number of repos with global gates (single approval per env for all projects).

Trigger policy:

Feature branches → DEV-only preview (in the app repo on PR) — no central orchestration.

Releases → tag on main (v*.*.*) — app repo “notifies” central, which then runs DEV→QA→UAT→PRD and waits for each repo’s result.

Technically, central never touches other repos’ secrets. It simply triggers a workflow inside each repo and waits for completion.

</p>