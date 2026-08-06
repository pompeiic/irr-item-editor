# Incursion Red River — Item Editor

Online editor for item stats, text, and vendor data. Companion to the in-engine **Item Sync** utility
(`UIRRItemSyncLibrary`, `IIRExtendedEditor` module): Unreal **exports** every `UIRRItemDefinition` asset +
the vendor data table into the JSON files in this repo, this page **edits** them, and Unreal **imports**
them back onto the assets. The game never reads this repo — changes ship with the next build.

- **Editor:** https://pompeiic.github.io/irr-item-editor/editor.html
- **Files:** `Schema.json` (stat labels/units, rarity/faction lists, categories) + one `Items_<Category>.json`
  per item category (Weapons, Attachments, Gear, …). All are produced by the Unreal export — never create
  them by hand. Only vendor-table items are exported. `Workflow.json` is editor-owned (edit history +
  attention flags) and never round-trips into Unreal.

## Workflow

1. **Unreal → repo:** run `Export Items` then `Upload Items To GitHub` (Item Sync utility / any Editor
   Utility Blueprint calling `UIRRItemSyncLibrary`). Repo + token are configured in
   Project Settings → IRR → Item Sync and Editor Preferences → IRR → Item Sync (User).
2. **Edit here:** sidebar views (All items / Recently edited / Needs attention / categories), table list with
   search/filters/sort and a **selectable stat column** (show + sort by any stat), and a per-item panel with
   **Overview** (texts, stats incl. effect stats, prices, rarity, max stack, shared editor **notes**), **Relations** (read-only:
   what the item's containers accept, and what it fits into — generic storage like backpacks/rigs is
   summarized, not listed) and **History** (per-item edit log, starts with the first commit made here).
   **Validate all** flags items with placeholder text (template/tooltip), missing text, bad abbreviations,
   price problems, or **non-canonical caliber spellings** (e.g. `556x45` → `5.56x45mm`; table in
   `editor.html` `CALIBERS`) as "needs attention" (flags can also be set manually per item). Bulk stat
   operations work on the filtered list. **Save changes / Commit** writes the changed category files + the
   workflow file; **Discard** reverts the selected item.
3. **Repo → Unreal:** run `Download Items From GitHub` then `Import Items`. The import report lists every
   applied change (paste-ready for patch notes — the editor's **Copy patch-note draft** button produces the
   same format). Save the dirtied assets.

## Editing rules (enforced by the importer)

- **Items are matched by their identifier tag.** The web editor never creates or deletes items, stats, or
  vendor rows — unknown entries are skipped with a warning on import.
- **Row names and item tags are identity** — not editable here, never renamed by the importer.
- **Names/descriptions**: edits preserve the localization identity (namespace/key) of the text, and
  unchanged text is skipped entirely, so a no-op round-trip dirties nothing.
- **Drift guard:** every entry carries a `sourceHash` of the asset state at export. If someone edited the
  asset in Unreal after the export, the import skips that entry (re-export, or force the import) instead of
  silently reverting their change. Don't touch `sourceHash` when hand-editing JSON.
- **Rarity changes** re-apply the vendor tier defaults (stock/restock/loyalty) on import, exactly like
  changing rarity in-editor.
- **Max stack** imports; the stackable flag and the container-support tags (Relations) are read-only —
  structural setup stays in Unreal.
- **A stat is identified by its tag *and* its display name.** The same tag can mean different things on
  different items (`Inventory.Stats.Durability` is "Armor Points" on armor, "Content" on containers), so each
  meaning gets its own label, unit and sortable column. Display names are read-only here — rename them on the
  stat config asset in Unreal (`Content/Blueprints/InventorySystem/Objects/Stats/U_*.uasset`).
- **Effect stats** (Heal Over Time, Stamina Boost) are the stats whose number lives on the stat's gameplay
  effect rather than on the item. They edit and sort like any other stat, with the effect's **duration** in
  place of the modify-type column.
  - **Heal Over Time** is the **total** healed across the duration, not a per-tick amount — 6 HP over 120s
    is 6 HP in total. The greyed line under the name shows the per-tick rate that works out to.
  - **Stamina Boost** is a **percent** held flat for the duration: `+10` means `RecoverMultiplier 1.1`, and
    the drain multiplier is kept symmetric (`0.9`).

  Which effect fields exist is structural — the web editor never adds or removes effects.
- Price `-1` means "not sold on that market".

## One-time setup: your editor token

Same flow as the patch-notes editor. The **Commit to GitHub** button needs a personal access token — stored
**only in your browser**, only able to touch this repo's files.

1. Be a **collaborator** on this repo (repo → Settings → Collaborators) and accept the invite.
2. github.com → avatar → **Settings** → **Developer settings** → **Personal access tokens** →
   **Tokens (classic)** → **Generate new token (classic)**.
3. Note: `item-editor` · Expiration: 1 year · Scopes: **`public_repo`** only.
4. Copy the `ghp_…` string, paste it into the editor's top-right field, click **Remember**.

The repo **owner** can use a fine-grained token instead (Only select repositories → this repo;
Permissions → Contents: Read and write).

**No token, no problem:** edit here, then hand the JSON to someone who can commit, or edit the files on
github.com directly (pencil icon) — but prefer the editor, it guards the identity fields.

## Rules

- **This is live balance data.** Prefer a PR + review for sweeping changes; the validation workflow only
  rejects malformed JSON, not bad balance.
- Commit, then import in Unreal soon after — the longer web edits sit uncommitted to assets, the more
  drift-guard conflicts you'll hit.
- Never hand-edit `schemaSoftVersion` / `schemaHardVersion` headers or `sourceHash` fields.
