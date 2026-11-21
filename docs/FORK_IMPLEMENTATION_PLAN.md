# OpenWhispr fork implementation plan

## Goal

Maintain a personal fork of OpenWhispr, adding requested capabilities while preserving the upstream user interface and existing behavior. Keep upstream changes mergeable and release independently from upstream.

The initial integration target is the EveryApp Gateway and its already-deployed Scribe app. Treat meeting transcript export as the first feature slice. Keep local dictation and on-device transcription usable without an EveryApp account.

## Current baseline

- Upstream repository: https://github.com/OpenWhispr/openwhispr
- Fork target: https://github.com/seankrux/openwhispr
- Audited upstream revision: `21103c76d01eb6324de06a625816b91fc8f8c592`
- License: MIT. Preserve the upstream copyright and license notices.
- Runtime: Node.js 24 or later. Electron, React, TypeScript, native helpers and platform-specific packaging.
- Existing repository already contains global dictation, local/cloud transcription, meeting recording, notes, Calendar OAuth and a custom ASR shim. Extend those surfaces instead of replacing them.
- The existing EveryApp Scribe deployment is reachable at `https://scribe.app.seanguillermo.com`. Its OAuth metadata points to `https://app.seanguillermo.com`. Unauthenticated `/api/v1/me` returns HTTP 401.
- A 401 response confirms the route is protected. It does not verify a complete native sign-in or meeting upload.

## Product and interface constraints

- Retain the existing screens, layout, component patterns, copy style and theme.
- Add actions within existing meeting, note, account and settings screens.
- Avoid broad redesigns, new navigation structures and unrelated branding edits.
- Keep existing upstream features operational.
- Local recording and local transcription are the default privacy path.
- Cloud upload is explicit per meeting and visibly reports success or failure.
- Do not upload raw audio for the initial Scribe integration. Upload transcript text only.
- EveryApp credentials, OAuth tokens and provider keys remain in the operating system credential store or Gateway secrets. Never commit secrets or put them in renderer code.

## Delivery phases

### Phase 0 — Establish the fork baseline

1. Keep upstream as the `upstream` Git remote and the account fork as `origin`.
2. Record the base revision and preserve the upstream MIT license.
3. Run the existing install, lint, typecheck and test workflows in a clean Node 24 environment.
4. Capture reference screenshots of onboarding, settings, dictation, meeting recording and notes.
5. Smoke-test current flows on each release platform before changing behavior.
6. Write down any baseline failures so fork regressions can be distinguished from inherited issues.

**Exit check:** reproducible local setup, baseline CI results, screenshots, and no working-tree changes outside planned work.

### Phase 1 — Fork identity and update channel

1. Choose a fork display name, bundle identifier, application protocol, website and support address.
2. Update Electron packaging metadata and build signing identifiers for the fork.
3. Point release publishing and the update feed at the account repository.
4. Make development, staging and production callback protocols distinct.
5. Confirm the fork can be installed alongside upstream and updates only from its own releases.

**Exit check:** macOS, Windows and Linux artifacts build with fork identities; installation and update checks cannot target upstream releases.

### Phase 2 — EveryApp sign-in

1. Add an account option in the current account/settings surface.
2. Discover the app origin and Gateway authorization metadata using the documented EveryApp native OAuth flow.
3. Use public-client registration, loopback redirect, PKCE, state and issuer validation.
4. Store and rotate refresh tokens in the operating system credential store.
5. Scope access to the Scribe app origin and show which Gateway account is connected.
6. Support disconnect, token revocation and reauthentication.
7. Leave local dictation and recording available when signed out or offline.

**Exit check:** sign in, restart, refresh, revoke and sign out against the real Gateway; invalid or expired credentials never expose transcripts.

### Phase 3 — Save transcript to Scribe

1. Add a clearly labeled save action to the existing meeting notes view.
2. Create the Scribe meeting with title, time and available attendee metadata.
3. Upload transcript segments in sequence, preserving timestamps and supported speaker labels.
4. Call the finish endpoint to ask Scribe to generate notes.
5. Handle network interruption and retries without duplicate meetings or duplicate segments.
6. Show a useful status, retry action and destination link in the existing UI.
7. Keep a local copy regardless of remote upload outcome.
8. Verify private-by-default visibility and confirm no audio is sent.

**Exit check:** a real test meeting appears once in Scribe with complete transcript and generated notes; retry, offline and signed-out cases behave safely.

### Phase 4 — Calendar integration

1. Use Scribe's existing Gateway-backed Calendar capability for meeting reminders where its API covers the needed data.
2. Keep the existing OpenWhispr Google Calendar connection available until feature parity is demonstrated.
3. Add any replacement within the existing Calendar settings and reminder UI.
4. Validate permissions, refresh behavior, disconnect behavior and calendar selection.
5. Avoid treating the desktop callback page URL as an OAuth server. It only displays the result of the desktop OAuth flow.

**Exit check:** reminders, event names and attendees work with the intended Google account; existing users can disconnect cleanly.

### Phase 5 — Security, compatibility and release

1. Review token storage, OAuth state/PKCE handling, IPC boundaries, log redaction and data deletion behavior.
2. Test with real EveryApp OAuth and Scribe APIs using non-sensitive test meetings.
3. Run repository lint, typecheck, tests and packaging for supported operating systems.
4. Compare screenshots and existing flows against Phase 0 references.
5. Publish a prerelease to the fork, install it on a clean machine and verify updates.
6. Document install, account setup, privacy choices, troubleshooting and upstream synchronization.

**Exit check:** release candidate passes automated checks, manual privacy checks and install/update smoke tests on declared platforms.

## Work breakdown

| Workstream | Main areas | Depends on |
| --- | --- | --- |
| Fork identity and updater | `electron-builder.json`, release workflows, app protocol, update configuration | Phase 0 |
| EveryApp OAuth | `src/lib/auth.ts`, Electron main-process auth, secure credential storage | Phase 1 |
| Scribe API client | meeting session flow, typed API client, retry state and existing notes UI | Phase 2 |
| Calendar | existing Calendar manager, OAuth settings, reminder scheduler | Phase 2 and capability parity review |
| Release quality | CI, platform builds, privacy and regression checks, user docs | All implementation phases |

## Operating workflow

- Make one focused pull request per phase or independently testable feature slice.
- Keep patches small enough to rebase on upstream.
- Run the checks relevant to each changed area before merge.
- Test privacy-sensitive changes against explicit acceptance cases.
- Sync upstream regularly after reviewing conflicts in fork-specific identity, auth and release files.
- Do not ship a feature whose fallback, retry and disconnect behavior is undefined.

## Open decisions

1. Confirm fork display name, bundle identifier, protocol scheme, website and support address.
2. Confirm transcript export is the first new feature to ship.
3. Confirm the supported release platforms for the first fork release.
4. Decide whether Calendar should use Scribe's shared Gateway Google connection after a parity test, or retain OpenWhispr's own Calendar OAuth initially.
