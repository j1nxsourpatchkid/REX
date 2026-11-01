# REX — Practical Examples

REX can be used in a range of reverse-engineering, software analysis, and digital forensics workflows. The examples below show how its commands can help investigate unfamiliar files and understand how compiled software is structured.

REX performs static analysis of files and offline analysis of packet captures. It does not need to execute a target binary to inspect its contents. Its heuristic findings are indicators to investigate further, not proof of malicious behaviour.

## 1. Initial Triage of an Unknown Executable

**Scenario:** You have received an unfamiliar executable and want to gather basic information before deciding what to investigate next.

**Command:**

```bash
rex investigate invoice.exe
```

**What REX can show:**

- File format, architecture, entry point, and cryptographic hashes.
- Section sizes and entropy measurements.
- Compiler and linker indicators, plus any recognized packer signatures.
- Imported libraries and exported symbol counts.
- Extracted strings, including URL and filesystem path indicators.
- Disassembly preview from the entry point.
- Correlated findings labelled FACT, PROBABLE, or SPECULATIVE.
- Evidence Chains connecting technical indicators to investigation conclusions.
- Behavioral categories for recognized imported APIs, such as Network / Socket, Process Injection, and Virtual Memory Allocation.

Evidence Chains help explain why REX reached a particular conclusion by connecting the supporting technical indicators instead of presenting an isolated alert.

The hashes can be used to look up the file in an external reputation service. REX itself does not submit files to VirusTotal or determine whether a file is malware.

High entropy or unusual imports can be worth investigating, but neither proves that a file is packed or malicious.

## 2. Deep Function-Level Analysis

**Scenario:** You have identified an interesting virtual address, such as an entry point, exported symbol, or API caller, and want a more detailed breakdown of the function.

**Command:**

```bash
rex function target.exe --address 0x140001000
```

To analyze the entry point instead, omit the address:

```bash
rex function target.exe
```

**What REX can show:**

- Function boundaries, estimated byte size, and instruction count.
- Basic block count and branch information.
- Incoming callers and references.
- Outgoing function calls.
- Resolved imported APIs with behavioral tags, such as `VirtualAlloc` (Virtual Memory Allocation).
- Read-only data strings referenced directly by instructions within the function.
- Confidence classifications: FACT, PROBABLE, or SPECULATIVE.

This brings together function discovery, disassembly, cross-reference analysis, API categorization, and other available findings into one report.

The results depend on the information that can be recovered from the binary. Function boundaries and some relationships may be estimated rather than definitively known.

## 3. Investigating Build Information

**Scenario:** You want to learn what compiler or toolchain may have been used to build an executable, or whether the binary contains leftover development information.

**Command:**

```bash
rex fingerprint sample.exe
```

**What REX can show:**

- Indicators associated with Rust, Go, GCC, Clang/LLVM, and MSVC.
- Microsoft Rich Header information when present.
- Recognized packer signatures such as UPX and Aspack.
- CodeView RSDS debug information and embedded PDB paths when available.

A PDB path may reveal a username, project name, or build directory. For example, a path such as `C:\Users\Developer\source\project\Release\app.pdb` can expose details about the build environment.

These clues do not necessarily identify the person who wrote the software. Build paths and compiler indicators can be incomplete, altered, or misleading.

## 4. Finding Embedded File Signatures

**Scenario:** A firmware image, installer, or other large binary may contain recognizable files embedded inside it.

**Command:**

```bash
rex carve installer.bin
```

**What REX can show:**

REX scans for known file signatures, including PNG, JPEG, GIF, PDF, ZIP/JAR, 7-Zip, RAR, PE, and ELF.

It reports the detected signature types and their offsets within the input file. These offsets can help you identify regions that may be worth examining with a separate extraction tool.

Finding a signature does not guarantee that the complete embedded file is present or that REX has extracted and reconstructed it.

## 5. Comparing Two Software Versions

**Scenario:** You have two versions of a DLL or executable and want to understand how their structures differ.

**Command:**

```bash
rex diff library_v1.dll library_v2.dll
```

**What REX can show:**

- File size changes and byte-level similarity.
- Whether the files are bitwise identical.
- Entry-point changes.
- Section additions, removals, resizing, and structural differences.
- Exported symbols added to or removed from the newer version.

This is useful for tracking changes between builds, checking compatibility-related changes, and deciding whether a more detailed investigation is needed.

REX does not currently produce a complete byte-by-byte offset map of every changed byte, nor can it determine which vulnerability a software update may have fixed.

## 6. Exploring a Binary Interactively

**Scenario:** You want to inspect a binary in stages, move between addresses, examine functions, and keep notes without repeatedly launching separate commands.

**Command:**

```bash
rex explore program.exe
```

Inside the interactive explorer, you can use commands such as:

```text
rex [0x140001000]> overview
rex [0x140001000]> sections
rex [0x140001000]> funcs
rex [0x140001000]> func 140001000
rex [0x140001000]> cfg 140001000
rex [0x140001000]> hex 0 128
rex [0x140001000]> xref GetModuleHandle
rex [0x140001000]> jump 1400021a0
rex [0x1400021a0]> disasm 15
rex [0x1400021a0]> mark 1400021a0 Interesting routine
rex [0x1400021a0]> marks
```

**What REX can do:**

- Display binary metadata and section information.
- Browse automatically discovered functions across executable sections.
- Generate ASCII Control Flow Graph diagrams for functions.
- Inspect raw bytes with an integrated hexadecimal and ASCII viewer.
- Run detailed function-level analysis.
- Search for references to imported APIs or addresses.
- Navigate to a virtual address and inspect instructions there.
- Resolve supported PE import references in disassembly.
- Search strings, imports, and correlated findings.
- Save address bookmarks with notes that persist between sessions.

Bookmarks are automatically saved to `.rex_bookmarks.json` in the working directory and reloaded when the explorer is opened again.

The explorer is useful for focused terminal-based inspection. It is not a replacement for a full graphical disassembler when you need advanced decompilation or extensive control-flow visualization.

## 7. Examining a Memory Dump

**Scenario:** You have a raw memory dump and want to identify regions with characteristics that may deserve closer examination.

**Command:**

```bash
rex memory process_dump.raw
```

**What REX can show:**

- 64 KB chunks of the input file.
- Candidate 64-bit virtual pointers found at aligned offsets.
- Pointer density measurements for each chunk.
- Heuristic classifications such as sparse, pointer-dense, high-entropy, or standard data.

This can help narrow down which regions to inspect with more specialized memory-forensics tools.

Pointer-dense regions may contain structured data, while high-entropy regions may contain compressed or encrypted content. These measurements do not independently identify objects, virtual tables, injected code, or encryption keys.

## 8. Summarizing a Network Capture

**Scenario:** You have an offline PCAP capture and want a quick overview of its packets and common protocols.

**Command:**

```bash
rex pcap network_capture.pcap
```

**What REX can show:**

- Ethernet, IPv4, IPv6, ARP, and ICMP packet information.
- TCP and UDP packet details.
- Common protocol indicators based on ports, including HTTP, TLS, and DNS.
- Packet identifiers, timestamps, lengths, source and destination information.
- Overall protocol distribution.

This is useful for an initial look at a capture before opening it in a dedicated network-analysis application.

REX currently provides per-packet summaries and protocol counts. It does not build full bidirectional TCP/UDP conversation histories or determine which host is infected. The current parser is for classic PCAP, not PCAPNG.

## 9. Examining an Unknown Binary Format

**Scenario:** You have a data file produced by an unfamiliar application or device and want to look for clues about how its records may be organized.

**Command:**

```bash
rex protocol stream_dump.bin
```

**What REX can look for:**

- Potential magic or preamble bytes.
- CRLF delimiters.
- Possible big-endian length-prefixed framing.
- Patterns in sampled 4-byte fields.
- Indicators resembling HTTP headers, TLS records, null padding, or structured payload blocks.

These observations can provide starting points for further format research. They are hypotheses based on byte patterns, not confirmation of a protocol specification.

## 10. Working Through Binary Analysis Challenges

**Scenario:** You are solving a legal CTF challenge or analyzing a sample specifically provided for reverse-engineering practice.

**Useful commands:**

```bash
rex strings challenge.bin --min 6
rex search challenge.bin "flag{"
rex binary challenge.bin
rex disasm challenge.bin --count 50
rex carve challenge.bin
rex funcs challenge.bin
```

**What REX can help with:**

- Finding readable strings and embedded indicators.
- Searching for a known text pattern.
- Identifying the file format and section layout.
- Inspecting the first instructions at the entry point.
- Discovering potential function boundaries.
- Locating embedded file signatures.

These commands can help organize an initial investigation. More complex challenges may require additional disassembly, debugging, or specialized tools.

## 11. Auditing a Directory of Files

**Scenario:** You have a folder containing files with unfamiliar names or extensions and want to identify recognizable file types.

**Command:**

```bash
rex scan ./samples/
```

**What REX can show:**

- Recognized PE, ELF, Mach-O, and PCAP file types based on file signatures.
- File sizes.
- Entropy-based indicators that may flag potentially packed or compressed binaries.

REX identifies files using their contents rather than relying only on their extensions.

The current scan checks files directly inside the supplied directory. It does not recursively scan nested folders.

## 12. Producing an Investigation Report

**Scenario:** You have completed an investigation and want to save its results in a format that can be shared or reviewed later.

**Commands:**

```bash
rex investigate sample.exe -f json > report.json
rex investigate sample.exe -f markdown > report.md
rex investigate sample.exe -f html > report.html
rex investigate sample.exe -f csv > report.csv
```

**What REX can produce:**

Full investigations support plain text, JSON, Markdown, HTML, and CSV output. These formats make it easier to archive results, process them with other tools, or share findings with teammates.

The additional Markdown, HTML, and CSV formats are currently supported by `investigate`; other commands may fall back to plain text when a requested format is not implemented.

## 13. Reviewing Previous Investigations

**Scenario:** You want to revisit which files you previously analyzed with REX.

**Command:**

```bash
rex history
```

**What REX can show:**

The command reads the local `.rex_history` ledger and displays previous targets recorded by `rex investigate`.

This provides a simple record of past investigation runs.

## Important Notes

REX is designed to assist with static binary inspection and offline data analysis. Some of its results are heuristic and should be validated with additional evidence.

A few current implementation limits are worth keeping in mind:

- The PCAP command parses classic PCAP captures, not PCAPNG.
- Directory scanning currently checks immediate files, not nested subdirectories.
- The carver reports recognized signatures and offsets; it does not guarantee complete file extraction.
- Binary comparison reports similarity and structural changes, not a complete byte-level change map.
- Memory, function discovery, and protocol classifications can be heuristic and are not definitive identifications.
- API behavioral categories describe recognized imported APIs, not proof that a particular behavior occurs.
- Evidence Chains explain the indicators supporting an inference, but the underlying evidence should still be reviewed.
- Bookmarks persist in `.rex_bookmarks.json` in the working directory.
- Markdown, HTML, and CSV reporting are available for `rex investigate`; other commands may fall back to text.

REX is most useful as a unified starting point for investigating unfamiliar binaries and data, correlating findings, and deciding where deeper analysis should take place.
