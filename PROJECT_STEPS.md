# Project implementation checklist

Work through these phases in order. The [launcher design](DESIGN.md) defines the expected behavior and the publishing and update protocol.

## 1. Set up the fork and test environment

- [x] Import PineconeMC `11.1.2` at commit `ac99ad44ef98dd73dd9626c7b2b668dd8cbd5c` with upstream history, license, and notices. The local `upstream` remote is configured and the pinned `libnbtplusplus` submodule is initialized.
- [ ] Build the baseline on Windows and the Mac architectures used by the group. Give the fork its own application identifier and credential-store service name.
- [ ] Set the default executable and data paths to the per-user locations in the design. Audit instance, runtime, cache, settings, and backup paths inherited from upstream so they resolve under the fork's own data directory.
- [ ] Choose the latest stable Minecraft Java release that works with Fabric and the required mods. Pin the game, loader, and mod versions in a small test pack. Prepare a test server on the group's private network.
- [ ] Verify that a username-only profile can join the test server. Check that optional Microsoft and Ely.by sign-in still work through their respective upstream account flows, and that the launcher explains an incompatible server login mode.

## 2. Build the pack format and local installer

- [ ] Specify the versioned invite, current-release record, pack manifest, and local subscription record. Define exact signing and encryption formats and keep a small fixture pack for tests.
- [ ] Add an export preview to the instance editor. Include pinned mods and dependencies, selected config paths, recommended RAM, and the server entry. Show added, changed, and removed files and flag assets that cannot be shared or downloaded automatically.
- [ ] Install the fixture from a local bundle using the upstream game, Java, and loader installers. Verify file integrity and validate archive paths and size limits before writing into an instance.
- [ ] Add managed-file tracking, preservation of overwritten local configs, and a clear choice for handling extra local mods. Seed the Multiplayer entry and implement a per-player RAM override.
- [ ] Test that the clean fixture launches and that an updated fixture replaces or removes only the intended files. Verify that worlds, keybinds, video settings, and other server entries survive.

## 3. Add GitHub publishing and subscriptions

- [ ] Create a dedicated distribution repository for the pack. Configure the admin's repository-scoped GitHub credential and store it and the publishing keys in the system credential store. Test recovery of the signing and pack keys.
- [ ] Implement **Publish update**. Build and encrypt a snapshot, upload and verify the Release asset, then update the signed current-release record. Handle upload retries, pointer conflicts, and timeouts without exposing a partial release.
- [ ] Add **Subscribe to instance**. Validate the invite and the signed pack record, show the trusted publisher and pack details, and create the instance without a GitHub account on the player's side.
- [ ] Test duplicate subscriptions, invalid invites, and key rotation. Verify that a replacement code reconnects the same instance and that the old code cannot open releases encrypted with the new key.

## 4. Wire updates into Play

- [ ] Check for a signed update when the player clicks **Play**. Track verified and installed revisions separately, retry a verified release after a failed install, and reject stale revisions before applying changes.
- [ ] Stage verified files, take a per-instance lock, and apply changes once Minecraft is closed. Use the transaction journal to finish or recover a partially applied update after a restart.
- [ ] Show install progress, **Retry**, and **Play installed version** when a usable copy exists. Distinguish a broken game launch from a failed update, and offer the previous usable managed snapshot when available.
- [ ] Add **Unsubscribe** with an option to keep the local instance. Verify that RAM overrides and personal files survive updates and rollback.

## 5. Add admin backup and restore

- [ ] Add **Pack history** and **Mark working**. Retain the latest confirmed working snapshot and its release assets across unsuccessful or unconfirmed publishes.
- [ ] Implement **Open backup instance** to recreate a separate local test copy. Implement **Restore backup for everyone** to publish the confirmed snapshot's contents as a higher revision under the active key.
- [ ] Test rollback from both a client that installed the bad release and a client that skipped it. Do a separate server-world backup and recovery rehearsal if testing changes to the game version or server mods.

## 6. Package, test, and hand off

- [ ] Package a per-user Windows installer and macOS app for the group's machines. Verify install, upgrade, and uninstall alongside existing Minecraft and launcher installations. Check that each fresh install uses the locations in the design without requesting administrator privileges.
- [ ] Test the full flow with one Windows player and one Mac player. Each should be able to install the launcher, paste a code, choose a username, and join the same server. Publish an update and restore the working backup to confirm both clients follow it.
- [ ] Exercise a failed upload, network outage, rate limit, corrupted archive, stale response, full disk, interrupted install, and an update requested while Minecraft is open. Check that retries work and that diagnostic logs omit secrets and invite contents.
- [ ] Write short player instructions for first open, subscription, and **Play**. Write short admin instructions for publishing, coordinating server changes, rotating a leaked invite, and restoring the working backup. Publish the launcher builds and share the invite with the group once those checks pass.
