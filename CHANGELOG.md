# Changelog

Each entry is a tag in this repository, named for the Zindex release that first served that version of the spec and schemas. Documents in every version so far carry `"schemaVersion": "0.1"`.

## v9.23.0

Changed: the reference engine no longer moves the drawing when a line or label reaches left of or above 0 (3.2, 13). Nothing is offset: every element is drawn where its coordinates put it, negative ones included, and the canvas starts where the line or label reaches. Before, it moved everything drawn right and down by that overshoot and grew the canvas by the same amount. A stored document and its `canvas` don't change. No input field or value changed.

Changed: when the reference engine lays out every node of a flowchart, workflow, UI flow, architecture, entity-relationship or no-family diagram, it opens a gap between two ranks wider where a label that stands on its own line would hide the ends of its line or lie against a line that crosses it (3.4). The spacings stay targets, not minimums. Before, the gap kept the spacing the layout chose, and such a label could cover most of a short line. A document that keeps its nodes' positions is laid out as before. No input field or value changed.

Changed: when the reference engine lays out every node of an entity-relationship diagram that flows left to right or right to left, it moves tables until two rows a relationship joins are level where they lie less than 16 px apart and the tables have the room (6.4). A table with a `layout` of its own never moves. Before, the line between such rows made a step of 8 to 10 px. No input field or value changed.

## v9.22.0

Added: the reference engine's warning `ACCENT_COLOR_NOT_DRAWN`, for a `style.accentColor` on an element that draws nothing with it (3.5). It returns one warning for each kind of element: a node with no accent mark, an `er.entity` or `er.weakEntity` with no `extensions.columns`, an edge, a text, an image, a guide, a group and a Google Cloud zone. Before, such an `accentColor` changed nothing and returned no warning.

Changed: a frame's `accentColor` colors its border and its title (3.5). The border is drawn as `stroke` would draw it, and the title in a shade of the accent that reaches 4.5:1 where the title sits. `stroke` and `textColor` override each on their own, and the accent adds no wash: `fill` tints the frame. A pool, a fragment, an `aws.*` group, a frame with an `aws:*` icon and no preset, and a `gcp.*` group other than a zone keep their own title color, the border taking the accent, and a Google Cloud zone's strips ignore it. Before, `accentColor` drew nothing on a frame. No input field or value changed.

Changed: in a sequence diagram, the reference engine reads an edge between two participants whose `edgeType` is absent, `default` or another family's as a message (5.9). Its line says which kind: a `sequence.reply` when `style.dash` gives it a pattern, whatever its arrowheads; a `sequence.async` when it is solid, its `endArrow` is `openTriangle` and its `startArrow` is `none` or absent; otherwise a `sequence.message`. Only `style.dash` and `endpoints` count, not an `edgeType`'s own dash or arrowheads. The reference engine works it out each time it draws the diagram and doesn't change the stored edge, and it never reads a create or a destroy this way. An edge with an end that isn't a participant is no message. Before, only an edge whose `edgeType` starts with `sequence.` was a message. No input field or value changed.

Changed: a `sequence.note` with no `extensions.anchor` is drawn over the lifelines of the message above it (8). A note above every message goes over the first message below it, and a note in a diagram with no message goes across every lifeline. With `side` and no anchor, the note sits beside the leftmost (`left`) or rightmost (`right`) lifeline of that message. The reference engine warns (`FAMILY_SEQ_NOTE_INVALID_ANCHOR`) and doesn't change the stored note. A note whose anchor names no participant that exists is still not drawn. Before, a note with no anchor was not drawn. No input field or value changed.

Changed: when the reference engine lays out every node, it moves the whole drawing right or down, by the least distance that leaves `nodeSpacing` between it and a text, an image or a frame with its own absolute `layout` that it would cover (6.4). A frame's own members, and a pool, a lane, a fragment or a cloud group the cloud layout arranges, don't count. Before, the nodes were drawn over it. A sequence diagram, a diagram with a pool or lane that has a `layout` of its own, and nodes that have positions aren't moved this way. No input field or value changed.

## v9.19.2

Changed: in a Google Cloud diagram the reference engine arranges, a load balancer over the gap between two Regions keeps its place when lines from outside the boundary drop through its row (5.5). It feeds the first tier of each of two neighboring Regions, and `gcp.cloud` lists it or the project that lists it holds both Regions. An item of its row that such a line crosses stands in the free place nearest to where it stood, beside the load balancer and 16 px clear of every line. Where none is free, it stands over the load balancer, `layoutStrategy.nodeSpacing` clear (30 px by default), or the room of a label on the line between them if that is more. Two items that `gcp.cloud` lists in the row over the Regions never touch on a new layout. A saved diagram keeps the gap it was saved with, touching items included, until `autoLayout`. An edit that moves a load balancer onto an item of its row that stays stands it under that item. Before, the whole row stood aside from the line, which could leave the load balancer and an item of its row touching. No input field or value changed.

Changed: in a stack of `layer` frames, the reference engine keeps room for a layer's title (5.5). In a stack of rows, each layer keeps room left of its top row's first card, so a line into that card from above clears the title by at least 16 px. A long title widens every layer, since the layers share one width. A stack of columns keeps no such room. A saved stack whose top row's first card stands nearer a layer's left edge than the title needs is drawn with its layers wider to the left, on any draw. No card moves. No input field or value changed.

Changed: in a diagram with pinned cards (cards with a `layout` of their own), the reference engine arranges a stack of `layer` frames none of whose cards has a `layout` as a new layout arranges it, and the pinned cards outside the stack stay where they are (5.5). A first save, or an edit that creates the stack's cards, is such a case. The stack stands `layoutStrategy.nodeSpacing` clear of the pinned cards, or 56 px if that is more. It stands after the pinned cards with more lines into it than out of it. With none of those, it stands before the other pinned cards it has lines with, and otherwise past them all. A `layer` with a `layout` of its own divides the stack: the layers listed before it stand before it, and the layers listed after it stand after it. These cards were placed next to their neighbors before, which could leave them in each other's frames. No input field or value changed.

Changed: in a stack of `layer` frames, a line between two cards of the stack stays inside the stack (5.5). It stays at least 16 px inside the stack's left and right borders, or its top and bottom borders in a stack of columns. A layer that such a line passes through without ending in it keeps at least 32 px between the frame fitted round its cards and the stack's right border, or its bottom border in a stack of columns. Every layer widens to give it, unless another layer already reaches that far. A line with a card outside the stack at either end isn't held between the borders. A saved stack with such a line is drawn with its layers wider on the right, or at the bottom in a stack of columns, on any draw. No card moves. Before, such a line could run down the outside of the stack's frames. No input field or value changed.

## v9.18.0

Changed: in a Google Cloud diagram the reference engine arranges, a line from an item outside the boundary, or from a global item over the Regions, into a tier past another tier of its Region takes a lane (5.5). The lane is a gap between the zone columns, else a lane beside the tiers on their right, and the line lands on the top of what it enters. An item takes lanes only when all its lines enter a tier, card or node past another tier of one Region and none is drawn curved. A line to another item outside the boundary doesn't count. Each item takes one gap, and only a top-to-bottom flow, or a bottom-to-top one for an item a top-to-bottom flow would stand below the boundary, gives a lane. Any other line is drawn as before. An edit that adds such an item with its line opens the lane, and the item stands over it where its row has room. A line an edit adds that the lane doesn't serve takes the item's saved lanes away, and no saved item moves to stand over a lane. No input field or value changed.

## v9.17.0

Changed: a Google Cloud Region keeps 24 px under the lowest card, row of items or subnet it holds, 21 px at its sides, and 24 px from the subnets under a VPC band (5.5). Regions that stand level share one bottom. A diagram saved on an earlier version is drawn with a Region that stood closer lowered to that, and the rows under the Regions move down with it. No node inside a Region moves, and the next edit saves the rows where they stand. No input field or value changed.

Changed: a labeled line that runs straight between matching rows of two neighboring Google Cloud Regions opens the gap between them to the label's width plus its room (5.5). The room is 24 px or more, and the gap stays as it is when the label already fits on a stretch of the line. The gap doesn't open for a line between different rows, past a card, past a Region or into a Region with no subnet. An edit that labels such a line moves the second Region, and what lies right of it, right by what the gap opens. No input field or value changed.

Changed: a line that drops through a project's own row, or the row `gcp.cloud` lists, makes the row stand aside (5.5). The items in its way move 16 px past it, and the box grows when the row then runs past it. A line added to a saved diagram moves no saved card. The router draws it round the row's cards, and `autoLayout` stands the row aside. No input field or value changed.

Changed: a project's title stands at its top center (5.5). It moves to its top left when a line between an item above it and a card under it would run down closer than 16 px to it and the top left is clear of such lines. A project with an `icon`, or no wider than its title needs, keeps its title where it is. Every layout decides it again from the places of the cards. No input field or value changed.

Changed: a dashboard service (a node with the `gcp:looker`, `gcp:looker-studio` or `gcp:looker-studio-pro` icon, or the alias `gcp:data-studio`) that the boundary or a project lists stands below the Regions (5.5). It does so when two things hold: the lines the Regions send it equal its lines into the Regions plus the calls from outside, and every card that feeds it ends its Region's flow. A user whose every line with the boundary reaches a dashboard service there stands below the boundary under it. A user with a line to anything else in the boundary stays above. No input field or value changed.

Changed: in a Google Cloud diagram the reference engine arranges, an edge label keeps a floor (5.2, 5.5). It stays 10 px from every other line and from every card but its edge's ends, at least twice as near its own line as any other, and out of a frame's title row. A label whose place breaks that wraps narrower, in at most three lines, where a narrower place keeps it. Other diagrams place labels as before. No input field or value changed.

## v9.14.0

Added: the reference engine's warning `EDGE_OFFSET_ON_GROWN_NODE`, for an edge end that pins an `offset` on a side of a node drawn bigger than its `layout` says (5.3, 6.5). An `offset` is a fraction of the side as drawn.

Changed: the reference engine stores the size a placed node is drawn at (6.5). A node whose label or icon needs more room than its stored `width` or `height` was already drawn larger; a create or an edit now stores that size, so the stored `layout` says what is drawn. A growth of 1 px or less isn't stored. A node whose edges dock on a smaller box than it draws (a workflow event or gateway, a UI flow entry or exit point, a node that draws its icon alone, or a card a `gcp.zone` spans) is the exception. It stores a drawn size only where the reference engine places or moves it, an edit measures it again, or its growth pushes a neighbor or a frame border. A Google Cloud card drawn as a stack, sequence nodes and relatively placed nodes store none. A read, a render, a check and `restoreRevision` store no size. No input field or value changed.

Changed: when an edit changes what a placed node shows, the reference engine measures the node again only on an axis where its stored size was what the old content needed, within 1 px (6.5). A size bigger than the old content needed stays when a label gets shorter, and a longer label still draws the node larger. A workflow event or gateway and a UI flow entry or exit point are measured on both axes, as before. No input field or value changed.

## v9.13.0

Changed: an `anchor` with a `side` and no `offset` leaves the place on that side to the reference engine (5.3). Where the two sides face each other and their faces overlap it lines the ends up so the line is straight, it spreads ends that share the side, and it slides a straight line past the title of a frame it enters, as it does for an edge with no anchor. An `offset` still places the end. No input field or value changed.

Changed: in a top-to-bottom Google Cloud diagram, a line that ends on a generic frame round cards, from an item above it, lands on the frame's top 16 px clear of the frame's title where it can drop straight under the item. Where the title leaves no such place, or the item has other lines down, it lands where the router puts it. The lane over the frame's cards for a second line runs 16 px under the title (5.5). No input field or value changed.

Changed: the reference engine keeps a stored stack of `layer` frames stacked when an edit changes it (5.5, 6.4, 6.5, 10.6). A card an edit adds to a layer, or moves into one, stands in that layer's row; a layer that grows, or is added, moves the layers after it along by the growth; a card that grows keeps the edge of its row nearest the start of the stack. `autoLayout` with `elements` places a card of a stack in its layer's row (10.6). A card added to a stored stack was placed by the layout's general rules before. No input field or value changed.

Changed: the reference engine warns about an untyped group (`GCP_GROUP_UNTYPED`, `AWS_GROUP_UNTYPED`, 5.5) only when the frame's title is a group's own name, alone or with one identifier (a letter, or a word with a digit), such as `VPC network` or `Region: us-east1`. A title that only contains such a word, such as `VPC Service Controls perimeter`, no longer gets the warning, and the warning's fix offers `or leave it generic`. In an AWS diagram a bare or identified `Subnet` names both subnet presets. No input field or value changed.

## v9.12.0

Added: the `layer` container type (5.5). In an `architecture` document, or one without a family, the reference engine stacks the `layer` frames one frame lists, or at the root those no frame lists, as rows in the order listed, each card in its layer's row, the layers one width, with the codes `LAYER_NOT_ARRANGED` and `LAYER_HAS_LAYOUT` for a layer it draws as a generic frame. Documents that don't write `layer` draw as before.

## v9.11.1

Changed: in a top-to-bottom Google Cloud diagram, a generic frame round cards, such as a GKE cluster or a service perimeter, keeps a lane beside or under its cards for a line from one of its cards to a card outside the frame when another card of the frame is in the line's way (5.5). The line runs in the lane clear of the cards and the frame's border. The lane opens in a new layout and after an `autoLayout` without `elements`; a saved diagram keeps its frames and cards as they are. No input field or value changed.

## v9.11.0

Changed: the reference engine wraps an edge label narrower, in at most three lines (or the lines written, if more), where it finds no clean place for it beside its line (5.2). A label that has a clean place keeps its lines. `textStyle.wrap: "none"` keeps a label as written wherever it goes. A label that only crowds another stays within 14 px of its own line; one that covers another label is still moved off it.

## v9.10.0

Changed: in a Google Cloud diagram, a project is one box round its Regions and the global items it lists itself, and the boxes of sibling projects don't overlap (5.5). A project's items stand over or under its own Regions. A row of them wider than the project's Regions widens the box, and the project after it moves right by the growth. A diagram saved on an earlier version keeps its places, unless an edit needs room. Its next edit also clears sibling projects' boxes saved overlapping: the project listed later, and what lies right of it, move right.

Changed: the cards a Google Cloud Region lists directly stand in rows by flow when one row would run wider than 1,400 px (5.5). Each card stands one row below the lowest card with an edge into it, and a row wider than 1,400 px wraps onto a further row. The cards of a row stand in an order chosen to cut the crossings between rows, not always to the fewest. Cards that fit one row stay in it, and a Region's subnets, tiers and zone columns are arranged as before. Cards that stand together in these rows are drawn at one width by the rule of 9.8.3, where their labels need nearly the same width. No input field or value changed.

## v9.8.3

Changed: in a Google Cloud diagram, cards that stand together are drawn at one width in a new layout and after an `autoLayout` without `elements` (5.5, 6.5). An edit doesn't draw cards at one width again: a card it leaves alone keeps its width, and a card it adds outside a zone column takes the width its own label needs. Cards of different products more than a few pixels apart keep the width their labels need.

Changed: a `gcp.vpc` band's margin opens a lane for a line from a global item above the Regions into a card past the bands' first row, and none for a line between two items outside the boundary (5.5). No input field or value changed.

## v9.8.0

Added: `textStyle` on an edge, and `textStyle` on `createEdge` and `updateEdge` (5.3, 10.3, 10.4). It holds `wrap` alone: `auto`, `none`, `word` or `character`. `createEdge` and `updateEdge` refuse `align` and `verticalAlign`, and `setText` ignores them. Absent means `auto`. The reference engine never adds `textStyle` to an edge, and a `wrap` you send is kept as sent. `updateEdge` merges `textStyle` field by field, and `setText` on an edge takes its `wrap`.

Changed: the reference engine wraps an edge's label wider than 200 px (5.3). A label breaks into lines at most that wide, where a node's label breaks, and a `\n` still starts a line. An `EVENT [guard] / action` label breaks between its parts first.

The layout leaves room for the wrapped label: a flow left to right reserves its width between ranks, and a flow top to bottom its height. `textStyle.wrap` `none` keeps the label on one line, as every label was drawn before. A version already saved keeps its image and its fields. An edit to a saved diagram draws a label wider than 200 px wrapped, in the room its layout left. A layout of every node again, such as `autoLayout` without `elements`, makes the room.

## v9.7.1

Changed: in a Google Cloud diagram, a Region with no subnet widens on its title's side when the lines into a card from a global item above the Regions leave the card's top no place clear of the title (5.5).

Changed: a `gcp.vpc` band opens its margin before the first Region for each line between an item above the band and one below it, not between two items of one Region (5.5). Those lines run down beside the Regions, clear of the band's title. A line an edit adds grows the band and the frames around it on that side and moves no node. No input field or value changed.

## v9.4.2

Changed: a longer `title` on a Google Cloud tier can widen the Region in a saved diagram (6.4, 10.4). When a line from a row above it, inside the boundary, ends on a card and the tier's title leaves no clear place on the card's top, the reference engine widens the Region's tiers on the title's side and moves what lies past the growth by it. An `updateFrame` that sets a longer `title`, or an edit that adds the line, can cause it. A line from outside the boundary causes none, and nothing moves where nothing lies on the title's side. Before, only a title wider than the whole tier widened them. The `updateFrame` description in the operations schema names the exception. No input field or value changed.

Changed: in a Google Cloud diagram, a line past a tier opens the room it needs (6.4). For a line that runs beside the tiers to an item above or below the subnets, the reference engine opens the inset beside the tiers and the gap between two tiers. An item below the subnets that the line feeds moves under it, and what lies past the opening moves by it. A tier's line to a node of a subnet's own row, past another tier, takes the gap between zone columns, as its line to an item below the subnets did. No input field or value changed.

## v9.4.1

Changed: where an edge's line runs on the blue of a `gcp.cloud` frame, outside the filled frames it holds, and reads below 3:1 there, the reference engine draws that stretch darker, at its own hue, only as far as 3:1 needs (5.5). A `gcp.cloud` whose `style` sets `fill` keeps the line as written. No input field or value changed.

## v9.4.0

Added: `multiple` in a style object (3.5). `multiple: true` says a node stands for several instances of one thing. The reference engine draws it only on a Google Cloud product card: one gray copy behind the card, in a box 4 px wider and 4 px taller. It draws no count, and every other node ignores the property. A named style, `createNode`, `updateNode` and `setStyle` take it as they take any style property.

Changed: in an AWS or Google Cloud diagram, when an edit makes a placed node bigger, what lies past it moves by the growth once (5.5, 6.5). Before, the reference engine could move it by twice the growth or more. No input field or value changed.

## v9.3.1

Added: Google Cloud's groups as `containerType` values: `gcp.cloud`, `gcp.project`, `gcp.region`, `gcp.vpc`, `gcp.zone`, `gcp.subnet`, `gcp.tier`, `gcp.instanceGroup` and `gcp.onPremises` (5.5). The reference engine draws each as Google's Architecture Center diagrams draw that group, in Google's colors on every canvas, with rounded corners and no icon. A frame's `style` overrides the preset, except on a zone. `gcp.cloud` draws Google's wordmark when it has no `icon` of its own and no title or the title `Google Cloud`. `gcp.vpc` and `gcp.zone` are spanning frames: a VPC is drawn round its subnets across their Regions, and a zone as a strip under each node it spans.

Added: three warnings (5.5): `GCP_GROUP_NESTING` for a preset where Google doesn't nest it; `GCP_GROUP_UNTYPED` for a frame titled like a Google Cloud group that holds `gcp:` icons and has no preset; and `GCP_FRAME_HIDES_GROUPS` for a generic frame that hides presets from Google's arrangement.

Added: Google's arrangement (5.5, 6.4). In an `architecture` document, or one without a family, the reference engine lays the boundary, Region and subnet frames out as Google's reference diagrams do: zones as columns through a Region's tiers, tiers as rows in the order their edges run, and Regions side by side. Their `layout` is a minimum. Edits grow a row, a zone column or a Region for what they add, or open a gap, and what lies past the growth moves. A saved item in a row moves aside from a new line that would cross it; below a Region's subnets it does so only where the row has room.

Added: Google's connector (5.5). In a document with Google's groups or nodes with `gcp:` icons, and no AWS ones, every edge is drawn as Google's connector. An edge's `style` still sets its line's color, width and dash.

Changed: in a document with a `gcp.*` frame or a node whose `icon` is a `gcp:` key, an edge whose `edgeType` is absent, `default`, or an `er.*`, `network.*` or `org.*` type ends in `openTriangle`, the open arrow Google Cloud draws, unless it sets its own (5.3). As in a document with AWS's groups and icons, an arrowhead equal to the document's default is treated as absent, so adding or removing a document's Google Cloud groups and icons changes the arrowheads that weren't set.

Changed: in the layered layout, outside AWS's and Google's arrangements, a node an edit adds to a frame goes inside it, and each frame round it without a `layout` grows to hold it with its padding and title (6.4, 6.6). Where no place keeps those frames clear of the frames and nodes beside them, the node goes past the frame's other members in its column. What lies past the frame's far border then moves across the flow by the growth. Before, the frames round it could grow over a node or a frame beside them. No input field or value changed.

## v9.2.0

Changed: the reference engine leaves the characters no SVG can hold out of the drawing (3.5): a NUL, a vertical tab and the other control characters below U+0020, apart from the tab, line feed and carriage return, and U+FFFE and U+FFFF. Text that carries one is measured and drawn as the text without it, and the stored document keeps the text as written. Before, such a character made the SVG not well formed.

## v9.1.3

Changed: a document that carries `autoSize: "children"` on a frame's absolute `layout`, a value the schema doesn't list, is read as if the frame had no `layout` (11, 13), as `computed` already is. Before, a version's record sent back whole failed validation on that frame. The schema is unchanged: `autoSize` names what an author writes, `content` or `none`.

## v9.1.0

Changed: the reference engine draws emoji in their own colors, where it drew empty boxes (3.5): every emoji of Unicode 18.0, with skin tones, flags, keycaps and ZWJ sequences each drawn as one emoji, never bold or slanted. A sign Unicode shows as text by default, such as ⚠ or ❤, is drawn as text, and as an emoji when the emoji selector (U+FE0F) follows it. An emoji is about a quarter wider than the font size, so a node sized by a label that holds one can be wider than before. Checking or storing a document no longer returns `CHARACTER_NOT_DRAWN` for an emoji. No input field or value changed.

## v9.0.1

Changed: without `lang`, the reference engine counts each Chinese character for every list that has it, the PRC's general list, Big5 and the Jōyō and Jinmeiyō kanji, and the list that has the most of the text decides; a tie of Big5 and the Japanese lists is Traditional Chinese (3.2). A short Traditional Chinese label, such as 資料庫, read as Simplified Chinese before.

Changed: the reference engine styles an edge label written as `EVENT [guard] / action` only in flowchart, workflow, UI flow and sequence diagrams; every other diagram, one with no `diagramFamily` included, draws edge labels as plain text (the edge `label` description). An icon given as a remote URL returns the warning `ASSET_URI_POLICY_VIOLATION`, as an asset's remote URI does (9.2).

## v8.42.0

Added: `scene.lang`, the language the document's text is in, as a BCP 47 language tag (3.2). `updateScene` sets it with `lang`, and `null` removes it (10.8). A tag that isn't well formed is invalid (`LANG_INVALID`, 9.2). The reference engine stores it in canonical form (11), and without it infers the language from the text (3.2).

Changed: `fontFamily` names one of two families, `sans`, the default, or `mono`, a monospace font (3.5). The reference engine draws any other value as `sans` and returns the warning `FONT_FAMILY_UNKNOWN`. The schema still accepts any string.

Changed: the reference engine draws Chinese, Japanese and Korean, where it drew empty boxes, a Chinese character in the forms of `scene.lang` (3.2, 3.5). It breaks wrapped text by the Unicode line breaking rules (UAX #14), with Korean kept whole between spaces (5.2). `mono` draws `->` as typed.

## v8.41.0

Changed: the reference engine draws a character Inter lacks from a fallback font, where it drew an empty box (3.5). That adds Armenian and Georgian letters, the rarer Latin, Cyrillic and Greek letters, the phonetic alphabet, and more arrows, math and logic signs, symbols and box drawing. Text is shaped as browsers shape it: letters are kerned, and `->` is drawn as an arrow. A label can be a few pixels narrower than before, and so can a node sized by its label. No input field or value changed.

## v8.39.7

Changed: `computed` is `readOnly` in the schema and accepts any value (3.1, 13). The reference engine ignores a document's `computed` without checking it, and works it out again as it always has. Before, a document carrying the reference engine's own `computed` failed validation, because the schema listed four of its fields and refused the rest. No input field changed.

The schema drops the definitions only `computed` used: `computedLayoutRegistry`, `computedEdgePathRegistry`, `computedEdgePath`, `diagnostic` and `bounds`. Section 9.3 gives a diagnostic's shape itself, and section 13 no longer lists `normalizedHash`. A schema of your own that refers to one of the definitions can copy it from the v8.39.0 tag.

## v8.39.0

Changed: the reference engine returns one warning, `CHARACTER_NOT_DRAWN`, for a document whose text holds a character it draws as an empty box (3.5). Such a character is a letter of a script other than Latin, Cyrillic or Greek, an emoji, or a sign Inter lacks. The warning lists up to five of the characters and names up to three of the elements whose drawn text holds one. The document is valid and is stored as written. No input field or value changed.

The spec now says that the reference engine draws all text in Inter, whatever `fontFamily` says (3.5). It always has.

## v8.36.0

Added: six `nodeType` values for a sequence diagram's participants (5.2): `sequence.boundary`, `sequence.control`, `sequence.entity`, `sequence.database`, `sequence.collections` and `sequence.queue`. Each heads a column as `sequence.lifeline` and `sequence.actor` do. Messages, notes, activation bars, participant boxes (5.5), create and destroy messages (5.9) and `after` and `before` (10.3) take any of the eight; before, they took a lifeline or an actor.

The reference engine draws `sequence.boundary`, `sequence.control`, `sequence.entity` and `sequence.database` as an icon with the label under it, as it draws an actor. It draws `sequence.collections` and `sequence.queue` as a box with the label inside, as it draws a lifeline.

## v8.35.0

Changed: the reference engine's drawing of frames in `sequence` documents (5.5). No input field or value changed. Before, only `fragment` frames were laid out with the document's rows.
- A frame of any other `containerType` whose `children` hold lifelines or actors is drawn as a participant box: a bordered rectangle round their columns, with its `title` centered above their heads.
- One whose `children` hold messages, notes, `fragment` frames or other bands is drawn as a highlight band: a filled rectangle behind those rows, in its `style.fill` or a faint tint, with its `title` at its top-left. Bands and fragments nest in each other.
- Both ignore their `layout`. A frame that holds lifelines and rows isn't drawn (`FAMILY_SEQ_FRAME_MIXED`), nor is one that holds neither (`FAMILY_SEQ_EMPTY_FRAGMENT`). Of two boxes that share a lifeline one is drawn (`FAMILY_SEQ_BOX_NESTED`), and a lifeline between a box's lifelines that the box doesn't list is drawn inside it (`FAMILY_SEQ_BOX_SPLIT`).

Changed: `after` and `before` (10.3) take a highlight band where they take a `fragment` frame: a message or note placed next to one of a band's rows joins the band, one given the band goes outside it, and a deleted band's rows stay in the fragment or band that holds it (10.4).

## v8.33.2

Changed: the reference engine's drawing of `sequence` documents. No input field, value or rule changed.
- A numbered message leaving an activation bar has its badge centered on the bar's axis (5.3, 8); the arrow still starts on the bar's face.
- The first row starts below the tallest head at the top, an actor's figure and name included, so a first message's label clears an actor's name.
- A fragment's guard whose text would meet an activation bar open in its row starts 6 px past that bar; the fragment's guards share one column when they can, and its right edge grows for a guard moved right.
- Fragments of one depth beside the same lifeline share a left edge.
- A dashed line ending at a `cross` has a dash end at the X's center.

## v8.31.0

Added: `autonumber` in a document's `extensions` (8). In a `sequence` document it numbers every drawn message in row order: `true` counts from 1 by 1, and `{ "start", "increment" }` sets where the count starts and what it goes up by, each an integer from 1. A message's own `step` is drawn as given and the count goes on from it; notes, hidden messages and messages to or from a hidden lifeline take no number. The reference engine draws each number as a step badge centered on the start of the message's arrow, and in any other family draws nothing for it and warns (`AUTONUMBER_NOT_DRAWN`).

Added: `side` in a `sequence.note`'s `extensions` (8), `left` or `right`: with a string `anchor` the note sits beside that lifeline instead of over it. The scene schema now lists the note's `anchor` (a string, or an array of strings) beside it. With an array `anchor` the reference engine draws the note over the lifelines and warns (`FAMILY_SEQ_NOTE_SIDE_IGNORED`).

Added: `cross`, an eighth value of `startArrow` and `endArrow` (5.3): an X on the line just short of the node.

Added: `extensions` on `updateScene` (10.8), which sets `autonumber`, `showLifelineFeet`, `autoGroup` and `layoutHints` in the document's `extensions`, each given replacing its value and `null` removing it; no other key. A changed `layoutHints` lays the document out again, as a changed `layoutStrategy` does.

Changed: `step` is drawn in every family (5.3); before, a `sequence` document drew no badges and warned (`STEP_NOT_DRAWN`, removed). In a `sequence` document the badge is centered on the start of the message's arrow.

Changed: `createFrame`'s `frame` takes each element the new frame lists out of that frame's `children` (10.3), so a nested `fragment` frame deleted and created again in one envelope goes back round its rows; before, its rows stayed in both frames (`MULTIPLE_FRAME_MEMBERSHIP`).

Changed: the reference engine's drawing of `sequence` documents. No input field, value or rule changed beyond those above.
- A lifeline whose first row is a `sequence.create` message arriving at it begins at that row, and one that a `sequence.destroy` message arrives at ends there with no foot bar (5.9). A lifeline used above its create message or below its destroy message is drawn full height on that side, with a warning (`FAMILY_SEQ_USED_BEFORE_CREATE`, `FAMILY_SEQ_USED_AFTER_DESTROY`).
- An activation bar that opens while another is open on its lifeline is drawn 8 px to the right of it, and messages meet the outermost bar face.
- A dashed message type's `6,3` dash gives way to the edge's `style.dash`, and `[]` draws it solid.

Changed: the reference service reads and writes Mermaid `sequenceDiagram` text, so a document of every family has a Mermaid view, and imports Mermaid's cross ends exactly as `cross`. Mermaid isn't part of this spec; the service's documentation describes it (section 14).

## v8.30.0

Added: `flowchart`, an eighth value of `diagramFamily` and of `updateScene`'s `diagramFamily` (3.3, 10.8), for steps, decisions and loops. It uses the generic node and edge types. The reference engine lays a flowchart out top to bottom when `layoutStrategy.direction` is absent, and aims for 50 px between ranks when `rankSpacing` is absent (3.4). It sends a decision's exit labeled with an affirmative answer (such as `yes`) straight on and one labeled with a negative answer (such as `no`) to the right, below in `LR` and `RL`; other labels, and a flowchart with a frame laid out with its members, keep the general decision rules. Each entry into a node keeps its own arrowhead, entries into one side of a decision join, and edges with the same label keep their labels. In the default style its nodes, except external systems, notes and nodes with an `aws:` icon, draw a light blue fill with a blue border unless their style or the palette sets a fill or stroke. When an edit inserts nodes between two placed nodes, the reference engine puts them on the first one's line and moves what lies past them along the flow (6.4), except a new node on a decision's negative exit, which goes beside its decision, and in a flowchart with a frame laid out with its members. It warns (9.3) about a decision with fewer than two exits (`FAMILY_FLOWCHART_DECISION_FEW_EXITS`) and about an unlabeled exit of a decision with two or more (`FAMILY_FLOWCHART_EXIT_UNLABELED`).

Added: the `document` (a box with a wavy bottom) and `subroutine` (a predefined process: a box with a line inside each side) values of `shape`, on nodes, `createNode` and `updateNode` (5.2). Every shape is valid in every family. When `shape` is absent, the reference engine sets a `document` node's `shape` to `document`; a `document` node stored with another shape keeps it.

Changed: the reference engine draws a document without `diagramFamily` as an `architecture` document (3.3). `extensions.autoGroup` bands it (8), and edges with the same label into one target each keep their label and arrowhead, as in `architecture`; before, it collapsed such edges into one labeled flow where their sources formed one unit. A document stored without a family keeps its node positions until it is laid out again.

Changed: the reference engine draws a parallelogram leaning by half its height, at most a quarter of its width, and a hexagon's side points a quarter of its height out from its top's corners, at most a quarter of its width. It widens either shape so its label fits between the slanted sides. No input field, value or rule changed.

Changed: the reference engine keeps an end that draws an arrowhead or glyph on a `roundedRect` or `pill` 6 px past the corner's curve, an anchored end included: at least 14 px from a `roundedRect`'s corners, and half the height plus 6 px along a pill's top and bottom. No input field, value or rule changed.

Changed: the reference service imports Mermaid `flowchart` and `graph` text as a `flowchart` document; before, as `architecture`. Mermaid's `doc` and `document` shapes become `document`, and `[[ ]]`, `subroutine`, `fr-rect` and `subproc` become `subroutine`. Reading a document of any family but `flowchart` and `entityRelationship` as Mermaid notes that the text doesn't name its family (`FAMILY_NOT_EXPORTED`). Mermaid isn't part of this spec; the service's documentation describes it (section 14).

Changed: descriptions only; no field, value or rule changed.
- `layoutStrategy`'s `direction` and `rankSpacing` give the flowchart defaults, and `layoutStrategy` lists `flowchart` among the layered families.
- `extensions.autoGroup` applies to `architecture` documents and documents without a family.
- `isFanInFeeder` and `junctions` call a decision diamond outside the `workflow.*` types a plain decision, and list `flowchart` among the families that draw no fork dots.
- `laneDirection` says lanes stack top to bottom in a left-to-right flow, the default outside flowcharts and org charts, and side by side in a top-to-bottom one.
- `resizeElement`'s `anchor` names the side a flow enters in each family's default direction.
- An edge end's `anchor` with an `offset` keeps the same corner distance as any end (8 px, more past a rounded corner); before, it said 8 px.
- The node `layout` description (scene schema) and `autoLayout`'s (ops schema) list the new case of 6.4.

## v8.28.3

Changed: the reference engine's layout and drawing. No input field, value or rule changed.
- A tier, a `generic` frame round subnets next to each other in one Availability Zone's rows, is fitted as a group nested in the zone (5.5). Its border stands 20 px out from its subnets, and the zone's border 20 px out from it; v8.28.0 fitted it 10 px out, inside the zone's border. Every zone of a VPC that holds a tier starts its subnets 40 px in from its left and right borders, so the subnets keep one width and the zones one size.
- A step badge that stands opposite another badge across that badge's line moves beside its own line, where its line has a clean place (5.3).
- Two lines that run along one side of a node from the same direction, into it or out of it, nest instead of crossing, where one run can move clear of nodes and other lines.
- Outside `sequence` documents, an edge label keeps off a place where another line crosses its run and ends nearby in an arrowhead or ER glyph, where the run has room.
- In an AWS diagram, an end leg moved apart from another line is never laid on another edge's line or left seeming to run through an icon.

Changed: in an AWS diagram, the reference engine keeps the room round a tier frame, a `generic` frame round subnets next to each other in one zone's rows, when it lays out stored positions (6.4). Where the tier's padding or its header narrows a gap round the zones, the gap gets back what the tier took, up to the room a fresh layout keeps there, such as each side of a load balancer between two zones. What lies past the gap moves right or down by that much. It does so whether an edit adds the tier or the document was stored with one. A document stored with the tier drawn as a box still keeps its node positions until it is laid out again. No input field, value or rule changed.

Changed: wording only; 6.4 lists the cases in which a stored position moves one to an item. No input field, value or rule changed.

Changed: descriptions only; no field, value or rule changed.
- The node `layout` description lists the new case of 6.4.

## v8.28.0

Added: `step` and `stepText` on edges, `createEdge` and `updateEdge` (5.3, 10.3, 10.4). `step` is an integer from 1 to 99; `stepText`, the edge's line in the step list, needs `step`. On `updateEdge`, `stepText: null` removes the text and `step: null` removes both. The reference engine draws a numbered badge beside the start of each stepped edge's line, one for the edges with one step and one source, and lists the steps with text beside the drawing. It warns about a step in a `sequence` document, which it doesn't draw (`STEP_NOT_DRAWN`).

Added: `spans` on frames, `createFrame` and `updateFrame` (5.5, 10.3, 10.4). A frame with `spans` instead of `children` is a spanning frame, drawn round the nodes and frames it names, outside the containment tree, such as an Auto Scaling group across Availability Zones. A frame needs `children` or `spans`; before, `children` was required. A spanning frame holds nothing, sits in no group or frame, and spans no other spanning frame (`SPAN_INVALID`), and each target must exist (`SPAN_TARGET_NOT_FOUND`, 9.2). In an edit, a target that nothing creates is `OP_TARGET_NOT_FOUND`, and `createFrame` with `spans` takes neither `children` nor `frame` (10.2). `deleteElement` drops what it removes from every `spans`, and removes a spanning frame left with no target (`SPAN_REMOVED`). The reference engine draws spanning frames in `architecture` documents and documents without a family, keeps room round their targets, and warns about one elsewhere (`SPAN_NOT_DRAWN`) and about a box that covers a node it doesn't span (`SPAN_COVERS_NODE`).

Changed: the reference engine reads through a `generic` frame round subnets next to each other in one Availability Zone's rows (5.5): the subnets join the zone's rows, shared with the other zones, and the frame is fitted round them. `AWS_FRAME_HIDES_GROUPS` no longer warns about such a frame, and for a frame that hides subnets its fix names `spans`. A document stored with such a frame drawn as a box keeps its node positions until it is laid out again. No input field, value or rule changed.

Changed: in an AWS diagram, the reference engine moves a Region's column right for a labeled line's room after an edit that only adds edges, as it did after one that places nodes, and a spanning frame's header grows the row it starts in (6.4). No input field, value or rule changed.

Changed: descriptions only; no field, value or rule changed.
- The node `layout` description lists the new cases of 6.4.
- `containerType`'s description (both schemas) says a frame round subnets next to each other in one zone is read through, and that a group across zones is a spanning frame.
- The descriptions of `updateFrame` and `deleteElement` name `spans`.

## v8.25.0

Changed: descriptions only; no field, value or rule changed.
- `isFanInFeeder`'s description says that edges join at a face with no room to keep them apart only in a family that dots forks: `architecture`, `entityRelationship` and `network` documents and documents without a family.

## v8.17.0

Changed: in a document with a node whose `icon` is an `aws:` key, with or without an `aws.*` frame, an edge whose `edgeType` is absent, `default`, or an `er.*`, `network.*` or `org.*` type ends in `openTriangle`, AWS's open arrow (5.3). Before, only a document with an `aws.*` frame did. An arrowhead equal to the document's default is treated as absent, so adding or removing a document's AWS icons changes the arrowheads that weren't set.

Changed: the reference engine docks a gateway a VPC holds on the side of its border facing the gateway's neighbors outside the VPC, when its lines inside stay in the zone nearest that side, or reach the row above the zones or an item beside a lone zone on that side, and on the top otherwise, where every gateway docked before (5.5). A gateway already placed keeps its side until the document is laid out again. No input field, value or rule changed.

Changed: the reference engine reads through a `generic` frame round whole VPCs of a Region, or round all of a VPC's zones, and arranges the presets inside it as AWS's reference diagrams do, the frame fitted round them (5.5). It warns about any other generic frame inside a Region, VPC, zone or subnet that holds presets (`AWS_FRAME_HIDES_GROUPS`), whose presets are still laid out as a generic frame's members. No input field, value or rule changed.

Changed: descriptions only; no field, value or rule changed.
- The edge arrowheads' description and `containerType`'s (both schemas) say the same.

## v8.13.1

Changed: corrects v8.13.0's open arrow (5.3). In a document with an `aws.*` frame, only an edge whose `edgeType` is absent, `default`, or an `er.*`, `network.*` or `org.*` type ends in `openTriangle`; `workflow.sequenceFlow`, `sequence.message`, `sequence.create`, `sequence.destroy` and the `uiflow.*` types keep `triangle`. The reference engine drew it this way in v8.13.0; v8.13.0's text said every type ending in `triangle`.

Changed: the reference engine applies an `order` constraint an edit adds to an AWS diagram (6.4): when its two nodes stand the wrong way around in one row or column, they trade places and nothing else moves; when they don't share one line, or trading would leave two nodes nearer each other than both the node spacing and the nearest two stood before, they are placed again as nodes an edit adds are, which can grow their row or column. New nodes an edit joins by an order constraint are placed in the order asked. In v8.13.0 the engine placed such a pair again still the wrong way around and could move other nodes. A fresh layout no longer splits an ordered pair between the two halves of a Region's column. No input field, value or rule changed.

Changed: descriptions only; no field, value or rule changed.
- AWS's arrangement (5.5): a Region holding zones or subnets directly lays them out as a VPC it doesn't draw. A frame of another type between a Region and its VPC, or a VPC and its zones, is laid out as a generic frame is, with the presets inside it; one between a zone and its subnets is placed in the zone's row after its subnets, everything in it on that row.
- The edge arrowheads' description and `containerType`'s (both schemas) say the same.

## v8.13.0

Added: AWS's groups as `containerType` values: `aws.cloud`, `aws.region`, `aws.az`, `aws.vpc`, `aws.subnet.public`, `aws.subnet.private`, `aws.securityGroup`, `aws.autoScalingGroup`, `aws.account` and `aws.corporateDataCenter` (5.5). The reference engine draws each as the AWS Architecture Icons draw that group, in AWS's color and line with its corner icon and no fill; a frame's `style` overrides the preset. It warns about a preset inside one AWS doesn't nest it in (`AWS_GROUP_NESTING`) and about a frame titled like an AWS group that holds AWS icons and has no preset (`AWS_GROUP_UNTYPED`). In an `architecture` document, or one without a family, it lays the Cloud, Region, VPC, Availability Zone and subnet frames out as AWS's reference diagrams do: zones as columns in the order of the VPC's `children`, subnets as rows every zone shares (public before private), a gateway the VPC holds on its top border, a Region's other members in a column beside its VPCs, and AWS Cloud's above its Regions. Their `layout` is a minimum. A node an edit adds takes a free place in its row or column; when there is none, the row or column grows and what lies past it moves (6.6).

Added: `icon` on frames, `createFrame` and `updateFrame`, a `family:id` key or an image URL drawn in the frame's header (5.5). A frame's `icon` replaces its preset's.

Added: `frame` on `createFrame`, the ID of a frame (10.3). It appends the new frame to that frame's `children`, so a lane goes after a pool's other lanes and an Availability Zone after a VPC's. A missing frame is `OP_TARGET_NOT_FOUND`, one that isn't a frame `OP_TARGET_KIND_MISMATCH` (10.2).

Added: the operation `updateFrame`, which changes a saved frame's `title`, `style`, `icon`, `containerType` or `laneDirection` (10.4). `null` removes the `style`, the `icon`, the `containerType` (the frame becomes `generic`) or the `laneDirection`. It doesn't change the frame's `children`, `layout` or `extensions`, and the reference engine keeps its members' positions.

Changed: in a document with an `aws.*` frame, an edge whose type's default end is `triangle` ends in `openTriangle`, AWS's open arrow, unless it sets its own (5.3). An arrowhead equal to the document's default is treated as absent, so adding or removing a document's AWS groups changes the arrowheads that weren't set.

Changed: computed edge paths (13). In an architecture, entity-relationship or network diagram, or one without a family, `junctions` also lists each point where edges drawn alike that leave one point (or end at one point) fork, drawn as a dot; a crossing carries none.

Changed: descriptions only; no field, value or rule changed.
- `style.dash` (scene schema) loses a wrong default of `[]`: left out, a generic frame is dashed `[4, 4]` and a preset takes its own; `[]` draws solid.
- A node's `layout` and stored positions (6.4) list an AWS row or column growing among the ways a placed node can move.

## v8.12.0

Changed: the reference engine draws nothing for a group (5.4). Its `title` is written into an SVG as the group's `<title>` element, which isn't drawn; earlier engines drew it as text without a position, at the canvas origin. A hidden group drops only that `<title>` (5.1). For a visible box with a title, use a frame (5.5).

Changed: in a sequence diagram the reference engine draws messages over lifelines, actors and activation bars, whatever their `zIndex` (5.1). `zIndex` still orders notes and every other element. No input field, value or rule changed.

Changed: descriptions only; no field, value or rule changed.
- A group's `title` (both schemas) and a hidden group (`visible`): not drawn, written into the SVG as the group's `<title>`.
- `autoGroup` (scene schema): it is also skipped when a band would overlap a frame or another band.

## v8.10.0

Added: `frame` on `createNode` and `updateNode`, the ID of a frame of any `containerType` (10.3, 10.4). On `createNode` it appends the new node to that frame's `children`, such as a task in a lane that already exists; on `updateNode` it moves the node into that frame and out of every other frame and group that lists it. The frame must exist when the operation runs, as the element `after` or `before` names must, and a missing one is `OP_TARGET_NOT_FOUND`, one that isn't a frame `OP_TARGET_KIND_MISMATCH` (10.2). `frame` takes a string only, so it can't take a node out of every frame.

Changed: the reference engine lays pools and lanes out as BPMN tools do (5.5, 6.4, 6.6). Lanes run along the flow and stack across it in the order the pool's `children` list them, every node a lane lists is placed inside it, and columns are shared across lanes and pools. A pool's `laneDirection` sets the flow (`vertical`, lanes stacked top to bottom, flows left to right; `horizontal`, lanes side by side, top to bottom), winning over the document's direction when the two disagree; without one, lanes stack across the document's direction. A pool's or lane's `layout` is a minimum: they grow to hold their members, so their members no longer get `NODE_OUTSIDE_FRAME`, and a pool's lanes no longer split its box evenly. A node an edit adds to a lane takes a free place in its column or grows the lane, one added between two placed columns opens a column, and a node an edit moves into a lane is placed again there (6.4, 6.6). Pools and lanes are drawn with BPMN header strips. No input field or value changed; every stored document with pools draws differently on its next layout.

Changed: descriptions of `containerType` and `laneDirection` in both schemas, and of the node `layout` and `autoLayout`, which list the new ways a placed node can move.

## v8.9.2

Changed: descriptions only; no field, value or rule changed.
- `resizeElement`'s `anchor` (ops schema): to make a placed node bigger where it stands, name the side its flow enters (`left` left to right, the direction when none is set; `top` top to bottom and in an org chart); without it the top-left corner stays put, as before, and the node's center moves by half the growth.
- `junctions` on a computed edge path (13): a flowchart decision's branches that share a vertex also carry the point where their shared line splits. The reference engine now draws a decision's branches past its free vertices that way (5.3).

## v8.9.0

Added: `"auto"` for `textStyle.wrap`, now a node's default (5.2). A label stays on one line while its node is at most 250 px wide, or 400 px for a diamond, and a longer one wraps at spaces there in lines of even length; a node whose `layout` is wider keeps the line up to its own width. A text element's default stays `none` (5.6). Documents an earlier engine saved store `wrap: "none"` on their nodes and draw as before. 6.5 says how `auto` sizes a node.

Changed: the reference engine routes a decision's exits out of different vertices, and edges entering one side of a flowchart decision join one trunk into its vertex, with a dot where their lines branch (5.3). No input field, value or rule changed.

Changed: computed edge paths (13). `junctions`, on the edge whose last leg carries a shared port's joined edges, lists every point where their lines branch, and `isFanInFeeder` marks the others. `keepFeederArrowhead` is removed, and `fanInPoint` and `isFanInTrunkOwner`, which earlier engines wrote without the schema listing them, aren't part of it: a document whose `computed.edgePaths` carries any of the three is refused. Computed output is the engine's to write, so a document sent without `computed` is unaffected.

## v8.8.0

Added: the operations `createText` and `createImage`, which add a text element and an image element to a saved document, and `assets` on `updateScene`, merged entry by entry, `null` removing one (10.3, 10.8).

Changed: the reference engine draws a node's label at its `textStyle.align` and `verticalAlign` whether or not the label wraps, and a text element's text at its `verticalAlign`, as 5.2 and 5.6 describe. It drew a label that didn't wrap centered, and a text element's line in the middle of its box, whatever they said.

## v8.5.0

Changed: the reference engine draws a `straight` edge as the line between its two nodes' centers, ending where it crosses each node's outline, and a `curved` edge as one quadratic Bezier between them, ending on each outline (5.3). No input field, value or rule changed.

Added: `control` on a computed edge path (`computedEdgePath`), a curved edge's control point: the curve is `M points[0] Q control points[1]` (13). A curved path without it, from an earlier engine, is drawn with its control point 30% of the chord off the chord's middle, left of travel.

Added: the reference engine's warning `EDGE_CROSSES_NODE`, for a `straight` or `curved` edge that runs through a node it doesn't join (5.3).

## v8.2.2

Changed: the spec and the schemas' descriptions use American English ("center", "neighbor", "behavior"), as the fields always have (`color`, `centerIn`). No field, value or rule changed.

## v8.0.0

Changed: a `fragment` frame's operands start at a row. Each item of `extensions.operands` is `{ "startsAt", "guard" }`, with `startsAt` the ID of a message or note in the fragment after its first row (5.5, 8), checked by `FAMILY_SEQ_OPERAND_NOT_FOUND` (9.2). An operand with a `y` is refused. The reference engine lays a fragment in a sequence diagram out with its rows and ignores its `layout`, and draws a divider for every operator, not only `alt`.

A version stored with `y` operands still draws: the reference engine maps each to the first of its fragment's rows drawn at or past that `y` before 8.0.0. There is no `updateFrame`, so to rewrite one, delete the frame and create it again with `startsAt` operands in one edit.

Added: `extensions.startsAt` and `extensions.endsAt` on `sequence.activation`, the messages a bar runs between (8), checked by `FAMILY_SEQ_ACTIVATION_ANCHOR_NOT_FOUND` (9.2). A bar stored with a box instead is drawn over the messages that box covered on the nearest lifeline.

Changed: `deleteElement` moves an operand or an activation off a removed row, and removes one left with nothing, with the `info` notes `FRAGMENT_OPERAND_REMOVED` and `ACTIVATION_REMOVED`. `after` and `before` (10.3, 10.4) put a row in the operand of the row it is placed next to, so a row placed before an operand's first row starts it.

## v7.2.0

Added: `after` and `before` on `createNode`, `createEdge`, `updateNode` and `updateEdge`, which put a sequence diagram's lifeline, actor, message or note in its place in the order, or move it there (5.9, 10.3, 10.4), with the errors `ORDER_TARGET_NOT_FOUND` and `ORDER_NOT_SUPPORTED` (10.2).

Changed: the reference engine no longer warns about a self-loop in a sequence or entity-relationship diagram (5.3).

Changed: the reference engine no longer stores the arrowheads an edge's `edgeType` gives; it works them out each time it draws the edge, so changing `edgeType` changes the arrowheads that weren't set (5.3, 11). A version stored with them draws the same, and an arrowhead equal to its type's is treated as absent.

## v7.0.0

Compared with v6.0.0, these no longer accept:

- `ports` on a node, and `ports` on `createNode` and `updateNode`. Nothing read them: no version of the reference engine placed an edge end at a port.
- `portId` on an edge end. Place an end with its `anchor` instead (5.3).

Every node stored before Zindex 7.0.0 carries `ports: []`, which validates against v6.0.0's schema, not this one. The Zindex API strips `ports` and `portId` from what it is sent before validating it, and warns (`PORTS_REMOVED`) where either said something; a version stored with them loses them when an edit next saves it.

Changed: an end's `anchor` takes `offset` optionally. A `side` alone fixes the side; with `offset` the place too. The reference engine now places anchored ends (5.3).

## v6.0.0

First public release: SPEC.md, `schemas/scene.schema.json`, `schemas/ops.schema.json` and the examples, under the Apache License 2.0.

Compared with the schemas zindex.ai served before (Zindex 5.1.1), these no longer accept:

- `transactionMode` on the operations envelope. Every edit applies whole or not at all (10.2).
- `baseRevision: 0`. Revisions are numbered from 1.
- `required` on a constraint, `rows` on a `grid` constraint and `axis` on a `centerIn` constraint. Nothing read them: a grid's rows follow from `columns`, and `centerIn` always centers on both axes. A version stored with them loses them when an edit next saves it.

These are no longer described, and are still accepted, because `layoutStrategy` accepts other keys: `layoutStrategy.algorithm`, which was required and is now optional and ignored, and `nodeWidth` and `nodeHeight`. The reference engine picks its placement method from `diagramFamily` (3.4).

Added: `textStyle` on `createNode` and `updateNode` (10.3, 10.4).
