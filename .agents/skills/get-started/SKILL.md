---
name: get-started
description: Check that this person is set up for the hackathon, explain what they see in Radar, and lay out the day, the way a coach at the Radar table would. Use when they ask to get set up, check their setup, or what to do first, when they ask what they're looking at in Radar, and at the start of a first session in this repo. Read-only.
---

# Get set up for the hackathon

Coach the person through their first minutes. Check the steps below in order. At the first one that fails, say in a sentence or two what to do, and wait for them. Keep replies short.

This skill only reads. Don't change the cluster or the repo without asking first.

## 1. Is this clone set up?

Read `namespace:` in `deploy/kustomization.yaml`. If it still says `team-__TEAM__`, `./scripts/setup.sh` hasn't run in this clone. Ask for the team name they typed on the join page, and offer to run `./scripts/setup.sh <team>`. It only edits files in this clone. Then it needs a commit and push (README, "Ship your own code", step 2): do that only when they say so.

Otherwise the team is the part after `team-`. It's `<team>` below.

## 2. Is Radar connected?

If you have no Radar tools (`list_clusters` and the others), or a call says the server needs authentication, the person has to sign in. You can't do it for them.

- **Claude Code:** run `/mcp`, pick **radar-cloud**, then **Authenticate**. If `/mcp` doesn't list radar-cloud, it was skipped when Claude Code asked. Quit, run `claude mcp reset-project-choices` in this folder, start `claude` again, and choose **Use this MCP server**.
- **Codex:** run `codex mcp add radar-cloud --url https://api.radarhq.io/mcp` in a terminal. It opens the browser to sign in. Then restart Codex.
- **Cursor:** Settings > Tools & MCP > radar-cloud > **Connect**.
- **VS Code:** open `.vscode/mcp.json` and click **Start** above radar-cloud.
- **Another agent:** in Radar Cloud, open **Agents** and pick it.

A browser opens. They sign in with the email their team listed on the join page, and under **Organization access** they choose **Cloud Native Hackathon**. Then they come back here.

## 3. Find the team's cluster

Call `list_clusters`. Take the entry whose `name` is exactly `<team>`. Its `id` is the `cluster` for every other call.

If there's none:
- **No `cnh` in the list either:** the connection went to another organization. They sign in again (Claude Code: `/mcp` > radar-cloud) and choose **Cloud Native Hackathon**. If it isn't offered, they haven't accepted the invite yet: open the invite email and sign in at app.radarhq.io.
- **`cnh` is there, but not `<team>`:** check the spelling. Then check `https://join.cnh.radarhq.io/status/<team>`: the cluster appears once that page says it's set up.

## 4. Can you read it?

Call `list_resources` with `kind: deployments` and `namespace: team-<team>`.

If Radar refuses, or says it can't read any namespace, and they signed up in the last few minutes, that's normal: access lands a minute or two after signing up. Say so, wait about a minute, and try again, for up to about 5 minutes. If it still fails, check they signed in with an email the team listed. If they did, send them to the Radar table.

When it works, say what's running: Deployment `app`, how many pods are ready, and its image. Check whether the Argo CD Application `app` exists in `team-<team>` (`get_resource`, group `argoproj.io`). If it doesn't, GitOps hasn't started yet: that's step 5 of the README's "Ship your own code", or the `ship` skill. Their app is at `https://<team>.cnh.radarhq.io`.

## 5. Explain the first screen

Tell them:
- Their cluster is the card named exactly `<team>` on Radar Cloud's Home. The other cards are other teams' clusters, and `cnh` is the platform's.
- If the cluster says **No Namespace Access … contact your administrator** right after they signed up, wait a minute and reload.
- Some of what Radar shows on a new cluster isn't theirs to fix. Go through "What every team sees" in AGENTS.md in a few lines.

Call `issues` with the cluster id. Sort what it returns into the known items from that list and the rest. Point out anything in `team-<team>`, or anything else not on the list.

## 6. Lay out the day

In a few lines, from the README's "The day":

1. **Before:** in Radar Cloud, open the cluster, then **Checks**. Set **Namespace** to `team-<team>` and take a screenshot. Unfiltered, the list is mostly the pre-installed platform.
2. **Ship:** start GitOps and see their commit live at the URL. Offer to do it with the `ship` skill.
3. **Build:** pick what the project needs from the README's "What Radar can do for you".
4. **Investigate once:** the judges look for at least one AI investigation, and an all-clear on a healthy app counts. In Radar Cloud, open a resource, such as Deployment `app`. The small sparkle button at the top right, next to Refresh, starts one. It only says **Investigate** when the resource has a problem. The Issues page has an **Investigate** button on each issue too.
5. **By 5:30:** tell the story from Radar: the timeline, the investigation, and Checks before and after.

Then ask what they want to build, or what to do first. Start nothing until they say.
