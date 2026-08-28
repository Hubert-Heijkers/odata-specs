# OData Compact JSON Format — AI-Assisted Drafting Audit Trail

This file records how the OData Compact JSON Format specification in
[odata-compact-json-format/](odata-compact-json-format/) was drafted with AI
assistance: what each session started from, what was asked and answered, what was
decided and by whom, what was produced, what was verified, and what was left for
the editors and the Technical Committee to decide.

It is **not** part of the specification and is not picked up by the document build
(`lib/iterator.js` only scans *directories* whose name starts with `odata-`).

## How to maintain this file

- Add a new `## Session N` section per working session, newest last. Record
  substantial changes and decisions, and also **discussions and decisions that
  changed nothing in the document** --- an option weighed and dropped, or a review
  comment judged unfounded, is exactly what gets asked about later and is otherwise
  unrecoverable.
- Date sessions from the git history, not from recollection.
- Record decisions in the [Decision Register](#decision-register) with a stable
  `D‑nn` identifier. Never renumber; supersede instead, and mark the old row.
- Distinguish clearly between **decisions taken by the TC or the editors** and
  **provisional choices made by the drafting assistant to keep the draft
  internally consistent**. The latter are proposals, not decisions, until
  ratified.
- Keep the "needs a call" list current; it is the agenda for the next session.

---

## Session 1 — 2026-08-20

Branch `odata-compact-json-format`, on top of commit `c7f6b61` ("Compact JSON
Format"). Participants: Hubert Heijkers (OData TC member since the TC's inception
in 2012; co-editor of this document) and Claude Opus 5, acting as drafting
assistant.

### Starting point

Three inputs existed:

1. **[odata-compact-json-format-v4.0.pdf](odata-compact-json-format-v4.0.pdf)** —
   the original proposal, Working Draft 01, Hubert Heijkers, dated 2018-01-29.
   Ten pages: a design sketch plus seven worked examples against an IBM TM1
   `Cubes`/`Dimensions`/`Views` model. Its substantive positions were:
   - structured types serialized as JSON arrays, order derived from the
     `odata.context` annotation plus CSDL declaration order;
   - **responses only** — "MUST NOT be used for requests";
   - `metadata` forced to `none`, `streaming` forced to `true`;
   - service document: not supported;
   - delta payloads: not supported;
   - open types: dynamic properties only via explicit `$select`;
   - heterogeneous collections of derived types: **error 278**, "Compact JSON
     content type can not be used to formulate a response to this request"
     (Example 3 of the proposal);
   - annotations wrapped in a JSON object carrying the annotation and a `value`
     member.

2. **A skeleton** in [odata-compact-json-format/](odata-compact-json-format/) —
   `meta.yaml` (targeting Version 4.02, CSD01, pubdate 19 August 2026),
   `0 frontmatter.md` and `Appendix.md`. No body chapters.

3. **The sibling specifications** in this repository, which set the conventions
   the new document had to follow: `##sec`/`##subsec` numbering, `::: example`
   blocks, `[section ##Anchor]` and `[#Doc#Anchor]` reference syntax, and the
   `lib/number.js` + Pandoc build.

### Stated goal

Turn the 2018 proposal into a first normative specification, with the scope
extended: the format should be a lossless alternative for representing OData
responses **and OData request payloads**, its prime aim being to minimize the
uncompressed size of payloads.

### Questions posed and answers given

The assistant identified three decisions that would materially change the shape
of the document and put them to Hubert before drafting.

#### Q1 — How should a consumer map array positions to properties?

Options offered: a self-describing header row carrying property names; the 2018
metadata-driven derivation; or a hybrid.

**Answer (Hubert):** the thinking has changed, partly because compact JSON for
request bodies could be of great value as well. The **Context URL** should get a
more prominent role — it has always had the role of allowing a response to be
interpreted without knowing the request, describing what is in the payload, and
here specifically **which properties show up at which position**.

Additionally, and not present in the 2018 service: **annotations**. These would
not typically show up as a field of their own, although where they are explicitly
selected they could. Otherwise a **wrapper object** is injected, holding the
annotations that apply to the property — and those can be just `@<annotation>`
rather than `property@annotation` as in the JSON format — together with the value
once again. For brevity, `_` instead of `value` was one of the thoughts.

#### Q2 — Which payload kinds should the first normative draft cover?

**Answer (Hubert):** as a guiding principle, **any valid JSON format payload
should be valid Compact JSON as well**. It would be nice if that could hold in
100% of cases; for that, the TC would be open to considering changes to the JSON
format itself.

The prime value of Compact JSON is in larger payloads — large(r) collections of
entities and instances of complex types — and it should not matter where those
collections appear: in responses to GET requests, but equally in payloads of
POST, PUT and PATCH requests. The original proposal only addressed responses, so
some scenarios still need working through, **including delta payloads**, which
are also important scenarios to tackle.

#### Q3 — How should the format be requested and identified?

Options offered: the `compact=true` format parameter; a distinct media subtype
such as `application/compact+json`; or a parameter with a value space
(`compact=rows`, leaving room for a future columnar variant).

**Answer (Hubert):** the `compact=true` format parameter.

### Research performed before drafting

To avoid inventing context URL syntax, the assistant read
[odata-protocol/10 Context URL.md](odata-protocol/10%20Context%20URL.md) in full
and cross-checked `$select`/`$expand` forms in
[odata-url-conventions/5 Query Options.md](odata-url-conventions/5%20Query%20Options.md).
Three findings shaped the draft:

- Contained structural properties are represented in the context URL using **path
  syntax** (`Address/City`), so selecting two sub-properties of one complex
  property yields two select-items sharing a first segment. A positional
  representation must collapse them into one position — hence the grouping rule,
  D‑03. The 2018 proposal's Example 4 already did exactly this, without stating
  the rule.
- "If the context URL includes only expanded navigation properties … then all
  structural properties are implicitly selected." This case has no stated
  ordering, so the draft had to define one — D‑04.
- `#$delta` is **already** defined as the context URL of an *update request body*
  for a collection of entities. This is existing precedent for a context URL in a
  request body, and it is the only such precedent — see open issue 1.

### Decisions and provisional choices

See the [Decision Register](#decision-register) below. In summary, the three
answers above produced a design in which:

- the context URL's select-list **is** the positional property list;
- one uniform *wrapper object* carries annotations and control information
  wherever a value may appear;
- the superset principle removes the need for the 2018 proposal's error 278: an
  instance that cannot be represented positionally simply falls back to its JSON
  format representation, inside an otherwise positional collection.

Two consequences were noted at the time as better than anticipated:

- **PATCH needs no sentinel value.** PATCH semantics rest on present-vs-absent,
  which a positional array cannot express. But narrowing the select-list narrows
  the positional property list, so "the properties being updated" and "the
  positional property list" are the same list. This was not designed for; it fell
  out of Hubert's answer to Q1.
- **Derived types and open types stop being error cases.** Both were bail-outs in
  2018; under the superset principle neither needs a mechanism of its own.

### What was produced

Thirteen normative sections plus a fourteenth, non-normative open-issues chapter —
about 9,400 words of new chapter text (11,300 including the pre-existing
frontmatter and appendix):

| File | Sections |
| :--- | :--- |
| [1 Introduction.md](odata-compact-json-format/1%20Introduction.md) | 1 Introduction, 2 Compact JSON Format Design |
| [2 Requesting the Compact JSON Format.md](odata-compact-json-format/2%20Requesting%20the%20Compact%20JSON%20Format.md) | 3 |
| [3 Common Characteristics.md](odata-compact-json-format/3%20Common%20Characteristics.md) | 4 |
| [4 Positional Representation.md](odata-compact-json-format/4%20Positional%20Representation.md) | 5 — the core |
| [5 Structured Instances.md](odata-compact-json-format/5%20Structured%20Instances.md) | 6 |
| [6 Annotations and Control Information.md](odata-compact-json-format/6%20Annotations%20and%20Control%20Information.md) | 7 |
| [7 Request Payloads.md](odata-compact-json-format/7%20Request%20Payloads.md) | 8 |
| [8 Delta Payload.md](odata-compact-json-format/8%20Delta%20Payload.md) | 9 |
| [9 Other Payloads.md](odata-compact-json-format/9%20Other%20Payloads.md) | 10 Service Document, 11 Error Response, 12 Batch |
| [10 Conformance.md](odata-compact-json-format/10%20Conformance.md) | 13 — 22 conformance clauses |
| [98 Open Issues.md](odata-compact-json-format/98%20Open%20Issues.md) | 14 — non-normative, to be removed before CS |

Editorial changes to existing files:

- [Appendix.md](odata-compact-json-format/Appendix.md) — added `[OData-JSON]`
  (normative) and `[OData-Aggregation]` (informative) references; seeded the
  revision history with WD01 (2018) and WD02.
- [0 frontmatter.md](odata-compact-json-format/0%20frontmatter.md) — removed a
  stray `\compact-json-format-` fragment in the "Latest stage" block; corrected
  the citation key from `[OData-JSON-Format-v4.02]` to
  `[OData-Compact-JSON-Format-v4.02]`.
- `odatatc-members.md` — created during the session and then discarded. The
  skeleton's `$$$include ../odatatc-members.md$$$` referenced a file that was not
  present in this working tree, so a stand-in was written to let the build
  resolve. It turned out the real file is added by a separate branch that lands
  ahead of this one, so the stand-in was removed and the include needs no change
  here.

### Verification performed, and its limits

`npm run build` could **not** be run in this environment: there is no
`node_modules` and Pandoc 3.8.3 (the version pinned in
`.github/workflows/nodejs.yml`) is not installed.

Instead the assistant ran `lib/number.js` directly against every document in the
repository, via a temporary harness that stubbed the Pandoc version check and
resolved `js-yaml` from a scratch install. The harness was deleted afterwards and
is not in the repository. Results:

- `odata-compact-json-format` resolves cleanly — 2089 lines of generated
  Markdown, correct section numbering, complete table of contents.
- All eight pre-existing documents still resolve, unchanged.
  (`odata-vocabularies` fails, as it does for anyone without the sibling
  `oasis-tcs/odata-vocabularies` repository cloned alongside; `build.js` skips it
  unless invoked with `repo:odata-vocabularies`.)
- Every cross-document reference into `odata-protocol` resolves to a real anchor
  and a real published URL.
- Two defects were caught and fixed by this check: an `{id=…}` anchor split
  across two source lines, which `number.js` scans line-by-line and therefore
  could not see; and an example reference written `[Example ##ex_countonly]`
  where the reference form is `[Example ##countonly]` — the latter would have
  passed silently into the output as literal text.

**Not verified:** Pandoc's rendering of the Markdown, PDF generation, and
`npm test`. Run `npm install` and install Pandoc 3.8.3 before pushing.

---

## Session 2 — 2026-08-26

Branch `odata-compact-json-format`, rebased onto Heiko Theißen's commits of
2026-08-21. Two changes, both editorial in origin, one of them substantive in
effect.

### Example formatting — HTTP and payload split into separate blocks

Every example that combined an HTTP request line, or a `Content-Type` header,
with its JSON payload in a single fenced block was split into two: the HTTP
portion in an untagged block, the payload in a block tagged `json`.

Nineteen blocks across six files were affected. Three untagged blocks were
deliberately left alone, having no payload to separate: the `Content-Type`
example in section 4.1, the bare context URL in section 5.3, and the document
conventions example in section 1.1.3.

Verified afterwards: 36 of the 37 `json`-tagged blocks parse as standalone JSON.
The one that does not is the `Capabilities.SupportedFormats` annotation in
section 3.2, which is deliberately a fragment — a name/value pair shown out of
context, as the sibling specifications also show annotations. Pandoc renders it
without lexer errors, so `npm run select ".json .er"` does not flag it.

Committed by Hubert as `446c109`, together with the regenerated `docs/` output.

### Review finding — the `value` name at the root of a message body

A TC reviewer objected to this sentence in section 7.1:

> At the root of the message body the name `value` is also recognized, with the
> same meaning, for compatibility with [OData-JSON], as required by the superset
> principle.

**The objection was correct.** OData-JSON uses `value` as the name
of the message body's value for a single primitive value and for collections
only. Where the payload is a single entity, a single complex value or a single
entity reference, the message body *is* the instance, and a name/value pair
named `value` there is an ordinary property. The withdrawn sentence granted
`value` at *every* root, which extended OData-JSON rather than
accommodating it — and the appeal to the superset principle was misplaced, since
where OData-JSON never produces `value` there is nothing to be
compatible with.

Worse, the sentence reintroduced the very ambiguity the section went on to warn
about. Had `value` been the wrapper's value name at a single-entity root, then

    {"@context": "…#Customers/$entity", "value": […]}

would have been at once the positional representation of an entity and the
OData-JSON representation of an entity with a collection-valued
property named `value`, with nothing to distinguish them.

**Resolution.** `value` is now recognized *by reference* rather than by
enumeration: "wherever OData-JSON specifies that the message body
contains a name/value pair whose name is `value`". Defined that way the
permission provably cannot exceed OData-JSON, which was the
reviewer's concern. Everywhere else `value` is a property name and a receiver
MUST NOT read it as a wrapper's value. See D‑08, restated.

Four places changed: section 7.1 (the rule and its rationale), section 4.2
(the same claim stated in passing), example 20's caption (which now says *why*
`value` is admissible there — the body is a collection), and conformance
clauses 5.2 and 17.

**A second collision, newly recorded and not resolved.** `_` is exposed to the
same problem in principle: a type declaring, or an open type carrying, a
property named `_` is indistinguishable from a wrapper object by name alone.
The draft does not reserve `_` in the model. Added to open issue 4.

---

## Session 3 — 2026-08-26

Continuation of the session 2 review thread. Sources had meanwhile been split one
file per section and renumbered.

### The value name, settled: `$`

Session 2 recorded that `_` collides — it is a valid simple identifier, so a type
declaring, or an open type carrying, a property named `_` is indistinguishable
from a wrapper object. The TC had already identified this independently, and had
discussed `$` and the empty string as replacements.

The assistant initially argued for `@`, on the grounds that every other member of
a wrapper object already begins with `@`, which would make the wrapper-object rule
syntactic: *a wrapper object is a JSON object all of whose names begin with `@`*.

**That argument was withdrawn**, for two reasons that emerged in discussion:

1. Hubert relayed a further TC direction — that a wrapper object should be able to
   carry named properties alongside the positional value (see below). That removes
   the universal-quantification rule the `@` case rested on, so the discriminator
   has to become the *presence* of the reserved name rather than a property of all
   names. For an existence test, the reserved name wants to be maximally distinct
   from its neighbours, not a prefix of them.
2. OData-JSON parsers conventionally separate annotations from
   properties by testing whether a name begins with `@`. A value named `@` would
   land in the annotation branch of existing code, silently — a concrete
   implementation hazard, not merely a stylistic one.

The decisive precedent is OData-CSDL JSON, which solves the identical
problem the identical way and, crucially, in objects that mix all three kinds of
member the compact format now needs together:

```json
"Product": {
  "$Kind": "EntityType",
  "$Key": [ "ID" ],
  "ID":   { "$Type": "Edm.Int32" },
  "@UI.DisplayName": { "$Path": "Name" }
}
```

`$`-prefixed fixed names, `@`-prefixed annotations, and model-defined names, in one
object. Adopting `$` gives the compact format the same three-way namespace split
that OData already uses elsewhere. Recorded as D‑08, now **TC**-decided.

The empty string was rejected: legal and collision-free, but `""` and `" "` are
indistinguishable in fonts, diffs and pasted payloads, so a stray space produces
valid JSON with the wrong meaning and no diagnostic; it cannot be named in prose
or spotted in an example; and its only advantage is one byte per wrapper object,
in a place where wrapper objects are not what dominates payload size.

Applied mechanically across the eight source files — 44 occurrences. Verified
afterwards that Pandoc's `tex_math_dollars` extension does not capture any of
them: every `$` is inside a code span or fenced block, and the generated HTML
contains no MathJax markup.

### Extended wrapper objects — direction recorded, not drafted

The TC wants an instance whose properties cannot all be placed in the select-list
— typically an open type carrying dynamic properties — to keep its positional
representation, rather than falling back wholesale to the
OData-JSON object form as section "Open Types and Dynamic Properties" currently requires. The shape is: leading
annotations, then `$` holding the positional representation, then the remaining
properties by name.

This supersedes D‑09 and largely resolves open issue 6 in the affirmative, but the
normative text has not been written. Four points were identified as needing
settlement first, and are recorded under open issue 6: the full recognition rule
(two-part, since a wrapper carrying only annotations has no `$`); which properties
may appear by name; the ordering constraint, which currently requires the value to
be last in a wrapper; and whether the same shape should extend to derived types.

A related consequence was noted for open issue 5: once a wrapper is recognized by
the presence of `$`, the empty object `{}` — used for "not applicable" — is no
longer a wrapper by the definition, and its meaning becomes a convention riding on
an edge case rather than a consequence. That needs restating too.


### Derived types, open types and the extended wrapper — drafted

Hubert put four points to the TC's thinking, and the answers settled the design.

**Derived types.** The assistant offered three shapes and recommended grouping by
the type cast, so that one position per derived type would hold either a nested
array or `{}`. The TC had reached a different and better answer: *the wrapper
carries `@type` ahead of the positional representation, and the consumer computes
the positional property list for that type from the context URL.* Peer subtypes
then cost nothing on instances that are not of that type — no gaps, no
placeholders, no extra nesting — which is the most compact of the four candidates
and, compactness being the point of the format, the right one. Recorded as D‑26,
superseding D‑14.

Two consequences were written in that the discussion had not yet priced:

- **`type` becomes load-bearing.** It is no longer informative: without it the
  receiver cannot compute the positional property list, so it cannot decode the
  array at all. The draft now requires it for any instance whose list differs
  from that of the context URL's declared type, **irrespective of the `metadata`
  format parameter** — that is, `metadata=none` does not excuse omitting it, for
  the same reason it does not excuse omitting `context`. section "Payload Ordering Constraints" already placed `type` before the value; the draft
  now says why that ordering matters rather than treating it as convention.
- **Row shape is now heterogeneous by design.** Instances in one collection may
  have positional representations of different lengths and meanings. This bears
  on open issue 12, and on any future column-oriented variant contemplated by
  open issue 7.

**Open types.** `{}` is retained, but only here: a selected dynamic property that
an instance does not have. The draft now states the contrast explicitly, because
the two cases look similar and behave differently — a type cast says *which
instances a property applies to*, so a non-applicable instance simply has a
shorter list; a selected dynamic property says nothing about which instances have
it, so it is in everyone's list and `{}` reports its absence.

**Extended wrapper.** Drafted as agreed: annotations, then `$`, then properties by
name; a property may not appear both positionally and by name. The wholesale
fallback to the OData-JSON object form in section "Open Types and Dynamic Properties" is gone.

Files touched: section "Payload Ordering Constraints", section "Determining the Positional Property List" (new step 3), section "Grouping of Select Items", section "Derived Types", section "Open Types and Dynamic Properties", section "The Wrapper Object", section "Control Information: type", and conformance clauses 3.3, 5.4, 13.1, 15 and 16.
Build green.

---

## Session 4 — 2026-08-28

Two review comments, both organizational rather than technical, and both accepted.
Committed as `146d4fa` and `c471f1e`.

### "that property of arrays" — a term collision, not a word choice

A reviewer found the fourth paragraph of the introduction confusing and proposed
"feature" in place of "property":

> JSON arrays preserve the order of their items, whereas the order of the
> name/value pairs of a JSON object is not significant; this format uses that
> **property** of arrays to convey, by position, which value belongs to which
> **property**.

The reviewer's instinct was right but the diagnosis understated it. "Property" is a
defined term in this document, used several hundred times, and here it appeared
twice in one clause with two different senses --- once meaning "characteristic",
once meaning an OData structural property. No synonym fixes that well, which is why
neither Hubert nor the reviewer was satisfied with "feature".

*Resolution:* drop the abstract noun rather than replace it, and borrow RFC 8259's
own vocabulary, which defines an array as an ordered sequence and an object as an
unordered collection:

> The items of a JSON array are **ordered**, whereas the name/value pairs of a JSON
> object are not; this format relies on **that ordering** to convey, by position,
> which value belongs to which property.

*Alternative offered if a noun is wanted:* "guarantee" --- precise, and it carries
weight "feature" does not, since the format's decodability genuinely rests on JSON
guaranteeing array order. "Characteristic" also works but is heavier.

The rest of the document was checked for the same collision: three other hits for
"property of" / "that property", all legitimate OData uses. This was the only one.

### The wrapper object was introduced after thirteen uses of it

A reviewer observed that it is confusing to define the wrapper object in section 7,
after it has already been used to describe annotations, control information and
dynamic properties. Hubert asked whether reordering would also let it be described
more easily.

Measured before acting: **thirteen** references to the wrapper preceded its
definition, in sections 1, 3, 4, 5 and 6.

The diagnosis was that the section was *mis-filed*, not merely mis-ordered. When it
was drafted the wrapper existed only to hold annotations, so the annotations chapter
was the right home. Since D‑25 it also carries data properties by name, and since
D‑26 it carries the `type` control information that a receiver needs in order to
decode the positional array at all. Of the four things it now carries, only one is
an annotation. It had become the format's second structural construct while still
being filed as an annotations mechanism.

*Resolution (D‑28).* Chapter 4 was renamed "Compact Representations" and now defines
both constructs the format adds; "Common Characteristics" moved to chapter 5, so its
three references to the wrapper became back-references. The wrapper is section 4.5,
immediately before 4.6 Position Values, which is the first place that needs it.
Forward references fell from thirteen to three, all of them legitimate: the glossary
entry that *is* the definition, the chapter's own statement of what it will define,
and one rationale aside in section 3.1.

Hubert's guess that it would also be easier to *describe* proved right. Filed under
annotations, the wrapper had to be introduced as somewhere for annotations to live.
Next to the positional representation it can be introduced by the constraint that
actually creates it --- a JSON array has no name/value pairs, and the containing
instance is no help because it may itself be an array --- which covers all four
things it carries instead of one.

`Positional Representation` was kept as section 4.1 rather than folded into the
chapter title, so the `PositionalRepresentation` anchor survives; no cross-reference
broke, since references here resolve by name rather than by number. The section 1.1.1
glossary entry, which still described the wrapper as carrying only annotations and
control information, was updated --- it had been stale since D‑25.

### Recording practice

Hubert asked that this trail be kept current for substantial changes and decisions,
and also for important discussions and decisions that do not change the document.
Both changes above had initially been left out of the register on the grounds that
they were editorial rather than design decisions; that judgment was wrong, and the
maintenance rules at the top of this file have been widened accordingly.

---

## Decision Register

Status values: **TC** — decided by the TC or by the editor in this session;
**Provisional** — chosen by the drafting assistant to keep the draft internally
consistent, awaiting ratification; **Open** — recorded as an open issue in
section 14.

| ID | Decision | Basis | Status | Where |
| :--- | :--- | :--- | :--- | :--- |
| D‑01 | Compact JSON is a **superset** of the OData JSON format: every valid JSON format payload is a valid compact payload, and a sender may fall back to the object representation for any instance. | Hubert, Q2 | TC | §2.1 principle 1 |
| D‑02 | The **context URL's select-list is the positional property list** — membership *and* order. No competing mechanism (no header row, no inline property list). | Hubert, Q1 | TC | §5.2 |
| D‑03 | Select-items sharing a **first segment** form one group occupying one position; the group's nested select-list determines the nested instance's list. | Assistant, forced by path syntax in context URLs | Provisional | §5.3 |
| D‑04 | Where structural properties are **implicitly selected**, they are prepended in CSDL declaration order, base type first, ahead of the listed items. | Assistant, forced by an unstated case in Part 1 | Provisional | §5.2 |
| D‑05 | A positional array MUST have **exactly** as many items as the list — no omission, no truncation, no reordering. To send fewer properties, narrow the select-list. | Assistant | Provisional | §5.1 |
| D‑06 | One uniform **wrapper object** carries annotations and control information wherever a value may appear: root, array position, collection member. | Hubert, Q1 | TC | §7.1 |
| D‑07 | Property annotations **drop the property-name prefix**: `Orders@count` becomes `@count`, because the position identifies the property. | Hubert, Q1 | TC | §7.3 |
| D‑08 | The wrapper's value is named **`$`**. `value` is recognized **only** where [OData-JSON] itself specifies it as the message body's value name — never where [OData-JSON] represents the body as the instance itself. | `_` (sessions 1–2) collided, being a valid simple identifier; **settled on `$` in session 3**, following [OData-CSDL] JSON, which reserves `$`-prefixed names for exactly this purpose | **TC** | §7.1, open issue 4 |
| ~~D‑09~~ | ~~A wrapper object MUST NOT carry data properties by name — only annotations and the value.~~ **Superseded by D‑25.** | Assistant, to keep wrappers distinguishable from JSON-format objects | Superseded | §7.1, open issue 6 |
| D‑10 | **PATCH** conveys "which properties to update" through the select-list; a property not in the positional property list is unchanged, one present with `null` is set to null. **No sentinel value.** | Assistant, consequence of D‑02 | Provisional | §8.4 |
| D‑11 | Request bodies **MAY carry a context URL**, using response templates unchanged. Without one, the list is the default selection of the target type. | Assistant, required by D‑02 for requests | Provisional — **needs a Part 1 change** | §8.1, open issue 1 |
| D‑12 | A **request** message body MAY be a bare JSON array (the request URL supplies the type). Responses always remain JSON objects. | Assistant | Provisional | §8.2 |
| D‑13 | **`{}`**, the empty wrapper, means "not applicable". | Assistant, required for losslessness | **Narrowed by D‑27** to selected dynamic properties only | §6.8, open issue 5 |
| D‑14 | **Derived types** are handled by `@type` in a wrapper; every instance in a collection shares one positional property list regardless of its type. Replaces the 2018 error 278. | Assistant, enabled by D‑01 | Provisional | §6.7 |
| D‑15 | **Open types**: dynamic properties occupy a position only if explicitly selected; an instance with unselected dynamic properties falls back to the object representation. | 2018 proposal + D‑01 | Provisional | §6.8 |
| D‑16 | **Delta payloads are in scope.** Added/changed entities are positional; deleted entities and links are not, having no select-list. | Hubert, Q2 | TC (scope), Provisional (treatment) | §9 |
| D‑17 | **No positional form** for: service document, error responses, batch documents, entity references, delta links. Each has members fixed by the format rather than the model, and none is a large payload. | Assistant | Provisional | §10–12, open issue 2 |
| D‑18 | The **service document is served**, in its JSON format representation, when requested with `compact=true`. Replaces the 2018 position of refusing the request. | Assistant, consequence of D‑01 | Provisional | §10 |
| D‑19 | Requested via the **`compact=true` format parameter** on `application/json`. | Hubert, Q3 | TC | §3 |
| D‑20 | `metadata=none` and `minimal` are permitted; **`metadata=full` is NOT RECOMMENDED** — it forces every instance into a wrapper and defeats the format. Departs from the 2018 position of forcing `metadata=none`. | Assistant | Provisional | §3.1 |
| D‑21 | The `context` control information is **always required**, including under `metadata=none`, since without it the payload cannot be interpreted at all. | 2018 proposal, retained | TC | §7.5.1 |
| D‑22 | `streaming` remains meaningful and is **not forced** to `true`, departing from the 2018 position; a compact payload meeting the constraints must declare `streaming=true`. | Assistant | Provisional | §3.1, §4.3 |
| D‑23 | CSDL **declaration order must be stable** for a given metadata document URL; services SHOULD reference versioned metadata from the context URL. | Assistant, forced by D‑02 + D‑04 | Provisional | §5.2, open issue 3 |
| D‑24 | Examples use the **`Customers`/`Orders`/`Address`** model of the sibling specifications, not the `Cubes`/`Dimensions` model of the 2018 proposal. | Assistant, for house consistency | Provisional | throughout |
| D‑25 | A wrapper object **MAY** carry data properties by name, after `$`, so that an open-type instance keeps its positional representation instead of falling back wholesale to the [OData-JSON] object form. A wrapper is then recognized by the presence of `$`, or by all of its members being annotations. **Supersedes D‑09.** | TC direction relayed in session 3 | **TC**, drafted | §6.8, §7.1, open issue 6 |
| ~~D‑14~~ | ~~Every instance in a collection shares one positional property list; a derived-type property selected via a type cast occupies a position in every instance, and `{}` fills the gaps.~~ **Superseded by D‑26.** | Assistant, session 1 | Superseded | §6.7 |
| D‑26 | The positional property list depends on the instance's **type** as well as the context URL: a type-cast select-item contributes a position only to instances of that type or a type derived from it. No gaps and no placeholders. Consequently `type` becomes load-bearing — a service **MUST** send it whenever an instance's list differs from that of the context URL's declared type, **irrespective of `metadata=none`**, and it must precede the positional representation. **Supersedes D‑14.** | TC direction relayed in session 3, in preference to gap-filling or grouping by type cast | **TC** | §5.2 step 3, §6.7, §7.5.3 |
| D‑27 | `{}` is retained, but **only** for a selected dynamic property that an instance does not have. It is no longer a wrapper by the D‑25 recognition rule, and conveys "undefined" by convention; the TC accepted this explicitly. | Hubert, session 3 | **TC** | §6.8, §7.1, open issue 5 |
| D‑28 | Chapter 4, **"Compact Representations"**, defines both constructs the format adds — the positional representation and the wrapper object (§4.5) — and "Common Characteristics" moves to chapter 5. The wrapper is no longer filed under annotations, since three of the four things it carries are not annotations. | Reviewer comment, session 4; forward references to the wrapper fell from 13 to 3 | **TC** (reviewer-driven) | §4, §5, §7 |

---

## What needs your call next

Ordered by how much rework a late reversal would cause.

### 1. Cross-part dependency — raise with the TC early

**Open issue 1: context URLs in request bodies (D‑11).** Part 1 states that
request payloads generally do not require context URLs, and defines a request-body
template only for `#$delta`. The entire request story — create, PATCH, deep
insert, bind — depends on response context URL templates appearing in POST/PUT/PATCH
bodies. This needs a Part 1 change, or a different mechanism that would contradict
design principle 3. Everything in section 8 rests on this.

**Open issue 3: stability of property order (D‑23).** CSDL does not currently make
declaration order significant, so a service may reorder properties without
considering it breaking. The draft imposes a stability requirement and recommends
versioned metadata URLs. The alternative — requiring an explicit `$select` in
every compact payload, removing the dependency entirely — is more honest but
costs bytes and constrains clients. Possibly a Core vocabulary term fixing each
property's ordinal.

**Open issue 2: the 100% losslessness goal.** You said the TC would consider
changing the JSON format itself to reach it. The constructs currently *without* a
positional counterpart are listed in D‑17. Is that list acceptable, or should
some of them drive JSON format changes?

### 2. Settled since — kept here for the record, no action needed

- **Open issue 4: the value name (D‑08).** Settled on `$` in session 3, following
  CSDL JSON. `_` collided, being a valid simple identifier; `@` was rejected
  because existing parsers branch on a leading `@`; the empty string was rejected
  as silently typo-fragile.
- **Open issue 5: `{}` (D‑13, narrowed by D‑27).** Retained, but now only for a
  selected dynamic property an instance does not have. You accepted that it
  conveys "undefined" by convention rather than as a consequence of the wrapper
  definition.
- **Open issue 6: wrapper objects carrying named properties (D‑09 → D‑25).**
  Resolved in the affirmative and drafted.
- **Open issue 7: `compact=true` versus `compact=<variant>`.** You chose the
  boolean form; this remains the door left open for a future columnar variant, and
  D‑26 (heterogeneous row shapes) now makes that variant harder to add later.

### 2a. Session 4 judgment calls — veto if you disagree

- **Chapter 4 is titled "Compact Representations".** It now covers both the
  positional representation and the wrapper object, so the old title
  "Positional Representation" would under-describe it. That title survives as
  section 4.1, which also preserves every cross-reference.
- **"Common Characteristics" now follows the constructs, as chapter 5.** This
  departs from OData-JSON's running order, where Common Characteristics is
  section 4. It moved because three of its references were to a construct defined
  later.

### 3. Not yet specified — needs drafting once the above settle

- **Open issue 8:** recursive expansion (`DirectReports+(FirstName,LastName)`,
  `$levels`) — the nested list at the second and subsequent levels.
- **Open issue 9:** operations in the select-list, and the `{namespace}.*`
  shortcut. Currently a consequence of the rules rather than a decision.
- **Open issue 10:** whether the action/function parameter object should itself
  be positional, given parameters have a declared order in CSDL.
- **Open issue 11:** interaction with `$apply`; `concat` in particular.
- **Open issue 12:** whether a collection must be homogeneous in its
  representation, which a receiver optimized for the positional case may prefer.
- **Open issue 13:** how the `/$count` segment appears in the context URL —
  underspecified in Part 1 and the JSON format, and the example under
  "Property Annotations" assumes it is retained.
- **Open issue 6, residual:** a property must not appear both positionally and by
  name, but the draft places no obligation on a receiver to detect a producer that
  breaks this, and does not say which wins.

### 4. Housekeeping

- **Build environment.** `npm install` plus Pandoc 3.8.3 are needed. Note that the
  generated products under `docs/` are **not** committed on this branch --- only the
  Markdown sources are; regeneration happens separately on its own branch. Sessions
  2 to 4 used `npm run build` to validate, which rewrites `docs/` as a side effect;
  from session 4 onwards validation runs `lib/number.js` into a throwaway stream
  instead, optionally piping through Pandoc into a temp directory, so the working
  tree is left alone.
- **Section 14 (Open Issues) must be removed** before the document advances to
  Committee Specification.
- **One JSON example does not parse standalone** and is not meant to: the
  `Capabilities.SupportedFormats` fragment under "Advertising Support" is a
  name/value pair shown out of context, as the sibling specifications also show
  annotations. Pandoc reports no lexer error, so `npm run select ".json .er"`
  does not flag it.
