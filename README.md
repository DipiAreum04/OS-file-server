# File Sharing Server

A multithreaded file sharing server with a simulated filesystem, built in Java as part of a group project for the *COEN 346 – Operating Systems* course at Concordia University.

> **Note:** The `FileClient` module and the base project structure were provided by the course instructor. Our implementation work is entirely within the `FileServer` module.

## Overview

The server simulates a disk-backed filesystem stored in a single binary file. Multiple clients connect over TCP and perform file operations concurrently. The server handles thousands of simultaneous connections using Java virtual threads, with a `ReentrantReadWriteLock` ensuring safe concurrent access.

## Key Concepts

- **Simulated filesystem** — A single backing file is divided into an FEntry table (directory), an FNode table (linked block metadata), and fixed-size data blocks
- **Virtual threads** — Each client connection runs in its own virtual thread (`Executors.newVirtualThreadPerTaskExecutor()`), making the server efficient for high-concurrency, I/O-bound workloads
- **Read/write locking** — Multiple clients can read concurrently; writes are exclusive. A fair `ReentrantReadWriteLock` is used to prevent starvation
- **Atomic writes** — `writeFile` allocates a new block chain before updating the FEntry pointer, so a partial write never corrupts the existing file

## Project Structure

```
FileServer/
└── src/main/java/ca/concordia/
    ├── Main.java
    ├── server/
    │   ├── FileServer.java          # Accepts connections, spawns virtual threads
    │   └── ClientHandler.java       # Parses and dispatches client commands
    └── filesystem/
        ├── FileSystemManager.java   # Core filesystem logic (singleton)
        └── datastructures/
            ├── FEntry.java          # Directory entry (name, size, firstBlock)
            └── FNode.java           # Block node (blockIndex, nextBlock)

FileClient/
└── src/main/java/ca/concordia/
    └── Main.java                    # Provided by instructor
```

## Supported Commands

| Command | Description |
|---|---|
| `CREATE <filename>` | Creates an empty file (max 11-char name) |
| `WRITE <filename> <content>` | Overwrites file contents |
| `READ <filename>` | Returns file contents |
| `DELETE <filename>` | Deletes the file and zeroes its blocks |
| `LIST` | Lists all files on the server |
| `QUIT` | Closes the client connection |

## Build & Run

Requires Java 21+ and Maven.

**Server:**
```bash
cd FileServer
mvn compile
java -cp target/classes ca.concordia.Main <port> <filesystem_name> <total_size_bytes>
# Example:
java -cp target/classes ca.concordia.Main 8080 myfs 1048576
```

**Client:**
```bash
cd FileClient
mvn compile
java -cp target/classes ca.concordia.Main <host> <port>
# Example:
java -cp target/classes ca.concordia.Main localhost 8080
```

## Authors

- Dipita Sinha (40273009)
- Arnav Singh Ahlawat (40258921)
