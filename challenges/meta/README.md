# META - Image Metadata Analysis

## Challenge Information

| Field | Details |
|---|---|
| Platform | Blue Team Labs Online (BTLO) |
| Challenge | META |
| Category | Digital Forensics / Metadata Analysis |
| Environment | Linux |
| Completion Date | 23 September 2026 |
| Completion Time | 11:31 |
| Training Context | BTL1 (Blue Team Level 1) preparation |

## Overview

This challenge focuses on extracting and analysing image metadata to identify information about the origin of photographs and assess the reliability of embedded metadata.

The investigation uses ExifTool to inspect image properties and Reverse Image Search to corroborate the location suggested by the visual content.

## Objectives

- Identify the camera model used to capture an image.
- Determine the original date and time of the photograph.
- Extract a comment embedded in image metadata.
- Identify the likely location using Reverse Image Search.
- Understand why metadata should be treated as evidence that requires verification.

## Environment and Tools

All documented practical work was performed using Linux.

| Tool | Purpose |
|---|---|
| ExifTool | Extract and inspect image metadata |
| Google Images / Google Lens | Reverse Image Search |
| Yandex Images | Alternative reverse image search |
| `grep` | Filter relevant metadata fields |
| `unzip` | Extract the supplied archive |

## Preparation

The challenge archive was extracted using the password provided with the challenge materials.



ExifTool was installed using:

```bash
sudo apt update
sudo apt install libimage-exiftool-perl -y
```

## Investigation Workflow

### 1. Inspecting Image Metadata

The metadata of the first image was inspected with:

```bash
exiftool uploaded_1.JPG
```

This command displays available metadata fields, including camera information, timestamps, comments, and potentially GPS-related information.

### 2. Identifying the Camera Model

To locate the camera model:

```bash
exiftool uploaded_1.JPG | grep -i "Model"
```

**Result:** Canon EOS 550D

### 3. Identifying the Capture Date and Time

The date-related metadata fields were inspected with:

```bash
exiftool uploaded_1.JPG | grep -i "Date"
```

The relevant timestamp was found in the image metadata.

**Result:** `2021:11:02 13:20:23`

### 4. Extracting the Image Comment

The comment field was extracted directly:

```bash
exiftool -Comment uploaded_1.JPG
```

**Result:**

`relying on altered metadata to catch me?`

This comment suggests that the metadata may have been deliberately manipulated.

### 5. Identifying the Location

The second image was submitted to Reverse Image Search using Google Images and/or Yandex Images.

This step was important because embedded GPS information can be modified and should not automatically be considered reliable.

**Result:** Kathmandu

The location was identified through visual search rather than relying exclusively on embedded GPS metadata.

## Results

| Question | Finding |
|---|---|
| Camera model | Canon EOS 550D |
| Date and time | 2021:11:02 13:20:23 |
| Comment on the first image | `relying on altered metadata to catch me?` |
| Likely location | Kathmandu |

## Key Findings and Forensic Considerations

- Image metadata can reveal useful information about a photograph, including camera model, timestamps, comments, and GPS coordinates.
- ExifTool provides a practical way to inspect these metadata fields.
- Metadata is not inherently trustworthy: an image's GPS information and other fields can be altered.
- Reverse Image Search can provide independent visual context and help corroborate a suspected location.
- Findings based on visual search should be treated as corroborating evidence, not as proof of the exact physical location or identity of the photographer.

## Conclusion

The investigation identified the camera model as Canon EOS 550D, established the recorded capture timestamp as 2 November 2021 at 13:20:23, and extracted a comment indicating possible metadata manipulation.

Reverse Image Search of the second image identified Kathmandu as the likely location.

The main forensic lesson is that metadata should be examined carefully and independently corroborated whenever its authenticity is uncertain.

## Evidence

- `screenshot.png` - BTLO challenge screenshot.

