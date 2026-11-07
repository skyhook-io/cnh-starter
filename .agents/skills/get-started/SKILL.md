---
name: get-started
description: Check that this person is set up for the hackathon, explain what they see in Radar, and lay out the day, the way a coach at the Radar table would. Use when they ask to get set up, check their setup, or what to do first, when they ask what they're looking at in Radar, and at the start of a first session in this repo. Read-only.
---

# Get set up for the hackathon

Check these in order. At the first one that fails, say in a sentence or two what to do, and wait. Keep replies short. This skill only reads: change nothing in the cluster or the repo without asking.

1. **Clone set up?** If `namespace:` in `deploy/kustomization.yaml` still says `team-__TEAM__`, ask for the team name they typed on the join page and offer to run `./scripts/setup.sh <team>`. It only edits this clone; push it only when they say so (README, "Ship your own code"). Otherwise the team is the part after `team-`, written `<team>` below.

2. **Radar connected?** If you have no Radar tools, or a call says the server needs authentication, the person signs in; you can't do it for them.
   - Claude Code: if `/mcp` doesn't list radar-cloud, run `claude mcp add --scope user radar-cloud --transport http https://api.radarhq.io/mcp` in a terminal and restart Claude Code. Then `/mcp` > radar-cloud > **Authenticate**.
   - Codex: `codex mcp add radar-cloud --url https://api.radarhq.io/mcp`, then restart Codex.
   - Other agents: in Radar Cloud, open **Agents** and pick theirs.

   In the browser they sign in with the email their team listed, and under **Organization access** choose **Cloud Native Hackathon**.

3. **Cluster found?** Call `list_clusters` and take the entry whose `name` is exactly `<team>`; its `id` is the `cluster` for every other call. If it's missing and so is `cnh`, the sign-in went to another organization: sign in again (`/mcp` > radar-cloud) and choose Cloud Native Hackathon. If it isn't offered, they haven't accepted the invite email yet. If `cnh` is there but `<team>` isn't, check the spelling and `https://join.cnh.radarhq.io/status/<team>`.

4. **Readable?** Call `list_resources` with `kind: deployments`, `namespace: team-<team>`. If Radar refuses or can't read any namespace within a few minutes of sign-up, that's normal: access lands a minute or two after signing up. Say so, wait a minute, retry, for up to about 5 minutes. After that: check they signed in with an email the team listed, then send them to the Radar table. In Radar Cloud the same lag shows as **No Namespace Access … contact your administrator**: wait a minute and reload.

   When it works, say what's running (Deployment `app`, pods ready, image), whether the Argo CD Application `app` exists in `team-<team>` (`get_resource`, group `argoproj.io`; if not, GitOps isn't started: the `ship` skill), and the URL `https://<team>.cnh.radarhq.io`.

5. **The first screen.** Their cluster is the card named exactly `<team>` on Radar Cloud's Home; the rest are other teams' and the platform's (`cnh`). Call `issues` and sort it against "What every team sees" in AGENTS.md: name the known noise in a line, and point out anything else.

6. **The day,** in a few lines:
   - **Before:** Radar Cloud > the cluster > **Checks**, set **Namespace** to `team-<team>`, screenshot.
   - **Ship** their own code through GitOps (offer the `ship` skill), then **build**: README, "What Radar can do for you".
   - **One AI investigation** (the judges look for it; an all-clear on a healthy app counts). Open a resource in Radar Cloud, such as Deployment `app`: the small sparkle button at the top right, next to Refresh, starts it. It only says **Investigate** when the resource has a problem. The Issues page has an **Investigate** button per issue.
   - **By 5:30:** the story from Radar: timeline, investigation, Checks before and after.

   Then ask what they want to build or do first, and start nothing until they say.
