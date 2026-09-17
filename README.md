# DitherDeck — E-Ink Wallpaper Generator

Transform your photos into beautifully dithered wallpapers optimized for e-ink displays, Kindle devices, and smartwatches. DitherDeck is a minimalist, privacy-preserving webapp that works entirely in your browser.

## Features

✨ **Hardware-Specific Optimization**
- Xteink X4: 4-level grayscale BMP (480 × 800)
- Kindle Paperwhite: 16-level grayscale PNG (1264 × 1680)
- Apple Watch 45mm/Ultra: Full-color OLED PNG (410 × 502) with battery-saving true black
- Easily add more devices via custom color profiles

🎨 **Advanced Image Processing**
- Floyd-Steinberg dithering for smooth gradients on limited color palettes
- Real-time contrast & brightness adjustment
- Drag-to-pan and zoom controls for precise cropping
- Live preview with device-specific aspect ratio

📦 **Batch Operations & Presets**
- Upload multiple images at once
- Save and load editor presets (settings templates)
- Apply presets to multiple wallpapers in one action
- Download all processed images as a ZIP file organized by device

🌙 **Dark/Light Theme**
- Toggle between dark and light UI themes
- Preference persists across sessions

🔗 **Share Wallpapers**
- Generate shareable links with device info encoded
- Custom messaging for WhatsApp, Email, and clipboard
- Viral loop: share your creations with friends

🔒 **Privacy-First Design**
- 100% client-side processing (no server uploads)
- All image data stays on your device
- No tracking, no analytics, no accounts required

## Getting Started

### Using DitherDeck

1. **Visit** [ditherdeck.vetchalabs.com](https://ditherdeck.vetchalabs.com)
2. **Select your device** (Xteink, Kindle, Apple Watch)
3. **Upload photos** from your device gallery
4. **Adjust & crop** using the interactive editor (drag to pan, zoom to crop)
5. **Tune settings**: contrast, brightness, dithering style
6. **Save & download** individual wallpapers or entire ZIP archive
7. **Share** links with friends via WhatsApp, Email, or clipboard

### Keyboard & Touch Shortcuts

- **Drag in preview**: Pan the image
- **Zoom slider**: Zoom in/out for cropping
- **Contrast slider**: Adjust image contrast (0.5x – 2.0x)
- **Brightness slider**: Shift luminance (−80 to +80)
- **Dither toggle**: Choose Floyd-Steinberg (smooth) or threshold (sharp)

## Device Specifications

| Device | Dimensions | Colors | Output Format | Use Case |
|--------|-----------|--------|---------------|----------|
| **Xteink X4** | 480 × 800 | 4-level gray | BMP (uncompressed) | E-ink tablet sleep screen |
| **Kindle Paperwhite** | 1264 × 1680 | 16-level gray | PNG | Kindle lock/screensaver |
| **Apple Watch** | 410 × 502 | Full RGB + true black | PNG | Watch face background (battery-optimized) |
| **Custom** | User-defined | 2–256 colors (Hex palette) | BMP or PNG | Any device with custom palette |

## Color Processing Pipeline

### Grayscale E-Ink (Xteink, Kindle)
1. Load input image
2. Convert RGB → Luminance (ITU-R BT.601 standard: 0.299R + 0.587G + 0.114B)
3. Apply user adjustments (contrast & brightness)
4. Quantize to palette (4-level or 16-level gray)
5. **Optional**: Apply Floyd-Steinberg error diffusion for dithering
6. Output grayscale image in device format

### OLED True Color (Apple Watch)
1. Load input image
2. Apply per-channel contrast & brightness
3. Battery optimization: Clamp RGB values < 14 to pure black (saves OLED power)
4. Output full RGB PNG

## Adding New Device Presets

### Option 1: Custom Color Profile (UI-based, no coding required)
1. Select **Custom** from the device dropdown
2. Enter dimensions (width × height)
3. Paste hex colors for your palette (e.g., `#000000`, `#FFFFFF`)
4. Choose output format (BMP or PNG)
5. Your custom profile is saved in browser localStorage

### Option 2: Code-based Addition (Developer)

Edit the `PRESETS` object in `docs/index.html`:

```javascript
const PRESETS = {
  mydevice: {
    name: "My Device",
    width: 400,
    height: 600,
    ext: "png",
    format: "png",
    mode: "gray8",  // "gray4", "gray16", "gray8", "colorOled", "colorCustom"
    folder: "my_device_folder",
    badge: "400 × 600 • 8-Level PNG",
    previewMaxH: 300
  }
};
```

Then add color transform logic in the `applyDeviceColorTransform()` function (see source code for examples).

See [DEVICE_GUIDE.md](DEVICE_GUIDE.md) for detailed instructions.

## Saving & Managing Presets

### Editor Presets
Save your favorite adjustment combinations:
1. Open the editor modal
2. Tune contrast, brightness, dither settings
3. Click **"Save Preset"** (stores: name, contrast, brightness, dither mode)
4. Next time, select your preset from dropdown before editing

### Batch Applying Presets
1. Select multiple images (checkboxes)
2. Choose preset from dropdown
3. Click **Apply to Selected**
4. All images get the same settings instantly

## Sharing Wallpapers

### Via Share Button

On each wallpaper card:
1. Click **Share**
2. Add optional custom message (prepends to default)
3. Choose platform:
   - **Copy to Clipboard**: Link + message as text
   - **WhatsApp**: Opens WhatsApp Web with pre-filled message (native app on mobile)
   - **Email**: Opens email client with subject & body

### Encoded URL Format

Shared links include device info as query params:
```
ditherdeck.vetchalabs.com/?device=xteink&preset=mypreset
```

Recipients visit the link, which pre-loads your device selection.

## Theme Preferences

Click the **☀️/🌙 toggle** in the header to switch themes:
- **Dark** (default): Better for evening use, reduces eye strain
- **Light**: Better for daytime use in bright environments

Your preference is stored in browser localStorage and persists across sessions.

## Download Options

### Single Wallpaper
Click **Save .BMP** (or .PNG) on any card to download individually.

### Batch Download
1. Upload and process multiple images
2. Click **Download ZIP** to get all wallpapers
3. ZIP is organized by device folder (e.g., `/sleep/`, `/kindle_wallpapers/`, `/watch_faces/`)

## Technical Details

### Architecture
- **Frontend**: Pure HTML, CSS, JavaScript (no framework)
- **Dependencies**: JSZip (for ZIP creation), FileSaver (for downloads)
- **Processing**: All image transforms run client-side in browser Web Workers (GPU-accelerated on supported devices)
- **Storage**: Browser localStorage for presets & theme preference
- **Hosting**: GitHub Pages (automatic deployment on code push)

### Browser Compatibility
- ✅ Chrome/Edge 60+
- ✅ Firefox 55+
- ✅ Safari 12+
- ✅ Mobile browsers (iOS Safari, Chrome Mobile)
- ✅ Tablet browsers (iPad, Android)

**Note**: Requires Canvas API support for image processing. No polyfills included (modern browsers only).

### File Formats

**BMP24 (Uncompressed)**
- Used for: Xteink X4, custom devices
- Advantages: Fast loading on resource-constrained devices, no decompression overhead
- Format: 24-bit RGB, bottom-up scanlines, 4-byte row padding

**PNG (Portable Network Graphics)**
- Used for: Kindle, Apple Watch, custom devices
- Advantages: Compressed, preserves quality, widely supported
- Format: PNG-8 (indexed color, grayscale) or PNG-24 (RGB/RGBA)

## Troubleshooting

### Image looks pixelated or dithered too heavily
→ Reduce **Dither Style** or switch to "Flat Threshold / No Dither"

### Colors are inverted or too dark
→ Adjust **Brightness** slider (−80 to +80)

### Image is cropped incorrectly
→ Use **Zoom slider** to adjust crop area; drag preview to reposition

### Download fails or ZIP is empty
→ Try reloading the page; check browser console for errors

### Presets aren't saving
→ Check if browser allows localStorage (Settings → Privacy → Site Settings → Cookies)

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### How to Contribute
- **Bug reports**: Open an issue with reproduction steps
- **Feature requests**: Discuss in an issue before submitting PR
- **Code**: Fork repo, create feature branch, submit PR with tests
- **Device support**: Add new device presets (see DEVICE_GUIDE.md)

## License

MIT License — See LICENSE file for details. Use DitherDeck freely for personal and commercial projects.

## Support

- 🐛 **Bug reports**: [GitHub Issues](https://github.com/kishorevetcha/ditherdeck/issues)
- 💬 **Questions**: Open a discussion or comment on related issues
- 🎨 **Showcase**: Share your wallpapers! Use #DitherDeck on social media

## Roadmap

- [x] Core e-ink rendering (Xteink, Kindle)
- [x] Apple Watch OLED support
- [x] Dark/Light theme
- [x] Batch processing
- [x] Preset management
- [x] Share feature
- [ ] Custom color profiles UI
- [ ] Mobile app (React Native)
- [ ] Integration with device managers (auto-push to devices)
- [ ] Community gallery (showcase wallpapers)
- [ ] Advanced dithering algorithms (Bayer, error-diffusion variants)

## Credits

Built with ❤️ at [Vetchalabs](https://vetchalabs.com)

Floyd-Steinberg dithering algorithm inspired by classic image processing research.

---

**Ready to create your first wallpaper?** → [Visit DitherDeck](https://ditherdeck.vetchalabs.com)
