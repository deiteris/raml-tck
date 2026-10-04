# Known issues in the fixtures

Bugs found in the suite, and what was done about them. Each entry states the spec
text that settles it, because "a parser disagrees with this fixture" is not by
itself evidence that the fixture is wrong.

Issues with a *processor* do not belong here. This file is about the test
documents; what any given implementation gets wrong belongs in that
implementation's repository.

---

## 1. `Annotations/target-locations/valid-response.raml` names the wrong target

**Fixed.**

Every fixture in `tests/raml-1.0/Annotations/target-locations/` follows one
convention: `valid-<target>.raml` declares `allowedTargets: <that target>` and
applies the annotation there. `valid-response.raml` alone declared
`allowedTargets: Method` while applying the annotation directly under a `200:`
response key — a copy of `valid-method.raml` with the application site changed
and the declaration left behind.

Spec § Annotations, Target Locations: `Response` is "a declaration of the
responses node, whose key is an HTTP status code". So a conforming parser that
enforces `allowedTargets` must **reject** the fixture as written, and the sibling
`invalid-response-used-in-api.raml` confirms `Response` is a target the suite
means to test.

This stayed invisible because the reference implementation parses
`allowedTargets` and never enforces it, so every fixture in the directory passed
for it regardless of what it declared.

---

## 2. Two NamedExample *includes* were named as though they were documents

**Fixed.**

`tests/raml-1.0/Fragments/namedexample-01/examples/` held
`invalid-one-example.raml` and `valid-multiple-examples.raml`. Neither is a
document to be parsed on its own: they are the `!include` targets of the two
fixtures in the directory above, and the first is "invalid" only in the context
of the parent that includes it as `examples:`. Parsed standalone it is a
well-formed NamedExample fragment.

A harness that discovers fixtures by the `*valid*` / `*invalid*` convention
therefore picked them up as entry points and expected the first to fail.

Renamed to `one-example.raml` and `multiple-examples.raml`, with both includers
updated, so the convention identifies documents and only documents.

---

## 3. `Annotations/complex-11` is unanchored where it means anchored

**Fixed.**

The pair differs only in `simpleAnnotationValueOnType` versus
`simpleAnnotation_value_on_type`, under:

```yaml
    pattern: "[a-zA-Z0-9]{8,32}"
```

A search finds `simpleAnnotation` — sixteen alphanumerics — inside the
underscored value, so **both files pass and the pair tests nothing**.

The reason this is a fixture bug rather than a parser bug is worth stating,
because the first attempt got it backwards. Spec § String Type defines `pattern`
in one line — "Regular expression that this string MUST match" — and says nothing
about anchoring. What settles it is that the spec writes the anchors *itself*
wherever it means anchored: `^.+@.+\..+$` for `EmailAddress`, `^\d+\-\w+$`,
`^\w{16}$`. Under a full match every one of those is redundant, written three
separate times. So `pattern` is a **search**, and a fixture that depends on it
being a full match relies on something the spec does not say.

Anchored to `^[a-zA-Z0-9]{8,32}$` in both files, which discriminates under a
search *and* under a full match, so the pair no longer silently encodes an
assumption about which is meant.

**Do not anchor `/regex/` property names on the same reasoning.** Those match
*against* a key rather than describing one, and `/^note\d+$/` is how the spec's
own example writes them.

---

## 4. `Overlays/override-default/invalid.raml` changes nothing

**Fixed.**

The master declares:

```yaml
          name:
            type: string
            default: Blah
```

and the overlay, expected to fail, restates it:

```yaml
          name:
            default: Blah
```

Spec § Overlays: after merging, "the tree of nodes in the merged document is
compared with the tree of nodes in the master RAML document", and "any
differences in the documents MUST be only in the nodes listed" in the
allowed-differences table. `default` is not in the table, but restating it with
the same value produces no difference, so an overlay that only does that is
valid. Merging Rules says the same from the other side: a single-value property
is replaced by the extension's value, which here is the master's own.

A processor could only reject the fixture by treating the *presence* of a
disallowed key in the overlay as a violation, which is not what the spec
compares. amf-client-js, which implements the overlay check, accepts the file
as written.

Changed to `default: Bleh`, a real difference in a node the table does not
list, so the fixture now fails for the reason its name gives.

---

## 5. `Overlays/double-displayname-override/base1.raml` is not a valid master

**Fixed.**

The master of both fixtures in the directory applies a security scheme it never
declares:

```yaml
securedBy: x-ttt
```

Spec § Applying Security Schemes: "The value assigned to the securedBy node
MUST be a list of any of the security schemes previously defined in the
securitySchemes node of RAML document root." There is no `securitySchemes`
node, so the master is invalid, and Merging Rules requires
that "Master Tree and Extension Tree are validated". `valid.raml` therefore
fails, and `invalid-add-trait-headers.raml` fails without ever reaching the
trait change it exists to test.

The sibling `Overlays/override-displayname/base.raml` is the same document with
the declaration present:

```yaml
securitySchemes:
   x-ttt:
      type: Digest Authentication
```

Declared the same way here.

---

## 6. `Types/ObjectTypes/pattern-property-chars/invalid-does-not-match-pattern.raml` rejects a key the spec accepts

**Fixed.**

The fixture declared `/a-zA-Z/: number` and expected the example `abc: bar` to
fail. The pattern is the literal text `a-zA-Z`, so `abc` matches no pattern
and is an ordinary additional property, which `additionalProperties` (omitted,
so `true`) allows.

Spec § Property Declarations settles it with its own example: beside
`/^note\d+$/: string`, the instance key `note: 123` is "valid as it does not
match the pattern" (`versions/raml-10/raml-10.md` line 712 in
`https://github.com/raml-org/raml-spec`). Pattern properties restrict the keys
they match; only `additionalProperties: false` closes the key set.

go-raml (`https://github.com/acronis/go-raml`, `complex.go`, object
validation) agrees: a key that is neither declared nor matched by a pattern is
refused only when `additionalProperties` is `false`.

The example key is now `a-zA-Z`, which the literal pattern matches, with the
value `bar`, which is not a `number`. The fixture still fails, now because the
matched pattern rejects the value. The file name is kept so that outcome
records keyed by it stay valid.

---

## 7. `Types/Facets/redefine-built-in/valid.raml` redeclares a built-in facet

**Fixed.**

The fixture declared a user-defined facet named `format` on a subtype of
`datetime`:

```yaml
  myDate:
    type: datetime
    facets:
       format: string
```

Spec § User-defined Facets: "User-defined facet names on a type MUST NOT match
built-in facets on that type, nor facet names of any ancestor type in the
inheritance chain of the type" (`versions/raml-10/raml-10.md` line 1341 in
`https://github.com/raml-org/raml-spec`). `format` is a built-in facet of
`datetime` (§ Date: "The additional facet `format` MUST be available only when
the type equals `datetime`"), so a conforming parser must reject the file.

go-raml (`https://github.com/acronis/go-raml`) accepts it. Its table of
type-specific built-in facets (`consts.go`, `typeSpecificFacets`) has no
`datetime` entry, and run with unwrap and validation it also accepts
`facets: {pattern: string}` on a `string` subtype.

The type is now a `string` subtype, keeping the facet name `format`. `format`
is a built-in facet of `number`, `integer` (§ Number, § Integer) and `datetime`, but not
of `string`, so the file is valid and checks the other side of the rule: a
parser must not refuse a facet name that is built-in only on some other type.
`invalid-redefine-datetime.raml`, beside it, redeclares the type name
`datetime`.
