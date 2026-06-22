<%*
// --- CONFIGURATION ---
const defaultCoverUrl = "https://islandpress.org/sites/default/files/default_book_cover_2015.jpg";
const targetFolder = "11 - PRIVATE/1101 - CC/Manga/"; 
const ANILIST_ENDPOINT = "https://graphql.anilist.co"; // AniList GraphQL endpoint [web:6]

const notice = (msg) => new Notice(msg, 6000);

// Fields
let title, titleRomaji, titleNative;
let author, artist, staffLine;
let synopsis, coverUrl;
let genres = [];
let mediaStatus = "";
let format = "";
let totalChapters = "";
let totalVolumes = "";
let startYear = "";
let isManual = false;

// Tracking fields (user)
let userStatusId = "queued";
let rating = "";

// 1) Prompt
const query = await tp.system.prompt("📚 Enter manga/manhwa title to search (AniList):");
if (!query) {
  notice("❌ No title entered. Template cancelled.");
  return;
}

notice(`🔍 Searching AniList for "${query}"...`);

// 2) Search AniList (GraphQL)
let searchResults = [];
try {
  // AniList Media search through Page { media(search:, type:) } [web:6]
  const gqlQuery = `
    query ($search: String!) {
      Page(page: 1, perPage: 20) {
        media(search: $search, type: MANGA) {
          id
          title { romaji english native }
          format
          status
          startDate { year }
          chapters
          volumes
          genres
          description(asHtml: false)
          coverImage { large medium }
          staff {
            edges {
              role
              node { name { full } }
            }
          }
        }
      }
    }
  `;

  const response = await requestUrl({
    url: ANILIST_ENDPOINT,
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Accept": "application/json",
    },
    body: JSON.stringify({
      query: gqlQuery,
      variables: { search: query }
    })
  });

  searchResults = response?.json?.data?.Page?.media ?? [];
} catch (error) {
  console.log(error);
  notice("⚠️ Error connecting to AniList. Switching to manual entry.");
  isManual = true;
}

// 3) Select result (or manual entry)
if (!isManual) {
  if (searchResults && searchResults.length > 0) {
    const manualEntryText = "✏️ --- Not listed / Enter manually ---";
    const manualEntrySentinel = { id: "MANUAL_ENTRY" };

    const bookChoices = [
      manualEntryText,
      ...searchResults.map(m => {
        const t = m.title?.english || m.title?.romaji || "Untitled";
        const y = m.startDate?.year ? `(${m.startDate.year})` : "";
        const fmt = m.format ? `[${m.format}]` : "";
        return `📖 ${t} ${y} ${fmt}`.trim();
      })
    ];

    const dataChoices = [manualEntrySentinel, ...searchResults];

    const selected = await tp.system.suggester(
      bookChoices,
      dataChoices,
      true,
      "📚 Select the correct manga/manhwa (or choose manual entry):"
    );

    if (!selected) {
      notice("❌ Selection cancelled. Switching to manual entry as fallback.");
      isManual = true;
    } else if (selected.id === "MANUAL_ENTRY") {
      notice("✏️ Switching to manual entry.");
      isManual = true;
    } else {
      const tEng = selected.title?.english || "";
      const tRom = selected.title?.romaji || "";
      const tNat = selected.title?.native || "";

      title = tEng || tRom || query;
      titleRomaji = tRom;
      titleNative = tNat;

      genres = selected.genres || [];
      synopsis = selected.description ? selected.description.replace(/<br\s*\/?>/gi, "\n") : "No synopsis available.";
      coverUrl = selected.coverImage?.large || selected.coverImage?.medium || defaultCoverUrl;

      totalChapters = (selected.chapters ?? "") === null ? "" : (selected.chapters ?? "");
      totalVolumes  = (selected.volumes ?? "") === null ? "" : (selected.volumes ?? "");
      mediaStatus = selected.status || "";
      format = selected.format || "";
      startYear = selected.startDate?.year ? String(selected.startDate.year) : "";

      const edges = selected.staff?.edges || [];
      const findRole = (needle) => edges.find(e => (e.role || "").toLowerCase().includes(needle));
      const authorEdge = findRole("story") || findRole("original") || findRole("author");
      const artistEdge = findRole("art");
      author = authorEdge?.node?.name?.full || "";
      artist = artistEdge?.node?.name?.full || "";
      staffLine = [author ? `Author: ${author}` : "", artist ? `Artist: ${artist}` : ""].filter(Boolean).join(" | ");
    }
  } else {
    notice("❌ No results found on AniList. Switching to manual entry.");
    isManual = true;
  }
}

// 4) Manual entry path
if (isManual) {
  title = await tp.system.prompt("📖 Title:", query);
  if (!title) {
    notice("❌ No title entered. Template cancelled.");
    return;
  }

  author = await tp.system.prompt("✍️ Author/Writer (optional):");
  artist = await tp.system.prompt("🎨 Artist (optional):");
  staffLine = [author ? `Author: ${author}` : "", artist ? `Artist: ${artist}` : ""].filter(Boolean).join(" | ");

  const genreInput = await tp.system.prompt("🏷️ Genre(s), comma-separated:");
  genres = genreInput ? genreInput.split(",").map(g => g.trim()).filter(Boolean) : [];

  totalChapters = await tp.system.prompt("🔢 Total chapters (optional):");
  totalVolumes  = await tp.system.prompt("📚 Total volumes (optional):");
  startYear = await tp.system.prompt("🗓️ Start year (optional):");

  const coverUrlInput = await tp.system.prompt("🖼️ Cover Image URL (leave blank for default):");
  coverUrl = (coverUrlInput && coverUrlInput.trim() !== "") ? coverUrlInput.trim() : defaultCoverUrl;

  synopsis = await tp.system.prompt("📝 Synopsis (optional):") || "No synopsis available.";
  mediaStatus = await tp.system.prompt("📡 Publication status (optional, e.g., RELEASING/FINISHED):") || "";
  format = await tp.system.prompt("🏷️ Format (optional, e.g., MANGA/ONE_SHOT/NOVEL):") || "";
}

// 5) Tracking info
userStatusId = await tp.system.suggester(
  ["📖 Ongoing", "✅ Finished", "⏸️ On-Hold", "❌ Dropped", "📋 Plan to Read"],
  ["ongoing", "finished", "onhold", "dropped", "queued"],
  false,
  "📊 What is your status for this title?"
);

rating = await tp.system.prompt("⭐ Your rating (e.g., 9/10, or leave blank):");

// 6) Checklist setup (chapters)
notice("📝 Now, let's set up the chapter checklist.");
const checklistMode = await tp.system.suggester(
  ["Chapters checklist", "Volumes checklist", "Skip checklist"],
  ["chapters", "volumes", "skip"],
  false,
  "📋 What checklist do you want?"
);

let checklist = "";
if (checklistMode !== "skip") {
  if (checklistMode === "chapters") {
    const lastChapterInput = await tp.system.prompt(
      "📖 Last chapter number to generate (e.g., 120):",
      totalChapters ? String(totalChapters) : "1"
    );
    const lastChapter = parseInt(lastChapterInput) || 1;

    checklist += "\t- ## Chapters\n";
    for (let c = 1; c <= lastChapter; c++) {
      checklist += `\t\t- [ ] Chapter ${c}\n`;
      checklist += `\t\t\t- \n`;
    }
  } else if (checklistMode === "volumes") {
    const lastVolInput = await tp.system.prompt(
      "📚 Last volume number to generate (e.g., 12):",
      totalVolumes ? String(totalVolumes) : "1"
    );
    const lastVol = parseInt(lastVolInput) || 1;

    checklist += "\t- ## Volumes\n";
    for (let v = 1; v <= lastVol; v++) {
      checklist += `\t\t- [ ] Volume ${v}\n`;
      checklist += `\t\t\t- \n`;
    }
  }
}

// 7) YAML + content (removed progress_chapter + progress_volume)
const yaml = `---
type: manga
title: "${title.replace(/"/g, '\\"')}"
title_romaji: "${(titleRomaji || "").replace(/"/g, '\\"')}"
title_native: "${(titleNative || "").replace(/"/g, '\\"')}"
author: "${(author || "").replace(/"/g, '\\"')}"
artist: "${(artist || "").replace(/"/g, '\\"')}"
format: "${(format || "").replace(/"/g, '\\"')}"
publication_status: "${(mediaStatus || "").replace(/"/g, '\\"')}"
start_year: "${(startYear || "").replace(/"/g, '\\"')}"
total_chapters: "${totalChapters || ""}"
total_volumes: "${totalVolumes || ""}"
genres: ${JSON.stringify(genres)}
status: ${userStatusId}
rating: ${rating ? `"${rating.replace(/"/g, '\\"')}"` : '""'}
cover_url: "${coverUrl.replace(/"/g, '\\"')}"
date_started: ${userStatusId !== 'queued' ? `"${tp.date.now("YYYY-MM-DD")}"` : '""'}
date_finished: ${userStatusId === 'completed' ? `"${tp.date.now("YYYY-MM-DD")}"` : '""'}
tags: [manga]
---
`;

const mainCheckbox = (userStatusId === "completed") ? "[x]" : "[ ]";
let content = `- ${mainCheckbox} **${title}**\n`;

if (staffLine) content += `\t- ${staffLine}\n`;
if (startYear) content += `\t- **Start year**: ${startYear}\n`;
if (format) content += `\t- **Format**: ${format}\n`;
if (mediaStatus) content += `\t- **Publication status**: ${mediaStatus}\n`;
if (totalVolumes) content += `\t- **Total volumes**: ${totalVolumes}\n`;
if (totalChapters) content += `\t- **Total chapters**: ${totalChapters}\n`;
if (genres && genres.length) content += `\t- **Genres**: ${genres.join(", ")}\n`;

content += `\n\t<div style="text-align: center;">\n\t\t<img src="${coverUrl}" width="120">\n\t</div>\n\n`;

const synopsisText = (synopsis || "No synopsis available.")
  .replace(/\r/g, "")
  .split("\n")
  .map(l => l.trim())
  .filter(Boolean);

content += `\t- **Synopsis**: ${synopsisText[0] || "No synopsis available."}\n`;
for (let i = 1; i < synopsisText.length; i++) {
  content += `\t\t${synopsisText[i]}\n`;
}

if (checklist) content += `\n${checklist}\n`;

content += `\n## My Notes\n- _\n\n---`;

// 8) File handling
const safeTitle = title.replace(/[\/:"*?<>|]+/g, "-").trim();
await tp.file.rename(safeTitle);

if (targetFolder && targetFolder.trim() !== "") {
  notice(`📁 Moving note to ${targetFolder}...`);
  await tp.file.move(targetFolder + safeTitle);
}

notice("✅ Manga/Manhwa note created successfully!");
tR += yaml + content;
%>
