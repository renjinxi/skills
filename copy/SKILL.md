---
name: copy
description: Copy what the user wants (a command, snippet, previous answer, file content, etc.) to the system clipboard.
disable-model-invocation: true
---

Copy the thing I want to my system clipboard.

## What to copy

- If I said what to copy (in the arguments or my message), copy exactly that.
- If I didn't, copy the most recent thing you produced that I'd plausibly want to paste: the last command, code block, message draft, or answer.
- Copy the raw content only. No surrounding explanation, no markdown code fences, unless I ask for them.
- If it's a file, copy the file's content (or the part I named), not its path — unless I asked for the path.
- If it's genuinely ambiguous between several candidates, ask me which one in one short line.

## How to copy

Pipe the content into the platform clipboard tool, using a quoted heredoc so nothing gets expanded or escaped:

```bash
pbcopy <<'__CLIP_EOF__'
<content>
__CLIP_EOF__
```

- macOS: `pbcopy`
- Linux Wayland: `wl-copy`; X11: `xclip -selection clipboard` (or `xsel --clipboard --input`)
- Windows / WSL: `clip.exe`

For a file, pipe it directly instead of retyping: `pbcopy < path/to/file`.

If the content itself contains the line `__CLIP_EOF__`, pick another delimiter.

## After copying

Verify by checking the clipboard is non-empty (e.g. `pbpaste | wc -c`), then reply with one short line saying what was copied (e.g. "已复制：git 命令，1 行"). Don't echo the full content back.
