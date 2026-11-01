REX — REVERSE ENGINEERING EXPLORER COMMAND REFERENCE =================

This document explains the commands currently available in REX, how to
use them, and what each command does.

REX has two command environments:

1.  CLI COMMANDS Run these from PowerShell, Command Prompt, or Bash
    using: rex [options]

2.  INTERACTIVE EXPLORER COMMANDS Run these after launching: rex explore

    The explorer provides its own prompt, typically: rex [0xENTRY]>

    Commands entered here are explorer commands, not normal terminal
    commands.

PART 1 — CLI COMMANDS

1. EXPLORE

Usage: rex explore

What it does: Opens the interactive terminal explorer for the specified
executable or binary. REX pre-computes and caches analysis data in
memory, then opens an interactive prompt where you can inspect sections,
disassemble code, search results, follow cross-references, jump to
addresses, and create bookmarks.

Example: rex explore program.exe

Starting REX without arguments: rex

    Running REX without arguments prompts you to enter a file path and then
    launches the interactive explorer.

2. INVESTIGATE

Usage: rex investigate

What it does: Runs a combined investigation using multiple REX analysis
modules. This is the main all-in-one analysis command.

    The investigation:
    - Calculates MD5, SHA-1, SHA-256, and SHA-512 hashes.
    - Inspects binary format, sections, structure, and Shannon entropy.
    - Fingerprints compiler and linker indicators, checks for packer
      signatures, and extracts available PDB debug paths.
    - Summarizes dynamic libraries, import counts, and export counts.
    - Disassembles instructions at the entry point and estimates basic blocks.
    - Extracts strings and identifies URL and filesystem path indicators.
    - Correlates results across modules and produces findings labelled
      FACT, PROBABLE, or SPECULATIVE.
    - Records the investigation in the local .rex_history file.

Example: rex investigate program.exe

3. BINARY

Usage: rex binary

What it does: Inspects the binary’s format and basic executable
structure. Supported formats include PE, ELF, and Mach-O.

    Reports information such as:
    - CPU architecture, including x86, x64, and ARM where identified.
    - Endianness and file size.
    - Entry-point virtual address.
    - Cryptographic hashes.
    - Section names, virtual addresses, raw sizes, and Shannon entropy.
    - Heuristic section classifications: PACKED, SPARSE, or NORMAL.

Example: rex binary program.exe

4. FINGERPRINT

Usage: rex fingerprint

What it does: Searches for compiler, linker, and packer indicators that
may reveal information about how a binary was built.

    It can:
    - Identify indicators associated with Rust (rustc), Go (gc), GCC,
      Clang/LLVM, and MSVC.
    - Check for Microsoft Rich Header data, including Rich/DanS information.
    - Search for known packer signatures such as UPX and Aspack.
    - Parse CodeView RSDS debug information and extract an embedded PDB path
      when present.

Example: rex fingerprint program.exe

5. DEPS

Usage: rex deps

What it does: Inspects the binary’s dynamic dependencies and symbol
information.

    It can display:
    - Dynamically linked DLLs and shared libraries.
    - Imported API functions.
    - Exported ABI symbols and their virtual addresses.

    In text output, the import list may be limited to the first 200 entries.

Example: rex deps program.exe

6. DISASM

Usage: rex disasm [–count ]

What it does: Disassembles machine code starting at the binary’s entry
point using iced-x86 and Intel syntax.

    For supported PE files, REX can resolve Import Address Table (IAT)
    references and show imported function names alongside relevant
    instructions.

    The output can include:
    - Instruction addresses and byte encodings.
    - Decoded assembly instructions.
    - Recognized function prologues and epilogues.
    - Recognized GS stack-cookie and synchronization-lock patterns.
    - Opcode distribution statistics.
    - Jump, call, and estimated basic-block information.

Options: –count Number of instructions to decode. Default: 30.

Examples: rex disasm program.exe rex disasm program.exe –count 100

7. STRINGS

Usage: rex strings [–min ] [–limit ]

What it does: Extracts contiguous printable strings from a file in ASCII
and UTF-16LE encodings.

    REX can also identify and tally indicators such as:
    - URLs beginning with http:// or https://.
    - Windows and Unix-style filesystem paths.
    - File and library references such as .dll.

Options: –min Minimum string length. Default: 4. –limit Maximum number
of strings displayed. Default: 100.

Examples: rex strings program.exe rex strings program.exe –min 6 rex
strings program.exe –min 5 –limit 250

8. DIFF

Usage: rex diff

What it does: Compares two binary files and reports information about
their differences and structural changes.

    It can:
    - Compare file sizes and calculate the size delta.
    - Calculate byte-level similarity.
    - Check whether the files are bitwise identical.
    - Compare entry-point addresses.
    - Compare section geometry and report size changes for matching sections.
    - Identify exported functions added to or removed from the second file.

Examples: rex diff old.exe new.exe rex diff version1.dll version2.dll

9. MEMORY

Usage: rex memory

What it does: Analyzes a raw memory dump by dividing it into 64 KB
chunks.

    REX scans aligned 8-byte values for candidate 64-bit virtual pointers,
    including values in recognized user-space and kernel-space address
    ranges. It calculates pointer density for each chunk and uses heuristics
    to classify regions.

    Possible classifications include:
    - Zero / Sparse Page
    - Pointer-Dense (possible structures or vtables)
    - High Entropy (possibly compressed or encrypted data)
    - Standard Data / Heap

    These are heuristic classifications, not definitive identifications
    of what a memory region contains.

Example: rex memory memory_dump.bin

10. PCAP

Usage: rex pcap

What it does: Parses an offline PCAP network packet capture.

    Supported packet and network formats include:
    - Ethernet frames.
    - IPv4 and IPv6.
    - ARP and ICMP.
    - TCP and UDP.

    REX identifies common application protocols using familiar port
    numbers, including HTTP on port 80, TLS on port 443, and DNS on port 53.

    The output includes packet identifiers, timestamps, lengths, source and
    destination information, and protocol distribution statistics.

    This command parses PCAP captures. PCAPNG support should not be assumed.

Example: rex pcap capture.pcap

11. PROTOCOL

Usage: rex protocol

What it does: Examines a raw binary stream or unknown data file for
possible protocol structure and framing patterns.

    It can:
    - Extract potential magic or preamble bytes.
    - Look for CRLF delimiters.
    - Check for possible big-endian length-prefixed framing.
    - Examine raw-stream patterns.
    - Sample 4-byte fields and look for indicators such as HTTP headers,
      TLS records, null padding, or structured payload blocks.

    The results are structural hypotheses. They do not automatically prove
    that a file uses a particular protocol.

Example: rex protocol unknown_data.bin

12. CARVE

Usage: rex carve

What it does: Searches raw binary data for known file signatures that
may indicate embedded or nested files. It reports the offsets where
matching signatures are found.

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

Example: rex carve firmware.bin

13. SCAN

Usage: rex scan

What it does: Scans files located directly inside the specified
directory and identifies file types using their magic bytes.

    Recognized types include PE, ELF, Mach-O, and PCAP.

    REX also reports file sizes and uses Shannon entropy as a heuristic
    to flag potentially packed or compressed binaries. An entropy value
    above 7.2 is treated as a PACKED indicator.

    This command scans the directory's immediate files; it does not
    recursively traverse subdirectories.

Example: rex scan ./samples

14. SEARCH

Usage: rex search

What it does: Searches a file’s raw bytes for the supplied query.

    It reports matching byte offsets and surrounding context, including
    an ASCII preview. The output displays up to 100 matches.

Examples: rex search program.exe kernel32 rex search firmware.bin config

15. HISTORY

Usage: rex history

What it does: Reads and displays the local .rex_history file. The
history contains previous targets analyzed using rex investigate.

Example: rex history

CLI OUTPUT FORMATS

Some commands accept the –format or -f option to select an output
format.

Available formats: -f text Plain-text output (default). -f json JSON
output. -f markdown Markdown output. -f md Markdown output. -f html HTML
output. -f htm HTML output. -f csv CSV output.

JSON output is supported across the CLI commands. Markdown, HTML, and
CSV reporting are supported by the investigate command. Other commands
fall back to plain text for formats they do not implement.

Examples: rex investigate program.exe –format json rex investigate
program.exe -f html rex investigate program.exe -f csv rex binary
program.exe -f json

PART 2 — INTERACTIVE EXPLORER COMMANDS

Launch the explorer from your terminal with:

    rex explore <path>

Once the explorer opens, commands are entered at its interactive prompt.
The prompt displays the current navigation address, for example:

    rex [0x140001000]>

The commands below are entered inside this prompt, not directly into
PowerShell, Command Prompt, or Bash.

1. OVERVIEW

Usage: overview 1

What it does: Displays the binary’s main metadata, including format,
architecture, hashes, and entry-point information.

Example: rex [0x140001000]> overview

2. SECTIONS

Usage: sections 2

What it does: Displays the binary’s section table, including section
names, virtual addresses, raw sizes, Shannon entropy, and heuristic
packing classifications.

Example: rex [0x140001000]> sections

3. DISASM

Usage: disasm [N] 3

What it does: Disassembles instructions starting at the current
navigation address. PE IAT references can be resolved to imported
function names inline.

Options: [N] Number of instructions to display. Default: 15.

Examples: rex [0x140001000]> disasm rex [0x140001000]> disasm 40

4. JUMP

Usage: jump goto

What it does: Changes the current navigation address to the specified
hexadecimal virtual address and displays the first 10 instructions at
that location.

Example: rex [0x140001000]> jump 140002000

5. XREF

Usage: xref xrefs

What it does: Searches code sections for calls or IP-relative references
targeting the supplied API name or address. This helps locate code that
references an imported function or a specific address.

Examples: rex [0x140001000]> xref GetModuleHandle rex [0x140001000]>
xref 140002040

6. DEPS

Usage: deps 4

What it does: Displays the binary’s dynamic libraries, imported APIs,
and exported symbols.

Example: rex [0x140001000]> deps

7. STRINGS

Usage: strings [filter] 5

What it does: Displays extracted strings from the binary. An optional
filter limits the displayed results to strings containing the supplied
keyword.

Examples: rex [0x140001000]> strings rex [0x140001000]> strings http

8. CARVE

Usage: carve 6

What it does: Lists embedded files detected by the binary carving
module, along with their detected types and offsets.

Example: rex [0x140001000]> carve

9. FINDINGS

Usage: findings 7

What it does: Displays conclusions produced by the cross-module
correlation engine. Findings are labelled FACT, PROBABLE, or SPECULATIVE
according to the confidence assigned to each conclusion.

Example: rex [0x140001000]> findings

10. FIND

Usage: find search

What it does: Searches across multiple types of investigation results,
including imported APIs, extracted strings, and correlated findings.

Examples: rex [0x140001000]> find CreateFile rex [0x140001000]> find
https

11. MARK

Usage: mark
bookmark
What it does: Saves a virtual address as a bookmark with a custom text
note. Bookmarks let you keep track of addresses you may want to revisit
during the current explorer session.

Example: rex [0x140001000]> mark 140002040 Possible initialization
routine

12. MARKS

Usage: marks bookmarks

What it does: Lists the bookmarks and notes created during the current
explorer session.

Example: rex [0x140001000]> marks

13. HELP

Usage: help

What it does: Displays the interactive explorer command reference.

Example: rex [0x140001000]> help

14. QUIT

Usage: quit q exit

What it does: Closes the interactive explorer session and returns to the
terminal.

Example: rex [0x140001000]> quit

QUICK REFERENCE

CLI COMMANDS

    rex explore <path>                  Open interactive explorer
    rex investigate <path>              Run a full investigation
    rex binary <path>                   Inspect binary structure
    rex fingerprint <path>              Identify compiler/toolchain clues
    rex deps <path>                     Inspect dependencies and symbols
    rex disasm <path> [--count N]       Disassemble from entry point
    rex strings <path> [--min N]        Extract strings
    rex diff <first> <second>           Compare two binaries
    rex memory <path>                  Analyze a memory dump
    rex pcap <path>                    Parse a PCAP capture
    rex protocol <path>                Inspect possible protocol structure
    rex carve <path>                   Find embedded file signatures
    rex scan <directory>               Scan immediate directory contents
    rex search <path> <query>           Search raw file bytes
    rex history                         View investigation history

INTERACTIVE EXPLORER COMMANDS

    overview                            Show binary overview
    sections                            List binary sections
    disasm [N]                          Disassemble at current address
    jump <hex>                          Navigate to an address
    xref <term>                         Find references to an API or address
    deps                                View dependencies and symbols
    strings [filter]                    Browse extracted strings
    carve                               List embedded files
    findings                            View correlated findings
    find <query>                        Search analysis results
    mark <hex> <note>                   Create an address bookmark
    marks                               List session bookmarks
    help                                Show explorer help
    quit                                Exit the explorer

NOTES

-   REX is an analysis toolkit. Several outputs, including entropy-based
    packing classifications, memory-region classifications, and protocol
    structure hypotheses, are heuristic and should be interpreted as
    indicators rather than definitive conclusions.
-   PCAP parsing and PCAPNG parsing are not interchangeable. The pcap
    command currently describes legacy PCAP capture parsing.
-   Directory scanning currently covers files directly inside the
    supplied directory, not nested subdirectories.
-   Markdown, HTML, and CSV output formats are implemented for full
    investigations. Other commands may use plain-text output when a
    requested format is not supported.
