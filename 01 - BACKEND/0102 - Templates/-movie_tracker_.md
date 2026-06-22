<%*
// =====================
// MOVIE TRACKER TEMPLATE (series + auto movie_number)
// =====================

// --- CONFIGURATION ---
const apiKey = "e0a86214a4949afb1d2a0beddde2a203";
const targetFolder = "11 - PRIVATE/1101 - CC/Movies/";
const defaultCoverUrl = "https://www.themoviedb.org/assets/2/v4/glyphicons/basic/glyphicons-basic-4-user-grey-d8fe957375e70239d6abdd5492136659632908079dd20405d225TCb85E94f094.svg";

const notice = (msg) => new Notice(msg, 6000);

if (!apiKey || apiKey.trim() === "") {
  notice("❌ TMDb API key is missing. Please add it to the template configuration.");
  return;
}

// --- VARIABLES ---
let title, coverUrl, synopsis, genres = [], releaseDate = "", runtime = "";
let series = "";
let movieNumber = "";          // will try to auto-fill
let collectionId = "";         // for debugging / optional YAML
let selectedMovieId = null;    // needed to compute position
let isManual = false;

// --- PROMPT ---
const query = await tp.system.prompt("🎬 Enter the movie title to search for:");
if (!query) { notice("❌ No title entered. Template cancelled."); return; }

notice(`🔍 Searching for "${query}" on TMDb...`);

// --- SEARCH ---
let searchResults = [];
try {
  const searchResponse = await requestUrl(
    `https://api.themoviedb.org/3/search/movie?api_key=${apiKey}&query=${encodeURIComponent(query)}`
  );
  searchResults = searchResponse.json?.results ?? [];
} catch (error) {
  notice("⚠️ Error connecting to TMDb API. Switching to manual entry.");
  isManual = true;
}

// --- SELECT + DETAILS ---
if (!isManual) {
  if (searchResults.length > 0) {
    const manualEntryText = "✏️ --- My Movie Isn't Listed (Enter Manually) ---";
    const manualEntrySentinel = { id: "MANUAL_ENTRY" };

    const movieChoices = [
      manualEntryText,
      ...searchResults.map(m => {
        const year = m.release_date ? `(${m.release_date.substring(0, 4)})` : "";
        return `🎬 ${m.title} ${year}`;
      })
    ];
    const movieData = [manualEntrySentinel, ...searchResults];

    const selectedMovie = await tp.system.suggester(
      movieChoices,
      movieData,
      true,
      "🎬 Select the correct movie, or choose manual entry:"
    );

    if (!selectedMovie) {
      notice("❌ Selection cancelled. Switching to manual entry.");
      isManual = true;
    } else if (selectedMovie.id === "MANUAL_ENTRY") {
      notice("✏️ Switching to manual entry.");
      isManual = true;
    } else {
      selectedMovieId = selectedMovie.id;

      notice(`🎬 Fetching details for ${selectedMovie.title}...`);
      const detailsResponse = await requestUrl(
        `https://api.themoviedb.org/3/movie/${selectedMovieId}?api_key=${apiKey}`
      );
      const details = detailsResponse.json;

      title = details.title;
      coverUrl = details.poster_path ? `https://image.tmdb.org/t/p/w500${details.poster_path}` : defaultCoverUrl;
      synopsis = details.overview || "No synopsis available.";
      genres = (details.genres || []).map(g => g.name);
      releaseDate = details.release_date || "";
      runtime = details.runtime ? `${details.runtime} min` : "";

      // Series from belongs_to_collection. [web:23]
      if (details.belongs_to_collection?.name) series = details.belongs_to_collection.name;
      if (details.belongs_to_collection?.id) collectionId = String(details.belongs_to_collection.id);

      // Auto movie number using collection parts (extra API call). [web:26]
      if (collectionId) {
        try {
          const colResp = await requestUrl(
            `https://api.themoviedb.org/3/collection/${collectionId}?api_key=${apiKey}`
          );
          const parts = colResp.json?.parts ?? [];

          // Sort parts by release_date; TMDb parts may not be pre-sorted. [web:21]
          const sorted = parts.slice().sort((a, b) =>
            (a.release_date || "9999-99-99").localeCompare(b.release_date || "9999-99-99")
          );

          const idx = sorted.findIndex(p => p.id === selectedMovieId);
          if (idx >= 0) movieNumber = idx + 1; // 1-based
        } catch (e) {
          notice("⚠️ Could not fetch series list to auto-calc movie number.");
        }
      }
    }
  } else {
    notice("❌ Movie not found online. Switching to manual entry.");
    isManual = true;
  }
}

// --- MANUAL ENTRY ---
if (isManual) {
  title = await tp.system.prompt("🎬 Title:", query);
  if (!title) { notice("❌ No title entered. Template cancelled."); return; }

  const coverUrlInput = await tp.system.prompt("🖼️ Cover Image URL (leave blank for default):");
  coverUrl = (coverUrlInput && coverUrlInput.trim() !== "") ? coverUrlInput.trim() : defaultCoverUrl;

  synopsis = await tp.system.prompt("📝 Synopsis (can be multiline):") || "No synopsis available.";

  const genreInput = await tp.system.prompt("🏷️ Genre(s), comma-separated:");
  genres = genreInput ? genreInput.split(",").map(g => g.trim()).filter(Boolean) : [];

  releaseDate = await tp.system.prompt("📅 Release date (YYYY-MM-DD, optional):") || "";

  const runtimeInput = await tp.system.prompt("⏱️ Runtime (minutes, optional):") || "";
  runtime = runtimeInput ? `${parseInt(runtimeInput, 10)} min` : "";

  series = await tp.system.prompt("🧩 Series (leave blank if none):") || "";

  if (series.trim() !== "") {
    const mn = await tp.system.prompt("🔢 Movie number in series (leave blank if not numbered):");
    const n = parseInt(mn, 10);
    movieNumber = Number.isFinite(n) ? n : "";
  }
}

// --- EDIT/CONFIRM SERIES + MOVIE NUMBER ---
series = await tp.system.prompt("🧩 Series (auto-detected, edit if needed):", series || "") || "";

if (series.trim() !== "") {
  const defaultNum = (movieNumber !== "" && movieNumber !== null) ? String(movieNumber) : "";
  const movieNumberInput = await tp.system.prompt("🔢 Movie number in series (auto if possible, edit if needed):", defaultNum);
  const n = parseInt(movieNumberInput, 10);
  movieNumber = Number.isFinite(n) ? n : "";
} else {
  movieNumber = "";
}

// --- TRACKING ---
const userStatusId = await tp.system.suggester(
  ["📋 Plan to Watch", "✅ Completed", "⏸️ On-Hold", "❌ Dropped"],
  ["queued", "finished", "onhold", "dropped"],
  false,
  "📊 What is your status for this movie?"
);

const rating = await tp.system.prompt("⭐ Your rating (e.g., 8/10, or leave blank):");

// --- YAML ---
const yaml = `---
type: movie
title: "${title}"
series: "${series || ''}"
movie_number: ${movieNumber !== "" ? movieNumber : '""'}
cover_url: "${coverUrl}"
status: ${userStatusId}
rating: ${rating ? `"${rating}"` : '""'}
genres: ${JSON.stringify(genres)}
release_date: ${releaseDate ? `"${releaseDate}"` : '""'}
runtime: ${runtime ? `"${runtime}"` : '""'}
tmdb_collection_id: ${collectionId ? `"${collectionId}"` : '""'}
date_started: ${userStatusId !== 'queued' ? `"${tp.date.now("YYYY-MM-DD")}"` : '""'}
date_finished: ${userStatusId === 'finished' ? `"${tp.date.now("YYYY-MM-DD")}"` : '""'}
tags: [movie]
---
`;

// --- BODY ---
const mainCheckbox = (userStatusId === "finished") ? "[x]" : "[ ]";
let content = `- ${mainCheckbox} **${title}**\n`;

if (series && series.trim() !== "") {
  const seriesText = (movieNumber !== "")
    ? `\t- **Series**: [[${series}]] (Movie ${movieNumber})\n`
    : `\t- **Series**: [[${series}]]\n`;
  content += seriesText;
}

content += `\n\t<div style="text-align: center;">\n\t\t<img src="${coverUrl}" width="140">\n\t</div>\n\n`;

if (releaseDate) content += `\t- **Release date**: ${releaseDate}\n`;
if (runtime) content += `\t- **Runtime**: ${runtime}\n`;
content += `\t- **Genres**: ${genres.length ? genres.join(", ") : "N/A"}\n`;

const synopsisLines = (synopsis || "").split("\n").filter(l => l.trim() !== "");
if (synopsisLines.length > 0 && synopsis !== "No synopsis available.") {
  content += `\t- **Synopsis**: ${synopsisLines[0]}\n`;
  if (synopsisLines.length > 1) {
    for (let i = 1; i < synopsisLines.length; i++) content += `\t\t${synopsisLines[i]}\n`;
  }
} else {
  content += `\t- **Synopsis**: No synopsis available.\n`;
}

content += `\n---\n## My Notes:\n- _ `;

// --- FILE OPS ---
const safeTitle = title.replace(/[\\/:"*?<>|]+/g, "-");
await tp.file.rename(safeTitle);
if (targetFolder && targetFolder.trim() !== "") {
  notice(`📁 Moving note to ${targetFolder}...`);
  await tp.file.move(targetFolder + safeTitle);
}

notice("✅ Movie note created successfully!");
tR += yaml + content;
%>
