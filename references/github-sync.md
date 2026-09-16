# GitHub synchronization

Repository: https://github.com/xuemingqi/xuemingqi-coding-style

Branch: `main`

## Scope and triggers

- Use local files immediately during normal coding tasks. Do not fetch, pull, check remote versions, or contact GitHub merely to load or apply the skill.
- After the user asks to change this skill and the changes are complete, publish the intended changes to this repository as part of that task, unless the user requests a local-only change.
- Download remote changes only when the user explicitly asks to pull, update, or synchronize from GitHub. Do not install timers, startup hooks, or filesystem watchers.
- Synchronization is a maintenance workflow performed by the agent or by the user with Git. Saving a file in an editor alone does not trigger publication.
- These rules cover only this skill and its supporting files. They do not authorize publishing other projects or unrelated local files.

## Install on another computer

Clone directly into the personal skill directory, so the installed files and the Git working directory are the same copy:

```sh
mkdir -p ~/.agents/skills
git clone https://github.com/xuemingqi/xuemingqi-coding-style.git ~/.agents/skills/xuemingqi-coding-style
```

If the destination already exists, compare and preserve its local changes before associating it with the repository; do not overwrite it blindly. Public cloning and pulling do not require GitHub write access. Publishing requires an authenticated Git client or another authorized GitHub write interface.

## Publish a completed skill change

1. Work in the installed skill directory. Check `git status`, the `origin` URL, and the diff. Preserve unrelated changes and do not publish secrets or local runtime files.
2. Verify changed Markdown links and the skill metadata. Stage only the intended files, commit with a descriptive message, then run `git push origin main`.
3. Verify the remote commit or read back every changed file before reporting synchronization as complete.
4. If Git authentication is unavailable, an already authorized GitHub connector or signed-in browser may publish the same changes. First compare remote content with the local base to avoid overwriting concurrent changes. Verify the published files, then fetch and reconcile the local commit history only if doing so preserves every local working file. This reconciliation belongs to publishing; it must not import unrelated remote edits.
5. If push is rejected because the remote has moved, or reconciliation would change local files, keep the local work and report the conflict. Do not force-push, discard changes, or silently pull/rebase. Ask the user to request a pull and resolve the divergence.

## Pull only on request

Run in the installed skill directory after the user explicitly requests an update:

```sh
git status --short
git pull --ff-only origin main
```

If there are uncommitted changes, preserve them and resolve how they should be handled before pulling; do not silently stash, reset, or overwrite them. If histories have diverged, stop and explain the conflict instead of forcing the update. After a successful pull, verify the latest commit and the changed skill files. The updated working files are already the installed skill; no second copy step is needed.
