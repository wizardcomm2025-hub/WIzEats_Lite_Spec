# Import & export — UI / UX

There is no visible UI today (IMPEX-D1).

The app contains `ImportExcelSection`, a Profile card titled `Import/Export Data` with two drop-downs:

- `Import` → `Floor` / `Menu` / `Zone`, which opens the system file picker for CSV;
- `Export` → `Floors` / `Menu` / `Zones`.

It is commented out in `profile.tsx`. If it is switched back on unchanged:

- the import posts the picked file as the field `file`, so the server answers 422 `At least one CSV file is required.`;
- the export posts to `APU_URL.FILE.GET_FILE + <name>`, not to `/api/export/…`.

The toasts it would show: `Please select a file to import`, `File uploaded successfully`, `Failed to select file. Please try again.`, `Failed to export file. Please try again.`
