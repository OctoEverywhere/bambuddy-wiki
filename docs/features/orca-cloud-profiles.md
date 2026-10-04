---
title: Orca Cloud Profiles
description: Sync your OrcaSlicer Cloud profiles into Bambuddy for slicing.
---

# Orca Cloud Profiles

Sync your OrcaSlicer 2.4.0+ cloud-synced filament / process / printer
profiles into Bambuddy and use them alongside your Bambu Cloud, locally
imported, and standard bundled presets.

!!! info "Companion to Bambu Cloud Profiles"
    Orca Cloud profiles **don't replace** [Bambu Cloud Profiles](cloud-profiles.md) — both tabs live side by side in the Profiles page and you can connect to either, both, or neither. Profile precedence in the slicer is **Imported > Orca Cloud > Bambu Cloud > Standard**, so a locally-imported preset wins over the same name in either cloud, and an Orca Cloud preset wins over a Bambu Cloud one with the same name.

---

## :material-cloud: Overview

Orca Cloud Profiles lets you:

- **Connect** your Orca Cloud account to Bambuddy by approving a pairing code
- **View** all profiles your Orca account has synced — filament, process, printer
- **Slice** with Orca profiles directly from the Bambuddy SliceModal
- **Assign** an Orca filament profile to an AMS slot from the Configure modal
- **Spool inventory**: pick Orca filaments when creating or editing a spool, including from a SpoolBuddy kiosk

The integration is read-only on the Orca Cloud side — Bambuddy lists and uses your profiles but doesn't create, edit, or delete them. Make changes in OrcaSlicer itself; refresh the tab in Bambuddy to pick them up.

!!! note "Only your own presets are in Orca Cloud"
    OrcaSlicer uploads presets you created or saved yourself. Ticking a built-in filament, such as *Elegoo PLA* or *PolyLite PLA*, in OrcaSlicer's filament list only shows it in your own OrcaSlicer, so it never reaches Orca Cloud or this tab. You don't need it there: with the OrcaSlicer sidecar, Bambuddy lists OrcaSlicer's built-in filaments in the **Standard** tier of the Slice dialog. To use your own changes to one, select it in OrcaSlicer and **Save as** a preset of your own; that copy syncs.

---

## :material-key: Connecting

### Pairing with Orca Cloud

Bambuddy connects to Orca Cloud by **approving a pairing code**. There is no password to enter in Bambuddy and no redirect URL to copy.

1. Open **Profiles** → **Orca Cloud** tab
2. Click **Connect Orca Cloud**
3. Bambuddy shows a short **pairing code**. Click **Open Orca Cloud approval page**, or go to the address shown under the code and enter it there
4. Sign in to Orca Cloud in that tab if asked, and approve the code
5. Bambuddy connects on its own as soon as you approve. The dialog shows *Waiting for you to approve…* until then

The code expires after 10 minutes. If it does, or if the pairing was denied, click **Connect Orca Cloud** again for a new one.

Bambuddy asks for **read-only** access: it lists and views your profiles and nothing more. The pairing stays valid without signing in again, and it works the same whether you reach Bambuddy by LAN IP, `localhost` or through a reverse proxy. **Disconnect** on the tab ends it.

### After signing in

Once connected, the Orca Cloud tab shows your synced profiles grouped into **Filament / Process / Printer** columns, with the same filter dropdowns and detail view as the Bambu Cloud tab.

---

## :material-permission: Permissions

| Permission | Required for |
|------------|--------------|
| `orca_cloud:auth` | Sign in / sign out / read profiles / slice with Orca presets |

The permission is granted to the **Administrators** group by default. Add it to other groups in **Settings → Users & Groups → Groups** if you want non-admin users to access Orca Cloud.

!!! tip "API keys"
    To let an API key read Orca Cloud presets on its owner's behalf (useful for headless slicing pipelines), enable **Allow cloud access** on the key. The same scope covers both Bambu Cloud and Orca Cloud — same trust dimension, one toggle.

---

## :material-scissors-cutting: Slicing with Orca profiles

The **SliceModal** preset dropdowns surface profiles from all four tiers in priority order:

1. **Imported** — local OrcaSlicer / Bambu Studio profile imports (highest; ship with parsed type / colour metadata)
2. **Orca Cloud** (this tab) — synced OrcaSlicer Cloud profiles
3. **Bambu Cloud** — your Bambu Cloud account
4. **Standard** — the slicer's stock bundled profiles (unconditional fallback)

If a profile of the same name exists in multiple tiers (rare in practice), the higher-priority one wins. Status banners at the top of the modal flag each cloud independently — you'll see a *"Sign in to Orca Cloud"* hint if you're connected to Bambu Cloud but not Orca, and vice versa.

For **multi-color** prints, the per-plate filament pre-pick benefits from Orca Cloud's inline `filament_type` and `default_filament_colour` metadata (no per-preset lookup needed), giving the matcher more to work with than Bambu Cloud presets where this metadata isn't surfaced in the list response &mdash; but a locally-imported preset of the same name still wins on tier priority above the cloud match.

---

## :material-printer-3d-nozzle: AMS slot assignment

The **Configure AMS Slot** dialog (both from the printer card and from SpoolBuddy) cannot send Orca's own profile ID to the printer. That ID is a 36-character UUID, and the printer's filament field holds eight characters. Instead, Bambuddy reads the profile's own `filament_id` from its slicer JSON and puts that in the slot's `tray_info_idx`. This is the eight-character `P…` ID that OrcaSlicer's Device tab also writes, and it is the only thing OrcaSlicer's **Sync filaments** uses to find your profile again. The lookup runs on the Bambuddy server when you configure the slot, and its result is written to the log.

A profile that only stores its changes against another profile has no `filament_id` of its own. Bambuddy then takes the ID from the profile it inherits from: either another of your Orca Cloud profiles, or the Bambu filament it was copied from (for example `GFA00` for *Bambu PLA Basic*). If no ID can be found, or Orca Cloud can't be reached, the slot gets the generic Bambu ID for the material instead (PLA → GFL99, PETG → GFG99, ABS → GFB99, and so on). The dialog then shows a warning that OrcaSlicer will see the slot as *Generic*. See [Why the printer may still show a generic filament](ams.md#custom-preset-display). Bambuddy remembers the profile you picked for the slot, so it can show the right name on hover and preselect it the next time you open the dialog. Once the slot is changed somewhere else, such as OrcaSlicer's Device tab or the printer's screen, Bambuddy stops showing that profile.

---

## :material-package-variant: Inventory and SpoolBuddy

Orca filament profiles appear in the filament-preset picker of:

- **Spool form** (Settings → Inventory → Add Spool / Edit Spool)
- **SpoolBuddy kiosk** Write-Tag page
- **SpoolBuddy AMS Configure** flow

Both clouds are fetched in parallel; either or both can be connected. If you only connect Orca, the picker simply shows Orca filaments where it would otherwise show Bambu Cloud ones.

---

## :material-debug-step-over: Troubleshooting

### "The Orca Cloud sign-in flow expired after 10 minutes"

The PKCE handshake state is held server-side for 10 minutes. If you took longer than that between clicking Connect and pasting back the callback URL (or the kiosk was reloaded mid-flow), click Connect again to start fresh — the new flow generates a fresh verifier.

### Profiles tab shows "Connected" but the list is empty

Your Orca account has no profiles synced yet, or you signed in with an account that doesn't have OrcaSlicer profile sync enabled. Open OrcaSlicer, enable cloud sync from Preferences, sync at least one filament profile, then click Refresh on the Bambuddy tab.

### The Configure AMS Slot picker doesn't show my Orca profile

If you're using an API key (e.g., from SpoolBuddy), check the key has **Allow cloud access** enabled — the same scope gates both Bambu Cloud and Orca Cloud reads. Otherwise the kiosk's request is silently rejected and no Orca profiles appear.

### Sign-in works, but the slicer doesn't see my Orca filament's metadata

Orca's filament profiles include `filament_type` and `default_filament_colour` inline, but only after the first sync round-trip. If you added a brand-new filament profile in OrcaSlicer and immediately tried to slice with it in Bambuddy, hit **Refresh** on the Orca Cloud tab to re-pull the latest snapshot.

---

## :material-link-variant: Related

- [Bambu Cloud Profiles](cloud-profiles.md) — the Bambu side of the dual-cloud setup
- [Local Profiles](local-profiles.md) — for offline / non-Orca / non-Bambu profile sources
- [Server-Side Slicing](slicer-api.md) — how preset tiers feed the slicer
- [API Keys & Webhooks](api-keys.md) — to expose cloud presets to headless callers
