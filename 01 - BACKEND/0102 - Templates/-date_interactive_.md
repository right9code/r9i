<%*
const today = tp.date.now("YYYY-MM-DD");
let d = await tp.system.prompt("Date (YYYY-MM-DD):", today);

// If you delete the default and submit empty, fallback to today
if (!d || d.trim() === "") d = today;

tR += `[[${d}]]`;
%>
