# PCIe Transaction Layer Packets (TLP) Protocol Implementation

A low-level implementation of the PCIe Transaction Layer Packets (TLP) protocol in C. This project provides packet parsing, validation, and manipulation for the three primary TLP packet types used in PCIe communication between host and endpoint devices.

## Overview

PCIe (PCI Express) is the primary communication protocol used to connect peripheral devices (like NVMe drives) to computer systems. The Transaction Layer Packets (TLP) protocol is the packet-level interface that defines how data is transmitted between the Host and Endpoint devices. This project implements parsing and generation of TLP packets at the byte and bit level.

## PCIe/TLP Background

Transaction Layer Packets (TLP) form the communication protocol layer in PCIe:
- **Host**: Issues read/write requests and receives completions
- **Endpoint**: Responds to requests with data or status
- **Protocol**: Defines three main packet types for this assignment

The TLP protocol uses precise bit-level encoding with fields for addressing, data, control signals, and tags.

## Packet Types Implemented

### 1. Memory Write Request (32-bit addressing)
Sends data from Host to Endpoint memory.

**Structure**:
- Bits 31-30: Format (01 for 32-bit write)
- Bits 29-24: Command Type
- Bits 23-20: Traffic Class
- Bits 19-12: Byte Enable (which bytes are valid)
- Bits 11-0: Packet Length (in dwords)
- Address field (4 bytes)
- Data payload (variable length)
- Tag field (for tracking)

**Use Case**: Writing configuration data, uploading firmware, sending commands to devices.

### 2. Memory Read Request (32-bit addressing)
Requests data from Endpoint memory.

**Structure**:
- Bits 31-30: Format (00 for 32-bit read)
- Bits 29-24: Command Type
- Bits 23-20: Traffic Class
- Bits 19-12: Byte Enable
- Bits 11-0: Packet Length (always 1 dword for read requests)
- Address field (4 bytes)
- Tag field (for matching with completion)

**Use Case**: Reading device status, retrieving data from disk, querying device capabilities.

### 3. Completion with Data
Endpoint responds to read requests with data.

**Structure**:
- Bits 31-30: Format
- Bits 29-24: Command Type (Completion)
- Data payload from requested address
- Completion status
- Tag (matches corresponding read request)

**Use Case**: Returning data to host, acknowledging successful operations.

## Features

✅ Byte-level packet parsing  
✅ Bit-level field extraction and manipulation  
✅ Complete validation of all three packet types  
✅ Byte-enable field handling  
✅ Tag tracking for request/response matching  
✅ Address validation and translation  
✅ Packet encoding and decoding  
✅ Comprehensive error handling

## Building

### Prerequisites
- GCC or Clang compiler
- CMake 3.10+
- C11 standard library
- Math library (libm)

### Compilation
```bash
# Configure
cmake -S . -B build

# Build
cmake --build build

# Run tests
./build/hw2_main
```

## Project Structure

```
├── src/
│   ├── hw2.c              # Core packet parsing and manipulation
│   ├── hw2_main.c         # Main program and test interface
│   └── utility functions
├── include/
│   ├── hw2.h              # Function declarations
│   └── type definitions
├── test/
│   └── inputs/            # Test packet data
└── CMakeLists.txt        # Build configuration
```

## Core Functions

### Packet Parsing
- `parse_tlp_write()` - Parse Memory Write Request packet
- `parse_tlp_read()` - Parse Memory Read Request packet
- `parse_tlp_completion()` - Parse Completion with Data packet
- `validate_packet()` - Verify packet structure and fields

### Field Extraction
- `extract_field()` - Get bit range from packet
- `get_byte_enable()` - Extract byte enable bits
- `get_address()` - Extract 32-bit address
- `get_tag()` - Extract packet tag
- `get_length()` - Extract data length (in dwords)

### Packet Construction
- `create_write_packet()` - Build Memory Write Request
- `create_read_packet()` - Build Memory Read Request
- `create_completion_packet()` - Build Completion packet
- `encode_field()` - Set bit range in packet

## Bit-Level Operations

The implementation uses bitwise operations for field manipulation:

```c
// Extract bits [end:start]
uint32_t mask = (1 << (end - start + 1)) - 1;
uint32_t value = (packet >> start) & mask;

// Set bits [end:start]
uint32_t mask = (1 << (end - start + 1)) - 1;
packet = (packet & ~(mask << start)) | ((value & mask) << start);
```

## Byte-Enable Encoding

The byte-enable field indicates which bytes are valid:
- Bit set (1) = corresponding byte is valid
- Bit clear (0) = corresponding byte should be ignored

**Example**:
```
Byte Enable: 1100 (binary)
Enables:     Bytes 2 and 3
Disables:    Bytes 0 and 1
```

## Example Usage

### Parsing a Write Request
```c
uint8_t packet[64];  // Incoming TLP packet
// ... packet data loaded ...

uint32_t address = get_address(packet);
uint8_t byte_enable = get_byte_enable(packet);
uint16_t length = get_length(packet);
uint8_t tag = get_tag(packet);

// Access payload data
uint8_t *data = packet + HEADER_SIZE;
```

### Creating a Read Request
```c
uint8_t packet[12];  // Read requests are minimal size
uint32_t address = 0x1000;
uint8_t tag = 5;

create_read_packet(packet, address, tag);
```

## Testing

Test cases are provided in `test/inputs/` directory covering:
- Valid packets of all types
- Boundary conditions (min/max values)
- Invalid packets (format errors, checksum failures)
- Various byte-enable patterns
- Different address ranges
- Multiple packet sequences

## PCIe Protocol Details

### Packet Layout
```
[Header (12 bytes)]
├─ DWord 0: Format, Type, TC, Byte Enable bits
├─ DWord 1: Packet Length, Byte Count
└─ DWord 2-3: Address (32-bit)

[Payload (variable)]
└─ Data dwords...

[Optional fields]
└─ Tag, Status, etc.
```

### Dword (Double Word)
- 1 dword = 4 bytes = 32 bits
- Used as packet length unit in TLP

### Key Fields
| Field | Bits | Purpose |
|-------|------|---------|
| Format | 31-30 | Packet type (read/write/completion) |
| Type | 29-24 | Specific packet command |
| TC | 23-20 | Traffic class (priority) |
| Byte Enable | 19-12 | Which bytes are valid |
| Length | 11-0 | Packet payload length |
| Address | 63-32 | Target memory address (32-bit) |

## Implementation Highlights

### Efficient Byte Manipulation
- Direct array indexing for byte access
- Bitwise operations for field extraction
- Minimal memory overhead

### Robust Validation
- Packet length verification
- Address boundary checking
- Type field validation
- Byte-enable pattern validation

### Clear Code Organization
- Modular function design
- Consistent naming conventions
- Comprehensive documentation
- Error code returns

## Limitations

- 32-bit addressing only (PCIe also supports 64-bit)
- Fixed packet types (3 of many PCIe TLP types)
- No CRC/checksum calculation
- Single-threaded operation

## Real-World Applications

This protocol implementation is used in:
- **NVMe Drivers**: Communication between CPU and SSD
- **PCIe Device Drivers**: Any discrete PCIe peripheral
- **Firmware Development**: Low-level hardware interaction
- **Networking Hardware**: Network interface cards (NICs)
- **GPU Drivers**: Graphics processor communication

## Further Reading

- [PCIe Specification](https://pcisig.com/) (official standard)
- Transaction Layer Packet Format documentation
- Your course materials and lecture notes

## Learning Outcomes

This project demonstrates:
- Byte-level data manipulation
- Bit-level operations and field extraction
- Protocol parsing and validation
- Hardware communication fundamentals
- Systematic debugging of binary data
- Low-level performance optimization

## Performance Considerations

- Parse time: O(1) per packet
- Validation: O(1) simple checks
- Encoding/Decoding: O(n) where n = packet size
- Memory: Minimal overhead, in-place operations

## Academic Integrity

This is a course assignment for CSE 220. Code is provided for educational purposes and should only be used for learning about low-level protocol implementation and bit-level programming in C.

## License

Educational project for CSE 220: Systems Fundamentals I at Stony Brook University (Fall 2024).

## Author

Implemented as part of Stony Brook University's CSE 220 course.
