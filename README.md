#### Core design ->  peter-evans/workflow-dispatch 
<p>
Repos/project fork a template repo for the workflows

Repos own deployment: Databricks CLI/DABs, workspaces, policies, and secrets live in each repo’s Environments.

Central repo orchestrates: One workflow kicks off the env chain across any number of repos with global gates (single approval per env for all projects).

Trigger policy:

Feature branches → DEV-only preview (in the app repo on PR) — no central orchestration.

Releases → tag on main (v*.*.*) — app repo “notifies” central, which then runs DEV→QA→UAT→PRD and waits for each repo’s result.

Technically, central never touches other repos’ secrets. It simply triggers a workflow inside each repo and waits for completion.

Uses peter-evans/workflow-dispatch to trigger the app repo’s deploy_central.yml and wait for completion. Adds one shared approval gate per stage via central Environments.

Central gates (qa-global, uat-global, prod-global) give you one approval covering all repos for each stage.

Each repo deploys inside itself (so it uses its own Databricks creds & DABs).

wait-for-completion: true makes central block until each repo finishes per env.

start/stop let you run partial chains (e.g., QA→PROD hotfix). 



##### Understanding the Workflow ,
<p> Human-readable name shown in the Actions UI. </p>
name: Promote (QA/UAT/PROD) with checkboxes 

on:
  repository_dispatch:
    types: [promote_request]  # creates a run you'll see in Actions list

<p> The workflow auto-starts when another repo (your app repo A/B/C) calls the GitHub API repository_dispatch with event_type: "promote_request".

The app repo includes client_payload with things like repo_full_name and sha. Those land in github.event.client_payload.* here.</p>
