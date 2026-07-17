Since we are finalizing the planning phase of our Waterfall process, this document acts as the technical manual for your JavaScript implementation. It ensures that you satisfy the core requirements of state management, duplicate prevention, and data persistence .

Copy the following content into your `docs/data-logic.md` file:

***

# Data Schema & Logic Flow: Student Attendance Register Tool

## 1. Data Schema (The "Brain")
To ensure data is preserved after a refresh, all information will be stored as an array of objects within `localStorage` .

### Student Object Definition
Each student record will be represented by the following object structure:
```javascript
{
    id: "timestamp_uniqueID", // Unique identifier for internal tracking
    studentNumber: "String",  // Unique identifier provided by the student (Primary Key)
    fullName: "String",       // Full name of the attendee
    isPresent: Boolean,       // Attendance status (true = present, false = absent)
    checkInTime: "String",    // ISO timestamp of when the student was marked present
    lastModified: "String"    // Timestamp of the last edit/update to the record
}
```

### Storage Keys
- **Primary Store:** `guild_sa_attendance_list` (Stores the array of student objects).
- **App Settings:** `guild_sa_app_settings` (Stores user preferences like theme: light/dark).

---

## 2. Logic Flow (The "Plumbing")

### A. Registration Flow (Adding Attendees)
**Goal:** Add a student while preventing duplicate records .
1. **Input Capture:** Retrieve values from `studentNumber` and `fullName` fields.
2. **Validation:** 
    - Check if fields are empty $\rightarrow$ Trigger Error Notice.
3. **Duplicate Check:** 
    - Search the existing array for any object where `studentNumber === inputStudentNumber` .
    - If found $\rightarrow$ Trigger "Student already registered" Error Notice.
4. **Object Creation:** Create a new Student Object with `isPresent: false` and `checkInTime: null`.
5. **Persistence:** Push object to the array $\rightarrow$ `JSON.stringify()` $\rightarrow$ Save to `localStorage` .
6. **UI Update:** Clear form $\rightarrow$ Trigger Success Notice $\rightarrow$ Update Dashboard Stats.

### B. Attendance Flow (Recording Presence)
**Goal:** Efficiently record and update student attendance .
1. **Identification:** Find student via Search Bar (filter by name/number) or QR scan.
2. **State Toggle:**
    - If `isPresent` is `false` $\rightarrow$ Set to `true` $\rightarrow$ Generate current timestamp for `checkInTime`.
    - If `isPresent` is `true` $\rightarrow$ Set to `false` $\rightarrow$ Set `checkInTime` to `null`.
3. **Persistence:** Update the specific object in the array $\rightarrow$ Save to `localStorage` .
4. **UI Update:** Refresh the Status Badge (Green/Grey) and update Dashboard Stats.

### C. Statistics Logic (Monitoring)
**Goal:** Provide real-time operational data for the organizer .
- **Total Registered:** `studentsArray.length`
- **Total Present:** `studentsArray.filter(s => s.isPresent === true).length`
- **Total Absent:** `studentsArray.filter(s => s.isPresent === false).length`
- **Attendance Percentage:** `(Total Present / Total Registered) * 100`

### D. Management Flow (Edit/Remove/Reset)
1. **Edit:** Load student object into the registration form $\rightarrow$ Update values $\rightarrow$ Save.
2. **Remove:** Filter out the specific student ID from the array $\rightarrow$ Save to `localStorage`.
3. **Reset All:** Call `localStorage.clear()` or remove the specific `guild_sa_attendance_list` key $\rightarrow$ Reload page.

---

## 3. Technical Constraints Compliance
- **No Backend:** All logic resides in the browser; data is stored locally on the organizer's machine .
- **Vanilla JS:** All flows are implemented using standard array methods (`.push()`, `.filter()`, `.find()`, `.map()`) without external frameworks .
- **Performance:** Since the tool is "lightweight," the entire dataset is loaded into memory (JS Variable) on startup and synced to `localStorage` on every change .