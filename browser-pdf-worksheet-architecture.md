# Generating printable handwriting PDFs in a static browser app

A worksheet maker can use static hosting for its interface while doing the document work in the browser. The important boundary is between publishing the application and generating a particular worksheet. Publishing delivers HTML, JavaScript, styles, and fonts; generating a sheet uses the visitor’s current text and settings.

I’m Ethan Lee, a maintainer of [Cursive Practice Sheets](https://cursivepracticesheets.com/). This note describes the approach used by its worksheet generator. It shares architectural decisions rather than publishing the application’s source code.

## Keep one layout model

Start with physical page dimensions instead of measuring the responsive preview container. PDF points provide a useful coordinate system: 72 points represent one inch. US Letter is 612 by 792 points; A4 uses its own dimensions. Margins, writing guides, baselines, and wrapped lines all belong to that model.

The preview then scales the same page model to the available screen width. Export renders it at a higher raster scale. This keeps browser window size from becoming an accidental input to pagination. Changing paper size, font size, or text still requires a fresh layout.

## Load fonts before measuring

A fallback font can change wrapping even when the replacement looks similar. Bundle the licensed worksheet fonts with the static assets and wait for the selected font to load before measuring text. Treat failure as a recoverable state: retain the entered text, show a useful message, and offer an explicit retry.

Measure candidate lines with the same font used for drawing. Account for visible ink extending beyond a nominal advance width, particularly with cursive flourishes and accents. Preserve explicit newlines and capitalization. Validate supported characters so missing glyphs become an actionable message rather than unnoticed boxes.

## Preserve shaping in the export

Draw each completed line as a whole instead of placing its letters independently. That lets the browser shape contextual letter forms and cursive joins. A page canvas can contain the model text and the writing guides together.

The worksheet generator embeds a PNG of each page into a PDF at 300 DPI. This preserves the rendered appearance, but it produces image-based pages rather than selectable text. A project needing searchable text, smaller documents, or accessible document structure should evaluate an appropriate text-based PDF pipeline instead. Raster output is a deliberate tradeoff, not a universal default.

## Bound the work

High-resolution canvases consume memory before they become compressed images. Limit the input and page count, generate export pages in sequence, and release canvas backing buffers when they are no longer needed. This implementation accepts at most 1,000 characters and eight pages.

Rapid edits introduce another issue: an older asynchronous render can finish after a newer request. Snapshot the settings for each render and discard obsolete results. Apply the same idea when taking the settings snapshot for a download.

## Validate the boundary

A practical review checklist includes a long word, accented letters, explicit line breaks, both paper sizes, a failed font load, and several quick edits. Compare wrapping between preview and export. Open the PDF and test one printed page at actual size before distributing copies; previewing on a screen cannot establish a particular printer’s margins or contrast.

This separation keeps the worksheet computation client-side while leaving the public pages suitable for static delivery. Analytics, authentication, or other services—if present elsewhere in an application—remain separate concerns; client-side document generation alone does not establish that an entire website makes no network requests.
