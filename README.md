# Incursion Red River — Item Editor

Online editor for item stats, text, and vendor data. Companion to the in-engine **Item Sync** utility
(`UIRRItemSyncLibrary`, `IIRExtendedEditor` module): Unreal **exports** every `UIRRItemDefinition` asset +
the vendor data table into the JSON files in this repo, this page **edits** them, and Unreal **imports**
them back onto the assets. The game never reads this repo — changes ship with the next build.

- **Editor:** https://pompeiic.github.io/irr-item-editor/editor.html
- **Files:** `Schema.json` (stat labels/units, rarity/faction lists, categories) + one `Items_<Category>.json`
  per item category (Weapons, Attachments, Gear, …). All are produced by the Unreal export — never create
  them by hand.

## Workflow

1. **Unreal → repo:** run `Export Items` then `Upload Items To GitHub` (Item Sync utility / any Editor
   Utility Blueprint calling `UIRRItemSyncLibrary`). Repo + token are configured in
   Project Settings → IRR → Item Sync and Editor Preferences → IRR → Item Sync (User).
2. **Edit here:** search/filter items, edit stats, prices, rarity, names; bulk operations
   ("+10% Damage on everything filtered"); the right column shows every pending change as `old → new`.
   **Commit to GitHub** writes the changed category files (token top-right, see below).
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
