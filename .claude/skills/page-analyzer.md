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
3. **Summarize sections**: For each section found, provide a summary of its purpose and content as well as a querySelector to identify the section.
4. **Identify blocks inside sections**: Based on https://www.aem.live/developer/markup-sections-blocks and https://www.aem.live/developer/block-collection, identify:
   - Common blocks (hero, columns, cards, carousel, mosaic, etc.)
   - Default content elements (headings, text, images)
   - Block variants (e.g., columns with 2 or 3 variants, cards with featured/simple/link variants)
5. **Output structured analysis**: Provide a consistent breakdown of sections and their constituent blocks. Format:
   - For each section, display: `Section: <name/querySelector>`
   - Next line: `Blocks: <block1>, <block2>, <block3>`
   - Use full block names with variants (e.g., `columns (two)`, `cards (featured)`, `links`, `callout (cta)`, `carousel`)
   - Do not number the blocks, just list them comma-separated in order they appear
   - Apply this same format for all sections: explicit `<section>` tags, implicit content areas, sidebars, all treated uniformly

## Block Detection Patterns

### Columns Block Detection
Detect by structural DOM layout, not CSS classes:
- Look for **multiple sibling container elements** at the same nesting level within a section
- Each sibling contains distinct content (text blocks, images, statistics, etc.)
- The siblings are arranged horizontally/side-by-side visually or stacked vertically with equal visual weight
- Count the number of distinct column children to determine variant:
  - 2 sibling columns = `columns (two)`
  - 3 sibling columns = `columns (three)`
  - 4 sibling columns = `columns (four)`
- Key indicator: A section header followed by N distinct content blocks arranged in parallel
- Common patterns:
  - Multiple text blocks with centered content (statistics, facts, key information)
  - Image + text layout
  - Multiple cards or text blocks with identical structure
- Look within each section for this sibling container pattern

### Cards Block Detection
Detect by identifying repeated similar sibling elements with consistent structure:
- Look for **multiple sibling containers** that repeat the same structural pattern
- Each card/item typically contains a combination of:
  - A title/heading or link
  - Description text or excerpt
  - An image
  - Optional clickable/action button or link behavior
- Key structural indicator: **Repeated sibling elements** with identical internal structure (e.g., all have image + text + link, or all have title + description + button)
- The repetition and similarity of structure indicates a card pattern, regardless of the specific container type or naming
- Cards are designed to be scanned visually as a group, all visible simultaneously

#### Cards Variant Naming
Identify card variants based on their primary structural/visual components:
- **`cards (featured)`** - Cards with: image + title + description + link/button. The image is a prominent visual element
- **`cards (simple)`** - Cards with: title + description (no image) + link/button. Minimal structure focused on text content
- **`cards (link)`** - Cards with: title/link only (minimal). No description or other content, just a heading with link behavior
- Use these variant names to distinguish cards by their visual complexity and content emphasis
- If cards are visually and structurally similar (e.g., both have title + description + button), use the same variant name even if the implementation differs

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

- Focus on DOM structure (parent-child relationships and sibling elements) rather than CSS class names
- **Columns**: Multiple sibling elements arranged side-by-side, all visible simultaneously. Each column contains distinct content topics
- **Cards**: Multiple sibling elements in a grid/list layout, all visible simultaneously. Each card has similar structure with title + description + optional link
- **Carousel**: Multiple sibling elements within a sliding/scrolling container, designed to be viewed sequentially/one-at-a-time. Items have identical structure but different content
- **Default Content**: Single or minimal content elements without repetitive sibling structures
- **Article/Content Pages**: Look for pages without explicit `<section>` tags. These typically have a multi-column layout with:
  - A main content area (left column) with flowing article content
  - Sidebar(s) (right column) with navigation and widget blocks
- **Sidebars as Implicit Sections**: Treat sidebars/widget columns as separate implicit sections and identify the blocks within them (links, callouts, etc.)
- Sections often contain just a single block with optional default content (heading) above it
- Pay attention to visual hierarchy and how content is arranged, not styling classes
- Key distinction: Columns and Cards are **spatial** (visible together), while Carousel is **temporal** (viewed one at a time)
- For article pages with implicit sections: treat the main content area as one section with "Default content", and treat the sidebar as another implicit section with identified blocks (link blocks, callouts, etc.)