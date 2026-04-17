When code is compiled, it becomes a **binary executable**.  
Each operating system uses its own executable format.

If you’re doing reverse engineering, malware analysis, or exploit dev — you _must_ understand these formats.

---

# 1️⃣ PE (Portable Executable)

Used in: **Windows**

Extensions:

- `.exe`
    
- `.dll`
    
- `.sys`
    

---

### Structure Overview

[DOS Header]  
[PE Header]  
[Optional Header]  
[Section Table]  
[Sections]

---

### Important Sections

|Section|Purpose|
|---|---|
|.text|Executable code|
|.data|Initialized variables|
|.rdata|Read-only data|
|.rsrc|Resources (icons, strings)|
|.idata|Import table|
|.edata|Export table|

---

### What REs Look For

- Import Address Table (IAT)
    
- Entry Point
    
- Suspicious API calls
    
- Packed sections
    

---

# 2️⃣ ELF (Executable and Linkable Format)

Used in: **Linux, Unix**

Extensions:

- (no extension common)
    
- `.so` (shared object)
    

---

### Structure Overview

[ELF Header]  
[Program Headers]  
[Section Headers]  
[Sections]

---

### Important Sections

|Section|Purpose|
|---|---|
|.text|Code|
|.data|Initialized variables|
|.bss|Uninitialized variables|
|.rodata|Read-only data|
|.plt|Procedure Linkage Table|
|.got|Global Offset Table|
|.symtab|Symbol table|

---

### What REs Look For

- Entry point
    
- PLT/GOT (important for exploitation)
    
- Linked libraries
    
- Stripped vs non-stripped binaries
    

---

# 3️⃣ Mach-O (Mach Object)

Used in: **macOS / iOS**

Extensions:

- No extension (executables)
    
- `.dylib`
    
- `.app` (bundled app container)
    

---

### Structure Overview

[Mach Header]  
[Load Commands]  
[Segments]  
[Sections]

---

### Important Segments

|Segment|Purpose|
|---|---|
|__TEXT|Code|
|__DATA|Data|
|__LINKEDIT|Linking info|

---

### Special Feature

Mach-O supports:

- Fat binaries (multiple architectures in one file)
    
- Strong integration with Apple ecosystem
    

---

# 4️⃣ COFF (Common Object File Format)

Used in:

- Old Unix systems
    
- Base format for PE
    

Primarily used for:

- Object files (`.obj`)
    
- Intermediate compilation stage
    

---

# 5️⃣ a.out (Assembler Output)

Used in:

- Very old Unix systems
    

Mostly historical.  
Replaced by ELF in modern Linux.

Still good to know historically.

---

# 6️⃣ WASM (WebAssembly)

Used in:

- Web browsers
    
- Sandbox environments
    

Extension:

- `.wasm`
    

Binary format designed for:

- Safe execution in browsers
    
- Near-native performance
    

RE relevance:

- Increasing in web exploitation & malware research
    

---

# 7️⃣ DEX (Dalvik Executable)

Used in:

- Android
    

Extension:

- `.dex`
    

Runs on:

- Dalvik VM / ART runtime
    

RE relevance:

- Android malware analysis
    
- Mobile app security
    

---

# 8️⃣ Firmware Formats (Embedded Systems)

Common formats:

- `.bin`
    
- `.img`
    
- `.hex`
    

Often contain:

- Raw memory dumps
    
- Bootloaders
    
- Embedded OS
    

Used in:

- IoT hacking
    
- Router exploitation
    
- Hardware reverse engineering
    

---

# Comparison Table

|Format|OS|Main Use|
|---|---|---|
|PE|Windows|Desktop apps, malware|
|ELF|Linux|Servers, tools|
|Mach-O|macOS|Apple ecosystem|
|COFF|Various|Object files|
|a.out|Old Unix|Historical|
|DEX|Android|Mobile apps|
|WASM|Web|Browser execution|
|Raw BIN|Embedded|Firmware|

---

# Key Concepts Across All Formats

## Entry Point

Address where execution begins.

## Sections / Segments

Logical divisions (code, data, linking info).

## Symbols

Function and variable names (may be stripped).

## Dynamic Linking

External libraries loaded at runtime.

## Static Linking

All code included inside binary.

---

# What You Actually Need to Master (Practical Priority)

If you're serious about RE:

1. **ELF** (Linux world)
    
2. **PE** (Malware world)
    
3. **Mach-O** (if touching macOS/iOS)
    
4. **DEX** (if touching Android)
    

The rest are situational.

---

# Summary

Executable formats define:

- How code is stored
    
- Where execution starts
    
- How libraries are linked
    
- How memory is organized
    

Understanding them lets you:

- Navigate binaries efficiently
    
- Find vulnerabilities
    
- Analyze malware
    
- Build exploits