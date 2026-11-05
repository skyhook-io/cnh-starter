# Dependencies, configuration and one-off work

Use these patterns when your project needs them. They are not judging requirements.
Replace `team-__TEAM__` with your assigned namespace. Discover the tools your connected
Radar advertises before using a tool: a tool described in newer Radar documentation
may not be deployed on this cluster yet.

## Keep credentials out of your public repository

Commit a `secretKeyRef` in your Deployment, never the credential value:

```yaml
env:
  - name: API_KEY
    valueFrom:
      secretKeyRef:
        name: app-credentials
        key: api-key
```

Create `app-credentials` in your namespace through Radar's resource editor, using
`type: Opaque` and `stringData.api-key` for the value. This Secret is deliberately
outside Argo CD: do not add its manifest to `deploy/`, a PR, your image, or a build log.
The generic `apply_resource` MCP tool can create a Secret too, but supplying its value
to an agent puts it in that agent's conversation/tool history. Prefer entering it in
the UI. A Secret reference in git does not create the Secret.

After changing a Secret consumed through environment variables, restart the workload
through `manage_workload`, then use `wait_for` to check the rollout. Don't print the
credential to prove it exists; check that the app's authenticated operation succeeds.

## Run one-off work as a Job

For a migration, import or batch operation, create a uniquely named Job through
`apply_resource` with `mode=create`. Use your already-built public application image,
its actual command, and only the permissions that command needs. This example assumes
your image contains `node` and `scripts/import.js`; change both for your app.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: app-import-001
  namespace: team-__TEAM__
spec:
  backoffLimit: 0
  activeDeadlineSeconds: 300
  ttlSecondsAfterFinished: 3600
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: import
          image: ghcr.io/YOUR-OWNER/YOUR-REPO:YOUR-BUILT-TAG
          command: [node, scripts/import.js]
          securityContext:
            runAsNonRoot: true
            allowPrivilegeEscalation: false
            seccompProfile: {type: RuntimeDefault}
            capabilities: {drop: [ALL]}
          resources:
            requests: {cpu: 100m, memory: 128Mi}
            limits: {cpu: 500m, memory: 256Mi}
```

Use `wait_for` with `kind=job`, the namespace/name and `condition=complete`. A `waiting`
result means the Job has not finished; call again within your own deadline. A `failed`
result needs investigation. Read `get_workload_logs` or inspect the Job's Pods to see
what the command actually did. Kubernetes completion only establishes that the process
exited successfully; verify the application's intended result too.

Jobs and non-git Secrets are exceptions to the starter's normal GitOps rule: they are
not resources Argo CD owns. Do not reuse a Job name to rerun work, or put a completed
one-off Job into the continuously reconciled app manifests. Make retries safe for your
data before rerunning a migration or import.

## Add a dependency

For a small database or queue, put its namespace-scoped Deployment/StatefulSet and
Service in `deploy/`, add them to `deploy/kustomization.yaml`, and push. Use a PVC when
data must survive a Pod replacement. Check the cluster's available storage and your
quota before choosing requests; your dependency shares the team's quota with the app.

If staff has allowed the chart repository in your AppProject, a chart can also be
managed by an Argo CD Application in your team's namespace. Use an exact chart version and check its rendered resources first. Charts
that need CRDs, cluster-wide Roles/Bindings or new namespaces do not fit the shared
cluster's team scope; ask staff instead of requesting broader permissions. Do not run
a direct Helm upgrade against resources already managed by Argo CD or Flux.

On your own cluster, `manage_helm` can manage a native release when advertised.
The owner must preconfigure the chart repository used by that Radar instance first. Upgrade values merge with stored
values by default; full replacement requires `replace_values: true`. Keep the shared
hackathon app's dependencies in GitOps. Native Helm is not required for this track.

## Inspect the running application

- For a public endpoint, use your agent's HTTP/browser tools against your team's URL.
- For Kubernetes state and failure evidence, use Radar's resource/log tools or `diagnose`.
- If advertised and enabled, `curl_service` GETs a selector-backed internal Service
  from in-cluster Radar and returns its HTTP status/body. It sends no credentials and
  is not a test of the public ingress path. Use `namespace`, Service `name` and its
  numeric `port`, plus the endpoint `path`.
- For an in-container check, use `exec_pod` if the operator has enabled it. It accepts an
  argv array, needs the executable in the image, and runs with your Kubernetes identity.
  It has no interactive input/TTY. Never put literal secrets in argv: audit logs can
  retain the command. If it is absent, ask staff; installing a client-side MCP connection
  does not enable the server's exec capability.
- A failed command or HTTP response is evidence, not automatically a Kubernetes fault.
  Check the application's response/body and expected behavior too.

`diagnose` gathers Kubernetes evidence. For a hosted AI SRE investigation, open the
resource's **Investigate** action in Radar Cloud. Accept the disclosure there if asked.
Newer hubs may also advertise investigation MCP tools; use their returned run ID to
read the result rather than starting the same investigation repeatedly. Human-started runs are private by default. Sharing with the organization means
sharing with all its teams, not only teammates. Automatic analyses follow the
organization's separate visibility policy; MCP only reads your own runs.

### Tool prerequisites and outcomes

`wait_for` may report `blocked` (for example, scheduling or quota prevents progress).
Read its reason and relevant events before retrying. `curl_service` requires
in-cluster Radar, a selector-backed Service with a ClusterIP (not headless), exec
tools enabled by the operator, and `get services/proxy` on
the target Service. Without auth, that grant must belong to Radar's ServiceAccount.
The default Radar chart does not grant it.

For native Helm, source configuration is a prerequisite outside MCP. Local Radar
uses its Helm repository config (for example a named source added with `helm repo add`).
Operator-managed OSS can use startup Helm configuration, including `helm.ociSources`.
That chart setting is rejected in Cloud mode. Cloud has no self-service first-install
source setup in this workflow; raw owner configuration is pod-local and lost on Pod
replacement. Use the starter's GitOps dependency path for Cloud-connected clusters
unless your operator has already supplied and maintains the required source. Upgrade overrides merge maps, but
lists replace as whole units and null/type changes remove prior values. Replacing
a container with hidden credentials is refused unless you explicitly replace the
whole reviewed values configuration.

Before starting a hosted investigation, verify the exact target with `get_resource`.
Missing or mistyped targets can still consume investigation allowance.

## Confirm that your push is actually live

The image tag is the first seven characters of the app commit that triggered
`build-and-deploy`, not the later bot commit that updates the manifests.

1. Save the pushed commit SHA. Check that commit's GitHub Actions run, for example
   `gh run list --workflow build-and-deploy --commit <FULL-SHA>`, then
   `gh run watch <RUN-ID> --exit-status`. If the build failed, inspect its logs before
   debugging Kubernetes. Retry briefly if the run is not registered yet, within the
   five-minute deadline; a missing/cancelled/superseded run does not prove a build.
   Markdown-only changes need no deploy. A kustomization-only push also skips the
   Action: retain the existing image tag and verify the intended configuration
   after sync instead of expecting a new app version. Other manifest changes build.
2. Read the resulting `deploy/kustomization.yaml` from the remote repository and
   confirm `newName`/`newTag` identify that build. A successful build for another
   commit is not the evidence you need.
3. Read the Argo Application with `get_resource`. Refresh/sync through `manage_gitops`
   if necessary. Read the live Deployment (or Rollout if you changed the example)
   and confirm its pod-template image matches the expected tag. If not, keep waiting
   on the desired revision; don't report the existing healthy app as the new build.
4. Use `wait_for` for rollout completion, then request your public `/api/sessions`.
   The starter's `version` field must equal that seven-character tag (or the unchanged
   image tag when the Action intentionally skipped a configuration-only push). Keep an equivalent
   version endpoint if you replace the starter app. Verify the feature you changed too.

## Delegate a real question to hosted AI SRE

These tools live on the organization `/mcp` endpoint, not `/c/<cluster-id>/mcp`.
Discover them first; if absent, use the Cloud UI. `diagnose` collects evidence itself,
while `start_investigation` starts a hosted AI investigator and can consume allowance.
It starts with a resource assessment; arbitrary questions are follow-up turns.
Start only when the person has requested or approved a hosted investigation; explain
that it uses the shared organization allowance. An existing request to investigate
is sufficient—do not ask again for the same authorized work.

- Get `cluster` from `list_clusters`, then use `get_resource` to verify the exact target.
- Call `start_investigation` with, for example,
  `{"cluster":"clu_example","group":"apps","kind":"Deployment","namespace":"team-__TEAM__","name":"app"}`.
  When coming from a Radar issue, add its ID as `issue_id`; the Hub validates its target
  and can associate the same counted issue episode. Do not invent an issue ID.
- Save `structuredContent.result.id` as `run_id`. Call `get_investigation` with
  `cluster` and `run_id`; keep `next_after`, and use it as `after` on the next event read.
  Drain `has_more` pages. `has_more:false` only means currently saved events are drained.
- Poll with a bounded interval/deadline. For status-only checks use `include_events:false`
  and omit `after`; never start a new investigation just to check progress.
- Only ask `follow_up_investigation` when `canContinue:true`. Supply `question` and read
  events after your saved cursor for the answer, usually in `done.diagnosis.report`.
  Saved `findings` can still be the prior assessment; don't present them as the new answer.
- Stop through `stop_investigation` when appropriate, then poll until `done`, `error`,
  or `stopped`. `stopping` is not terminal. Check tool errors before reading result fields.

If disclosure or allowance blocks a start, surface the refusal. A human accepts
organization disclosure in the UI; agents cannot do that or approve spending changes.
All teams share the configured organization allowance and concurrency pool.
