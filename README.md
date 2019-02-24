# minimal-source-map

Minimal implementation of source-map specification described in:

[Source Map Revision 3 Proposal](https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k)

## Installation

`npm i minimal-source-map`

## API

* *parseBase64VLQSegment(string)*: given a Base 64 VLQ segment, e.g,, 
    `ABCD` from mappings, return 1,4 or 5 variable length array of integers.
  * TODO: throw exceptions if wrong number of elements is returned.
  * [See: Proposed Format](https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit#heading=h.qz3o9nc69um5).
* *extractSourceMapComment(string)*: given a string representing JavaScript or CSS
  source, extract a comment of the format `//#|/*# sourceMappingURL=`.
  * [See: Linking Generated Code](https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit#heading=h.lmz475t4mvbx).
  
## Background and Rationale

* At HackIllinois 2019, [Ben Coe](https://www.github.com/bcoe) suggested the idea of rewriting a lightweight Source Map for usage with Node. The current solutions have a couple of issues that we wanted to counter:
  * Current source map is too large: it adds another chunk to Node
  * Asynchronous: when exceptions are thrown, you only have one program loop to deal with what needs to happen before the program exits. Our rewrite is not async.
Background on source mapping
* Minimized and combined files benefit performance, but the linked generated files are difficult to read and even worse to debug. Source maps ascertain where the original locations in the code are, but can give improper locations back to the user: for example, an exception can be thrown on "line 1:900," but the actual location was line 26:35. 
* Source maps aid with debugging. There are also currently exisiting APIs, but they are insufficiently lightweight for efficient Node use.
Therefore, we wanted to spike a barebones synchronous implementation.
  
## Construction

* Worked based off of the [Proposed Format](https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit#heading=h.qz3o9nc69um5). Began by creating a basic base64 encoding reader.
* Created a segment parser that would take in a segment and parsed it into individual values. This required reading the continuation and signage bits on each character of the segment. 
* The least significant bit was the furthest left. In every segment, the first character is special because it has both a continuation and a signage bit. Following bits do not use the signage bit. 
  * The continuation bit is the first bit in the binary representation, and the last bit is the signage bit. 
  * Because following bits do not have the signage bit, the actual data contained by the character are the bits ignoring continuation.
* Continuation required shifting the segment: values are read from right to left, so the new value getting read in had to be shifted right 5 places before adding in the old value.
* A lot of the mathematics utilize bit wise binary comparisons: this allows us to determine the sign and the continuation bits in multiple instances
* Got source URL through some regex and following regulation standards.
  
## Next Steps

* Currently: [NPM Package Publication](https://www.npmjs.com/package/minimal-source-map)
* Work on exception handling and taking user to actual sources of error
* Implementing more tracing of locations
* Process more utilizing the source URL

## Final Thanks
I went into this project knowing almost no JavaScript, and had never heard of source mapping before. Huge thank you to [Ben Coe](https://www.github.com/bcoe) for his incredible mentorship. Showing me how these things work together and good testing practices was an enormous boon. I learned a ton and am looking forward to future projects!
