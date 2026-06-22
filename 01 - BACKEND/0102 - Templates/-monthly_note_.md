<%*  
// Prompt for month in format "YYYY-MM"  
let monthInput = await tp.system.prompt("Enter month (e.g., 2025-08):");  
if (!monthInput) monthInput = tp.date.now("YYYY-MM");

const dateMoment = moment(monthInput, "YYYY-MM");  
if (!dateMoment.isValid()) throw new Error("Invalid month. Use YYYY-MM.");

const year = dateMoment.format("YYYY");  
const monthNum = dateMoment.format("MM");  
const monthName = dateMoment.format("MMMM");  
const quarter = dateMoment.quarter();

// Auto-rename  
await tp.file.rename(`${year}-M${monthNum}-${monthName}`);

// Auto-move  
await tp.file.move(`11 - PRIVATE/1102 - Journal/${year}/Monthly Notes/${year}-M${monthNum}-${monthName}`);

tR += `---
type:
  - journal
  - monthly_note
year_num: ${year}  
Year_Link: "[[11 - PRIVATE/1102 - Journal/${year}/${year}]]"  
quarter_num: ${quarter}  
Quarterly_Link: "[[11 - PRIVATE/1102 - Journal/${year}/Quarterly Notes/${year}-Q${quarter}]]"  
month_num: ${monthNum}
Monthly_Link: "[[11 - PRIVATE/1102 - Journal/${year}/Monthly Notes/${year}-M${monthNum}-${monthName}]]"
aliases:
  - "${monthName} ${year}"  
tags:
  - monthly-note  
monthly_avg_sleep_time: "Xh Ym"  
monthly_avg_wakeup_time: "Xh Ym"  
monthly_avg_sleep_duration: "Xh Ym"  
monthly_pomos: 0  
monthly_pomo_duration: "0h 0m"
---
`;

// Breadcrumbs  
tR += `**YEAR:** [[11 - PRIVATE/1102 - Journal/${year}/${year}|${year}]] • **QUARTER:** [[11 - PRIVATE/1102 - Journal/${year}/Quarterly Notes/${year}-Q${quarter}|${year}-Q${quarter}]] • **MONTH:** [[11 - PRIVATE/1102 - Journal/${year}/Monthly Notes/${year}-M${monthNum}-${monthName}|${monthName}]]\n\n`;  
%>
```dataviewjs
// Weekly Notes for {MonthName}, {Year}
const year = dv.current().year_num;
const monthNum = dv.current().month_num;
const monthName = moment(dv.current().file.name, "YYYY-[M]MM-MMMM").format("MMMM");

dv.paragraph(`## Weekly Notes for ${monthName}, ${year}`);

const weeklyNotesPath = `11 - PRIVATE/1102 - Journal/${year}/Weekly Notes`;
const weeklyNotes = dv.pages(`"${weeklyNotesPath}"`).where(w => w.month_num == monthNum);

if (weeklyNotes.length > 0) {
    weeklyNotes
        .sort(w => w.week_num)
        .forEach(weekNote => {
            dv.paragraph(`- [[${weekNote.file.path}|${weekNote.file.name.replace(/\.md$/, "")}]]`);
        });
} else {
    dv.paragraph(`No weekly notes found for ${monthName}, ${year}.`);
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
const monthNum = dv.current().month_num;
const monthName = moment().month(monthNum - 1).format("MMMM");

const dailyNotesPath = `11 - PRIVATE/1102 - Journal/${year}/Daily Notes`;

// This now filters by BOTH year and month for maximum robustness.
const dailyInMonth = dv.pages(`"${dailyNotesPath}"`)
    .where(d => d.year_num == year && d.month_num == monthNum);

let totalPomos = 0;
let wakeupTimes = [];
let sleepTimes = [];
let sleepDurations = []; // This will be an array of total minutes per day

// Collect data directly from daily notes
for (let day of dailyInMonth) {
    // Sum Pomos
    const pomos = parseInt(day.pomos ?? 0);
    if (!isNaN(pomos)) {
        totalPomos += pomos;
    }

    // Collect times for averaging
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

// --- HELPER FUNCTIONS ---

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

// --- DISPLAY RESULTS ---

dv.paragraph(`**Monthly Summary for ${monthName}, ${year}**`);
dv.paragraph(`- **Average Sleep Time:** ${avgTime(sleepTimes)}`);
dv.paragraph(`- **Average Wakeup Time:** ${avgTime(wakeupTimes)}`);
dv.paragraph(`- **Average Sleep Duration:** ${avgDuration(sleepDurations)}`);
dv.paragraph(`- **Total Pomos:** ${totalPomos}`);
dv.paragraph(`- **Total Pomodoro Duration:** ${formatPomoDuration(totalPomos)}`);
```
