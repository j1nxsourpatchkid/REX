# REX — Command Reference

This document explains the commands currently available in REX, how to use them, and what each command does.

REX has two command environments:

1. **CLI Commands:** Run these from PowerShell, Command Prompt, or Bash using `rex [command]`.
2. **Interactive Explorer Commands:** Run these after launching `rex explore <path>`.

The explorer provides its own prompt, typically:

```text
rex [0x140001000]>
```

Commands entered here are explorer commands, not normal terminal commands.

# Part 1 — CLI Commands

## 1. EXPLORE

**Usage:**

```bash
rex explore <path>
```

**What it does:**

Opens the interactive terminal explorer for the specified executable or binary. REX pre-computes and caches analysis data in memory, then opens an interactive prompt where you can inspect sections, disassemble code, search results, follow cross-references, jump to addresses, browse functions, and create bookmarks.

**Example:**

```bash
rex explore program.exe
```

**Starting REX without arguments:**

Running REX without arguments prompts you to enter a file path and then launches the interactive explorer.

## 2. INVESTIGATE

**Usage:**

```bash
rex investigate <path>
```

**What it does:**

Runs a combined investigation using multiple REX analysis modules. This is the main all-in-one analysis command.

The investigation:

- Calculates MD5, SHA-1, SHA-256, and SHA-512 hashes.
- Inspects binary format, sections, structure, and Shannon entropy.
- Fingerprints compiler and linker indicators, checks for packer signatures, and extracts available PDB debug paths.
- Summarizes dynamic libraries, import counts, and export counts.
- Disassembles instructions at the entry point and estimates basic blocks.
- Extracts strings and identifies URL and filesystem path indicators.
- Correlates results across modules and produces findings labelled FACT, PROBABLE, or SPECULATIVE.
- Generates Evidence Chains connecting technical indicators to correlated conclusions.
- Categorizes recognized imported APIs by behavioral classification.
- Records the investigation in the local `.rex_history` file.

**Example:**

```bash
rex investigate program.exe
```

## 3. BINARY

**Usage:**

```bash
rex binary <path>
```

**What it does:**

Inspects the binary's format and basic executable structure. Supported formats include PE, ELF, and Mach-O.

Reports information such as:

- CPU architecture, including x86, x64, and ARM where identified.
- Endianness and file size.
- Entry-point virtual address.
- Cryptographic hashes.
- Section names, virtual addresses, raw sizes, and Shannon entropy.
- Heuristic section classifications: PACKED, SPARSE, or NORMAL.

**Example:**

```bash
rex binary program.exe
```

## 4. FINGERPRINT

**Usage:**

```bash
rex fingerprint <path>
```

**What it does:**

Searches for compiler, linker, and packer indicators that may reveal information about how a binary was built.

It can:

- Identify indicators associated with Rust (rustc), Go (gc), GCC, Clang/LLVM, and MSVC.
- Check for Microsoft Rich Header data, including Rich/DanS information.
- Search for known packer signatures such as UPX and Aspack.
- Parse CodeView RSDS debug information and extract an embedded PDB path when present.

**Example:**

```bash
rex fingerprint program.exe
```

## 5. DEPS

**Usage:**

```bash
rex deps <path>
```

**What it does:**

Inspects the binary's dynamic dependencies and symbol information.

It can display:

- Dynamically linked DLLs and shared libraries.
- Imported API functions.
- Exported ABI symbols and their virtual addresses.
- Behavioral categories assigned to recognized imported APIs.

Recognized API categories include:

- Network / Socket
- Process / Thread Control
- Process Injection
- Virtual Memory Allocation
- File System Access
- Registry Configuration
- Cryptography / Hashing
- Anti-Analysis / Debug Check
- Reconnaissance / System Information
- General Runtime

These categories describe the operational areas associated with recognized APIs. They do not independently prove that a particular behavior occurs.

In text output, the import list may be limited to the first 200 entries.

**Example:**

```bash
rex deps program.exe
```

## 6. DISASM

**Usage:**

```bash
rex disasm <path> [--count N]
```

**What it does:**

Disassembles machine code starting at the binary's entry point using iced-x86 and Intel syntax.

For supported PE files, REX can resolve Import Address Table (IAT) references and show imported function names alongside relevant instructions.

The output can include:

- Instruction addresses and byte encodings.
- Decoded assembly instructions.
- Recognized function prologues and epilogues.
- Recognized GS stack-cookie and synchronization-lock patterns.
- Opcode distribution statistics.
- Jump, call, and estimated basic-block information.

**Options:**

- `--count N`: Number of instructions to decode. Default: 30.

**Examples:**

```bash
rex disasm program.exe
rex disasm program.exe --count 100
```

## 7. STRINGS

**Usage:**

```bash
rex strings <path> [--min N] [--limit N]
```

**What it does:**

Extracts contiguous printable strings from a file in ASCII and UTF-16LE encodings.

REX can also identify and tally indicators such as:

- URLs beginning with `http://` or `https://`.
- Windows and Unix-style filesystem paths.
- File and library references such as `.dll`.

**Options:**

- `--min N`: Minimum string length. Default: 4.
- `--limit N`: Maximum number of strings displayed. Default: 100.

**Examples:**

```bash
rex strings program.exe
rex strings program.exe --min 6
rex strings program.exe --min 5 --limit 250
```

## 8. DIFF

**Usage:**

```bash
rex diff <first> <second>
```

**What it does:**

Compares two binary files and reports information about their differences and structural changes.

It can:

- Compare file sizes and calculate the size delta.
- Calculate byte-level similarity.
- Check whether the files are bitwise identical.
- Compare entry-point addresses.
- Compare section geometry and report size changes for matching sections.
- Identify exported functions added to or removed from the second file.

**Examples:**

```bash
rex diff old.exe new.exe
rex diff version1.dll version2.dll
```

## 9. MEMORY

**Usage:**

```bash
rex memory <path>
```

**What it does:**

Analyzes a raw memory dump by dividing it into 64 KB chunks.

REX scans aligned 8-byte values for candidate 64-bit virtual pointers, including values in recognized user-space and kernel-space address ranges. It calculates pointer density for each chunk and uses heuristics to classify regions.

Possible classifications include:

- Zero / Sparse Page
- Pointer-Dense
- High Entropy
- Standard Data / Heap

These are heuristic classifications, not definitive identifications of what a memory region contains.

**Example:**

```bash
rex memory memory_dump.bin
```

## 10. PCAP

**Usage:**

```bash
rex pcap <path>
```

**What it does:**

Parses an offline PCAP network packet capture.

Supported packet and network formats include:

- Ethernet frames.
- IPv4 and IPv6.
- ARP and ICMP.
- TCP and UDP.

REX identifies common application protocols using familiar port numbers, including HTTP on port 80, TLS on port 443, and DNS on port 53.

The output includes packet identifiers, timestamps, lengths, source and destination information, and protocol distribution statistics.

This command parses classic PCAP captures. PCAPNG support should not be assumed.

**Example:**

```bash
rex pcap capture.pcap
```

## 11. PROTOCOL

**Usage:**

```bash
rex protocol <path>
```

**What it does:**

Examines a raw binary stream or unknown data file for possible protocol structure and framing patterns.

It can:

- Extract potential magic or preamble bytes.
- Look for CRLF delimiters.
- Check for possible big-endian length-prefixed framing.
- Examine raw-stream patterns.
- Sample 4-byte fields and look for indicators such as HTTP headers, TLS records, null padding, or structured payload blocks.

The results are structural hypotheses. They do not automatically prove that a file uses a particular protocol.

**Example:**

```bash
rex protocol unknown_data.bin
```

## 12. CARVE

**Usage:**

```bash
rex carve <path>
```

**What it does:**

Searches raw binary data for known file signatures that may indicate embedded or nested files. It reports the offsets where matching signatures are found.

Recognized signatures include:

- PNG
- JPEG
- GIF87a and GIF89a
- PDF
- ZIP / JAR
- Embedded PE files (MZ)
- Embedded ELF files
- 7-Zip
- RAR

The command reports detected signatures and offsets. It does not guarantee complete extraction or reconstruction of embedded files.

**Example:**

```bash
rex carve firmware.bin
```

## 13. SCAN

**Usage:**

```bash
rex scan <directory>
```

**What it does:**

Scans files located directly inside the specified directory and identifies file types using their magic bytes.

Recognized types include PE, ELF, Mach-O, and PCAP.

REX also reports file sizes and uses Shannon entropy as a heuristic to flag potentially packed or compressed binaries. An entropy value above 7.2 is treated as a PACKED indicator.

This command scans the directory's immediate files; it does not recursively traverse subdirectories.

**Example:**

```bash
rex scan ./samples
```

## 14. SEARCH

**Usage:**

```bash
rex search <path> <query>
```

**What it does:**

Searches a file's raw bytes for the supplied query.

It reports matching byte offsets and surrounding context, including an ASCII preview. The output displays up to 100 matches.

**Examples:**

```bash
rex search program.exe kernel32
rex search firmware.bin config
```

## 15. HISTORY

**Usage:**

```bash
rex history
```

**What it does:**

Reads and displays the local `.rex_history` file. The history contains previous targets analyzed using `rex investigate`.

**Example:**

```bash
rex history
```

## 16. FUNCTION

**Usage:**

```bash
rex function <path> [--address <hex>]
```

**What it does:**

Generates a comprehensive function-level intelligence report for a discovered or specified virtual address.

If `--address` is omitted, REX analyzes the binary's entry point by default.

The report can include:

- Function boundaries, estimated byte size, and instruction count.
- Basic block count and execution branch counts.
- Incoming callers and references.
- Outgoing function calls.
- Resolved imported APIs with behavioral categories.
- Read-only data strings referenced directly by instructions within the function.
- Confidence classifications.

This command connects function discovery, disassembly, cross-reference analysis, API categorization, and other available findings into one report.

**Examples:**

```bash
rex function program.exe
rex function program.exe --address 0x140001000
```

## 17. FUNCS

**Usage:**

```bash
rex funcs <path>
```

**What it does:**

Discovers potential unexported function boundaries across executable sections.

It displays discovered function information, including:

- Virtual addresses.
- Generated function names.
- Confidence classifications.
- Caller counts.

Function discovery is based on recognizable code patterns and structural indicators. Identified boundaries are estimates and should not automatically be treated as definitive function boundaries.

**Example:**

```bash
rex funcs program.exe
```

## 18. CFG

**Usage:**

```bash
rex cfg <path> [--address <hex>]
```

**What it does:**

Generates and renders an ASCII Control Flow Graph for a function.

The diagram represents basic blocks and their relationships, including jump edges, conditional branches, fallthrough paths, and instruction flows.

**Examples:**

```bash
rex cfg program.exe
rex cfg program.exe --address 0x140001000
```

## 19. HEX

**Usage:**

```bash
rex hex <path> [--offset <hex>] [--length <N>]
```

**What it does:**

Displays a hexadecimal dump of the specified file region with an aligned ASCII side-pane.

This allows raw binary bytes and their printable character representations to be inspected together.

**Options:**

- `--offset <hex>`: Starting file offset. Defaults to 0.
- `--length N`: Number of bytes to display.

**Example:**

```bash
rex hex program.exe --offset 0 --length 128
```

# CLI Output Formats

Some commands accept the `--format` or `-f` option to select an output format.

Available formats:

| Option | Output |
|---|---|
| `-f text` | Plain-text output (default) |
| `-f json` | JSON output |
| `-f markdown` | Markdown output |
| `-f md` | Markdown output |
| `-f html` | HTML output |
| `-f htm` | HTML output |
| `-f csv` | CSV output |

JSON output is supported across the CLI commands. Markdown, HTML, and CSV reporting are supported by the `investigate` command. Other commands fall back to plain text for formats they do not implement.

**Examples:**

```bash
rex investigate program.exe --format json
rex investigate program.exe -f html
rex investigate program.exe -f csv
rex binary program.exe -f json
```

# Part 2 — Interactive Explorer Commands

Launch the explorer from your terminal with:

```bash
rex explore <path>
```

Once the explorer opens, commands are entered at its interactive prompt. The prompt displays the current navigation address, for example:

```text
rex [0x140001000]>
```

The commands below are entered inside this prompt, not directly into PowerShell, Command Prompt, or Bash.

## 1. OVERVIEW

**Usage:**

```text
overview
1
```

**What it does:**

Displays the binary's main metadata, including format, architecture, hashes, and entry-point information.

**Example:**

```text
rex [0x140001000]> overview
```

## 2. SECTIONS

**Usage:**

```text
sections
2
```

**What it does:**

Displays the binary's section table, including section names, virtual addresses, raw sizes, Shannon entropy, and heuristic packing classifications.

**Example:**

```text
rex [0x140001000]> sections
```

## 3. DISASM

**Usage:**

```text
disasm [N]
3
```

**What it does:**

Disassembles instructions starting at the current navigation address. PE IAT references can be resolved to imported function names inline.

**Options:**

- `[N]`: Number of instructions to display. Default: 15.

**Examples:**

```text
rex [0x140001000]> disasm
rex [0x140001000]> disasm 40
```

## 4. JUMP

**Usage:**

```text
jump <hex>
goto
```

**What it does:**

Changes the current navigation address to the specified hexadecimal virtual address and displays the first 10 instructions at that location.

**Example:**

```text
rex [0x140001000]> jump 140002000
```

## 5. XREF

**Usage:**

```text
xref <term>
xrefs
```

**What it does:**

Searches code sections for calls or IP-relative references targeting the supplied API name or address. This helps locate code that references an imported function or a specific address.

**Examples:**

```text
rex [0x140001000]> xref GetModuleHandle
rex [0x140001000]> xref 140002040
```

## 6. DEPS

**Usage:**

```text
deps
4
```

**What it does:**

Displays the binary's dynamic libraries, imported APIs, and exported symbols.

Recognized imported APIs are also grouped into behavioral categories, providing additional context about their operational purpose.

**Example:**

```text
rex [0x140001000]> deps
```

## 7. STRINGS

**Usage:**

```text
strings [filter]
5
```

**What it does:**

Displays extracted strings from the binary. An optional filter limits the displayed results to strings containing the supplied keyword.

**Examples:**

```text
rex [0x140001000]> strings
rex [0x140001000]> strings http
```

## 8. CARVE

**Usage:**

```text
carve
6
```

**What it does:**

Lists embedded files detected by the binary carving module, along with their detected types and offsets.

**Example:**

```text
rex [0x140001000]> carve
```

## 9. FINDINGS

**Usage:**

```text
findings
7
```

**What it does:**

Displays conclusions produced by the cross-module correlation engine.

Findings are labelled FACT, PROBABLE, or SPECULATIVE according to the confidence assigned to each conclusion. Evidence Chains provide additional context about the technical indicators supporting those conclusions.

**Example:**

```text
rex [0x140001000]> findings
```

## 10. FIND

**Usage:**

```text
find <query>
search
```

**What it does:**

Searches across multiple types of investigation results, including imported APIs, extracted strings, and correlated findings.

**Examples:**

```text
rex [0x140001000]> find CreateFile
rex [0x140001000]> find https
```

## 11. MARK

**Usage:**

```text
mark <hex> <note>
bookmark
```

**What it does:**

Saves a virtual address as a bookmark with a custom text note.

Bookmarks are automatically serialized to `.rex_bookmarks.json` in the working directory. They persist between explorer sessions and are reloaded when the explorer is opened again.

**Example:**

```text
rex [0x140001000]> mark 140002040 Possible initialization routine
```

## 12. MARKS

**Usage:**

```text
marks
bookmarks
```

**What it does:**

Lists saved bookmarks and their associated notes, including bookmarks loaded from previous sessions.

**Example:**

```text
rex [0x140001000]> marks
```

## 13. FUNCS

**Usage:**

```text
funcs
functions
```

**What it does:**

Lists discovered function boundaries across executable sections.

The output includes virtual addresses, generated names, confidence classifications, and caller counts.

**Example:**

```text
rex [0x140001000]> funcs
```

## 14. FUNC

**Usage:**

```text
func <addr>
fn
```

**What it does:**

Runs a detailed function-level intelligence report for the specified virtual address. If no address is supplied, it uses the current navigation address.

The report can include function size, instruction count, basic blocks, branches, incoming callers, outgoing calls, imported APIs, referenced strings, and confidence classifications.

**Example:**

```text
rex [0x140001000]> func 140001000
```

## 15. CFG

**Usage:**

```text
cfg [addr]
graph
```

**What it does:**

Generates an ASCII Control Flow Graph for the current or specified function.

The diagram displays basic blocks and their control-flow relationships, including conditional branches, jump edges, fallthrough paths, and function exits.

**Examples:**

```text
rex [0x140001000]> cfg
rex [0x140001000]> cfg 140001000
```

## 16. HEX

**Usage:**

```text
hex [offset] [len]
dump
```

**What it does:**

Displays a hexadecimal dump with an aligned ASCII side-pane.

The optional offset selects the starting file offset, while the optional length controls how many bytes are displayed.

**Examples:**

```text
rex [0x140001000]> hex
rex [0x140001000]> hex 0 128
```

## 17. HELP

**Usage:**

```text
help
```

**What it does:**

Displays the interactive explorer command reference.

**Example:**

```text
rex [0x140001000]> help
```

## 18. QUIT

**Usage:**

```text
quit
q
exit
```

**What it does:**

Closes the interactive explorer session and returns to the terminal.

**Example:**

```text
rex [0x140001000]> quit
```

# Quick Reference

## CLI Commands

| Command | Description |
|---|---|
| `rex explore <path>` | Open interactive explorer |
| `rex investigate <path>` | Run a full investigation |
| `rex binary <path>` | Inspect binary structure |
| `rex fingerprint <path>` | Identify compiler/toolchain clues |
| `rex deps <path>` | Inspect dependencies and symbols |
| `rex disasm <path> [--count N]` | Disassemble from entry point |
| `rex strings <path> [--min N]` | Extract strings |
| `rex diff <first> <second>` | Compare two binaries |
| `rex memory <path>` | Analyze a memory dump |
| `rex pcap <path>` | Parse a PCAP capture |
| `rex protocol <path>` | Inspect possible protocol structure |
| `rex carve <path>` | Find embedded file signatures |
| `rex scan <directory>` | Scan immediate directory contents |
| `rex search <path> <query>` | Search raw file bytes |
| `rex history` | View investigation history |
| `rex function <path> [--address hex]` | Analyze a specific function |
| `rex funcs <path>` | Discover function boundaries |
| `rex cfg <path> [--address hex]` | Generate a Control Flow Graph |
| `rex hex <path> [--offset hex] [--length N]` | Display hexadecimal data |

## Interactive Explorer Commands

| Command | Shorthand | Description |
|---|---|---|
| `overview` | `1` | Show binary overview |
| `sections` | `2` | List binary sections |
| `disasm [N]` | `3` | Disassemble at current address |
| `jump <hex>` | `goto` | Navigate to an address |
| `xref <term>` | `xrefs` | Find references to an API or address |
| `deps` | `4` | View dependencies and symbols |
| `strings [filter]` | `5` | Browse extracted strings |
| `carve` | `6` | List embedded files |
| `findings` | `7` | View correlated findings and evidence |
| `find <query>` | `search` | Search analysis results |
| `mark <hex> <note>` | `bookmark` | Save a persistent address bookmark |
| `marks` | `bookmarks` | List saved bookmarks |
| `funcs` | `functions` | Browse discovered functions |
| `func [addr]` | `fn` | Run function-level analysis |
| `cfg [addr]` | `graph` | Generate an ASCII Control Flow Graph |
| `hex [offset] [len]` | `dump` | Display hexadecimal and ASCII data |
| `help` | — | Show explorer help |
| `quit` | `q`, `exit` | Exit the explorer |

# Notes

- REX is an analysis toolkit. Several outputs, including entropy-based packing classifications, memory-region classifications, function boundary estimates, and protocol structure hypotheses, are heuristic and should be interpreted as indicators rather than definitive conclusions.
- API behavioral categories describe recognized API functionality and do not independently prove that a particular behavior occurs.
- Evidence Chains explain which available indicators contribute to an inference. Their conclusions should still be evaluated against the underlying data.
- Bookmarks are stored in `.rex_bookmarks.json` in the working directory and persist between explorer sessions.
- PCAP parsing and PCAPNG parsing are not interchangeable. The pcap command currently describes legacy PCAP capture parsing.
- Directory scanning currently covers files directly inside the supplied directory, not nested subdirectories.
- Markdown, HTML, and CSV output formats are implemented for full investigations. Other commands may use plain-text output when a requested format is not supported.
