# FileCompressor

A Huffman compressor and decompressor for text files, written in C in a single
file with **no libraries** — the min-heap, the Huffman tree, the bit packing and
the bit reading are all implemented from scratch on top of the standard library.

## Building and running

```sh
gcc -O2 -o 8zip 8-zip.c
./8zip
```

It then asks for three things: the input file, the output file, and the mode —
**0 to compress, 1 to decompress**. So a full round trip is:

```sh
printf 'sample/prose.txt\nprose.8z\n0\n' | ./8zip     # compress
printf 'prose.8z\nprose.out\n1\n'        | ./8zip     # decompress
cmp sample/prose.txt prose.out                        # identical
```

## Results

Measured on files in this repository, so they can be reproduced exactly:

| Input | Bytes in | Bytes out | Ratio | Saved |
|---|---|---|---|---|
| `sample/prose.txt` | 2,622 | 1,670 | 0.64 | 36% |
| `LICENSE` | 11,357 | 6,799 | 0.60 | 40% |
| `8-zip.c` | 15,990 | 9,863 | 0.62 | 38% |

`sample/prose.8z` is the compressed form of `sample/prose.txt`, committed so the
decompressor can be checked without compressing anything first.

**Around 0.60 on English text is the expected result, not a disappointing one.**
English carries roughly 4.5 bits of information per character against the 8 bits a
byte spends, and a symbol-by-symbol Huffman code cannot do better than that
entropy. Getting substantially below 0.5 needs a method that models sequences
rather than single characters. Small files do worse — `prose.txt` is only 2.6 KB
and the frequency table is a fixed overhead the saved bits have to repay — and a
file under a few hundred bytes can come out larger than it went in.

An earlier version of this README claimed 10 KB compressed to 3 KB. That figure
was wrong; the table above is measured.

## How it works

**Compression** makes two passes. The first counts the frequency of every byte.
Those counts seed a min-heap, and the tree is built by repeatedly extracting the
two lowest-frequency nodes and reinserting their merged parent — so the rarest
symbols end up deepest and carry the longest codes. Walking the finished tree
assigns each symbol its code. The second pass replaces each character with its
code and packs the bits eight at a time into output bytes.

Each merge costs two extractions and one insertion, each logarithmic, and there is
one merge per symbol, so tree construction is **O(n log n)** in the size of the
alphabet. Both file passes are linear in the length of the input.

**Decompression** does not store the code table. It stores the *frequencies* and
rebuilds the identical tree from them, which is smaller to store — then walks the
tree bit by bit, emitting a symbol each time it reaches a leaf.

## File format

The first line is the header:

```
<total bits> <byte value> <frequency> <byte value> <frequency> ...
```

The encoded body follows on the next line, packed eight bits to a byte. The total
bit count is needed because the final byte is usually padded.

Symbols are written as **decimal byte values rather than as literal characters**.
That matters: space and newline are themselves symbols in any text file, so a
header that wrote them literally could not be told apart from its own delimiters.
The original format did write them literally, and as a result the decompressor
silently rebuilt a different tree and produced plausible-looking garbage rather
than the original file — decompression never actually worked. Storing byte values
makes the header unambiguous.

## Limitations

- Text files only. The input is read in text mode, and a byte value of 0 ends the
  symbol table, so arbitrary binary input is not supported.
- The whole frequency table is written even for symbols that appear once, which is
  what makes small files compress poorly.
- The program takes its arguments interactively rather than from `argv`.
