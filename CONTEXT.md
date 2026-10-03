# Shared project context

## Model

A host can run independent Herdr server sessions. Each session owns spaces
(workspaces), tabs, and panes; a pane may run an agent, shell, tests, or a server.
Agent and pane IDs are scoped to their server. Saved machine profiles target one
session, not all sessions on a host. A client can display multiple machines.

## Findings from both conversations

- Use plain `herdr` on the Mac plus `herdr machine add vps` for a combined view.
- `herdr --remote vps` attaches directly to the remote session.
- Local is the client's built-in local connection. Saving `localhost` can create
  an unnecessary second connection to the same local session.
- Inspect `herdr machine list` before removing a duplicate using its actual ID.
  Removing a profile disconnects the client without stopping the remote server.
- Check `herdr machine status vps`; use `herdr machine reconnect vps` for SSH
  authentication recovery where supported by the installed release.
- UI selection does not retarget CLI commands in existing panes. Use `hostname`
  to establish execution location, and explicit machine targeting for automation.
- An always-on VPS is useful for ongoing work; a sleeping laptop cannot keep
  executing agents merely because its terminal client was detached.

## Android

The Herdr Mobile app discussed in the Mac conversation is the WNT app with package
ID `dev.herdr.mobile`. Its published privacy policy stated password-only SSH
authentication during this investigation. The initial suggestion of this app did
not mention that limitation; the later answer corrected it. This is an app
limitation, not a limitation of Herdr's SSH workflow.

Termius or Termux can provide key-authenticated SSH access followed by `herdr`.
The tradeoff is terminal navigation and rendering on a small touchscreen instead
of app-specific agent views and attachment controls. No Android client was tested
end to end in these sessions.

Sources:
- [Herdr Mobile privacy policy](https://herdrmobile.zephwu.com/privacy-policy)
- [Herdr Mobile store listing](https://play.google.com/store/apps/details?id=dev.herdr.mobile)
- [Termius Android](https://www.termius.com/free-ssh-client-for-android)
- [Termux](https://github.com/termux/termux-app)

## Verified at consolidation

Both machines report Herdr 0.9.3. The two original project directories contain no
files or Git history. One relevant Codex conversation was found on each host.
The VPS Codex memory directory contained no files, and no project-specific Claude
memory directory was found there. Local global memory included an older Herdr
installation/workspace observation; current filesystem and version checks take
precedence over that observation.

## Open checks

- Verify Android key authentication and interactive Herdr rendering on the device.
- Verify current saved-machine connectivity before treating historical profile
  listings as current state. The VPS conversation suggested removing a duplicate
  localhost profile but did not record confirmation that it was removed.
- Recheck mobile app capabilities before choosing an app based on these notes.
