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

## Session 5 — 2026-08-28

Opened as a discussion; the select-list requirement was settled and drafted in
the same session. The two follow-on items were deliberately left undrafted.

### Proposal: the select-list MUST always appear in the context URL

The TC looked more closely at how the positional property list is determined
(sections 4.3 and 8.1, which currently define two routes: an explicit select-list,
or a *default selection* derived from the CSDL document) and concluded that there is
no good definition, against the CSDL document as it stands, of what that default
selection is. The proposal is that a compact payload MUST always carry an explicit
select-list in its context URL.

Hubert's argument is sharper than the one recorded under open issue 3 in session 1.
That issue was framed as *declaration order is not normatively stable, so it might
change between metadata versions*. The stronger form is: **a service always knows
which version of the CSDL document it used; a client composing a request payload may
not know, and cannot determine it from the payload alone.** Enumerating the
select-list moves the knowledge to the only party that reliably has it. That
reasoning applies to responses and applies with more force to requests.

*Assessment:* accept. It closes open issue 3 outright, retires D‑04 and D‑23, and
removes two of the five steps in section 4.3. The cost is one select-list per
payload, weighed against N instances --- negligible for the large collections this
format targets, and a single-instance compact payload was already a net loss under
design principle 2.

*Consequences identified that the proposal had not yet priced:*

1. **`*` must be forbidden as well.** Step 2 of section 4.3 expands `*` into the
   structural properties in declaration order --- the very dependency being removed.
   The same applies to the `{namespace}.*` operation shortcut (open issue 9) and to
   the [OData-Protocol] rule that a select-list containing only expanded navigation
   properties implicitly selects all structural properties. If any of the three
   survives, the requirement is hollow.
2. **Section 8.2, the bare JSON array request body, must go**, and with it D‑12. Its
   whole premise was that the request URL supplies the type and the default
   selection supplies the list; with no context URL it cannot carry a select-list.
3. **Action and function parameter values lose the positional representation**
   (section 8.8), which currently derives the list from the default selection of the
   parameter's declared type. Parameters have no context URL. Either positional
   parameter values are given up, or something new is needed --- and an action taking
   a large collection parameter is squarely a compact-JSON use case.
4. **Open issue 1 changes from a nicety to a blocker.** A compact request body may
   currently carry `@context`; under this rule it MUST. The Part 1 change that would
   sanction context URLs in request bodies is then load-bearing: without it, compact
   request payloads cannot be specified at all.

### Proposal: control information in the positional representation

Following from the above, the TC is considering allowing control information, and
selected instance annotations, to occupy positions in the positional representation.

Note that selected *instance annotations* already do --- see "Selected Annotations".
It is control information that would be new.

*Assessment:* valuable, and it repairs the weakest point in the current draft.
Section 3.1 currently says `metadata=full` is NOT RECOMMENDED precisely because full
metadata forces every instance into a wrapper object, inverting the format's
purpose. If `@id` and `@etag` can occupy positions, that inversion disappears:

```json
{ "@id": "Customers('ALFKI')", "@etag": "W/\"1\"", "$": ["ALFKI", "Alfreds"] }
```

becomes

```json
[ "Customers('ALFKI')", "W/\"1\"", "ALFKI", "Alfreds" ]
```

which drops two names, the `$` name and the braces from every instance --- in exactly
the case, an updatable collection carrying ids and etags, where compact JSON
currently degrades worst. D‑20 would then be worth revisiting.

*Problems to solve before this can be drafted:*

1. **`type` in a position is circular.** Under D‑26 the positional property list
   depends on the instance's type, so reading `@type` out of a position requires
   already knowing the list. Resolvable by requiring control-information items to
   form a *type-independent prefix* --- they must precede any type-cast-qualified
   item --- so that the leading positions are identical for every instance in the
   collection. That constrains where such items may appear in the select-list and
   must be stated normatively.
2. **Absence.** `@etag` may be absent on some instances. This is the same situation
   as a selected dynamic property an instance does not have, so `{}` (D‑27) reuses
   cleanly and no new mechanism is needed.
3. **The cross-specification ask is larger than open issue 1.** `$select` supports
   instance annotations today, but not control information: `@id`, `@etag` and
   `@count` are not terms and are not selectable. Extending the select-list to admit
   them is a change to Part 1 and Part 2, and a bigger one than adding request-body
   context URL templates.

*Suggested staging:* adopt the mandatory select-list first, since it stands on its
own and simplifies the document; pursue control information in positions as a
follow-on, once the select-list extension has been discussed with the TC.

### Resolution

Hubert answered each of the four consequences.

1. **`*`.** Confirmed: `*` must go from the *context URL*, replaced by the explicit
   positional list. But `*` in `$select` or `$expand` remains perfectly valid --- the
   restriction is on the context URL, not on the request. The service resolves the
   query as it normally would and then enumerates what it actually placed in the
   positional representation, and may still convey further dynamic properties by
   name in a wrapper object. This distinction is now stated explicitly in the draft,
   because the requirement reads as far more restrictive than it is without it.

2. **The bare JSON array request body.** Hubert noted this is invalid under OData in
   any case: [OData-JSON](#ODataJSON) wraps request bodies in an object
   *deliberately*, so there is always somewhere for control information and
   annotations to go, and so that the body's shape does not vary with whether such
   information happens to be present. So D‑12, introduced by the assistant in
   session 1, was wrong on its own terms and not merely incompatible with the new
   rule --- it had quietly contradicted a considered decision in the base format.
   Removed, and the reason is now recorded in section 8.2 rather than left implicit.

3. **Action and function parameters.** Accepted as a real loss worth solving, not a
   technicality. Hubert's position: *the fact that a context URL has never needed to
   carry this information --- the case arises only in request payloads --- is no
   reason not to add it.* The TC will craft one or more proposals and discuss them
   before anything is adopted. The draft therefore forbids positional parameter
   values for now and says plainly that this is a limitation awaiting a mechanism,
   rather than presenting it as a considered exclusion. Open issue 14.

4. **`type` in a position.** Hubert proposed a stricter and simpler rule than the
   assistant's "type-independent prefix": if type information is conveyed
   positionally, `type` MUST occupy the **first** position of the array. This is the
   minimal sufficient constraint --- `type` is the only thing that must be readable
   before the positional property list is known, so it is the only thing whose
   position must be fixed. Recorded in open issue 15, pending the larger question of
   extending the select-list to admit control information at all.

### Drafted

Nine sites depended on a default selection; all are now gone. Section 4.3 lost two
of its five steps and gained the requirement and its rationale; the two examples
that relied on implicit selection were rewritten to show a service enumerating what
it returned; the nested-select-list default was removed; section 8.1 now requires
`context` in every compact request body; section 8.2 was rewritten around
[OData-JSON](#ODataJSON)'s reason for wrapping; section 8.8 forbids positional
parameter values; conformance clauses 3.1, 13, 18 and 22 were restated. Open issue 3
is closed, and open issues 14 and 15 were opened. Validated with no undefined links.

---

## Session 6 — 2026-08-28

Discussion, plus a correction to the assistant's assessment of open issue 15. No
normative text changed; the open issues were restructured.

### Control information in positions: the ask is smaller than recorded

Session 5 recorded that admitting control information into the positional
representation "is a change to Part 1 and Part 2, and a bigger one than open issue
1". **That was wrong**, and Hubert's reasoning shows why.

The select-list in a *context URL* is written by the service, not by the client. The
context URL describes what the payload contains; `$select` is what the client asked
for. D‑29 already turns on that distinction --- a service resolves `$select=*`, or no
`$select` at all, and then enumerates what it actually returned. Whether `@id` and
`@etag` appear in a response is driven by the protocol, the model and the service's
own choice, exactly as it already is in OData-JSON; a client is not surprised to
receive them, in a wrapper object or positionally. As Hubert put it, it would be
very odd to change `$select` for this.

So `$select` is untouched, and only the context URL grammar is affected. That is a
materially smaller ask than the one recorded in session 5, and smaller than issue 1
rather than larger.

`type` is the one piece that differs, being included only when it is needed to
distinguish the declared type from the actual type. But that too is the service's
decision, and the service may put it in the wrapper or at the first position of the
positional array as it sees fit.

### Four points added to open issue 15

Working through the consequences surfaced four that had not been stated:

- **`count` and `nextLink` are out of scope.** They apply to a collection, not to an
  instance, so they have no per-instance position. Session 5 had wrongly cited
  `count` as an example of eligible control information; the eligible set is the
  instance-scoped information --- `id`, `etag`, `type`, `editLink`, `readLink` and
  the media ones.
- **Whether the ABNF can separate the two select-lists.** The context URL grammar is
  the [OData-ABNF] rule `context`. If its select-list shares the `selectItem` rule
  with `$select`, admitting control information in one but not the other needs a
  distinct rule rather than a widened shared one. This is the whole of the remaining
  cross-specification ask and should be checked before the proposal is written.
- **No duplication.** If the select-list places `type` at position 0, the wrapper
  object around that instance must not also carry it.
- **The base-type cost.** If `type` occupies position 0 then every instance carries
  something there, including instances of the declared type, which would use `{}`.
  That trades a small fixed cost on every instance against removing the wrapper from
  every derived instance; which wins depends on how derived-heavy the collection is.
  Since the service chooses per payload, the draft should say so rather than leave
  producers to guess.

### Open issues 1 and 14 merged

Agreed with Hubert: the two missing context URL templates --- one for request bodies,
one for action and function parameter payloads --- go to the TC as a single proposal.
Issue 1 now carries both as 1(a) and 1(b); issue 14 remains as a pointer so that
existing references do not break, and section 8.8 now points at 1(b).


### Review: a wrongly renamed `value` in the property-annotation example

Hubert reported that the `Core.ValueException` example in "Property Annotations"
showed the annotation's own value as `{ "$": "1234567890123456789" }`, where the
[OData-JSON](#ODataJSON) form has `{ "value": "1234567890123456789" }`.

**The report was correct.** `Core.ValueException` is a complex-typed term whose type
declares a property named `value` --- [OData-JSON](#ODataJSON) says the exact value
goes in "the `value` property of the annotation". It is a model-defined property
name, not the reserved name of a wrapper object's value, and renaming it to `$` was
simply wrong. The error was introduced in session 1, when the example was first
written with `_`, and the session 3 rename carried it into `$` unexamined.

Two things came out of it.

*The draft was missing a rule.* Nothing said that the *value* of an annotation is
represented as defined in [OData-JSON](#ODataJSON) and is never conveyed
positionally. It follows from D‑29 --- a positional representation needs a
select-list, and no select-list applies to an annotation's value --- but it was
nowhere stated, which is precisely how the mistake survived. The chapter introduction
now states it, and calls out the `value`-named-property case by name.

*The other half of the report was not an error, but the confusion was fair.* Hubert
also observed that `$` "is always of array type containing the positional property
values", which it is not here: the outer wrapper has `"$": 1234567890123456800`, a
number. That is correct and necessary --- a wrapper standing at the position of a
primitive property must hold that primitive under `$`, or an annotated primitive
property could not be represented at all. But every other example in the document
shows `$` holding an array, so the reading was a reasonable one to arrive at.
[Section ##TheWrapperObject] now says explicitly what `$` may hold --- a positional
representation, an [OData-JSON](#ODataJSON) object, a collection, or a primitive ---
and the example's caption points out why it is a number in this instance.

*Audit:* every non-array use of `"$"` in the document was checked. There were two,
both in this example: the erroneous one, now fixed, and the correct primitive.

## Session 7 — 2026-09-22 / 2026-09-23

Branch `odata-compact-json-format`, on top of commit `cdddf38`. Participants:
Hubert Heijkers and Claude Opus 5.

The first session driven by the review comments on
[PR #2270](https://github.com/oasis-tcs/odata-specs/pull/2270) rather than by the
draft itself. Hubert's instruction was to treat the comments as themes to be
applied consistently across the document, not as one-off edits at the lines where
a reviewer happened to notice them.

### Getting the comments into one place

`gh` is not installed in this environment, so the comments were read from the
public GitHub REST API: `pulls/2270/comments` (85 inline comments),
`issues/2270/comments` and `pulls/2270/reviews`. Grouped into reply chains they
form **66 threads** --- 55 comments from Mike Pizzo, 15 from Heiko Theissen, 10
from Hubert, 5 from Ralf Handl. Seven threads sit on outdated lines and were
already dealt with in sessions 2 to 6 (the file-per-section renumbering, examples
split into two code blocks, `_` to `$`, `value` at the root, the wrapper object
moved into chapter 4, derived types resolved per row).

The threads are written up as `odata-compact-json-format-pr2270-triage.md` in the
repository root, numbered `T01`–`T66` in document order. **That file is a working
artifact, not part of the specification, and is not committed.** Thread numbers
are quoted below and in the decision register so that a decision can be traced
back to the comment that prompted it.

Two facts worth recording. The PR's head branch `compact-json` and the local
branch have **identical specification sources** --- the PR carries rebuilds and
`main` merges on top --- so every comment maps directly onto the working tree. And
the REST API does **not** expose which threads have been resolved on GitHub, so
resolution status was not available to this session.

### The themes

Of the 59 live threads, most fall into eight themes. Hubert agreed to take them in
an order that puts the vocabulary changes first, so that later edits are not
rewritten: **A** the array represents properties, not the instance; **B** content
negotiation; **C** sender/receiver to producer/consumer; **D** annotation versus
control information; **E** intent rather than a string-parsing algorithm;
**F** justifying prose in normative sections; **G** select-list and the reliance on
`$metadata`; **H** gaps in the wrapper object's semantics. This session covers
**A** and **B**.

### Theme A --- the positional representation conveys *properties* (D‑32)

Twelve threads make the same point in different places (T06, T14, T27, T28, T43,
T45, T46, T51, T53, T57–T59). The draft said "a structured instance … MAY be
represented as a JSON array … Such an array is called the *positional
representation* of the instance." Mike's objection is that the array does not
represent the instance --- it represents the values of the instance's properties.
The instance may also carry annotations, control information and properties that
are not in the positional property list, none of which are in the array.

T58 goes furthest: an entity, entity reference or complex value is *always* an
entity, reference or complex value, and what varies is only whether its properties
appear as individual name/value pairs or collapsed into a single `$` member.

Hubert chose the full reframing. Section "Positional Representation" now opens
with the three forms a structured instance may take --- the OData-JSON object, the
bare positional array, and the wrapper object --- and says which of them this
format adds. Section "The Wrapper Object" states that where the value is a
structured instance, a wrapper object *is* the OData-JSON object representation of
that instance with the positional properties replaced by `$`. That is Mike's
reading, and it makes the wrapper object less of a new construct than the draft
had made it.

The change was applied to the glossary, the design chapter, chapter 4 throughout,
the message-body section of chapter 5, and the entity and complex-value sections of
chapter 6 --- not only where a reviewer had flagged it.

*Not adopted:* Mike's phrase "positional properties" for the properties in the
list, which he himself withdrew in T14 in favour of "known properties". Neither is
OData terminology, and the properties in question need not be declared --- a
selected dynamic property occupies a position. The document's own term,
*positional property list*, is used instead.

*Also applied, being inside the rewritten text:* T44 and T52, both of theme F. The
paragraph justifying why the format "introduces a single construct" is gone, and
the note explaining that `{}` satisfies a bullet "vacuously rather than by
construction" is replaced by a plain statement of what `{}` is.

### Theme B --- content negotiation (D‑33, D‑34, D‑35, D‑36)

Four threads (T17, T19, T21, T22) argue about the `406`/`415` rules, with Mike and
Heiko on one side and Hubert's earlier replies on the other. The substance:

- Hubert's reading of `406` (T17) is **correct as HTTP** --- a service returns `406`
  only when nothing in the client's `Accept` list is acceptable. But that is exactly
  why the sentence does not belong here: it restates ordinary content negotiation,
  and restated imprecisely it reads as a new obligation. Mike made both points
  (T19). The rule is removed (**D‑33**).
- Heiko's observation (T17, T22) is the one that settles it: [RFC2045](#rfc2045)
  requires a MIME implementation to **ignore parameters whose names it does not
  recognize**, so a service unaware of this specification does not fail --- it
  ignores `compact=true` and returns ordinary JSON. The superset principle makes
  that response safe to consume. Chapter 3 now says this, and draws the consequence
  the reviewers asked for: a client cannot infer support from a successful response,
  and must establish it from `Capabilities.SupportedFormats` or out of band.
- The same RFC 2045 reasoning undercuts a **MUST** on `415` for request bodies: it
  would impose a requirement on services that predate this document. Mike proposed
  SHOULD or MAY; Heiko asked why not plain `400`. Both are right about different
  services, so the draft now distinguishes them --- a service that *supports* this
  format SHOULD answer `415`; one that does not will read the body as OData-JSON
  and in most cases answer `400` (**D‑34**). Conformance clause 20 (`406`) is
  removed and clause 21 becomes the SHOULD.
- [RFC2045](#rfc2045) is new to this document and is added to the normative
  references.

**`metadata=full` (T25, D‑35).** The draft had it NOT RECOMMENDED because it
"forces every instance into a wrapper object and defeats the purpose of the
format". Mike pointed out that this was decided before the wrapper object took its
present form: a wrapper carries control information *alongside* `$`, so the saving
on property names survives; only the extra saving is lost. "Defeats the purpose"
was too strong. `metadata=full` is now permitted with the cost stated plainly, and
the "Services MAY reject the combination with `406`" sentence goes with it --- a
service that advertises both parameters should not then refuse them together.
**Supersedes D‑20.** Open issue 15, which rested on the old wording, is restated
accordingly.

**`streaming` (T23, T24, D‑36).** Two separate defects in one sentence. The
**MUST** made a payload non-conforming for happening to satisfy the ordering
constraints without declaring them, which Mike rightly called out --- it is now a
SHOULD, amending D‑22. And the justification, "a positional representation is
inherently ordered", is misleading: `streaming=true` is about the ordering of
control information, not of property values. The "Payload Ordering Constraints"
section already explains this correctly, so the clause is simply cut.

**Also applied in chapter 3:** `Accept` is now described before `$format` (T15);
the phrase "followed by the `compact=true` format parameter, optionally followed by
other format parameters" is gone, since it implied an ordering that format
parameters do not have (T15); the standalone case-insensitivity sentence is folded
into the Format Parameters section, where `compact` is defined, instead of
duplicating what OData-JSON and MIME already say (T16); the
`Capabilities.SupportedFormats` example advertises `streaming=true` rather than
`IEEE754Compatible=true` (T26); and "A service that does not support the compact
JSON format MUST NOT return a compact payload" is removed as self-evident (T19).

*Flagged for Hubert, not a decision:* reversing `Accept` and `$format` makes
chapter 3 differ in order from OData-JSON chapter 3, which introduces `$format`
first and `Accept` second. Nothing normative turns on it. Mike asked for the change
explicitly, so it was made; keeping the family consistent would mean the same
change in the JSON format specification.

### Found while editing, not from the review

The "Message Body" section of chapter 5 read "Each message body is represented as a
single JSON object … **with the single exception described in** [section
##MessageBodyofaRequest]". There is no exception: that section says a compact
request body "is never a bare JSON array", which is the same rule, and OData-JSON
states it without exception too. The cross-reference is now a plain pointer.

### Verification

`lib/number.js` was run over **all** documents in one pass, through a throwaway
harness mirroring `lib/build.js` but writing to a temporary directory, so that
cross-document links into `odata-protocol` resolve and `docs/` is left alone. All
eight documents resolve, including `odata-compact-json-format`; Pandoc 3.8.3
produced HTML without error. Of the 38 JSON examples, 37 parse standalone; the
38th is the `Capabilities.SupportedFormats` fragment, which is shown out of context
by design and is recorded as such in the housekeeping notes of session 1.

*Not verified:* `npm test`, PDF generation, and Pandoc's visual rendering.

*Note on the harness:* a first attempt ran `node lib/number.js odata-compact-json-format`
directly and exited 0 having done nothing --- the file is a module, and running it
as a script only checks the Pandoc version. The assistant reported this as a
successful validation before noticing. It is recorded here because the same
mistake would look like a passing check to anyone repeating it.

## Session 8 — 2026-09-23

Branch `odata-compact-json-format`, continuing the PR #2270 review. Participants:
Hubert Heijkers and Claude Opus 5. Theme G of the eight identified in session 7.

### Theme G --- the select-list and what a position may hold (D‑37 to D‑40)

Threads T29, T31–T35 and T40, plus open issue 9.

**Nested context URLs may define a positional property list (D‑37).** Heiko's
T32 asks whether a nested `Prop@context` can do this. Hubert: a definite yes. A
new section, "Nested Context URLs", says that where a wrapper object carries its
own `context`, that context URL's select-list determines the positional property
list of the instances it describes, in place of the nested select-list otherwise
derived from the containing context URL. The case that makes it matter is Heiko's
own: a property carried *by name* in a wrapper object occupies no position and so
has no nested select-list at all, and without this rule its value could not be
conveyed positionally. [OData-JSON](#ODataJSON) already requires a nested context
URL where a nested collection's entity set cannot be determined from the
containing one, so the construct is not new --- only its effect on the positional
property list is.

**Both shortcuts stay banned, and now for a stated reason (D‑38).** Mike's T33
asks why `{namespace}.*` cannot appear alongside the properties.

Two corrections came out of checking this. First, the premise: `{namespace}.*`
selects **bound operations**, not annotations --- `Model2.*` is "all actions and
functions in schema `Model2` that bind to this entity"
([#OData-URL#SystemQueryOptionselect]). There is no `@ns.*` star for annotations;
instance annotations are always named individually, which is exactly why [section
##SelectedAnnotations] can give a selected annotation a position. Second, the
assistant's initial recommendation --- that operations should never occupy
positions, which would make `{namespace}.*` harmless and let Mike have his yes ---
was argued from the wrong premise. It rested on [OData-JSON](#ODataJSON) leaving a
service free to advertise a bound operation or not, and on D‑05 forbidding
omissions, and concluded that a position would strip the service of that freedom.

Hubert's answer: the select-list in question is the one the **service** writes in
the context URL, so the service simply enumerates what each position holds. That
is D‑29's principle, and it dissolves the objection --- the freedom not to
advertise is exercised when the context URL is written, not per instance. Both
`*` and `{namespace}.*` are therefore excluded, and [section
##DeterminingthePositionalPropertyList] now says why: a shortcut names a rule for
finding a set rather than the set itself, leaving sender and receiver each to
resolve it against a metadata document whose version they need not agree on and
whose declaration order [OData-CSDL](#ODataCSDL) does not make significant, and
saying neither how many positions it occupies nor in what order.

**Open issue 9 is largely closed (D‑39).** A new section, "Bound Operations",
makes normative what the draft had left as an unexamined consequence: a
select-item naming a bound action or function occupies a position; the value there
is the advertisement, or `null` where the service advertises non-availability ---
which is the value [OData-JSON](#ODataJSON) already defines for that, so no new
sentinel is needed and `{}` keeps the single meaning D‑27 gave it. The
`#`-prefixed name is not transmitted, the position identifying the operation. An
operation advertised only for instances of a derived type is named with a
type-cast segment, on the model of D‑26; where the instances are not distinguished
by type, the service advertises it by name in a wrapper object instead.

*What remains open:* whether the [OData-ABNF](#ODataABNF) rule
[selectItem]{.abnf} admits a type-cast segment before a qualified operation name
as it does before a property name. If it does not, the fallback is already in the
text. Open issue 9 now records this single question instead of the whole topic.

**The reliance on `$metadata`, stated (D‑40).** Heiko's T29 and T40 make the point
that a consumer cannot tell an array that is a collection from an array that is a
nested instance without knowing the property's cardinality, and that this
dependency is new relative to the format defined by [OData-JSON](#ODataJSON),
where a property name accompanies its value. [Section
##PositionalRepresentation] now says so directly --- "a positional representation
is not self-describing" --- with a two-payload example whose positional
representations are identical in shape and differ only in what the metadata says.
The existing note in [section ##PositionValues] now points at it rather than
asserting the same thing weakly a second time.

**Also applied:** T34, the select-list bullet now says "a structural or navigation
property of a structured type"; T35, the rule about implicit selection is stated
positively, as what the select-list MUST enumerate, rather than as a rule of
[OData-Protocol](#ODataProtocol) that does not apply; T31, Heiko is right that
narrowing the select-list shortens the array too, so the trailing clause goes.

**A consequence for theme A.** Session 7 defined the positional representation as
conveying "the values of the properties" in the positional property list. With
selected annotations and now bound operations occupying positions, that is too
narrow. The definition in [section ##PositionalPropertyList] and the glossary
entry now say what the list actually holds --- properties, and where the
select-list names them, selected instance annotations and bound operations --- while
keeping the established name, which D‑02 and D‑29 both use.

### Verification

The all-documents harness described in session 7 was re-run: all eight documents
resolve, including the two new sections and every reference to them. 41 of the 42
JSON examples parse standalone; the 42nd remains the `Capabilities.SupportedFormats`
fragment.

### Not a change: `Accept` before `$format`

Session 7 flagged that reversing the two made chapter 3 differ in order from
[OData-JSON](#ODataJSON). Hubert: the order is only the order in which the two
ways of requesting the format are shown, and the sentence that follows them ---
"If specified, `$format` overrides any value specified in the `Accept` header" ---
still states the precedence, which is unchanged and always has been. No action.

## Session 9 — 2026-09-23

Branch `odata-compact-json-format`, continuing the PR #2270 review. Participants:
Hubert Heijkers and Claude Opus 5. Theme H, the semantics of the wrapper object.

### Everything about one property is in one place (D‑42)

Heiko's T48 asks the question the draft had left unanswered. The draft forbade a
*property* that occupies a position from also being carried by name, but said
nothing about that property's annotations. Is

```json
{"$": ["ALFKI", "Alfred"],
 "Name@Core.Permission": "Read"}
```

forbidden, or is it another way of writing the annotation that would otherwise sit
in a wrapper object at `Name`'s position?

**Hubert: forbidden.** If a property occupies a position, everything that applies
to it --- annotations and control information alike --- is carried in the wrapper
object at that position. Carrying it by name instead makes such payloads
unnecessarily painful to process, and it is not what
[OData-JSON](#ODataJSON) does either, where the annotations of a property are
grouped with, and precede, that property's value.

[Section ##TheWrapperObject] now states the prohibition in both directions --- a
positional property is not carried by name, and neither is anything that qualifies
it; a property carried by name takes its annotations and control information with
it, under the prefixed names [OData-JSON](#ODataJSON) gives them --- with the
consequence spelled out: a receiver that has read the value at a position never
has to look elsewhere for something that qualifies it, and one that reads a
name/value pair never has to check whether that property also occupies a position.
[Section ##PropertyAnnotations] repeats the rule where a reader looking for
annotations will meet it, with an example showing Heiko's payload as the form that
is *not* permitted. [Section ##OpenTypesandDynamicProperties] and conformance
clause 15 are extended to match.

### What "carried by name" means, and what it follows (T47, T49)

Two of Mike's questions turned out to be about the same sentence. "Carried by
name" was never defined --- the term now is, on first use: a property appears as an
ordinary name/value pair of the wrapper object, as [OData-JSON](#ODataJSON)
represents it, instead of occupying a position. And "MUST follow the value" did
not say *which* value; Heiko answered it on the thread and the text now says it:
after the wrapper object's `$` name/value pair. The reason is added, since without
it the constraint looks arbitrary --- a receiver reading the payload as a stream has
the whole positional representation in hand before it meets any property that is
not part of it.

### Why a producer would ever choose `value` (T54)

Ralf asked what a sender gains by choosing `value` over `$`, and in which cases
that would be helpful. The draft recommended `$` without saying. It now answers
him: nothing is *saved* --- `value` is the longer name --- and what it offers is that
a client written for [OData-JSON](#ODataJSON), which looks for `value` in the
message body of a collection, finds the collection where it expects it. That is
the superset principle, and it is the only reason to prefer it.

Mike's reply on the same thread, that one would "use `value` for singleton and `$`
for array", has it the other way round: `value` is recognized *only* in those
message bodies where [OData-JSON](#ODataJSON) itself uses it, which are the
collection cases, never the single entity or complex value. The text already said
so; the clarified rationale should stop the reading recurring.

### Heiko was right about the justifying example (T55, D‑43)

The paragraph defending the restriction argued that
`{"@context": "…#Customers/$entity", "value": […]}` would be at once a positional
representation and an [OData-JSON](#ODataJSON) entity with a collection-valued
property named `value`. Heiko pointed out that it would **not** be a valid
positional representation, the context URL having no select-list --- which D‑29
requires.

He is right, and checking it showed the claim was also overstated. Because a
positional representation has exactly as many items as the list has positions, the
two readings usually differ in shape and *are* distinguishable given the metadata.
Constructing a genuinely ambiguous case needs a select-list, a property named
`value`, and that property to be loosely enough typed to accept either reading.
The example is replaced by one that meets all three --- select-list `(ID,value)`,
`value` of type `Collection(Edm.Untyped)` --- and the claim is narrowed to what is
true: a receiver would have to consult the metadata to choose a reading, and where
the property is untyped the metadata does not settle it either. The restriction
stands; its justification is now sound.

### Verification

All eight documents resolve through the session 7 harness. 44 of the 45 JSON
examples parse standalone; the 45th remains the `Capabilities.SupportedFormats`
fragment.

## Session 10 — 2026-09-23

Branch `odata-compact-json-format`, closing out the PR #2270 review. Participants:
Hubert Heijkers and Claude Opus 5. Themes C, D, E and F, and the remaining
one-off threads. **With this session all 66 threads are addressed.**

### Theme C --- producer and consumer (D‑44)

Mike's T11 and T30. The conformance section already spoke of a "conforming
producer" and a "conforming consumer" while the body spoke of senders and
receivers; the sibling specifications do not use "sender" at all. 38 occurrences
swept. Three sentences needed more than the swap: the introduction had become
"the producer producing and the consumer consuming", and the superset example
"a producer that cannot produce".

Sessions 7 to 9 deliberately left the old terms in the text they rewrote, so that
this sweep would remain a single reviewable change rather than being scattered
through four commits.

### Theme D --- control information where control information is meant (D‑45)

Three threads, and a fourth thing found while checking them.

- Ralf's T66: the bind operation is **control information**, not a property
  annotation. Corrected.
- Mike's T56: a property with no value at all can result from control information
  having been requested for it, not only annotations. Corrected.
- Mike's T51 was already taken in session 7, the wrapper object's placement
  bullets now naming control information alongside annotations.
- *Found while checking:* [section ##PropertyAnnotations] and conformance clause 6
  said only that **annotations** drop the property-name prefix --- yet the example
  sitting under them is `Orders@count`, which is control information, and D‑07's
  own wording uses `Orders@count` as its illustration. The rule always covered
  both; the text did not say so. Both now do.

### Theme E --- intent, not an algorithm over strings (D‑46)

Mike's T39: "I dislike being quite so tied to the exact syntax of the contextUrl,
and would rather describe the intent than an algorithm for string parsing." T36
and T37 say the same of the three numbered steps that determined the positional
property list.

The steps are gone. [Section ##DeterminingthePositionalPropertyList] now opens
with what the list *is* --- the properties the select-list names that apply to the
instance's type, each at the place the select-list first names it --- and then
states the two things that make it precise: a type-cast select-item names a
property only for instances of that type, and several select-items may name the
same property and share one position. [Section ##GroupingofSelectItems] keeps the
precision an implementer needs but leads with the intent, and names the property
*P*, which Heiko asked for in T38 because the text used *P* in the bullets without
introducing it.

*Found while rewriting:* the grouping section cited "step 3" of the numbered
algorithm for what was step 1. The rewrite removes the reference.

Two examples were improved with it. T41: the grouping example now selects
`ID,Address/City,Name,Address/PostalCode`, so that `Name` visibly takes the
position *after* `Address` rather than between its two sub-properties --- the
rearrangement was invisible in the old example, where the grouped items were
already adjacent. T42: Mike read the `DirectReports` example as losing the nested
`FirstName,LastName`; it does not, and the example now says what the group's
nested select-list is.

### Theme F --- less justification in normative text

Mike's T44 and T52 were taken in session 7, being inside text rewritten there.
What remained was design principle 2 (T08, T09, T10), which had three separate
problems in four lines: the saving is in *avoiding* the repetition, not in the
repetition; the property names are not removed from the payload but transmitted
once in the context URL; and the optionality of the positional representation does
not follow from the benefit being negligible for a single instance. All three
corrected.

### One-offs

- **T01.** Mike asked whether this document extends the core protocol or the JSON
  format. The sentence is house boilerplate shared with
  [OData-JSON](#ODataJSON), so it is kept and extended rather than replaced: the
  document is an extension of the core protocol, *and* of the JSON format, which
  is now said.
- **T02.** "the uncompressed size of OData **JSON** request and response payloads".
- **T03.** The comparison to "traditional rowset interfaces" is struck. Mike was
  not sure what it implied and thought the comparison to CSV closer; it carried
  no weight either way.
- **T07.** The superset principle said a compact consumer "accepts strictly more
  payloads". Mike: the point is that it accepts [OData-JSON](#ODataJSON), not that
  it accepts more. Reworded.
- **T13.** "for that instance *alone*" struck, a producer being free to do it for
  any number of instances in a payload.
- **T18** is moot: Ralf's correction applied to the `406` sentence, which D‑33
  removed. **T20** needed no change, Hubert having answered Mike on the thread and
  the advertising example now showing combinations of parameters.
- **T65 --- a real contradiction, now fixed.** Ralf noticed that the `PATCH`
  section discussed "a `PATCH` request body that does not contain the `context`
  control information" while [section
  ##DeterminingthePositionalPropertyListinRequests] requires one of *every*
  compact request body. The paragraph predates that requirement --- it survives
  from D‑11, when a request body only *MAY* carry a context URL. It is replaced by
  what now follows from the rule: `context` is what makes `PATCH` expressible in
  this format at all, and without it there would be no way to say which properties
  a `PATCH` leaves alone.

### Not adopted

**T53.** Mike suggested "The name of the positional property values in a wrapper
object is `$`". Left as "the name of the value", because `$` does not always hold
positional property values --- a wrapper standing at the position of an annotated
primitive property holds that primitive under `$`, as session 6 established and as
[section ##TheWrapperObject] states explicitly.

### Verification

All eight documents resolve. 44 of the 45 JSON examples parse standalone; the 45th
remains the `Capabilities.SupportedFormats` fragment.

## Session 11 — 2026-09-23

Branch `odata-compact-json-format`. Participants: Hubert Heijkers and Claude
Opus 5. Not a review thread --- an addition Hubert proposed to the TC years ago for
the OData JSON format itself, contributed here.

### Grouping a property's annotations with its value (D‑48)

[OData-JSON](#ODataJSON) puts a property's annotations inside the object
representing that property's value where it can. Where it cannot --- the value
being a primitive, a collection or `null`, none of which can hold a name/value
pair --- it puts them in a separate pair carrying the property name before the
`@`. The annotations and the value then sit apart.

The wrapper object this format introduced for positional representations solves
exactly that problem, and nothing about it depends on positions. [Section
##WrapperObjectsinanObjectRepresentation] therefore permits a wrapper object as
the value of a name/value pair in a JSON object representing a structured
instance, carrying that property's control information --- `@type` most obviously
--- its annotations, and its value under `$`.

This is an addition to the format, not a review comment applied. It is also the
closing of an asymmetry the review work opened: D‑42 put everything that applies
to a positional property at its position, leaving a property conveyed by name
still obliged to use prefixed names. It need not be.

**Scope --- and a drafting overreach, corrected within the session.** Hubert asked
for primitive-typed properties and collection-valued ones: the cases where
[OData-JSON](#ODataJSON) has to repeat the property name in a
`Property@ns.term` pair because the value --- a primitive or an array --- cannot
hold a name/value pair of its own. The assistant proposed instead to allow a
wrapper at the value of *any* property, on the ground that [section
##TheWrapperObject] already says a wrapper may appear wherever a value may
appear, and offered that as the recommended option. Hubert took it.

It was the wrong recommendation, and checking it against the superset principle
showed why. Extending the wrapper to a **single-valued structured** property
collides with the second of the three recognition rules of [section
##TheWrapperObject]: an object of nothing but annotations and control information
is a wrapper carrying no value. A complex value carrying an annotation and no
properties --- `"Address": {"@Core.Permissions": "Read"}` --- is exactly such an
object and is valid under [OData-JSON](#ODataJSON). Reading it as a value-less
wrapper would change the meaning of a payload that D‑01 requires a compact
consumer to read unchanged.

The assistant's response was to patch the symptom: require `$` in a wrapper at a
property's value, disapplying the second recognition rule there. Recorded as
D‑49, flagged as unasked-for, and put on the agenda for the TC to confirm.

**Hubert rejected the patch and removed the cause.** The allowance covers
primitive-valued and collection-valued properties only. A single-valued property
of a structured type is excluded, because the object representing its value
already carries its own annotations and control information within itself, where
[OData-JSON](#ODataJSON) puts them and where no property name is repeated --- so a
wrapper buys nothing there. With that scope there is no object-valued property at
which a wrapper may stand, the collision cannot arise, and D‑49 is unnecessary:
the second recognition rule needs no exception because the case it would have
misread no longer occurs. D‑50 records the scope; D‑48 and D‑49 are superseded.

The lesson for the register: "a wrapper may appear wherever a value may appear"
read as an argument for uniformity, but the uniformity was only apparent. The
wrapper earns its place where the value cannot hold a name --- at a position, and
at a primitive or a collection. Where the value is an object it already has one.

**Placement.** Hubert: "I would have loved it to be part of OData JSON back then
but with the implementation of Compact JSON I'm more than happy if it is allowed
in Compact JSON." So it is specified here, with no open issue proposing its
relocation. The practical consequence is that using it makes a payload a compact
payload, needing `compact=true` --- which a service may negotiate without ever
sending a positional array, and [section
##WrapperObjectsinanObjectRepresentation] says so.

### Verification

All eight documents resolve. 48 of the 49 JSON examples parse standalone; the
49th remains the `Capabilities.SupportedFormats` fragment.

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
| ~~D‑20~~ | ~~`metadata=none` and `minimal` are permitted; **`metadata=full` is NOT RECOMMENDED** — it forces every instance into a wrapper and defeats the format.~~ **Superseded by D‑35.** | Assistant | Superseded | §3.1 |
| D‑21 | The `context` control information is **always required**, including under `metadata=none`, since without it the payload cannot be interpreted at all. | 2018 proposal, retained | TC | §7.5.1 |
| D‑22 | `streaming` remains meaningful and is **not forced** to `true`, departing from the 2018 position; a compact payload meeting the constraints declares `streaming=true`. **Amended by D‑36**, which makes that a SHOULD rather than a MUST. | Assistant | Provisional, amended | §3.1, §4.3 |
| D‑23 | CSDL **declaration order must be stable** for a given metadata document URL; services SHOULD reference versioned metadata from the context URL. | Assistant, forced by D‑02 + D‑04 | Provisional | §5.2, open issue 3 |
| D‑24 | Examples use the **`Customers`/`Orders`/`Address`** model of the sibling specifications, not the `Cubes`/`Dimensions` model of the 2018 proposal. | Assistant, for house consistency | Provisional | throughout |
| D‑25 | A wrapper object **MAY** carry data properties by name, after `$`, so that an open-type instance keeps its positional representation instead of falling back wholesale to the [OData-JSON] object form. A wrapper is then recognized by the presence of `$`, or by all of its members being annotations. **Supersedes D‑09.** | TC direction relayed in session 3 | **TC**, drafted | §6.8, §7.1, open issue 6 |
| ~~D‑14~~ | ~~Every instance in a collection shares one positional property list; a derived-type property selected via a type cast occupies a position in every instance, and `{}` fills the gaps.~~ **Superseded by D‑26.** | Assistant, session 1 | Superseded | §6.7 |
| D‑26 | The positional property list depends on the instance's **type** as well as the context URL: a type-cast select-item contributes a position only to instances of that type or a type derived from it. No gaps and no placeholders. Consequently `type` becomes load-bearing — a service **MUST** send it whenever an instance's list differs from that of the context URL's declared type, **irrespective of `metadata=none`**, and it must precede the positional representation. **Supersedes D‑14.** | TC direction relayed in session 3, in preference to gap-filling or grouping by type cast | **TC** | §5.2 step 3, §6.7, §7.5.3 |
| D‑27 | `{}` is retained, but **only** for a selected dynamic property that an instance does not have. It is no longer a wrapper by the D‑25 recognition rule, and conveys "undefined" by convention; the TC accepted this explicitly. | Hubert, session 3 | **TC** | §6.8, §7.1, open issue 5 |
| ~~D‑04~~ | ~~Where structural properties are implicitly selected, they are prepended in CSDL declaration order, base type first.~~ **Retired by D‑29** — nothing is implicitly selected any more. | Assistant, session 1 | Retired | — |
| ~~D‑12~~ | ~~A request message body MAY be a bare JSON array, the request URL supplying the type.~~ **Retired by D‑29.** Also wrong independently: [OData-JSON] wraps request bodies in an object deliberately, so that control information always has a place and the body's shape does not vary. | Assistant, session 1 | Retired | §8.2 |
| ~~D‑23~~ | ~~CSDL declaration order must be stable for a given metadata document URL; services SHOULD reference versioned metadata.~~ **Retired by D‑29** — the dependency on declaration order is gone. | Assistant, session 1 | Retired | — |
| D‑29 | The context URL of a compact payload **MUST** carry a select-list enumerating every property conveyed positionally, at every level; `*` and `{namespace}.*` are forbidden and implicit selection does not apply. This constrains the **context URL, not the request**: `$select=*` and an absent `$select` stay valid, and the service enumerates what it returned. **Retires D‑04, D‑12, D‑23; closes open issue 3.** | TC, session 5. A service always knows which CSDL version it used; a client composing a request body does not and cannot learn it from the payload | **TC** | §4.3, §8.1, §8.2, §13 |
| D‑30 | A parameter value of an action or function **MUST NOT** use the positional representation, there being no context URL for a parameter payload. Recorded as a limitation awaiting a mechanism, not a considered exclusion. | Consequence of D‑29; TC to propose a mechanism | **TC**, provisional pending open issue 14 | §8.8 |
| D‑31 | If type information is conveyed positionally, `type` **MUST** occupy the first position of the array. Minimal sufficient rule, `type` being the only thing that must be read before the positional property list is known. | Hubert, session 5, in preference to the assistant's broader "type-independent prefix" | **TC** (in principle), not drafted — gated on open issue 15 | open issue 15 |
| D‑28 | Chapter 4, **"Compact Representations"**, defines both constructs the format adds — the positional representation and the wrapper object (§4.5) — and "Common Characteristics" moves to chapter 5. The wrapper is no longer filed under annotations, since three of the four things it carries are not annotations. | Reviewer comment, session 4; forward references to the wrapper fell from 13 to 3 | **TC** (reviewer-driven) | §4, §5, §7 |
| D‑32 | A positional representation conveys the **values of an instance's properties**, not the instance. A structured instance takes one of three forms: the [OData-JSON] object, the bare positional array, or a wrapper object — which, for a structured instance, *is* the [OData-JSON] object representation with the positional properties replaced by `$`. | PR #2270 T06, T14, T27, T28, T43, T45, T46, T51, T53, T57–T59 (Mike Pizzo); full reframing chosen by Hubert, session 7 | **TC** (reviewer-driven) | §1.1, §2, §4.1, §4.5, §5.2, §6.1, §6.2 |
| D‑33 | The **`406 Not Acceptable` rule is removed.** It restated ordinary HTTP content negotiation; restated imprecisely it read as a new obligation. Hubert's reading of it was correct as HTTP, which is precisely why the document need not say it. | PR #2270 T17, T19 | **TC** (reviewer-driven) | §3, §13 clause 20 |
| D‑34 | A service that **supports** this format and cannot accept a `compact=true` request body **SHOULD** answer `415`; a service unaware of this specification reads the body as [OData-JSON] and will in most cases answer `400`. A MUST would bind services that predate this document, since [RFC2045] tells an unaware implementation to ignore the parameter. **Supersedes the MUST of §3.** | PR #2270 T21, T22 (Mike Pizzo, Heiko Theissen) | **TC** (reviewer-driven) | §3, §13 clause 20 |
| D‑35 | **`metadata=full` is permitted** with `compact=true`. A wrapper object carries control information alongside `$`, so the saving on property names survives and only the further saving is lost; the cost is stated instead of the combination being discouraged. The "services MAY reject with `406`" sentence is dropped. **Supersedes D‑20.** | PR #2270 T25 (Mike Pizzo) — the NOT RECOMMENDED predated the wrapper's present form | **TC** (reviewer-driven) | §3.1, open issue 15 |
| D‑36 | A compact payload meeting the payload ordering constraints **SHOULD** — not MUST — include `streaming=true`, and the claim that "a positional representation is inherently ordered" is cut: `streaming` concerns the order of control information, not of property values. **Amends D‑22.** | PR #2270 T23, T24 (Mike Pizzo) | **TC** (reviewer-driven) | §3.1 |
| D‑37 | A **nested context URL** determines the positional property list of the instances it describes, in place of the nested select-list otherwise derived from the containing context URL, and MUST itself carry a conforming select-list. Makes it possible to convey positionally a property carried *by name* in a wrapper object, which occupies no position and so has no nested select-list. | PR #2270 T32 (Heiko Theissen); Hubert, session 8 | **TC** | §4.5, §7.5.1, §13 clause 19 |
| D‑38 | **Both shortcuts are excluded** from the select-list of a compact payload, `*` and `{namespace}.*` alike, and the document now says why: a shortcut names a rule for finding a set rather than the set itself, and the service writing the context URL already knows what it placed at each position. Reaffirms the ban with a reason; the assistant had recommended admitting `{namespace}.*` by making operations non-positional. | PR #2270 T33 (Mike Pizzo); **Hubert overruled the assistant's recommendation**, session 8, applying D‑29's principle that the service enumerates | **TC** | §4.3 |
| D‑39 | A select-item naming a **bound operation** occupies a position; the value there is the advertisement, or `null` where the service advertises non-availability — the value [OData-JSON] already defines, so no new sentinel and `{}` keeps its D‑27 meaning. The `#`-prefixed name is not transmitted. Derived-type case follows D‑26 via a type-cast segment, with advertisement by name in a wrapper as the fallback. **Largely closes open issue 9.** | Session 8; makes normative what the draft had left as an unexamined consequence | **TC**, one ABNF question open | §7.4, open issue 9 |
| D‑40 | The format's **reliance on the metadata document is stated explicitly**: a positional representation is not self-describing, and a receiver needs the cardinality of a property to tell a collection from a nested instance. A stronger dependency than [OData-JSON] creates, and now said so rather than left to be inferred. | PR #2270 T29, T40 (Heiko Theissen) | **TC** (reviewer-driven) | §4.1, §4.7 |
| D‑41 | The **positional property list is not only properties**: it holds the instance's properties and, where the select-list names them, selected instance annotations and bound operations. Widens the definition D‑32 gave it while keeping the established name used by D‑02 and D‑29. | Consequence of D‑39 and of the existing treatment of selected annotations, session 8 | **TC** | §1.1, §4.2 |
| D‑42 | **Everything that applies to a positional property is carried at its position.** A property occupying a position is not carried by name, and neither are its annotations or control information; a property carried by name takes its annotations and control information with it, prefixed as [OData-JSON] writes them. So a receiver never has to look in two places for one property. | PR #2270 T48 (Heiko Theissen); **Hubert**, session 9 — processing cost, and [OData-JSON] groups a property's annotations with its value too | **TC** | §4.6, §6.8, §7.3, §13 clause 15.1 |
| D‑43 | The justification for restricting the name `value` is **narrowed to what is true**: a receiver would have to consult the metadata to choose a reading, and where the property is untyped the metadata does not settle it either. The former claim of outright ambiguity was overstated, and its example was invalid for want of a select-list. The restriction itself is unchanged. | PR #2270 T55 (Heiko Theissen), who was right | **TC** (reviewer-driven) | §4.6 |
| D‑44 | The document says **producer** and **consumer**, not sender and receiver, matching its own conformance section and the sibling specifications. | PR #2270 T11, T30 (Mike Pizzo) | **TC** (reviewer-driven) | throughout |
| D‑45 | **Property-level control information is treated exactly as a property annotation**: carried in the wrapper object at the property's position, with the property-name prefix dropped. Always what D‑07 meant — its own illustration is `Orders@count` — but §7.3 and conformance clause 6 had said only "annotations". **Clarifies D‑07.** | PR #2270 T56, T66 (Mike Pizzo, Ralf Handl), plus the gap found in checking them | **TC** | §4.7, §7.3, §8.6, §13 clause 6 |
| D‑46 | The positional property list is defined **by what it is**, not by a three-step algorithm over context-URL strings: the properties the select-list names that apply to the instance's type, each at the place the select-list first names it. Grouping keeps the precision an implementer needs but leads with the intent. | PR #2270 T36, T37, T39 (Mike Pizzo), T38 (Heiko Theissen) | **TC** (reviewer-driven) | §4.3, §4.4 |
| D‑47 | The `PATCH` section no longer contemplates a request body **without** `context`, which contradicted the requirement that every compact request body carry one. The paragraph survived from D‑11, when a request body only *MAY* have carried a context URL; D‑29 made it mandatory and the paragraph was not revisited. | PR #2270 T65 (Ralf Handl) | **TC** (reviewer-driven) | §8.4 |
| ~~D‑48~~ | ~~A wrapper object MAY stand as the value of a name/value pair in a JSON object representing a structured instance, **uniform across property types**.~~ **Superseded by D‑50**, which narrows the scope. | Assistant recommended the uniform scope; it was the wrong recommendation | Superseded | §4.7 |
| ~~D‑49~~ | ~~A wrapper object at a property's value MUST contain `$`, disapplying the second recognition rule of §4.6 there.~~ **Unnecessary under D‑50** — a patch on a symptom of the scope D‑48 had taken. With single-valued structured properties excluded, no object-valued property admits a wrapper and the collision cannot arise. | Assistant, session 11; withdrawn the same session | Withdrawn | — |
| D‑50 | A wrapper object MAY stand as the value of a **primitive-valued or collection-valued** property in a JSON object representing a structured instance, grouping that property's control information, annotations and value under one name instead of the prefixed `Prop@ns.term` form — the cases where [OData-JSON] must repeat the property name because the value cannot hold a name/value pair. **Not** a single-valued structured property, whose object representation already carries them within itself. Nothing in it depends on a positional representation. **Supersedes D‑48 and D‑49.** | **Hubert**, session 11 — proposed years ago for the JSON format itself, contributed here; specified in this document only, at his choice | **TC** (editor's proposal) | §4.6, §4.7, §13 clause 5.5 |

---

## What needs your call next

Ordered by how much rework a late reversal would cause.

### 0. PR #2270 — all 66 threads addressed

Sessions 7 to 10 worked through the eight themes identified in the review of
[PR #2270](https://github.com/oasis-tcs/odata-specs/pull/2270): **A** the array
conveys properties, **B** content negotiation, **G** the select-list and the
reliance on `$metadata`, **H** the wrapper object's semantics, **C**
producer/consumer, **D** annotation versus control information, **E** intent
rather than an algorithm, **F** justifying prose --- and the one-off threads.
Decisions D‑32 to D‑47 record what changed and why; `T`-numbers in those rows
refer to `odata-compact-json-format-pr2270-triage.md`, which is a working artifact
and is not committed.

Two things to be aware of when the PR is updated:

- The REST API does not expose which threads were **resolved** on GitHub, so this
  work proceeded from all 66 regardless of their resolution state. Some may have
  been resolved already.
- The PR's head branch `compact-json` carries rebuilds and `main` merges that this
  branch does not; the specification sources were identical when the review was
  read.

### 1. Cross-part dependency — raise with the TC early

**Open issue 1: context URLs in request bodies (D‑11) — now a blocker, not a
nicety.** Part 1 states that request payloads generally do not require context
URLs, and defines a request-body template only for `#$delta`. Since D‑29 a compact
request body MUST carry one, so without a Part 1 change compact request payloads
cannot be specified at all. Everything in chapter 8 rests on this.

Issue 1 now also covers parameter payloads (former issue 14, D‑30), the two being
put to the TC as one proposal.

**Open issue 15: control information in positions (D‑31).** Under D‑35
`metadata=full` is permitted and every instance then takes a wrapper object for its
control information; positions for control information would remove that wrapper
and recover the remaining saving. Downgraded
in session 6: because the context URL's select-list is service-authored, `$select`
is untouched and only the context URL grammar is affected. The remaining question is
whether the [OData-ABNF] `context` rule can admit control information without
widening `selectItem` and so changing `$select` too — worth checking before the
proposal is written. The `type`-first rule (D‑31) is settled in principle.

**Open issue 2: the 100% losslessness goal.** You said the TC would consider
changing the JSON format itself to reach it. The constructs currently *without* a
positional counterpart are listed in D‑17. Is that list acceptable, or should
some of them drive JSON format changes?

### 2. Settled since — kept here for the record, no action needed

- **Open issue 3: property order (D‑23 → D‑29).** Closed in session 5. The draft no
  longer derives the positional property list from the CSDL document at all, so
  declaration order and versioned metadata URLs are no longer load-bearing.
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
