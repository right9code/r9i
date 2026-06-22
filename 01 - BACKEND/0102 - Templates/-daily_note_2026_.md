<%*
// Date
let dateInput = await tp.system.prompt(
  "Enter date (e.g., 2025-08-11):",
  tp.date.now("YYYY-MM-DD")
); // tp.system.prompt [web:2]

const dateMoment = moment(dateInput, "YYYY-MM-DD");
if (!dateMoment.isValid()) throw new Error("Invalid date. Use YYYY-MM-DD.");

// Prompt order you want: ideal WUT -> WUT, ideal ST -> ST
let idealSleepTime  = await tp.system.prompt("Ideal sleep time (e.g., 09:30 PM):", "09:30 PM") || "09:30 PM"; // tp.system.prompt [web:2]
let sleepTime       = await tp.system.prompt("Sleep time (e.g., 10:00 PM):", idealSleepTime) || idealSleepTime; // default parameter usage [web:2]

let idealWakeupTime = await tp.system.prompt("Ideal wakeup time (e.g., 05:15 AM):", "05:15 AM") || "05:15 AM"; // tp.system.prompt [web:2]
let wakeupTime = await tp.system.prompt("Wakeup time (e.g., 6:00 AM):", idealWakeupTime) || idealWakeupTime; // default parameter usage [web:2]

let energyLevel = await tp.system.prompt("Energy level (1-10):", "6") || "6";

// Helpers
function ampmTo24(timeStr) {
  const [time, period] = timeStr.split(" ");
  let [h, m] = time.split(":").map(Number);
  if (period === "PM" && h !== 12) h += 12;
  if (period === "AM" && h === 12) h = 0;
  return { hours: h, minutes: m };
}

function minutesBetweenTimes(startStr, endStr) {
  const s = ampmTo24(startStr);
  const e = ampmTo24(endStr);

  const start = new Date(2000, 0, 1, s.hours, s.minutes);
  let end = new Date(2000, 0, 1, e.hours, e.minutes);
  if (end < start) end.setDate(end.getDate() + 1); // handle crossing midnight [web:41]

  return Math.round((end - start) / 60000); // minutes from ms difference [web:10]
}

function minutesToHM(mins) {
  const h = Math.floor(mins / 60);
  const m = mins % 60;
  return `${h}h ${m}m`;
}

// Durations
const idealMinutes = minutesBetweenTimes(idealSleepTime, idealWakeupTime);
const actualMinutes = minutesBetweenTimes(sleepTime, wakeupTime);

const idealSleepDuration = minutesToHM(idealMinutes);
const sleepDuration = minutesToHM(actualMinutes);

// Signed variance: + = slept more than ideal, - = slept less than ideal
const sleepVariance = actualMinutes - idealMinutes;

// Extract components
const year       = dateMoment.format("YYYY");
const monthNum   = dateMoment.format("MM");
const month      = dateMoment.format("MMMM");
const dayNum     = dateMoment.format("DD");
const weekNum    = dateMoment.format("ww");
const quarter    = dateMoment.quarter();
const weekday    = dateMoment.format("dddd");
const weekdayNum = dateMoment.isoWeekday();

// Auto-rename & move
await tp.file.rename(`${year}-${monthNum}-${dayNum}`);
await tp.file.move(`11 - PRIVATE/1102 - Journal/${year}/Daily Notes/${year}-${monthNum}-${dayNum}`);

// YAML
tR += `---
type:
  - journal
  - daily_note
year_num: ${year}
Year_Link: "[[11 - PRIVATE/1102 - Journal/${year}/${year}]]"
quarter_num: ${quarter}
Quarterly_Link: "[[11 - PRIVATE/1102 - Journal/${year}/04 - Quarterly Notes/${year}-Q${quarter}]]"
month_num: ${monthNum}
Monthly_Link: "[[11 - PRIVATE/1102 - Journal/${year}/03 - Monthly Notes/${year}-M${monthNum}-${month}]]"
week_num: ${weekNum}
Week_Link: "[[11 - PRIVATE/1102 - Journal/${year}/02 - Weekly Notes/${year}-W${weekNum}]]"
date: ${dateInput}
Date_Link: "[[11 - PRIVATE/1102 - Journal/${year}/01 - Daily Notes/${year}-${monthNum}-${dayNum}]]"
weekday: "${weekday}"
Weekday_Link: "[[11 - PRIVATE/1102 - Journal/Weekdays/${weekdayNum} - ${weekday}|${weekday}]]"
Day_type:
  - "[[LineDAY]]"
  - "[[ArcDAY]]"
  - "[[SwayDAY]]"
aliases:
  - "${weekday}, ${month} ${dayNum} ${year}"
tags:
  - daily-note
ideal_sleep_time: "${idealSleepTime}"
ideal_wakeup_time: "${idealWakeupTime}"
sleep_time: "${sleepTime}"
wakeup_time: "${wakeupTime}"
energy_lvl: ${energyLevel}
ideal_sleep_duration: "${idealSleepDuration}"
sleep_duration: "${sleepDuration}"
sleep_variance: ${sleepVariance}
pomos: 0
pomo_duration: "0h 0m"
---
`;

// Breadcrumb
tR += `**YEAR:** [[11 - PRIVATE/1102 - Journal/${year}/${year}|${year}]] • **QUARTER:** [[11 - PRIVATE/1102 - Journal/${year}/Quarterly Notes/${year}-Q${quarter}|${year}-Q${quarter}]] • **MONTH:** [[11 - PRIVATE/1102 - Journal/${year}/Monthly Notes/${year}-M${monthNum}-${month}|${month}]] • **WEEK:** [[11 - PRIVATE/1102 - Journal/${year}/Weekly Notes/${year}-W${weekNum}|${year}-W${weekNum}]] • **DATE:** [[11 - PRIVATE/1102 - Journal/${year}/Daily Notes/${year}-${monthNum}-${dayNum}|${dateInput}]] • **DAY:** [[11 - PRIVATE/1102 - Journal/Weekdays/${weekdayNum} - ${weekday}|${weekday}]]\n\n`;
%>
## PRE-JOURNAL
- 
## TASKS:
- [ ] 
## TO-DO:

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
		- [ ] **[[09 - PROJECTS/0905 - Home/MOTHER LIT/MOTHER LIT -basenote|TEACHING MY MAA]]** **`S`**
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

***
# JOURNAL Entry:

- ## Morning ME:
	- [ ] **JOURNAL Entry**
		- 
	- [ ] **Physical**
	- [ ] Writing

- ## FOCUS Work ME: `BLOCK 1`
	- #### **BLOCK 1**:
		- [ ] P1:
		- [ ] P2:
		- [ ] P3:
		- [ ] P4:

## ==BREAKFAST== `10:00 AM`

- ## CORE Work ME: `BLOCK 2` 
	- #### **BLOCK 2**:
		- [ ] P1:
		- [ ] P2:
		- [ ] P3:
		- [ ] P4:
		- [ ] P5:

##  ==LUNCH== `02:00 AM`

- ## RELAXED Work ME: `BLOCK 3`
	- [ ] P1:
	- [ ] P2:
	- [ ] P3:

- ## ME 4 ME:
	- **Physical Activity**:
	- **Hobby**:

## ==DINNER==

- ## Sleep ME:
	- **Connect/Surf**:
	- **ShowTime**:
	- **Reading**:

## POST-JOURNAL
- 

## MY TAKEAWAYS:
- 

***

```dataviewjs
const pomos = dv.current().pomos ?? 0;
const totalMinutes = pomos * 25;
const hours = Math.floor(totalMinutes / 60);
const mins = totalMinutes % 60;
const pomo_duration = `${hours}h ${mins}m`;

dv.paragraph(`**Total Pomos Today:** ${pomos}`);
dv.paragraph(`**Total Pomodoro Duration Today:** ${pomo_duration}`);
```
