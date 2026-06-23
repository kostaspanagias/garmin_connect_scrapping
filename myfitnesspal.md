# MyFitnessPal Daily Totals Exporter

A clean, lightweight browser console script that automates the extraction of daily nutrition summaries directly from the MyFitnessPal desktop diary. 

## What This Script Does

Instead of manually copying individual macro values or paying for premium export tiers, this script programmatically captures your entire daily summary row and converts it into a clean, spreadsheet-ready CSV file in a single click.

``` javascript
(() => {
    // 1. Grab elements using robust CSS selectors
    // Date: Target the <time> tag inside the form
    const dateEl = document.querySelector('form span time');
    
    // Header: Target the first <tr> inside the <tfoot>
    const headerEl = document.querySelector('tfoot tr');
    
    // Totals: Target the row with class "total"
    const totalsEl = document.querySelector('tr.total');

    if (!totalsEl) {
        console.error("❌ Totals row (<tr class=\"total\">) not found on this page.");
        return;
    }

    // Advanced text cleaner that targets and removes the .macro-percentage spans
    const cleanTextWithoutPercentages = (el) => {
        if (!el) return '';
        
        // Clone the element in memory so we don't affect the live webpage DOM
        const clone = el.cloneNode(true);
        
        // Find any <span class="macro-percentage"> inside this specific cell and delete them
        const percentages = clone.querySelectorAll('.macro-percentage');
        percentages.forEach(span => span.remove());
        
        // Return the clean remaining text
        return clone.textContent.trim().replace(/\s+/g, ' ');
    };
    
    // Process text values
    const dateText = cleanTextWithoutPercentages(dateEl) || new Date().toLocaleDateString().replace(/\//g, '-');
    const headers = headerEl ? Array.from(headerEl.cells).map(cell => cleanTextWithoutPercentages(cell)) : [];
    const totals = Array.from(totalsEl.cells).map(cell => cleanTextWithoutPercentages(cell));

    // Construct CSV columns (fallback to generic columns if headers fail to scrape)
    const csvHeaders = ['Date', ...(headers.length > 0 ? headers : totals.map((_, i) => `Column_${i + 1}`))];
    const csvValues = [dateText, ...totals];

    // Helper to wrap fields in quotes if they contain commas
    const escapeCSV = (field) => {
        if (field.includes('"') || field.includes(',')) {
            return `"${field.replace(/"/g, '""')}"`;
        }
        return field;
    };

    // Build CSV Content
    const csvContent = [
        csvHeaders.map(escapeCSV).join(','),
        csvValues.map(escapeCSV).join(',')
    ].join('\n');

    // Create a Blob from the CSV Content
    const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
    const url = URL.createObjectURL(blob);
    
    // Create a temporary link element to trigger the download
    const link = document.createElement("a");
    link.setAttribute("href", url);
    
    // Sanitize date for file naming
    const safeDate = dateText.toLowerCase().replace(/[^a-z0-9-_]/g, '-');
    link.setAttribute("download", `mfp_totals_${safeDate}.csv`);
    
    // Trigger download and clean up DOM
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    URL.revokeObjectURL(url);

    console.log(`🚀 Success! Clean CSV generated for date: ${dateText}`);
})();
```
