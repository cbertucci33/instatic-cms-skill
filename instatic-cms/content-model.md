# Content Model

## Tables (`data_tables`)
Five kinds (source union `DataTableKindSchema`):
- `postType`: authored in the Content workspace. Editorial states: draft, published, unpublished, scheduled. Published copies keep version history. Built-in fields: title, slug, body, featuredMedia, seoTitle, seoDescription; built-ins are enable/disable only, never rename or delete. Custom fields are added per table.
- `data`: plain spreadsheet grid, no editorial workflow. Good for teams, testimonials, product rows, form submissions.
- `page`, `component`, `layout`: system tables behind the editor's page tree, visual components, and saved layouts. Read-only via `GET /pages?id=`, `/components?id=`, `/layouts?id=`; edited through the canvas or agent.

## Fields (16 types, source union `DataFieldSchema`)
text, longText, richText, number, boolean, date, dateTime, select, multiSelect, url, email, media, relation, repeater, pageTree, fieldSchema.
- richText carries `format: markdown | html`.
- number carries `format: number | currency | percent` plus optional ISO-4217 `currency` code.
- media carries `mediaKind: image | video | any`.
- select stores one option id; multiSelect an option id array; relation references rows in another table; repeater holds nested item fields.
- There is no icon-picker field control at v0.0.20. Model a per-card icon as a media field or a select whose option labels name the icon.

## Row values
`cells_json[fieldId]`: strings for text kinds, numbers, booleans, ISO strings for date kinds, option ids for selects. Write through the rows API or UI, never raw SQL.

## Reuse mechanisms (three, distinct)
1. **Templates** (page wrappers). A `pages` row with target `everywhere`, `{postTypes, tableSlugs}`, or `notFound`, plus priority. The router resolves the chain outer to inner and splices inner trees into the template's single `base.outlet`. Templates are never served at their own slug. Use for: site chrome, per-type page frames, designed 404.
2. **Visual Components** (element templates). Reusable subtree with typed params (string, number, boolean, url, enum, color, image, richText) plus named slots. Slot model: the `base.slot-outlet` node IS the slot, no separate declaration. Instances drop onto any canvas as refs; the publisher inlines them, zero runtime cost. Editing the definition propagates everywhere. Use for: cards, CTA blocks, tiles.
3. **Saved Layouts** (subtree snapshots). A named page subtree plus the style rules its nodes referenced at capture. Inserting uses the paste engine: fresh node ids, scoped classes cloned and remapped. Use for: section patterns repeated without parameters.

Pick by job: recurring content with per-instance fields wants a VC; wrapping chrome wants a Template; paste-around blocks want a Saved Layout.

## Loops
`base.loop` renders a collection (published rows of a table, media, or plugin-provided sources) across its body variants round-robin. The canvas feeds preview rows from `GET /data/tables/:id/loop-preview`. Loop bodies auto-classify as request-dependent at publish time; see `publishing.md` holes.

## Forms
Native form primitives compose on the canvas; submissions land in data tables (auto-created field-matched table available). Public endpoints: `/_instatic/form/submit` plus `/_instatic/form/challenge`.
