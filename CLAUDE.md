# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains course materials for CS221: Artificial Intelligence: Principles and Techniques (Autumn 2019). It is a static website built with HTML, CSS, and JavaScript that generates the course schedule, assignments, and project information.

## Project Structure

- `index.html`: Main course homepage
- `events.js`: JavaScript file that generates the course schedule, assignments, and deadlines
- `assignments/`: Directory containing individual assignment directories (each with its own `index.html`)
- `lectures/`: Lecture materials and demonstrations
- `live/`: Live programming examples
- `projects.js`: Data for project listings
- `project.html`: Project information page
- `sections/`: Section slides (PDF)
- `sample-projects/`: Example student projects
- `q/`: Quiz or question materials
- `photos/`: Course staff photos
- `2018/`: Materials from previous offering
- `plugins/`: JavaScript plugins (including `main.js`)
- `bayes3/`: Additional resources for Bayesian networks topic

## Development Commands

### Running the Site
Since this is a static website, there are no build steps or dependencies:
- Open `index.html` in any modern web browser to view the course homepage
- Individual assignments can be viewed by opening their respective `index.html` files in the `assignments/` directories
- The site uses client-side JavaScript only; no server is required

### Editing Files
- HTML files: Edit directly and refresh browser to see changes
- CSS: Found in `<style>` tags in HTML files or separate CSS files (e.g., `project-list.css`)
- JavaScript: 
  - Main logic in `events.js`
  - Assignment-specific JavaScript may be found in assignment directories
  - Plugins in `plugins/` directory
- After editing JavaScript files, refresh the browser to see changes (may need hard refresh to clear cache)

### Testing
There are no automated tests in this repository. Verification is manual:
- Check that schedule renders correctly in `index.html`
- Verify assignment due dates and links work
- Test individual assignment pages for correctness
- Ensure all links point to existing resources

### Common Tasks
1. **Updating course schedule**: Modify `events.js`:
   - Update `firstDateOfClass` variable (line ~13)
   - Add/modify events in the `nextClass()` and enable/disable item calls
   - Adjust homework/project due dates via `closedHomework()`, `openHomework()`, `project()` calls

2. **Adding/updating assignments**:
   - Create new directory in `assignments/` with assignment materials
   - Add assignment call in `events.js` (e.g., `closedHomework('assignment-name', 'Assignment Title', numDays)`)
   - Update assignment details in the `assignments` array if needed for submission system

3. **Updating lectures/demos**:
   - Place new lecture materials in `lectures/` directory
   - Reference in `events.js` via `enableItem()` calls with appropriate names
   - Live demos go in `live/` directory and referenced via `liveProgrammingLink()`

4. **Updating projects**:
   - Edit `projects.js` to change project listings
   - Update `project.html` if structural changes needed
   - Add project calls in `events.js` using `project()` function

## Important Notes

- The `events.js` file has a `serverSide` flag that when true outputs JSON for a submission system (used in actual course deployment). For local development, keep this false (default) to generate HTML view.
- Date handling in `events.js` uses integer timestamps; helper functions `parseDate()`, `advanceDate()`, and `formatDate()` are provided.
- All paths are relative to the repository root.
- The site uses jQuery (loaded via plugins/main.js) for DOM manipulation.
- When updating due dates, note that the `numDaysTillDueDate` parameter in homework/functions is typically 8 days unless overridden.
- The course uses a specific color scheme: green for current/out items, red for due items, brown for exams.

## Repository Specifics

- This is the `gh-pages` branch (as seen in git status)
- The main branch is `main`
- Last commit: "Server" (71512ba)
- No package.json, build tools, or dependencies beyond standard web technologies

## Getting Started

1. Clone the repository (if not already done)
2. Open `index.html` in your browser
3. Make changes to HTML/CSS/JS files as needed
4. Refresh browser to view changes
5. For schedule changes, edit `events.js` and reload homepage