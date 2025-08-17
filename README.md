# UUIDvX Technical Specification

**Version:** 1.0  
**Status:** Draft Standard  
**Authors:** Omar Hamdan <omar@phpdot.com>  
**Date:** August 15, 2025

---

## Abstract

UUIDvX is a distributed unique identifier standard designed for high-throughput, multi-machine environments requiring globally unique identifiers with embedded metadata. This specification defines the binary format, bit allocation algorithms, encoding schemes, and implementation requirements for creating interoperable UUIDvX implementations across programming languages and platforms.

UUIDvX addresses limitations in existing UUID standards by providing configurable bit allocation, embedded metadata compression, and distributed generation coordination while maintaining 128-bit compatibility with existing UUID infrastructure.

## Status of This Document

This document is submitted as a technical specification for consideration as a standard for distributed unique identifier generation. Implementation feedback and interoperability testing are encouraged.

## Copyright Notice

Copyright (c) 2025 Omar Hamdan. This document is made available under the MIT License.

---

## Table of Contents

1. [Introduction](#1-introduction)
   - 1.1 [Motivation](#11-motivation)
   - 1.2 [Scope](#12-scope)
   - 1.3 [Requirements Language](#13-requirements-language)

2. [UUIDvX Format Specification](#2-uuidvx-format-specification)
   - 2.1 [Overall Structure](#21-overall-structure)
   - 2.2 [Field Definitions](#22-field-definitions)

3. [Metadata Compression Specification](#3-metadata-compression-specification)
   - 3.1 [Compression Algorithm](#31-compression-algorithm)
   - 3.2 [Compression Constraints](#32-compression-constraints)
   - 3.3 [Decompression Algorithm](#33-decompression-algorithm)
   - 3.4 [Validation Rules](#34-validation-rules)

4. [Bit Allocation Algorithms](#4-bit-allocation-algorithms)
   - 4.1 [Optimal Allocation Algorithm](#41-optimal-allocation-algorithm)
   - 4.2 [Manual Allocation](#42-manual-allocation)

5. [Clock Drift Handling Strategies](#5-clock-drift-handling-strategies)
   - 5.1 [Strategy Types](#51-strategy-types)
   - 5.2 [Sequence Number Management](#52-sequence-number-management)

6. [Encoding Formats](#6-encoding-formats)
   - 6.1 [Hexadecimal Format](#61-hexadecimal-format)
   - 6.2 [UUID-Style Format](#62-uuid-style-format)
   - 6.3 [Base62 Format](#63-base62-format)
   - 6.4 [Format Conversion Requirements](#64-format-conversion-requirements)

7. [Generation Algorithm](#7-generation-algorithm)
   - 7.1 [Generation Process Overview](#71-generation-process-overview)
   - 7.2 [Timestamp Generation](#72-timestamp-generation)
   - 7.3 [Clock Drift Handling](#73-clock-drift-handling)
   - 7.4 [Sequence Number Management](#74-sequence-number-management)
   - 7.5 [Random Bits Generation](#75-random-bits-generation)
   - 7.6 [ID Assembly](#76-id-assembly)

8. [Parsing Algorithm](#8-parsing-algorithm)
   - 8.1 [Parsing Process Overview](#81-parsing-process-overview)
   - 8.2 [Format Normalization](#82-format-normalization)
   - 8.3 [Field Extraction](#83-field-extraction)
   - 8.4 [Metadata Decoding](#84-metadata-decoding)
   - 8.5 [Flexible Section Parsing](#85-flexible-section-parsing)

9. [Security Considerations](#9-security-considerations)
   - 9.1 [Randomness Requirements](#91-randomness-requirements)
   - 9.2 [Information Disclosure](#92-information-disclosure)
   - 9.3 [Collision Resistance](#93-collision-resistance)
   - 9.4 [Predictability](#94-predictability)

10. [Interoperability Requirements](#10-interoperability-requirements)
    - 10.1 [Cross-Platform Compatibility](#101-cross-platform-compatibility)
    - 10.2 [Version Compatibility](#102-version-compatibility)
    - 10.3 [Endianness Handling](#103-endianness-handling)

11. [Test Vectors](#11-test-vectors)
    - 11.1 [Basic Generation Test](#111-basic-generation-test)
    - 11.2 [Metadata Encoding Test](#112-metadata-encoding-test)
    - 11.3 [Clock Drift Test Cases](#113-clock-drift-test-cases)
    - 11.4 [Format Conversion Test](#114-format-conversion-test)

12. [Implementation Requirements](#12-implementation-requirements)
    - 12.1 [Error Handling](#121-error-handling)
    - 12.2 [Performance Requirements](#122-performance-requirements)

13. [Migration from Other ID Systems](#13-migration-from-other-id-systems)
    - 13.1 [From UUIDv4](#131-from-uuidv4)
    - 13.2 [From Snowflake IDs](#132-from-snowflake-ids)
    - 13.3 [From Sequential IDs](#133-from-sequential-ids)

14. [Future Extensions](#14-future-extensions)
    - 14.1 [Version 2+ Considerations](#141-version-2-considerations)
    - 14.2 [Backwards Compatibility](#142-backwards-compatibility)

15. [References](#15-references)
    - 15.1 [Normative References](#151-normative-references)
    - 15.2 [Informative References](#152-informative-references)

16. [Security and Privacy Considerations](#16-security-and-privacy-considerations)
    - 16.1 [Privacy Implications](#161-privacy-implications)
    - 16.2 [Mitigation Strategies](#162-mitigation-strategies)

**Appendices**
- [Appendix A: Complete PHP Reference Implementation](#appendix-a-complete-php-reference-implementation)
- [Appendix B: Implementation Checklist](#appendix-b-implementation-checklist)
- [Authors' Addresses](#authors-addresses)
- [Copyright Statement](#copyright-statement)

---

## 1. Introduction

### 1.1 Motivation

Existing UUID standards were designed for different use cases than modern distributed systems. UUIDvX addresses these limitations:

- **Configurable Structure**: Adaptable bit allocation for different scale requirements
- **Embedded Metadata**: Compressed configuration metadata for parsing compatibility
- **High Throughput**: Optimized for high-frequency generation scenarios
- **Clock Drift Handling**: Multiple strategies for time synchronization issues
- **Distributed Coordination**: Machine ID allocation for collision avoidance

### 1.2 Scope

This specification defines:
- Binary format and bit layout
- Bit allocation algorithms
- Metadata compression schemes
- Clock drift handling strategies
- Encoding formats (hex, UUID, Base62)
- Interoperability requirements
- Test vectors and validation procedures

### 1.3 Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

---

## 2. UUIDvX Format Specification

### 2.1 Overall Structure

UUIDvX uses a 128-bit format organized into distinct sections:

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     Timestamp (bits 127-96)                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|    Timestamp (bits 95-86)    |Ver|    Metadata (15 bits)    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                    Flexible Section (67 bits)                |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Where:
- **Timestamp**: 42 bits total (127-86)
- **Version**: 4 bits (85-82) 
- **Metadata**: 15 bits (81-67)
- **Flexible Section**: 67 bits (66-0)

### 2.2 Field Definitions

#### 2.2.1 Timestamp (42 bits)
- **Precision**: 1 millisecond
- **Epoch**: Unix epoch (January 1, 1970 00:00:00 UTC)
- **Range**: 139.4 years from epoch
- **Encoding**: Big-endian unsigned integer
- **Maximum Value**: 4,398,046,511,103 (0x3FFFFFFFFFF)

#### 2.2.2 Version (4 bits)
- **Current Version**: 1
- **Maximum Version**: 15
- **Purpose**: Format versioning and future compatibility
- **Reserved Values**: 0 (invalid), 2-15 (future use)

#### 2.2.3 Metadata (15 bits)
- **Purpose**: Compressed bit allocation information
- **Encoding**: Custom compression scheme (Section 3)
- **Content**: Machine ID bits, Type bits, Sequence bits encoding

#### 2.2.4 Flexible Section (67 bits)
- **Components**: Machine ID + Type ID + Sequence + Random
- **Allocation**: Configurable based on requirements
- **Constraints**: See Section 4 for allocation algorithms

---

## 3. Metadata Compression Specification

### 3.1 Compression Algorithm

The 15-bit metadata section encodes bit allocation information using a compressed format:

```
Metadata = (s << 10) | (t << 5) | q

Where:
- s = (machine_id_bits - 1) ÷ 2  (compressed machine bits, 5 bits)
- t = (type_bits - 2) ÷ 2        (compressed type bits, 5 bits) 
- q = (sequence_bits - 12) ÷ 2   (compressed sequence bits, 5 bits)
```

### 3.2 Compression Constraints

- **Machine ID bits**: Must be odd, minimum 1
- **Type bits**: Must be even, minimum 2  
- **Sequence bits**: Minimum 12, maximum varies
- **Total constraint**: s + t + q ≤ 18 (metadata field limit)

### 3.3 Decompression Algorithm

```
machine_id_bits = (s * 2) + 1
type_bits = (t * 2) + 2
sequence_bits = (q * 2) + 12
random_bits = 67 - machine_id_bits - type_bits - sequence_bits
```

### 3.4 Validation Rules

Implementations MUST validate:
1. `machine_id_bits ≥ 1` and odd
2. `type_bits ≥ 2` and even
3. `sequence_bits ≥ 12`
4. `random_bits ≥ 16`
5. Total flexible bits = 67

---

## 4. Bit Allocation Algorithms

### 4.1 Optimal Allocation Algorithm

This algorithm calculates the most efficient bit distribution for given requirements:

#### 4.1.1 Input Parameters
- `max_machines`: Maximum number of machines that will generate IDs
- `max_types`: Maximum number of type categories needed

#### 4.1.2 Calculation Steps

**Step 1: Calculate minimum required bits**
```
machine_bits_required = CEILING(LOG2(max_machines))
type_bits_required = CEILING(LOG2(max_types))
```

**Step 2: Apply minimum constraints**
```
machine_bits = MAX(machine_bits_required, MIN_MACHINE_BITS)
type_bits = MAX(type_bits_required, MIN_TYPE_BITS)
```
Where:
- `MIN_MACHINE_BITS = 1`
- `MIN_TYPE_BITS = 2`

**Step 3: Enforce compression compatibility**
```
IF machine_bits MOD 2 = 0 THEN
    machine_bits = machine_bits + 1
END IF

IF type_bits MOD 2 = 1 THEN  
    type_bits = type_bits + 1
END IF
```

**Step 4: Calculate remaining bits**
```
sequence_bits = MIN_SEQUENCE_BITS
random_bits = FLEXIBLE_BITS_TOTAL - machine_bits - type_bits - sequence_bits
```
Where:
- `MIN_SEQUENCE_BITS = 12`
- `FLEXIBLE_BITS_TOTAL = 67`

**Step 5: Validate minimum random bits constraint**
```
IF random_bits < MIN_RANDOM_BITS THEN
    THROW InvalidConfigurationError("Insufficient bits for random component")
END IF
```
Where:
- `MIN_RANDOM_BITS = 16`

### 4.2 Manual Allocation

Implementations MAY allow manual bit allocation with these constraints:
- Total flexible bits MUST equal 67
- Minimum requirements MUST be met
- Compression constraints MUST be satisfied

---

## 5. Clock Drift Handling Strategies

### 5.1 Strategy Types

Implementations MUST support at least one of these strategies:

#### 5.1.1 Strict Clock Strategy
- **Behavior**: Throws error on any backward time movement
- **Use Case**: Environments with reliable time synchronization
- **Implementation**: Detect `current_time < last_time` and error

#### 5.1.2 Tolerant Clock Strategy  
- **Behavior**: Uses last known time when clock moves backward
- **Use Case**: Environments with occasional time drift
- **Implementation**: Use `max(current_time, last_time)` as timestamp

#### 5.1.3 Wait Clock Strategy
- **Behavior**: Sleeps until time advances beyond last timestamp
- **Use Case**: Real-time systems requiring temporal ordering
- **Implementation**: Sleep for `(last_time - current_time + 1)` milliseconds

### 5.2 Sequence Number Management

For any timestamp (repeated or new):
1. If `timestamp > last_timestamp`: reset sequence to 0
2. If `timestamp == last_timestamp`: increment sequence
3. If sequence overflows: apply clock strategy for next millisecond

---

## 6. Encoding Formats

### 6.1 Hexadecimal Format
- **Format**: 32 lowercase hexadecimal characters
- **Example**: `"01234567890abcdef01234567890abcdef"`
- **Use Case**: Binary representation, debugging

### 6.2 UUID-Style Format
- **Format**: 8-4-4-4-12 hyphen-separated groups (UUID-like formatting)
- **Example**: `"01234567-890a-bcde-f012-34567890abcdef"`
- **Use Case**: Familiar formatting, though UUIDvX is not RFC 4122 compliant

### 6.3 Base62 Format
- **Character Set**: `0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz`
- **Length**: Variable (typically ~22 characters)
- **Example**: `"1NyYhOgdKQvEK2eQWlRhNr"`
- **Use Case**: URL-safe, compact representation

### 6.4 Format Conversion Requirements

Implementations MUST:
- Support bidirectional conversion between all formats
- Preserve exact bit values across conversions
- Validate format syntax before conversion

---

## 7. Generation Algorithm

### 7.1 Generation Process Overview

The generation process consists of five main steps:
1. Obtain current timestamp
2. Apply clock drift handling strategy
3. Manage sequence numbers for same-millisecond uniqueness
4. Generate cryptographically secure random bits
5. Assemble the 128-bit ID according to bit layout

### 7.2 Timestamp Generation

#### 7.2.1 Timestamp Requirements
- **Precision**: 1 millisecond
- **Epoch**: Unix epoch (January 1, 1970 00:00:00 UTC)
- **Range**: 42 bits = 4,398,046,511,103 milliseconds ≈ 139.4 years
- **Format**: Big-endian unsigned integer

#### 7.2.2 Timestamp Calculation
```
current_timestamp_ms = CURRENT_TIME_MILLISECONDS_SINCE_UNIX_EPOCH()
```

### 7.3 Clock Drift Handling

Apply the configured clock strategy to handle potential time drift:

```
processed_timestamp = APPLY_CLOCK_STRATEGY(current_timestamp_ms, last_timestamp)
```

### 7.4 Sequence Number Management

#### 7.4.1 Sequence Logic
```
IF processed_timestamp = last_timestamp THEN
    sequence = (last_sequence + 1) MOD max_sequence_value
    IF sequence = 0 THEN
        // Sequence overflow - advance to next millisecond
        processed_timestamp = HANDLE_SEQUENCE_OVERFLOW(processed_timestamp)
        sequence = 0
    END IF
ELSE
    sequence = 0
    last_timestamp = processed_timestamp
END IF

last_sequence = sequence
```

Where:
```
max_sequence_value = 2^sequence_bits
```

### 7.5 Random Bits Generation

#### 7.5.1 Random Requirements
- **Source**: Cryptographically secure random number generator
- **Bits**: As specified in bit allocation (minimum 16 bits)
- **Distribution**: Uniform distribution across bit range

#### 7.5.2 Random Generation
```
random_value = SECURE_RANDOM_BITS(random_bits_count)
random_value = random_value AND ((2^random_bits_count) - 1)
```

### 7.6 ID Assembly

#### 7.6.1 Bit Layout Assembly
The 128-bit ID is assembled as follows:

```
// Initialize 128-bit value
id_value = 0

// Position 127-86: Timestamp (42 bits)
id_value = id_value OR (processed_timestamp << 86)

// Position 85-82: Version (4 bits)  
id_value = id_value OR (CURRENT_VERSION << 82)

// Position 81-67: Metadata (15 bits)
metadata_value = ENCODE_METADATA(machine_bits, type_bits, sequence_bits)
id_value = id_value OR (metadata_value << 67)

// Position 66-0: Flexible section (67 bits)
flexible_value = ASSEMBLE_FLEXIBLE_SECTION(machine_id, type_id, sequence, random_value, bit_allocation)
id_value = id_value OR flexible_value
```

#### 7.6.2 Flexible Section Assembly
```
FUNCTION ASSEMBLE_FLEXIBLE_SECTION(machine_id, type_id, sequence, random_value, bit_allocation):
    flexible_value = 0
    bit_position = bit_allocation.random_bits
    
    // Random bits at position 0 to (random_bits-1)
    flexible_value = flexible_value OR random_value
    
    // Sequence bits at position random_bits to (random_bits + sequence_bits - 1)
    flexible_value = flexible_value OR (sequence << bit_position)
    bit_position = bit_position + bit_allocation.sequence_bits
    
    // Type bits at position (random_bits + sequence_bits) to (random_bits + sequence_bits + type_bits - 1)
    flexible_value = flexible_value OR (type_id << bit_position)
    bit_position = bit_position + bit_allocation.type_bits
    
    // Machine bits at position (random_bits + sequence_bits + type_bits) to (66)
    flexible_value = flexible_value OR (machine_id << bit_position)
    
    RETURN flexible_value
END FUNCTION
```


---

## 8. Parsing Algorithm

### 8.1 Parsing Process Overview

The parsing process consists of four main steps:
1. Normalize input format to 128-bit binary value
2. Extract fixed fields (timestamp, version, metadata)
3. Decode metadata to determine bit allocation
4. Extract flexible section components based on decoded allocation

### 8.2 Format Normalization

#### 8.2.1 Input Format Detection

```
FUNCTION DETECT_FORMAT(input_string):
    input_length = LENGTH(input_string)
    
    IF input_length = 32 AND ALL_HEX_CHARACTERS(input_string) THEN
        RETURN "hexadecimal"
    ELSE IF input_length = 36 AND CHAR_AT(input_string, 8) = '-' AND 
            CHAR_AT(input_string, 13) = '-' AND CHAR_AT(input_string, 18) = '-' AND
            CHAR_AT(input_string, 23) = '-' THEN
        RETURN "uuid_style"
    ELSE IF ALL_BASE62_CHARACTERS(input_string) THEN
        RETURN "base62"
    ELSE
        THROW InvalidFormatError("Unrecognized format")
    END IF
END FUNCTION
```

#### 8.2.2 Format Conversion

```
FUNCTION NORMALIZE_TO_128_BIT(input_string, format):
    SWITCH format:
        CASE "hexadecimal":
            RETURN PARSE_HEX_TO_128_BIT(input_string)
        CASE "uuid_style":
            hex_string = REMOVE_HYPHENS(input_string)
            RETURN PARSE_HEX_TO_128_BIT(hex_string)
        CASE "base62":
            RETURN PARSE_BASE62_TO_128_BIT(input_string)
        DEFAULT:
            THROW InvalidFormatError("Unknown format")
    END SWITCH
END FUNCTION
```

### 8.3 Field Extraction

#### 8.3.1 Fixed Fields Extraction

```
FUNCTION EXTRACT_FIXED_FIELDS(id_value):
    // Extract timestamp (bits 127-86, 42 bits)
    timestamp = (id_value >> 86) AND 0x3FFFFFFFFFF
    
    // Extract version (bits 85-82, 4 bits)  
    version = (id_value >> 82) AND 0xF
    
    // Extract metadata (bits 81-67, 15 bits)
    metadata = (id_value >> 67) AND 0x7FFF
    
    // Extract flexible section (bits 66-0, 67 bits)
    flexible = id_value AND 0x7FFFFFFFFFFFFFF
    
    RETURN (timestamp, version, metadata, flexible)
END FUNCTION
```

#### 8.3.2 Version Validation

```
IF version ≠ CURRENT_VERSION THEN
    THROW InvalidVersionError("Unsupported version: " + version)
END IF
```

Where `CURRENT_VERSION = 1`

### 8.4 Metadata Decoding

#### 8.4.1 Metadata Decompression

```
FUNCTION DECODE_METADATA(metadata_value):
    // Extract 5-bit components from 15-bit metadata
    s = (metadata_value >> 10) AND 0x1F    // Bits 14-10
    t = (metadata_value >> 5) AND 0x1F     // Bits 9-5  
    q = metadata_value AND 0x1F            // Bits 4-0
    
    // Decompress to actual bit allocations
    machine_id_bits = (s * 2) + 1
    type_bits = (t * 2) + 2
    sequence_bits = (q * 2) + 12
    random_bits = FLEXIBLE_BITS_TOTAL - machine_id_bits - type_bits - sequence_bits
    
    // Validate decompressed values
    VALIDATE_BIT_ALLOCATION(machine_id_bits, type_bits, sequence_bits, random_bits)
    
    RETURN (machine_id_bits, type_bits, sequence_bits, random_bits)
END FUNCTION
```

Where `FLEXIBLE_BITS_TOTAL = 67`

### 8.5 Flexible Section Parsing

#### 8.5.1 Component Extraction

```
FUNCTION EXTRACT_FLEXIBLE_COMPONENTS(flexible_value, bit_allocation):
    bit_position = 0
    
    // Extract random bits (bits 0 to random_bits-1)
    random_mask = (2^bit_allocation.random_bits) - 1
    random_value = flexible_value AND random_mask
    bit_position = bit_allocation.random_bits
    
    // Extract sequence bits  
    sequence_mask = (2^bit_allocation.sequence_bits) - 1
    sequence = (flexible_value >> bit_position) AND sequence_mask
    bit_position = bit_position + bit_allocation.sequence_bits
    
    // Extract type bits
    type_mask = (2^bit_allocation.type_bits) - 1
    type_id = (flexible_value >> bit_position) AND type_mask
    bit_position = bit_position + bit_allocation.type_bits
    
    // Extract machine bits
    machine_mask = (2^bit_allocation.machine_id_bits) - 1
    machine_id = (flexible_value >> bit_position) AND machine_mask
    
    RETURN (machine_id, type_id, sequence, random_value)
END FUNCTION
```


---

## 9. Security Considerations

### 9.1 Randomness Requirements

- Random bits MUST use cryptographically secure random number generators
- Minimum 16 bits of entropy per ID
- Implementations SHOULD NOT reduce randomness for performance

### 9.2 Information Disclosure

- Machine IDs MAY reveal infrastructure information
- Timestamps reveal generation time (by design)
- Type IDs MAY reveal application structure
- Sequence numbers reveal generation rate

### 9.3 Collision Resistance

With proper configuration:
- **Same millisecond, same machine**: Protected by sequence + random bits
- **Same millisecond, different machines**: Protected by machine ID + random bits
- **Different milliseconds**: Protected by timestamp

### 9.4 Predictability

- Future IDs are partially predictable (timestamp component)
- Machine and type IDs are deterministic
- Random component provides unpredictability
- Sequence numbers are predictable within timestamp

---

## 10. Interoperability Requirements

### 10.1 Cross-Platform Compatibility

All implementations MUST:
- Generate identical IDs given same inputs
- Parse IDs generated by other implementations
- Handle all encoding formats correctly
- Validate metadata constraints consistently

### 10.2 Version Compatibility

- Version 1 parsers MUST reject version 0 IDs
- Future versions SHOULD be backward compatible
- Unknown versions MUST be rejected with clear error messages

### 10.3 Endianness Handling

- All multi-byte values use big-endian encoding
- Bit operations use consistent bit ordering
- String representations use consistent character ordering

---

## 11. Test Vectors

### 11.1 Basic Generation Test

**Configuration:**
- Max machines: 4 (requires 3 bits, adjusted to 3 for odd constraint)  
- Max types: 4 (requires 2 bits, adjusted to 2 for even constraint)
- Bit allocation: machine=3, type=2, sequence=12, random=50

**Input:**
- Machine ID: 1
- Type ID: 2
- Timestamp: 1692115200000 (2023-08-15 12:00:00 UTC)
- Sequence: 0
- Random: 0x123456789ABC (truncated to 50 bits)

**Metadata Calculation:**
- s = (3-1)/2 = 1
- t = (2-2)/2 = 0  
- q = (12-12)/2 = 0
- Metadata = (1 << 10) | (0 << 5) | 0 = 1024 = 0x400

**Expected Output (Hex):**
`18a4c320000120000024681357d780`

**Breakdown:**
- Timestamp (42 bits): `1692115200000` = `0x18A4C320000`
- Version (4 bits): `1` = `0x1`
- Metadata (15 bits): `0x400`
- Flexible (67 bits): Contains machine=1, type=2, sequence=0, random bits

### 11.2 Metadata Encoding Test

**Input Configurations:**

| Machines | Types | Machine Bits | Type Bits | Sequence Bits | Random Bits | s | t | q | Metadata |
|----------|--------|--------------|-----------|---------------|-------------|---|---|---|----------|
| 2        | 4      | 3           | 2         | 12           | 50          | 1 | 0 | 0 | 0x400    |
| 100      | 8      | 7           | 4         | 12           | 44          | 3 | 1 | 0 | 0xC20    |
| 1000     | 16     | 11          | 6         | 14           | 36          | 5 | 2 | 1 | 0x1441   |

### 11.3 Clock Drift Test Cases

**Strict Clock Strategy:**
- Last timestamp: 1000
- Current timestamp: 999
- Expected: Error/Exception

**Tolerant Clock Strategy:**
- Last timestamp: 1000  
- Current timestamp: 999
- Expected: Use timestamp 1000

**Wait Clock Strategy:**
- Last timestamp: 1000
- Current timestamp: 999  
- Expected: Sleep 2ms, use timestamp 1001

### 11.4 Format Conversion Test

**Source (Hex):** `018a4c32000010000000123456789abc`

**Expected Conversions:**
- **UUID:** `018a4c32-0000-1000-0000-123456789abc`
- **Base62:** `1B6RYUfxUKQvEK2eRWlRhNr` (example)

---

## 12. Implementation Requirements

### 12.1 Error Handling

Implementations MUST define specific error types for consistent behavior:

#### 12.1.1 Required Error Types
- **InvalidVersionError**: Thrown when encountering unsupported version numbers
- **InvalidFormatError**: Thrown when input string format is malformed or unrecognized
- **InvalidConfigurationError**: Thrown when bit allocation constraints are violated
- **ClockDriftError**: Thrown when backward time movement occurs (strict clock mode)
- **SequenceOverflowError**: Thrown when sequence counter exhausts available values

#### 12.1.2 Error Conditions

**InvalidVersionError** conditions:
- Version field contains value other than 1
- Version field extraction results in invalid bit pattern

**InvalidFormatError** conditions:
- Input string length doesn't match any supported format
- Hexadecimal format contains non-hex characters
- UUID-style format has hyphens in wrong positions
- Base62 format contains invalid characters

**InvalidConfigurationError** conditions:
- Total flexible bits ≠ 67
- Machine ID bits < 1 or not odd
- Type bits < 2 or not even  
- Sequence bits < 12
- Random bits < 16
- Metadata compression constraint s + t + q > 18

### 12.2 Performance Requirements

#### 12.2.1 Mandatory Optimizations
- **Metadata Caching**: Cache encoded metadata values for repeated configurations
- **Bit Mask Precomputation**: Precompute bit masks for component extraction
- **Sequence State Management**: Maintain sequence counters efficiently

#### 12.2.2 Recommended Optimizations
- **Memory Pooling**: Reuse allocated objects in high-frequency scenarios
- **Batch Generation**: Optimize for generating multiple IDs with same configuration
- **Platform-Specific Math**: Use native 128-bit arithmetic when available

---

## 13. Migration from Other ID Systems

### 13.1 From UUIDv4

**Advantages of Migration:**
- Embedded configuration metadata
- Higher throughput potential  
- Machine coordination
- Flexible bit allocation

**Migration Strategy:**
1. Implement UUIDvX alongside UUIDv4
2. Gradually migrate new entities
3. Maintain parsing for both formats
4. Plan for dual-format support period

### 13.2 From Snowflake IDs

**Comparison:**
- UUIDvX: 128-bit vs Snowflake: 64-bit
- UUIDvX: Configurable vs Snowflake: Fixed allocation
- UUIDvX: Metadata compression vs Snowflake: No metadata

**Migration Considerations:**
- 64-bit Snowflake IDs can be embedded in UUIDvX
- Timestamp epochs may differ
- Machine ID mappings need coordination

### 13.3 From Sequential IDs

**Benefits:**
- Global uniqueness without coordination
- Distributed generation capability
- Collision resistance
- Temporal ordering within machine

**Challenges:**
- Larger storage requirements (16 vs 4-8 bytes)
- More complex parsing
- Performance considerations

---

## 14. Future Extensions

### 14.1 Version 2+ Considerations

Potential enhancements:
- Alternative timestamp precisions (microseconds, nanoseconds)
- Different metadata compression schemes
- Additional clock drift strategies
- Custom encoding formats
- Cryptographic signatures

### 14.2 Backwards Compatibility

Future versions SHOULD:
- Maintain 128-bit total size
- Preserve timestamp and version field positions
- Support version 1 parsing
- Clearly document breaking changes

---

## 15. References

### 15.1 Normative References

- **RFC 2119**: Key words for use in RFCs to Indicate Requirement Levels
- **RFC 4122**: A Universally Unique IDentifier (UUID) URN Namespace (for reference only - UUIDvX is not RFC 4122 compliant)
- **RFC 4648**: The Base16, Base32, and Base64 Data Encodings

### 15.2 Informative References

- Twitter Snowflake ID Generation
- MongoDB ObjectId Specification  
- PostgreSQL UUID Functions
- Distributed Systems Clock Synchronization

---

## 16. Security and Privacy Considerations

### 16.1 Privacy Implications

UUIDvX IDs contain:
- **Timestamp**: Reveals when ID was generated
- **Machine ID**: May reveal infrastructure topology  
- **Type ID**: May reveal application structure
- **Sequence**: Reveals generation rate patterns

### 16.2 Mitigation Strategies

- Use network-local machine IDs (not public)
- Rotate machine IDs periodically
- Consider generic type names
- Monitor sequence pattern disclosure

---

## Appendix A. Complete PHP Reference Implementation

See the open source PHP implementation at: https://github.com/phpdot/uuidvx

Key files:
- `src/Core/Generator.php` - Generation algorithm
- `src/Core/Parser.php` - Parsing algorithm  
- `src/Metadata/Encoder.php` - Metadata compression
- `src/Metadata/Decoder.php` - Metadata decompression
- `src/Clock/` - Clock drift strategies

---

## Appendix B. Implementation Checklist

### B.1 Core Requirements

- [ ] 128-bit ID generation
- [ ] Version 1 format support
- [ ] Metadata encoding/decoding
- [ ] Bit allocation algorithms
- [ ] All three encoding formats (hex, UUID, Base62)
- [ ] At least one clock drift strategy

### B.2 Quality Requirements  

- [ ] Comprehensive test suite
- [ ] Performance benchmarks
- [ ] Security review
- [ ] Documentation
- [ ] Example usage

### B.3 Interoperability Requirements

- [ ] Cross-implementation testing
- [ ] Test vector validation
- [ ] Format conversion accuracy
- [ ] Error handling consistency

---

## Authors' Addresses

Omar Hamdan  
Email: omar@phpdot.com  
GitHub: @phpdot

---

## Copyright Statement

Copyright (c) 2025 Omar Hamdan.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
