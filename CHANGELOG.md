# Changelog

Each entry is a tag in this repository, named for the Zindex release that first served that version of the spec and schemas. Documents in every version so far carry `"schemaVersion": "0.1"`.

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
