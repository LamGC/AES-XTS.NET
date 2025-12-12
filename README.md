# AES-XTS.NET

[English](README.md) | [中文](README.zh.md)

A streaming AES-XTS encryption implementation.

## ✨ Features

- **NIST XTSVS Compliance** - Passed all NIST XTSVS (XTS-AES Validation System) test vectors (except non-8-bit aligned
  cases)
- **Streaming Encryption** - Process data in chunks using `ProcessBytes`/`ProcessByte` methods, ideal for large files or
  network streams
- **Cross-Sector Streaming** - A single cipher instance can seamlessly process data across multiple sectors with
  automatic sector boundary handling, perfect for continuous data streams
- **Comprehensive Test Suite** - Extensive unit tests covering lifecycle management, CTS (Ciphertext Stealing), boundary
  conditions, and round-trip verification
- **Dual Operation Modes**:
    - `Continuous` - Multi-sector continuous encryption with automatic sector rollover
    - `Independent` - Single sector encryption with strict size validation
- **Zero-Allocation API** - Span-based methods to minimize GC pressure in performance-critical scenarios
- **SIMD Optimization** - Hardware-accelerated XOR operations using `Vector128` when available
- **Dual Key Size Support** - Supports both AES-128-XTS (256-bit total key) and AES-256-XTS (512-bit total key)

## 📦 Installation

### NuGet Package

```bash
dotnet add package LamGC.AES-XTS
```

### Build from Source

```bash
git clone https://github.com/LamGC/AES-XTS.NET.git
cd AES_XTS.NET
dotnet build -c Release
```

## 🚀 Quick Start

### Basic Encryption/Decryption

```csharp
using LamGC.AES_XTS;

// Prepare keys (AES-128-XTS requires two 16-byte keys, AES-256-XTS requires two 32-byte keys)
byte[] key1 = new byte[16]; // Data encryption key (K1)
byte[] key2 = new byte[16]; // Tweak encryption key (K2)
// Fill keys with secure random bytes...

// Create parameters
var parameters = new XtsAesCipherParameters(
    mode: XtsAesMode.Continuous,  // or XtsAesMode.Independent
    dataEncryptKey: key1,
    tweakCalcKey: key2,
    sectorSize: 512,              // Sector size in bytes (minimum 16)
    sectorIndex: 0                // Starting sector index
);

// Create and initialize cipher
using var cipher = new XtsAesBufferedCipher();
cipher.Init(forEncryption: true, parameters);

// Encrypt data
byte[] plaintext = new byte[1024];
byte[] ciphertext = cipher.DoFinal(plaintext);

// Decrypt data
cipher.Init(forEncryption: false, parameters);
byte[] decrypted = cipher.DoFinal(ciphertext);
```

### Streaming Encryption

```csharp
using var cipher = new XtsAesBufferedCipher();
cipher.Init(true, parameters);

using var outputStream = new MemoryStream();

// Process data in chunks
foreach (var chunk in dataChunks)
{
    byte[] output = cipher.ProcessBytes(chunk);
    outputStream.Write(output);
}

// Finalize and get remaining data
byte[] finalOutput = cipher.DoFinal();
outputStream.Write(finalOutput);
```

### Zero-Allocation Pattern

> **⚠️ Warning**: Zero-allocation is not fully implemented yet. The underlying C# `Aes` class still creates a
> significant number of objects internally. This issue will be addressed in future releases.

```csharp
using var cipher = new XtsAesBufferedCipher();
cipher.Init(true, parameters);

byte[] input = new byte[4096];
byte[] output = new byte[4096];

// Use Span-based API to avoid allocations
int bytesWritten = cipher.DoFinal(input.AsSpan(), output.AsSpan());
```

## ⚠️ Important Notes

1. **Minimum Data Size**: XTS mode requires at least 16 bytes of data per data unit
2. **Key Separation**: K1 and K2 should be different. Using identical keys significantly weakens security
3. **CTS Restrictions**: In `Continuous` mode, Ciphertext Stealing cannot cross sector boundaries
4. **Thread Safety**: `XtsAesBufferedCipher` is not thread-safe. Use separate instances for concurrent operations
5. **Zero-Allocation Limitation**: Zero-allocation is not fully implemented yet. The underlying C# `Aes` class still
   creates a significant number of objects internally. This issue will be addressed in future releases.

## 🧪 Running Tests

```bash
cd LamGC.AES_XTS.Tests
dotnet test
```

## 📊 Benchmarks

```bash
cd LamGC.AES_XTS.Benchmarks
dotnet run -c Release
```

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

```plaintext
MIT License

Copyright (c) 2025 LamGC

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## 🔗 References

- [IEEE P1619 - Standard for Cryptographic Protection of Data on Block-Oriented Storage Devices](https://ieeexplore.ieee.org/document/4493450)
- [NIST SP 800-38E - Recommendation for Block Cipher Modes of Operation: The XTS-AES Mode for Confidentiality on Storage Devices](https://csrc.nist.gov/publications/detail/sp/800-38e/final)
