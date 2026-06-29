# School Calendar 2026
```dataviewjs
const calendarData = [
    // ACADEMIC EXAMS
    { type: "📝 Exam", event: "I Unit Test", start: "2026-04-20", end: "2026-04-25" },
    { type: "📝 Exam", event: "II Unit Test", start: "2026-05-25", end: "2026-06-01" },
    { type: "🚩 Term", event: "I Term Examination", start: "2026-07-06", end: "2026-07-14" },
    { type: "📝 Exam", event: "III Unit Test", start: "2026-08-20", end: "2026-08-27" },
    { type: "📝 Exam", event: "IV Unit Test", start: "2026-10-27", end: "2026-11-02" },
    { type: "🚩 Term", event: "Final Term Examination", start: "2026-11-23", end: "2026-12-07" },

    // MAJOR HOLIDAYS & BREAKS
    { type: "🌴 Break", event: "Holi", start: "2026-03-03", end: "2026-03-04" },
    { type: "🌴 Break", event: "Easter Holidays", start: "2026-04-03", end: "2026-04-05" },
    { type: "🌴 Break", event: "Summer Break", start: "2026-06-21", end: "2026-06-30" },
    { type: "🌴 Break", event: "Deepawali Holidays", start: "2026-11-08", end: "2026-11-11" },
    { type: "🏖️ Holiday", event: "Id-Ul-Fitr", start: "2026-03-21", end: "2026-03-21" },
    { type: "🏖️ Holiday", event: "Independence Day", start: "2026-08-15", end: "2026-08-15" },
    { type: "🏖️ Holiday", event: "Gandhi Jayanti", start: "2026-10-02", end: "2026-10-02" },

    // SPORTS
    { type: "🏆 Sports", event: "Cricket (Inter House)", start: "2026-04-21", end: "2026-04-22" },
    { type: "🏆 Sports", event: "Football (Inter House)", start: "2026-07-17", end: "2026-07-18" },
    { type: "🏆 Sports", event: "Athletics Meet", start: "2026-10-25", end: "2026-10-31" },

    // SPECIAL EVENTS
    { type: "✨ Special", event: "Investiture Ceremony", start: "2026-04-02", end: "2026-04-02" },
    { type: "✨ Special", event: "Annual School Fete", start: "2026-05-09", end: "2026-05-09" },
    { type: "✨ Special", event: "School Picnic", start: "2026-10-21", end: "2026-10-23" },
    { type: "✨ Special", event: "Annual Function", start: "2026-10-16", end: "2026-10-16" },

    // PTM & ADMIN
    { type: "🤝 PTM", event: "PTM (May 2)", start: "2026-05-02", end: "2026-05-02" },
    { type: "🤝 PTM", event: "PTM (Sept 12)", start: "2026-09-12", end: "2026-09-12" },
    { type: "🎓 Final", event: "Report Card Day", start: "2026-12-15", end: "2026-12-15" }
];

const today = new Date();

dv.table(["Status", "Type", "Event", "Date"], 
    calendarData
    .sort((a, b) => new Date(a.start) - new Date(b.start))
    .map(d => {
        const start = new Date(d.start);
        const end = new Date(d.end);
        let status = "✅ Passed";
        
        // Reset time for accurate date-only comparison
        const todayReset = new Date(today.getFullYear(), today.getMonth(), today.getDate());
        const startReset = new Date(start.getFullYear(), start.getMonth(), start.getDate());
        const endReset = new Date(end.getFullYear(), end.getMonth(), end.getDate());

        if (todayReset < startReset) status = "⏳ Upcoming";
        else if (todayReset >= startReset && todayReset <= endReset) status = "🔥 Ongoing";

        return [status, d.type, d.event, d.start === d.end ? d.start : `${d.start} to ${d.end}`];
    })
);
```


## 🗓️ List of Holidays
| Month         | Date and Day        | Occasion                    |
| :------------ | :------------------ | :-------------------------- |
| **MARCH**     | 3rd-4th (Tue-Wed)   | Holi                        |
|               | 21st (Sat)          | Id-Ul-Fitr                  |
|               | 26th (Thu)          | Ram-Navmi                   |
|               | 31st (Tue)          | Mahavir Jayanti             |
| **APRIL**     | 3rd-5th (Fri-Sat)   | Easter Holidays             |
|               | 14th (Tue)          | Baisakhi / Ambedkar Jayanti |
| **MAY**       | 1st (Thu)           | Budh Purnima Institute Day  |
|               | 27th (Wed)          | ID-UL-ZUHA                  |
| **JUNE**      | 21st-30th (Sun-Mon) | Summer Break                |
|               | 26th (Fri)          | Moharram                    |
| **JULY**      | 16th (Wed)          | Harela                      |
| **AUGUST**    | 15th (Sat)          | Independence Day            |
|               | 26th (Wed)          | Milad-Ul-Nabi               |
|               | 28th (Fri)          | Raksha Bandhan              |
| **SEPTEMBER** | 4th (Fri)           | Janmashtami                 |
| **OCTOBER**   | 2nd (Fri)           | Gandhi Jayanti              |
|               | 4th (Sun)           | St. Francis Feast           |
|               | 20th (Mon)          | Navmi / Dusshera            |
|               | 26th (Mon)          | Valmiki Jayanti             |
| **NOVEMBER**  | 8th-11th (Sun-Wed)  | Deepawali Holidays          |
|               | 20th (Fri)          | Igas Bagwal                 |
|               | 24th (Tue)          | Guru Nanak Birthday         |

---

## ✍️ Academic Examinations
* **I Unit Test:** April 20th to April 25th
* **II Unit Test:** May 25th to June 1st
* **I Term Examination:** July 6th to 14th
* **III Unit Test:** August 20th to 27th
* **IV Unit Test:** October 27th to Nov 2nd
* **Final Term Examination:** Nov 23rd to Dec 7th

---

## 🏆 Sports Activities
* **Cricket Tournament (Inter House):** April 21st - 22nd
* **Cricket Tournament (Inter School):** April 29th - 30th
* **Throw Ball Girls (Inter School):** May 8th - 9th
* **Table Tennis (Inter School):** May 15th - 16th
* **Badminton Tournament (Inter House):** June 5th - 6th
* **Badminton Tournament (Inter School):** July 18th - 19th
* **Football Tournament (Inter House):** July 17th - 18th
* **Inter School Marathon:** August 7th - 8th
* **Football Tournament (Inter School):** August 13th - 14th
* **Athletics Meet:** October Last

---

## 🎭 School Activities & Competitions
* **Mar 07:** Women's Day Celebration
* **Mar 14:** Bulletin Board Competition (Easter)
* **Apr 02:** Special Assembly (Good Friday & Easter)
* **Apr 28:** Bulletin Board (Institute Week)
* **Apr 29:** Diary Quiz Competition
* **Apr 30:** Special Assembly (Institute week & Labor Day) and Tableau Competition
* **May 09:** Eng Writing comp (Juniors) & Eng Essay Writing comp (Seniors)
* **Jun 20:** Eng Story Telling Comp (I to VI) & Eng Debate Comp (VII to X)
* **Jul:** G.K Quiz- Juniors House & Science / G.K Quiz S
* **Aug 08:** Bulletin Board Competition (Independence Day & St. Clare's Feast)
* **Aug 11:** Special assembly St. Clare's Feast
* **Aug 15:** Independence Day Celebration (Fancy dress, Patriotic song & dance)
* **Sep 05:** Special Assembly (Teacher's Day)
* **Sep 14:** Hindi story Telling (Juniors) & Hindi Debate (Seniors)
* **Oct 05:** Entrance Exam for new session
* **Oct 21-23:** School Picnic
* **Nov 16-17:** Staff Picnic

---

## ✨ Special Events
* **Apr 02:** Investiture Ceremony
* **Apr 28 - 30:** Institute week & Labor Day celebration
* **May 09:** Annual School Fete
* **Sep 05:** Teacher's Day Celebration
* **Oct 16:** Annual Function
* **Nov 25:** Christmas Celebration
* **Dec 15:** Final Report Card Day

---

## 👥 Parent Teacher Meetings (PTM)
* **May 02:** 2nd Saturday
* **Jun 13:** 2nd Saturday
* **Sep 12:** 2nd Saturday
* **Nov 07:** 1st Saturday
* **Dec 15:** Tuesday (Report Card Day)