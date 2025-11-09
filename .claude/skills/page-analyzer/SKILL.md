---
name: page-analyzer
description: Analyze website structure and extract main content areas, sections, and identify blocks based on AEM markup. Use when analyzing web pages to understand their content organization, sections, and block structure for site auditing and documentation.
---

# Page Analyzer

Analyze a website URL to extract main content areas, sections, and identify blocks and default content elements based on the AEM markup structure.

## Instructions

1. **Get website source code**: Use curl with a Chrome user agent to fetch the HTML from the provided URL
2. **Extract main content areas**: Parse the HTML and identify the main content sections. Look for:
   - Explicit `<section>` tags with IDs (most common on template-based pages)
   - Implicit hero sections at the top of pages (even without explicit `<section>` tags):
     - Video or image backgrounds with title overlays
     - Large background images with h1 headings
     - Hero-style content positioned above the first explicit section
   - For article/content pages without explicit sections, treat the main content area as a single implicit section
   - **Always exclude footer content**: Do not analyze or include `<footer>` tags or footer-related elements (elements with classes/ids containing "footer") in your section breakdown
3. **Summarize sections**: For each section found, provide a summary of its purpose and content as well as a querySelector to identify the section.
4. **Identify blocks inside sections**: Based on https://www.aem.live/developer/markup-sections-blocks and https://www.aem.live/developer/block-collection, identify:
   - Common blocks (hero, columns, cards, carousel, mosaic, etc.)
   - Default content elements (headings, text, images)
   - Block variants (e.g., columns with 2 or 3 variants, cards with featured/simple/link variants)
5. **Output structured analysis**: Provide a consistent breakdown of sections and their constituent blocks. Format STRICTLY as follows (one section per block of output):

   ```
   Section: <name or heading summary>
   querySelector: <selector>
   Blocks: <block1>, <block2>, <block3>
   ```

   Guidelines:
   - For each section, display: `Section: <name or heading summary>` (prefer heading/h1/h2 text if available for clarity, otherwise use ID or name attribute)
   - Include a single querySelector for easy identification: `querySelector: <selector>` - **always wrap the selector in backticks** to prevent markdown formatting issues with special characters like underscores
   - Next line: `Blocks: <block1>, <block2>, <block3>`
   - **querySelector best practices**:
     - Prefer IDs when available (e.g., `` `section#facts` `` instead of `section[id="facts"]`)
     - Use simplified class selectors (e.g., `` `section.img.blue` `` instead of `section[class*="img blue"]`)
     - Avoid using attribute selectors with wildcards like `[class*=]` - use direct class selectors instead
     - Chain multiple classes when needed (e.g., `` `.vucontainer-container.blue` `` for elements with both classes)
     - Always wrap selectors containing underscores or other special characters in backticks (e.g., `` `div#container-layout-_-1156700515` ``)
   - Use full block names with variants (e.g., `columns (two)`, `cards (featured)`, `links`, `callout (cta)`, `carousel`)
     - Keep variant values simple and concise (e.g., `columns (three)` not `columns (three, repeated twice for 6 items total)`)
     - Variants should describe the block type, not the layout complexity or repetition
   - Do not include default content elements (text, image, heading) as separate blocks - only include named block types
   - If a section contains only default content elements with no named blocks, write: `Blocks: default content`
   - Do not number the blocks, just list them comma-separated in order they appear
   - Apply this same format for all sections: explicit `<section>` tags, implicit content areas, sidebars, all treated uniformly
   - Include implicit hero sections that appear above the first explicit section (with video/image backgrounds and h1 headings)
   - Separate each section's output block with a blank line
   - Do NOT add any other commentary, explanation, or analysis beyond these three lines per section

## Block Detection Patterns

### Columns Block Detection
Detect by structural DOM layout, not CSS classes:
- Look for **multiple sibling container elements** at the same nesting level within a section
- **Key distinction from cards**: Columns contain semantically distinct content (different topics/purposes), while cards are structurally identical repeating items
- Each sibling contains distinct content (text blocks, images, statistics, etc.) on different topics
- The siblings are arranged horizontally/side-by-side visually or stacked vertically with equal visual weight
- Count the number of distinct column children to determine variant:
  - 2 sibling columns = `columns (two)`
  - 3 sibling columns = `columns (three)`
  - 4 sibling columns = `columns (four)`
- **For multi-row layouts**: If the same columns pattern repeats across multiple rows (e.g., 3 columns in row 1, 3 columns in row 2, 3 columns in row 3), treat it as a single `columns` block with the column count based on the pattern per row, not duplicate the block for each row
- Key indicator: A section header followed by N distinct content blocks (different topics) arranged in parallel
- **Critical distinction from cards**: Look at the *internal structure* of each sibling, not the wrapper classes. If all siblings have identical internal structure (same child elements in same order), treat as cards instead
- Common patterns:
  - Multiple text blocks with centered content (statistics, facts, key information)
  - Image + text layout with different topics in each column
  - Grid layouts that repeat the same column structure across multiple rows with different topics
- **NOT columns if**: All items follow identical structure with the same child elements (image + title + description + button, etc.) - these are cards instead
- Look within each section for this sibling container pattern

### Cards Block Detection
Detect by identifying repeated similar sibling elements with consistent structure:
- Look for **multiple sibling containers** that repeat the same structural pattern
- **Key distinction from columns**: Cards are structurally identical repeating items, while columns contain semantically distinct content
- Each card/item typically contains a combination of:
  - A title/heading or link
  - Description text or excerpt
  - An image
  - Optional clickable/action button or link behavior
- Key structural indicator: **Repeated sibling elements** with identical/near-identical internal structure (e.g., all have image + text + link, or all have title + description + button)
- The repetition and similarity of structure indicates a card pattern, regardless of the specific container type or naming
- Cards are designed to be scanned visually as a group, all visible simultaneously (grid or list layout)
- **Examples of cards**: "Ways to Give" options with identical structure (title + description + button), portfolio items, team member profiles, service offerings, etc.

#### Cards Variant Naming
Identify card variants based on their primary structural/visual components:
- **`cards (featured)`** - Cards with: image + title + description + link/button. The image is a prominent visual element (may be displayed at top, side, or as background). Description content may be expanded/collapsed or always visible
- **`cards (simple)`** - Cards with: title + description (no image) + link/button. Minimal structure focused on text content only
- **`cards (link)`** - Cards with: title/link only (minimal). No description or other content, just a heading with link behavior
- Use these variant names to distinguish cards by their visual complexity and content emphasis
- If cards are visually and structurally similar (e.g., both have title + description + button), use the same variant name even if the implementation differs
- The presence of an image is the primary differentiator between featured and simple variants, regardless of how the description content is displayed

### Carousel Block Detection
Detect by structural DOM layout indicating sequential/sliding content:
- Look for a **parent container** with **multiple similar sibling item/slide elements**
- Each item/slide element contains related content (images, text, etc.) that would be revealed sequentially
- Content is designed to be viewed one at a time (or a few at a time), with navigation between items
- Key structural indicators:
  - Multiple `item` or `slide` or `itemImage` sibling divs within a container
  - Each sibling has identical/similar structure and styling
  - Implies horizontal scrolling or fade/slide transitions
  - May have explicit carousel/slider container with multiple content items
- Carousel variants typically don't have a numbered designation like columns

### Grid/Masonry Block Detection
Detect grid or masonry layouts by structural DOM layout:
- Look for a **parent container with a grid-like class pattern** (e.g., `masonry-wrapper`, `grid-container`) containing **multiple similar sibling box/item elements**
- Each item/box element typically contains:
  - An image or background image as primary visual
  - A heading/title overlay or linked content
  - Optional description or metadata
- **Key distinction from cards**: Grid/masonry items are displayed in a responsive grid layout with variable sizing/positioning (often using CSS Grid or Masonry library), whereas cards are in a uniform grid
- The items are designed to fill space efficiently in a grid arrangement
- Key structural indicators:
  - Parent container with class names like `masonry`, `grid`, `gallery`, or similar wrapper patterns
  - Sibling elements with classes like `box`, `item`, `tile`, or similar consistent naming
  - Each sibling has identical/similar structure (image + heading + optional link)
  - Layout is responsive and items may reflow/reposition based on available space
- Variants: Typically no numbered designation, but may be noted if there's a specific visual pattern
- **Examples**: Portfolio galleries, image galleries with overlaid titles, product grids, research/project showcases

### Link Block / Navigation Block Detection
- A list of links organized as a block
- Typically found in:
  - Sidebar navigation panels
  - Collapsible menus with multiple links
  - Related topics/navigation sections
- Structure: heading + unordered list of links
- Often wrapped in a panel or collapsible container
- May have variants like `links (nav)` or `links (sidebar)` depending on styling/purpose

### Callout Block Detection
- A small widget/box with: image + heading/text + optional button
- Typically found in sidebars
- Designed to draw attention to specific content
- Variants: `callout` or `callout (cta)` if it has a call-to-action button
- Often styled with background colors or borders

### Hero Block Detection
Detect hero sections as either explicit blocks or implicit sections at the top of pages:
- First content section with minimal structured content
- Often contains:
  - Large background image with h1 heading/title overlay
  - Video background with title text
  - Large banner-style image with text overlay
- Limited text content (typically just a heading/title)
- Positioned at the top of the page
- May not have explicit `<section>` tags with IDs - look for visual/structural indicators of hero content above the first explicit section
- When found as implicit section, treat it as a separate hero section even if not wrapped in a `<section>` tag

### Article/Content Page Detection
Pages without explicit `<section>` tags often follow an article or content page pattern:
- Multi-column layout with main content area and sidebar(s)
- Main content area (typically in a `div.pagecontent` or similar) contains:
  - A title/hero area at the top (image + heading)
  - Body text paragraphs
  - Embedded images within the text flow
  - Mix of **default content blocks** and **images** interspersed throughout
- Sidebar/widget columns (typically `col-md-4` or similar) act as **implicit sections** and contain:
  - Navigation panels (link lists)
  - Call-out boxes with image + text + button
  - Collapsible menu blocks
  - Other widget-style content
- **Analysis approach**:
  - Treat the main content area as one implicit section with default content
  - Treat sidebars as separate implicit sections and identify the blocks within them (link blocks, call-out cards, etc.)
  - Describe the mix of content types found (e.g., "Image + text blocks", "Image + text + button")

### Default Content
- Single text blocks, images, or simple paragraphs
- Not part of a multi-column, card, or carousel layout
- No parallel sibling content structure
- Minimal repeated elements
- In article pages: the flowing mix of images and text throughout the main content area

## Key Considerations

- **Exclude footer sections**: Never include `<footer>` elements or any footer-related content (classes/ids containing "footer") in your analysis. Only analyze main page content.
- Focus on DOM structure (parent-child relationships and sibling elements) rather than CSS class names
- **Columns**: Multiple sibling elements arranged side-by-side, all visible simultaneously. Each column contains distinct content topics
- **Cards**: Multiple sibling elements in a uniform grid/list layout, all visible simultaneously. Each card has similar structure with title + description + optional link
- **Grid/Masonry**: Multiple sibling elements in a responsive grid layout with variable sizing/positioning, designed to fill space efficiently. Items are displayed simultaneously in a flexible layout
- **Carousel**: Multiple sibling elements within a sliding/scrolling container, designed to be viewed sequentially/one-at-a-time. Items have identical structure but different content
- **Default Content**: Single or minimal content elements without repetitive sibling structures
- **Article/Content Pages**: Look for pages without explicit `<section>` tags. These typically have a multi-column layout with:
  - A main content area (left column) with flowing article content
  - Sidebar(s) (right column) with navigation and widget blocks
- **Sidebars as Implicit Sections**: Treat sidebars/widget columns as separate implicit sections and identify the blocks within them (links, callouts, etc.)
- Sections often contain just a single block with optional default content (heading) above it
- Pay attention to visual hierarchy and how content is arranged, not styling classes
- Key distinctions:
  - **Columns vs Cards**: Columns have semantically distinct content in parallel; cards are structurally identical items
  - **Cards vs Grid/Masonry**: Cards use uniform grid sizing; grid/masonry use variable sizing and flexible repositioning
  - **Spatial vs Temporal**: Columns, Cards, and Grid/Masonry are spatial (visible together), while Carousel is temporal (viewed one at a time)
- For article pages with implicit sections: treat the main content area as one section with "Default content", and treat the sidebar as another implicit section with identified blocks (link blocks, callouts, etc.)
