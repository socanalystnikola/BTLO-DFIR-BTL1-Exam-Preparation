# Squid Game - Multi-Layer Steganography

## Challenge Information

| Field | Details |
|---|---|
| Platform | Blue Team Labs Online (BTLO) |
| Category | Digital Forensics / Steganography |
| Environment | Kali Linux |
| Completion Date | 23 September 2026 |
| Completion Time | 09:15 |
| Training Context | BTL1 preparation |

## Overview

A multi-stage steganography investigation using an invitation image and an extracted PNG file. The workflow combines file identification, metadata inspection, hidden-file extraction, image-layer analysis, RGB pixel inspection, and ASCII decoding.

## Tools

| Tool | Purpose |
|---|---|
| `file`, ExifTool | File and metadata inspection |
| Steghide | Hidden-file extraction |
| Stegsolve | Image-layer and colour-channel analysis |
| PixSpy | RGB pixel inspection |
| dCode | ASCII decoding |

## Investigation Workflow

### 1. File Identification

```bash
cd ~/Downloads
file "Invitation Card.jpg"
exiftool "Invitation Card.jpg"
```

### 2. Passphrase Identification

Online research related to the invitation card identified the phone number **86504006**. This value was used as the Steghide passphrase.

### 3. Hidden File Extraction

```bash
steghide extract -sf "Invitation Card.jpg"
```

Passphrase: `86504006`

Extracted file: **Dalgona.png**

### 4. Image-Layer Analysis

The extracted PNG was opened in Stegsolve. Inspection of image layers, including Gray Bits, revealed the hint **Red Pixel**.

### 5. RGB Pixel Analysis

PixSpy was used to inspect relevant pixels in Dalgona.png. The red-channel (R) values were collected and decoded using the dCode ASCII decoder.

## Results

| Question | Answer |
|---|---|
| Invitation phone number | `86504006` |
| Extracted file | `Dalgona.png` |
| Hidden hint | `Red Pixel` |
| Final flag | `SBT{S4v3_My4nm4r_Guy5}` |

## Final Flag

```text
SBT{S4v3_My4nm4r_Guy5}
```

## Key Findings

- Steghide extracts data embedded in supported image files.
- Stegsolve reveals information hidden in image layers.
- RGB channel values can encode information.
- ASCII decoding converts numeric values into readable text.
- Multi-stage steganography can require several complementary tools.

## Conclusion

The investigation identified the invitation phone number, extracted Dalgona.png using Steghide, discovered the Red Pixel hint in Stegsolve, and analysed red-channel pixel values to recover the final flag.

## Evidence

- `screenshot.png` - BTLO challenge screenshot.
