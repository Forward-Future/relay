# Relay deployment

Use this branch only when the user authorized deployment. Assign the settled release plan to an execution model appropriate for the repository's release mechanics. The release thread executes the plan; it does not invent one during deployment. If production evidence invalidates the plan, stop the release and return evidence to the coordinator for renewed planning, execution, and independent review.

When the user says `deploy`, commit and land only the relay's authorized task changes, then deploy affected services from the newest `origin/main` revision containing that commit.

1. Use the repository's shared deployment lock and wait for any active deployment to finish.
2. After acquiring the lock, discard previously selected revisions and fetch again.
3. In a dedicated clean deployment checkout with an attached branch and configured upstream, run `git pull --ff-only`.
4. Stop without discarding local work if the checkout is dirty, detached, missing its upstream, diverged, or cannot pull.
5. Verify the thread commit is an ancestor of the pulled revision.
6. Build complete artifacts for affected services from that integrated revision. Never deploy from a task worktree, feature branch, dirty checkout, or partial file overlay.
7. Hold the deployment lock through production health verification.
8. Never deploy an older revision after a newer one unless the user explicitly authorizes a rollback.

Deployment passes only when the integrated revision, affected services, deployment result, and production health evidence are recorded. Review of the implementation must pass before release; production verification remains the release thread's final gate.
