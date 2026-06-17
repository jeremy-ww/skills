---
name: data-populator
description: Converts user-provided English words, sentences, or phrases into structured JSON matching the vocabulary data schema (word format with phonetics and examples, or sentence format with translation). Use when the user pastes a word, sentence, or batch of items to be added to the vocab database. Triggers on "populate this", "add to my vocab", "convert to JSON", "format these words", or "make this into the schema".
---

# Data Populator

Converts user-supplied English input (a single word, a sentence, or a batch of either) into JSON that conforms exactly to the vocabulary data schema. Output is code — no commentary, no explanations, no markdown wrappers.

## Overview

The schema has two shapes. Pick the right one based on whether the input is a word or a sentence. Apply the conversion rules deterministically; do not invent fields the schema does not declare.

## When to Use

- The user provides a single English word, a list of English words, or a sentence/phrase.
- The output destination is the vocabulary database.
- The user invokes "populate", "convert", "add to my list", or "format as JSON".

**Not for:**

- Translation tasks where the goal is a readable line, not data.
- Source languages other than English (the schema targets English vocabulary).
- Reshaping the schema itself.

## Output Formats

### 1. Word Format

```json
{
  "id": "<numeric_id>",
  "word": "<Word>",
  "phonetic": "/<American IPA>/",
  "translation": "<Chinese translation>",
  "examples": [
    {
      "text": "<Example sentence 1>",
      "translation": "<Translation 1>"
    },
    {
      "text": "<Example sentence 2>",
      "translation": "<Translation 2>"
    }
  ]
}
```

### 2. Sentence Format

```json
{
  "id": "<numeric_id>",
  "text": "<Sentence>",
  "translation": "<Chinese translation>"
}
```

## Routing Rules

Apply these in order:

| Input shape | Output |
|---|---|
| One word | Single object, **word format** |
| Multiple words | Array, **word format** for each |
| One sentence or phrase | Single object, **sentence format** |
| Multiple sentences | Array, **sentence format** for each |

## Process

1. **Classify the input.** Decide word vs sentence for every item.
2. **For words:**
   1. Capitalize the first letter for the `word` field.
   2. Generate the American IPA transcription for `phonetic` (slashes included).
   3. Write a concise Chinese translation for `translation`.
   4. Produce **exactly two** examples that are **relatively advanced** — beyond beginner textbook phrasing, idiomatic or context-rich.
   5. Translate both examples into Chinese.
3. **For sentences:**
   1. Keep the original casing and punctuation for `text`.
   2. Write a natural Chinese translation.
4. **Assign `id`.** Use a stable, unique identifier. Match the project's existing id convention when one is implied.
5. **Emit JSON only.** No prose, no markdown fences, no commentary around the block.

## Rules

- Use **American IPA** for the `phonetic` field.
- The `word` field must be **capitalized** (first letter uppercase).
- Provide **exactly two** examples per word — not one, not three.
- Examples should be **relatively advanced**, not "I eat an apple" tier.
- Single items return a single object; multiple items return an array.
- Never add fields the schema does not declare.
- Never wrap output in markdown code fences (the consumer parses raw JSON).

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll add a third example for richness" | The schema says exactly two. Extra fields break the consumer. |
| "British IPA is fine" | The schema specifies American. British IPA fails downstream TTS and phonetics checks. |
| "Lowercasing is more natural" | Capitalization is part of the schema contract. Lowercase breaks the import. |
| "I'll wrap the JSON in a code block to be helpful" | The consumer parses raw JSON. Code fences break the parse. |
| "One example is enough" | Exactly two. Consistency is the point. |

## Red Flags

- Output wrapped in ```json fences.
- `word` is lowercase.
- `phonetic` uses British IPA or omits slashes.
- A word has 0, 1, or 3+ examples.
- Extra fields appear (`tags`, `notes`, `level`, etc.).
- Single input is emitted as an array (or vice versa).
- The output contains English commentary around the JSON.

## Verification

Before returning a response:

- [ ] JSON parses with no errors.
- [ ] Output is not wrapped in markdown fences.
- [ ] Each word has exactly two advanced examples with Chinese translations.
- [ ] `phonetic` is in American IPA, slashes included.
- [ ] `word` is capitalized.
- [ ] Shape matches the routing table (word format vs sentence format, single vs array).

Here is the input: 