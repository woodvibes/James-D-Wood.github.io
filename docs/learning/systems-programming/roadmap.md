---
sidebar_position: 1
title: Roadmap
---

# Systems Programming Roadmap

**Goal:** go from backend development with no C or OS background to having
finished a teaching kernel, with optional extensions that build a TCP/IP stack
and a crash-safe storage engine.

| Phase | Topic | Ends in |
|---|---|---|
| 0 | C | A `grep` with a tiny regex engine |
| 1a | The machine | A shell and your own `malloc` |
| 1b | System I/O and network programming | An HTTP server |
| 2 | Operating systems (xv6 + OSTEP) | A finished kernel |
| Extension A | Networking from the bottom | Your own TCP/IP stack |
| Extension B | Storage engines | A crash-safe key-value store |

**Ordering:** Phases 0–2 are the core path. The extensions build on Phase 2 and
are independent of each other — take either, both, or neither, in any order.
The kernel work (paging, the page cache, block layers, `fsync` semantics,
scheduling, locking) underpins a storage engine, so it comes first. Networking
is split in two: sockets and HTTP in Phase 1b, protocol internals in Extension A
once the xv6 network driver lab has shown what a NIC is.

---

## Phase 0 — C

**Why:** everything later is written in C. The goal is the memory model —
memory as a flat addressable array, manual ownership, undefined behaviour — not
just the syntax.

### Setup
- [ ] **A Linux environment.** The material, syscalls, tooling (Valgrind, gdb),
  and kernel work all assume Linux. Make it **x86-64**: the CSAPP Bomb and
  Attack Lab binaries in Phase 1a are x86-64, while UTM, Lima, and OrbStack
  give you arm64 Linux by default on Apple Silicon. An x86-64 VM (e.g. on a
  Proxmox host) or a VPS works.

### Read
- [ ] ***Effective C* (Seacord)** — primary text. Modern C (C17/C23) with a focus
  on what the standard guarantees, where undefined behaviour lurks, and which
  idioms are portable.
- [ ] ***21st Century C* (Klemens), Part I** — the C tooling ecosystem: compilers,
  debuggers, `make`, sanitizers, libraries. Part II covers modern idioms once you
  can write C. Skip the autotools material.
- [ ] ***The C Programming Language* (K&R), chapter 5** — pointers and arrays,
  with well-designed exercises.
- [ ] **Beej's *Guide to C Programming*** — quick reference.
- [ ] ***Modern C* (Gustedt, free PDF)** — optional reference.
- [ ] ***Tiny C Projects* (Gookin)** — optional extra practice: small
  command-line utilities. Build them with the flags and sanitizers below so they
  exercise the memory model, not just the syntax.

### Build
- [ ] A dynamic array (`push`, `pop`, growth policy) with `malloc`/`realloc`/`free`
- [ ] A hash table — open addressing and chaining
- [ ] `wc`
- [ ] **Milestone: `grep`** — fixed substring first, then a tiny regex engine
  (`.`, `*`, `^`, `$`).

### Tooling
- [ ] Compile with `-Wall -Wextra -Werror`
- [ ] Test under `-fsanitize=address,undefined`
- [ ] `gdb` / `lldb`: breakpoints, `bt`, `p`, watchpoints, examining raw memory
- [ ] `valgrind`
- [ ] Hand-written `Makefile`s

### Learning objectives
- Read a double-pointer signature and know what it expects.
- Know who owns and frees every allocation.
- Debug a heap corruption and a use-after-free.

---

## Phase 1a — The machine

**Why:** the concepts here — data representation, assembly, the memory
hierarchy, linking, processes, virtual memory — are the foundation for every
later phase.

### Read
- [ ] ***Computer Systems: A Programmer's Perspective* (CSAPP)** — read the whole
  book. Chapters 1–9 here: data representation, x86-64 assembly, caches,
  linking, processes, virtual memory. Chapters 10–12 are covered in Phase 1b.
- [ ] ***System Programming in Linux* (Weiss)** — the Linux system-call API,
  taught by reimplementing Unix commands. Read the process, signal, and pipe
  material alongside CSAPP ch. 8 and the shell build.

### CSAPP labs
- [ ] Data Lab — bit manipulation
- [ ] Bomb Lab — reading x86-64 assembly in a debugger
- [ ] Attack Lab — buffer overflows and ROP
- [ ] Cache Lab — how memory layout affects performance

### Build
- [ ] **A shell** (CSAPP Shell Lab) — `fork`/`exec`, pipes, redirection, job
  control, signal handling.
- [ ] **Milestone: your own allocator** (CSAPP Malloc Lab; read §9.9 first) —
  `malloc`/`free` on top of `mmap`, with free lists, block splitting, and
  coalescing.

### Tooling
- [ ] `strace -f` on your shell — watch `fork`, `execve`, `dup2`, `pipe`, and
  `wait4`, and compare against `bash -c 'ls | wc'`
- [ ] `ltrace` on a real program — see its `malloc`/`free` pattern before
  choosing a free-list design
- [ ] `/proc/<pid>/maps`, `/proc/<pid>/status`, and `pmap` — find your
  allocator's `mmap`'d regions; RSS versus virtual size
- [ ] `perf stat -e cache-misses,cache-references` on your Cache Lab transpose —
  the same misses the lab simulates, counted on real hardware

### Learning objectives
- Explain why a page fault is not an error.
- Describe what a context switch costs and why.
- Explain how your allocator trades fragmentation against throughput.

---

## Phase 1b — System I/O and network programming

**Why:** Unix I/O, sockets, and concurrency are where the machine model meets
real servers — and the designs here come back in both extensions.

### Read
- [ ] **CSAPP, chapters 10–12** — system-level I/O, network programming, and
  concurrent programming.
- [ ] ***System Programming in Linux* (Weiss)**, continued — the file I/O,
  directory, and terminal material, alongside CSAPP ch. 10.
- [ ] **Beej's *Guide to Network Programming*** — practical sockets programming.
- [ ] ***Unix Network Programming* (Stevens)** — reference for sockets and I/O models.
- [ ] **RFC 9112** — HTTP/1.1 message syntax, for the server below.
- [ ] **Kegel's *The C10K Problem*** — why I/O models matter at scale; read after
  the TCP server.

### Build
- [ ] **A thread pool** — mutexes, condition variables, a work queue. Find a
  deliberate data race with ThreadSanitizer.
- [ ] **A TCP server, three ways** — thread-per-connection, `select`/`poll`, then
  `epoll`. Benchmark them with `wrk` or `hey`.
- [ ] **Milestone: an HTTP/1.1 server** — request parsing, `Content-Length` and
  chunked encoding, keep-alive, timeouts, graceful shutdown, static files via
  `sendfile`. Load-test with `h2load` and fuzz it.
- [ ] Optional: add `io_uring` as a fourth I/O model and compare it with `epoll`.

### Tooling
- [ ] `perf record` and flame graphs on each TCP server under load — explain
  the benchmark results, not just record them
- [ ] `strace -c` or `perf trace` — syscalls per request, and why `epoll` and
  `sendfile` win
- [ ] `ss` and `/proc/net` — connection states, the accept backlog, and
  `TIME_WAIT` build-up under load
- [ ] `tcpdump` / Wireshark — capture your server's keep-alive and chunked
  responses on the wire

### Learning objectives
- Explain `accept`, `EAGAIN`, and edge-triggered `epoll`, and how non-blocking
  sockets change a program's structure.
- Explain the difference between Unix I/O and standard I/O buffering, and when
  each is safe to use on a socket.

---

## Phase 2 — Operating systems (xv6 + OSTEP)

**Why:** writing kernel code turns paging, traps, scheduling, locking, and file
systems from concepts into things you have implemented.

### Resources
- [ ] **MIT 6.1810 (formerly 6.828)** — free lectures, labs, and the xv6 book.
  xv6 is a small Unix-like kernel in C on RISC-V, run under QEMU. The labs are
  tested and build on each other. The lab list changes between semesters, so
  pick one semester's offering and follow it throughout.
- [ ] ***Operating Systems: Three Easy Pieces* (OSTEP, free)** — the companion
  text. Read the matching part alongside the labs:
  - Virtualization — with system calls, page tables, traps, copy-on-write fork
  - Concurrency — with multithreading and locks
  - Persistence — with the file system and mmap labs

### Labs
- [ ] Utilities — toolchain and QEMU setup
- [ ] System calls — add a syscall end to end
- [ ] Page tables
- [ ] Traps — interrupts, exceptions, the trap frame
- [ ] Copy-on-write fork
- [ ] Multithreading — user-level threads, a parallel hash table
- [ ] Network driver
- [ ] Locks — reducing contention
- [ ] File system — large files, symlinks
- [ ] mmap

### Skills
- [ ] Debugging a kernel with QEMU's gdb stub (`-S -gdb tcp::1234`)
- [ ] Reading a linker script and the memory map it produces
- [ ] Reading RISC-V assembly well enough to follow a trap path
- [ ] Optional: `strace` a Linux process doing what a lab implements (e.g.
  `mmap`) and compare the two kernels

### Milestone
- [ ] **Read all of xv6 end to end** after finishing the labs.

### Optional
- [ ] Extend xv6: a scheduler policy (lottery, CFS-like, priority), a larger
  buffer cache, or a simple network stack.

### Learning objectives
- Explain what happens on a page fault, from instruction to resumption.
- Explain what xv6's block cache and logging layer do and where they live.
- Trace what happens between `write()` returning and bytes reaching disk.

---

## Extension A — Networking from the bottom

**Why:** this connects the NIC from the xv6 driver lab (a ring buffer plus an
interrupt) to the sockets API from Phase 1b — everything between bytes on a wire
and `read()` returning.

### Read
- [ ] ***Computer Networks: A Systems Approach* (Peterson & Davie, free)** —
  networking theory, layer by layer.
- [ ] **RFC 9293** — the TCP specification and state machine.
- [ ] ***High Performance Browser Networking* (Grigorik, free)** — how TCP, TLS,
  and HTTP behave in practice and why they perform the way they do.

### Build
- [ ] **Draw the TCP state machine from the RFC by hand.**
- [ ] **A user-space TCP/IP stack over a TAP device**, in stages. TAP delivers
  Ethernet frames; a TUN device would deliver bare IP packets and skip the
  first stage.
  - [ ] Ethernet framing + ARP
  - [ ] IPv4: headers, checksums, fragmentation
  - [ ] ICMP — `ping` your own stack
  - [ ] TCP: handshake, sequence numbers, teardown
  - [ ] TCP: retransmission, RTO estimation, flow control (the receive window)
  - [ ] TCP: congestion control — slow start and one algorithm of your choice
  - [ ] A socket API on top, enough to run the Phase 1b HTTP server over it

  References: `smoltcp` (Rust) and the "Let's code a TCP/IP stack" series.
- [ ] **TLS with OpenSSL or BoringSSL** — use a library rather than implementing
  it; learn the handshake, certificate verification, and ALPN.
- [ ] **An HTTP/2 or QUIC client** — framing, multiplexing, stream-level flow control.

### Read code
- [ ] **nginx or haproxy** — event loop and connection lifecycle
- [ ] **lwIP** — a small, complete, production TCP/IP stack

### Tooling
- [ ] Wireshark on `tap0` from the ARP stage onward — the primary debugger for
  your stack
- [ ] `ss -ti` — compare the kernel's RTT, congestion window, and retransmits
  with your stack's during the RTO and congestion-control stages

### Learning objectives
- Explain what happens when an ACK is dropped.
- Explain why Nagle's algorithm and delayed ACK interact badly.
- Explain what `TIME_WAIT` is for and its effect at scale.
- Read a `tcpdump` capture.

---

## Extension B — Storage engines

**Why:** a storage engine puts the earlier phases to work — the page cache,
`fsync`, and locking from the kernel, plus crash consistency and concurrency,
which are the hard parts of building one correctly.

### Read
- [ ] ***Database Internals* (Petrov)** — storage engine design: B-trees, LSM-trees,
  logging, recovery, and the distributed side.
- [ ] **CMU 15-445 (Pavlo)** — public lectures and assignments on database
  internals: buffer pools, indexes, concurrency control, recovery.
- [ ] ***Designing Data-Intensive Applications* (Kleppmann), "Storage and
  Retrieval" chapter** — an overview of storage and retrieval structures.
- [ ] **The LSM-tree paper (O'Neil et al.) and the original B-tree material** —
  after building both.

### Build — one engine, in stages
- [ ] **Stage 1: append-only log** — put/get by scanning.
- [ ] **Stage 2: in-memory index** over the log (Bitcask-style), with compaction.
- [ ] **Stage 3: crash-safe log** — per-record checksums, torn-write detection,
  an explicit `fsync` policy, and recovery by truncating at the first bad
  record. Here the log *is* the data, so no separate WAL is needed yet.
- [ ] **Stage 4: on-disk structure**, one path:
  - **B+tree:**
    - a buffer pool first — page replacement (clock or LRU-K), pinning, dirty
      page tracking. Compare with `mmap`, and see why LMDB chose it.
    - the tree on top — page splits, rebalancing, a free list
    - a write-ahead log for in-place page updates, with redo on recovery
  - **LSM-tree:**
    - a write-ahead log for the memtable
    - memtable and SSTables
    - levelled or tiered compaction, bloom filters
    - a block cache for SSTable reads
- [ ] **Stage 5: physical concurrency** — latch crabbing on the B+tree, or a
  concurrent memtable with background flush and compaction on the LSM path.
- [ ] **Stage 6: transactions** — ACID transactions, using MVCC to provide
  snapshot isolation. Then read about ARIES (most relevant to the B+tree path).
- [ ] **Stage 7: a network front end** — a wire protocol (RESP) served by the
  Phase 1b server design. Covers where `fsync` sits relative to client
  acknowledgement, what a client timeout means for durability, and pipelining
  without reordering writes.

### Verification
- [ ] Property-based testing against a reference implementation (a `HashMap`)
- [ ] Crash injection, at two levels. Verify invariants on recovery for both:
  - **Process crashes** — `kill -9` at random points. This tests recovery
    logic, but the page cache survives the kill, so it cannot catch a missing
    `fsync`.
  - **Power loss** — hard-reset a VM, or replay writes with `dm-log-writes`,
    `dm-flakey`, or LazyFS. This is what tests durability.
- [ ] A fuzzer over the operation stream
- [ ] **"Can Applications Recover from fsync Failures?", "All File Systems Are
  Not Created Equal" (the ALICE paper), and Dan Luu's "Files are hard"** —
  common crash-consistency mistakes in real systems.

### Read code
- [ ] **SQLite** — the btree and pager layers, which are well documented
- [ ] **LevelDB** — a small, canonical LSM implementation
- [ ] **BoltDB / LMDB** — the mmap-based B+tree approach

### Tooling
- [ ] `strace -e trace=fsync,fdatasync,pwrite64,rename` — check your Stage 3
  `fsync` policy, then compare against SQLite or LevelDB
- [ ] `iostat`, `/proc/meminfo` (`Dirty`, `Writeback`), and `vmtouch` — watch
  the page cache and writeback while the engine runs
- [ ] `perf` at Stage 5 — find latch contention and compaction stalls
- [ ] Optional: eBPF (`bpftrace`, `biolatency`, `ext4slower`) — block-layer
  latency from inside the kernel

### Learning objectives
- The engine survives a thousand random crashes — both `kill -9` and simulated
  power loss — with no lost acknowledged writes and no corruption.
- Explain and justify the engine's isolation level.
