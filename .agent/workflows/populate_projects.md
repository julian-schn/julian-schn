---
description: Update projects.md based on README.md content
---

1. Read `README.md` and `projects.md` to understand the current state.
2. Extract the projects list from the `## Cool projects` (or similar) section in `README.md`.
3. For each project found in `README.md`, extract:
    - **Project Name**: The name of the project.
    - **Link**: The URL if it's a link.
    - **Description**: The brief description provided.
    - **Technologies**: The list of technologies mentioned.
4. Determine the "When" date for each project.
    - If the project is already in `projects.md`, preserve the existing date.
    - If it's a new project, use `git log` to find the approximate start or most active date, or use "2025" as a default if unsure.
5. Format the data into a markdown table in `projects.md` with the columns: `| Project | When | Description | Technologies |`.
6. Ensure existing entries in `projects.md` that are NOT in `README.md` are preserved if they seem important, or ask the user if they should be removed.
7. Update `projects.md` with the new table, ensuring the formatting is clean.
