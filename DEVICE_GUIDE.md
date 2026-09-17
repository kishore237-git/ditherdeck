# DEVICE_GUIDE — Adding New Device Presets to DitherDeck

This guide explains how to add support for new e-ink devices or smartwatches to DitherDeck.

## Quick Start (5 Minutes)

### No Coding Required: Use Custom Color Profile UI

1. Open DitherDeck
2. Select **"Custom"** from device dropdown (once added to Phase 2E)
3. Enter device dimensions (width × height)
4. Paste your palette colors as hex (e.g., `#000000`, `#FFFFFF`, `#808080`)
5. Choose output format (BMP or PNG)
6. Click **"Save Preset"**

Your custom device profile is automatically stored in browser localStorage and available for future sessions.

## Developer: Add Device Preset to Code

### Step 1: Gather Device Specifications

Before adding a preset, research your device:

| Spec | Example | Where to Find |
|------|---------|---------------|
| **Display Dimensions** | 480 × 800 | Device manual / specifications |
| **Color Depth** | 4-level grayscale | Device manual |
| **Supported File Formats** | BMP, PNG | Device docs or forum |
| **Color Space** | Grayscale, RGB, 16-bit | Device manual |
| **Aspect Ratio** | 0.6 (480/800) | Calculated from dimensions |
| **Typical Use** | Sleep screen, lock screen | Community forum |

### Step 2: Identify Color Mode

Determine which color mode your device uses:

#### Mode: `gray4` (4-level grayscale)
- **Example devices**: Xteink X4, most low-end e-ink
- **Colors**: 0 (black), 85 (dark gray), 170 (light gray), 255 (white)
- **Best for**: Fast rendering on old hardware
- **Dithering recommended**: Floyd-Steinberg for smooth gradients

#### Mode: `gray8` (8-level grayscale)
- **Example devices**: Boox devices, some Kindle variants
- **Colors**: 8 evenly-spaced grays (0, 36, 73, 109, 146, 182, 219, 255)
- **Best for**: Better tone gradation
- **Dithering**: Optional

#### Mode: `gray16` (16-level grayscale)
- **Example devices**: Kindle Paperwhite, Kobo Libra
- **Colors**: 16 evenly-spaced grays (0, 17, 34, 51, ..., 255)
- **Best for**: Smooth gradients on e-ink
- **Dithering**: Generally not needed

#### Mode: `colorOled` (True color OLED with battery optimization)
- **Example devices**: Apple Watch, smartwatches
- **Colors**: Full RGB (24-bit)
- **Special**: True black clipping (<14 RGB → #000000) to save battery
- **Best for**: Full-color displays

#### Mode: `colorCustom` (Custom palette, user-defined)
- **Colors**: 2–256 custom colors (hex)
- **Best for**: Specialized devices, experimentation
- **Dithering**: User-configurable

### Step 3: Add Device to PRESETS Object

Open `docs/index.html` and find the `PRESETS` object (~line 210). Add your device:

```javascript
const PRESETS = {
  xteink: { /* ... */ },
  kindle: { /* ... */ },
  applewatch: { /* ... */ },
  
  // Add your new device:
  mykobo: {
    name: "Kobo Libra 2",                    // Display name in dropdown
    width: 1072,                             // Screen width in pixels
    height: 1404,                            // Screen height in pixels
    ext: "png",                              // File extension (bmp or png)
    format: "png",                           // Format type (bmp24 or png)
    mode: "gray16",                          // Color mode (see Step 2)
    folder: "kobo_wallpapers",               // ZIP folder name
    badge: "1072 × 1404 • 16-Level PNG",     // Info badge text
    previewMaxH: 300                         // Editor preview height (pixels)
  }
};
```

### Step 4: (Optional) Add Custom Color Transform Logic

If your device needs special handling beyond standard grayscale/RGB, add logic to `applyDeviceColorTransform()`:

```javascript
function applyDeviceColorTransform(imgData, params, preset) {
  const d = imgData.data;
  const w = preset.width;
  const h = preset.height;

  // Your device: Kobo Libra 2 (16-level grayscale with dithering)
  if (preset.mode === 'gray16') {
    const gray = new Float32Array(w * h);
    
    // Convert to luminance
    for (let i = 0; i < gray.length; i++) {
      const idx = i * 4;
      let lum = 0.299 * d[idx] + 0.587 * d[idx + 1] + 0.114 * d[idx + 2];
      lum = ((lum - 128) * params.contrast) + 128 + params.brightness;
      gray[i] = Math.min(255, Math.max(0, lum));
    }

    // Quantize to 16-level palette
    const palette = Array.from({length: 16}, (_, i) => Math.round((i / 15) * 255));
    
    for (let y = 0; y < h; y++) {
      for (let x = 0; x < w; x++) {
        const idx = y * w + x;
        const oldVal = gray[idx];
        
        // Find nearest color in palette
        let closest = palette[0];
        let minDist = Math.abs(oldVal - palette[0]);
        for (let p = 1; p < palette.length; p++) {
          const dist = Math.abs(oldVal - palette[p]);
          if (dist < minDist) {
            minDist = dist;
            closest = palette[p];
          }
        }
        
        gray[idx] = closest;
        
        // Optional: Floyd-Steinberg error diffusion
        if (params.dither === 'fs') {
          const err = oldVal - closest;
          if (x + 1 < w)               gray[idx + 1] += (err * 7) / 16;
          if (x - 1 >= 0 && y + 1 < h) gray[idx + w - 1] += (err * 3) / 16;
          if (y + 1 < h)               gray[idx + w] += (err * 5) / 16;
          if (x + 1 < w && y + 1 < h)  gray[idx + w + 1] += (err * 1) / 16;
        }
      }
    }

    // Write quantized grayscale back to image data
    for (let i = 0; i < gray.length; i++) {
      const idx = i * 4;
      const val = gray[i];
      d[idx] = val;
      d[idx + 1] = val;
      d[idx + 2] = val;
      d[idx + 3] = 255;
    }
    return;
  }
  
  // ... other color modes
}
```

### Step 5: Test Your Addition

1. **Open `docs/index.html`** in a browser
2. **Select your new device** from the dropdown
3. **Upload a test image** (e.g., gradient, portrait, landscape)
4. **Edit and preview** — adjust zoom, contrast, brightness, dither
5. **Save and download** — verify file format and dimensions are correct

Test on:
- ✅ Desktop browsers (Chrome, Firefox, Safari)
- ✅ Tablet (iPad, Android)
- ✅ Mobile (iPhone, Android phone)
- ✅ Both light and dark themes

### Step 6: Verify Output File

Use ImageMagick or similar tool to verify:

```bash
# For BMP files
file my_wallpaper.bmp
identify -verbose my_wallpaper.bmp

# For PNG files
file my_wallpaper.png
identify -verbose my_wallpaper.png
```

Check:
- ✅ Dimensions match (e.g., 1072 × 1404)
- ✅ Color depth correct (e.g., 8-bit grayscale, 24-bit RGB)
- ✅ File format matches preset (BMP or PNG)
- ✅ Dithering visible (if enabled)

### Step 7: Submit PR

Create a pull request with:

1. **Title**: `[Device] Add Kobo Libra 2 preset`
2. **Description**:
   ```markdown
   Adds support for Kobo Libra 2 e-reader.
   
   **Device Specs**:
   - Dimensions: 1072 × 1404 (7.8" display)
   - Colors: 16-level grayscale
   - File format: PNG
   - Typical use: Lock screen, screensaver
   
   **Testing**:
   - [x] Verified on actual device
   - [x] Desktop browser test (Chrome, Firefox, Safari)
   - [x] Mobile browser test (iPhone, Android)
   - [x] Dithering works correctly
   - [x] File downloads successfully
   
   **References**:
   - [Kobo Libra 2 Specs](https://kobo.com/us/en/p/kobo-libra2)
   - [Community Discussion](https://example.com)
   ```

3. **Attach screenshots**:
   - Preview in editor
   - Downloaded file properties
   - Device with wallpaper (if available)

## Common Color Mode Examples

### Example 1: Kobo Clara 2 (gray8)

```javascript
kobo_clara: {
  name: "Kobo Clara 2",
  width: 758,
  height: 1024,
  ext: "png",
  format: "png",
  mode: "gray8",
  folder: "kobo_clara",
  badge: "758 × 1024 • 8-Level PNG",
  previewMaxH: 280
}
```

### Example 2: Boox Page (gray16 with dithering)

```javascript
boox_page: {
  name: "Boox Page",
  width: 1404,
  height: 1872,
  ext: "png",
  format: "png",
  mode: "gray16",
  folder: "boox_wallpapers",
  badge: "1404 × 1872 • 16-Level PNG",
  previewMaxH: 320
}
```

### Example 3: Google Pixel Watch (colorOled with battery optimization)

```javascript
pixel_watch: {
  name: "Google Pixel Watch",
  width: 384,
  height: 384,
  ext: "png",
  format: "png",
  mode: "colorOled",
  folder: "pixel_watch_faces",
  badge: "384 × 384 • OLED Color PNG",
  previewMaxH: 280
}
```

## Troubleshooting Device Addition

### Problem: Preview doesn't update when switching device
**Solution**: Check that `devicePreset.addEventListener('change', ...)` correctly calls `renderItem()` for all items.

### Problem: Downloaded file has wrong dimensions
**Solution**: Verify `width` and `height` in PRESETS match device specs exactly.

### Problem: Dithering not visible
**Solution**: 
1. Check that your color mode includes dithering logic
2. User may have selected "Flat Threshold" instead of "Floyd-Steinberg"
3. Test with a high-contrast image (gradient from black to white)

### Problem: Colors look inverted
**Solution**: Check your color transform logic — ensure RGB → Luminance conversion is correct (standard: 0.299R + 0.587G + 0.114B).

### Problem: File size too large or too small
**Solution**: 
- BMP files should be uncompressed: `(width * height * 3 + padding) + 54 bytes`
- PNG files size depends on compressibility; test with different image types

## Advanced: Custom Palette Mode

For devices with non-standard color palettes, use `mode: 'colorCustom'` and store palette in `customPalette` array:

```javascript
myweirddevice: {
  name: "Weird Device",
  width: 320,
  height: 240,
  ext: "bmp",
  format: "bmp24",
  mode: "colorCustom",
  customPalette: [
    0x000000, // Black
    0xFF0000, // Red
    0x00FF00, // Green
    0xFFFFFF  // White
  ],
  folder: "weird",
  badge: "320 × 240 • 4-Color Custom",
  previewMaxH: 200
}
```

Then in `applyDeviceColorTransform()`, add:

```javascript
if (preset.mode === 'colorCustom') {
  // Quantize to custom palette
  const palette = preset.customPalette;
  // ... quantization logic
}
```

## Resources

- **Color Space Reference**: https://en.wikipedia.org/wiki/Grayscale
- **Floyd-Steinberg Dithering**: https://en.wikipedia.org/wiki/Floyd%E2%80%93Steinberg_dithering
- **BMP File Format**: https://en.wikipedia.org/wiki/BMP_file_format
- **PNG Specification**: https://www.w3.org/TR/png/
- **ITU-R BT.601 (Luminance)**: https://en.wikipedia.org/wiki/Rec._601

## Questions?

Open a GitHub issue or discussion to ask questions about device additions — we're happy to help!

---

Happy adding! 🎨
