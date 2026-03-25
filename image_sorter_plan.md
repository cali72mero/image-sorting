# Image Sorter Web App Architecture Plan

## Objective
Create a single-file, client-side HTML web application that allows users to locally sort massive amounts of images into 'NSFW' and 'SFW' categories without memory leaks, supporting both ZIP downloads (in chunks) and shell script generation (Windows/Mac/Linux) for physical file sorting.

## Key Features
1. **Memory-Safe File Handling:** Uses `<input type="file" webkitdirectory directory multiple>` to get `File` objects. Crucially, images are loaded into the DOM using `URL.createObjectURL(file)`, and immediately destroyed with `URL.revokeObjectURL()` when the next image loads.
2. **Keyboard Navigation:** Left Arrow (SFW), Right Arrow (NSFW), Up/Backspace (Undo).
3. **Undo System:** Maintains a history stack to revert the last decision.
4. **Dual Export System:**
   - **ZIP Export:** Uses `JSZip`. To prevent OOM errors, it must split large exports into chunks (e.g., max 500MB or 500 files per ZIP).
   - **Script Export:** Generates `.bat` (Windows) and `.sh` (Mac/Linux) scripts that move files locally based on absolute/relative paths.
5. **Preserve Structure:** The relative path of the file (`file.webkitRelativePath`) is maintained in both the ZIP export and the script generation.
6. **Modern UI:** Dark mode, clean, responsive, suitable for GitHub Pages.

## File Structure
Since it needs to be a single HTML file for GitHub Pages, we will embed CSS and JavaScript directly into the `index.html`. 
External dependencies (like JSZip for ZIP creation) will be loaded via CDN.

## Implementation Steps

### 1. HTML Structure
- Header: Title, File Input (Directory), Clear Button.
- Stats Bar: Total, Remaining, NSFW count, SFW count.
- Main Display: Large image container, maintaining aspect ratio.
- Controls: SFW (Green/Left), Undo (Gray/Up), NSFW (Red/Right).
- Export Modal/Section: Options for ZIP (Chunked) or Scripts (Windows/Linux/Mac).

### 2. CSS (Tailwind or Custom CSS)
- We will use custom CSS with CSS Variables for a dark mode theme to keep it truly standalone and fast, avoiding heavy framework parsing on a single file.

### 3. JavaScript Logic
- **State Management:**
  - `files`: Array of `File` objects.
  - `currentIndex`: Integer.
  - `decisions`: Object mapping `file.webkitRelativePath` to 'nsfw' or 'sfw'.
  - `history`: Array of previous `currentIndex` and their states for the Undo feature.
- **File Loading:** Filtering for `image/*`.
- **Image Rendering:** `URL.createObjectURL()` and memory cleanup.
- **Export Logic:**
  - **Script Generator:** Iterates through `decisions`. Generates `mkdir -p "nsfw/path"` and `mv "path" "nsfw/path"` commands.
  - **ZIP Generator:** Asynchronous JSZip generation. Prompts user to download multiple ZIPs if file count is high.

## Edge Cases & Limitations
- **File Paths in Scripts:** Browsers obfuscate absolute paths for security. The script will rely on `webkitRelativePath`. The user must place the generated script *next to* the root folder they uploaded.
- **RAM during ZIP:** JSZip must hold the entire ZIP in RAM before triggering the download. If a user sorts 10GB of images, JSZip *will* crash the tab. We must enforce chunking (e.g., 500MB per ZIP max).

## Final Deliverable
A single `index.html` containing HTML, CSS, and JS.