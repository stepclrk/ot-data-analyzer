# Customer Data Analyzer - Detailed Code Documentation

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Code Structure](#code-structure)
3. [Core Classes](#core-classes)
4. [Global Variables and State](#global-variables-and-state)
5. [Function Reference](#function-reference)
6. [CSS Architecture](#css-architecture)
7. [HTML Components](#html-components)
8. [External Dependencies](#external-dependencies)
9. [Data Flow and Processing](#data-flow-and-processing)
10. [Event Handling](#event-handling)
11. [Error Handling Patterns](#error-handling-patterns)
12. [Performance Optimizations](#performance-optimizations)
13. [Security Implementation](#security-implementation)
14. [Memory Management](#memory-management)
15. [Code Patterns](#code-patterns)

---

## Architecture Overview

### Application Metrics
- **Total Lines of Code**: ~17,800
- **File Size**: ~837KB
- **Language Composition**: HTML (5%), CSS (8%), JavaScript (87%)
- **Dependencies**: 4 external libraries via CDN
- **Browser APIs Used**: FileReader, Blob, URL, LocalStorage

### Design Architecture
```javascript
// Single-Page Application (SPA) Architecture
customer_data_analyzer.html
├── Self-Contained Deployment (no build process)
├── Client-Side Processing (no server required)
├── Modular JavaScript Classes
├── Event-Driven UI Updates
└── Asynchronous File Processing
```

---

## Code Structure

### File Organization (Line Numbers)
```
Lines 1-9:       HTML DOCTYPE and Meta Tags
Lines 10-481:    CSS Styles and Animations
Lines 482-594:   Modal HTML Templates
Lines 595-611:   Menu System HTML
Lines 612-1481:  Main Application HTML
Lines 1482-1787: Initialization and Setup
Lines 1788-2476: File Handling Functions
Lines 2477-4170: DynamicDataAnalyzer Class
Lines 4171-17550: Report Generation Functions
Lines 17551-17814: Utility and Helper Functions
```

---

## Core Classes

### DynamicDataAnalyzer Class (Lines 2477-4170)

#### Constructor (Lines 2477-2506)
```javascript
class DynamicDataAnalyzer {
    constructor() {
        // Core Properties
        this.dataFolder = "data";                     // Data directory path
        this.customerName = null;                     // Extracted from filename
        this.filePattern = null;                      // Regex pattern for files
        this.fileType = null;                         // 'excel' or 'csv'
        this.sheetNames = [];                         // Available sheet names
        this.allData = {};                            // Main data storage
        
        // Performance Optimization
        this.lazyLoadedSheets = new Map();           // Sheet cache
        this.fileReferences = new Map();             // File object references
        
        // Configuration
        this.sheetPatterns = [                       // Sheet detection patterns
            "hub", "tp", "doc", "summary", "date",
            "trading partner", "document", "hub id"
        ];
        
        this.sheetNameMapping = {                    // Standardized names
            'Date_Summary': 'DATE SUMMARY',
            'Doc_Summary': 'DOCUMENT SUMMARY',
            'Hub_Summary': 'HUB ID SUMMARY',
            'TP_Summary': 'TRADING PARTNER SUMMARY',
            'TP_Doc_Summary': 'TRADING PARTNER/DOCUMENT SUMMARY'
        };
        
        // API Integration
        this.claudeApiKey = localStorage.getItem('claudeApiKey') || '';
        
        // Report State
        this.reportData = null;
        this.lastAnalysisData = null;
    }
}
```

#### Key Methods

##### detectCustomerAndPattern(files) - Lines 2508-2570
```javascript
async detectCustomerAndPattern(files) {
    // Extract customer name from first file
    const firstFileName = files[0].name;
    
    // Pattern: CustomerName_Type_Period.extension
    const match = firstFileName.match(/^([^_]+)_.*_(\d{6,8})\.(xlsx?|csv)$/);
    
    if (match) {
        this.customerName = match[1];
        this.fileType = match[3].toLowerCase().includes('xls') ? 'excel' : 'csv';
        
        // Build pattern for all files
        this.filePattern = new RegExp(
            `^${this.customerName}_.*_\\d{6,8}\\.${this.fileType === 'excel' ? 'xlsx?' : 'csv'}$`
        );
        
        return true;
    }
    
    throw new Error('Unable to detect customer name pattern');
}
```

##### extractSheetData(file, sheetName) - Lines 2572-2650
```javascript
async extractSheetData(file, sheetName) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        
        reader.onload = async (e) => {
            try {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, {type: 'array', cellDates: true});
                
                // Find matching sheet
                const actualSheetName = this.findSheetName(workbook.SheetNames, sheetName);
                
                if (!actualSheetName) {
                    resolve({headers: [], data: [], metrics: null});
                    return;
                }
                
                // Convert to JSON with header detection
                const jsonData = XLSX.utils.sheet_to_json(
                    workbook.Sheets[actualSheetName], 
                    {header: 1, defval: null, raw: false}
                );
                
                // Process and structure data
                const headers = jsonData[0] || [];
                const dataRows = jsonData.slice(1).filter(row => 
                    !this.isRowEmpty(row) && !this.isTotalRow(row)
                );
                
                // Calculate metrics
                const metrics = this.calculateSheetMetrics(headers, dataRows);
                
                resolve({
                    headers: headers,
                    data: dataRows,
                    metrics: metrics,
                    sheetName: actualSheetName
                });
            } catch (error) {
                reject(error);
            }
        };
        
        reader.readAsArrayBuffer(file);
    });
}
```

##### processAllFiles(files) - Lines 2652-2780
```javascript
async processAllFiles(files) {
    const processedData = {};
    const totalFiles = files.length;
    let processedCount = 0;
    
    // Group files by period
    const filesByPeriod = this.groupFilesByPeriod(files);
    
    for (const [period, periodFiles] of filesByPeriod.entries()) {
        processedData[period] = {};
        
        for (const file of periodFiles) {
            // Update progress
            processedCount++;
            const progress = 35 + (processedCount / totalFiles * 35);
            updateProgress(progress, `Processing ${file.name}...`, 'Processing');
            
            // Extract all sheets from file
            for (const sheetName of this.sheetNames) {
                const sheetData = await this.extractSheetData(file, sheetName);
                
                if (sheetData.data.length > 0) {
                    if (!processedData[period][sheetName]) {
                        processedData[period][sheetName] = [];
                    }
                    processedData[period][sheetName].push(...sheetData.data);
                }
            }
        }
    }
    
    return processedData;
}
```

##### generateReport(data) - Lines 2782-4170
```javascript
async generateReport(data) {
    updateProgress(75, 'Generating comprehensive report...', 'Analysis');
    
    // Build report structure
    const report = {
        customerName: this.customerName,
        analysisDate: new Date().toISOString(),
        periods: Object.keys(data).sort(),
        summary: await this.generateSummary(data),
        detailAnalysis: await this.generateDetailAnalysis(data),
        entityAnalysis: await this.generateEntityAnalysis(data),
        tradingPartnerAnalysis: await this.generateTradingPartnerAnalysis(data),
        documentAnalysis: await this.generateDocumentAnalysis(data),
        trends: await this.analyzeTrends(data),
        insights: await this.generateInsights(data)
    };
    
    // Generate HTML report
    const htmlReport = this.renderHTMLReport(report);
    
    updateProgress(95, 'Finalizing report...', 'Completion');
    
    return htmlReport;
}
```

---

## Global Variables and State

### File Management Variables (Lines 1482-1487)
```javascript
// File Storage
let uploadedFiles = [];              // Array<File> - User uploaded files
let crossReferenceData = null;       // Object - ID to Name/Region mapping
let tprData = null;                  // Object - Trading Partner Report data
let mapData = null;                  // Object - Map configuration data

// Processing State
let analyzer = null;                 // DynamicDataAnalyzer instance
let currentReport = null;            // Generated HTML report
let isProcessing = false;            // Processing flag
```

### DOM References (Lines 1489-1520)
```javascript
// Upload Areas
const uploadArea = document.getElementById('uploadArea');
const fileInput = document.getElementById('fileInput');
const xrefUploadArea = document.getElementById('xrefUploadArea');
const tprUploadArea = document.getElementById('tprUploadArea');
const mapUploadArea = document.getElementById('mapUploadArea');

// UI Elements
const fileList = document.getElementById('fileList');
const analyzeBtn = document.getElementById('analyzeBtn');
const reportFrame = document.getElementById('reportFrame');
const progressContainer = document.getElementById('progressContainer');
const progressBar = document.getElementById('progressBar');
const progressStatus = document.getElementById('progressStatus');
```

### Configuration Constants (Lines 1522-1540)
```javascript
// Validation Rules
const FILE_VALIDATION = {
    maxFileSize: 50 * 1024 * 1024,           // 50MB limit
    minFileSize: 100,                        // 100 bytes minimum
    allowedExtensions: ['.xlsx', '.xls', '.xlsm', '.csv'],
    requiredSheets: ['Date_Summary', 'Doc_Summary', 'Hub_Summary', 
                     'TP_Summary', 'TP_Doc_Summary']
};

// Processing Configuration
const PROCESSING_CONFIG = {
    chunkSize: 1000,                         // Rows per chunk
    maxConcurrentFiles: 5,                   // Parallel processing limit
    progressUpdateInterval: 100,             // MS between updates
    memoryThreshold: 500 * 1024 * 1024      // 500MB memory limit
};
```

---

## Function Reference

### File Handling Functions

#### handleFiles(files) - Lines 1788-1855
```javascript
function handleFiles(files) {
    const errors = [];
    const validFiles = [];
    
    // Validation pipeline
    for (const file of files) {
        // 1. Check file size
        if (file.size > FILE_VALIDATION.maxFileSize) {
            errors.push(`${file.name}: exceeds 50MB limit`);
            continue;
        }
        
        // 2. Check file extension
        const ext = file.name.toLowerCase().match(/\.[^.]+$/)?.[0];
        if (!FILE_VALIDATION.allowedExtensions.includes(ext)) {
            errors.push(`${file.name}: unsupported file type`);
            continue;
        }
        
        // 3. Check filename format
        const formatCheck = validateFilenameFormat(file.name);
        if (!formatCheck.isValid) {
            errors.push(`${file.name}: ${formatCheck.message}`);
            continue;
        }
        
        validFiles.push(file);
    }
    
    // Update state
    uploadedFiles = [...uploadedFiles, ...validFiles];
    updateFileList();
    
    // Show errors if any
    if (errors.length > 0) {
        showStatus(errors.join('\n'), 'error');
    }
}
```

#### validateFilenameFormat(filename) - Lines 1856-1871
```javascript
function validateFilenameFormat(filename) {
    // Expected: CustomerName_Type_YYYYMM.extension
    const idealPattern = /^[A-Za-z0-9]+_[A-Za-z]+_\d{6,8}\.(xlsx?|csv)$/;
    const acceptablePattern = /^.+\.(xlsx?|xlsm?|csv)$/i;
    
    if (idealPattern.test(filename)) {
        return { isValid: true, message: 'Valid format' };
    } else if (acceptablePattern.test(filename)) {
        return { isValid: true, message: 'Acceptable format (not ideal)' };
    } else {
        return { 
            isValid: false, 
            message: 'Expected format: CustomerName_Type_YYYYMM.xlsx' 
        };
    }
}
```

#### updateFileList() - Lines 1872-1925
```javascript
function updateFileList() {
    const fileListElement = document.getElementById('fileList');
    const fileCountElement = document.getElementById('fileCount');
    
    // Clear existing list
    fileListElement.innerHTML = '';
    
    // Display each file
    uploadedFiles.forEach((file, index) => {
        const fileItem = document.createElement('div');
        fileItem.className = 'file-item';
        
        // File info
        const fileInfo = document.createElement('span');
        fileInfo.textContent = `${file.name} (${formatFileSize(file.size)})`;
        
        // Remove button
        const removeBtn = document.createElement('button');
        removeBtn.textContent = '×';
        removeBtn.className = 'remove-btn';
        removeBtn.onclick = () => removeFile(index);
        
        fileItem.appendChild(fileInfo);
        fileItem.appendChild(removeBtn);
        fileListElement.appendChild(fileItem);
    });
    
    // Update count
    fileCountElement.textContent = `${uploadedFiles.length} file(s) selected`;
    
    // Enable/disable analyze button
    analyzeBtn.disabled = uploadedFiles.length === 0;
}
```

### Cross-Reference Processing

#### handleXrefFile(file) - Lines 1926-2039
```javascript
async function handleXrefFile(file) {
    const reader = new FileReader();
    
    return new Promise((resolve, reject) => {
        reader.onload = async (e) => {
            try {
                let data;
                
                // Detect file type and parse accordingly
                if (file.name.endsWith('.csv')) {
                    // Parse CSV with PapaParse
                    const result = Papa.parse(e.target.result, {
                        header: true,
                        dynamicTyping: true,
                        skipEmptyLines: true
                    });
                    data = result.data;
                } else {
                    // Parse Excel with XLSX
                    const workbook = XLSX.read(e.target.result, {type: 'binary'});
                    const firstSheet = workbook.Sheets[workbook.SheetNames[0]];
                    data = XLSX.utils.sheet_to_json(firstSheet);
                }
                
                // Process cross-reference data
                crossReferenceData = processXrefData(data);
                
                // Update UI
                updateXrefStatus(true);
                
                resolve(crossReferenceData);
            } catch (error) {
                reject(error);
            }
        };
        
        if (file.name.endsWith('.csv')) {
            reader.readAsText(file);
        } else {
            reader.readAsBinaryString(file);
        }
    });
}
```

### Trading Partner Report Processing

#### handleTprFile(file) - Lines 2040-2175
```javascript
async function handleTprFile(file) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        
        reader.onload = async (e) => {
            try {
                let rawData = [];
                
                // Parse file based on type
                if (file.name.toLowerCase().endsWith('.csv')) {
                    const result = Papa.parse(e.target.result, {
                        header: false,
                        skipEmptyLines: true
                    });
                    rawData = result.data;
                } else {
                    const workbook = XLSX.read(e.target.result, {type: 'binary'});
                    const sheet = workbook.Sheets[workbook.SheetNames[0]];
                    rawData = XLSX.utils.sheet_to_json(sheet, {header: 1});
                }
                
                // Process TPR data structure
                const processed = processTprData(rawData);
                
                // Store globally
                window.tprData = {
                    raw: processed.data,
                    mailboxTypeColumn: processed.mailboxColumn,
                    dateColumns: processed.dateColumns,
                    partnerColumns: processed.partnerColumns,
                    fileName: file.name
                };
                
                resolve(window.tprData);
            } catch (error) {
                reject(error);
            }
        };
        
        reader.readAsBinaryString(file);
    });
}
```

### Map Configuration Processing

#### handleMapFile(file) - Lines 2176-2275
```javascript
async function handleMapFile(file) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        
        reader.onload = async (e) => {
            try {
                let data;
                
                // Parse based on file type
                if (file.name.toLowerCase().endsWith('.csv')) {
                    const result = Papa.parse(e.target.result, {
                        header: true,
                        dynamicTyping: true,
                        skipEmptyLines: true
                    });
                    data = result.data;
                } else {
                    const workbook = XLSX.read(e.target.result, {type: 'binary'});
                    const sheet = workbook.Sheets[workbook.SheetNames[0]];
                    data = XLSX.utils.sheet_to_json(sheet);
                }
                
                // Process map configurations
                const processedMaps = data.map(row => ({
                    direction: row.Direction || row.DIRECTION || '',
                    senderId: row['Sender ID'] || row.SENDER_ID || '',
                    receiverId: row['Receiver ID'] || row.RECEIVER_ID || '',
                    docType: row['Document Type'] || row.DOC_TYPE || '',
                    mapName: row['Map Name'] || row.MAP_NAME || row.Name || '',
                    tradingPartner: determineTPFromDirection(row)
                }));
                
                // Store globally
                window.mapData = {
                    processed: processedMaps,
                    fileName: file.name
                };
                
                resolve(window.mapData);
            } catch (error) {
                reject(error);
            }
        };
        
        reader.readAsBinaryString(file);
    });
}
```

### Progress Management Functions

#### initializeProgress() - Lines 2401-2410
```javascript
function initializeProgress() {
    const progressContainer = document.getElementById('progressContainer');
    const progressBar = document.getElementById('progressBar');
    const progressStatus = document.getElementById('progressStatus');
    
    progressContainer.style.display = 'block';
    progressBar.style.width = '0%';
    progressBar.textContent = '0%';
    progressStatus.textContent = 'Initializing...';
    
    // Force browser repaint
    progressContainer.offsetHeight;
}
```

#### updateProgress(percentage, status, stepName) - Lines 2411-2430
```javascript
function updateProgress(percentage, status, stepName) {
    const progressBar = document.getElementById('progressBar');
    const progressStatus = document.getElementById('progressStatus');
    const currentStep = document.getElementById('currentStep');
    
    // Update bar
    progressBar.style.width = percentage + '%';
    progressBar.textContent = Math.round(percentage) + '%';
    
    // Update status text
    if (status) {
        progressStatus.textContent = status;
    }
    
    // Update step name
    if (stepName && currentStep) {
        currentStep.textContent = stepName;
    }
    
    // Apply color based on progress
    if (percentage < 30) {
        progressBar.style.background = 'linear-gradient(90deg, #667eea 0%, #764ba2 100%)';
    } else if (percentage < 70) {
        progressBar.style.background = 'linear-gradient(90deg, #f093fb 0%, #f5576c 100%)';
    } else {
        progressBar.style.background = 'linear-gradient(90deg, #4facfe 0%, #00f2fe 100%)';
    }
}
```

### Analysis Functions

#### analyzeFiles() - Lines 17005-17120
```javascript
async function analyzeFiles() {
    if (uploadedFiles.length === 0) {
        showStatus('Please upload files first', 'error');
        return;
    }
    
    try {
        // Initialize progress
        initializeProgress();
        updateProgress(5, 'Starting analysis...', 'Initialization');
        
        // Create analyzer instance
        analyzer = new DynamicDataAnalyzer();
        
        // Step 1: Detect customer and pattern
        updateProgress(10, 'Detecting customer patterns...', 'Pattern Detection');
        const detected = await analyzer.detectCustomerAndPattern(uploadedFiles);
        
        if (!detected) {
            throw new Error('Could not detect customer name pattern');
        }
        
        // Step 2: Process all files
        updateProgress(35, 'Processing files...', 'File Processing');
        const processedData = await analyzer.processAllFiles(uploadedFiles);
        
        // Step 3: Generate analysis
        updateProgress(70, 'Performing analysis...', 'Analysis');
        const report = await analyzer.generateReport(processedData);
        
        // Step 4: Display report
        updateProgress(90, 'Rendering report...', 'Rendering');
        displayReport(report);
        
        // Complete
        updateProgress(100, 'Analysis complete!', 'Complete');
        setTimeout(hideProgress, 2000);
        
    } catch (error) {
        console.error('Analysis error:', error);
        showStatus('Analysis failed: ' + error.message, 'error');
        hideProgress();
    }
}
```

---

## CSS Architecture

### Design System Variables (Lines 10-25)
```css
:root {
    /* Color Palette */
    --primary-blue: #1a365d;
    --secondary-blue: #2c5282;
    --accent-purple: #667eea;
    --success-green: #38a169;
    --warning-orange: #dd6b20;
    --error-red: #e53e3e;
    --neutral-gray: #f5f5f5;
    --text-primary: #2d3748;
    --text-secondary: #718096;
    
    /* Spacing */
    --spacing-xs: 4px;
    --spacing-sm: 8px;
    --spacing-md: 16px;
    --spacing-lg: 24px;
    --spacing-xl: 32px;
    
    /* Border Radius */
    --radius-sm: 4px;
    --radius-md: 8px;
    --radius-lg: 16px;
    
    /* Shadows */
    --shadow-sm: 0 1px 3px rgba(0,0,0,0.12);
    --shadow-md: 0 4px 6px rgba(0,0,0,0.1);
    --shadow-lg: 0 10px 25px rgba(0,0,0,0.15);
}
```

### Component Styles

#### Upload Area Styles (Lines 100-150)
```css
.upload-area {
    border: 2px dashed #cbd5e0;
    border-radius: var(--radius-lg);
    padding: 40px;
    text-align: center;
    cursor: pointer;
    transition: all 0.3s ease;
    background: linear-gradient(135deg, #f6f9fc 0%, #ffffff 100%);
}

.upload-area:hover {
    border-color: var(--accent-purple);
    background: linear-gradient(135deg, #eef2ff 0%, #ffffff 100%);
    transform: translateY(-2px);
    box-shadow: var(--shadow-lg);
}

.upload-area.drag-over {
    border-color: var(--success-green);
    background: linear-gradient(135deg, #f0fdf4 0%, #ffffff 100%);
    animation: pulse 1s infinite;
}

@keyframes pulse {
    0% { transform: scale(1); }
    50% { transform: scale(1.02); }
    100% { transform: scale(1); }
}
```

#### Modal Styles (Lines 200-250)
```css
.modal {
    display: none;
    position: fixed;
    z-index: 1000;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.6);
    backdrop-filter: blur(5px);
    animation: fadeIn 0.3s ease;
}

.modal-content {
    position: relative;
    background: white;
    margin: 5% auto;
    padding: 30px;
    width: 90%;
    max-width: 800px;
    border-radius: var(--radius-lg);
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
    animation: slideIn 0.3s ease;
    max-height: 80vh;
    overflow-y: auto;
}

@keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
}

@keyframes slideIn {
    from { 
        transform: translateY(-50px);
        opacity: 0;
    }
    to { 
        transform: translateY(0);
        opacity: 1;
    }
}
```

---

## HTML Components

### Menu System (Lines 595-611)
```html
<!-- Menu Button -->
<button class="menu-button" onclick="toggleMenu()">☰ Menu</button>

<!-- Menu Dropdown -->
<div id="menuDropdown" class="menu-dropdown">
    <div class="menu-item" onclick="showModal('instructionsModal')">📖 Instructions</div>
    <div class="menu-item" onclick="showModal('explanationsModal')">📊 Tool Explanations</div>
    <div class="menu-item" onclick="showModal('aboutModal')">ℹ️ About</div>
    <div class="menu-item has-submenu" onclick="toggleConverterSubmenu(event)">
        🔄 Converter ▶
        <div class="submenu">
            <a href="./csv_to_xlsx_converter.html" class="submenu-item">CSV to Excel</a>
            <a href="./Classic_to_TG_Billing_format_converter.html" class="submenu-item">
                Classic to TG Format
            </a>
        </div>
    </div>
    <div class="menu-item" onclick="goHome()">🏠 Home</div>
</div>
```

### Upload Interface (Lines 1398-1445)
```html
<div class="upload-container" id="analyzerSection">
    <!-- Main File Upload -->
    <div class="upload-section">
        <h3>📁 Upload Files</h3>
        <div class="upload-area" id="uploadArea">
            <input type="file" id="fileInput" multiple 
                   accept=".xlsx,.xls,.xlsm,.csv" style="display: none;">
            <div class="upload-icon">📤</div>
            <p class="upload-text">
                Drag and drop your Excel files here or click to browse
            </p>
            <p class="upload-hint">
                Supports .xlsx, .xls, .xlsm, and .csv files
            </p>
        </div>
        <div id="fileList" class="file-list"></div>
        <div id="fileCount" class="file-count">0 file(s) selected</div>
    </div>
    
    <!-- Optional Upload Sections -->
    <div class="optional-uploads">
        <!-- Cross-Reference Upload -->
        <div class="xref-upload-section">
            <!-- Content -->
        </div>
        
        <!-- TPR Upload -->
        <div class="tpr-upload-section">
            <!-- Content -->
        </div>
        
        <!-- Map Upload -->
        <div class="map-upload-section">
            <!-- Content -->
        </div>
    </div>
</div>
```

---

## External Dependencies

### SheetJS/XLSX.js Integration
```javascript
// Library: xlsx.full.min.js v0.18.5
// CDN: https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js

// Read Excel file
const workbook = XLSX.read(data, {
    type: 'array',        // Input type
    cellDates: true,      // Parse dates
    cellNF: false,        // Number formats
    cellStyles: false     // Cell styles
});

// Convert sheet to JSON
const jsonData = XLSX.utils.sheet_to_json(worksheet, {
    header: 1,           // Use first row as header
    defval: null,        // Default value for empty cells
    raw: false,          // Use formatted strings
    dateNF: 'yyyy-mm-dd' // Date format
});

// Write Excel file
XLSX.writeFile(workbook, filename, {
    bookType: 'xlsx',    // Output format
    compression: true    // Use compression
});
```

### PapaParse Integration
```javascript
// Library: papaparse.min.js v5.4.1
// CDN: https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js

// Parse CSV
Papa.parse(csvString, {
    header: true,          // First row contains headers
    dynamicTyping: true,   // Auto-detect data types
    skipEmptyLines: true,  // Ignore empty rows
    delimiter: ',',        // Field delimiter
    complete: function(results) {
        // results.data contains parsed data
        // results.errors contains any errors
    }
});

// Generate CSV
const csv = Papa.unparse(data, {
    quotes: true,          // Quote fields
    delimiter: ',',        // Field delimiter
    header: true          // Include header row
});
```

### Chart.js Integration
```javascript
// Library: chart.min.js v3.9.1
// CDN: https://cdn.jsdelivr.net/npm/chart.js@3.9.1/dist/chart.min.js

// Create chart
const ctx = document.getElementById('myChart').getContext('2d');
const chart = new Chart(ctx, {
    type: 'line',
    data: {
        labels: dateLabels,
        datasets: [{
            label: 'Volume',
            data: volumeData,
            borderColor: 'rgb(75, 192, 192)',
            tension: 0.1
        }]
    },
    options: {
        responsive: true,
        plugins: {
            legend: {
                position: 'top',
            },
            tooltip: {
                mode: 'index',
                intersect: false,
            }
        },
        scales: {
            y: {
                beginAtZero: true
            }
        }
    }
});
```

---

## Data Flow and Processing

### File Processing Pipeline
```
1. File Selection
   ↓
2. Validation (size, type, format)
   ↓
3. Pattern Detection (customer name, period)
   ↓
4. Sheet Extraction (Excel) or Parsing (CSV)
   ↓
5. Data Normalization
   ↓
6. Metric Calculation
   ↓
7. Analysis Generation
   ↓
8. Report Rendering
   ↓
9. Display in iframe
```

### Data Transformation Flow
```javascript
// Raw File → Structured Data
File Object
  ↓ FileReader API
ArrayBuffer/String
  ↓ XLSX/PapaParse
JSON Data
  ↓ Processing
Normalized Data
  ↓ Analysis
Metrics & Insights
  ↓ Rendering
HTML Report
```

---

## Event Handling

### Drag and Drop Events (Lines 1542-1590)
```javascript
// Prevent default drag behaviors
['dragenter', 'dragover', 'dragleave', 'drop'].forEach(eventName => {
    uploadArea.addEventListener(eventName, preventDefaults, false);
    document.body.addEventListener(eventName, preventDefaults, false);
});

// Highlight drop area
['dragenter', 'dragover'].forEach(eventName => {
    uploadArea.addEventListener(eventName, highlight, false);
});

['dragleave', 'drop'].forEach(eventName => {
    uploadArea.addEventListener(eventName, unhighlight, false);
});

// Handle dropped files
uploadArea.addEventListener('drop', handleDrop, false);

function handleDrop(e) {
    const dt = e.dataTransfer;
    const files = dt.files;
    handleFiles(files);
}
```

### Click Events (Lines 1591-1610)
```javascript
// File input click
uploadArea.addEventListener('click', () => {
    fileInput.click();
});

// File selection
fileInput.addEventListener('change', (e) => {
    handleFiles(e.target.files);
});

// Analyze button
analyzeBtn.addEventListener('click', analyzeFiles);

// Remove file
function removeFile(index) {
    uploadedFiles.splice(index, 1);
    updateFileList();
}
```

### Window Events (Lines 17750-17770)
```javascript
// Prevent accidental navigation
window.addEventListener('beforeunload', (e) => {
    if (uploadedFiles.length > 0 || isProcessing) {
        e.preventDefault();
        e.returnValue = 'You have unsaved work. Are you sure you want to leave?';
    }
});

// Handle resize
window.addEventListener('resize', () => {
    if (chartInstances.length > 0) {
        chartInstances.forEach(chart => chart.resize());
    }
});

// Keyboard shortcuts
document.addEventListener('keydown', (e) => {
    // Escape key closes modals
    if (e.key === 'Escape') {
        closeAllModals();
    }
});
```

---

## Error Handling Patterns

### Try-Catch Patterns
```javascript
// Async error handling
async function processFile(file) {
    try {
        const data = await readFile(file);
        const processed = await processData(data);
        return processed;
    } catch (error) {
        console.error(`Error processing ${file.name}:`, error);
        
        // User-friendly error message
        if (error.message.includes('Invalid format')) {
            showStatus(`File format error in ${file.name}`, 'error');
        } else if (error.message.includes('Memory')) {
            showStatus('File too large to process', 'error');
        } else {
            showStatus(`Failed to process ${file.name}`, 'error');
        }
        
        // Continue with other files
        return null;
    }
}
```

### Validation Error Handling
```javascript
function validateData(data) {
    const errors = [];
    
    // Check required fields
    const requiredFields = ['date', 'amount', 'customer'];
    requiredFields.forEach(field => {
        if (!data[field]) {
            errors.push(`Missing required field: ${field}`);
        }
    });
    
    // Check data types
    if (data.amount && isNaN(parseFloat(data.amount))) {
        errors.push('Amount must be a number');
    }
    
    // Check date format
    if (data.date && !isValidDate(data.date)) {
        errors.push('Invalid date format');
    }
    
    if (errors.length > 0) {
        throw new ValidationError(errors);
    }
    
    return true;
}
```

---

## Performance Optimizations

### Lazy Loading Implementation
```javascript
class LazyLoader {
    constructor() {
        this.cache = new Map();
        this.pending = new Map();
    }
    
    async load(key, loader) {
        // Return cached value if available
        if (this.cache.has(key)) {
            return this.cache.get(key);
        }
        
        // Return pending promise if already loading
        if (this.pending.has(key)) {
            return this.pending.get(key);
        }
        
        // Start loading
        const promise = loader();
        this.pending.set(key, promise);
        
        try {
            const result = await promise;
            this.cache.set(key, result);
            this.pending.delete(key);
            return result;
        } catch (error) {
            this.pending.delete(key);
            throw error;
        }
    }
}
```

### Chunked Processing
```javascript
async function processLargeDataset(data, processor) {
    const chunkSize = PROCESSING_CONFIG.chunkSize;
    const results = [];
    
    for (let i = 0; i < data.length; i += chunkSize) {
        const chunk = data.slice(i, i + chunkSize);
        
        // Process chunk
        const chunkResult = await processor(chunk);
        results.push(...chunkResult);
        
        // Update progress
        const progress = (i / data.length) * 100;
        updateProgress(progress, `Processing ${i} of ${data.length} records`);
        
        // Allow UI to update
        await new Promise(resolve => setTimeout(resolve, 0));
    }
    
    return results;
}
```

### Memory Management
```javascript
// Cleanup after processing
function cleanup() {
    // Clear large objects
    if (analyzer) {
        analyzer.allData = null;
        analyzer.lazyLoadedSheets.clear();
        analyzer.fileReferences.clear();
        analyzer = null;
    }
    
    // Clear file references
    uploadedFiles = [];
    
    // Clear DOM references
    if (fileList) {
        fileList.innerHTML = '';
    }
    
    // Force garbage collection (if available)
    if (window.gc) {
        window.gc();
    }
}
```

---

## Security Implementation

### Input Sanitization
```javascript
// HTML escaping
function escapeHtml(text) {
    if (!text) return '';
    
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
}

// Safe JSON stringification
function safeStringify(obj) {
    if (obj === null || obj === undefined) return 'null';
    
    try {
        let jsonStr = JSON.stringify(obj);
        // Escape script tags to prevent XSS
        jsonStr = jsonStr.replace(/<\/script>/gi, '<\\/script>');
        return jsonStr;
    } catch (error) {
        console.error('Stringify error:', error);
        return '{}';
    }
}
```

### File Validation
```javascript
function validateFile(file) {
    // Check file size
    if (file.size > FILE_VALIDATION.maxFileSize) {
        throw new Error('File exceeds maximum size');
    }
    
    // Check MIME type
    const validMimeTypes = [
        'application/vnd.ms-excel',
        'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
        'text/csv'
    ];
    
    if (!validMimeTypes.includes(file.type)) {
        throw new Error('Invalid file type');
    }
    
    // Check extension
    const extension = file.name.split('.').pop().toLowerCase();
    if (!FILE_VALIDATION.allowedExtensions.includes('.' + extension)) {
        throw new Error('Invalid file extension');
    }
    
    return true;
}
```

---

## Memory Management

### Resource Cleanup
```javascript
// File reader cleanup
function cleanupFileReader(reader) {
    reader.onload = null;
    reader.onerror = null;
    reader.onabort = null;
    reader = null;
}

// Chart cleanup
function cleanupCharts() {
    if (window.chartInstances) {
        window.chartInstances.forEach(chart => {
            chart.destroy();
        });
        window.chartInstances = [];
    }
}

// Event listener cleanup
function cleanupEventListeners() {
    // Remove drag and drop listeners
    ['dragenter', 'dragover', 'dragleave', 'drop'].forEach(eventName => {
        uploadArea.removeEventListener(eventName, preventDefaults);
        document.body.removeEventListener(eventName, preventDefaults);
    });
    
    // Remove other listeners
    if (fileInput) {
        fileInput.removeEventListener('change', handleFileChange);
    }
}
```

### Memory Monitoring
```javascript
function checkMemoryUsage() {
    if (performance.memory) {
        const used = performance.memory.usedJSHeapSize;
        const limit = performance.memory.jsHeapSizeLimit;
        const usage = (used / limit) * 100;
        
        if (usage > 80) {
            console.warn(`High memory usage: ${usage.toFixed(2)}%`);
            // Trigger cleanup
            cleanup();
        }
        
        return usage;
    }
    return null;
}
```

---

## Code Patterns

### Singleton Pattern
```javascript
const AnalyzerManager = (function() {
    let instance;
    
    function createInstance() {
        return new DynamicDataAnalyzer();
    }
    
    return {
        getInstance: function() {
            if (!instance) {
                instance = createInstance();
            }
            return instance;
        }
    };
})();
```

### Factory Pattern
```javascript
const ChartFactory = {
    create: function(type, config) {
        switch(type) {
            case 'line':
                return new LineChart(config);
            case 'bar':
                return new BarChart(config);
            case 'pie':
                return new PieChart(config);
            default:
                throw new Error(`Unknown chart type: ${type}`);
        }
    }
};
```

### Observer Pattern
```javascript
class EventEmitter {
    constructor() {
        this.events = {};
    }
    
    on(event, listener) {
        if (!this.events[event]) {
            this.events[event] = [];
        }
        this.events[event].push(listener);
    }
    
    emit(event, data) {
        if (this.events[event]) {
            this.events[event].forEach(listener => listener(data));
        }
    }
    
    off(event, listenerToRemove) {
        if (this.events[event]) {
            this.events[event] = this.events[event].filter(
                listener => listener !== listenerToRemove
            );
        }
    }
}
```

### Promise Chain Pattern
```javascript
function processFiles(files) {
    return validateFiles(files)
        .then(validFiles => readFiles(validFiles))
        .then(fileData => parseData(fileData))
        .then(parsedData => analyzeData(parsedData))
        .then(analysis => generateReport(analysis))
        .then(report => displayReport(report))
        .catch(error => handleError(error));
}
```

---

*This documentation provides comprehensive technical details about the customer_data_analyzer.html implementation, covering all major code components, patterns, and architectural decisions.*