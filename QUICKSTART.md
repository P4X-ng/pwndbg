# Pwndbg Quickstart Guide

## What is pwndbg?

**pwndbg** (/paʊnˈdiˌbʌɡ/) is a GDB and LLDB plugin that transforms these debuggers into powerful tools for:
- **Reverse Engineering** - Understanding and analyzing binaries
- **Exploit Development** - Finding and exploiting vulnerabilities
- **Binary Analysis** - Inspecting program behavior and memory
- **CTF Competitions** - Rapid binary challenge solving
- **Vulnerability Research** - Discovering security issues

Unlike vanilla GDB/LLDB, pwndbg provides an intuitive, enhanced debugging experience with color-coded memory displays, automatic context updates, heap inspection, and extensive helper commands.

## When to Use pwndbg vs Other Tools

| Task | Best Tool | Why |
|------|-----------|-----|
| **Dynamic binary analysis** | pwndbg | Real-time memory inspection, register tracking |
| **Heap exploitation** | pwndbg | Built-in heap analysis (glibc, jemalloc, SLUB) |
| **Static analysis** | IDA/Ghidra/Binary Ninja | Full program overview, decompilation |
| **Quick disassembly** | pwndbg | Context section shows surrounding code automatically |
| **Linux kernel debugging** | pwndbg + GDB | Kernel-specific commands (kbase, vmmap, slab) |
| **macOS debugging** | pwndbg + LLDB | LLDB support with same pwndbg commands |
| **ROP chain building** | pwndbg | Find gadgets in loaded memory with `rop` command |
| **Memory corruption** | pwndbg | Real-time heap/stack inspection |

### Integration in RE Workflow

```
┌─────────────┐
│   Binary    │
└──────┬──────┘
       │
       ├──► Static Analysis (IDA/Ghidra) ──► Understand structure
       │
       ├──► pwndbg ──┬──► Dynamic analysis
       │             ├──► Memory inspection
       │             ├──► Exploit development
       │             └──► Vulnerability validation
       │
       └──► Decompiler Integration ──► Synchronized debugging
```

**pwndbg fits perfectly when you need to:**
- Understand what a binary does at runtime
- Inspect memory layouts and data structures
- Validate hypotheses from static analysis
- Develop exploits with immediate feedback
- Analyze heap allocations and corruption

## Core Functionality Overview

### 🎯 Context Display (Auto-shown on every break)

Every time execution stops, pwndbg automatically displays:

```
┌─[ REGISTERS ]──────────────────────────────────
│ RAX  0x0              RBX  0x7ffff7dd0000
│ RCX  0x7ffff7e1e000   RDX  0x0
├─[ DISASM ]────────────────────────────────────
│ => 0x400567 <main+4>    mov    edi, eax
│    0x400569 <main+6>    call   0x400440
├─[ STACK ]─────────────────────────────────────
│ 00:0000│ rsp  0x7fffffffe420 —▸ 0x400580
│ 01:0008│      0x7fffffffe428 —▸ 0x7ffff7a05b97
└────────────────────────────────────────────────
```

**Key Commands:**
- `context` - Show context sections manually
- `contextwatch <expr>` - Add custom expressions to watch
- `contextoutput <section> <tty>` - Split context across terminals
- `theme` - Customize colors and appearance

### 🧠 Memory Inspection

**View and search memory:**
- `vmmap` - Show memory mappings (like `/proc/self/maps`)
- `vmmap <addr>` - Find which mapping contains an address
- `telescope <addr>` - Recursively dereference pointers (show what points where)
- `hexdump <addr>` - Clean hexdump of memory
- `search <pattern>` - Search for bytes, strings, or values in memory
- `xinfo <addr>` - Get detailed info about an address (offsets from various regions)

**Examples:**
```gdb
pwndbg> vmmap libc
    0x7ffff7a0e000     0x7ffff7bd0000 r-xp   1c2000 0      /lib/x86_64-linux-gnu/libc-2.27.so
    
pwndbg> telescope $rsp 10
00:0000│ rsp  0x7fffffffe420 —▸ 0x400580 ◂— add    byte ptr [rax], al
01:0008│      0x7fffffffe428 —▸ 0x7ffff7a05b97 (__libc_start_main+231)

pwndbg> search -t string "password"
Searching for 'password' in: /home/user/binary
0x601060 "password123"
```

### 🏗️ Heap Analysis

pwndbg excels at heap exploitation with support for multiple allocators:

**glibc ptmalloc2:**
- `heap` - Walk the heap and show all chunks
- `bins` - Display all bins (tcache, fastbins, unsorted, small, large)
- `fastbins` / `tcachebins` - Show specific bin contents
- `malloc-chunk <addr>` - Parse and display a chunk at address
- `find-fake-fast <addr>` - Find overlapping fake chunks
- `try-free <addr>` - Simulate what free() would do (detect corruption)
- `vis-heap-chunks` - Visual representation of heap chunks
- `arena` / `arenas` - Examine arena structures

**Other allocators:**
- `jemalloc-*` commands for jemalloc
- `mallocng-*` commands for musl's mallocng
- `slab` - Linux kernel SLUB allocator
- `buddydump` - Linux kernel buddy allocator

**Tracking:**
- `track-heap` - Enable real-time heap allocation tracking
- `track-got` - Track GOT/PLT updates

**Example:**
```gdb
pwndbg> heap
Allocated chunk | PREV_INUSE
Addr: 0x602000
Size: 0x21

pwndbg> bins
tcachebins
0x20 [  7]: 0x602000 —▸ 0x602020 —▸ 0x602040
```

### 📊 Process & System Info

- `procinfo` - Complete process information (UIDs, GIDs, file descriptors, SELinux)
- `vmmap` - Virtual memory mappings
- `checksec` - Binary security features (NX, PIE, ASLR, Canary, etc.)
- `elfsections` - ELF section mappings
- `got` / `plt` - Show GOT/PLT entries
- `pid` - Get process ID
- `threads` - List all threads

### 🔍 Disassembly & Emulation

- `nearpc` - Disassemble near program counter (done automatically in context)
- `emulate` - Emulate forward execution to show likely path
- `asm <instructions>` - Assemble instructions to shellcode
- `decomp` - Show decompiled code (requires integration with IDA/Binja/Ghidra)

### 🎯 Breakpoints & Flow Control

**Smart breakpoints:**
- `break-if-taken` / `break-if-not-taken` - Conditional branch breakpoints
- `breakrva <offset>` - Break at RVA from PIE base

**Enhanced stepping:**
- `nextcall` - Break at next call instruction
- `nextret` - Break at next return
- `nextjmp` - Break at next jump
- `nextsyscall` - Break at next syscall
- `stepover` - Step over current instruction
- `stepuntilasm <instruction>` - Step until matching instruction

### 🧬 Exploitation Helpers

**Stack:**
- `canary` - Show current stack canary value
- `retaddr` - Show all return addresses on stack
- `stack` / `stackf` - Dereference stack with context

**ROP Gadgets:**
- `rop --grep "<pattern>"` - Find ROP gadgets (uses ROPgadget)
- `ropper` - Alternative gadget finder (uses ropper)

**Patterns:**
- `cyclic <size>` - Generate De Bruijn pattern for offset finding
- `cyclic -l <value>` - Find offset of pattern value

**Leak Finding:**
- `leakfind <addr>` - Find pointer chains to address
- `probeleak <addr>` - Scan memory for pointer leaks
- `p2p <mapping1> <mapping2>` - Find pointers from one mapping to another

### 🔐 Kernel Debugging

pwndbg includes extensive kernel debugging support:

**Basic:**
- `kbase` - Find kernel base address
- `kconfig` - Show kernel configuration
- `kchecksec` - Check kernel security features
- `kversion` / `kcmdline` - Kernel version and boot parameters
- `kdmesg` - Show kernel ring buffer

**Memory:**
- `pagewalk <addr>` - Walk page tables
- `v2p <vaddr>` - Virtual to physical address translation
- `p2v <paddr>` - Physical to virtual address translation
- `slab` - Inspect SLUB allocator

**Subsystems:**
- `kmem-trace` - Trace kernel memory allocations
- `kbpf` - Inspect BPF programs and maps
- `knft-*` - Netfilter/nftables inspection
- `ktask` - Show kernel tasks
- `kfile` - Show file descriptors

### 🍎 macOS/Darwin Support

- `commpage` - Dump macOS commpage values

### 🪟 WinDbg Compatibility

For Windows reverse engineers, pwndbg provides WinDbg-style commands:

**Memory:**
- `db/dw/dd/dq <addr>` - Display bytes/words/dwords/qwords
- `eb/ew/ed/eq <addr> <value>` - Edit memory
- `da/ds <addr>` - Display ASCII/string
- `dds/dps <addr>` - Display pointers with symbols

**Breakpoints:**
- `bp <addr>` - Set breakpoint
- `bl` - List breakpoints
- `bc/bd/be <id>` - Clear/disable/enable breakpoint

**Other:**
- `k` - Stack trace (backtrace)
- `ln <addr>` - List nearest symbols
- `go` - Continue (continue execution)

### 🤖 AI & Integration

- `ai <question>` - Ask GPT about current debugging context
- `bn-sync` / `j` - Sync with Binary Ninja / IDA
- `r2` / `rz` - Launch radare2 / rizin integration
- `r2pipe` / `rzpipe` - Execute r2/rizin commands

### ⚙️ Configuration

- `config` - Show all pwndbg configuration options
- `theme` - Show theme settings
- `set <option> <value>` - Change settings
- `configfile` / `themefile` - Generate config files

## Quick Command Reference by Use Case

### Starting a Debug Session

```gdb
# Start debugging
$ gdb ./binary
pwndbg> start                  # Run and break at convenient location
pwndbg> entry                  # Run and break at entry point
pwndbg> attachp <name>         # Attach to process by name

# Or with arguments
pwndbg> set args arg1 arg2
pwndbg> start
```

### Understanding Program State

```gdb
pwndbg> context                # Show full context
pwndbg> vmmap                  # Memory mappings
pwndbg> checksec               # Security features
pwndbg> procinfo               # Process details
pwndbg> info registers         # or just 'i r'
pwndbg> backtrace              # or 'bt'
```

### Analyzing Memory

```gdb
pwndbg> telescope $rsp 20      # Follow pointers from stack
pwndbg> hexdump $rip 0x100     # Dump code
pwndbg> xinfo $rax             # What is this address?
pwndbg> vmmap $rax             # Which mapping?
pwndbg> search -s "flag{"      # Find strings
```

### Heap Debugging

```gdb
pwndbg> heap                   # Walk heap
pwndbg> bins                   # Show all bins
pwndbg> malloc-chunk $rax      # Parse chunk
pwndbg> try-free $rax          # Test free()
pwndbg> vis-heap-chunks        # Visual heap
pwndbg> track-heap on          # Enable tracking
```

### Finding Exploitable Conditions

```gdb
pwndbg> canary                 # Stack canary value
pwndbg> got                    # GOT entries (for overwrites)
pwndbg> retaddr                # Return addresses
pwndbg> rop --grep "pop rdi"   # Find gadgets
pwndbg> leakfind $rax          # Find leak chains
```

### Developing Exploits

```gdb
pwndbg> cyclic 200             # Generate pattern
pwndbg> cyclic -l 0x61616171   # Find offset
pwndbg> distance $rsp $rbp     # Calculate distances
pwndbg> asm "xor eax, eax; ret" # Generate shellcode
```

## Essential Tips

### 1. Use Context Splitting for Better Layout

```gdb
pwndbg> contextoutput stack /dev/pts/1
pwndbg> contextoutput regs /dev/pts/2
pwndbg> contextoutput disasm terminal
```

Run `tty` in separate terminals to get their device paths.

### 2. Save Common Settings

Create `~/.gdbinit` or `.gdbinit` in project directory:

```python
# Auto-context sections
set context-sections "regs disasm code stack backtrace expressions"

# Increase telescope depth
set telescope-lines 20

# Enable heap tracking by default
track-heap on
```

### 3. Integration with Decompilers

pwndbg can sync with IDA Pro, Binary Ninja, and Ghidra:

```gdb
pwndbg> j                      # Sync with IDA (cursor follows execution)
pwndbg> bn-sync                # Sync with Binary Ninja
pwndbg> decomp                 # Show decompiled code in pwndbg
```

### 4. Quick Command Discovery

```gdb
pwndbg> pwndbg                 # List all pwndbg commands
pwndbg> help <command>         # Get command help
pwndbg> apropos <keyword>      # Search for commands
```

### 5. Keyboard Shortcuts

- `Ctrl+C` - Interrupt execution
- `Enter` - Repeat last command
- `Ctrl+R` - Search command history
- `Tab` - Auto-complete

### 6. Common Pitfalls

**Problem:** Context doesn't update
- **Solution:** Check if context is enabled: `context` or restart pwndbg

**Problem:** Commands not found
- **Solution:** Ensure pwndbg loaded correctly. Look for pwndbg banner at GDB start

**Problem:** Symbols not loading
- **Solution:** Install debug symbols: `apt install libc6-dbg` or `debuginfod`

**Problem:** Can't attach to process
- **Solution:** `echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope`

## Where pwndbg Fits in Your Toolchain

### Complete RE/Exploit Dev Workflow

1. **Static Analysis** (IDA/Ghidra/Binary Ninja)
   - Identify functions of interest
   - Understand control flow
   - Find potential vulnerabilities

2. **Dynamic Analysis** (pwndbg)
   - Validate static analysis findings
   - Inspect actual memory layouts
   - Test vulnerability hypotheses
   - Develop exploitation primitives

3. **Exploit Development** (pwndbg + Python/pwntools)
   - Find gadgets: `rop --grep "pop rdi"`
   - Calculate offsets: `cyclic` and `distance`
   - Test payload: Run in pwndbg, iterate
   - Validate exploitation: `track-heap`, `try-free`

4. **Finalize Exploit** (Python/pwntools)
   - Script final exploit
   - Test against pwndbg for verification
   - Deploy

### Tool Synergy

| Tool | Purpose | Synergy with pwndbg |
|------|---------|---------------------|
| **IDA Pro** | Static analysis | Sync cursor, share symbols |
| **Binary Ninja** | Static analysis | Sync cursor, decompile integration |
| **Ghidra** | Static analysis | Decompiler integration via r2/rizin |
| **ROPgadget** | Gadget finding | Built-in via `rop` command |
| **pwntools** | Exploit scripting | Use together for validation |
| **radare2/rizin** | Swiss army knife | Direct integration with `r2pipe` |

## Advanced Features

### Custom Expressions

```gdb
pwndbg> contextwatch execute "x/10gx $rsp"
pwndbg> contextwatch execute "p/x $rax ^ $rbx"
```

### Memory Patching

```gdb
pwndbg> set {int}0x400500 = 0x90909090    # NOP
pwndbg> patch 0x400500 "nop\nnop\nnop\nnop"
pwndbg> eb $rip 90 90 90 90                # WinDbg style
```

### Custom Types

```gdb
pwndbg> dt struct_name            # Display type
pwndbg> cymbol add my_struct      # Add custom structure
```

### Go Debugging

```gdb
pwndbg> go-dump <addr> <type>     # Dump Go values
pwndbg> go-type <addr>            # Show Go type
```

## Learning Resources

- **Full Documentation:** https://pwndbg.re/
- **Command Reference:** https://pwndbg.re/dev/commands/
- **Cheatsheet PDF:** https://pwndbg.re/dev/CHEATSHEET.pdf
- **GitHub:** https://github.com/pwndbg/pwndbg
- **Discord:** https://discord.gg/x47DssnGwm

## Getting Help

```gdb
pwndbg> help <command>           # Command-specific help
pwndbg> pwndbg                   # List all commands
pwndbg> apropos <keyword>        # Search commands
pwndbg> bugreport                # Generate bug report
```

For questions or issues, visit the [Discord server](https://discord.gg/x47DssnGwm) or [GitHub Issues](https://github.com/pwndbg/pwndbg/issues).

---

**Pro Tip:** Print the [cheatsheet](https://pwndbg.re/dev/CHEATSHEET.pdf) and keep it handy for quick reference during CTFs or security assessments!
