<%*
const notice = (msg) => new Notice(msg, 6000);

// ---- CONFIG ----
const targetFolder = "11 - PRIVATE/1107 - People/";

// ---- HELPERS ----
const csv = (s) => (s || "").split(",").map(x => x.trim()).filter(Boolean);
const safeFile = (s) => (s || "").replace(/[\/:"*?<>|]+/g, "-").trim();
const atify = (s) => {
  const v = (s || "").trim();
  if (!v) return "";
  return v.startsWith("@") ? v : `@${v}`;
};

async function prompt3PartDate(titlePrefix) {
  let day = (await tp.system.prompt(`${titlePrefix} — Day (optional). Type 1–31, blank if unknown:`)).trim();
  if (day) {
    day = day.replace(/\D/g, "");
    if (day.length === 1) day = "0" + day;
    if (day.length > 2) day = day.slice(0, 2);
  }

  const months = ["January","February","March","April","May","June","July","August","September","October","November","December"];

  let month = "";
  if (day) {
    month = await tp.system.suggester(months, months);
  } else {
    const pickMonth = await tp.system.suggester(["Skip month", "Select month"], [false, true]);
    if (pickMonth) month = await tp.system.suggester(months, months);
  }

  let year = (await tp.system.prompt(`${titlePrefix} — Year (optional). Type YYYY, blank => XXXX:`)).trim();
  if (!year) year = "XXXX";

  let out = "";
  if (day && month) out = `${day} ${month} ${year}`;
  else if (day && !month) out = `${day} ${year}`;
  else if (!day && month) out = `${month} ${year}`;
  else out = "";

  return out;
}

// 1) Identity
const first = (await tp.system.prompt("First name (required):")).trim();
if (!first) { notice("No first name entered. Cancelled."); return; }

const surname = (await tp.system.prompt("Surname / last name (optional):")).trim();
const fullName = `${first} ${surname}`.trim();

const aliases = csv(await tp.system.prompt("Aliases / nicknames (optional, comma-separated):"));

// 2) Relationship (single-token values)
const relOptions = [
  "friend","classmate","peer","family","close_relative","relative",
  "teacher","mentor","colleague","neighbor","online_friend","other (type manually)"
];

let relationship = await tp.system.suggester(relOptions, relOptions);
if (relationship === "other (type manually)") {
  relationship = (await tp.system.prompt("Relationship (manual):")).trim();
}

// 3) Priority (lowercase in YAML)
const prLabels = ["s (very close / critical)","a (close)","b (regular)","c (low priority)"];
const prValues = ["s","a","b","c"];
const priority_lvl = await tp.system.suggester(prLabels, prValues);

// 4) Birthday
const birthday = await prompt3PartDate("Birthday");

// 5) Contact
const contact_number = (await tp.system.prompt("Contact number (optional):", "+91 ")).trim();
const email = (await tp.system.prompt("Email (optional):")).trim();
const telegram = (await tp.system.prompt("Telegram username (optional, without @ is fine):")).trim();
const instagram = (await tp.system.prompt("Instagram username (optional, without @ is fine):")).trim();

// 6) Work / Study
const profession = (await tp.system.prompt("Profession / role (optional):")).trim();
const institution = (await tp.system.prompt("Institution / workplace (optional):")).trim();

// 7) Addresses + places + copy option
const current_address = (await tp.system.prompt("Current address (optional):")).trim();
const place = (await tp.system.prompt("Place (optional; can be a wikilink like [[BANGLOW KI KANDI]]):")).trim();

const samePermanent = await tp.system.suggester(
  ["Permanent = same as current", "Enter permanent separately"],
  [true, false]
);

let permanent_address = "";
let permanent_place = "";

if (samePermanent) {
  permanent_address = current_address;
  permanent_place = place;
} else {
  permanent_address = (await tp.system.prompt("Permanent address (optional):")).trim();
  permanent_place = (await tp.system.prompt("Permanent place (optional; can be a wikilink):")).trim();
}

// 8) Timeline (3-part date)
const first_meeting_date = await prompt3PartDate("First meeting");
const first_meeting_context = (await tp.system.prompt("First meeting place/context (optional):")).trim();

// 9) Takeaways + tags
const takeawaysRaw = (await tp.system.prompt("Takeaways (optional). Separate points with ';' :")).trim();
const takeaways = (takeawaysRaw || "").split(";").map(x => x.trim()).filter(Boolean);

const tags = csv(await tp.system.prompt("Tags (optional, comma-separated):"));

// 10) RENAME & MOVE (Execute after all prompts to minimize race conditions)
const finalName = safeFile(`P ${fullName}`);

// Use Templater's native rename and move for stability
await tp.file.rename(finalName);

const doMove = await tp.system.suggester(
  [`Move to ${targetFolder}`, "Keep in current folder"],
  [true, false]
);

if (doMove) {
  // tp.file.move expects path relative to vault root
  await tp.file.move(`${targetFolder}${finalName}`);
}

// ---- YAML ----
const yamlAliases = aliases.length ? aliases.map(a => `  - ${a}`).join("\n") : "  -";
const yamlTags = tags.length ? tags.map(t => `  - ${t}`).join("\n") : "  -";

const yaml = `---
type:
  - people_note
name: ${first}
surname: ${surname}
aliases:
${yamlAliases}
relationship: ${String(relationship || "").toLowerCase()}
priority_lvl: ${String(priority_lvl || "").toLowerCase()}
birthday: ${birthday}
contact_number: ${contact_number}
current_address: ${current_address}
place: ${place}
permanent_address: ${permanent_address}
permanent_place: ${permanent_place}
email: ${email}
telegram: ${telegram}
instagram: ${instagram}
profession: ${profession}
institution: ${institution}
tags:
${yamlTags}
---
`;

// ---- BODY ----
let body = `# ${fullName}
- **Priority**
\t- \`${String(priority_lvl || "").toUpperCase()}\`
- **Contact Number**
\t- ${contact_number}
- **Relationship**
\t- ${relationship}
- **Birthday**
\t- ${birthday}
- **Current Address**
\t- ${current_address}
- **Place**
\t- ${place}

## ADDITIONAL DETAILS
- **Permanent Address**
\t- ${permanent_address}
- **Permanent Place**
\t- ${permanent_place}
- **Email**
\t- ${email}
- **Telegram**
\t- ${atify(telegram)}
- **Instagram**
\t- ${atify(instagram)}
- **Profession**
\t- ${profession}
- **Institution**
\t- ${institution}

## TIMELINE
### First Meeting
- ${[first_meeting_date, first_meeting_context].filter(Boolean).join(" — ")}

## TAKEAWAYS
`;

if (takeaways.length) {
  takeaways.forEach(t => body += `- *${t}*\n`);
} else {
  body += "- *\n";
}

tR += yaml + "\n" + body;
%>
