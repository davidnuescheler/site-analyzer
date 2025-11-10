---
name: screenshot-generator
description: Capture block variant screenshots for both mobile and desktop devices using the image-forest service and save them with standardized naming conventions.
---

# Screenshot Generator

Generate and store screenshots of block variants for both mobile and desktop devices using the image-forest screenshot service.

## Instructions

### Input Requirements
You will receive:
1. **URL**: The webpage URL to capture screenshots from
2. **Block name**: The name of the block (e.g., "hero", "cards", "columns")
3. **Variant** (optional): The variant of the block (e.g., "featured", "two", "cta")
4. **CSS selector**: The CSS selector to target the specific block element
5. **Output folder**: The folder where screenshots should be stored (create if it doesn't exist)

### Process

1. **Build image service URLs**: Using the image-forest service at `https://image-forest-58aa.david8603.workers.dev/`, construct two URLs:
   - Mobile: `https://image-forest-58aa.david8603.workers.dev/?url=[URL]&device=mobile&selector=[SELECTOR]`
   - Desktop: `https://image-forest-58aa.david8603.workers.dev/?url=[URL]&device=desktop&selector=[SELECTOR]`

   Where:
   - `[URL]` is the webpage URL (URL-encoded)
   - `[SELECTOR]` is the CSS selector (URL-encoded)

   **Before each request, output**: Block name, variant (if any), and the full service URL being requested

2. **Download screenshots**: Fetch both URLs using curl or similar tool, saving the webp images
   - Output the block name, variant, and the service URL for each request (mobile and desktop)
   - Provide progress feedback as screenshots are being captured

3. **Name files** according to the pattern:
   - **Without variant**: `[block]-mobile.webp` and `[block]-desktop.webp`
   - **With variant**: `[block]-[variant]-mobile.webp` and `[block]-[variant]-desktop.webp`

   Examples:
   - `hero-mobile.webp`, `hero-desktop.webp`
   - `cards-featured-mobile.webp`, `cards-featured-desktop.webp`
   - `columns-three-mobile.webp`, `columns-three-desktop.webp`

4. **Save to folder**: Store both screenshots in the specified output folder with the standardized filenames

5. **Verify**: Confirm both files were created successfully and display their paths

### Error Handling

- If the output folder doesn't exist, create it
- If the image service returns an error, display the error message and HTTP status
- If the selector doesn't match any elements on the page, note that in the output
- Validate that URLs are properly encoded before making requests

### Output

**During Processing:**
For each block being captured, output:
- Block name
- Variant (if any)
- Full image-forest service URL for mobile request
- Full image-forest service URL for desktop request
- Status of each capture (in progress, completed, or error)

**After Completion:**
Report the final operation results:
- Summary of all blocks captured
- Confirmation that both mobile and desktop screenshots were captured for each block
- Full paths to the saved files
- File sizes (if available)
- Any errors or warnings encountered
