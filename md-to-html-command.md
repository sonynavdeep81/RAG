# How to convert a MD file to HTML

```
md2html file.md
```

- Creates `file.html` next to the `.md`, with draggable table columns + top/bottom scrollbars.
- From then on, every save of the `.md` auto-updates the `.html` within ~2 seconds (background service, survives reboots).
- In VS Code: open the `.md` file and press **Ctrl+Alt+H** (runs the "Convert MD to HTML" task).
- In the Files app (Nautilus, not VS Code): right-click the file → **Scripts → Convert MD to HTML**.
- **To stop syncing a file and delete its HTML:** `md2html -r file.md` — just deleting the `.html` by hand won't work; the service will recreate it.
- List of synced files: `~/.config/md2html/files.txt`.
- You can also just ask Claude: "what was the command to convert md to html?"
