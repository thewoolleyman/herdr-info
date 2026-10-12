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

## Keyboard reference

- Rename the current space (workspace) in the Herdr client UI with **Ctrl+B,
  release, then Shift+W** using the default bindings. Enter confirms the name.
  Verified against the [keyboard documentation](https://herdr.dev/docs/keyboard/)
  on 2026-10-06; custom bindings may differ.
- Rename the current tab with **Ctrl+B, release, then Shift+T**.
- Resize terminal splits with **Ctrl+B, release, then r**, followed by **h/l**
  for width or **j/k** for height; **Esc** exits. This is pane resizing, not
  sidebar resizing ([upstream README](https://github.com/motionharvest/herdr)).

## Sidebar width configuration

On 2026-10-10, changing the VPS config's `ui.sidebar_width` from the default 26
to 36 and `ui.sidebar_max_width` from 36 to 46 did not visibly resize the user's
sidebar after the suggested reload. The user confirmed the visible Herdr client
runs on the Mac and connects to the VPS. The VPS overrides were reverted.

The user then opened a reverse SSH tunnel from a local Mac terminal, using its
existing `vps` alias:

```sh
ssh -NT -o ExitOnForwardFailure=yes -o ServerAliveInterval=30 -R 127.0.0.1:22222:localhost:22 vps
```

This exposes the Mac's SSH service only on the VPS loopback interface; connecting
back still requires a valid Mac account and SSH authentication. Existing SSH
authentication worked, so no temporary authorized key was installed.

The Mac was verified to run Herdr 0.9.3 with `herdr --remote vps`. Its config had
no sidebar width overrides, and its saved client preferences contained no manual
sidebar width. The Mac config was backed up outside this repository and changed:

```toml
[ui]
sidebar_width = 36
sidebar_max_width = 46
```

The default width increased from 26 to 36 columns; the maximum increased from 36
to 46. TOML parsing and a comparison confirmed that other settings were preserved.
The repository was cloned to `~/workspace/herdr-test` on the Mac and pulled with
`git pull --ff-only` before recording the change there.
The connected VPS accepted `herdr server reload-config` with status `applied` and
no diagnostics. In v0.9.3 this broadcasts a reload to attached clients, which
reread their own local config. The on-screen result has not been independently
observed; if necessary, use **Ctrl+B, release, then Shift+R** in the Mac client.

The [v0.9.3 client reload code](https://github.com/herdrdev/herdr/blob/v0.9.3/src/client/shell/config.rs)
preserves a manually selected sidebar width on reload. Layout clamps that width
to `sidebar_min_width` and `sidebar_max_width`. A double-click on the sidebar
divider resets a manual width to the configured default, per the
[mouse handler](https://github.com/herdrdev/herdr/blob/v0.9.3/src/client/shell/mouse.rs).
These are client-owned settings: a Mac client attached to the VPS reads the Mac
config, whereas `ssh` followed by `herdr` on the VPS reads the VPS config.

## Agent status legend

Verified on 2026-10-08 against the
[v0.9.3 client source](https://github.com/herdrdev/herdr/blob/v0.9.3/src/client/shell.rs):

| Status | Theme color | Default icon | Symbol mode | Meaning |
| --- | --- | --- | --- | --- |
| Blocked | Red | ● | × | Needs input, approval, or a decision |
| Working | Yellow | ● | ◐ | Actively working |
| Done | Teal | ● | ✓ | Finished; not yet viewed in this client |
| Idle | Green | ○ | ○ | Waiting for work; no unseen completion indicator |
| Unknown | Muted gray (`overlay0`) | · | · | State cannot be determined confidently |

Actual shades depend on the theme. The v0.9.3 mobile renderer uses blue for Done.
`ui.status_indicators = "symbols"` selects the alternate static glyphs.
See also [status concepts](https://herdr.dev/docs/concepts/).

## Pane text selection

Checked against local default config and public docs on 2026-10-08:
`[ui] mouse_capture = true` enables Herdr mouse handling; `copy_on_select = true`
copies its selection automatically. Both default to true. Reload edited config
with **Ctrl+B, release, then Shift+R**. Herdr selection is scoped to pane text;
outer-terminal selection can include sidebar and other UI text. Mouse gestures
must reach Herdr for its selection to apply; enabling capture cannot constrain
selection performed locally by the Android SSH app. Tablet gesture behavior
remains unverified. Keyboard fallback: **Ctrl+B, then [**, move with **h/j/k/l**,
start selection with **Space**, extend it, and copy with **y**.
See [config reference](https://herdr.dev/docs/config-reference/) and
[keyboard copy mode](https://herdr.dev/docs/keyboard/#copy-mode).

## Moving terminal work between workspaces

Local Herdr 0.9.3 help checked on 2026-10-06 exposes no `herdr tab move`
subcommand. The supported CLI workaround for a single-pane tab is
`herdr pane move <pane_id> --new-tab --workspace <workspace_id> --label <name>`.
Discover IDs with `herdr workspace list` and `herdr pane list` on the host/session
that owns the work. This moves a running pane into a new destination tab; it does
not transfer a whole split layout. See the
[CLI reference](https://herdr.dev/docs/cli-reference/).

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

The user clarified that the tablet client is Termius, not Termux. Termius's
[official mobile tips](https://www.linkedin.com/posts/termius_vibecoding-ai-termius-activity-7447815156828450816-gkrd)
describe long-press plus vertical drag as arrow-key input (checked 2026-10-06).
Gesture interception is therefore a plausible explanation for failed Herdr border
dragging, but the tablet's app version and actual input events are unverified.
Do not apply Termux-specific implementation findings to Termius.

Separately, Termux touchscreen behavior was checked against upstream source on 2026-10-06:
long-press starts text selection, and finger movement does not send held-button
mouse drag events. This explains why touch taps can work while Herdr border
dragging fails. Physical mouse input has a separate drag-reporting path; using
an external mouse remains untested on the user's tablet.
See [Termux TerminalView source](https://github.com/termux/termux-app/blob/master/terminal-view/src/main/java/com/termux/view/TerminalView.java).
For the workspace sidebar, Herdr exposes `ui.sidebar_width`,
`ui.sidebar_min_width`, and `ui.sidebar_max_width` in its
[config reference](https://herdr.dev/docs/config-reference/).
These are Herdr UI settings: when SSH is followed by running `herdr` on the VPS,
the Herdr client runs on the VPS even though the Android SSH app renders it.

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

## Test-created workspace pollution (2026-10-12)

Read-only inspection of the VPS default session found 70 workspaces labelled
`topic` and four labelled `supervisor`, each containing one pane whose cwd was a
deleted pytest temporary repository. These are separate workspaces; `fabro`
itself held five panes. A sampled leftover pane had only its retained zsh shell
in the foreground. No leftover workspace or live session was closed.

The matching tests belong to `livespec-overseer`, plan `herdr-overseer`, anchored
by epic `overseer-uzvcbn` (Support herdr and tmux overseer panes with verified
automatic selection). The initial-start implementation is owned by child
`overseer-zbpqnj` (Exact owned-pane initial start and uncertain-launch preservation).
Its public start path now detects the invoking process's terminal through real
ancestry and selects a native Herdr driver. Several legacy CLI tests substitute
only `TmuxIO`; their setup changes cwd and installs a fake Claude executable but
does not substitute the native ownership/router boundary. This provides a mechanism
for tests run from a Herdr-owned process to create real `topic` workspaces despite
the tmux fake, consistent with the observed leftovers. A controlled reproduction
was not run during this investigation. The plan's live-verification research already requires isolated named
sessions and cleanup; ordinary unit-suite isolation must enforce the same
production-session protection. Clearing environment selectors alone is
insufficient because backend selection uses process ancestry.

## Open checks

- Verify Android key authentication and interactive Herdr rendering on the device.
- Verify current saved-machine connectivity before treating historical profile
  listings as current state. The VPS conversation suggested removing a duplicate
  localhost profile but did not record confirmation that it was removed.
- Recheck mobile app capabilities before choosing an app based on these notes.
