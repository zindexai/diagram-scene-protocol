# Diagram Scene Protocol

The Diagram Scene Protocol (DSP) is a JSON format for diagrams that programs, AI agents in particular, create and then keep editing. A scene document lists a diagram's elements: nodes, edges, frames, text and a few more. Each element has an ID that stays the same from one revision to the next. An edit is a list of typed operations, such as `createNode` or `updateEdge`, sent with the number of the revision it was planned against.

This repository holds the spec, [SPEC.md](SPEC.md), and the two JSON Schemas that define both formats. Zindex stores every diagram as a DSP document.

## The schemas

| File | Official URL | Defines |
|---|---|---|
| [`schemas/scene.schema.json`](schemas/scene.schema.json) | <https://zindex.ai/schemas/scene.schema.json> | Scene documents |
| [`schemas/ops.schema.json`](schemas/ops.schema.json) | <https://zindex.ai/schemas/ops.schema.json> | Operation envelopes (edits) |

Both use JSON Schema draft 2020-12. Each schema's `$id` is its official URL, and the files here are identical to the ones served there. [`examples/`](examples/) has a small scene and an edit to it.

## What the spec covers

SPEC.md defines:

- the scene document: its fields, the seven diagram families and the seven element kinds;
- element IDs, and how documents and edits refer to elements;
- the operation envelope and every operation;
- revisions: base revisions, conflicts, safe retries with `requestId`, and edits that change nothing;
- constraints;
- what stored positions and sizes mean;
- extensions;
- the diagnostic format the schemas carry (`code`, `path`, `message`, `severity`), and the rules a valid document follows beyond its shape, such as unique element IDs and edges whose ends exist.

It doesn't define how to place nodes, route edges or draw a diagram. Those algorithms belong to each implementation, and two implementations can draw the same document differently. It doesn't define an HTTP API either.

## Validating a document

A schema checks a document's shape. These examples validate `examples/scene.json` against the scene schema; use `schemas/ops.schema.json` and `examples/edit.json` to check an edit.

JavaScript, with [Ajv](https://ajv.js.org/) (`npm install ajv ajv-formats`), saved as `validate.mjs`:

```js
import Ajv2020 from "ajv/dist/2020.js";
import addFormats from "ajv-formats";
import { readFileSync } from "node:fs";

const read = (path) => JSON.parse(readFileSync(path, "utf8"));

// strict: false stops Ajv logging that the schemas use unevaluatedProperties
// without a type beside it. Validation is the same either way.
const ajv = new Ajv2020({ allErrors: true, strict: false });
addFormats(ajv);
const validate = ajv.compile(read("schemas/scene.schema.json"));

const scene = read("examples/scene.json");
if (validate(scene)) {
  console.log("valid");
} else {
  for (const error of validate.errors) console.log(error.instancePath, error.message);
  process.exitCode = 1;
}
```

Python, with [jsonschema](https://python-jsonschema.readthedocs.io/) (`pip install jsonschema`):

```python
import json
import sys

from jsonschema import Draft202012Validator

with open("schemas/scene.schema.json") as f:
    schema = json.load(f)
with open("examples/scene.json") as f:
    scene = json.load(f)

errors = list(Draft202012Validator(schema).iter_errors(scene))
for error in errors:
    print("/" + "/".join(map(str, error.absolute_path)), error.message)
print("valid" if not errors else f"{len(errors)} errors")
sys.exit(1 if errors else 0)
```

A document that passes is well formed, and can still break a rule a schema can't express. SPEC.md section 9 lists those rules.

## Versioning

Every document and edit carries `"schemaVersion": "0.1"`, the only value the schemas accept today.

Each tag in this repository, such as `v6.0.0`, names the Zindex release that first served that version of the spec and schemas. A Zindex release that changes neither makes no new tag. [CHANGELOG.md](CHANGELOG.md) says what changed in each tag. While `schemaVersion` is 0.x, a release can change the schemas in ways that reject documents an earlier version accepted, and the changelog says when one does.

## Zindex and other implementations

Zindex's hosted engine is the reference implementation of DSP. It validates documents, lays them out, draws them, applies edits and keeps every revision. SPEC.md says where its behavior goes beyond the spec, and where it doesn't yet do what the spec says. The engine and the hosted service are closed source; their documentation is at <https://zindex.ai/docs/>.

Other implementations are welcome: readers, writers, validators, converters or whole engines. They don't need permission from Zindex.

## License

The spec text and the schemas are licensed under the [Apache License 2.0](LICENSE). See [NOTICE](NOTICE).

## Trademark

"Zindex" is a trademark. The license covers the spec text and the schemas in this repository, not the Zindex name or logo. Section 6 of the license says the same.

## Reporting a spec issue

Open an issue on [this repository](https://github.com/zindexai/diagram-scene-protocol/issues). Name the section of SPEC.md or the schema path (such as `scene.schema.json#/$defs/endpoint`), say what it says and what you expected, and include the smallest document that shows the problem.

Problems with the hosted service, such as an API error or a drawing bug, go to support@zindex.ai instead.
