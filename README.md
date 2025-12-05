# FAT32 File System Utility

A user-space shell utility for interpreting and manipulating FAT32 file system images, built in C.

![C](https://img.shields.io/badge/C-00599C?style=flat&logo=c&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat&logo=linux&logoColor=black)
![License](https://img.shields.io/badge/License-MIT-green.svg)

## Features

- **File System Navigation** - Browse directories with `ls` and `cd` commands
- **File Operations** - Create, read, write, and delete files
- **Directory Management** - Create and remove directories with proper `.` and `..` entries
- **FAT Table Manipulation** - Direct cluster allocation and deallocation
- **Multi-file Support** - Track up to 10 simultaneously open files with independent offsets
- **File Seeking** - Random access read/write with `lseek` support

## Technical Highlights

- **Boot Sector Parsing** - Reads and interprets FAT32 BPB (BIOS Parameter Block)
- **Cluster Chain Traversal** - Navigates linked clusters for large files/directories
- **8.3 Filename Handling** - Converts between human-readable and DOS format names
- **Dual FAT Updates** - Maintains consistency across both FAT copies
- **Memory-Safe Design** - Proper allocation/deallocation with no memory leaks

## Architecture

```
src/
├── main.c        # Shell loop and command dispatcher
├── fat32.c       # Core FAT32 operations (mount, FAT, clusters)
├── commands.c    # Command implementations (ls, cd, read, write, etc.)
└── lexer.c       # Input tokenization

include/
├── fat32.h       # FAT32 structures and constants
├── commands.h    # Command function declarations
└── lexer.h       # Tokenizer interface
```

## Building

```bash
make            # Build the executable
make clean      # Remove build artifacts
```

## Usage

```bash
./bin/filesys <fat32_image>
```

### Example Session

```
fat32/> ls
DOCUMENTS
PHOTOS
README.TXT

fat32/> cd DOCUMENTS
fat32/DOCUMENTS> creat NOTES
fat32/DOCUMENTS> open NOTES -rw
fat32/DOCUMENTS> write NOTES "Hello, World!"
fat32/DOCUMENTS> lseek NOTES 0
fat32/DOCUMENTS> read NOTES 13
Hello, World!
fat32/DOCUMENTS> close NOTES
fat32/DOCUMENTS> exit
```

### Supported Commands

| Command | Description |
|---------|-------------|
| `info` | Display file system metadata |
| `ls` | List directory contents |
| `cd <dir>` | Change directory |
| `mkdir <dir>` | Create directory |
| `creat <file>` | Create empty file |
| `open <file> <mode>` | Open file (-r, -w, -rw) |
| `close <file>` | Close file |
| `read <file> <size>` | Read bytes from file |
| `write <file> "text"` | Write to file |
| `lseek <file> <offset>` | Set file offset |
| `lsof` | List open files |
| `mv <src> <dest>` | Move/rename |
| `rm <file>` | Delete file |
| `rmdir <dir>` | Remove empty directory |
| `exit` | Exit program |

## FAT32 Implementation Details

### Data Structures

The implementation handles key FAT32 structures:
- **Boot Sector** - Contains BPB with cluster size, FAT location, root directory cluster
- **File Allocation Table** - 32-bit entries mapping cluster chains
- **Directory Entries** - 32-byte structures with 8.3 names, attributes, cluster pointers

### Cluster Management

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Cluster 5  │────▶│  Cluster 8  │────▶│  Cluster 12 │────▶ EOC
│  (File data)│     │  (File data)│     │  (File data)│
└─────────────┘     └─────────────┘     └─────────────┘
       │                  │                   │
       ▼                  ▼                   ▼
   FAT[5] = 8        FAT[8] = 12       FAT[12] = 0x0FFFFFF8
```

## Testing

Tested on Linux (Ubuntu/Debian) with various FAT32 images. The utility correctly:
- Reads existing files and directories
- Creates new files with proper cluster allocation
- Extends files when writing beyond current size
- Reclaims clusters on file deletion
- Maintains directory structure integrity