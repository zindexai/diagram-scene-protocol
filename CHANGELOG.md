# Changelog

Each entry is a tag in this repository, named for the Zindex release that first served that version of the spec and schemas. Documents in every version so far carry `"schemaVersion": "0.1"`.

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
