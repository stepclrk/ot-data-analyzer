# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a customer traffic data analysis toolkit consisting of web-based tools for analyzing and converting customer billing data. Despite the directory name "KC_CSV_Java", this is purely a JavaScript/HTML project with no Java components.

## Key Components

### Main Application
- **customer_data_analyzer.html**: Primary analysis tool that processes customer Excel/CSV files and generates comprehensive reports with visualizations including:
  - Overview dashboard with key metrics and insights
  - Trading Partner (TP) analysis with ID cross-reference support
  - Hub analysis with network performance metrics
  - Document analysis by type
  - Multi-year seasonality patterns
  - Sortable data tables throughout
  - ID Cross-Reference file support for mapping IDs to Names/Regions

### Data Converters
- **Classic_to_TG_Billing_format_converter.html**: Converts MS Classic CSV files to TG Billing XLSX format
- **csv_to_xlsx_converter.html**: Converts TG CSV exports to structured XLSX with multiple sheets

### Landing Page
- **index.html**: Professional landing page with animated backgrounds and navigation to all tools

### Configuration & Utilities
- **js/config.js**: Centralized configuration including file validation rules, sheet mappings, chart colors, and analysis thresholds
- **js/utils.js**: Common utility functions (debounce, formatBytes, formatNumber, etc.)

## Key Commands

### Running the Application

```bash
# Open main analyzer
open customer_data_analyzer.html              # macOS
xdg-open customer_data_analyzer.html         # Linux  
start customer_data_analyzer.html            # Windows

# Open converters
open Classic_to_TG_Billing_format_converter.html
open csv_to_xlsx_converter.html

# Open landing page
open index.html
```

### Development Commands

No build system is configured. To verify JavaScript syntax:
```bash
# Check for syntax errors (requires Node.js)
node -c customer_data_analyzer.html         # Check embedded JS
node -c js/config.js                       # Check config module
node -c js/utils.js                        # Check utilities
```

## Architecture

### Data Processing Pipeline
1. File Upload → Parse Excel/CSV data using XLSX.js/PapaParse
2. Extract customer, date, amount, document, and trading partner information
3. Calculate metrics (documents, kilocharacters)
4. Generate visualizations using Chart.js
5. Create HTML report in iframe with interactive elements

### External Dependencies (CDN-loaded)
- **XLSX.js** (v0.18.5): Excel file parsing and generation
- **PapaParse** (v5.4.1): CSV parsing
- **Chart.js** (v3.9.1): Data visualization

### Expected Data Format

#### Main Data Files
Customer data files should contain these sheets:
- `Date_Summary`: Time-based analysis data
- `Doc_Summary`: Document type summaries
- `Hub_Summary`: Hub/network location data
- `TP_Summary`: Trading partner summaries
- `TP_Doc_Summary`: Trading partner by document type matrix

Required columns:
- `Customer`: Customer identifier (e.g., "KCC102")
- `Invoice`: Invoice reference
- `Amount`: Numeric amount value
- `Date`: Date in various formats (MM/DD/YYYY, YYYY-MM-DD, etc.)
- `Trade_Partner`: Trading partner identifier
- `Document`: Document type

#### ID Cross-Reference File (Optional)
Excel or CSV file with columns:
- `ID`: Trading partner or hub identifier
- `Name`: Descriptive name for the ID
- `Region`: Geographical region or business unit

### Report Generation Architecture

The report is generated in an iframe (`reportFrame`) with:
- Dynamic HTML generation with embedded CSS/JS
- Cross-window communication for data passing
- Export functions injected into iframe context
- Table sorting functions dynamically generated per table
- ID/Name/Region switching via `setTPDisplayMode()` function

### Key Features

1. **Performance Scorecard**: Combined health assessment using both document count and kilocharacter metrics
2. **Multi-Year Seasonality**: Automatic year detection with filtering capabilities
3. **Universal Table Sorting**: All tables support click-to-sort functionality
4. **Efficiency Scoring**: KC per Document metrics with explanatory tooltips
5. **Download All**: Batch download for multiple converted files (avoids ZIP security warnings)
6. **ID Cross-Reference**: Dynamic switching between ID, Name, and Region display in tables

### Common Development Tasks

When modifying the analyzer:
1. Main report generation logic is embedded in `customer_data_analyzer.html`
2. Chart configurations are inline within the HTML report generation
3. Table sorting is implemented via dynamically generated functions in the report
4. Cross-reference functionality uses `window.idCrossReference` object
5. The converter tools are self-contained single HTML files with embedded JavaScript
6. Configuration changes should be made in `js/config.js` for consistency
