---
# Image generation provider
# Options: svg, dalle-3, gemini, manual
provider: svg

# SVG-specific settings (only used when provider: svg)
svg_style: minimal

# Dark mode support
# false = light mode only, true = dark mode only, both = generate both variants
dark_mode: false

# Output settings
output_path: .github/social-preview.svg
dimensions: 1280x640
include_text: true
colors: auto

# README infographic settings
infographic_output: .github/readme-infographic.svg
infographic_style: hybrid

# Upload to repository (requires gh CLI)
upload_to_repo: false
---

# GitHub Social Plugin Configuration

This configuration was created by `/github-social:setup`.

## Provider: SVG (Selected)

Claude generates clean, minimal SVG graphics directly. No API key required.
- **Pros**: Free, instant, editable, small file size (10-50KB)
- **Best for**: Professional, predictable results

## Style: Minimal

Clean, simple design with project name and subtle geometric accents.
Maximum 3-5 shapes, generous whitespace.

## Available Commands

- `/github-social:social-preview` - Generate social preview image
- `/github-social:readme-enhance` - Add badges and infographic to README
- `/github-social:repo-metadata` - Generate repository description and topics
- `/github-social:all` - Run all skills in sequence

## Command Overrides

Override any setting via command flags:
```bash
/github-social:social-preview --provider=dalle-3 --dark-mode
/github-social:readme-enhance --provider=gemini
```

## Reconfigure

Run `/github-social:setup` again to modify these settings.
