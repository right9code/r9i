<%*
// --- CONFIG ---
const defaultCoverUrl = "https://islandpress.org/sites/default/files/default_book_cover_2015.jpg";
const targetFolder = "11 - PRIVATE/1101 - CC/Manhwa/";
const KITSU_BASE = "https://kitsu.io/api/edge"; // Kitsu base API [web:40]

const notice = (msg) => new Notice(msg, 6000);

// Metadata
let title = "";
let synopsis = "";
let coverUrl = defaultCoverUrl;
let genres = [];
let volumeCount = "";
let chapterCount = "";
let pubStatus = "";
let startDate = "";
let endDate = "";

// Staff (manual)
let author = "";
let artist = "";
let staffLine = "";

let isManual = false;

// Tracking (same as your books)
let userStatusId = "queued";
let rating = "";

// Progress for pre-checking
let completedCh = 0;
let completedVol = 0;

// 1) Prompt
const query = await tp.system.prompt("📚 Enter MANHWA title to search (Kitsu):");
if (!query) { notice("❌ No title entered. Template cancelled."); return; }

notice(`🔍 Searching Kitsu for "${query}"...`);

let results = [];
try {
  const url = `${KITSU_BASE}/manga?filter%5Btext%5D=${encodeURIComponent(query)}&page%5Blimit%5D=20`; // [web:40]
  const response = await requestUrl({
    url,
    method: "GET",
    headers: { "Accept": "application/vnd.api+json" }
  });
  results = response?.json?.data ?? [];
} catch (e) {
  console.log(e);
  notice("⚠️ Could not reach Kitsu API. Switching to manual entry.");
  isManual = true;
}

// 2) Manhwa-only filter
let manhwaResults = [];
if (!isManual) {
  manhwaResults = results.filter(x => (x?.attributes?.subtype || "").toLowerCase() === "manhwa");
  if (!manhwaResults.length) {
    notice("❌ No MANHWA results found on Kitsu. Switching to manual entry.");
    isManual = true;
  }
}

// 3) Choose result
if (!isManual) {
  const manualEntryText = "✏️ --- Not listed / Enter manually ---";
  const manualEntrySentinel = { id: "MANUAL_ENTRY" };

  const choices = [
    manualEntryText,
    ...manhwaResults.map(item => {
      const a = item.attributes || {};
      const t = a.canonicalTitle || query;
      const y = a.startDate ? `(${String(a.startDate).slice(0, 4)})` : "";
      return `📘 ${t} ${y}`.trim();
    })
  ];

  const data = [manualEntrySentinel, ...manhwaResults];

  const selected = await tp.system.suggester(choices, data, true, "📚 Select the correct MANHWA:");
  if (!selected) { notice("❌ Selection cancelled. Switching to manual entry."); isManual = true; }
  else if (selected.id === "MANUAL_ENTRY") { notice("✏️ Switching to manual entry."); isManual = true; }
  else {
    const a = selected.attributes || {};
    title = a.canonicalTitle || query;
    synopsis = a.synopsis || "No synopsis available.";
    coverUrl = a.posterImage?.large || a.posterImage?.medium || a.posterImage?.small || defaultCoverUrl;

    volumeCount = (a.volumeCount ?? "");
    chapterCount = (a.chapterCount ?? "");
    pubStatus = (a.status || "");
    startDate = (a.startDate || "");
    endDate = (a.endDate || "");

    const genreInput = await tp.system.prompt("🏷️ Genres (comma-separated, optional):");
    genres = genreInput ? genreInput.split(",").map(g => g.trim()).filter(Boolean) : [];
  }
}

// 4) Manual entry / manual overrides
if (isManual) {
  title = await tp.system.prompt("📖 Title:", query);
  if (!title) { notice("❌ No title entered. Template cancelled."); return; }

  synopsis = await tp.system.prompt("📝 Synopsis (optional):") || "No synopsis available.";
  volumeCount = await tp.system.prompt("📚 Total volumes (optional):") || "";
  chapterCount = await tp.system.prompt("🔢 Total chapters (optional):") || "";
  pubStatus = await tp.system.prompt("📡 Publication status (optional):") || "";
  startDate = await tp.system.prompt("🗓️ Start date (YYYY-MM-DD optional):") || "";
  endDate = await tp.system.prompt("🗓️ End date (YYYY-MM-DD optional):") || "";

  const genreInput = await tp.system.prompt("🏷️ Genres (comma-separated, optional):");
  genres = genreInput ? genreInput.split(",").map(g => g.trim()).filter(Boolean) : [];

  const coverUrlInput = await tp.system.prompt("🖼️ Cover Image URL (blank = default):");
  coverUrl = (coverUrlInput && coverUrlInput.trim()) ? coverUrlInput.trim() : defaultCoverUrl;
}

// Staff prompts (always, since Kitsu doesn’t reliably provide author/artist here) [web:40]
author = await tp.system.prompt("✍️ Author/Writer (optional):") || "";
artist = await tp.system.prompt("🎨 Artist (optional):") || "";
staffLine = [author ? `Author: ${author}` : "", artist ? `Artist: ${artist}` : ""].filter(Boolean).join(" | ");

// Tracking status (same as your books)
userStatusId = await tp.system.suggester(
  ["📖 Ongoing", "✅ Finished", "⏸️ On-Hold", "❌ Dropped", "📋 Plan to Read"],
  ["ongoing", "finished", "onhold", "dropped", "queued"],
  false,
  "📊 What is your status for this manhwa?"
);

rating = await tp.system.prompt("⭐ Your rating (e.g., 9/10, or blank):");

// 5) Checklist + completed count (pre-check)
notice("📝 Checklist setup.");

const checklistMode = await tp.system.suggester(
  ["Chapters checklist", "Volumes checklist", "Skip checklist"],
  ["chapters", "volumes", "skip"],
  false,
  "📋 What checklist do you want?"
);

let checklist = "";
if (checklistMode === "chapters") {
  const lastChapterInput = await tp.system.prompt(
    "📖 Last chapter number to generate (e.g., 210):",
    chapterCount ? String(chapterCount) : "1"
  );
  const lastChapter = parseInt(lastChapterInput) || 1;

  const completedChInput = await tp.system.prompt(
    "✅ Chapters completed (pre-check up to):",
    "0"
  );
  completedCh = parseInt(completedChInput) || 0;

  checklist += "\t- ## Chapters\n";
  for (let c = 1; c <= lastChapter; c++) {
    const box = (c <= completedCh) ? "[x]" : "[ ]";
    checklist += `\t\t- ${box} Chapter ${c}\n`;
    checklist += `\t\t\t- \n`;
  }

  if (!chapterCount) chapterCount = String(lastChapter);
}

if (checklistMode === "volumes") {
  const lastVolInput = await tp.system.prompt(
    "📚 Last volume number to generate (e.g., 12):",
    volumeCount ? String(volumeCount) : "1"
  );
  const lastVol = parseInt(lastVolInput) || 1;

  const completedVolInput = await tp.system.prompt(
    "✅ Volumes completed (pre-check up to):",
    "0"
  );
  completedVol = parseInt(completedVolInput) || 0;

  checklist += "\t- ## Volumes\n";
  for (let v = 1; v <= lastVol; v++) {
    const box = (v <= completedVol) ? "[x]" : "[ ]";
    checklist += `\t\t- ${box} Volume ${v}\n`;
    checklist += `\t\t\t- \n`;
  }

  if (!volumeCount) volumeCount = String(lastVol);
}

// 6) YAML
const yaml = `---
type: manhwa
title: "${title.replace(/"/g, '\\"')}"
author: "${(author || "").replace(/"/g, '\\"')}"
artist: "${(artist || "").replace(/"/g, '\\"')}"
genres: ${JSON.stringify(genres)}
status: ${userStatusId}
rating: ${rating ? `"${rating.replace(/"/g, '\\"')}"` : '""'}
publication_status: "${(pubStatus || "").replace(/"/g, '\\"')}"
start_date: "${(startDate || "").replace(/"/g, '\\"')}"
end_date: "${(endDate || "").replace(/"/g, '\\"')}"
volume_count: "${volumeCount ?? ""}"
chapter_count: "${chapterCount ?? ""}"
cover_url: "${coverUrl.replace(/"/g, '\\"')}"
date_started: ${userStatusId !== 'queued' ? `"${tp.date.now("YYYY-MM-DD")}"` : '""'}
date_finished: ${userStatusId === 'finished' ? `"${tp.date.now("YYYY-MM-DD")}"` : '""'}
tags: [manhwa]
---
`;

// 7) Main note content (like your example)
const mainCheckbox = (userStatusId === "finished") ? "[x]" : "[ ]";
let content = `- ${mainCheckbox} **${title}**\n`;

if (staffLine) content += `\t- ${staffLine}\n`;
if (startDate) content += `\t- **Start date**: ${startDate}\n`;
if (pubStatus) content += `\t- **Publication status**: ${pubStatus}\n`;
if (volumeCount !== "" && volumeCount !== null) content += `\t- **Total volumes**: ${volumeCount}\n`;
if (chapterCount !== "" && chapterCount !== null) content += `\t- **Total chapters**: ${chapterCount}\n`;
if (genres && genres.length) content += `\t- **Genres**: ${genres.join(", ")}\n`;

content += `\n\t<div style="text-align: center;">\n\t\t<img src="${coverUrl}" width="120">\n\t</div>\n\n`; // HTML img works in Obsidian [web:139]

const synopsisText = (synopsis || "No synopsis available.")
  .replace(/\r/g, "")
  .split("\n")
  .map(l => l.trim())
  .filter(Boolean);

content += `\t- **Synopsis**: ${synopsisText[0] || "No synopsis available."}\n`;
for (let i = 1; i < synopsisText.length; i++) content += `\t\t${synopsisText[i]}\n`;

if (checklist) content += `\n${checklist}\n`;
content += `\n## My Notes\n- _\n\n---`;

// 8) File ops
const safeTitle = title.replace(/[\/:"*?<>|]+/g, "-").trim();
await tp.file.rename(safeTitle);

if (targetFolder && targetFolder.trim() !== "") {
  notice(`📁 Moving note to ${targetFolder}...`);
  await tp.file.move(targetFolder + safeTitle);
}

notice("✅ Manhwa note created successfully!");
tR += yaml + content;
%>
