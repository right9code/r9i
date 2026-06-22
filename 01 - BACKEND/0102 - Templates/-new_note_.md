<%*
let inputTitle = await tp.system.prompt("📝 Enter title for the note (leave empty for timestamp):");
if (!inputTitle || inputTitle.trim() === "") {
  inputTitle = tp.date.now("YYYY-MM-DD HH-mm-ss");
}
await tp.file.rename(inputTitle);
-%>
---
type:
  - "notes"
date: <% tp.date.now("YYYY-MM-DD") %>
timestamp: <% tp.date.now("YYYY-MM-DD HH:mm:ss") %>
tags:
  - "#xx"
  - "#yy"
  - "#zz"
---
