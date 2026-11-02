<h1 align="center">simhash.mq</h1>

> [!WARNING]
> This project is currently under active development. APIs and features may change without notice.

> [!NOTE]
> This module needs the development version of mq, which is not released yet: it uses the `hash` builtin and `band` on numbers.

Near-duplicate detection for [mq](https://github.com/harehare/mq), as a module. A SimHash fingerprint is 64 bits where similar texts get fingerprints that differ in few bits, so duplicates can be found without comparing every pair of texts.

## Installation

`simhash.mq` imports [ds.mq](https://github.com/harehare/ds.mq) for its union-find, which is included as a git submodule:

```sh
git clone --recurse-submodules https://github.com/harehare/simhash.mq
```

Use it by adding both directories to the module path:

```sh
mq -L simhash.mq -L simhash.mq/vendor/ds.mq -I null 'import "simhash" | simhash::fingerprint("hello world")'
```

## Usage

```sh
mq -L . -L vendor/ds.mq -I null '
  import "simhash"
  | simhash::clusters([
      "the quick brown fox jumps over the lazy dog near the river bank today",
      "completely unrelated sentence regarding databases indexing and query planning",
      "the quick brown fox jumped over the lazy dog near the river bank today",
      "another unrelated line about cooking pasta with tomatoes and fresh basil leaves"
    ], 12)
'
# [[0, 2], [1], [3]]
```

A fingerprint is a string of 64 characters, each `"0"` or `"1"`, and the distance between two fingerprints is the number of positions where they differ. Unrelated texts sit around 32 apart.

| Function | Description |
| --- | --- |
| `fingerprint(text)` | fingerprint of the words in a text, ignoring order |
| `fingerprint_shingles(text, n)` | fingerprint over runs of `n` words, which also reflects word order |
| `fingerprint_tokens(tokens)` | fingerprint of an array of tokens; repeated tokens count more |
| `distance(a, b)` | number of differing positions |
| `near(a, b, max_distance)` | whether two fingerprints are within `max_distance` |
| `clusters(texts, max_distance)` | groups of indexes whose fingerprints chain within `max_distance` |
| `cluster_fingerprints(fps, max_distance)` | the same, for fingerprints you already have |
| `tokenize(text)` / `shingles(tokens, n)` | the word splitter and the n-word runs used above |

Groups are ordered by their smallest index, and a text that matches nothing is a group of one. Texts with no words get 64 zeros, so they all land in one group.

## Notes

- Clustering is exact: it returns the groups a comparison of every pair would, but only compares fingerprints that share a slice of their bits. `max_distance + 1` slices are used, so very large distances compare more pairs.
- Fingerprints chain: if A is near B and B is near C, all three share a group even when A and C are not near each other.
- Words are split on anything that is not a letter or a digit, so text written without spaces stays one token per run.

## Development

```sh
git submodule update --init
mq-test -L vendor/ds.mq simhash_tests.mq
mq-bench -L vendor/ds.mq -L . simhash_bench.mq
```

One of the goals of this module is also to check mq for performance problems, so it comes with a benchmark in `simhash_bench.mq`.
