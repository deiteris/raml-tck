# RAML TCK

RAML's Test Compatibility Kit (RAML TCK) provides a way for any RAML processor to
test its compliance with the
[RAML 1.0 Spec](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md).
It contains a set of RAML documents meant to test correct and incorrect usage of
each RAML feature.

A fork of [raml-org/raml-tck](https://github.com/raml-org/raml-tck), which was
**archived on 19 January 2024** and is read-only. Nothing here goes back
upstream, so this fork diverges deliberately rather than carrying patches.

The fixtures are under [`tests/raml-1.0/`](tests/raml-1.0/).

## Naming convention

- `*valid*.raml`: valid RAML file expected to be successfully processed
- `*invalid*.raml`: invalid RAML file with syntax/semantic/spec error(s),
  expected to be unsuccessfully processed (error or exit code returned)

`invalid` contains `valid` as a substring, so a harness matching `*valid*` has to
exclude `*invalid*` explicitly or every negative fixture lands in both sets.

Not every `.raml` file is an entry point. Includes and libraries sit beside the
fixtures that use them and carry neither word in their names.

Discovery is by this convention alone; upstream's `manifest.json` is not kept.

## What this fork changes

### The fixture set tracks the acronis/go-raml copy

Upstream's fixtures were replaced with the customised copy vendored in
[acronis/go-raml](https://github.com/acronis/go-raml). That copy is not a light
touch — measured against upstream it is **63 files modified, 38 removed and 57
added**:

| | Where |
|---|---|
| Added | mostly `Types/` (32) and `Libraries/` (14) |
| Removed | mostly `Types/` (18) and `EdgeCases/` (14), including the `Types/External Types/include-type-xsd/` XML Schema fixtures |
| Modified | spread across the suite |

The two lineages had already diverged in practice: a processor tested against the
go-raml copy was running a different suite from one tested against upstream, and
nothing recorded the difference. This fork puts that copy under version control,
so it is a diff rather than folklore.

### Five fixtures were wrong and are fixed

Each tested nothing, or tested the wrong thing.
[KNOWN-ISSUES.md](./KNOWN-ISSUES.md) records the spec text that settles each.

| Fixture | What was wrong |
|---|---|
| `Annotations/target-locations/valid-response.raml` | Declared `allowedTargets: Method` while applying the annotation to a response. A parser that enforces `allowedTargets` must reject it as written. |
| `Fragments/namedexample-01/examples/*.raml` | Two `!include` targets were named `invalid-one-example.raml` and `valid-multiple-examples.raml`, so a harness picked them up as entry points. Renamed. |
| `Annotations/complex-11/*-multiple-annots.raml` | `pattern: "[a-zA-Z0-9]{8,32}"` is unanchored, so it matched inside the value it was meant to reject and the valid/invalid pair tested nothing. Anchored. |
| `Overlays/override-default/invalid.raml` | Set `default` to the value the master already had, which is not a difference between the trees. Now a different value. |
| `Overlays/double-displayname-override/base1.raml` | Applied `securedBy: x-ttt` without declaring `x-ttt`, so the master was invalid and both fixtures failed for that reason instead of the overlay. Declared. |

A fixture being wrong is a bug in the suite. Fixing it here is what stops every
consumer encoding the same workaround separately.

### Fixtures added for spec requirements the suite did not test

Each directory pins a MUST that no fixture exercised, so a parser could ignore it
and still pass the whole suite.

| Directory | Spec text |
|---|---|
| `Root/baseuri-syntax/` | § Base URI: the value "MUST conform to the URI specification RFC2396 or a Template URI". The invalid fixtures hold a space, a `%` that starts no pct-encoded octet, and a scheme beginning with a digit. `valid.raml` is a template with a port, a pct-encoded octet and a query. |
| `Root/baseuriparameters-08/` | § Base URI: `baseUriParameters` "MUST follow the same structure as the uriParameters node", and § Template URIs and URI Parameters: "Every property in a uriParameters declaration MUST correspond exactly to the name of a URI parameter". The invalid fixtures declare a parameter the base URI does not use, and one with no base URI at all. `valid.raml` also declares the reserved `version`, which the base URI uses. |
| `Methods/querystring-type/` | § The Query String as a Whole: "all base types in type hierarchy of the data type MUST be either a scalar type or the object type, after fully expanding any union type expressions at every level". The invalid fixtures give an inline array, a named array type, and a union with an array member. `valid.raml` covers a scalar, an object with an array property, and an object-or-string union. |

### The runners and the manifest are gone

Upstream shipped per-language runners (`runner/`, in Go, Java, JavaScript, Python
and Ruby), a `manifest.json` with its `genmanifest.js` generator, an npm manifest
for the JavaScript runner, and a GitHub Pages workflow. All removed: they target
parsers that are themselves archived, and a consumer brings its own harness.

## Using it

As a git submodule:

```bash
git submodule add https://github.com/deiteris/raml-tck.git tests/tck/raml-tck
```

The fixtures are then at `tests/tck/raml-tck/tests/raml-1.0/`. Walk it for
`*.raml`, split on the naming convention, and record an expected outcome per
fixture so that progress and regressions are both visible.

Two fixtures reach the network: `Root/include-02/valid-https.raml` and
`invalid-https.raml` both `!include` a gist. A suite that runs them is not
hermetic, and the negative one passes offline for the wrong reason — an
unreachable host and an unregistered URI scheme both produce an error.

`Overlays/` and `Extensions/` are a distinct language feature. A processor that
has not implemented them should skip those directories rather than record
failures.

## Projects using this TCK

* Python: [fastRAML](https://github.com/deiteris/FastRAML) — consumes this fork
  as a submodule, with a ratchet recording the expected result of every fixture.

Upstream additionally listed parsers in Go, JavaScript, Python, Ruby and Java,
along with a [published results page](http://raml-org.github.io/raml-tck/)
generated by `raml-tck-runner`. Both are of historical interest only: the runner
and most of those parsers are archived.

## Contributing

New fixtures are welcome — see [CONTRIBUTING.md](./CONTRIBUTING.md) for the
layout each test case follows. A fix to an *existing* fixture needs the spec text
that settles it, because "a parser disagrees with this fixture" is not by itself
evidence that the fixture is wrong.

## Licence

Upstream states none — there is no `LICENSE` file in
[raml-org/raml-tck](https://github.com/raml-org/raml-tck) — and this fork adds
none, being in no position to grant terms over material published without them.
It is redistributed on the footing every RAML processor has redistributed it: a
conformance suite an archived project published to be run against.

If you are the rights holder and want this changed, open an issue.
