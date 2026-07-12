# TOOLS.md - Local Notes

Skills define _how_ tools work. This file is for _your_ specifics — the stuff that's unique to your setup.

## What Goes Here

Things like:

- Camera names and locations
- SSH hosts and aliases
- Preferred voices for TTS
- Speaker/room names
- Device nicknames
- Anything environment-specific

## Examples

```markdown
### Cameras

- living-room → Main area, 180° wide angle
- front-door → Entrance, motion-triggered

### SSH

- home-server → 192.168.1.100, user: admin

### TTS

- Preferred voice: "Nova" (warm, slightly British)
- Default speaker: Kitchen HomePod
```

## Why Separate?

Skills are shared. Your setup is yours. Keeping them apart means you can update skills without losing your notes, and share skills without leaking your infrastructure.

---

Add whatever helps you do your job. This is your cheat sheet.

## KiCad Environment

- **KiCad version:** 10.0.4 (Windows native)
- **Python:** `C:\Program Files\KiCad\10.0\bin\python.exe` (3.11.5)
- **Symbol libraries:** `C:\Program Files\KiCad\10.0\share\kicad\symbols\`
- **kicad-sch-api installed:** `C:\Users\user\AppData\Roaming\Python\Python311\site-packages\kicad_sch_api`
- **Active project:** `C:\Users\user\Desktop\warehouse_keycard_rev2\warehouse_keycard_rev2\`

## Related

- [Agent workspace](/concepts/agent-workspace)
