# BMP-Height-adjuster-for-CTFs
Adjusts height of bmp images so you can see what's hidden in them
# BMP Height Patcher 🕵️‍

A simple Python script to fix BMP image files with incorrect height values. This is a common technique used in CTF (Capture The Flag) steganography challenges to hide data or flags at the bottom of an image.



---

## The Problem

In some steganography challenges, an image file (like a BMP) might contain hidden information. A clever way to hide this is to simply change the **height value** in the file's header to be smaller than the actual height. When an image viewer opens the file, it reads the incorrect height and only renders the top portion of the image, leaving the rest of the data hidden from view.

---

## The Solution

This script automatically detects and fixes this issue. It works by:

1.  **Reading** the entire BMP file into memory.
2.  **Ignoring** the suspect height value in the header.
3.  **Recalculating** the *true* height based on the total size of the pixel data and the image's width.
4.  **Patching** the header with a new, corrected height.
5.  **Saving** the result as a new, fully visible image.

---

## Usage

Save the script as `fix_bmp.py`. Run it from your terminal, providing the input file and desired output file name.

```bash
python fix_bmp.py <input_file.bmp> <output_file.bmp>
```

**Example:**
```bash
python fix_bmp.py corrupted.bmp recovered.bmp
```

You can also specify an optional `bump` value to add extra height, just in case.

```bash
# Adds an extra 100 pixels to the calculated height
python fix_bmp.py corrupted.bmp recovered.bmp --bump 100
```
---

## The Code

```python
import argparse

def fix_bmp_height(file_path, output_path, bump=50):
    """
    Recalculates BMP height based on pixel data size and patches the header.
    """
    try:
        with open(file_path, "rb") as f:
            bmp = bytearray(f.read())
    except FileNotFoundError:
        print(f"Error: File not found at {file_path}")
        return

    # Read width (4 bytes at offset 18) and height (4 bytes at offset 22)
    width = int.from_bytes(bmp[18:22], "little", signed=True)
    height = int.from_bytes(bmp[22:26], "little", signed=True)

    # Bits per pixel (2 bytes at offset 28)
    bpp = int.from_bytes(bmp[28:30], "little")

    # Pixel array offset (4 bytes at offset 10)
    pixel_offset = int.from_bytes(bmp[10:14], "little")

    # Pixel data size = total file size - header offset
    pixel_data_size = len(bmp) - pixel_offset

    # Row size (each row padded to multiple of 4 bytes)
    row_size = ((width * bpp + 31) // 32) * 4
    
    if row_size == 0:
        print("Error: Calculated row size is zero. Check BMP width and bpp.")
        return

    # Calculate the real height
    real_height = pixel_data_size // row_size
    new_height = real_height + bump  # Make it a bit bigger

    print(f"File: {file_path}")
    print(f"Old height in header: {height}")
    print(f"Calculated real height: {real_height}")
    print(f"New (bumped) height: {new_height}")

    # Write new height back to header
    bmp[22:26] = new_height.to_bytes(4, "little", signed=True)

    # Save modified BMP
    with open(output_path, "wb") as f:
        f.write(bmp)

    print(f"Success! Saved BMP with corrected height to {output_path}")

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Fixes BMP file height for CTF challenges.")
    parser.add_argument("file_path", help="The path to the corrupted BMP file.")
    parser.add_argument("output_path", help="The path to save the fixed BMP file.")
    parser.add_argument("--bump", type=int, default=50, help="An optional value to add to the calculated height.")
    args = parser.parse_args()
    fix_bmp_height(args.file_path, args.output_path, args.bump)
```
