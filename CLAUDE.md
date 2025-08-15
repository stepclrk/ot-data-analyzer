# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a client-side web application suite for customer data analysis and billing format conversion. All tools are standalone HTML files with embedded JavaScript - no build process or server required.

## Architecture

### Application Structure
- **Static HTML Applications**: Each tool is a self-contained HTML file with inline CSS and JavaScript
- **Client-Side Processing**: All data processing happens in the browser for security and privacy
- **No Backend**: No server, API, or database - completely browser-based
- **External Dependencies**: Loaded via CDN (Chart.js, XLSX.js, PapaParse)

### Main Applications

1. **index.html**: Landing page and navigation hub
   - Marketing/information site with links to all tools
   - Contains embedded media (video tutorial, podcast)
   - Responsive design with animations

2. **customer_data_analyzer.html**: Primary analytics platform
   - Multi-file Excel/CSV processing
   - 5 analysis views (Summary, Detail, Entity, Trading Partner, Map)
   - Chart visualizations using Chart.js
   - Export functionality to Excel/CSV

3. **Classic_to_TG_Billing_format_converter.html**: Format converter
   - Converts MS Classic CSV to TG Billing format
   - Batch file processing
   - Automatic data mapping

4. **csv_to_xlsx_converter.html**: CSV to Excel converter
   - Converts TG CSV exports to structured Excel workbooks
   - Multi-sheet organization
   - Drag-and-drop interface

5. **id_name_extractor.html**: Data extraction utility
   - Extracts IDs and names from data files
   - Pattern recognition capabilities

## Development Guidelines

### File Modifications
- Each HTML file is completely self-contained - modify CSS and JavaScript within the file
- Preserve inline structure when making changes
- Test all changes directly by opening the HTML file in a browser

### Adding Features
- Keep all functionality client-side
- Use existing CDN libraries where possible (Chart.js, XLSX.js, PapaParse already included)
- Maintain browser compatibility - avoid cutting-edge APIs without fallbacks

### Data Processing
- All file processing must happen in-browser using FileReader API
- Never send data to external servers
- Use Web Workers for heavy processing if needed to avoid UI blocking

### Testing
- No automated tests - manual testing by opening HTML files directly
- Test with sample data files in `/example` directory
- Verify Excel export/import functionality with actual Excel files
- Check browser console for errors

## Common Tasks

### Run Locally
Simply open any HTML file directly in a web browser. No server needed.

### Deploy
Copy HTML files to any web server or static hosting service. No build step required.

### Debug
Open browser Developer Tools (F12) and check Console for errors. All processing logic is visible in the HTML files.

## Important Patterns

### File Upload Pattern
All tools use similar drag-and-drop and file input patterns:
```javascript
// Drag and drop handling
element.addEventListener('dragover', (e) => e.preventDefault());
element.addEventListener('drop', handleFileDrop);
// File input handling  
fileInput.addEventListener('change', handleFileSelect);
```

### Excel Processing
Uses SheetJS (XLSX.js) library for reading/writing Excel files:
```javascript
// Read: XLSX.read(data, {type: 'binary'})
// Write: XLSX.writeFile(workbook, filename)
```

### CSV Processing
Uses PapaParse for CSV parsing:
```javascript
Papa.parse(file, { complete: (results) => processData(results.data) })
```

## Known Issues & Fixes

### Special Characters in Company Names
**Problem**: Apostrophes and quotes in company names were breaking JavaScript/HTML generation
**Fixed in**: customer_data_analyzer.html
**Solutions Implemented**:
1. Added `escapeHtml()` helper function (line 10895) for HTML content
2. Added `safeStringify()` method (line 3004) for JSON embedding in script tags
3. Fixed script tag concatenation using template literals: `</scr${'ipt>'}`

### Debug Code Removal
**Completed**: Removed 67 console.log/console.error/console.warn statements
**Status**: All debug code has been cleaned up

## Error Handling Patterns

### Safe HTML Embedding
```javascript
// Function to escape HTML special characters
function escapeHtml(text) {
    if (!text) return '';
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
}
```

### Safe JSON Stringification for Script Tags
```javascript
safeStringify(obj) {
    if (obj === null || obj === undefined) return 'null';
    let jsonStr = JSON.stringify(obj);
    // Escape </script> patterns to prevent breaking HTML parsing
    jsonStr = jsonStr.replace(/<\/script>/gi, '<\\/script>');
    return jsonStr;
}
```

### Script Tag in Template Literals
```javascript
// Correct way to include script tags in template literals
const html = `<scr${'ipt>'}console.log('safe');</scr${'ipt>'}`;
// Never use: `<script>...</script>` directly in template literals
```

## Testing Checklist

When modifying the code, test for:
- [ ] Files with special characters in names (apostrophes, quotes, ampersands)
- [ ] Large file processing (>10MB)
- [ ] Multiple file batch processing
- [ ] Export functionality (Excel generation)
- [ ] Drag and drop functionality
- [ ] Cross-browser compatibility (Chrome, Firefox, Edge, Safari)
- [ ] Empty or malformed CSV files

## Version Control

Current Version: 3.0
- All special character issues resolved
- Debug code removed
- Performance optimizations applied

## Contact

Created by Stephen Clarke
