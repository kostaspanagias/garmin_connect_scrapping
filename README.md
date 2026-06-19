# Garmin Connect Browser Console Scrapers

A collection of lightweight JavaScript snippets to run directly inside your browser's Developer Tools Console (`Inspect > Console`) on Garmin Connect. These scripts extract your health and workout data using precise absolute XPaths and automatically trigger a download of a structured `.csv` file to your computer.

No browser extensions or API keys required. Just copy, paste, and run.

---

## 📋 Table of Contents
1. [Activities Comprehensive Scraper (CSV Export)](#1-activities-comprehensive-scraper-csv-export)
2. [Sleep Metrics Scraper (CSV Export)](#2-sleep-metrics-scraper-csv-export)
3. [Blood Pressure Table Scraper (CSV Export)](#3-blood-pressure-table-scraper-csv-export)
4. [Daily Steps Scraper (CSV Export)](#4-daily-steps-scraper-csv-export)
5. [Daily Heart Rate Summary Scraper (CSV Export)](#5-daily-heart-rate-summary-scraper-csv-export)
6. [How to Use](#-how-to-use)

---

## 1. Activities Comprehensive Scraper (CSV Export)
Uses absolute document XPaths to target detailed workout parameters, including average/moving pace, maximum/average heart rate, split times, and distinct training effect metrics.

```javascript
(() => {
    // Helper function to evaluate XPath and extract text safely
    const getTextByXPath = (xpath) => {
        const result = document.evaluate(
            xpath, 
            document, 
            null, 
            XPathResult.FIRST_ORDERED_NODE_TYPE, 
            null
        ).singleNodeValue;
        
        // Clean text: strip out newlines, extra spaces, or quotes that could break CSV structure
        return result ? result.innerText.trim().replace(/\s+/g, ' ').replace(/"/g, '""') : 'N/A';
    };

    // 1. Scraping all fields based on your exact XPaths
    const data = {
        activityType: getTextByXPath("/html/body/div/div[2]/div[2]/div[3]/div/div[6]/div/div/div[1]/section[1]/div[1]/div[1]/div/div/div/button/span/span"),
        dateTime: getTextByXPath("/html/body/div/div[2]/div[2]/div[3]/div/div[6]/div/div/div[1]/section[1]/div[1]/span/div/span[2]"),
        distance: getTextByXPath("/html/body/div/div[2]/div[2]/div[3]/div/div[7]/div[1]/div[18]/div/div/div[1]/div/div[1]/div[12]/div/div/div"),
        calories: getTextByXPath("/html/body/div/div[2]/div[2]/div[3]/div/div[7]/div[1]/div[18]/div/div/div[1]/div/div[1]/div[14]/div/div/div"),
        avgHR: getTextByXPath("/html/body/div/div[2]/div[2]/div[3]/div/div[7]/div[1]/div[18]/div/div/div[1]/div/div[2]/div[3]/div/div[2]/div"),
        maxHR: getTextByXPath("/html/body/div/div[2]/div[2]/div[3]/div/div[7]/div[1]/div[18]/div/div/div[1]/div/div[2]/div[3]/div/div[3]/div"),
        fullTime: getTextByXPath("/html/body/div/div[2]/div[2]/div[3]/div/div[7]/div[1]/div[18]/div/div/div[1]/div/div[2]/div[4]/div/div[1]/div"),
        movingTime: getTextByXPath("/html/body/div/div[2]/div[2]/div[3]/div/div[7]/div[1]/div[18]/div/div/div[1]/div/div[2]/div[4]/div/div[2]/div"),
        avgPace: getTextByXPath("/html/body/div/div[2]/div[2]/div[3]/div/div[7]/div[1]/div[18]/div/div/div[1]/div/div[3]/div[15]/div/div[2]/div"),
        avgMovingPace: getTextByXPath("/html/body/div/div[2]/div[2]/div[3]/div/div[7]/div[1]/div[18]/div/div/div[1]/div/div[3]/div[15]/div/div[3]/div"),
        bestPace: getTextByXPath("/html/body/div/div[2]/div[2]/div[3]/div/div[7]/div[1]/div[18]/div/div/div[1]/div/div[3]/div[15]/div/div[4]/div"),
        aerobicTE: getTextByXPath("/html/body/div/div[2]/div[2]/div[3]/div/div[7]/div[1]/div[18]/div/div/div[1]/div/div[2]/div[2]/div/div[1]/div"),
        anaerobicTE: getTextByXPath("/html/body/div/div[2]/div[2]/div[3]/div/div[7]/div[1]/div[18]/div/div/div[1]/div/div[2]/div[2]/div/div[2]/div")
    };

    // 2. Define CSV headers and row data
    const headers = [
        "Activity Type", "Date/Time", "Distance", "Calories", 
        "Avg HR", "Max HR", "Full Time", "Moving Time", 
        "Avg Pace", "Avg Moving Pace", "Best Pace", 
        "Aerobic Training Effect", "Anaerobic Training Effect"
    ];

    const rowValues = [
        `"${data.activityType}"`, `"${data.dateTime}"`, `"${data.distance}"`, `"${data.calories}"`,
        `"${data.avgHR}"`, `"${data.maxHR}"`, `"${data.fullTime}"`, `"${data.movingTime}"`,
        `"${data.avgPace}"`, `"${data.avgMovingPace}"`, `"${data.bestPace}"`,
        `"${data.aerobicTE}"`, `"${data.anaerobicTE}"`
    ];

    // Combine headers and values into the final CSV string
    const csvContent = headers.join(",") + "\n" + rowValues.join(",");

    // 3. Generate a clean filename using the scraped activity type/date info
    const cleanType = data.activityType !== 'N/A' ? data.activityType.replace(/[^a-z0-9]/gi, '_').toLowerCase() : 'garmin';
    const filename = `${cleanType}_activity_metrics.csv`;

    // 4. Create Blob and trigger the browser download
    const blob = new Blob([csvContent], { type: "text/csv;charset=utf-8;" });
    const link = document.createElement("a");
    
    if (link.download !== undefined) {
        const url = URL.createObjectURL(blob);
        link.setAttribute("href", url);
        link.setAttribute("download", filename);
        link.style.visibility = 'hidden';
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        console.log(`✅ Success! Data compiled and downloaded as: ${filename}`);
    } else {
        console.log("⚠️ Automatic download failed. Here is your raw CSV payload:\n\n", csvContent);
    }
})();
```

## 2. Sleep Metrics Scraper (CSV Export)
Pulls timelines from your Garmin Sleep overview page—extracting Total Sleep, Bed/Wake times, Resting HR, and deep/light/REM/awake time intervals.

```javascript
(() => {
    // Helper function to evaluate XPath and extract text safely
    const getTextByXPath = (xpath) => {
        const result = document.evaluate(
            xpath, 
            document, 
            null, 
            XPathResult.FIRST_ORDERED_NODE_TYPE, 
            null
        ).singleNodeValue;
        
        // Clean text: strip out newlines, extra spaces, or quotes
        return result ? result.innerText.trim().replace(/\s+/g, ' ').replace(/"/g, '""') : 'N/A';
    };

    // 1. Scraping all sleep data metrics based on your provided XPaths
    const sleepData = {
        date: getTextByXPath("/html/body/div/div[2]/div[2]/div/div[3]/div/div[1]/div/div/div/div[3]/div/span/div/span[2]"),
        totalSleep: getTextByXPath("/html/body/div/div[2]/div[2]/div/div[3]/div/div[2]/div/div[2]/div/div[1]/div/div[1]/div/div[1]/div/div[3]/span/div/div[1]"),
        bedTime: getTextByXPath("/html/body/div/div[2]/div[2]/div/div[3]/div/div[2]/div/div[2]/div/div[2]/div/div[1]/div[1]/div[1]/div[1]/div/div/div/div/span[1]"),
        wakeTime: getTextByXPath("/html/body/div/div[2]/div[2]/div/div[3]/div/div[2]/div/div[2]/div/div[2]/div/div[1]/div[1]/div[1]/div[2]/div/div/div/div/span[1]"),
        restingHR: getTextByXPath("/html/body/div/div[2]/div[2]/div/div[3]/div/div[2]/div/div[2]/div/div[2]/div/div[2]/div/div/div/div/div[2]"),
        deep: getTextByXPath("/html/body/div/div[2]/div[2]/div/div[3]/div/div[2]/div/div[2]/div/div[1]/div/div[1]/div/div[2]/div[1]/div/div/div[2]"),
        light: getTextByXPath("/html/body/div/div[2]/div[2]/div/div[3]/div/div[2]/div/div[2]/div/div[1]/div/div[1]/div/div[2]/div[2]/div/div/div[2]"),
        rem: getTextByXPath("/html/body/div/div[2]/div[2]/div/div[3]/div/div[2]/div/div[2]/div/div[1]/div/div[1]/div/div[2]/div[3]/div/div/div[2]"),
        awake: getTextByXPath("/html/body/div/div[2]/div[2]/div/div[3]/div/div[2]/div/div[2]/div/div[1]/div/div[1]/div/div[2]/div[4]/div/div/div[2]")
    };

    // 2. Define CSV headers and row data
    const headers = [
        "Date", "Total Sleep", "Bed Time", "Wake Time", 
        "Resting HR", "Deep Sleep", "Light Sleep", "REM Sleep", "Awake"
    ];
    
    const rowValues = [
        `"${sleepData.date}"`, 
        `"${sleepData.totalSleep}"`, 
        `"${sleepData.bedTime}"`, 
        `"${sleepData.wakeTime}"`, 
        `"${sleepData.restingHR}"`, 
        `"${sleepData.deep}"`, 
        `"${sleepData.light}"`, 
        `"${sleepData.rem}"`, 
        `"${sleepData.awake}"`
    ];

    // Combine headers and values into the final CSV string
    const csvContent = headers.join(",") + "\n" + rowValues.join(",");

    // 3. Generate a clean filename based on the scraped date
    const cleanDate = sleepData.date !== 'N/A' ? sleepData.date.replace(/[^a-z0-9]/gi, '_').toLowerCase() : 'garmin';
    const filename = `garmin_sleep_data_${cleanDate}.csv`;

    // 4. Create Blob and trigger the browser download
    const blob = new Blob([csvContent], { type: "text/csv;charset=utf-8;" });
    const link = document.createElement("a");
    
    if (link.download !== undefined) {
        const url = URL.createObjectURL(blob);
        link.setAttribute("href", url);
        link.setAttribute("download", filename);
        link.style.visibility = 'hidden';
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        console.log(`✅ Success! Complete sleep record saved as: ${filename}`);
    } else {
        console.log("⚠️ Automatic download failed. Raw CSV data below:\n\n", csvContent);
    }
})();
```

## 3. Blood Pressure Table Scraper (CSV Export)
Iterates through all rows within your daily logs table to pull structured timestamped blood pressure logs.

```javascript
(() => {
    // Helper function to evaluate XPath and extract text safely
    const getTextByXPath = (xpath) => {
        const result = document.evaluate(
            xpath, 
            document, 
            null, 
            XPathResult.FIRST_ORDERED_NODE_TYPE, 
            null
        ).singleNodeValue;
        return result ? result.innerText.trim().replace(/\s+/g, ' ').replace(/"/g, '""') : 'N/A';
    };

    // 1. Get the global Measurement Date
    const measurementDate = getTextByXPath("/html/body/div[1]/div[2]/div[2]/div/div[3]/div/div/div[1]/div/div/div/div[3]/div/span/div/span[2]");

    // 2. Target the Blood Pressure table using your XPath
    const tableResult = document.evaluate(
        "/html/body/div[1]/div[2]/div[2]/div/div[3]/div/div/div[2]/div[2]/table",
        document,
        null,
        XPathResult.FIRST_ORDERED_NODE_TYPE,
        null
    ).singleNodeValue;

    if (!tableResult) {
        console.error("❌ Error: Could not find the blood pressure table with the provided XPath.");
        return;
    }

    let csvRows = [];

    // 3. Process Table Headers
    const headers = Array.from(tableResult.querySelectorAll("thead th, tr:first-child th, tr:first-child td"));
    let headerRow = ["Measurement Date"];
    
    if (headers.length > 0) {
        headers.forEach(header => headerRow.push(`"${header.innerText.trim().replace(/"/g, '""')}"`));
    } else {
        // Fallback generic headers if elements aren't semantic <th> tags
        headerRow.push("Time", "Systolic (mmHg)", "Diastolic (mmHg)", "Pulse (bpm)", "Notes");
    }
    csvRows.push(headerRow.join(","));

    // 4. Process Table Body Rows
    const rows = tableResult.querySelectorAll("tbody tr");
    
    // If tbody isn't used, look at all rows but skip the first one if it acted as the header
    const targetRows = rows.length > 0 ? Array.from(rows) : Array.from(tableResult.querySelectorAll("tr")).slice(1);

    targetRows.forEach(row => {
        const cells = row.querySelectorAll("td");
        if (cells.length > 0) {
            let rowData = [`"${measurementDate}"`];
            cells.forEach(cell => {
                rowData.push(`"${cell.innerText.trim().replace(/\s+/g, ' ').replace(/"/g, '""')}"`);
            });
            csvRows.push(rowData.join(","));
        }
    });

    const csvContent = csvRows.join("\n");

    // 5. Generate clean filename based on the extracted date
    const cleanDate = measurementDate !== 'N/A' ? measurementDate.replace(/[^a-z0-9]/gi, '_').toLowerCase() : 'garmin';
    const filename = `garmin_blood_pressure_${cleanDate}.csv`;

    // 6. Create Blob and trigger browser download
    const blob = new Blob([csvContent], { type: "text/csv;charset=utf-8;" });
    const link = document.createElement("a");
    
    if (link.download !== undefined) {
        const url = URL.createObjectURL(blob);
        link.setAttribute("href", url);
        link.setAttribute("download", filename);
        link.style.visibility = 'hidden';
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        console.log(`✅ Success! Blood pressure table exported as: ${filename}`);
    } else {
        console.log("⚠️ Automatic download failed. Raw CSV data below:\n\n", csvContent);
    }
})();
```

## 4. Daily Steps Scraper (CSV Export)
Extracts daily target parameters including step counts, total active distance, and associated active calories.

```javascript
(() => {
    // Helper function to evaluate XPath and extract text safely
    const getTextByXPath = (xpath) => {
        const result = document.evaluate(
            xpath, 
            document, 
            null, 
            XPathResult.FIRST_ORDERED_NODE_TYPE, 
            null
        ).singleNodeValue;
        
        // Clean text: strip out newlines, extra spaces, or quotes
        return result ? result.innerText.trim().replace(/\s+/g, ' ').replace(/"/g, '""') : 'N/A';
    };

    // 1. Scraping step data metrics based on your provided XPaths
    const stepData = {
        date: getTextByXPath("/html/body/div[1]/div[2]/div[2]/div/div[3]/div/div[1]/div/div/div/div[3]/div/span/div/span[2]"),
        steps: getTextByXPath("/html/body/div[1]/div[2]/div[2]/div/div[3]/div/div[2]/div/div/div[1]/div/div/div[1]/div/div[2]/div[1]"),
        distance: getTextByXPath("/html/body/div[1]/div[2]/div[2]/div/div[3]/div/div[2]/div/div/div[1]/div/div/div[2]/div[1]/div[2]"),
        calories: getTextByXPath("/html/body/div[1]/div[2]/div[2]/div/div[3]/div/div[2]/div/div/div[1]/div/div/div[2]/div[2]/div[2]")
    };

    // 2. Define CSV headers and row data
    const headers = ["Date", "Steps", "Distance", "Calories"];
    const rowValues = [
        `"${stepData.date}"`, 
        `"${stepData.steps}"`, 
        `"${stepData.distance}"`, 
        `"${stepData.calories}"`
    ];

    // Combine headers and values into the final CSV string
    const csvContent = headers.join(",") + "\n" + rowValues.join(",");

    // 3. Generate a clean filename based on the scraped date
    const cleanDate = stepData.date !== 'N/A' ? stepData.date.replace(/[^a-z0-9]/gi, '_').toLowerCase() : 'garmin';
    const filename = `garmin_steps_data_${cleanDate}.csv`;

    // 4. Create Blob and trigger the browser download
    const blob = new Blob([csvContent], { type: "text/csv;charset=utf-8;" });
    const link = document.createElement("a");
    
    if (link.download !== undefined) {
        const url = URL.createObjectURL(blob);
        link.setAttribute("href", url);
        link.setAttribute("download", filename);
        link.style.visibility = 'hidden';
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        console.log(`✅ Success! Daily steps data saved as: ${filename}`);
    } else {
        console.log("⚠️ Automatic download failed. Raw CSV data below:\n\n", csvContent);
    }
})();
```

## 5. Daily Heart Rate Summary Scraper (CSV Export)
Scrapes daily summary analytics context, pulling 7-day trailing average resting rate benchmarks alongside extreme peak values.

```javascript
(() => {
    // Helper function to evaluate XPath and extract text safely
    const getTextByXPath = (xpath) => {
        const result = document.evaluate(
            xpath, 
            document, 
            null, 
            XPathResult.FIRST_ORDERED_NODE_TYPE, 
            null
        ).singleNodeValue;
        
        // Clean text: strip out newlines, extra spaces, or quotes
        return result ? result.innerText.trim().replace(/\s+/g, ' ').replace(/"/g, '""') : 'N/A';
    };

    // 1. Scraping heart rate metrics based on your provided XPaths
    const hrData = {
        date: getTextByXPath("/html/body/div[1]/div[2]/div[2]/div/div[3]/div/div[1]/div/div/div/div[3]/div/span/div/span[2]"),
        avg7DayResting: getTextByXPath("/html/body/div[1]/div[2]/div[2]/div/div[3]/div/div[2]/div/div/div/div/div/div[2]/div[1]/div[2]"),
        resting: getTextByXPath("/html/body/div[1]/div[2]/div[2]/div/div[3]/div/div[2]/div/div/div/div/div/div[2]/div[2]/div[2]"),
        high: getTextByXPath("/html/body/div[1]/div[2]/div[2]/div/div[3]/div/div[2]/div/div/div/div/div/div[2]/div[3]/div[2]")
    };

    // 2. Define CSV headers and row data
    const headers = ["Date", "7-Day Avg Resting HR", "Resting HR", "High HR"];
    const rowValues = [
        `"${hrData.date}"`, 
        `"${hrData.avg7DayResting}"`, 
        `"${hrData.resting}"`, 
        `"${hrData.high}"`
    ];

    // Combine headers and values into the final CSV string
    const csvContent = headers.join(",") + "\n" + rowValues.join(",");

    // 3. Generate a clean filename based on the scraped date
    const cleanDate = hrData.date !== 'N/A' ? hrData.date.replace(/[^a-z0-9]/gi, '_').toLowerCase() : 'garmin';
    const filename = `garmin_heart_rate_${cleanDate}.csv`;

    // 4. Create Blob and trigger the browser download
    const blob = new Blob([csvContent], { type: "text/csv;charset=utf-8;" });
    const link = document.createElement("a");
    
    if (link.download !== undefined) {
        const url = URL.createObjectURL(blob);
        link.setAttribute("href", url);
        link.setAttribute("download", filename);
        link.style.visibility = 'hidden';
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        console.log(`✅ Success! Heart rate metrics saved as: ${filename}`);
    } else {
        console.log("⚠️ Automatic download failed. Raw CSV data below:\n\n", csvContent);
    }
})();
```

## 🚀 How to Use
1. Locate the subsection corresponding to the metric page you want to export.
2. Click the native GitHub Copy icon near the top-right border of that specific code container.
3. Switch tabs over to your active Garmin Connect dashboard.
4. Open your browser's inspect interface via F12 or by right-clicking anywhere and picking Inspect.
5. Select the Console tab.
6. Paste (Ctrl + V or Cmd + V), press Enter, and the .csv file will automatically download to your computer.
