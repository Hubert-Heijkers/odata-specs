
## T01 — odata-compact-json-format/1 Introduction.md @ line 9 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963492520
```diff
+# ##sec Introduction
+
+The OData protocol is comprised of a set of specifications for
+representing and interacting with structured content. The core
+specification for the protocol is in [OData-Protocol](#ODataProtocol);
+this document is an extension of the core protocol. Representations for
```
**mikepizzo** (2026-09-09):
extension of the core protocol, or of the JSON format?


## T02 — odata-compact-json-format/1 Introduction.md @ line 15 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963499408
```diff
+OData requests and responses using the JavaScript Object Notation (JSON),
+see [RFC8259](#rfc8259), are defined in [OData-JSON](#ODataJSON).
+
+This document defines a *compact* JSON format: a lossless, alternative
+representation of the same information, whose prime aim is to minimize
+the uncompressed size of OData request and response payloads.
```
**mikepizzo** (2026-09-09):
```suggestion
the uncompressed size of OData JSON request and response payloads.
```


## T03 — odata-compact-json-format/1 Introduction.md @ line 25 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963516716
```diff
+uncompressed payload still deal with the full volume of JSON text. The
+repetition of property names, once per property per instance, dominates
+that volume in exactly the payloads where it hurts most: large collections
+of entities and large collections of complex values.
+
+The approach taken in this format, inspired by traditional rowset
```
**mikepizzo** (2026-09-09):
I'm not sure what is implied by "traditional rowset interfaces".  I think of this more similar to CSV, but not sure we need the comment at all.


## T04 — odata-compact-json-format/1 Introduction.md @ line 30 (OUTDATED) — ralfhandl 2026-08-26
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3864061398
```diff
+The approach taken in this format, inspired by traditional rowset
+interfaces, is to represent a structured instance as a JSON *array* rather
+than a JSON object, so that a property name is transmitted at most once
+per payload instead of once per instance. JSON arrays preserve the order
+of their items, whereas the order of the name/value pairs of a JSON object
+is not significant; this format uses that property of arrays to convey,
```
**ralfhandl** (2026-08-26):
```suggestion
is not significant; this format uses that feature of arrays to convey,
```

**Hubert-Heijkers** (2026-08-28):
Yeah, I see, property appears twice with two different meanings but 'feature' is a bit vague (and faintly marketing-flavored for a normative document according to Claude <LOL>) so changed it to read:

```
The items of a JSON array are ordered, whereas the name/value pairs of a JSON object are not; this format relies on that ordering to convey, by position, which value belongs to which property.
```


## T05 — odata-compact-json-format/1 Introduction.md @ line 111 (OUTDATED) — HeikoTheissen 2026-08-24
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3843730947
```diff
+
+All other text is normative unless otherwise labeled.
+
+-------
+
+# ##sec Compact JSON Format Design
```
**HeikoTheissen** (2026-08-24):
Since this starts section 2, all subsequent file names are off by 1. ("2 Requesting the ..." should be "3 Requesting the ..." and so on.)

**Hubert-Heijkers** (2026-08-27):
Created a file per section and fixed up the numbering accordingly

**HeikoTheissen** (2026-08-27):
But you did not push that change yet, right?


## T06 — odata-compact-json-format/2 Compact JSON Format Design.md @ line 12 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963552752
```diff
+name/value pairs that annotate a JSON object, property or array, and a set
+of canonical name/value pairs for control information such as ids, types
+and links.
+
+This format extends [OData-JSON](#ODataJSON) further by allowing a
+structured instance -- an entity or a complex value -- to be represented as
```
**mikepizzo** (2026-09-09):
specifically, the properties of a structured instance, no?


## T07 — odata-compact-json-format/2 Compact JSON Format Design.md @ line 25 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963569700
```diff
+
+1. [*Compact JSON is a superset of the OData JSON format.*]{id=supersetprinciple}
+   Every payload that is valid according
+   to [OData-JSON](#ODataJSON) is also a valid compact JSON payload. A
+   receiver that accepts compact JSON therefore accepts strictly more
+   payloads than a receiver that accepts only [OData-JSON](#ODataJSON),
```
**mikepizzo** (2026-09-09):
the point is not that it accepts more, but that it accepts any valid OData JSON Payload.
```suggestion
   to [OData-JSON](#ODataJSON) is also a valid compact JSON payload. A
   receiver that accepts compact JSON therefore also accepts [OData-JSON](#ODataJSON),
```
although this may be redundant with the statement that "Compact JSON is a superset of the OData JSON Format."


## T08 — odata-compact-json-format/2 Compact JSON Format Design.md @ line 29 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963575127
```diff
+   receiver that accepts compact JSON therefore accepts strictly more
+   payloads than a receiver that accepts only [OData-JSON](#ODataJSON),
+   and a sender is never forced to use a positional representation where
+   it is inconvenient or impossible.
+
+2. *The saving is in the repetition.* A positional representation removes
```
**mikepizzo** (2026-09-09):
nit: isn't the savings in *avoiding* the repetition?


## T09 — odata-compact-json-format/2 Compact JSON Format Design.md @ line 30 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963578528
```diff
+   payloads than a receiver that accepts only [OData-JSON](#ODataJSON),
+   and a sender is never forced to use a positional representation where
+   it is inconvenient or impossible.
+
+2. *The saving is in the repetition.* A positional representation removes
+   property names from a payload; the mapping from positions to properties
```
**mikepizzo** (2026-09-09):
the property names are still in the context url...
```suggestion
   redundant property names from a payload; the mapping from positions to properties
```


## T10 — odata-compact-json-format/2 Compact JSON Format Design.md @ line 34 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963622681
```diff
+2. *The saving is in the repetition.* A positional representation removes
+   property names from a payload; the mapping from positions to properties
+   is transmitted once. Consequently the benefit grows with the number of
+   instances that share a positional property list, and is negligible or
+   negative for a payload containing a single instance. This format
+   therefore never *requires* a positional representation.
```
**mikepizzo** (2026-09-09):
I don't know that the optionality of the positional representation follows directly from the fact that benefit is negligible or negative for a single instance. I think there are a number of benefits to the design principle that you compact JSON is a superset of JSON, and this is just one of them.


```suggestion
   negative for a payload containing a single instance. As the Compact JSON format
   never *requires* a positional representation, producers can fall back to the standard
   representation where convenient or more concise.
```


## T11 — odata-compact-json-format/2 Compact JSON Format Design.md @ line 50 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963630442
```diff
+   requests and responses must differ, the difference is called out
+   explicitly.
+
+::: example
+Example ##ex_superset: because of the [superset
+principle](#supersetprinciple), a sender that cannot produce a positional
```
**mikepizzo** (2026-09-09):
throughout -- I think we generally use "producer" rather than "sender".


## T12 — odata-compact-json-format/2 Compact JSON Format Design.md @ line 50 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963642376
```diff
+   requests and responses must differ, the difference is called out
+   explicitly.
+
+::: example
+Example ##ex_superset: because of the [superset
+principle](#supersetprinciple), a sender that cannot produce a positional
```
**mikepizzo** (2026-09-09):
cannot, or choses not to


## T13 — odata-compact-json-format/2 Compact JSON Format Design.md @ line 53 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963639385
```diff
+::: example
+Example ##ex_superset: because of the [superset
+principle](#supersetprinciple), a sender that cannot produce a positional
+representation for a particular instance may fall back to the
+representation defined by [OData-JSON](#ODataJSON) for that instance
+alone, within an otherwise positional payload:
```
**mikepizzo** (2026-09-09):
I would strike "alone" -- they can do it for an arbitrary number within the payload.
```suggestion
 within an otherwise positional payload:
```


## T14 — odata-compact-json-format/2 Compact JSON Format Design.md @ line 75 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963648090
```diff
+specification: for the representation of primitive values, for the
+definition and meaning of control information, for URL and relative URL
+handling, and for everything else it does not mention, the rules of
+[OData-JSON](#ODataJSON) apply unchanged.
+
+In particular, this format changes only *how a structured instance is
```
**mikepizzo** (2026-09-09):
is it valid to say that it changes only "how the positional properties of a structured instance are laid out"?

**mikepizzo** (2026-09-09):
shouldn't use "positional properties" here -- should say something like "known properties" that anyone familiar with the json specification understands.


## T15 — odata-compact-json-format/3 Requesting the Compact JSON Format.md @ line 13 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963684803
```diff
+`compact=true` format parameter, optionally followed by other format
+parameters.
+
+Alternatively, this format can be requested using the `Accept` header with
+the media type `application/json` followed by the `compact=true` format
+parameter, optionally followed by other format parameters.
```
**mikepizzo** (2026-09-09):
1. I would reverse the order of these. Content-negotiation is the standard HTTP way to request a particular format, so we should describe that first, and then the OData-specific query option mechanism.
2. "followed by the compact=true format parameter, optionally followed by other format parameters" implies an ordering -- that compact=true must be the first format parameter. I don't believe the format parameters are (or should be) ordered.

```suggestion
The compact JSON format can be requested using the `Accept` header with
the media type `application/json` with the `compact=true` format
parameter.

Alternatively, this format can be requested using the `$format` query option
in the request URL with the media type `application/json` with the
`compact=true` format parameter.

```


## T16 — odata-compact-json-format/3 Requesting the Compact JSON Format.md @ line 18 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963735453
```diff
+parameter, optionally followed by other format parameters.
+
+If specified, `$format` overrides any value specified in the `Accept`
+header.
+
+The names and values of the format parameters are case-insensitive.
```
**mikepizzo** (2026-09-09):
Maybe move this to before "If specified..." since it applies to both?  Also, this is already defined in the MIME spec.


## T17 — odata-compact-json-format/3 Requesting the Compact JSON Format.md @ line 25 (current) — mikepizzo 2026-08-26
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3864305239
```diff
+A service that does not support the compact JSON format MUST NOT return a
+compact payload. Because the `compact` format parameter selects a payload
+shape that a receiver unaware of this specification will misinterpret, a
+service MUST NOT return a compact response unless the client requested it
+with `compact=true`, and MUST return `406 Not Acceptable` if the client
+requests `compact=true` and the service does not support it.
```
**mikepizzo** (2026-08-26):
If regular JSON is a subset of compact JSON, why must the service return not acceptable if the client requests compact=true and the service doesn't support it.

**mikepizzo** (2026-08-26):
Note: service should not claim to support compact json if all responses are standard json, but a service should always be allowed to return regular json if the client requests compact=true.

**Hubert-Heijkers** (2026-08-28):
It would only be if the client's Accept list ONLY allows for `application/json; compact=true` and NOT `application/json`. This to me is standard content-type negotiation saying only if `application/json; compact=true` is in the Accept list you are allowed to choose it and if it's not in there and it's the only acceptable content type then you MUST return `406 Not Acceptable` (which is the correct error according to the content negotiation specification).

**HeikoTheissen** (2026-09-10):
Doesn't RFC2045 (quoted by Mike below) apply here, which states:
> MIME implementations must ignore any parameters whose names they do not recognize.

Wouldn't the service simply ignore the `compact=true`? To prevent clients from falling into this trap, we should specify
> Clients are advised that services unaware of compact JSON will treat `Accept: application/json;compact=true` like `Accept: application/json` (because RFC2045 instructs them to ignore the unknown `compact=true`). Therefore, a client SHOULD NOT send requests with `Accept: application/json;compact=true` unless it has established â€¦ [continues like below]


## T18 — odata-compact-json-format/3 Requesting the Compact JSON Format.md @ line 25 (current) — ralfhandl 2026-08-31
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3892823615
```diff
+A service that does not support the compact JSON format MUST NOT return a
+compact payload. Because the `compact` format parameter selects a payload
+shape that a receiver unaware of this specification will misinterpret, a
+service MUST NOT return a compact response unless the client requested it
+with `compact=true`, and MUST return `406 Not Acceptable` if the client
+requests `compact=true` and the service does not support it.
```
**ralfhandl** (2026-08-31):
```suggestion
only accepts `compact=true` and the service does not support it.
```


## T19 — odata-compact-json-format/3 Requesting the Compact JSON Format.md @ line 25 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963764597
```diff
+A service that does not support the compact JSON format MUST NOT return a
+compact payload. Because the `compact` format parameter selects a payload
+shape that a receiver unaware of this specification will misinterpret, a
+service MUST NOT return a compact response unless the client requested it
+with `compact=true`, and MUST return `406 Not Acceptable` if the client
+requests `compact=true` and the service does not support it.
```
**mikepizzo** (2026-09-09):
I'm not sure what "A service that does not support a compact JSON format MUST NOT return a compact payload" means -- isn't this self-evident?

Also, since compact JSON is a superset of JSON, I think it should be legal for a client to request compact json and the service to return standard JSON. If we want to know if the service supports compact json, we should instead use the appropriate capability to discover which formats are supported by the service.

Also, content-negotiation is a bit more complicated than saying that a service must return not acceptable if the client requests a format that it doesn't support -- since the client can specify multiple acceptable formats, the service only returns not acceptable if it doesn't support *any* of the requested formats.

```suggestion
Because the `compact=true` format parameter selects a payload
shape that a receiver unaware of this specification will misinterpret, a
service MUST NOT return a compact response unless the client requested it
with `compact=true`.
```


## T20 — odata-compact-json-format/3 Requesting the Compact JSON Format.md @ line 29 (current) — mikepizzo 2026-08-26
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3864310515
```diff
+with `compact=true`, and MUST return `406 Not Acceptable` if the client
+requests `compact=true` and the service does not support it.
+
+A client MUST NOT send a compact request body unless it has established
+that the service supports the compact JSON format, either from the
+[`Capabilities.SupportedFormats`](#AdvertisingSupport) annotation or from
```
**mikepizzo** (2026-08-26):
how is support for compact json specified in supported formats?  is it just application/json;compact=true?

**Hubert-Heijkers** (2026-08-28):
That's what I'd expect indeed. This is what we have today already:

```
"@Capabilities.SupportedFormats": [
Â  Â  Â  Â  Â  Â  Â  Â  "application/json",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=full",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=minimal",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=none",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;IEEE754Compatible=false",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;IEEE754Compatible=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.streaming=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;IEEE754Compatible=false;odata.streaming=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;IEEE754Compatible=true;odata.streaming=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=full;IEEE754Compatible=false",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=minimal;IEEE754Compatible=false",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=none;IEEE754Compatible=false",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=full;IEEE754Compatible=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=minimal;IEEE754Compatible=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=none;IEEE754Compatible=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=full;odata.streaming=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=minimal;odata.streaming=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=none;odata.streaming=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=full;odata.streaming=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=minimal;odata.streaming=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=none;odata.streaming=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=full;IEEE754Compatible=false;odata.streaming=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=minimal;IEEE754Compatible=false;odata.streaming=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=none;IEEE754Compatible=false;odata.streaming=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=full;IEEE754Compatible=true;odata.streaming=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=minimal;IEEE754Compatible=true;odata.streaming=true",
Â  Â  Â  Â  Â  Â  Â  Â  "application/json;odata.metadata=none;IEEE754Compatible=true;odata.streaming=true"
Â  Â  Â  Â  Â  Â  ]
```

Isn't graph representing the combination of parameters supported that way as well @mikepizzo?


## T21 — odata-compact-json-format/3 Requesting the Compact JSON Format.md @ line 30 (current) — mikepizzo 2026-08-26
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3864326130
```diff
+requests `compact=true` and the service does not support it.
+
+A client MUST NOT send a compact request body unless it has established
+that the service supports the compact JSON format, either from the
+[`Capabilities.SupportedFormats`](#AdvertisingSupport) annotation or from
+out-of-band knowledge. A service that receives a request body labeled with
```
**mikepizzo** (2026-08-26):
An existing service won't necessarily know to return this.  We should just say that "A service that does not support compact=true returns `415 Unsupported Media Type` " and drop the MUST.

**Hubert-Heijkers** (2026-08-28):
Think you are mixing something up here @mikepizzo, `415 Unsupported Media Type` would be returned by the server if the server DOES NOT support compact JSON and a client sent it a request with a request body containing compact JSON and as such a content type header containing `application/json; compact=true`.


## T22 — odata-compact-json-format/3 Requesting the Compact JSON Format.md @ line 32 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963811559
```diff
+A client MUST NOT send a compact request body unless it has established
+that the service supports the compact JSON format, either from the
+[`Capabilities.SupportedFormats`](#AdvertisingSupport) annotation or from
+out-of-band knowledge. A service that receives a request body labeled with
+`compact=true` that it does not support MUST reject the request with
+`415 Unsupported Media Type`.
```
**mikepizzo** (2026-09-09):
I'm not sure we can say that a service that requests a request body with `compact=true` must reject with unsupported media type; doing so adds this as a requirement to existing services and [rfc2045](https://datatracker.ietf.org/doc/html/rfc2045#page-5) currently says: " MIME implementations must ignore any parameters whose names they do not recognize."

We could say that the behavior of a service that receives a request body labeled with `compact=true` that it does not support is undefined, and/or that services SHOULD (MAY?) reject the request with `415 Unsupported Media Type`.

**HeikoTheissen** (2026-09-10):
> doing so adds this as a requirement to existing services

Perhaps Hubert was referring to services that do advertise support for compact JSON in general but not support a particular compact JSON payload?

But why not just let the service respond like a compact-JSON-unaware service would: with 400 Bad Request?


## T23 — odata-compact-json-format/3 Requesting the Compact JSON Format.md @ line 41 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963833473
```diff
+The format parameters defined in [OData-JSON](#ODataJSON) apply to the
+compact JSON format with the meaning defined there, subject to the
+following.
+
+The `streaming` parameter is meaningful and MAY be specified. A positional
+representation is inherently ordered, and a compact payload that meets the
```
**mikepizzo** (2026-09-09):
I think the statement that "A positional representation is inherently ordered" is misleading.  Yes, the properties are ordered, but the streaming=true is parameter is more about control information ordering.

Note that OData-defined structures, like a batch request/response objects, are an interesting case -- their order is defined by odata for ordered payloads, but there are no properties in a contextUrl. can such properties be optimized in compact json?  I guess I'll read further to find out...


## T24 — odata-compact-json-format/3 Requesting the Compact JSON Format.md @ line 43 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963820472
```diff
+following.
+
+The `streaming` parameter is meaningful and MAY be specified. A positional
+representation is inherently ordered, and a compact payload that meets the
+[payload ordering constraints](#PayloadOrderingConstraints) MUST include
+`streaming=true`.
```
**mikepizzo** (2026-09-09):
MUST?  if the payload happens to follow ordering constraints but does not specify streaming=true, is it non-compliant? I think this needs to be SHOULD...

```suggestion
[payload ordering constraints](#PayloadOrderingConstraints) SHOULD include
`streaming=true`.
```


## T25 — odata-compact-json-format/3 Requesting the Compact JSON Format.md @ line 58 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963851342
```diff
+to be interpretable at all. See [section ##ControlInformationcontext].
+
+The value `metadata=full` is NOT RECOMMENDED with `compact=true`. Full
+metadata requires control information to be present for every instance,
+which forces every instance into a [wrapper object](#wrapperobject) and
+defeats the purpose of the format. Services MAY reject the combination
```
**mikepizzo** (2026-09-09):
last week we I think we discussed being able to specify control information (and dynamic properties) as name/value pairs withing the wrapper object along with known properties in the array, which would still provide benefit to a compact representation of full metadata.


## T26 — odata-compact-json-format/3 Requesting the Compact JSON Format.md @ line 75 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963878964
```diff
+Example ##ex: a service advertising support for both the format defined in
+[OData-JSON](#ODataJSON) and the compact JSON format
+```json
+"@Capabilities.SupportedFormats": [
+  "application/json;IEEE754Compatible=true",
+  "application/json;IEEE754Compatible=true;compact=true"
```
**mikepizzo** (2026-09-09):
I think we want to discourage IEE754Compatible unless absolutely required. I would use streaming=true instead.

```suggestion
  "application/json;",
  "application/json;streaming=true",
  "application/json;compact=true",
  "application/json;streaming=true;compact=true"
```


## T27 — odata-compact-json-format/4 Compact Representations.md @ line 7 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963894719
```diff
+-------
+
+# ##sec Compact Representations
+
+This section defines the two representations that this format adds to
+[OData-JSON](#ODataJSON): the *positional representation*, which conveys a
```
**mikepizzo** (2026-09-09):
```suggestion
[OData-JSON](#ODataJSON): the *positional representation*, which conveys the properties of a
```


## T28 — odata-compact-json-format/4 Compact Representations.md @ line 17 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963916078
```diff
+
+## ##subsec Positional Representation
+
+A structured instance -- an entity or a complex value -- MAY be represented
+as a JSON array instead of a JSON object. Such an array is called the
+*positional representation* of the instance.
```
**mikepizzo** (2026-09-09):
```suggestion
Properties of an entity or a complex value MAY be represented
```
```suggestion
Structural and navigation properties of an entity or a complex value MAY be represented
as a JSON array instead of name-value pairs within a JSON object. Such an array is called the
*positional representation* of the properties.
```


## T29 — odata-compact-json-format/4 Compact Representations.md @ line 19 (current) — HeikoTheissen 2026-09-11
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3987614113
```diff
+
+A structured instance -- an entity or a complex value -- MAY be represented
+as a JSON array instead of a JSON object. Such an array is called the
+*positional representation* of the instance.
+
+The items of the array are the values of the instance's properties. A
```
**HeikoTheissen** (2026-09-11):
But the items of an array can also be items of a collection. In order to distinguish these two, parsers must know the cardinalities of properties from the $metadata, see https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3979658023

Insert another example after example 4 that illustrates this.


## T30 — odata-compact-json-format/4 Compact Representations.md @ line 60 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963932503
```diff
+- the number of items in the array MUST equal the number of items in the
+  positional property list, and
+- item *n* of the array MUST be the value of item *n* of the positional
+  property list, formatted as described in [section ##PositionValues].
+
+A sender MUST NOT omit an item, MUST NOT add an item, and MUST NOT
```
**mikepizzo** (2026-09-09):
```suggestion
A producer MUST NOT omit an item, MUST NOT add an item, and MUST NOT
```


## T31 — odata-compact-json-format/4 Compact Representations.md @ line 72 (current) — HeikoTheissen 2026-09-11
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3988311597
```diff
+sender may omit a property whose value it does not wish to transmit. In a
+positional representation there is no way to omit a value without
+shifting every subsequent value, so the positional property list must be
+transmitted in full. If a sender wishes to transmit fewer properties, it
+narrows the positional property list -- by narrowing the select-list in the
+context URL -- rather than shortening the array.
```
**HeikoTheissen** (2026-09-11):
```suggestion
context URL.
```
Because the array is also shortened if the select-list is narrowed.


## T32 — odata-compact-json-format/4 Compact Representations.md @ line 82 (current) — HeikoTheissen 2026-09-10
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3978683355
```diff
+select-list, as defined in [#OData-Protocol#ContextURL], that applies to
+that instance.
+
+The context URL of a compact payload MUST contain a select-list, and that
+select-list MUST enumerate every property conveyed positionally, at every
+level of nesting. In particular:
```
**HeikoTheissen** (2026-09-10):
Can nested context URLs also define a positional property list? For example, if not every customer has addresses:
```json
{"@context": "$metadata#Customers(ID,Name)",
 "$": [
  {"$": ["ALFKI", "Alfreds Futterkiste"],
   "Addresses@context": "#Addresses(Street,City)",
   "Addresses": [
    ["FriedrichstraÃŸe", "Berlin"],
    ["BahnhofstraÃŸe", "Walldorf"]
   ]
  }, ...
 ]}
```


## T33 — odata-compact-json-format/4 Compact Representations.md @ line 86 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963941538
```diff
+select-list MUST enumerate every property conveyed positionally, at every
+level of nesting. In particular:
+
+- the select-list MUST NOT be omitted, and MUST NOT be empty;
+- the select-list MUST NOT contain the shortcut `*`, nor the shortcut
+  `{namespace}.*` for the bound operations of a type;
```
**mikepizzo** (2026-09-09):
why can it not contain `{namespace}.*`, along with the properties?


## T34 — odata-compact-json-format/4 Compact Representations.md @ line 87 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963945886
```diff
+level of nesting. In particular:
+
+- the select-list MUST NOT be omitted, and MUST NOT be empty;
+- the select-list MUST NOT contain the shortcut `*`, nor the shortcut
+  `{namespace}.*` for the bound operations of a type;
+- a select-item for a structured property whose value is conveyed
```
**mikepizzo** (2026-09-09):
should we clarify "structured navigation or structural property"?


## T35 — odata-compact-json-format/4 Compact Representations.md @ line 92 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963954845
```diff
+- a select-item for a structured property whose value is conveyed
+  positionally MUST carry a nested select-list, rather than the empty
+  parentheses that [OData-Protocol](#ODataProtocol) permits;
+- the rule of [OData-Protocol](#ODataProtocol) whereby a select-list
+  containing only expanded navigation properties implicitly selects all
+  structural properties does not apply to a compact payload.
```
**mikepizzo** (2026-09-09):
rather than say what doesn't apply, can we turn this around and say what must exist?  i.e., something like "For Compact JSON payloads, the select-list MUST always include all selected structural as well as expanded navigation properties."


## T36 — odata-compact-json-format/4 Compact Representations.md @ line 118 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3963980660
```diff
+to the instance. The positional property list is determined as follows:
+
+1. Each item of *S* that begins with a type-cast segment --- a qualified
+   type name followed by a forward slash --- is removed from *S* unless *T*
+   is that type or is derived from it. From each such item that remains,
+   the leading type-cast segment is removed.
```
**mikepizzo** (2026-09-09):
I'm not sure that we need to say that the leading type-cast segment is removed. This is a logical algorithm; what difference does it make if the type cast segment is conceptually there or not?


## T37 — odata-compact-json-format/4 Compact Representations.md @ line 130 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3964011121
```diff
+   order.
+
+Step 1 is what makes the positional property list depend on the instance
+and not only on the context URL: two instances of different types in one
+collection have different positional property lists. See [section
+##DerivedTypes].
```
**mikepizzo** (2026-09-09):
do we need this conceptual algorithm, or can we just say the following:

```suggestion
The positional property list for any instance is the ordered list of all properties
in the context URL that apply to the type of the instance. Thus, two instances of
different types in one collection have different positional property lists. See [section
##DerivedTypes].
```


## T38 — odata-compact-json-format/4 Compact Representations.md @ line 216 (current) — HeikoTheissen 2026-09-10
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3978569826
```diff
+
+Two items of *S* belong to the same group if and only if their first
+segments are equal. The group occupies the position of the first of its
+items.
+
+If the property addressed by the first segment of a group is of a
```
**HeikoTheissen** (2026-09-10):
```suggestion
If the property addressed by the first segment `P` of a group is of a
```
You use the variable `P` below, I assume this is the first segment.


## T39 — odata-compact-json-format/4 Compact Representations.md @ line 230 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3964172139
```diff
+The nested select-list MUST NOT be empty, for the reason given in [section
+##DeterminingthePositionalPropertyList]: there is no well-defined default
+to fall back on.
+
+The nested select-list determines the positional property list of the
+instances of that property, applying this section recursively.
```
**mikepizzo** (2026-09-09):
so, this is really saying that all nested structural and navigation properties belonging to the same structured object are grouped together and positioned at the first occurrence of such a property, right?  I got a little lost in the algorithm.

I dislike being quite so tied to the exact syntax of the contextUrl, and would rather describe the intent than an algorithm for string parsing.

```suggestion
A select-list may contain multiple items that address different structural and
expanded properties of the same complex-typed property. The positional
representation groups all nested structural and navigation properties
belonging to the same complex object together as a nested select-list,
positioned at the first occurrence of a such a property in the select-list.
```


## T40 — odata-compact-json-format/4 Compact Representations.md @ line 243 (current) — HeikoTheissen 2026-09-10
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3979658023
```diff
+```
+```json
+{
+  "@context": "$metadata#Customers(Name,Address/City,Address/PostalCode)",
+  "$": [
+    ["Alfreds Futterkiste", ["Berlin", "12209"]],
```
**HeikoTheissen** (2026-09-10):
If `Address` was collection-valued, then the context URL for
```
GET ~/Customers?$select=Name,Address($select=City,PostalCode)
```
would be
```
$metadata#Customers(Name,Address(City,PostalCode))
```
and the representation would be
```json
["Alfreds Futterkiste", [["Berlin", "12209"]]]
```
Correct?

A consumer of compact JSON payloads can parse this only with knowledge about the cardinality of the `Address` property, which it obtains from the $metadata. This reliance on $metadata should be mentioned in the text somewhere, because it wasn't the case for standard JSON format.


## T41 — odata-compact-json-format/4 Compact Representations.md @ line 251 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3969492572
```diff
+}
+```
+The positional property list of each `Customer` is (`Name`, `Address`),
+and the positional property list of each `Address` is (`City`,
+`PostalCode`).
+:::
```
**mikepizzo** (2026-09-09):
should we add a property between Address/City and Address/PostalCode to show the rearrangement, i.e.:


```suggestion
GET ~/Customers?$select=ID,Address/City,Name,Address/PostalCode
```
```json
{
  "@context": "$metadata#Customers(ID,Address/City,Name,Address/PostalCode)",
  "$": [
    ["ALFKI", ["Berlin", "12209"],"Alfreds Futterkiste"],
    ["ANATR", ["MÃ©xico D.F.", "05021"],Ana Trujillo"]
  ]
}
```
The positional property list of each `Customer` is (`ID, `Address`,`Name`),
and the positional property list of each `Address` is (`City`,
`PostalCode`).
:::
```


## T42 — odata-compact-json-format/4 Compact Representations.md @ line 255 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3964186785
```diff
+`PostalCode`).
+:::
+
+::: example
+Example ##ex: a bare item and a suffixed item addressing the same
+navigation property form one group and therefore one position
```
**mikepizzo** (2026-09-09):
I don't understand the "bare" and "suffixed" items referred to in this example.

**mikepizzo** (2026-09-09):
Okay; I get it -- we're talking about DirectReports and recursive DirectReports with an explicit select list?  In this case, doesn't the first DirectReports just refer to including the navigation link? Should we include that the DirectReports have a nested select list with FirstName and LastName? Right now the example makes it appear that those nested properties are lost.


## T43 — odata-compact-json-format/4 Compact Representations.md @ line 264 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3969500317
```diff
+The positional property list of each `Manager` is (`DirectReports`).
+:::
+
+## ##subsec The Wrapper Object
+
+A positional representation is a JSON array, and a JSON array has no
```
**mikepizzo** (2026-09-09):
```suggestion
A positional property representation is a JSON array, and a JSON array has no
```


## T44 — odata-compact-json-format/4 Compact Representations.md @ line 273 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3969531894
```diff
+instance, or to the object containing a property; and any property that
+cannot be placed in the positional property list at all. The containing
+instance is no help, since it may itself be an array.
+
+This format introduces a single construct for all of them, used uniformly
+wherever a value would otherwise appear.
```
**mikepizzo** (2026-09-09):
I'm not sure this is needed. We don't normally include this must justification/explanation -- we just explain the intended design/behavior.  In this case, I think it's pretty straightforward why we chose what we chose.


```suggestion
```


## T45 — odata-compact-json-format/4 Compact Representations.md @ line 278 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3964762844
```diff
+wherever a value would otherwise appear.
+
+A *wrapper object* is a JSON object whose name/value pairs are
+
+- annotations and control information that apply to a value,
+- optionally, that value itself, under the reserved name `$`, and
```
**mikepizzo** (2026-09-09):
```suggestion
- optionally, that positional values of the instance, under the reserved name `$`, and
```


## T46 — odata-compact-json-format/4 Compact Representations.md @ line 301 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3964803603
```diff
+positional representation of a structured instance, a JSON object as
+defined in [OData-JSON](#ODataJSON), a JSON array for a collection, or a
+JSON primitive where the wrapper stands at the position of a primitive
+property. It is not restricted to arrays --- a wrapper carrying an
+annotation on a primitive property holds that primitive under `$`.
+
```
**mikepizzo** (2026-09-09):
```suggestion
An object with a `$` property is a wrapper object in which all of the
positional properties are represented by the property named `$` and any
other properties, control information, or  annotations are represented as
defined by [OData-JSON](#ODataJSON).
```


## T47 — odata-compact-json-format/4 Compact Representations.md @ line 302 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3969610975
```diff
+defined in [OData-JSON](#ODataJSON), a JSON array for a collection, or a
+JSON primitive where the wrapper stands at the position of a primitive
+property. It is not restricted to arrays --- a wrapper carrying an
+annotation on a primitive property holds that primitive under `$`.
+
+Properties carried by name in a wrapper object MUST NOT include a property
```
**mikepizzo** (2026-09-09):
would it be clearer to say "identified by name" rather than "carried by name"?


## T48 — odata-compact-json-format/4 Compact Representations.md @ line 303 (current) — HeikoTheissen 2026-09-11
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3988966117
```diff
+JSON primitive where the wrapper stands at the position of a primitive
+property. It is not restricted to arrays --- a wrapper carrying an
+annotation on a primitive property holds that primitive under `$`.
+
+Properties carried by name in a wrapper object MUST NOT include a property
+that occupies a position in the instance's positional property list, and
```
**HeikoTheissen** (2026-09-11):
Does this only apply to the property itself, or also to its annotations/control information?

In other words: Is the following fobidden (assuming select-list `(ID,Name)`)?
```json
{"$": ["ALFKI", "Alfred"],
 "Name@Core.Permission": "Read"}
```
Or is it equivalent to
```json
["ALFKI",
 {"$": "Alfred", "@Core.Permission": "Read"}]
```


## T49 — odata-compact-json-format/4 Compact Representations.md @ line 304 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3969597764
```diff
+property. It is not restricted to arrays --- a wrapper carrying an
+annotation on a primitive property holds that primitive under `$`.
+
+Properties carried by name in a wrapper object MUST NOT include a property
+that occupies a position in the instance's positional property list, and
+MUST follow the value, as required by [section
```
**mikepizzo** (2026-09-09):
why must the property carried by name follow the value?  what value (the $ property?)

**HeikoTheissen** (2026-09-11):
It must follow the `$` member in the wrapper object.


## T50 — odata-compact-json-format/4 Compact Representations.md @ line 310 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3969651424
```diff
+##PayloadOrderingConstraints]. Carrying properties by name is what allows
+an instance of an open type to keep its positional representation while
+conveying dynamic properties that could not be placed in the select-list;
+see [section ##OpenTypesandDynamicProperties].
+
+A wrapper object MAY appear wherever a value may appear:
```
**mikepizzo** (2026-09-09):
where-ever a JSON value may appear, or a positional property value may appear, or?


## T51 — odata-compact-json-format/4 Compact Representations.md @ line 318 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3969720516
```diff
+  [`context`](#ControlInformationcontext) and the payload's content;
+- at a position in a positional representation, in which case it carries
+  the annotations of the property at that position and the property's
+  value;
+- as an item of a collection, in which case it carries the annotations of
+  that member of the collection and the member itself.
```
**mikepizzo** (2026-09-09):
not just annotations, but also control information and dynamic properties, right?


```suggestion
- at a position in a positional property representation, in which case it represents the
  the annotations, control information, positional and dynamic properties
  of the property at that position
- as an item of a collection, in which case it represents the annotations, control
information, positional and dynamic properties of that member of the collection.
```

**HeikoTheissen** (2026-09-10):
"Dynamic properties of a property" is not needed. A wrapper object in position N of a positional property representation carries information about the N-th property:
- context (not: control) information for that property, for example, its `@type`
- annotations of that property
- the value of that property (this can be a structured value with dynamic properties, but then we are one nesting level deeper).


## T52 — odata-compact-json-format/4 Compact Representations.md @ line 327 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3969769616
```diff
+object `{}` carries neither annotations nor a value and therefore denotes
+"no value"; it is used at the position of a selected dynamic property that
+an instance does not have, see [section ##OpenTypesandDynamicProperties].
+Note that `{}` satisfies the second bullet above vacuously rather than by
+construction: it is treated as a wrapper carrying no value by convention,
+being the only reading that is useful.
```
**mikepizzo** (2026-09-09):
```suggestion
The empty JSON object `{}` is treated as a wrapper object with no value;
it is used at the position of a selected dynamic property that
has no value for the instance. See [section ##OpenTypesandDynamicProperties].
```


## T53 — odata-compact-json-format/4 Compact Representations.md @ line 329 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3969795037
```diff
+an instance does not have, see [section ##OpenTypesandDynamicProperties].
+Note that `{}` satisfies the second bullet above vacuously rather than by
+construction: it is treated as a wrapper carrying no value by convention,
+being the only reading that is useful.
+
+The name of the value in a wrapper object is `$`, in every position in
```
**mikepizzo** (2026-09-09):
```suggestion
The name of the positional property values in a wrapper object is `$`, in every position in
```


## T54 — odata-compact-json-format/4 Compact Representations.md @ line 348 (current) — ralfhandl 2026-08-31
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3893092564
```diff
+single entity reference -- a name/value pair named `value` in that message
+body is a *property* named `value`, and a receiver MUST NOT read it as the
+value of a wrapper object. A single entity or complex value represented
+positionally at the root of the message body therefore uses `$`.
+
+Producers of compact payloads SHOULD use `$` wherever this document
```
**ralfhandl** (2026-08-31):
What do senders gain by being able to choose `value` instead of `$`?

In which cases would that choice be helpful?

**mikepizzo** (2026-09-09):
savings is in the array response, so would likely use value for singleton and $ for array.


## T55 — odata-compact-json-format/4 Compact Representations.md @ line 355 (current) — HeikoTheissen 2026-09-10
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3980046792
```diff
+
+This restriction is what keeps the two representations distinguishable.
+Were `value` also the wrapper's value name at the root of a message body
+representing a single entity, then
+`{"@context": "â€¦#Customers/$entity", "value": [â€¦]}` would be at once the
+positional representation of an entity and the
```
**HeikoTheissen** (2026-09-10):
This would _not_ be a valid positional representation, because the context URL lacks the select-list that defines the positional property list.


## T56 — odata-compact-json-format/4 Compact Representations.md @ line 440 (current) — mikepizzo 2026-09-09
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3969821116
```diff
+array of arrays. A receiver that knows the positional property list also
+knows, from the metadata, whether a property is collection-valued, and can
+therefore distinguish the two nestings without ambiguity.
+
+A property that has no value at all -- as opposed to a property whose value
+is null -- occurs when only annotations were requested for it, for example
```
**mikepizzo** (2026-09-09):
```suggestion
is null -- occurs when only annotations or control information were requested for it, for example
```


## T57 — odata-compact-json-format/5 Common Characteristics.md @ line 30 (current) — mikepizzo 2026-09-18
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r4049198173
```diff
+## ##subsec Message Body
+
+Each message body is represented as a single JSON object, as defined in
+[OData-JSON](#ODataJSON), with the single exception described in [section
+##MessageBodyofaRequest]. A response message body is always a JSON
+object: the positional representation applies to the instances *within*
```
**mikepizzo** (2026-09-18):
```suggestion
object: the positional representation applies to the property instances *within*
```


## T58 — odata-compact-json-format/5 Common Characteristics.md @ line 43 (current) — mikepizzo 2026-09-18
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r4049240356
```diff
+- a [wrapper object](#wrapperobject), whose value is the correct
+  representation for the payload's content.
+
+The name of the value in a wrapper object is `$`. The name `value` is also
+recognized, but only in those message bodies in which
+[OData-JSON](#ODataJSON) itself uses it; see [section ##TheWrapperObject].
```
**mikepizzo** (2026-09-18):
Alternatively, could we say that the object is *always* a representation of an entity, ref, or complex type, and that the positional properties may either be represented as individual name/value pairs or as a single positional  wrapper obect named `$`?



```suggestion
This properties of an entity, entity reference, or complex type are represented as either

- individual name/value pairs
- 

```
```suggestion
An entity, entity reference, or complex type is represented as either

- the standard OData JSON representation of an [entity](#Entity), an [entity
  reference](#EntityReferences) or a [complex value](#ComplexValue) with 
  properties represented as individual name/value pairs
- the standard OData JSON representation of an [entity](#Entity), an [entity
  reference](#EntityReferences) or a [complex value](#ComplexValue) with 
  all positional properties represented a single array-valued property whose
  name is `$`
- an array of positional property values
```


## T59 — odata-compact-json-format/5 Common Characteristics.md @ line 47 (current) — mikepizzo 2026-09-18
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r4049929397
```diff
+recognized, but only in those message bodies in which
+[OData-JSON](#ODataJSON) itself uses it; see [section ##TheWrapperObject].
+
+::: example
+Example ##ex: a message body containing a collection of entities
+represented positionally
```
**mikepizzo** (2026-09-18):
```suggestion
represented as arrays of positional property values.
```


## T60 — odata-compact-json-format/5 Positional Representation.md @ line 116 (OUTDATED) — HeikoTheissen 2026-08-22
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3835508701
```diff
+GET ~/Customers('ALFKI')?$select=Name,ID
+{
+  "@context": "$metadata#Customers(Name,ID)/$entity",
+  "_": ["Alfreds Futterkiste", "ALFKI"]
+}
+```
```
**HeikoTheissen** (2026-08-22):
Suggest to write such examples as two code blocks:
````suggestion
```
GET ~/Customers('ALFKI')?$select=Name,ID
```
```json
{
  "@context": "$metadata#Customers(Name,ID)/$entity",
  "_": ["Alfreds Futterkiste", "ALFKI"]
}
```
````

**Hubert-Heijkers** (2026-08-24):
I'll tell Claude to get it's act together on this one;-!

I'm reviewing the whole doc myself again as well, got a number of little changes/open points as a follow up (some of which I might address myself before Wednesday).


## T61 — odata-compact-json-format/6 Annotations and Control Information.md @ line 52 (OUTDATED) — HeikoTheissen 2026-08-24
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3843721134
```diff
+[section ##DerivedTypes].
+
+The name of the value in a wrapper object is `_`.
+
+At the root of the message body the name `value` is also recognized, with
+the same meaning, for compatibility with [OData-JSON](#ODataJSON) as
```
**HeikoTheissen** (2026-08-24):
But [OData-JSON] allows the `value` name only for a collection of entities,
```json
{"@context": "...",
 "value": {...}}
```
is not a valid representation of a single entity.

**Hubert-Heijkers** (2026-08-26):
Correct @HeikoTheissen, there is an inherent conflict here, updated the sections around this for further clarification.


## T62 — odata-compact-json-format/6 Structured Instances.md @ line 180 (OUTDATED) — mikepizzo 2026-08-26
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3864627939
```diff
+```
+GET ~/Customers?$select=ID,Model.VipCustomer/PreferredContact
+{
+  "@context": "$metadata#Customers(ID,Model.VipCustomer/PreferredContact)",
+  "_": [
+    ["ALFKI", {}],
```
**mikepizzo** (2026-08-26):
alternatively, we could identify the set of relevant properties per row based on the type.  That is, since this is the base type I know that only "ALFKI" applies, so I would only look for one property.

**Hubert-Heijkers** (2026-08-28):
Worked this into the pending updated proposal as well!


## T63 — odata-compact-json-format/6 Structured Instances.md @ line 213 (OUTDATED) — mikepizzo 2026-08-26
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3864644360
```diff
+```json
+{
+  "@context": "$metadata#Customers(ID,Name)",
+  "_": [
+    ["ALFKI", "Alfreds Futterkiste"],
+    { "ID": "ANATR", "Name": "Ana Trujillo", "Nickname": "Ana" }
```
**mikepizzo** (2026-08-26):
alternatively, could I do:
    { "_": ["ANATR", "Ana Trujillo"], "Nickname": "Ana" }

**Hubert-Heijkers** (2026-08-28):
Had a good heart-to-heart with Claude/Fable and after looking at the definition of the wrapper a bit more, discussion what property name to use for the positional representation after agreeing that `_` wasn't a good choice (discussed "", "@" and "$" mainly), adding this alternative to the mix settled it, `$` it is (what we propose) ;-)


## T64 — odata-compact-json-format/7 Annotations and Control Information.md @ line 22 (OUTDATED) — mikepizzo 2026-08-26
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3864668672
```diff
+Neither pattern is available in a positional representation: an array has
+no name/value pairs, and the containing instance may itself be an array.
+This format therefore introduces a single construct, the *wrapper
+object*, which is used uniformly wherever a value would otherwise appear.
+
+## ##subsec The Wrapper Object
```
**mikepizzo** (2026-08-26):
Organizationally, it's a bit confusing to introduce the wrapper object *after* it's been used a number of times to describe things like annotations, control information, and dynamic properties.

**Hubert-Heijkers** (2026-08-28):
Agreed @mikepizzo, moved it into section 4 Compact Representation as it is the combination of the positional representation and these wrapper objects that allows us to structure these compact payloads.


## T65 — odata-compact-json-format/8 Request Payloads.md @ line 100 (current) — ralfhandl 2026-08-31
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3893296587
```diff
+[OData-Protocol](#ODataProtocol) and is left unchanged. A property that
+is in the positional property list with the value `null` is set to null.
+
+A `PATCH` request body that does not contain the `context` control
+information therefore specifies a value for every structural property of
+the type, which is rarely the intent. Clients SHOULD include the `context`
```
**ralfhandl** (2026-08-31):
Lines 22-23 say

>  compact request body MUST contain the
> [`context`](#ControlInformationcontext) control information

Why relax that here?


## T66 — odata-compact-json-format/8 Request Payloads.md @ line 166 (current) — ralfhandl 2026-08-31
url: https://github.com/oasis-tcs/odata-specs/pull/2270#discussion_r3893318380
```diff
+`Property@bind` or `Property@id` on the containing object.
+
+In a compact request body the navigation property occupies a position in
+the positional property list, and the bind operation is carried in a
+[wrapper object](#wrapperobject) at that position, as a property
+annotation, following [section ##PropertyAnnotations].
```
**ralfhandl** (2026-08-31):
```suggestion
[wrapper object](#wrapperobject) at that position, as control
information, following [section ##PropertyAnnotations].
```
