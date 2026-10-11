# Student Matcher

A Thai-language web application for looking up student pairings in the **พี่เทคน้องเทค** school buddy activity. Students can find who they take care of and who takes care of them, view classroom details, and export the results.

Created by **SAPA14**.

## Features

- View senior-to-junior pairings: ม.4–6 → ม.1–3.
- View junior-to-senior pairings: ม.1–3 → ม.4–6.
- Search either student by name, surname, student ID, or classroom, such as `ม.2/3`.
- Display both students’ names, IDs, classrooms, and class numbers.
- Copy displayed results to the clipboard or download them as CSV.
- Open a separate admin page to view and export all pairings for a selected direction.
- Use a responsive Thai interface with horizontally scrollable results.

## How it works

The application reads two CSV files: `user.csv` contains student details, and `pairs_all.csv` contains predefined caretaker–receiver relationships.

When a user requests results, the browser loads both files, filters pairings by direction, and looks up each student by `student_id`. The main page then applies the search text and renders matching rows. An empty search displays every pairing in the selected direction.

**Pairings must be prepared beforehand.** The repository does not include a matching algorithm, random assignment, or a form for editing students or pairings.

## Quick start: static website

The working interface uses HTML, CSS, and browser JavaScript. You can run both pages without the Node.js server by placing the HTML and CSV files in the same serving directory.

From the repository root, with Python 3 installed:

```bash
mkdir -p static-site
cp client/index.html client/admin.html static-site/
cp client/public/user.csv client/public/pairs_all.csv static-site/
python3 -m http.server 8000 --directory static-site
```

Open:

- Student lookup: `http://localhost:8000/`
- Admin page: `http://localhost:8000/admin.html`

Use an HTTP server rather than opening the HTML files directly, because the pages fetch CSV files over HTTP. The `static-site/` directory can also be deployed to a static web host, including GitHub Pages.

## Node.js development

The repository also includes an Express and Vite development setup. Install a Node.js version supported by the locked Vite dependency, plus npm, then run:

```bash
npm ci
npm run dev
```

The server defaults to `http://localhost:5000`. To choose another port:

```bash
PORT=3000 npm run dev
```

The commands use POSIX environment-variable syntax, suitable for macOS and Linux shells.

### Available scripts

| Command | Purpose |
| --- | --- |
| `npm run dev` | Start Express with Vite development middleware. |
| `npm run build` | Build the client into `dist/public/` and server into `dist/index.cjs`. |
| `npm start` | Run the production server after building. |
| `npm run check` | Run the TypeScript compiler for type checking. |
| `npm run db:push` | Apply the optional Drizzle database schema; unused by the CSV lookup pages. |

For a production build:

```bash
npm run build
npm start
```

The current Vite configuration builds the main `index.html` only. It does not configure `admin.html` as a build entry. To serve the admin page in production, copy it into the built public directory after building:

```bash
cp client/admin.html dist/public/admin.html
```

For predictable access to both HTML pages during local use, use the static setup above. The Express development fallback explicitly serves `client/index.html` and does not define a separate admin route.

## Using the student lookup

1. Choose **พี่เทคน้อง** or **น้องเทคพี่** from the dropdown.
2. Enter a name, surname, student ID, or classroom. Leave the field empty to show all pairings for that direction.
3. Click **แสดงผล** or press Enter in the search field.
4. Review the caretaker and receiver details.
5. Click **คัดลอกทั้งหมด** to copy the displayed results, or **ดาวน์โหลด CSV** to export them.

Search uses a case-insensitive substring match across both students’ details. CSV exports contain the pairing direction and the two student IDs; names and classroom details are not included in the export.

## CSV data format

Update the source files in `client/public/`. For the static setup, copy the updated files into `static-site/` afterward.

### `user.csv`

Required headers:

```csv
number,student_id,role,name,surname,class
1,DEMO001,นาย,Example,Senior,401
2,DEMO002,เด็กหญิง,Example,Junior,203
```

These rows are fictional examples.

| Column | Description |
| --- | --- |
| `number` | Student’s number within the class. |
| `student_id` | ID used to look up the student in pairing records. |
| `role` | Name prefix or title, displayed before the name. |
| `name` | First name. |
| `surname` | Last name. |
| `class` | Numeric classroom code, such as `203` for `ม.2/3` or `401` for `ม.4/1`. Non-numeric values are displayed as supplied. |

### `pairs_all.csv`

Required headers:

```csv
type,caretaker_student_id,receiver_student_id
senior_to_junior,DEMO001,DEMO002
junior_to_senior,DEMO002,DEMO001
```

| Column | Description |
| --- | --- |
| `type` | Exactly `senior_to_junior` or `junior_to_senior`. |
| `caretaker_student_id` | ID of the student taking care of the other student. |
| `receiver_student_id` | ID of the student receiving care. |

Both IDs should exist in `user.csv`. If an ID is missing, the interface displays a missing-student message and a placeholder for classroom details. Each direction is stored independently; the application does not automatically create the reverse relationship.

Use UTF-8 encoding for Thai text. The built-in parser splits on commas and line breaks, so fields must not contain embedded commas or multiline values. Quoted CSV fields are not supported.

## Admin page and data access

The admin page compares the entered password with `ADMIN_PASSWORD` in `client/admin.html`. This check runs entirely in the browser, and the password is visible in the source. It is a convenience gate rather than secure authentication.

Both CSV files are publicly downloadable when hosted, and the student lookup page can display all pairings without a password. Before publishing this repository or deploying it publicly, replace private student records with fictional data unless the records are intended for public access. Protecting real records requires server-side authentication and authorization for the data itself.

## Project structure

```text
Student-Matcher/
├── client/
│   ├── index.html          # Student lookup interface and logic
│   ├── admin.html          # Admin results interface and logic
│   ├── public/
│   │   ├── user.csv        # Student details
│   │   └── pairs_all.csv   # Predefined pairings
│   └── src/                # React scaffold and UI components
├── server/
│   ├── index.ts            # Express server entry point
│   ├── routes.ts           # API route scaffold; no matching endpoints
│   ├── static.ts           # Production static-file serving
│   ├── storage.ts          # Storage scaffold, unused by the lookup pages
│   └── vite.ts             # Development middleware
├── shared/schema.ts        # Database schema scaffold
├── script/build.ts         # Client and server build script
├── vite.config.ts
└── package.json
```

The active lookup pages use inline JavaScript. React, Tailwind CSS, UI components, and Drizzle are included in the surrounding scaffold, but the CSV lookup does not use the React application or require a database.

## Troubleshooting

- **Data does not load:** Check that `user.csv` and `pairs_all.csv` are served beside the HTML page. In the Node.js setup, they come from `client/public/`.
- **No results appear:** Check the selected direction, search text, exact `type` values, and CSV headers.
- **A student’s name is missing:** Make sure the pairing ID matches a `student_id` in `user.csv`.
- **Copy fails:** Allow clipboard access and use localhost or HTTPS. CSV download is an alternative.
- **The admin URL shows the student page after deployment:** Ensure `admin.html` is included in the deployed files.

## License

`package.json` declares the project license as MIT. The repository currently has no standalone `LICENSE` file.
