# UI/UX Blueprint: Student Attendance Register Tool

## 1. Interface Architecture
**Design Pattern:** Single Page Application (SPA) Simulation using State-Based View Switching.
**Core Mechanism:** A global `currentView` state in JavaScript will determine which container is visible. All views are present in the HTML but toggled via CSS `display: none` and `display: block`.

## 2. Global Navigation (The Shell)
To ensure a responsive mobile layout , a fixed navigation system will be implemented:
- **Position:** Bottom of the screen (Mobile-first approach).
- **Navigation Items:**
    - **Dashboard:** Link to Home/Stats view.
    - **Register:** Link to Student Intake form.
    - **Attendance:** Link to the searchable register.
    - **Settings:** Link to admin utilities and themes.
- **Visual Feedback:** The active navigation item will be highlighted to indicate the current state.

## 3. Detailed View Specifications

### View A: Dashboard (Overview)
*The landing page for the organizer to monitor the event at a glance.*
- **Header:** `<h1>Event Dashboard</h1>`
- **Statistics Grid:** Four high-visibility cards displaying:
    - **Total Registered:** (Sum of all records in `localStorage`).
    - **Total Present:** (Count of students marked as 'Checked-in').
    - **Total Absent:** (Count of students marked as 'Not Checked-in').
    - **Attendance %:** (Calculation: `Present / Registered * 100`).
- **Quick Actions:** 
    - "Quick Check-in" input field for rapid student number entry.

### View B: Registration (Intake)
*The area used to build the initial student list.*
- **Header:** `<h2>Student Registration</h2>`
- **Input Form:**
    - **Student Number Field:** Text input (Required, Primary Key).
    - **Full Name Field:** Text input (Required).
    - **Submit Button:** "Add Student" trigger.
- **Feedback System:** 
    - A notification area for **Confirmation Notices** (e.g., "Student added successfully!") or **Error Notices** (e.g., "Error: Student number already exists") .

### View C: Attendance Register (Management)
*The primary workspace for recording attendance.*
- **Header:** `<h2>Attendance List</h2>`
- **Control Bar:**
    - **Search Input:** Real-time filter to search by name or student number .
    - **Sort Toggle:** Button to toggle between Alphabetical and Registration Date.
- **Student Record List:** A vertical list of cards. Each card contains:
    - **Student Info:** Name and Student Number.
    - **Status Indicator:** Visual badge (e.g., Green for Present, Grey for Absent).
    - **Timestamp:** Display of the exact time of check-in.
    - **Action Buttons:** 
        - `Toggle Presence` (Check-in $\leftrightarrow$ Check-out).
        - `Edit` (Open edit mode for student details).
        - `Remove` (Delete record from `localStorage`).

### View D: Settings & Admin (Utilities)
*Backend-style controls for the application.*
- **Header:** `<h2>System Settings</h2>`
- **Theme Control:** Toggle switch for **Light Mode** and **Dark Mode** .
- **Data Operations:**
    - **CSV Export Button:** Trigger to download the current list as a report .
    - **Bulk Import:** Text area for pasting comma-separated student data.
    - **Reset Button:** A "Danger Zone" button to clear all `localStorage` data.

## 4. User Interaction Flow (State Transitions)
1. **Start:** User opens `index.html` $\rightarrow$ App loads `localStorage` $\rightarrow$ `currentView` defaults to **Dashboard**.
2. **Add Student:** User clicks `Register` Nav $\rightarrow$ State changes to **Registration** $\rightarrow$ User submits form $\rightarrow$ Data saves $\rightarrow$ Success notice appears.
3. **Record Attendance:** User clicks `Attendance` Nav $\rightarrow$ State changes to **Attendance** $\rightarrow$ User searches for student $\rightarrow$ User clicks `Check-in` $\rightarrow$ Timestamp recorded $\rightarrow$ Dashboard stats update automatically.
4. **Export/Reset:** User clicks `Settings` Nav $\rightarrow$ State changes to **Settings** $\rightarrow$ User triggers CSV Export.Since we have successfully completed the **Project Charter** and the **UI/UX Blueprint**, we are now moving to the final piece of the planning phase before we touch the code.
