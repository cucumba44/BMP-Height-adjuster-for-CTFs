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

