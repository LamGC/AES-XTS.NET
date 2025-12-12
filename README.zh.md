# AES-XTS.NET

[English](README.md) | [中文](README.zh.md)

一个流式 AES-XTS 加密实现。

## ✨ 特性

- **NIST XTSVS 合规** - 通过了所有 NIST XTSVS（XTS-AES 验证系统）测试向量（非 8 位对齐的情况除外）
- **流式加密** - 使用 `ProcessBytes`/`ProcessByte` 方法分块处理数据，适用于大文件或网络流
- **跨扇区流式处理** - 单个加密器实例可无缝处理跨多个扇区的数据，自动处理扇区边界，完美适用于连续数据流
- **完善的测试套件** - 全面的单元测试，覆盖生命周期管理、CTS（密文窃取）、边界条件及往返验证
- **双操作模式**：
  - `Continuous` - 多扇区连续加密，自动处理扇区切换
  - `Independent` - 单扇区加密，严格的大小验证
- **零分配 API** - 基于 Span 的方法，在性能敏感场景下最小化 GC 压力
- **SIMD 优化** - 当硬件支持时使用 `Vector128` 进行加速的 XOR 操作
- **双密钥长度支持** - 同时支持 AES-128-XTS（256 位总密钥）和 AES-256-XTS（512 位总密钥）

## 📦 安装

### NuGet 包

```bash
dotnet add package LamGC.AES-XTS
```

### 从源码构建

```bash
git clone https://github.com/LamGC/AES-XTS.NET.git
cd AES_XTS.NET
dotnet build -c Release
```

## 🚀 快速开始

### 基本加解密

```csharp
using LamGC.AES_XTS;

// 准备密钥（AES-128-XTS 需要两个 16 字节密钥，AES-256-XTS 需要两个 32 字节密钥）
byte[] key1 = new byte[16]; // 数据加密密钥 (K1)
byte[] key2 = new byte[16]; // Tweak 加密密钥 (K2)
// 使用安全随机数填充密钥...

// 创建参数
var parameters = new XtsAesCipherParameters(
    mode: XtsAesMode.Continuous,  // 或 XtsAesMode.Independent
    dataEncryptKey: key1,
    tweakCalcKey: key2,
    sectorSize: 512,              // 扇区大小（字节），最小 16
    sectorIndex: 0                // 起始扇区索引
);

// 创建并初始化加密器
using var cipher = new XtsAesBufferedCipher();
cipher.Init(forEncryption: true, parameters);

// 加密数据
byte[] plaintext = new byte[1024];
byte[] ciphertext = cipher.DoFinal(plaintext);

// 解密数据
cipher.Init(forEncryption: false, parameters);
byte[] decrypted = cipher.DoFinal(ciphertext);
```

### 流式加密

```csharp
using var cipher = new XtsAesBufferedCipher();
cipher.Init(true, parameters);

using var outputStream = new MemoryStream();

// 分块处理数据
foreach (var chunk in dataChunks)
{
    byte[] output = cipher.ProcessBytes(chunk);
    outputStream.Write(output);
}

// 完成处理并获取剩余数据
byte[] finalOutput = cipher.DoFinal();
outputStream.Write(finalOutput);
```

### 零分配模式

> **⚠️ 警告**：零分配功能尚未完全实现。底层 C# `Aes` 类内部仍会创建大量对象。该问题将在后续版本中修复。

```csharp
using var cipher = new XtsAesBufferedCipher();
cipher.Init(true, parameters);

byte[] input = new byte[4096];
byte[] output = new byte[4096];

// 使用基于 Span 的 API 避免内存分配
int bytesWritten = cipher.DoFinal(input.AsSpan(), output.AsSpan());
```

## ⚠️ 重要说明

1. **最小数据大小**：XTS 模式要求每个数据单元至少 16 字节
2. **密钥分离**：K1 和 K2 应该不同。使用相同的密钥会显著降低安全性
3. **CTS 限制**：在 `Continuous` 模式下，密文窃取不能跨越扇区边界
4. **线程安全**：`XtsAesBufferedCipher` 不是线程安全的。并发操作请使用单独的实例
5. **零分配限制**：零分配功能尚未完全实现。底层 C# `Aes` 类内部仍会创建大量对象。该问题将在后续版本中修复。

## 🧪 运行测试

```bash
cd LamGC.AES_XTS.Tests
dotnet test
```

## 📊 基准测试

```bash
cd LamGC.AES_XTS.Benchmarks
dotnet run -c Release
```

## 📄 许可证

本项目采用 MIT 许可证 - 详情请参阅 [LICENSE](LICENSE) 文件。

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

## 🔗 参考资料

- [IEEE P1619 - 面向块存储设备数据加密保护标准](https://ieeexplore.ieee.org/document/4493450)
- [NIST SP 800-38E - 分组密码操作模式建议：用于存储设备机密性的 XTS-AES 模式](https://csrc.nist.gov/publications/detail/sp/800-38e/final)

