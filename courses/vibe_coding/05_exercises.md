# Module 05: Interactive Exercises

---

## How to Use These Exercises

Each exercise gives you a goal, constraints, and hints. Build each project using only natural language prompts with your vibe coding tool. Solutions show the prompts that would produce a working result.

**Difficulty levels:**
- **Starter** -- 1-3 prompts to complete
- **Intermediate** -- 5-10 prompts with iteration
- **Advanced** -- Multi-file projects requiring planning

---

## Exercise 1: Landing Page (Starter)

### Goal

Build a landing page for a fictional product called "FocusFlow" -- an AI-powered productivity timer.

### Requirements

- Hero section with a tagline and call-to-action button
- Features section (3 features with icons)
- Testimonials section (2-3 fake quotes)
- Footer with links
- Modern, clean design
- Responsive

### Constraints

- Single HTML file
- No external libraries (except Google Fonts if desired)
- Must look professional

### Hints

Start with the overall layout, then refine each section.

<details>
<summary>Solution: Prompts Used</summary>

**Prompt 1:**
> Create a modern landing page for "FocusFlow" -- an AI-powered productivity timer. Include a hero section with the tagline "Your AI Focus Partner", a features section with 3 features (Smart Timer, Focus Analytics, Team Sync), a testimonials section with 3 quotes, and a footer. Use a purple and white color scheme. Single HTML file with embedded CSS.

**Prompt 2:**
> The hero section needs more visual impact. Add a gradient background (purple to indigo), make the headline larger, and add a subtle animation to the CTA button (a gentle pulse or glow effect).

**Prompt 3:**
> The features section looks cramped. Add more whitespace, use card-style layouts for each feature, and add simple SVG icons (a clock, a chart, and a group of people).

**Prompt 4:**
> Make it fully responsive. On mobile, the features should stack vertically and the hero text should be centered. Test at 375px width.

</details>

---

## Exercise 2: Interactive Quiz (Starter)

### Goal

Build a multiple-choice quiz about a topic you enjoy (movies, history, science, etc.).

### Requirements

- At least 5 questions with 4 answer options each
- Show one question at a time
- Track score and show results at the end
- Visual feedback for correct/incorrect answers

### Constraints

- Single HTML file
- Questions stored in a JavaScript array (not hardcoded in HTML)
- Must include a progress bar

<details>
<summary>Solution: Prompts Used</summary>

**Prompt 1:**
> Build an interactive multiple-choice quiz as a single HTML file. Topic: space exploration. 5 questions, 4 options each. Show one question at a time with a progress bar at the top. When the user selects an answer, highlight it green (correct) or red (incorrect) and show the right answer, then show a "Next" button. At the end, display the score with a message based on how well they did. Store questions in a JavaScript array. Clean modern design.

**Prompt 2:**
> Add a smooth transition between questions -- a subtle slide animation. Also add a timer showing how many seconds have elapsed total.

**Prompt 3:**
> At the results screen, show which questions were answered correctly and which were wrong, with the correct answers. Add a "Try Again" button that resets everything.

</details>

---

## Exercise 3: Expense Tracker (Intermediate)

### Goal

Build a personal expense tracker that lets you log expenses and see a breakdown.

### Requirements

- Add expenses with amount, category, date, and description
- Show a running total
- Display expenses in a table with sorting
- Show a breakdown by category (as a simple bar chart)
- Delete expenses
- Persist data in localStorage

### Constraints

- No chart libraries -- build the bar chart with CSS
- Responsive design
- Must handle edge cases (negative numbers, empty fields)

### Planning Hint

Use the scaffold-then-refine pattern from Module 04.

<details>
<summary>Solution: Prompt Strategy</summary>

**Phase 1 -- Scaffold:**
> Build an expense tracker with these sections: a form to add expenses (amount, category dropdown, date, description), a summary bar showing total and count, a table listing all expenses, and a category breakdown chart area (placeholder for now). Use a green and white color scheme. Single HTML file.

**Phase 2 -- Core functionality:**
> Make the form functional. When submitted, add the expense to the table with all fields displayed. Include a delete button on each row. Show the running total in the summary bar. Validate that amount is a positive number and all fields are filled.

**Phase 3 -- Persistence:**
> Save all expenses to localStorage. Load them when the page opens. Make sure delete also updates localStorage.

**Phase 4 -- Chart:**
> Build the category breakdown chart using only CSS. It should be a horizontal bar chart where each bar's width represents the percentage of total spending for that category. Show the dollar amount and percentage on each bar. Categories: Food, Transport, Entertainment, Bills, Shopping, Other.

**Phase 5 -- Sorting and polish:**
> Add sorting to the table -- click column headers to sort by amount (high to low), date (newest first), or category (alphabetical). Add a hover effect on table rows and smooth transitions.

</details>

---

## Exercise 4: Pomodoro Timer (Intermediate)

### Goal

Build a Pomodoro timer with session tracking.

### Requirements

- 25-minute work timer, 5-minute short break, 15-minute long break
- Visual countdown with a circular progress indicator
- Sound notification when timer ends
- Session counter (long break after every 4 work sessions)
- Start, pause, and reset controls
- History of completed sessions

### Constraints

- No external libraries
- The circular progress must use CSS or SVG (not canvas)
- Must work when the tab is in the background

<details>
<summary>Solution: Prompt Strategy</summary>

**Prompt 1:**
> Build a Pomodoro timer app. 25-minute work sessions, 5-minute short breaks, 15-minute long breaks after every 4 work sessions. Show a large circular progress indicator (using SVG) that counts down. Include Start, Pause, and Reset buttons. Use a warm color scheme -- orange for work, blue for breaks. Single HTML file.

**Prompt 2:**
> Add a sound notification when the timer reaches zero. Use the Web Audio API to generate a pleasant chime (no external sound files). Also show a browser notification if the user has granted permission.

**Prompt 3:**
> Add a session history section below the timer. Show completed sessions as small cards with the type (Work/Break), duration, and timestamp. Persist the history in localStorage.

**Prompt 4:**
> The timer drifts when the browser tab is in the background. Fix this by tracking the actual end time and calculating remaining time from the system clock instead of decrementing a counter.

**Prompt 5:**
> Add settings that let the user customize work duration, short break duration, and long break duration. Store settings in localStorage. Add a gear icon that opens a settings modal.

</details>

---

## Exercise 5: Full-Stack Notes App (Advanced)

### Goal

Build a note-taking application with markdown support, organized by folders.

### Requirements

- Create, edit, and delete notes
- Organize notes into folders
- Markdown rendering (bold, italic, headings, lists, code blocks)
- Search across all notes
- Keyboard shortcuts (Ctrl+N for new note, Ctrl+S to save)
- Split view: folder tree on left, note list in middle, editor on right

### Constraints

- Build your own simple markdown parser (no libraries)
- All data in localStorage
- Must handle 100+ notes without performance issues

### Planning Requirement

Before building, ask the AI to plan the architecture. Review and adjust the plan before any code is written.

<details>
<summary>Solution: Prompt Strategy</summary>

**Phase 1 -- Architecture:**
> Plan a note-taking application with markdown support, folders, and search. I need:
> - File structure (HTML, CSS, JS files)
> - Data model for notes and folders
> - Component breakdown
> - State management approach
>
> Present the plan before writing code.

**Phase 2 -- Layout scaffold:**
> Build the three-panel layout: folder tree sidebar (250px), note list panel (300px), and note editor panel (remaining space). Use placeholder content. Add a toolbar at the top of the editor with formatting buttons. Make the panels resizable.

**Phase 3 -- Data layer:**
> Create the data management module. Notes have: id, title, content (markdown), folderId, createdAt, updatedAt. Folders have: id, name, parentId (for nesting). Store everything in localStorage with a single state object. Include functions for CRUD operations on both notes and folders.

**Phase 4 -- Markdown parser:**
> Build a simple markdown parser that handles: headings (#, ##, ###), bold (**text**), italic (*text*), unordered lists (- item), ordered lists (1. item), code blocks (triple backticks), inline code (single backticks), links, and horizontal rules. Don't use any libraries.

**Phase 5 -- Wire it together:**
> Connect the UI to the data layer. Clicking a folder shows its notes in the list panel. Clicking a note shows its content in the editor. The editor should show raw markdown while editing and rendered markdown in preview mode. Add a toggle button for edit/preview.

**Phase 6 -- Search and shortcuts:**
> Add a search bar at the top that filters notes by title and content as the user types. Implement keyboard shortcuts: Ctrl+N creates a new note, Ctrl+S saves the current note, Ctrl+Shift+F focuses the search bar.

**Phase 7 -- Performance:**
> Optimize for handling 100+ notes. Use virtual scrolling or lazy rendering for the note list. Debounce the search input. Minimize localStorage writes by batching updates.

</details>

---

## Exercise 6: API-Connected Dashboard (Advanced)

### Goal

Build a dashboard that displays data from a public API.

### Requirements

- Use a free public API (suggestions: OpenWeatherMap, REST Countries, or the PokeAPI)
- Display data in cards, a table, and at least one chart
- Include search/filter functionality
- Loading states and error handling
- Responsive grid layout

### Constraints

- No frameworks or libraries
- Build charts with CSS/SVG
- Must handle API rate limits gracefully
- Cache API responses in localStorage to reduce API calls

<details>
<summary>Solution: Starting Prompts</summary>

**Prompt 1 (Architecture):**
> Plan a dashboard that uses the REST Countries API (restcountries.com). It will show country data in three views: summary cards (top 10 by population), a filterable data table, and a bar chart of population by region. Plan the file structure and data flow.

**Prompt 2 (API layer with caching):**
> Build the API module. It should fetch from https://restcountries.com/v3.1/all, cache the response in localStorage with a 1-hour expiry, and provide functions to search by name, filter by region, and sort by population or area. Include proper error handling and loading states.

**Prompt 3 (Layout and cards):**
> Build the dashboard layout: a search bar and region filter dropdown at the top, a row of summary cards below (Total Countries, Total Population, Largest Country, Most Languages), and a main content area with a table tab and a chart tab.

**Prompt 4 (Table):**
> Build the countries table with columns: Flag (emoji), Name, Region, Population (formatted with commas), Area (km2), Languages. Make the table sortable by clicking column headers. Add pagination showing 20 countries per page.

**Prompt 5 (Chart):**
> Build a horizontal bar chart using CSS showing total population per region (Africa, Americas, Asia, Europe, Oceania, Antarctic). Bars should be proportional to population with the actual number displayed. Animate the bars on load.

</details>

---

## Grading Yourself

For each exercise, evaluate your work against this rubric:

| Criteria | Points |
|----------|--------|
| **Functionality** -- Does it work as described? | /25 |
| **Design** -- Is it visually appealing and consistent? | /25 |
| **Prompt efficiency** -- Did you achieve the result in minimal prompts? | /25 |
| **Edge cases** -- Does it handle errors, empty states, and invalid input? | /25 |

### Score Guide

| Score | Level |
|-------|-------|
| 90-100 | Vibe coding master -- you communicate with AI like a pro |
| 70-89 | Strong foundation -- keep refining your prompts |
| 50-69 | Getting there -- review Module 04 patterns |
| Below 50 | Revisit Modules 01-03 before continuing |

---

## What's Next?

Congratulations on completing the Vibe Coding course. To continue learning:

1. **Build something you actually need** -- the best practice is solving real problems
2. **Try different tools** -- each AI tool has different strengths
3. **Join communities** -- Vibe coding communities on Discord, Reddit, and X are active
4. **Continue to the [Claude Code course](../claude_code/README.md)** for deep expertise in terminal-based AI coding
5. **Explore [Vibe Ops](../vibe_ops/README.md)** for applying these principles to operations
