# Diagram Scene Protocol

Format version 0.1. Every document and edit in this version carries `"schemaVersion": "0.1"`.

The Diagram Scene Protocol (DSP) is a JSON format for diagrams that programs keep and edit over time. A scene document describes one diagram: its elements, their styles and constraints, and where its nodes are. An operation envelope describes one edit: a list of typed operations, applied to a named revision of a document.

Two JSON Schemas, both draft 2020-12, define the shape of these documents:

| File | `$id` | Defines |
|---|---|---|
| `schemas/scene.schema.json` | `https://zindex.ai/schemas/scene.schema.json` | Scene documents |
| `schemas/ops.schema.json` | `https://zindex.ai/schemas/ops.schema.json` | Operation envelopes |

The schemas are the machine-readable contract. This text says what the fields mean and states the rules a schema can't express. Where this text and a schema disagree about a document's shape, the schema is right, and the disagreement is a bug in this text.

## 1. Scope

This spec defines:

- the scene document: its fields, the diagram families and the element kinds;
- element IDs, and how documents and edits refer to elements;
- the operation envelope and every operation;
- revisions: base revisions, conflicts, retries with `requestId`, and edits that change nothing;
- constraints;
- what stored positions and sizes mean;
- extensions;
- the diagnostic format the schemas carry, and the rules a valid document follows beyond its shape.

It doesn't define how to place nodes, route edges, place labels or draw a diagram. Those algorithms belong to each implementation, and two implementations can draw the same document differently. It doesn't define an API either: how documents are stored, sent, listed and rendered is up to the implementation.

## 2. Conventions

The words MUST and MUST NOT are used as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when they appear in capitals. This spec uses them only for rules that the schemas enforce or that the reference engine enforces. Text without them describes what fields mean, or what the reference engine does.

The reference engine is Zindex's implementation of DSP, and the reference service is the hosted API around it (section 14). Where this spec describes their behaviour, the sentence names them. Another implementation can behave differently in those places and still read and write the same documents.

Paths such as `/elements/0/layout` are JSON Pointers into a document. "Absent" means the property isn't in the object.

## 3. Scene documents

### 3.1 Top-level fields

| Field | Required | Meaning |
|---|---|---|
| `schemaVersion` | Yes | The string `"0.1"`. |
| `scene` | Yes | The diagram's ID, title, brief and canvas (3.2). |
| `elements` | Yes | The diagram's elements, in order (section 5). |
| `diagramFamily` | No | The kind of diagram (3.3). |
| `layoutStrategy` | No | How to place nodes that have no position (3.4). |
| `palette` | No | Colours for elements without their own (3.5). |
| `styles` | No | Named styles (3.5). |
| `assets` | No | Images that `image` elements use (3.6). |
| `constraints` | No | Spatial rules between elements (section 7). |
| `extensions` | No | Data the schemas don't define (section 8). |
| `computed` | No | Output of a layout, not input (section 13). |

Most objects in the schemas reject properties they don't list. `metadata` and `extensions` objects accept any properties, for data the schemas don't define.

### 3.2 The scene object

| Field | Meaning |
|---|---|
| `id` | The diagram's ID: 1 to 256 characters from `A-Z a-z 0-9 . _ : -`. |
| `title` | A title for people. |
| `description` | A longer description. |
| `brief` | Up to 1,000 characters for the next editor: what the diagram shows, conventions to keep, decisions made. It belongs to the document, so each revision has its own. |
| `units` | `"px"`, the only unit. |
| `canvas` | `width`, `height`, `background` (a CSS colour), `grid` and `snap`. |
| `viewport` | `originX`, `originY` and `zoom` (above 0). |
| `defaults` | `fontFamily`, `fontSize`, `strokeWidth` and `router`. |
| `metadata` | `createdBy`, `createdAt` and `updatedAt` (RFC 3339 date-times), `tags` and `provenance` (any object). |
| `extensions` | An open object. |

A service that stores documents can assign `scene.id` itself. The reference service does: it gives each new diagram its own ID and ignores the one sent.

`canvas.width` and `canvas.height` are a minimum. The reference engine grows the drawing to fit its content, and on a layout that places every node it writes the size it used into `canvas`. When `canvas` has no size, the reference engine uses 800 by 600.

The reference engine keeps `units`, `canvas.grid`, `canvas.snap`, `viewport` and `defaults`, but doesn't use them to lay out or draw. It doesn't keep `scene.extensions`: put extension data at the document's top level or on an element.

### 3.3 Diagram families

`diagramFamily` names the kind of diagram. It decides which conventions an implementation applies: the family's node and edge types, its validation warnings, and how the reference engine lays the diagram out and draws it.

| Family | Node types | Edge types |
|---|---|---|
| `architecture` | The generic types (5.2) | `default` |
| `workflow` | `workflow.*` | `workflow.sequenceFlow`, `workflow.messageFlow` |
| `entityRelationship` | `er.*` | `er.relationship`, `er.identifying` |
| `sequence` | `sequence.*` | `sequence.message`, `sequence.reply`, `sequence.create`, `sequence.async`, `sequence.destroy` |
| `network` | `network.*` | `network.dataFlow`, `network.controlFlow` |
| `orgchart` | `org.*` | `org.directReport`, `org.dottedLine` |
| `uiflow` | `uiflow.*` | `uiflow.navigate`, `uiflow.back`, `uiflow.action` |

Workflow diagrams use `pool` and `lane` frames for swimlanes, and sequence diagrams use `fragment` frames for combined fragments (5.5). The table shows each family's own types. The schemas don't tie a type to a family: any node type, edge type and frame type is valid in any document.

`diagramFamily` is optional. Without it, the reference engine lays the diagram out and draws it as a generic flowchart, and applies no family's validation warnings. Declare it on every document that has a family.

### 3.4 Layout strategy

`layoutStrategy` guides where an implementation places the nodes that have no position (6.4). It has `direction` (`TB`, `BT`, `LR` or `RL`: top to bottom, bottom to top, left to right, right to left), `nodeSpacing` (pixels between nodes in the same rank) and `rankSpacing` (pixels between ranks). The schema accepts other keys in `layoutStrategy`, and implementations ignore them.

The reference engine chooses its placement method from `diagramFamily`, not from `layoutStrategy`. Its layered layout, used for architecture, workflow, entity-relationship and UI flow diagrams and for diagrams without a family, reads all three fields, and a field left out takes the `default` the schema shows: `LR`, 30 and 80. The spacings are what it aims for, not minimums: when it lays out every node it also tries tighter and wider spacings and keeps the layout that scores best. Org charts (a tree, top to bottom), network diagrams (placed by forces) and sequence diagrams (placed from their messages) don't read `layoutStrategy`. The document's `extensions.layoutHints.primaryDirection` turns an org chart's tree, and wins over `direction` in the layered layout (section 8).

### 3.5 Styles and the palette

An element's `style` is either the name of an entry in the document's `styles`, or an inline style object. A style object can have:

| Property | Value |
|---|---|
| `fill`, `stroke`, `textColor`, `labelBackground`, `accentColor` | CSS colours |
| `strokeWidth`, `cornerRadius`, `fontSize` | Numbers, 0 or more |
| `fontFamily` | A font family name |
| `fontWeight` | 100 to 900 in steps of 100, or `normal`, `bold`, `lighter`, `bolder` |
| `dash` | A dash pattern: numbers, 0 or more, such as `[6, 4]` |
| `opacity` | 0 to 1 |
| `color` | Another name for `textColor` |
| `forceLabel` | Edges only (5.3) |

`accentColor` colours the header and PK badges of ER tables, the stripe of org chart cards and the title bar of UI flow screens and modals.

`styles` maps names (the characters an element ID allows) to style objects. A style name that an element uses MUST be in `styles`.

`palette` sets colours for elements that don't set their own: `background`, `foreground` (the text of nodes and frames), `nodeFill`, `nodeStroke`, `edgeStroke`, `accentPrimary` (the `accentColor` of nodes that set none), `accentSecondary` (no effect: nothing draws a second accent yet), `mutedForeground` (edge label text), `containerFill` and `containerStroke` (frames and groups). In the reference engine an element's style wins over the palette, and the palette wins over the drawing theme. The schema accepts other keys in `palette`, and implementations ignore them.

### 3.6 Assets

`assets` maps asset IDs to image entries: `kind` (`"image"`), `mimeType`, `uri`, and optional `integrity` and `metadata`. An `image` element names its asset with `assetRef`, and that asset MUST be in `assets`.

The reference engine warns about an asset whose `uri` is an `http` or `https` URL, and doesn't fetch it. Give images as `data:` URIs.

## 4. Element IDs

Every element has an `id`: 1 to 256 characters from `A-Z a-z 0-9 . _ : -`, compared exactly, case included. Element IDs MUST be unique within a document, across all element kinds.

The author chooses each ID, and it stays with its element. Edges, groups, frames, relative layouts, constraints and operations all refer to elements by ID, and revisions are compared element by element by ID. The reference engine never assigns, changes or reuses an element ID by itself: an element keeps its ID in every revision until an operation deletes it. After a delete, a later operation can create a new element with the same ID.

Constraint IDs (section 7) use the same characters. They are optional, and a constraint ID MUST be unique among the document's constraints.

## 5. Elements

`elements` is an array. Each element has a `kind`: `node`, `edge`, `group`, `frame`, `text`, `image` or `guide`.

### 5.1 Fields every element has

| Field | Meaning |
|---|---|
| `id` | The element's ID (section 4). Required. |
| `kind` | The element kind. Required. |
| `name` | A name for people and programs. The reference engine doesn't draw it. |
| `zIndex` | An integer paint order: higher draws on top. |
| `locked`, `visible` | Booleans. |
| `style` | A style name or an inline style object (3.5). |
| `metadata` | Any object, for provenance and annotations. |
| `extensions` | An open object (section 8). |
| `x`, `y`, `width`, `height` | Shorthand for an absolute layout (6.2). |

When `zIndex` is absent, the reference engine assigns one by kind, counting up in array order within each kind: frames from 0, groups from 100, edges from 200, nodes from 300, text from 400, images from 500 and guides from 600. So frames draw behind edges, and edges behind nodes.

`visible: false` hides an element. The reference engine still lays it out, so it keeps its space, but doesn't draw it, or any edge with an end at it. A hidden frame hides only its box and title, and a hidden group only its title, not their members.

The reference engine keeps `locked` but doesn't act on it: it edits and draws an element whatever its value.

### 5.2 Nodes

A node is a shape with a label. Required fields: `id`, `kind` and `nodeType`.

| Field | Meaning |
|---|---|
| `nodeType` | What the node is (below). |
| `shape` | `rect`, `roundedRect`, `ellipse`, `diamond`, `cylinder`, `bucket`, `pill`, `hexagon`, `parallelogram`, `cloud`, `box3d`, `textBox` or `iconBox`. |
| `label` | The text. `\n` starts a new line. |
| `layout` | Position and size (section 6). |
| `ports` | Named connection points (below). |
| `icon` | A `family:id` key, such as `lucide:database`, or an image URL. |
| `textStyle` | `wrap` (`none`, `word` or `character`), `align` (`left`, `center` or `right`) and `verticalAlign` (`top`, `middle` or `bottom`). |

Generic node types: `generic`, `service`, `database`, `queue`, `actor`, `note`, `decision`, `document`, `process`, `container`, `externalSystem`, `storage`, `objectStorage`, `event` and `iconNode`.

Family node types:

- `workflow.start`, `workflow.end`, `workflow.task`, `workflow.decision`, `workflow.gateway`, `workflow.gatewayParallel`, `workflow.gatewayInclusive`, `workflow.gatewayEvent`, `workflow.subprocess`, `workflow.timer`, `workflow.error`, `workflow.intermediateTimer`, `workflow.intermediateMessage`, `workflow.intermediateError`, `workflow.intermediateThrowMessage`, `workflow.terminate`, `workflow.messageStart` and `workflow.messageEnd`;
- `er.entity`, `er.weakEntity`, `er.attribute` and `er.relationship`;
- `sequence.lifeline`, `sequence.activation`, `sequence.actor` and `sequence.note`;
- `network.server`, `network.router`, `network.switch`, `network.firewall`, `network.cloud` and `network.client`;
- `org.person`, `org.role` and `org.department`;
- `uiflow.screen`, `uiflow.modal`, `uiflow.entry` and `uiflow.exit`.

When `shape` is absent, the reference engine sets it from `nodeType` and stores it:

| Shape | Node types |
|---|---|
| `roundedRect` | `service`, `queue`, `actor`, `process`, `externalSystem`, `workflow.task`, `workflow.subprocess`, `org.person`, `org.role`, `uiflow.screen`, `uiflow.modal` |
| `cylinder` | `database`, `storage` |
| `bucket` | `objectStorage` |
| `ellipse` | `event`, `er.attribute`, `uiflow.entry`, `uiflow.exit`, and the workflow events: `workflow.start`, `workflow.end`, `workflow.timer`, `workflow.error`, `workflow.intermediate*`, `workflow.terminate`, `workflow.messageStart`, `workflow.messageEnd` |
| `diamond` | `decision`, `workflow.decision`, `workflow.gateway*`, `er.relationship` |
| `cloud` | `network.cloud` |
| `iconBox` | `iconNode` |
| `rect` | Every other type |

When `textStyle` or one of its fields is absent, a node's text has `wrap: "none"`, `align: "center"` and `verticalAlign: "middle"`.

A port has an `id` (unique on its node), a `side` (`top`, `right`, `bottom` or `left`), an `offset` along that side from 0 to 1, a `label` and `metadata`. An edge end names a port with `portId` (5.3). The port IDs `left`, `right`, `top`, `bottom` and `center` are always accepted, declared or not. Only nodes have ports. The reference engine checks port references, but it chooses where each edge meets a node itself and doesn't draw ports.

The reference engine has its own icon registry. An icon key it doesn't know is kept, and the node draws nothing in the icon's place.

#### ER tables

An `er.entity` or `er.weakEntity` node lists its columns in `extensions.columns`. Each column has a `name` (required) and can have `type`, `pk`, `fk`, `unique`, `nullable` and `comment`. `key: "PK"` and `key: "FK"` are other ways to write `pk: true` and `fk: true`; a column that is both needs `pk` and `fk`. The reference engine draws each column as a row, and stores `key` as `pk` or `fk`.

### 5.3 Edges

An edge connects two elements. Required fields: `id`, `kind`, `from` and `to`.

| Field | Meaning |
|---|---|
| `from`, `to` | The two ends (below). |
| `edgeType` | `default` when absent, or a family edge type (3.3). |
| `label` | The text. `\n` starts a new line. |
| `endpoints` | `startArrow` and `endArrow`: `none`, `triangle`, `diamond`, `circle`, `bar`, `openTriangle` or `openCircle`. |
| `router` | `straight`, `orthogonal`, `curved` or `polyline`. |
| `waypoints` | Points (`x`, `y`) kept with the edge. |

An end has an `elementId` (required) and can have:

- `portId`, a port on that node, or `anchor`, a `side` and an `offset` from 0 to 1. An end MUST NOT have both.
- `column`, the name of a column of that entity (ER tables only).
- `cardinality`: `one`, `many`, `zero-or-one`, `zero-or-many` or `one-or-many`.

`elementId` MUST name an element in the document. An edge connects nodes, frames or groups. An end at a text, image or guide element is valid, and so is an edge from an element to itself; the reference engine warns about both.

When an arrowhead is absent, it comes from `edgeType`: `workflow.messageFlow` has `openCircle` at the start and `openTriangle` at the end; `sequence.async` and `sequence.reply` have none at the start and `openTriangle` at the end; every other type has none at the start and `triangle` at the end. The reference engine stores the arrowheads it fills in.

The reference engine routes every edge itself. It draws `polyline` like `orthogonal`, keeps `waypoints` without drawing through them, and doesn't use `portId` or `anchor` to place an end.

#### ER edges

On an edge between ER tables, the reference engine works out more for each end, every time it lays the document out, and stores none of it:

- An end with `column` meets that column's row. A `column` that names no column of the entity is a warning, and the end meets the entity's side instead.
- An edge whose `label` is exactly the name of a column of either end's entity anchors that end at the column. The label isn't drawn then, unless the edge's style has `forceLabel: true`.
- An end with no column meets the entity's only primary-key column, when it has exactly one.
- An end without `cardinality` that meets a column gets one from the column. For a foreign-key column: `zero-or-one` when it is unique and nullable, `one` when it is unique, `zero-or-many` when it is nullable, and `many` otherwise. A column is unique when it has `unique: true` or is its entity's only primary-key column. For a primary-key column that isn't a foreign key: `one` when it is the only one, and `many` when the key has several columns.

A `cardinality` the author sets always wins.

### 5.4 Groups

A group gathers elements. Required fields: `id`, `kind` and `children`, an array of element IDs. A group can also have a `title` and a `layout` with `padding`, `titleBar` and `autoResize`.

Each ID in `children` MUST name an element in the document. An element MUST NOT be in more than one group. A group MUST NOT contain itself, directly or through other groups and frames, and a guide MUST NOT be a child of a group.

The reference engine draws no box or border for a group, and doesn't use a group's `layout`. It draws a group's `title` without a position, at the canvas origin. Moving a group moves its members (10.5).

### 5.5 Frames

A frame is a container drawn with a title. Required fields: `id`, `kind` and `children`.

| Field | Meaning |
|---|---|
| `children` | IDs of the elements inside the frame. |
| `title` | The text drawn on the frame. |
| `layout` | Optional. With one, the frame keeps that box, unless a member that grew or was pushed would cross its border (6.6). Without one, it fits around its members. |
| `containerType` | `generic` when absent, or `pool`, `lane`, `subprocess`, `partition`, `package`, `boundary` or `fragment`. |
| `laneDirection` | `vertical` or `horizontal`: how a pool's lanes are arranged. |

Each ID in `children` MUST name an element in the document. An element MUST NOT be in more than one frame. To nest frames, list the inner frame in the outer frame's `children`. A container MUST NOT contain itself, directly or through other containers, and a guide MUST NOT be a child of a frame.

In the reference engine, lanes stack top to bottom when `laneDirection` is `vertical` or absent, and left to right when it is `horizontal`. A `fragment` frame is a sequence diagram's combined fragment: its `extensions.operator` (such as `alt`, `loop` or `opt`) is drawn in its corner, and `ref` is drawn when it is absent.

### 5.6 Text

A text element is a block of text. Required fields: `id`, `kind`, `text` and `layout`. It can have a `textStyle`, as nodes do. When `textStyle` or one of its fields is absent, the text has `wrap: "none"`, `align: "left"` and `verticalAlign: "top"`.

### 5.7 Images

An image element draws an asset. Required fields: `id`, `kind`, `assetRef` and `layout`. `fit` says how the image fills its box: `contain`, `cover`, `stretch` or `original`. The reference engine draws `original` like `contain`.

### 5.8 Guides

A guide is a horizontal or vertical line across the canvas. Required fields: `id`, `kind`, `orientation` (`horizontal` or `vertical`) and `position`, the guide's y coordinate when it is horizontal and its x coordinate when it is vertical.

### 5.9 The order of elements

The order of `elements` matters in two places. In a sequence diagram, messages (edges) and notes appear from top to bottom in array order. Among elements with the same `zIndex`, the reference engine draws in a fixed order of kinds. Operations that create elements add them at the end of the array.

## 6. Positions and sizes

### 6.1 Coordinates

Positions and sizes are in pixels. x grows to the right and y grows down. A box's `x` and `y` are its top-left corner.

### 6.2 Absolute layout and the shorthand

An absolute layout is `{ "mode": "absolute", "x", "y", "width", "height" }`, with optional `rotation` and `autoSize`. `width` and `height` are 0 or more.

On a node or a frame, `x`, `y`, `width` and `height` are shorthand for an absolute layout. When the element has a `layout`, the reference engine ignores the shorthand. Otherwise it turns the shorthand into a layout, and fills any of the four that is missing: `x` and `y` with 0, `width` with 150 and `height` with 60. The schema accepts the shorthand on every element, and the reference engine ignores it on the other kinds.

`autoSize` is `none` (the default) or `content`; section 6.5 says what `content` does. The reference engine keeps `rotation` but doesn't rotate anything.

### 6.3 Relative layout

A relative layout places an element next to another: `{ "mode": "relative", "place": { ... } }`, with optional `preferredSize` (`width` and `height`) and `rotation`. `place` has exactly one of `above`, `below`, `leftOf`, `rightOf` and `inside`, each naming an element, and can have a `gap`, `alignX` (`left`, `center` or `right`) and `alignY` (`top`, `center` or `bottom`).

The element `place` names MUST exist, and MUST NOT be the element itself. Relative layouts MUST NOT form a cycle. `inside` MUST name a group, a frame or a node whose `nodeType` is `container`.

A relatively placed element follows the element it names. The reference engine never stores a position for it, `autoLayout` doesn't move it, `moveElement` refuses it, and `resizeElement` sets its `preferredSize` (10.5). When the element it names is deleted, it gets an absolute layout where it was drawn (10.4).

### 6.4 Stored positions

A document stores where its nodes are. A node is placed when it has an absolute layout that isn't at exactly `x` 0, `y` 0 and whose `width` or `height` is above 0. Any other node with an absolute layout is unplaced, and a node with no `layout` counts as an absolute layout at `x` 0, `y` 0.

The reference engine places each unplaced node when it lays the document out: around the placed nodes, which it doesn't move, or every node when none is placed. When it stores the document, it writes each box it chose into the node's `layout`, with the numbers exactly as computed. From then on the node keeps that position. It moves only when an operation moves it (`updateNode` with a new `layout`, `moveElement`, `resizeElement`, `autoLayout`, `addConstraint` with an `order` constraint, `updateScene` with a new `layoutStrategy` or `diagramFamily`, or `restoreRevision`), when an edit changes what it shows (6.5), when a node that grows pushes it (6.5), or when an edit changes its frame (6.6).

Sequence diagrams are the exception. The reference engine lays them out from their messages every time and writes nothing back, so their nodes stay at `x` 0, `y` 0.

### 6.5 Sizes

A placed node's `width` and `height` are a minimum. In the reference engine, a node whose label or icon needs more room is drawn larger. The drawn box is in `computed.layout` (section 13).

A node whose label wraps (`textStyle.wrap` is `word` or `character`) keeps its width instead, and its label wraps to fit. With `autoSize: "content"`, the reference engine also gives it the height its wrapped label and icon need, growing it down from its top edge, and the stored `height` stays the minimum. `content` changes nothing for a label that doesn't wrap, or for nodes that lay out their own text: org chart people and roles, UI flow screens and modals, ER tables with columns, sequence nodes, workflow events and gateways, and UI flow entry and exit points.

The size of an unplaced node is the reference engine's to choose. When it places one, it stores the size the node's content needs, whatever size the layout gave, so a wrapped label is measured on one line. The exception is a node whose label wraps and whose layout has `autoSize: "content"`: it keeps its width, and its height stays the minimum. `resizeElement` refuses an unplaced node (10.5).

When an edit changes what a placed node shows (its label, shape, node type, icon, ER columns, text style, or the font size or weight its style gives it) and doesn't set its `layout`, the reference engine measures the node again. It stores the size the new content needs, which can be smaller than before, and keeps the node in its place in the layout. A node whose label wraps isn't measured again.

When a placed node is drawn bigger than its stored box, whatever the reason (a new label, `autoSize: "content"`, a layout smaller than its content, or `resizeElement`), the reference engine makes room for it. The nodes in its way move just far enough to keep the gap they had, at most the spacing (`nodeSpacing` across the flow, `rankSpacing` along it), and keep their order; a frame in the way moves whole, with its members. Nodes push only what shares their frame, lane or the top level: a frame, a lane and a pool without lanes each make room inside themselves, and groups don't count. A frame without a `layout` grows with its members and pushes what is around it, and a frame with a `layout` grows where a member that grew or was pushed would cross its border (6.6). The engine stores each node it moved at its new position, and the grown node at the box it is drawn with, apart from an `autoSize: "content"` height, which stays the minimum. A node drawn bigger with nothing in its way pushes nothing, and keeps its stored size unless an edit measured it again (above). A smaller node pulls nothing back. Nodes that already overlap in the stored document stay where they are, and the overlap is a warning (`NODE_OVERLAP_DETECTED`). Sequence diagrams don't make room: they are laid out from their messages (6.4).

### 6.6 Frames and the canvas

A frame without a `layout` fits around its members every time the document is laid out, and the reference engine stores it without a `layout`. It doesn't move placed members to fit, with one exception. When an edit puts placed nodes in a frame or takes them out of one (`createFrame` around them, or deleting their frame), and keeping them where they are would make a frame without a `layout` overlap another frame or a node that isn't its member, or make one of them overlap a frame that doesn't hold it, the reference engine places those nodes again (6.4). A frame with a `layout` keeps that box, except that it grows where a member that grew or was pushed would cross its border (6.5); a lane, and a pool with its own `layout`, don't grow. `autoLayout` without `elements` drops the `layout` of most frames (10.6). A member outside a frame's box for any other reason gets a warning (`NODE_OUTSIDE_FRAME`).

The canvas size is a minimum (3.2).

## 7. Constraints

`constraints` is an array of constraint objects. Each has a `type` and can have an `id` and `metadata`. An `id` MUST be unique among the document's constraints; `removeConstraint` removes a constraint by it.

| Type | Fields | Meaning in the reference engine |
|---|---|---|
| `align` | `axis`, `elements` | Moves the elements so their centres line up on `axis` (`x` or `y`), at the average centre. |
| `distribute` | `axis`, `elements`, `gap` | Sorts the elements by position along `axis` and puts each `gap` pixels after the one before, starting from the first. |
| `stack` | `axis`, `elements`, `gap` | Puts the elements one after another along `axis`, in the order listed, `gap` pixels apart, starting from the first. |
| `grid` | `elements`, `columns`, `columnGap`, `rowGap` | Lays the elements out left to right and top to bottom, `columns` per row (2 when absent), from the first element's position. |
| `sameSize` | `elements`, `dimension` | Gives every element the first element's `width`, `height` or `both` (the default). |
| `contain` | `containerId`, `elements`, `padding` | Sets the container's box to the elements' bounding box plus `padding`. |
| `centerIn` | `containerId`, `elements` | Centres each element in the container, on both axes. |
| `minGap` | `axis`, `elements`, `gap` | Sorts the elements along `axis` and pushes each one at least `gap` pixels past the one before. |
| `noOverlap` | `elements` | Sorts the elements by `x` and pushes each one that overlaps the one before to the right. |
| `order` | `source`, `target`, `relation` | Places `source` relative to `target` when the engine places nodes (below). |

`elements` lists element IDs, at least one. `gap`, `padding`, `columnGap` and `rowGap` are pixels, 0 or more and 0 when absent; `minGap` requires its `gap`. Every type but `contain` and `centerIn` relates its elements to each other, so it needs two to mean anything; the reference engine warns about one with fewer (`CONSTRAINT_INSUFFICIENT_MEMBERS`). `contain` and `centerIn` relate their elements to the container, and need one.

Every element a constraint names, `containerId` included, MUST exist, and an `order` constraint's `source` and `target` MUST be different elements.

The reference engine applies every type but `order` to the drawing, in array order, after it places the nodes, each time it lays the document out. These constraints change the boxes it draws, and don't change the stored position of a node that is already placed.

An `order` constraint's `relation` says where `source` goes relative to `target`: `leftOf`, `rightOf`, `above`, `below` or `sameRank` (the same layer of a layered layout). The reference engine applies it when it places nodes (6.4). It doesn't move nodes that are already placed, except the two nodes of an `order` constraint that `addConstraint` adds (10.7). Order constraints that form a cycle, or that put two nodes both in the same rank and in a direction, are warnings, and the engine drops the ones it can't satisfy.

`removeConstraint` removes a constraint by its `id`, so give an `id` to any constraint you may want to remove. Deleting an element removes or changes the constraints that name it (10.4).

## 8. Extensions and metadata

`extensions` holds data the schemas don't define. The document, each element and each operation can carry an `extensions` object, and the operations envelope has one too. The reference engine keeps every key of a document's and its elements' `extensions`, including keys it doesn't know. An operation's `extensions` become the element's on `createNode`, `createEdge`, `createFrame`, `groupElements`, `updateNode` and `updateEdge` (10.3, 10.4), and are ignored on the others.

The reference engine reads these keys:

| Where | Key | Meaning |
|---|---|---|
| Document | `showLifelineFeet` | Sequence diagrams: `false` stops drawing a foot bar at the bottom of each lifeline. |
| Document | `autoGroup` | Architecture diagrams: `"byNodeType"` draws a band behind each set of nodes of the same `nodeType`. The engine skips it when frames already hold those nodes, when every node has the same type, or when a type's nodes are scattered. |
| `er.entity`, `er.weakEntity` | `columns` | The table's columns (5.2). |
| `sequence.note` | `anchor` | The lifeline or actor ID the note sits over, or an array of two or more IDs it spans. |
| `fragment` frame | `operator` | The combined fragment's operator (5.5). |
| `fragment` frame | `operands` | An `alt` fragment's sections: an array of `{ "y", "guard" }`. Each draws a dashed line across the fragment at `y`, in drawing coordinates, with `[guard]` under it. |
| Document | `layoutHints.primaryDirection` | `TB`, `BT`, `LR` or `RL`: the direction of an org chart's tree (`TB` when absent), and of the layered layout, where it wins over `layoutStrategy.direction` (3.4). |

`metadata` holds provenance and annotations: who made an element, from what. Implementations keep it and don't act on it.

The reference service keeps its owner's account ID in the document's top-level `extensions` as `ownerId`, and leaves it out of the copy anyone else can read.

## 9. Validation

### 9.1 Shape

A document MUST be valid against `scene.schema.json`, and an envelope against `ops.schema.json`. Schema validity covers each object's fields, types, enumerations, ranges and required properties.

### 9.2 Document rules

A valid document MUST also follow these rules, which a schema can't express. The reference engine rejects a document that breaks one, with the error code shown.

| Rule | Error code |
|---|---|
| Element IDs are unique (section 4). | `ELEMENT_ID_DUPLICATE` |
| A style name an element uses is in `styles`. | `STYLE_NOT_FOUND` |
| Each ID in a group's or frame's `children` names an element. | `CHILD_ELEMENT_NOT_FOUND` |
| An edge has both ends, and each end's `elementId` names an element. | `EDGE_ENDPOINT_MISSING`, `EDGE_ENDPOINT_ELEMENT_NOT_FOUND` |
| A `portId` names a port on its node, or is `left`, `right`, `top`, `bottom` or `center`. | `PORT_NOT_FOUND` |
| Port IDs are unique on their node. | `PORT_ID_DUPLICATE` |
| A container doesn't list itself in `children`. | `SELF_CONTAINMENT` |
| No container contains itself through other containers. | `CONTAINMENT_CYCLE` |
| A guide isn't in a group or frame. | `INVALID_CHILD_KIND_FOR_CONTAINER` |
| An element is in at most one group. | `MULTIPLE_GROUP_MEMBERSHIP` |
| An element is in at most one frame. | `MULTIPLE_FRAME_MEMBERSHIP` |
| A relative layout names an element other than itself. | `RELATIVE_TARGET_NOT_FOUND`, `RELATIVE_SELF_REFERENCE` |
| Relative layouts form no cycle. | `RELATIVE_LAYOUT_CYCLE` |
| `inside` names a group, a frame or a `container` node. | `INVALID_INSIDE_TARGET` |
| Constraint IDs are unique (section 7). | `CONSTRAINT_ID_DUPLICATE` |
| Every element a constraint names exists. | `CONSTRAINT_TARGET_NOT_FOUND` |
| An `order` constraint names two different elements. | `ORDER_CONSTRAINT_SELF_REFERENCE` |
| An image's `assetRef` is in `assets`. | `ASSET_NOT_FOUND` |

The reference engine also limits size, with the error `CONTENT_LIMIT_EXCEEDED`: 1,000 elements, edges included, containers nested 10 deep, 10,000 characters in one label or text, and a canvas of 20,000 by 20,000 pixels. Another implementation can set other limits.

### 9.3 Diagnostics

A diagnostic reports one problem. Its shape is the scene schema's `diagnostic`: a `code`, a `message`, a `severity`, and an optional `path`, a JSON Pointer to the field concerned.

| Severity | Meaning |
|---|---|
| `fatal` | Processing stopped. The document or edit is rejected. |
| `error` | The document or edit is invalid and is rejected. |
| `warning` | The document is valid, and something in it is likely a mistake. |
| `info` | Something the reader may want to know. |

A document with an `error` or `fatal` diagnostic is invalid. An implementation MUST NOT store a document it is sent when checking it gives an `error` or `fatal` diagnostic, or apply an edit when checking the edit gives one or the edit's result has one its base didn't have (10.2). Warnings and info never block either.

Codes are strings in capitals with underscores. The error codes in 9.2 and 10.2 are the reference engine's, and other implementations can use them. The reference engine also returns warnings: rules for each diagram family (such as a workflow with no start node), and problems its layout notices (such as overlapping nodes or text that doesn't fit). Those warnings aren't part of this spec.

## 10. Operations

### 10.1 The envelope

An edit is an operation envelope:

| Field | Required | Meaning |
|---|---|---|
| `schemaVersion` | Yes | `"0.1"`. |
| `sceneId` | Yes | The diagram's `scene.id`. |
| `baseRevision` | Yes | The revision the edit was planned against (section 12). |
| `ops` | Yes | The operations, applied in order. |
| `message` | No | One line saying what the edit does, kept with the revision it makes. |
| `requestId` | No | The client's ID for this edit, for safe retries: 1 to 255 printable ASCII characters, no spaces. |
| `extensions` | No | An open object. |

Each operation is an object with an `op` field naming it, plus the fields below. Every operation can also carry `metadata` and `extensions`.

A service can take `sceneId` from somewhere else, such as a URL path, and fill in `schemaVersion` and `sceneId` itself. The reference service does.

### 10.2 Applying an envelope

An implementation applies the operations in order, each to the document that the operations before it left, and the edit applies whole or not at all. It MUST check each operation against the document it runs on, and reject the whole edit if any check fails. The element an operation changes MUST exist when the operation runs. An element or style an operation only names (an edge's ends, `children`, a style name, a relative layout's target, a constraint's elements) MUST exist when the operation runs or be created by a later operation of the same envelope, so the order of the operations doesn't matter for those.

| Rule | Error code |
|---|---|
| A create operation's `id` isn't in use, by an element or by an element created earlier in the envelope. | `OP_ID_ALREADY_EXISTS` |
| The element an update, delete, move, resize, style, text or ungroup operation names exists. So does each element `autoLayout` names. | `OP_TARGET_NOT_FOUND` |
| `updateNode` names a node, `updateEdge` an edge, `ungroupElements` a group, and `autoLayout` only nodes. | `OP_TARGET_KIND_MISMATCH` |
| The ends of `createEdge` and `updateEdge` name elements that exist or that the envelope creates. | `EDGE_ENDPOINT_ELEMENT_NOT_FOUND` |
| The `children` of `groupElements` and `createFrame` name elements that exist or that the envelope creates. | `CHILD_ELEMENT_NOT_FOUND` |
| A style name an operation gives is in `styles`, or an `updateScene` of the envelope adds it. | `STYLE_NOT_FOUND` |
| A relative layout an operation gives names an element that exists or that the envelope creates. | `RELATIVE_TARGET_NOT_FOUND` |
| The elements an `addConstraint` constraint names exist or the envelope creates them, and its `id` isn't in use. | `CONSTRAINT_TARGET_NOT_FOUND`, `CONSTRAINT_ID_DUPLICATE` |
| `setText` names a node, an edge or a text element. | `TEXT_OPERATION_KIND_INVALID` |
| `resizeElement` names a node, a frame, an image or a text element. | `RESIZE_NOT_SUPPORTED` |
| `moveElement` doesn't name an edge. | `MOVE_NOT_SUPPORTED` |
| `moveElement` doesn't name an element with a relative layout. | `MOVE_NOT_ALLOWED_FOR_RELATIVE_LAYOUT` |
| `restoreRevision` is the only operation in its envelope. | `RESTORE_NOT_ALONE` |
| `restoreRevision` names a revision of 1 or more, and that revision exists. | `RESTORE_REVISION_INVALID`, `REVISION_NOT_FOUND` |

The reference engine also refuses moves and resizes it would undo, with the codes in 10.5.

The document an edit produces is a document like any other, and the rules of section 9 apply to it. An implementation MUST NOT apply an edit whose result breaks a rule of 9.2 that the document it ran on didn't break, and rejects it with that rule's error code. An error the document already had doesn't block an edit that leaves it as it was. The reference engine checks the result as the operations left it, before it places nodes. A `restoreRevision` brings back an earlier revision exactly, and its result isn't checked.

### 10.3 Creating elements

`createNode` adds a node. Required: `id` and `nodeType`. Optional: `name`, `zIndex`, `locked`, `visible`, `style`, `shape`, `label`, `layout`, `ports`, `icon`, `textStyle`, the shorthand `x`, `y`, `width` and `height`, and `metadata` and `extensions`, which become the node's. A node created without a `layout` is unplaced, and the next layout places it (6.4).

`createEdge` adds an edge. Required: `id`, `from` and `to`. Optional: `name`, `zIndex`, `locked`, `visible`, `style`, `label`, `edgeType`, `endpoints`, `router`, `waypoints`, `metadata` and `extensions`. Missing arrowheads come from `edgeType`, as in a document (5.3).

`createFrame` adds a frame. Required: `id` and `children` (at least one). Optional: `name`, `zIndex`, `locked`, `visible`, `style`, `title`, `layout`, `containerType`, `laneDirection`, `metadata` and `extensions`. Without a `layout`, the frame fits around its members (6.6).

`groupElements` adds a group. Required: `id` and `children` (at least one). Optional: `name`, `zIndex`, `locked`, `visible`, `style`, `title`, `layout` (`padding`, `titleBar`, `autoResize`), and `metadata` and `extensions`, which become the group's.

Each of these adds its element at the end of `elements`. When `zIndex` is absent, the reference engine assigns the next value for the element's kind (5.1).

### 10.4 Changing elements

`updateNode` changes a node, and `updateEdge` an edge. Required: `id`, and at least one other field. They take the fields their create operation takes, except `id`, and `updateNode` doesn't take the shorthand. Each field given replaces the element's field whole: `layout`, `style`, `ports`, `from`, `to`, `endpoints`, `metadata` and `extensions` aren't merged. To add an ER column, send the whole `extensions` object with the new `columns` array. On `updateNode`, `icon: null` removes the icon, and `textStyle` is merged into the node's field by field, as with `setText`.

`setText` sets the text of a node (its `label`), an edge (its `label`) or a text element (its `text`). Required: `id` and `text`. Its optional `textStyle` is merged into the element's, field by field; edges have no `textStyle` and ignore it.

`setStyle` changes an element's style. Required: `id`, and exactly one of `style` and `styleOverride`. `style`, a style name or a style object, replaces the element's style. `styleOverride`, a style object, is merged into what the element looks like now: when the element uses a named style, that style's properties are copied in first, and the element ends up with an inline style. Setting the style name an element already has is a warning.

`deleteElement` removes an element. Required: `id`. It also removes every edge with an end at that element, and leaves nothing pointing at what it removed:

- the IDs leave every group's and frame's `children`;
- a constraint that names a removed element as its `containerId`, `source` or `target` is removed, and so is one left with fewer elements than it needs (section 7); any other constraint loses the removed elements from its `elements`;
- an element whose relative layout names a removed element gets an absolute layout at the box it was last drawn with, so it stays where it is. In the reference engine, one not drawn yet gets an absolute layout at `x` 0, `y` 0, so the next layout places it.

The members of a deleted group or frame stay. The reference engine reports each constraint it removes (`CONSTRAINT_REMOVED`) and each layout it makes absolute (`RELATIVE_LAYOUT_FIXED`) as `info` diagnostics with the edit's result.

`ungroupElements` removes a group and keeps its members. Required: `id`, which MUST name a group.

### 10.5 Moving and resizing

`moveElement` moves an element by `delta` (`dx`, `dy`). Required: `id` and `delta`. It works on elements with an absolute layout, on frames and groups, and on guides, which move along the axis they mark. Moving a frame or a group moves its members by the same delta, and the members of frames and groups inside it. A member that is unplaced (6.4) stays unplaced, and a frame without a `layout` fits around its members where they are now. An edge MUST NOT be moved, nor an element with a relative layout. The reference engine also refuses a sequence diagram's nodes and frames, which it lays out from the messages (`MOVE_NOT_SUPPORTED`).

`resizeElement` gives a node, frame, image or text element a new `size` (`width`, `height`). Required: `id` and `size`. Optional: `anchor`, a side (`top`, `right`, `bottom` or `left`) that stays put: the element grows or shrinks away from that side and stays centred along it. Without `anchor`, the top-left corner stays put. A placed node made bigger pushes the nodes in its way, keeping the gap they had, at most the node spacing (6.5); a smaller one pulls nothing back. Resizing a frame without a `layout` gives it one: the box it was last drawn with, at the new size. Resizing an element with a relative layout sets its `preferredSize` and keeps its placement, and takes no `anchor`.

The reference engine refuses a resize it would undo. An unplaced node (6.4) gets the size its content needs when it is placed, and a frame without a `layout` created in the same edit hasn't been drawn, so resize either in a later edit (`RESIZE_NOT_PLACED`). It sizes a sequence diagram's nodes and frames from the messages (`RESIZE_NOT_SUPPORTED`).

### 10.6 Layout

`autoLayout` has the implementation place nodes again, and stores where it puts them. Optional: `elements`, a list of node IDs.

- With `elements`, the reference engine unplaces the listed nodes, by setting their positions to `x` 0, `y` 0 and keeping their sizes. The next layout places them next to their neighbours, around the other nodes, which stay where they are.
- Without `elements`, in the diagrams it lays out with its layered layout (3.4), the reference engine first drops the `layout` of every frame except pools, lanes and fragments, so that each fits around its members and is laid out with them. Then it unplaces every node except the members of frames that still have their own `layout`, at any depth, which keep their positions: pools, lanes and fragments, and in org charts and network diagrams, whose layouts place nodes without regard to frames, every frame. When no node keeps a position, the result is a fresh layout, and the canvas size is set to fit.

`autoLayout` doesn't move nodes with a relative layout, and does nothing in a sequence diagram. When every node lands where it already was, the edit changes nothing (12.4).

### 10.7 Constraints

`addConstraint` adds `constraint`, a constraint object (section 7), at the end of `constraints`. When it adds an `order` constraint whose two nodes are both placed and don't already stand in that relation, the reference engine unplaces both, and the next layout places them in order around the other nodes. Nodes already in order stay where they are. This doesn't happen in sequence diagrams.

`removeConstraint` removes the constraint whose `id` is the operation's `id`. Removing an ID no constraint has changes nothing.

### 10.8 The scene

`updateScene` changes the document's scene-level fields. Every field is optional, and an absent field is left as it is.

| Field | Effect |
|---|---|
| `title` | Replaces `scene.title`. |
| `brief` | Replaces `scene.brief` (up to 1,000 characters); `null` removes it. |
| `canvas` | Merged into `scene.canvas`: `width`, `height` and `background`. |
| `palette` | Merged into `palette`, colour by colour; `null` removes the palette. |
| `styles` | Merged into `styles`: each named style given replaces the one with that name. |
| `layoutStrategy` | Replaces `layoutStrategy`; `null` removes it. |
| `diagramFamily` | Replaces `diagramFamily`; `null` removes it. |

A `layoutStrategy` or `diagramFamily` that differs from the current one, including its removal, lays the document out again as `autoLayout` without `elements` does (10.6), in the family the document has after the edit. Sending the current value again changes nothing.

### 10.9 History

`restoreRevision` makes the next revision's content equal to an earlier revision's. It MUST be the only operation in its envelope. Give exactly one of:

- `revision`, a revision number, 1 or more;
- `asOf`, a date (`2026-06-30`, meaning the end of that day in UTC) or an ISO 8601 timestamp with an offset (`2026-06-30T14:00:00Z`), meaning the revision that was current at that instant.

The restored content is the earlier revision's exactly, node positions included. History isn't rewritten: the revisions in between stay as they were (section 12).

## 11. The stored form

The reference engine stores documents with their defaults written in, and returns them that way. A document read back can differ from the one sent:

- `shape` is set on every node (5.2), `textStyle` on every node and text element (5.2, 5.6), and `zIndex` on every element (5.1).
- `locked` (false), `visible` (true), `metadata` (`{}`) and `extensions` (`{}`) are set on every element.
- Every node has an absolute or relative `layout`. A node sent without one has `{ "mode": "absolute", "x": 0, "y": 0, "width": 150, "height": 60 }` until it is placed, and then its box (6.4). The shorthand becomes a `layout`, absolute layouts get `rotation` (0) and `autoSize` (`none`), and relative layouts get `place.gap` (0).
- Edges get `edgeType` (`default`), both arrowheads (5.3) and `waypoints` (`[]`). Nodes get `ports` (`[]`), and ports get `offset` (0) and `metadata` (`{}`).
- Frames get `containerType` (`generic`), and groups get `layout` with `padding` (0), `titleBar` (0) and `autoResize` (false). A frame sent without a `layout` stays without one.
- `scene` gets `units` (`px`), a `canvas` with `width`, `height`, `grid` (0) and `snap` (false), `viewport` (`originX` 0, `originY` 0, `zoom` 1), `defaults` and `metadata.tags`.
- The document gets `styles` (`{}`), `assets` (`{}`), `constraints` (`[]`) and `extensions` (`{}`).
- ER columns written with `key` get `pk` or `fk` instead (5.2). In named styles, `color` becomes `textColor`.
- `scene.extensions` and `computed` are dropped.

What the engine infers for ER edge ends (5.3) isn't stored.

## 12. Revisions

### 12.1 Numbering

A stored diagram has a history of revisions numbered from 1. Storing a new diagram makes revision 1. Each edit that changes the diagram makes the next number. An edit never changes an existing revision: going back to earlier content makes a new revision (10.9).

### 12.2 The base revision

`baseRevision` names the revision the edit was planned against. An implementation MUST NOT apply an edit to any other revision.

When `baseRevision` is the current revision, the edit applies to it. When it isn't, the reference engine applies the edit to revision `baseRevision` instead and compares the result with the current revision:

- If the result is the current content, the edit is already there, most often because an earlier attempt of the same edit landed. The edit succeeds, makes no revision, and returns the current revision. Where the engine placed nodes is left out of this comparison, so a retry still matches when its nodes were placed differently.
- Otherwise the edit is rejected as a conflict, with the code `REVISION_CONFLICT`, and nothing changes. Read the current revision, plan the edit again against it, and send it with the new `baseRevision`.

A `baseRevision` above the current revision is always a conflict. Revision numbers start at 1, and so does `baseRevision`.

### 12.3 Retries with `requestId`

A `requestId` names one edit, and MUST be unique within its diagram. The reference engine records it with the revision its edit made, for the life of the diagram, and checks it before `baseRevision`:

- When an edit with the same `requestId` already made a revision, and the two edits are the same (the same `baseRevision`, `ops` and `message`), the retry returns that revision and makes no new one.
- When the edits differ, the retry MUST be rejected, with the code `REQUEST_ID_REUSED`.

An edit that made no revision records nothing, so its `requestId` stays free.

### 12.4 Edits that change nothing

An edit whose result has the same content as its base revision makes no revision. It succeeds and returns the current revision. In the reference engine this covers edits that set fields to the values they have, an `autoLayout` that places every node where it was, and a restore of the current content.

Two documents have the same content when their stored forms (section 11), without `computed`, have the same canonical JSON under [RFC 8785](https://www.rfc-editor.org/rfc/rfc8785). The reference engine compares the SHA-256 hashes of those canonical forms.

## 13. Computed output

`computed` holds what an implementation worked out when it laid the document out. It is output, not input:

| Field | Meaning |
|---|---|
| `layout` | Each drawn element's box, by element ID. |
| `edgePaths` | Each edge's path by ID: its `points` (at least two), and where its label goes (`labelPosition`, `labelWidth`, `labelHeight`). `router` is `polyline` or `curved`, `cornerRadius` rounds the path's corners, and `isFanInFeeder` and `keepFeederArrowhead` mark edges merged into a shared trunk near their target. |
| `diagnostics` | The diagnostics of that layout (9.3). |
| `normalizedHash` | A hash an implementation can set. The reference engine doesn't. |

The boxes in `computed.layout` are where the elements were drawn. They can be larger than the stored sizes (6.5), and the drawing can be shifted from the document's coordinates by one offset shared by every element.

The reference engine works `computed` out again each time it lays a document out, whatever the document carried. It leaves `computed` out of content comparisons (12.4) and out of the documents it returns.

## 14. The reference implementation

Zindex runs the reference implementation of DSP as a hosted service. Its engine validates documents, lays them out, draws them, applies edits and keeps every revision. Its layout and drawing algorithms aren't part of this spec, and the engine and the service are closed source. The schemas it validates against are the files in this repository, served at their `$id` URLs.

The service's own documentation, which covers its API, limits and rendering, is at <https://zindex.ai/docs/>.
