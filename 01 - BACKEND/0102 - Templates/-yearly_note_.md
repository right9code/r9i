<%*
// Prompt for year in format "YYYY"
let yearInput = await tp.system.prompt("Enter year (e.g., 2025):");
if (!yearInput) yearInput = tp.date.now("YYYY"); // Fallback to the current year

const year = yearInput;

// AUTO-RENAME THE FILE
await tp.file.rename(`${year}`);

// AUTO-MOVE THE FILE
await tp.file.move(`11 - PRIVATE/1102 - Journal//${year}/${year}`);

tR += `---
type:
  - journal
  - yearly_note
year_num: ${year}
Year_Link: "[[01 - Journal/${year}/${year}]]"
aliases:
  - "${year}"
tags:
  - yearly-note
yearly_avg_sleep_time: "Xh Ym"
yearly_avg_wakeup_time: "Xh Ym"
yearly_avg_sleep_duration: "Xh Ym"
yearly_pomos: 0
yearly_pomo_duration: "0h 0m"
---

**YEAR:** [[11 - PRIVATE/1102 - Journal//${year}/${year}|${year}]]

`;
%>
```dataviewjs
const year = dv.current().year_num;

const quarterlyNotesPath = `11 - PRIVATE/1102 - Journal/${year}/Quarterly Notes`;

// This block finds and lists the quarterly notes for this year.
const quarterlyNotesInYear = dv.pages(`"${quarterlyNotesPath}"`)
    .where(q => q.year_num == year);

dv.paragraph(`## Quarterly Notes for ${year}`);

if (quarterlyNotesInYear.length > 0) {
    quarterlyNotesInYear
        .sort(q => q.quarter_num) // Sort by quarter number
        .forEach(qNote => {
            dv.paragraph(`- [[${qNote.file.path}|${qNote.file.name.replace(/\.md$/, "")}]]`);
        });
} else {
    dv.paragraph("No quarterly notes found for this year.");
}
```

- ### **BASE 3**: `CONFIRMED`
	- **HEALTH**
	    - [ ] **[[09 - PROJECTS/0901 - Health/OPTIMISING SLEEP PROTOCOL/OPTIMISING SLEEP PROTOCOL 2026 - mainnote|OPTIMISING SLEEP PROTOCOL 2026]]** **`S`**
	    - [ ] **[[09 - PROJECTS/0901 - Health/OPTIMISING WORKOUT PROTOCOL/OPTIMISING WORKOUT PROTOCOL-basenote|OPTIMISING WORKOUT PROTOCOL]]** **`A`**
    - **ACADEMICS**
	    - [ ] **[[05 - ACADEMICS/0501 - PGT/Uttarakhand Lecturer/Syllabus for Uttarakhand Lecturer|ACE UTTARAKHAND LECTURER]]** **`S`**
	    - [ ] **[[05 - ACADEMICS/0501 - PGT/PGT KVS/Syllabus for PGT KVS|BEST SHOT AT KVS PGT EXAM]]** **`A`**
    - **WRITING**
	    - [ ] **[[09 - PROJECTS/0903 - Writing/BAL BHADRA CHRONICLES/BAL BHADRA CHRONICLES-basenote|BAL BHADRA CHRONICLES]]** **`S`**
	- **LEARN**
		- [ ] **[[09 - PROJECTS/0904 - Learn/LEARN TOUCHTYPE/LEARN TOUCHTYPE-basenote|LEARN TOUCHTYPE]]** **`S`**
		- [ ] **[[09 - PROJECTS/0904 - Learn/LEARN ASL/LEARN ASL-basenote|LEARN ASL]]** **`A`**
	- **HOME**
		- [ ] **[[09 - PROJECTS/0905 - Home/MOTHER LIT/MOTHER LIT -basenote|TEACHING MY MAA]]** **`SSITUPS`**
	- **ONESHOT**
		- [ ] **[[09 - PROJECTS/0906 - Oneshot/OS A SLEEP HYPNOSIS SESSION/OS A SLEEP HYPNOSIS SESSION - basenote|RECORD A SLEEP HYPNOSIS SESSION]]** **`S`**
	- **Miscellaneous**
		- [ ] **[[09 - PROJECTS/0907 - Miscellaneous/POETRY CHANNEL/POETRY CHANNEL-basenote|POETRY CHANNEL]]** **`S`**
- ### **OTHERS**: `PERHAPS`
	- **HEALTH**
	- **ACADEMICS**
		- [[09 - PROJECTS/0902 - Academics/IX - XII WRITING SECTION CBSE YT/IX - XII WRITING SECTION CBSE YT-basenote|IX - XII WRITING SECTION CBSE YT]] **`A`**
	- **WRITING**
		- [[09 - PROJECTS/00 - Unsorted (09)/WHY AM I ANXIOUS-basenote|WHY AM I ANXIOUS]] **`B`**
	- **ONE SHOT**
	- **Miscellaneous**

---
## **My THOUGHTS**:
- *

---
```dataviewjs
const year = dv.current().year_num;

const dailyNotesPath = `11 - PRIVATE/1102 - Journal//${year}/010101 - Daily Notes`;

// This block calculates the yearly summary from all daily notes in this year.
const dailyInYear = dv.pages(`"${dailyNotesPath}"`)
    .where(d => d.year_num == year);

let totalPomos = 0;
let wakeupTimes = [];
let sleepTimes = [];
let sleepDurations = [];

for (let day of dailyInYear) {
    const pomos = parseInt(day.pomos ?? 0);
    if (!isNaN(pomos)) {
        totalPomos += pomos;
    }

    if (day.wakeup_time) wakeupTimes.push(day.wakeup_time);
    if (day.sleep_time) sleepTimes.push(day.sleep_time);

    // --- ROBUST SLEEP DURATION PARSING ---
    if (day.sleep_duration) {
        const durStr = String(day.sleep_duration);
        let totalMinutes = 0;

        // Independently find hours and minutes from the string
        const hourMatch = durStr.match(/(\d+)\s*h/i);
        const minMatch = durStr.match(/(\d+)\s*m/i);

        if (hourMatch) {
            totalMinutes += parseInt(hourMatch[1]) * 60;
        }
        if (minMatch) {
            totalMinutes += parseInt(minMatch[1]);
        }
        
        // Only add to the array if a valid duration was found
        if (totalMinutes > 0) {
            sleepDurations.push(totalMinutes);
        }
    }
}

// --- HELPER FUNCTIONS (Consistent with all other templates) ---

function avgTime(times) {
    if (!times.length) return "No data";
    const totalMinutes = times.reduce((acc, t) => {
        const m = moment(t, ["h:mm A", "H:mm"]);
        return acc + (m.hours() * 60 + m.minutes());
    }, 0);
    const avg = totalMinutes / times.length;
    const h = Math.floor(avg / 60);
    const m = Math.floor(avg % 60);
    return moment({ hour: h, minute: m }).format("h:mm A");
}

function avgDuration(minArr) {
    if (!minArr.length) return "No data";
    const avg = minArr.reduce((a, b) => a + b, 0) / minArr.length;
    const h = Math.floor(avg / 60);
    const m = Math.floor(avg % 60);
    return `${h}h ${m}m`;
}

function formatPomoDuration(pomoCount) {
    const totalMinutes = pomoCount * 25;
    const h = Math.floor(totalMinutes / 60);
    const m = totalMinutes % 60;
    return `${h}h ${m}m`;
}

// --- DISPLAY RESULTS (Consistent Order) ---

dv.paragraph(`**Yearly Summary for ${year}**`);
dv.paragraph(`- **Average Sleep Time:** ${avgTime(sleepTimes)}`);
dv.paragraph(`- **Average Wakeup Time:** ${avgTime(wakeupTimes)}`);
dv.paragraph(`- **Average Sleep Duration:** ${avgDuration(sleepDurations)}`);
dv.paragraph(`- **Total Pomos:** ${totalPomos}`);
dv.paragraph(`- **Total Pomodoro Duration:** ${formatPomoDuration(totalPomos)}`);
```
