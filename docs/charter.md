# Project Charter: GUILD SA Student Attendance Register Tool

## 1. Project Overview
The objective is to develop a lightweight, browser-based digital tool to replace pen-and-paper registers for Buildathon 01. The tool must allow event organizers to manage student attendance end-to-end without the need for a backend infrastructure.

## 2. Hard Constraints (Zero-Tolerance Rules)
Failure to adhere to these constraints will result in automatic disqualification.

* **File Structure:** The entire application must be contained within one single file titled exactly `index.html`
* **Technology Stack:** Only HTML, CSS, and vanilla JavaScript are permitted.
* **Prohibitions:**
  * No backend or external database.
  * No frameworks (e.g., React, Vue, Angular).
  * No build processes or multiple source files.
* **Execution:** The tool must work immediately upon opening the `index.html` file in any modern web browser.

## 3. Functional Requirements
The tool must successfully implement the following core features:
* **Attendee Management:** Ability to add new attendees and prevent duplicate records.
* **Attendance Tracking:** Record attendance (check-in/out) and maintain a searchable attendee list.
* **Data Persistence:** Use localStorage to ensure data is preserved after a page refresh.
* **Organizer Dashboard:** Display real-time statistics including:
  * Total registered attendees.
  * Total attendees present.
  * Total attendees not checked in.
  * Attendance percentage.
* **Data Maintenance:** Ability to edit, remove, or reset all records.


## 4. Extended Features
To improve the submission score (Creativity & Enhancements), the following will be implemented:
* **QR-Based Concept:** Integration of a QR check-in system.
* **Data Portability:** CSV export for downloadable reports.
* **UX/UI Enhancements:** Responsive mobile layout and Dark/Light theme toggles.
* **Audit Trail:** Implementation of check-in timestamps.


## 5. Submission Deliverables
The final submission package must include:
* Required
  * The complete `index.html` file.
  * A technical summary (maximum 200 words).
  * An AI and external-tool disclosure statement.
* **Strongly Encouraged:**
  * A public GitHub repository.
  * A live deployed application (in this case im using Cloudflare Pages)


## 6. Timeline & Deadline
* Submission Deadline: Wednesday, 22 July 2026, at 12:00 PM
* Constraint: Late submissions will not be accepted as the portal closes exactly at the deadline .