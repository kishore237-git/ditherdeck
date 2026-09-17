# Contributing to DitherDeck

Thank you for your interest in contributing! We welcome bug reports, feature requests, and code contributions.

## Code of Conduct

Be respectful, inclusive, and constructive in all interactions. We're committed to fostering a welcoming community.

## Getting Started

### Local Development

1. **Fork and clone** the repository:
   ```bash
   git clone https://github.com/YOUR_USERNAME/ditherdeck.git
   cd ditherdeck
   ```

2. **No build step required** — DitherDeck is a static single-page app. Simply:
   ```bash
   # Option 1: Open in browser
   open docs/index.html
   
   # Option 2: Serve locally (Python 3)
   python3 -m http.server 8000
   # Visit http://localhost:8000/docs/
   ```

3. **Make your changes** in `docs/index.html` or documentation files

4. **Test** in multiple browsers (Chrome, Firefox, Safari, mobile)

### File Structure

```
ditherdeck/
├── docs/
│   ├── index.html          # Main application (HTML + CSS + JS)
│   └── CNAME               # Custom domain config
├── README.md               # User documentation
├── CONTRIBUTING.md         # This file
├── DEVICE_GUIDE.md         # Device preset developer guide
├── LICENSE                 # MIT License
└── .github/
    └── workflows/
        └── pages.yml       # GitHub Pages auto-deploy
```

## Types of Contributions

### Bug Reports

1. **Check existing issues** to avoid duplicates
2. **Create a new issue** with:
   - Clear title describing the bug
   - Steps to reproduce
   - Expected vs. actual behavior
   - Browser/device info (User-Agent)
   - Screenshots if applicable

Example:
```
Title: Dithering not applied to Kindle Paperwhite wallpapers

Steps:
1. Select Kindle Paperwhite
2. Upload image
3. Select "Floyd-Steinberg Diffusion" dither
4. Click Edit

Expected: Dithering visible in preview
Actual: Preview shows no dithering pattern

Browser: Chrome 120 on macOS 14
```

### Feature Requests

1. **Open an issue** with tag `[FEATURE]` in title
2. **Describe the use case** — why this feature matters
3. **Suggest implementation** (optional but helpful)
4. **Gather feedback** before coding

Example:
```
Title: [FEATURE] Support for Kobo e-readers

Use Case: Kobo devices use a 16-level grayscale similar to Kindle but with 
different screen dimensions (1072 × 1404). Users want to optimize wallpapers 
for Kobo devices without manually resizing.

Suggested Implementation:
- Add "Kobo Libra 2" preset to PRESETS
- Reuse gray16 mode from Kindle
- Add to device dropdown
```

### Code Contributions

#### Adding a New Device Preset

**No coding required** for simple additions! Use the **Custom Color Profile** UI, or:

1. **Edit `docs/index.html`** (lines ~200–220 in PRESETS object):
   ```javascript
   mydevice: {
     name: "My E-Reader",
     width: 600,
     height: 900,
     ext: "png",
     format: "png",
     mode: "gray8",
     folder: "my_device",
     badge: "600 × 900 • 8-Level PNG",
     previewMaxH: 300
   }
   ```

2. **Add color transform** in `applyDeviceColorTransform()` function if needed (for custom palette logic)

3. **Test** thoroughly across devices (mobile, tablet, desktop)

4. **Submit PR** with:
   - Description of device (manufacturer, specs, use case)
   - Why this device should be included
   - Link to device documentation or specs
   - Test results on actual device (if possible)

See [DEVICE_GUIDE.md](DEVICE_GUIDE.md) for detailed instructions.

#### Improving UI/UX

1. **Identify the improvement** (e.g., mobile responsiveness, accessibility)
2. **Create an issue** to discuss approach
3. **Fork and implement** changes in `docs/index.html`
4. **Test** on target devices/browsers
5. **Submit PR** with screenshots/video of changes

#### Bug Fixes

1. **Link to related issue** in your PR description
2. **Explain the root cause**
3. **Show how your fix resolves it**
4. **Test** edge cases

Example PR description:
```markdown
Fixes #42 — Zoom slider doesn't update preview on touch devices

**Root Cause**: `pointermove` event listener wasn't properly tracking touches 
on devices with low DPI.

**Solution**: Changed from `clientX/clientY` to `pageX/pageY` for better 
cross-device compatibility.

**Testing**: Verified on iPhone 14, iPad Pro, Android tablet, desktop Chrome.
```

### Documentation

1. **Update README.md** if adding features
2. **Update DEVICE_GUIDE.md** if adding device presets
3. **Add inline code comments** for complex logic
4. **Keep language clear and friendly**

## Pull Request Process

1. **Create a feature branch**:
   ```bash
   git checkout -b feature/my-feature
   git checkout -b fix/issue-number
   ```

2. **Make atomic commits** with clear messages:
   ```bash
   git commit -m "Add Kobo Libra 2 device preset"
   git commit -m "Fix: dithering preview not updating on mobile"
   ```

3. **Test your changes** before pushing:
   - Browser compatibility (Chrome, Firefox, Safari)
   - Mobile responsiveness (375px, 768px, 1024px widths)
   - Touch/mouse input
   - All device presets

4. **Push and open a PR**:
   ```bash
   git push origin feature/my-feature
   ```

5. **PR title format**:
   ```
   [Feature] Add Kobo Libra 2 preset
   [Fix] Dithering not visible on Safari
   [Docs] Update README device specifications
   [Improvement] Mobile responsiveness in editor modal
   ```

6. **PR description** should include:
   - What problem does this solve?
   - How does it work?
   - Testing checklist
   - Related issues (e.g., "Fixes #42")

7. **Respond to feedback** — maintainers may request changes

8. **Squash commits** if requested (keep history clean):
   ```bash
   git rebase -i main
   ```

## Testing Checklist

Before submitting a PR, test on:

- [ ] Chrome (latest)
- [ ] Firefox (latest)
- [ ] Safari (latest on macOS + iOS)
- [ ] Mobile browser (Android Chrome)
- [ ] Tablet (iPad or Android)
- [ ] All device presets (Xteink, Kindle, Apple Watch, Custom)
- [ ] Dark and Light themes
- [ ] Image upload with multiple files
- [ ] Editor modal (drag, zoom, sliders, dither toggle)
- [ ] Batch operations (if applicable)
- [ ] Preset save/load (if applicable)
- [ ] Share feature (if applicable)
- [ ] ZIP download

## Code Style

We follow these conventions:

- **Variable names**: camelCase (`myVariable`, `currentPreset`)
- **Function names**: camelCase (`applyDeviceColorTransform()`)
- **Constants**: UPPER_SNAKE_CASE (`PRESETS`, `FLOAT32_MAX`)
- **CSS classes**: kebab-case (`.modal-content`, `.card-actions`)
- **Indentation**: 2 spaces (no tabs)
- **Semicolons**: Required
- **Comments**: Clear, concise; explain *why*, not *what*

Example:
```javascript
// Floyd-Steinberg error diffusion: spread quantization error to neighboring pixels
// for smooth dithering on limited color palettes
if (params.dither === 'fs') {
  const err = oldVal - closest;
  if (x + 1 < w) gray[idx + 1] += (err * 7) / 16;
  // ... remaining error distribution
}
```

## License

By contributing, you agree that your code will be licensed under the MIT License.

## Questions?

- 📖 Check [README.md](README.md) for usage documentation
- 🔧 Check [DEVICE_GUIDE.md](DEVICE_GUIDE.md) for device customization
- 💬 Open a GitHub issue to discuss
- 🐛 Report bugs with clear reproduction steps

Thank you for making DitherDeck better! 🎨
