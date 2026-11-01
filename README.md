# REX — Reverse Engineering Explorer

REX is an all-in-one, Rust-based command-line reverse-engineering toolkit for investigating executables, firmware, memory dumps, network captures, and unknown binary formats.

It brings several areas of reverse engineering into one environment, combining binary inspection, disassembly, compiler fingerprinting, memory analysis, protocol inspection, function discovery, and structural comparison. Rather than treating each result independently, REX correlates findings across its analysis modules to provide a broader picture of the file being investigated.

## Features

### Binary and Executable Analysis

REX supports PE, ELF, and Mach-O identification and parsing. It inspects executable sections, including their names, virtual addresses, sizes, and Shannon entropy. It also identifies architecture, endianness, entry points, and potentially unusual section characteristics through heuristic classification.

### Disassembly and Code Analysis

Using iced-x86, REX provides x86 and x64 disassembly with Intel syntax and exact instruction bytes. It analyzes branches, calls, opcode frequency, and estimated basic block boundaries.

REX includes an ASCII Control Flow Graph (CFG) generator that partitions instructions into basic blocks and renders their relationships as text-based flow diagrams. These diagrams represent conditional branches, fallthrough paths, jump edges, and function exits.

Its function discovery engine scans executable sections for indicators such as function prologues, frame anchors, alignment padding, and branch targets to discover potential unexported functions and estimate their boundaries. Discovered functions are assigned generated names and confidence classifications.

For PE files, REX can resolve Import Address Table entries into recognizable Windows API names directly within disassembly output. It also supports disassembling from arbitrary virtual addresses and tracing cross-references to locate instructions that reference imported APIs and functions.

REX recognizes various compiler and runtime instruction patterns, including stack allocation, shadow space, frame pointer setup, security cookie initialization, and alignment padding.

### Function-Level Intelligence

The `rex function` command combines information from multiple analysis modules into a unified report for an individual function.

Depending on the available binary information, reports can include:

- Function boundaries, estimated byte size, and instruction count.
- Basic block counts and branch information.
- Incoming callers and outgoing function calls.
- Resolved imported APIs and their behavioral categories.
- Read-only data strings referenced by instructions.
- Recognized compiler and runtime patterns.
- Confidence classifications.

This provides a more complete view of an individual function by connecting its callers, callees, control flow, references, and other available analysis results.

### Compiler and Toolchain Fingerprinting

REX identifies compiler and toolchain indicators associated with Rust, Go, GCC, Clang/LLVM, and MSVC. It examines PE Rich Headers, known packer signatures, and CodeView RSDS debug information, including embedded PDB paths.

These findings can help establish how a binary was built and identify information left behind during compilation.

### Dependencies and Symbol Analysis

REX inspects dynamic libraries, imported functions, and exported symbols. It provides import and export counts, library information, and export addresses and RVAs. Export tables can also be compared between binaries to identify ABI-level changes.

Its API Behavioral Classification Engine groups recognized imported APIs by operational category, including:

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

These classifications provide additional context about the functionality indicated by a binary's imported APIs. An API's presence alone does not establish how or why it is used.

### Memory Analysis

REX analyzes memory dump files in 64 KB chunks, scanning for candidate 64-bit virtual pointers and measuring pointer density across regions. It uses these measurements alongside entropy and sparse-data detection to classify potentially interesting memory regions.

### Network and Protocol Analysis

REX includes an offline PCAP capture parser supporting Ethernet, IPv4, IPv6, ARP, ICMP, TCP, and UDP. It identifies common traffic types such as HTTP, TLS, and DNS, and provides packet summaries and protocol distribution statistics.

Its protocol analysis module also examines unknown binary data for potential magic values, framing patterns, length-prefixed structures, and other indicators that may help identify undocumented formats.

### Strings and Data Extraction

REX extracts ASCII and UTF-16LE strings with their corresponding file offsets and lengths. It can identify URLs, filesystem paths, and other potentially interesting strings.

Its embedded file carver searches binary data for known signatures, including PNG, JPEG, GIF, PDF, ZIP/JAR, 7z, RAR, PE, and ELF formats. A separate search command allows raw binary data to be searched for specific terms, displaying matching offsets and surrounding hexadecimal context.

### Hexadecimal Inspection

REX includes an integrated hexadecimal viewer that displays raw file bytes alongside their ASCII representations.

The viewer supports offset-based inspection and configurable display lengths, making it easier to examine binary structures and compare raw data with other analysis results.

### Binary Comparison and Compatibility

REX compares binaries using file size, byte-level similarity calculations, and exact identity checks. Its structural comparison capabilities examine individual sections, identifying additions, removals, resizing, and other structural changes.

It also compares entry-point locations and exported ABI symbols, providing more context than a simple overall file similarity score.

### Unified Investigation and Evidence Chains

The `rex investigate` command coordinates multiple analysis modules through a single operation. It collects information about the binary, examines its structure, fingerprints compiler indicators, inspects dependencies, extracts strings, and performs disassembly analysis.

Findings from different modules are correlated to identify patterns that may be less obvious when examining individual results.

REX also generates Evidence Chains that connect technical indicators to the conclusions drawn by its analysis engine. These chains help explain which observations contributed to an inference and how different findings relate to one another.

For example, an inference may connect imported API indicators, relevant instruction references, and section entropy measurements. The evidence chain provides context for the conclusion rather than presenting it as an isolated alert.

REX uses three confidence classifications:

**FACT** represents information directly verified from the available binary data.

**PROBABLE** represents conclusions supported by multiple indicators.

**SPECULATIVE** represents observations based on limited evidence or individual heuristics.

Every investigation is also recorded in a local history ledger.

### Interactive Terminal Explorer

The `rex explore` command opens an interactive terminal environment for examining a binary without repeatedly running separate commands. Analysis is loaded and cached within the session, allowing navigation between different areas of the investigation.

The explorer supports section browsing, disassembly, dependencies, strings, carved files, findings, and unified searching.

Additional capabilities include:

- Discovered function browsing using `funcs`.
- Detailed function analysis using `func`.
- ASCII Control Flow Graph generation using `cfg`.
- Integrated hexadecimal and ASCII inspection using `hex`.
- Arbitrary-address navigation and API/function cross-reference lookup.
- Persistent address bookmarks with personal notes.

Bookmarks are automatically saved to `.rex_bookmarks.json` in the working directory and reloaded in future explorer sessions.

### Workspace and Reporting

REX includes directory scanning, automatic file identification, binary searching, and investigation history.

Full investigations support multiple output formats, including plain text, JSON, Markdown, HTML, and CSV.

## Design Philosophy

REX is built around bringing different reverse-engineering disciplines together rather than requiring separate tools for every stage of an investigation.

The goal is to make it easier to inspect binaries, understand their structure, discover functions, follow code references, identify build information, examine unknown data, and connect findings through one consistent command-line interface.

**REX — Reverse Engineering Explorer.**
