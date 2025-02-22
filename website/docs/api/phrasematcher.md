---
title: PhraseMatcher
teaser: Match sequences of tokens, based on documents
tag: class
source: spacy/matcher/phrasematcher.pyx
new: 2
---

The `PhraseMatcher` lets you efficiently match large terminology lists. While
the [`Matcher`](/api/matcher) lets you match sequences based on lists of token
descriptions, the `PhraseMatcher` accepts match patterns in the form of `Doc`
objects.

## PhraseMatcher.\_\_init\_\_ {#init tag="method"}

Create the rule-based `PhraseMatcher`. Setting a different `attr` to match on
will change the token attributes that will be compared to determine a match. By
default, the incoming `Doc` is checked for sequences of tokens with the same
`ORTH` value, i.e. the verbatim token text. Matching on the attribute `LOWER`
will result in case-insensitive matching, since only the lowercase token texts
are compared. In theory, it's also possible to match on sequences of the same
part-of-speech tags or dependency labels.

If `validate=True` is set, additional validation is performed when pattern are
added. At the moment, it will check whether a `Doc` has attributes assigned that
aren't necessary to produce the matches (for example, part-of-speech tags if the
`PhraseMatcher` matches on the token text). Since this can often lead to
significantly worse performance when creating the pattern, a `UserWarning` will
be shown.

> #### Example
>
> ```python
> from spacy.matcher import PhraseMatcher
> matcher = PhraseMatcher(nlp.vocab)
> ```

| Name                                    | Type            | Description                                                                                 |
| --------------------------------------- | --------------- | ------------------------------------------------------------------------------------------- |
| `vocab`                                 | `Vocab`         | The vocabulary object, which must be shared with the documents the matcher will operate on. |
| `attr` <Tag variant="new">2.1</Tag>     | int / unicode   | The token attribute to match on. Defaults to `ORTH`, i.e. the verbatim token text.          |
| `validate` <Tag variant="new">2.1</Tag> | bool            | Validate patterns added to the matcher.                                                     |
| **RETURNS**                             | `PhraseMatcher` | The newly constructed object.                                                               |

<Infobox title="Changed in v2.1" variant="warning">

As of v2.1, the `PhraseMatcher` doesn't have a phrase length limit anymore, so
the `max_length` argument is now deprecated.

</Infobox>

## PhraseMatcher.\_\_call\_\_ {#call tag="method"}

Find all token sequences matching the supplied patterns on the `Doc`.

> #### Example
>
> ```python
> from spacy.matcher import PhraseMatcher
>
> matcher = PhraseMatcher(nlp.vocab)
> matcher.add("OBAMA", None, nlp("Barack Obama"))
> doc = nlp("Barack Obama lifts America one last time in emotional farewell")
> matches = matcher(doc)
> ```

| Name        | Type  | Description                                                                                                                                                              |
| ----------- | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `doc`       | `Doc` | The document to match over.                                                                                                                                              |
| **RETURNS** | list  | A list of `(match_id, start, end)` tuples, describing the matches. A match tuple describes a span `doc[start:end]`. The `match_id` is the ID of the added match pattern. |

## PhraseMatcher.pipe {#pipe tag="method"}

Match a stream of documents, yielding them in turn.

> #### Example
>
> ```python
>   from spacy.matcher import PhraseMatcher
>   matcher = PhraseMatcher(nlp.vocab)
>   for doc in matcher.pipe(texts, batch_size=50):
>       pass
> ```

| Name         | Type     | Description                                               |
| ------------ | -------- | --------------------------------------------------------- |
| `docs`       | iterable | A stream of documents.                                    |
| `batch_size` | int      | The number of documents to accumulate into a working set. |
| **YIELDS**   | `Doc`    | Documents, in order.                                      |

## PhraseMatcher.\_\_len\_\_ {#len tag="method"}

Get the number of rules added to the matcher. Note that this only returns the
number of rules (identical with the number of IDs), not the number of individual
patterns.

> #### Example
>
> ```python
>   matcher = PhraseMatcher(nlp.vocab)
>   assert len(matcher) == 0
>   matcher.add("OBAMA", None, nlp("Barack Obama"))
>   assert len(matcher) == 1
> ```

| Name        | Type | Description          |
| ----------- | ---- | -------------------- |
| **RETURNS** | int  | The number of rules. |

## PhraseMatcher.\_\_contains\_\_ {#contains tag="method"}

Check whether the matcher contains rules for a match ID.

> #### Example
>
> ```python
>   matcher = PhraseMatcher(nlp.vocab)
>   assert "OBAMA" not in matcher
>   matcher.add("OBAMA", None, nlp("Barack Obama"))
>   assert "OBAMA" in matcher
> ```

| Name        | Type    | Description                                           |
| ----------- | ------- | ----------------------------------------------------- |
| `key`       | unicode | The match ID.                                         |
| **RETURNS** | bool    | Whether the matcher contains rules for this match ID. |

## PhraseMatcher.add {#add tag="method"}

Add a rule to the matcher, consisting of an ID key, one or more patterns, and a
callback function to act on the matches. The callback function will receive the
arguments `matcher`, `doc`, `i` and `matches`. If a pattern already exists for
the given ID, the patterns will be extended. An `on_match` callback will be
overwritten.

> #### Example
>
> ```python
>   def on_match(matcher, doc, id, matches):
>       print('Matched!', matches)
>
>   matcher = PhraseMatcher(nlp.vocab)
>   matcher.add("OBAMA", on_match, nlp("Barack Obama"))
>   matcher.add("HEALTH", on_match, nlp("health care reform"),
>                                   nlp("healthcare reform"))
>   doc = nlp("Barack Obama urges Congress to find courage to defend his healthcare reforms")
>   matches = matcher(doc)
> ```

| Name       | Type               | Description                                                                                   |
| ---------- | ------------------ | --------------------------------------------------------------------------------------------- |
| `match_id` | unicode            | An ID for the thing you're matching.                                                          |
| `on_match` | callable or `None` | Callback function to act on matches. Takes the arguments `matcher`, `doc`, `i` and `matches`. |
| `*docs`    | list               | `Doc` objects of the phrases to match.                                                        |
