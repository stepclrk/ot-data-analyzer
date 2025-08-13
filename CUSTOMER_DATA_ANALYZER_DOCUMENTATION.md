# Customer Data Analyzer - Comprehensive Documentation

## Table of Contents
1. [Application Overview](#application-overview)
2. [Technical Architecture](#technical-architecture)
3. [Getting Started](#getting-started)
4. [File Upload and Data Processing](#file-upload-and-data-processing)
5. [Analysis Tabs Overview](#analysis-tabs-overview)
6. [Detailed Feature Documentation](#detailed-feature-documentation)
7. [Data Visualization](#data-visualization)
8. [Export Capabilities](#export-capabilities)
9. [Interactive Features](#interactive-features)
10. [Technical Implementation](#technical-implementation)
11. [Troubleshooting](#troubleshooting)
12. [Best Practices](#best-practices)

---

## Application Overview

### Purpose
The Customer Data Analyzer is a comprehensive web-based analytics platform designed for processing and analyzing customer billing data, Trading Partner Reports (TPR), and Map configurations. It provides multi-dimensional analysis through 7 specialized views, offering insights into trading patterns, geographic distribution, and business relationships.

### Key Capabilities
- **Multi-file batch processing** for Excel (.xlsx) and CSV files
- **7 specialized analysis views** for comprehensive data exploration
- **Trading Partner Communications analysis** from TPR files
- **Map configuration analysis** from Trading Grid Cartographer exports
- **Interactive geographic mapping** with OpenStreetMap integration
- **AI-powered insights** generation
- **Advanced filtering and search** capabilities
- **Comprehensive export functionality** (Excel, CSV, JSON)
- **100% client-side processing** for data security

### Target Users
- Business Analysts
- Customer Success Teams  
- Operations Managers
- Support Representatives
- Trading Partner Managers
- Technical Administrators

---

## Technical Architecture

### Application Type
- **Platform**: Single-page web application (SPA)
- **Architecture**: Self-contained HTML with embedded CSS and JavaScript
- **Processing**: 100% client-side (browser-based)
- **Deployment**: Static file - no server required
- **File Size**: ~18,000 lines of code in single HTML file

### Core Dependencies
```javascript
// External Libraries (loaded via CDN)
- Chart.js v3.9.1         // Data visualization
- SheetJS (XLSX.js)       // Excel file processing  
- PapaParse               // CSV parsing
- Leaflet.js              // Interactive mapping
- OpenStreetMap           // Map tile provider
```

### Browser Requirements
- Chrome 80+ (recommended)
- Firefox 75+
- Safari 13+
- Edge 80+
- JavaScript enabled
- Minimum 4GB RAM for large datasets

---

## Getting Started

### Quick Start
1. **Open the application**: Double-click `customer_data_analyzer.html`
2. **Upload files**: Drag and drop Excel/CSV files onto the upload area
3. **Analyze**: Click "Analyze Files" button
4. **Explore**: Navigate through the 7 analysis tabs
5. **Export**: Save results in multiple formats

### File Requirements

#### Main Billing Data Files
**Format**: Excel (.xlsx) files  
**Naming Convention**: `CustomerName_DataType_YYYYMM.xlsx`  
**Required Sheets**:
- `Date_Summary` - Daily transaction summaries
- `Doc_Summary` - Document type analysis
- `Hub_Summary` - Hub/location data
- `TP_Summary` - Trading partner summaries
- `TP_Doc_Summary` - Partner document details

#### Optional Data Files

**ID Cross-Reference File**:
- Format: Excel or CSV
- Required columns: ID, Name, Region
- Purpose: Maps IDs to meaningful names

**Trading Partner Report (TPR)**:
- Format: CSV export from Trading Grid Executive Dashboard
- Contains: Monthly communication volumes by partner
- Columns: Partner details, communication paths, volumes

**Map Configuration File**:
- Format: CSV export from Trading Grid Cartographer
- Contains: Map configurations for all trading partners
- Columns: Direction, Sender ID, Receiver ID, Document Type, Map Name

---

## File Upload and Data Processing

### Upload Methods

#### Drag and Drop (Recommended)
```
1. Select files in Windows Explorer/Finder
2. Drag over the dotted upload area
3. Release when area highlights blue
4. Files begin processing automatically
```

#### Click to Browse
```
1. Click anywhere in upload area
2. Navigate to file location
3. Select files (Ctrl/Cmd for multiple)
4. Click "Open"
```

### Processing Pipeline
```
File Upload → Format Detection → Parsing → Validation → 
Data Extraction → Normalization → Analysis → Visualization
```

### Data Validation
- **File format verification** - Ensures .xlsx or .csv
- **Structure validation** - Checks required sheets/columns
- **Data type checking** - Validates dates, numbers, text
- **Consistency checks** - Ensures data integrity
- **Error reporting** - Specific line/column error details

---

## Analysis Tabs Overview

### 1. 📊 Overview Tab
**Purpose**: Executive summary and key insights  
**Features**:
- AI-generated insights and recommendations
- Key performance indicators (KPIs)
- Trend analysis with visual charts
- Seasonal pattern detection
- Month-over-month comparisons
- Export to Word-like format

### 2. 📈 Month over Month Tab
**Purpose**: Temporal analysis and trends  
**Features**:
- Monthly comparison tables
- Growth rate calculations
- Volume trends (documents and KCs)
- Interactive filtering by year
- Efficiency metrics over time

### 3. 🌐 Hub Analysis Tab
**Purpose**: Network and location performance  
**Features**:
- Hub performance metrics
- Top performers identification
- Monthly volume charts
- Document/KC ratios by hub
- Export capabilities

### 4. 🤝 TP Analysis Tab
**Purpose**: Trading partner relationships  
**Features**:
- Partner ranking by volume
- Market share analysis
- Growth trends by partner
- Partner concentration metrics
- Detailed partner profiles

### 5. 📄 Document Analysis Tab
**Purpose**: Document type intelligence  
**Features**:
- Document type distribution
- Volume analysis by type
- Efficiency metrics per document
- Trend analysis by document type
- Cross-partner comparisons

### 6. 📡 Trading Partner Communications Tab
**Purpose**: Communication path analysis (requires TPR file)  
**Features**:
- Communication method breakdown (VAN, API, WEB)
- Monthly volume trends by path
- Partner filtering and search
- Cross-reference with billing data
- Export filtered results

### 7. 🗺️ Map Analysis Tab
**Purpose**: Map configuration review (requires Map file)  
**Features**:
- Complete map inventory
- Direction analysis (Inbound/Outbound)
- Document type statistics
- Trading partner identification
- Advanced filtering with exact/partial match
- Cross-reference with active partners

---

## Detailed Feature Documentation

### Search and Filtering

#### Global Search
- Real-time search across all data fields
- Highlighted results
- Case-insensitive matching

#### Advanced Filters
```javascript
// Filter Types Available
- Text filters (contains, exact match)
- Numeric filters (greater than, less than, between)
- Date range filters
- Multi-select filters
- Combination filters (AND/OR logic)
```

#### Cross-Reference Filtering
- Filter maps/TPR data by active trading partners
- Show only data with billing activity
- Exclude inactive relationships

### Data Tables

#### Features
- **Sortable columns** - Click headers to sort
- **Resizable columns** - Drag column borders
- **Pagination** - 20/50/100 rows per page
- **Sticky headers** - Headers remain visible while scrolling
- **Row highlighting** - Hover effects for readability
- **Export selection** - Export visible or all data

#### Table Controls
```html
<!-- Standard table controls -->
- Search box
- Rows per page selector
- Export buttons (JSON/CSV)
- Clear filters button
- Exact match toggle
```

### Interactive Charts

#### Chart Types
- **Line Charts**: Trend analysis over time
- **Bar Charts**: Comparative analysis
- **Pie Charts**: Distribution visualization
- **Stacked Charts**: Component breakdown
- **Combo Charts**: Multiple metrics overlay

#### Interactivity
- Hover tooltips with detailed data
- Click to filter/drill down
- Legend toggling
- Zoom and pan capabilities
- Export as image (PNG)

---

## Data Visualization

### Chart.js Implementation
```javascript
// Standard chart configuration
const chartConfig = {
    type: 'line',
    data: {
        labels: dateLabels,
        datasets: [{
            label: 'Volume',
            data: volumeData,
            borderColor: '#3182ce',
            tension: 0.1
        }]
    },
    options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
            legend: { position: 'top' },
            tooltip: { mode: 'index' }
        }
    }
};
```

### Color Coding System
- **Green (#38a169)**: Positive growth/good performance
- **Red (#e53e3e)**: Negative growth/needs attention
- **Blue (#3182ce)**: Neutral/informational
- **Yellow (#f6ad55)**: Warning/caution
- **Gray (#718096)**: Inactive/disabled

### Performance Indicators
- **Growth arrows**: ↑ positive, ↓ negative
- **Status badges**: Active, Inactive, Pending
- **Progress bars**: Visual completion indicators
- **Heat maps**: Intensity-based coloring

---

## Export Capabilities

### Export Formats

#### Excel Export (.xlsx)
```javascript
// Multi-sheet workbook structure
- Summary sheet with KPIs
- Detailed data sheets
- Charts as images
- Formatted with colors and styles
- Preserved formulas where applicable
```

#### CSV Export
```javascript
// Standard CSV features
- Configurable delimiters
- Header row inclusion
- Quoted text fields
- UTF-8 encoding
- Large dataset support
```

#### JSON Export
```javascript
// Structured JSON output
{
    "metadata": {
        "exportDate": "2025-08-13",
        "recordCount": 1000,
        "filters": []
    },
    "data": [...]
}
```

### Specialized Exports

#### Save Analysis Report
- Complete HTML report with all tabs
- Embedded charts and visualizations
- Preserves interactivity
- Includes all processed data
- Can be reopened later

#### Filtered Exports
- Export only visible/filtered data
- Maintains filter context
- Includes filter criteria in export

---

## Interactive Features

### Drag and Drop
- File upload via drag and drop
- Visual feedback during drag
- Multi-file support
- Progress indicators

### Keyboard Shortcuts
- `Ctrl+F`: Focus search box
- `Escape`: Close modals
- `Tab`: Navigate between controls
- `Enter`: Submit forms

### Context Menus
- Right-click on tables for options
- Copy cell values
- Export selected rows
- Filter by cell value

### Responsive Design
- Adapts to screen size
- Mobile-friendly tables
- Touch-enabled controls
- Collapsible sections

---

## Technical Implementation

### Core Classes and Functions

#### DataAnalyzer Class
```javascript
class DataAnalyzer {
    constructor() {
        this.customerName = '';
        this.analysisData = {};
        this.periods = [];
    }
    
    async processFiles(files) {
        // Main processing logic
    }
    
    generateAnalysis(data) {
        // Analysis generation
    }
    
    exportResults(format) {
        // Export functionality
    }
}
```

#### Key Processing Functions
```javascript
// File processing
handleFileUpload(files)
parseExcelFile(file)
parseCSVFile(file)
validateData(data)

// Analysis functions
calculateGrowthRate(current, previous)
detectSeasonality(data)
generateInsights(metrics)

// Export functions
exportToExcel(data)
exportToCSV(data)
exportToJSON(data)
```

### Memory Management
- Chunked processing for large files
- Garbage collection optimization
- Lazy loading of chart data
- Virtual scrolling for tables

### Error Handling
```javascript
try {
    // Processing logic
} catch (error) {
    console.error('Processing error:', error);
    showErrorModal(error.message);
}
```

---

## Troubleshooting

### Common Issues and Solutions

#### File Upload Issues
| Issue | Solution |
|-------|----------|
| Files not uploading | Check file format (.xlsx, .csv only) |
| Upload fails | Verify file size < 50MB |
| Parsing errors | Ensure file isn't corrupted |
| Missing data | Check required sheets/columns exist |

#### Performance Issues
| Issue | Solution |
|-------|----------|
| Slow processing | Process smaller batches |
| Browser freezing | Close other tabs |
| Memory errors | Restart browser |
| Charts not loading | Check JavaScript console |

#### Export Problems
| Issue | Solution |
|-------|----------|
| Export button not working | Disable pop-up blocker |
| Corrupted exports | Check disk space |
| Missing data in export | Verify filters |
| Format issues | Try different export type |

### Debug Mode
Enable debug mode by appending `?debug=true` to the URL:
```
file:///path/to/customer_data_analyzer.html?debug=true
```

### Browser Console Commands
```javascript
// Useful debugging commands
analyzer.getProcessedData()  // View processed data
analyzer.clearCache()        // Clear cached data
analyzer.exportDebugLog()    // Export debug information
```

---

## Best Practices

### Data Preparation
1. **Clean your data** before upload
   - Remove empty rows
   - Fix formatting issues
   - Standardize date formats

2. **Use consistent naming**
   - Follow naming convention
   - Keep customer names consistent
   - Use standard date formats (YYYYMM)

3. **Validate data quality**
   - Check for missing values
   - Verify numeric fields
   - Ensure required columns exist

### Performance Optimization
1. **File management**
   - Keep files under 50MB
   - Process in batches of 12 or less
   - Use modern file formats (.xlsx over .xls)

2. **Browser optimization**
   - Use Chrome or Edge for best performance
   - Close unnecessary tabs
   - Clear cache periodically
   - Disable unnecessary extensions

3. **Analysis workflow**
   - Start with Overview tab
   - Progress through tabs systematically
   - Export important findings early
   - Save analysis reports regularly

### Security Considerations
- **Data Privacy**: All processing is client-side
- **No Data Transmission**: Nothing sent to servers
- **Local Storage**: Data temporarily cached locally
- **Secure Cleanup**: Data cleared on page refresh

---

## Advanced Features

### AI-Powered Insights
The application generates intelligent insights including:
- Trend identification
- Anomaly detection
- Seasonal pattern recognition
- Performance recommendations
- Growth predictions

### Cross-Reference Capabilities
- Link IDs to meaningful names
- Regional grouping and analysis
- Dynamic display switching (ID/Name/Region)
- Enhanced filtering options

### Map Integration
- Geographic visualization of data
- Interactive OpenStreetMap integration
- Custom markers and clustering
- Regional performance overlay

---

## Version History

### Version 3.0 (Current)
- Added Trading Partner Communications tab
- Added Map Analysis tab
- Enhanced filtering capabilities
- Improved performance optimization
- Fixed special character handling

### Version 2.0
- Complete JavaScript rewrite
- Removed Python dependencies
- Added Hub Analysis
- Enhanced export capabilities

### Version 1.0
- Initial release
- Basic analysis features
- Excel/CSV support

---

## Support and Resources

### File Format Templates
Example files available in `/example` directory:
- Sample billing data files
- ID cross-reference template
- TPR file example
- Map configuration sample

### Known Limitations
- Maximum file size: 50MB recommended
- Browser memory limits for very large datasets
- Internet required for map tiles only
- Some features require modern browser support

### Contact Information
**Created by**: Stephen Clarke  
**Documentation Version**: 1.0  
**Last Updated**: August 2025  
**License**: Proprietary - Internal Use Only

---

## Appendix

### Glossary of Terms
- **KC**: Kilocharacters (1,000 characters)
- **DOC**: Documents processed
- **TP**: Trading Partner
- **TPR**: Trading Partner Report
- **Hub**: Processing location/network node
- **Map**: EDI map configuration

### File Structure Reference
```
customer_data_analyzer.html
├── Styles (lines 1-500)
├── HTML Structure (lines 500-1000)
├── Menu System (lines 1000-1500)
├── File Processing (lines 1500-3000)
├── Analysis Generation (lines 3000-11000)
├── Visualization (lines 11000-15000)
├── Export Functions (lines 15000-17000)
└── Utility Functions (lines 17000-18000)
```

### Performance Benchmarks
- Small dataset (<1MB): < 1 second
- Medium dataset (1-10MB): 2-5 seconds
- Large dataset (10-50MB): 10-30 seconds
- Export generation: 1-5 seconds

---

*End of Documentation*