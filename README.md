# javascript-project
# Gradify

Gradify is an instructor portal for tracking students, attendance, courses and tasks in one place. It is a front-end project built with plain **HTML, CSS and JavaScript (ES modules)** and uses **json-server** as a mock REST API.

---

## Features

| Area | What it does |
|---|---|
| **Authentication** | Sign up with live password rules (8+ characters, upper case, lower case, number), sign in, sign out. The logged-in user is kept in `localStorage`. |
| **Dashboard** | Average attendance, pending tasks and total enrolled students, a grade distribution chart (Chart.js), a course filter for active tasks, and a students table with *All / Active / At risk / Archive* tabs. |
| **Students** | Add, edit and delete students. Filter by course, status, date and attendance. Record attendance (Present / Absent) per student and date. Live search. |
| **Tasks** | Create a task (name, description, deadline, course), list all published tasks and delete them. |
| **Add course** | Create a course (code + name). Duplicate codes are rejected. Shows the list of existing courses. |
| **Shared layout** | One sidebar and header for every inner page: navigation, search, light/dark theme toggle, profile menu (Support, Sign out) and a mobile drawer. |

---

## Tech stack

| Part | Technology |
|---|---|
| Front end | HTML5, CSS3, JavaScript (ES modules, no framework) |
| Charts | [Chart.js](https://www.chartjs.org) via CDN (dashboard only) |
| Mock API | [json-server](https://github.com/typicode/json-server) `0.17.4` |
| Data | `db.json` |

---

## Project structure

The HTML files reference `../css`, `../js` and `../assets`, so the project is organised like this:

```
Gradify/
├── pages/
│   ├── login.html
│   ├── signup.html
│   ├── dashboard.html
│   ├── students.html
│   ├── tasks.html
│   ├── addcourse.html
│   └── layout.html        # template for new pages
├── css/
│   ├── theme.css          # colors + design tokens (light and dark)
│   ├── layout.css         # sidebar, header, shared components
│   └── login.css, signup.css, dashboard.css, students.css, tasks.css, addcourse.css
├── js/
│   ├── apiservice.js      # ALL requests to json-server
│   ├── layout.js          # shared sidebar + header + session + theme + toast
│   └── login.js, signup.js, dashboard.js, students.js, tasks.js, addcourse.js
├── assets/
│   └── logo.jpeg
└── db.json
```

---

## Getting started

### Requirements
- [Node.js](https://nodejs.org) (LTS)
- VS Code with the **Live Server** extension. The app uses ES modules, so it must be served over HTTP; opening the files with a double click will not work.

### 1. Start the API
Open a terminal in the folder that contains `db.json`:

```bash
npx json-server@0.17.4 --watch db.json --port 3000
```

Open <http://localhost:3000/students> to check it works, and keep the terminal open.

- Use version `0.17.4`: the app relies on features (such as `?q=` search) that json-server 1.x does not support.
- Port `3000` must match `BASE_URL` in `apiservice.js`.
- On Windows PowerShell, if you get *"running scripts is disabled"*, use Command Prompt or run `npx.cmd json-server@0.17.4 --watch db.json --port 3000`.

### 2. Start the front end
Right-click `pages/login.html` and choose **Open with Live Server**.

### 3. Sign in
Use one of the accounts below, or create one from the sign-up page.

---

## Demo accounts

Local testing only. Check `db.json` for the current list.

| Name | Email | Password | Role |
|---|---|---|---|
| Dr. shatha | rana@university.edu | password123 | instructor |
| Dr. Ahmad | ahmad@university.edu | password123 | instructor |
| Dr. tala | admin@university.edu | admin123 | admin |

---

## Data model (`db.json`)

| Collection | Fields |
|---|---|
| `users` | `id, name, email, password, role` |
| `courses` | `id, name, code` |
| `students` | `id, name, email, courseId, status, grade, attendanceRate` |
| `tasks` | `id, title, description, dueDate, courseId, progress, pendingGrading` |
| `attendance` | `id, studentId, date, status` (`Present` / `Absent`). **Required by the Students page, see Known issues** |

Relations: `students.courseId` and `tasks.courseId` point to `courses.id`; `attendance.studentId` points to `students.id`.
Student status is one of `Active`, `At risk`, `Archived`.

---

## API service (`apiservice.js`)

Pages never call `fetch` directly. Everything goes through this file.

```js
import { getStudents, getCourses, STATUS } from './apiservice.js';
```

| Function | Description |
|---|---|
| `loginUser(email, password)` | Returns the user (without password) or `null` |
| `registerUser(data)` | Creates an instructor, fails if the email already exists |
| `getCourses()` / `addCourse(data)` | List courses / add one (duplicate code is rejected) |
| `getStudents(filters)` | Filters: `courseId`, `status`, `q`, `id` |
| `addStudent(data)` / `updateStudent(id, data)` / `deleteStudent(id)` | Student CRUD |
| `getTasks()` / `getTasksByCourse(id)` | Read tasks |
| `addTask(data)` / `deleteTask(id)` | Create / delete a task (new tasks start with `progress: 0`, `pendingGrading: 0`) |
| `getDashboardData({ courseId })` | Stats, status counts, grade distribution, course summaries, active tasks |
| `STATUS` | `{ ACTIVE, AT_RISK, ARCHIVED }`, use these instead of typing strings |

---

## Shared layout (`layout.js`)

Every inner page gets the sidebar and header from one script. A page only needs:

```html
<link rel="stylesheet" href="../css/theme.css">   <!-- 1st -->
<link rel="stylesheet" href="../css/layout.css">  <!-- 2nd -->

<div class="app-shell">
  <aside id="app-sidebar"></aside>
  <div class="app-body">
    <div id="app-header"></div>
    <main class="app-main page"> ... your content ... </main>
  </div>
</div>

<script type="module" src="../js/layout.js"></script>
<script type="module" src="../js/your-page.js"></script>
```

Start from `pages/layout.html`, it already contains this skeleton and the no-flash theme script.

**Exports** you can import from `layout.js`:
`getSessionUser()`, `setSessionUser(user)`, `clearSession()`, `applyTheme(theme)`, `showToast(message)`.

**Events** fired on `window`:
- `gradify:search` with `detail.query`: the header search box (live on the Students page; on other pages pressing Enter opens `students.html?q=...`).
- `gradify:theme` with `detail.theme`: after the theme changes (useful for redrawing charts).

**Storage keys:** `gradify:user` (session) and `gradify:theme`.

**To add a page to the sidebar**, add an entry to the `NAV` array at the top of `layout.js`.

### Styling rules
- Never write colors by hand (`#fff`, `#333`). Use the variables from `theme.css` (`var(--surface)`, `var(--text)`, `var(--border)` ...) so the page supports dark mode automatically.
- Use page-specific class prefixes (`stu-`, `crs-`, ...) to avoid clashes with shared classes.

---

## Known issues and to-do

These were found while reviewing the code and should be fixed before the final delivery.

1. **Students page crashes on load.** `students.js` imports `getAttendance`, `recordAttendance` and `ATTENDANCE` from `apiservice.js`, but those are not exported there yet, and `db.json` has no `attendance` collection. Add:
   - `export const ATTENDANCE = Object.freeze({ PRESENT: 'Present', ABSENT: 'Absent' })`
   - `getAttendance()`, and `recordAttendance({ studentId, date, status })` that updates the record for the same student and date, or creates it.
   - an `"attendance": []` array in `db.json`.
2. **Inconsistent import paths for `apiservice.js`.** `login.js`, `signup.js` and `students.js` use `./apiservice.js`; `dashboard.js` and `tasks.js` use `../apiservice.js`; `addcourse.js` uses `./apiService.js` (capital **S**, which breaks on case-sensitive systems and hosting). Keep one file at `js/apiservice.js` and use `./apiservice.js` everywhere. Also `addcourse.html` loads `addcourse.js` from the same folder instead of `../js/`.
3. **Dashboard and Tasks do not use the shared layout yet.** `dashboard.html` and `tasks.html` have empty sidebar/header containers and do not load `theme.css`, `layout.css` or `layout.js`, so they have no navigation or dark mode. Convert them to the skeleton in `pages/layout.html`.
4. **Dashboard details.**
   - `page-welcome` and `dashboard-term` are never filled.
   - The course filter only filters the task cards, not the stats, the chart or the students table.
   - The hard-coded `<option>` values in `dashboard.html` are replaced by JS and can be removed.
   - Leftover `console.log` calls and a large commented-out block in `dashboard.js` should be cleaned.
   - The chart uses fixed colors that are not theme variables.
5. **No route guard.** Inner pages can be opened without signing in. Redirect to `login.html` when `getSessionUser()` is `null`.
6. **Branding is inconsistent.** The product is called *Gradify* in most pages and *Solvably* in `addcourse.html` and `dashboard.html`. Pick one.
7. **Passwords are stored in plain text** in `db.json`. Acceptable for a mock API, but never reuse this approach in production.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `Cannot reach the API ... Is json-server running?` | Start json-server (step 1) and check the port is `3000` |
| Blank page or `Failed to load module script` | Open the page through Live Server, not by double-clicking |
| `does not provide an export named ...` | A function is missing in `apiservice.js`, see Known issues 1 |
| Styles missing or no dark mode | Check the CSS order: `theme.css` before `layout.css` |
| Live Server reloads after every add/delete | Add `{ "liveServer.settings.ignoreFiles": ["**/db.json"] }` to `.vscode/settings.json` |

---

## Team

| Area | Owner |
|---|---|
| Layout and header | _Tala_ |
| Add course | Shatha |
| Tasks API and page | Omar |
| Students | _Shatha_ |
| Dashboard | _Abdullah_ |
| Login and sign up | _Yossif_ |