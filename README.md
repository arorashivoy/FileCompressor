# FileCompressor
This program written in C compresses a given .txt file, which can be decompressed later using the same program

The program `8-zip.c` is using Huffman compression algorithm using min-heaps to compress a given txt file.

### Meta Data Format
The metadata is the first line of the compressed file
* The first line contains space-separated characters and number
  * The first number is the number of bits in the encoded file
  * Then, there are space-separated pairs of character with there frequency
  * `Bits char1 freq1 char2 freq2 ... charN freqN`
* From the second line the decoded message is present


#### Test Case
It compressed a file of 10KB into just 3KB including the metadata
