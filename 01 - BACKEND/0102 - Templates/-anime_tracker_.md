<%*
// --- CONFIGURATION ---
const defaultCoverUrl = "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/default.jpg";
const targetFolder = "11 - PRIVATE/1101 - CC/Anime/"; // The destination folder

// --- SCRIPT START ---
const notice = (msg) => new Notice(msg, 6000);

// --- VARIABLE DECLARATION ---
let title, coverUrl, synopsis, genres, allSeasonsData = [], allSeasonsEpisodes = [];
let isManual = false;

// 1. ALWAYS start with a search prompt with icon
const query = await tp.system.prompt("🎌 Enter the anime title to search for:");
if (!query) {
    notice("❌ No title entered. Template cancelled.");
    return;
}

notice(`🔍 Searching for "${query}"...`);
let searchResults;
try {
    const searchResponse = await requestUrl(`https://api.jikan.moe/v4/anime?q=${encodeURIComponent(query)}&sfw`);
    searchResults = searchResponse.json.data;
} catch (error) {
    notice("⚠️ Error connecting to Jikan API. Switching to manual entry.");
    isManual = true;
}

// 2. Process the search results
if (!isManual) {
    if (searchResults && searchResults.length > 0) {
        const manualEntryText = "✏️ --- My Anime Isn't Listed (Enter Manually) ---";
        const manualEntrySentinel = { mal_id: "MANUAL_ENTRY", title: "Manual" };
        const animeChoices = [manualEntryText, ...searchResults.map(a => `🎌 ${a.title_english || a.title} (${a.year || 'N/A'})`)];
        const animeData = [manualEntrySentinel, ...searchResults];
        const selectedAnime = await tp.system.suggester(animeChoices, animeData, true, "🎌 Select the correct anime, or choose manual entry:");
        
        if (!selectedAnime) {
            notice("❌ Selection cancelled. Switching to manual entry.");
            isManual = true;
        } else if (selectedAnime.mal_id === "MANUAL_ENTRY") {
            notice("✏️ Switching to manual entry.");
            isManual = true;
        } else {
            // --- AUTOMATED PATH ---
            // Fetch all seasons recursively
            async function getAnimeSeries(animeEntry, fetchedIds = new Set()) {
                if (!animeEntry || !animeEntry.mal_id || fetchedIds.has(animeEntry.mal_id)) return [];
                fetchedIds.add(animeEntry.mal_id);
                notice(`📺 Fetching data for: ${animeEntry.title || animeEntry.name}`);
                try {
                    const response = await requestUrl(`https://api.jikan.moe/v4/anime/${animeEntry.mal_id}/full`);
                    const fullData = response.json.data;
                    let series = [fullData];
                    const sequelRelation = fullData.relations?.find(r => r.relation === 'Sequel');
                    if (sequelRelation && sequelRelation.entry.length > 0) {
                        series = series.concat(await getAnimeSeries(sequelRelation.entry[0], fetchedIds));
                    }
                    return series;
                } catch (e) { return []; }
            }
            allSeasonsData = await getAnimeSeries(selectedAnime);
            
            // Fetch all episode lists
            notice("📋 Fetching all episode lists...");
            const episodePromises = allSeasonsData.map(season => 
                requestUrl(`https://api.jikan.moe/v4/anime/${season.mal_id}/episodes`).catch(e => ({ json: { data: [] } }))
            );
            const episodeResponses = await Promise.all(episodePromises);
            allSeasonsEpisodes = episodeResponses.map(res => res.json.data);
            
            // Populate variables from the main entry
            const mainAnime = allSeasonsData[0];
            title = mainAnime.title_english || mainAnime.title;
            coverUrl = mainAnime.images?.jpg?.large_image_url || defaultCoverUrl;
            synopsis = mainAnime.synopsis ? mainAnime.synopsis.replace(/\n\n\[Written by MAL Rewrite\]/g, "").trim() : "No synopsis available.";
            genres = mainAnime.genres.map(g => g.name);
        }
    } else {
        notice("❌ Anime not found online. Switching to manual entry.");
        isManual = true;
    }
}

// --- MANUAL ENTRY PATH ---
if (isManual) {
    title = await tp.system.prompt("🎌 Title:", query);
    if (!title) { notice("❌ No title entered. Template cancelled."); return; }
    
    const coverUrlInput = await tp.system.prompt("🖼️ Cover Image URL (leave blank for default):");
    coverUrl = coverUrlInput.trim() === "" ? defaultCoverUrl : coverUrlInput;
    
    synopsis = await tp.system.prompt("📝 Synopsis (can be multiline):") || "No synopsis available.";
    
    const genreInput = await tp.system.prompt("🏷️ Genre(s), comma-separated:");
    genres = genreInput ? genreInput.split(',').map(g => g.trim()) : [];
    
    // Manually build the season/episode structure
    const totalSeasons = parseInt(await tp.system.prompt("📺 How many seasons?")) || 1;
    for (let s = 1; s <= totalSeasons; s++) {
        const seasonTitle = await tp.system.prompt(`🎌 Title for Season ${s}:`, `${title} Season ${s}`);
        const numEpisodes = parseInt(await tp.system.prompt(`📋 How many episodes in Season ${s}?`)) || 1;
        // Create a data structure that mimics the API's
        allSeasonsData.push({ title: seasonTitle, type: 'TV', episodes: numEpisodes });
        // Create a simplified episode list for the renderer
        const manualEpisodes = [];
        for (let e = 1; e <= numEpisodes; e++) {
            manualEpisodes.push({ mal_id: e, title: `Episode ${e}` });
        }
        allSeasonsEpisodes.push(manualEpisodes);
    }
}

// --- DATA GATHERING COMPLETE ---

// 4. Get personal tracking info with consistent icons
const userStatusId = await tp.system.suggester(
    ["🎌 Currently Watching", "✅ Completed", "⏸️ On-Hold", "❌ Dropped", "📋 Plan to Watch"], 
    ["ongoing", "finished", "onhold", "dropped", "queued"], 
    false, "📊 What is your status for this anime?"
);

const rating = await tp.system.prompt("⭐ Your rating (e.g., 8/10, or leave blank):");

// --- FILE CONTENT GENERATION ---
const yaml = `---
type: anime
title: "${title}"
cover_url: "${coverUrl}"
status: ${userStatusId}
rating: ${rating ? `"${rating}"` : '""'}
genres: ${JSON.stringify(genres)}
date_started: ${userStatusId !== 'queued' ? `"${tp.date.now("YYYY-MM-DD")}"` : '""'}
date_finished: ${userStatusId === 'finished' ? `"${tp.date.now("YYYY-MM-DD")}"` : '""'}
tags: [anime]
---
`;

const mainCheckbox = userStatusId === 'finished' ? '[x]' : '[ ]';
let content = `- ${mainCheckbox} **${title}**\n\n`;
content += `\t<div style="text-align: center;">\n\t\t<img src="${coverUrl}" width="100">\n\t</div>\n\n`;

const synopsisLines = synopsis.split('\n').filter(line => line.trim() !== '');
if (synopsisLines.length > 0 && synopsis !== "No synopsis available.") {
    content += `\t- **Synopsis**: ${synopsisLines[0]}\n`;
    if (synopsisLines.length > 1) {
        for (let i = 1; i < synopsisLines.length; i++) {
            content += `\t\t${synopsisLines[i]}\n`;
        }
    }
} else {
    content += `\t- **Synopsis**: No synopsis available.\n`;
}

for (const [index, season] of allSeasonsData.entries()) {
    content += `\t- ## Season ${index + 1}: ${season.title}\n\t\t- ### Episodes\n`;
    const episodes = allSeasonsEpisodes[index];
    if (episodes && episodes.length > 0) {
        // FIXED: Chronological order instead of reversed
        for (const ep of episodes) {
            content += `\t\t\t- [ ] Episode ${ep.mal_id}: ${ep.title}\n\t\t\t\t- \n`;
        }
    } else if (['Movie', 'OVA', 'ONA', 'Special'].includes(season.type)) {
        content += `\t\t\t- [ ] ${season.title}\n\t\t\t\t- \n`;
    } else if (season.episodes > 0) {
        // FIXED: Forward order for manual entries too
        for (let e = 1; e <= season.episodes; e++) {
            content += `\t\t\t- [ ] Episode ${e}\n\t\t\t\t- \n`;
        }
    } else {
        content += `\t\t\t- No episode data could be found.\n`;
    }
}

content += `\n---\n## My Thoughts:\n- `;

// --- FILE MANIPULATION ---
const safeTitle = title.replace(/[\/:"*?<>|]+/g, '-');
await tp.file.rename(safeTitle);
if (targetFolder && targetFolder.trim() !== "") {
    notice(`📁 Moving note to ${targetFolder}...`);
    await tp.file.move(targetFolder + safeTitle);
}

notice("✅ Anime note created successfully!");
tR += yaml + content;
%>
