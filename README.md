# SANER 2027 Research Project
This repository contains the research tools and datasets

## Repository Overview
The project is divided into four primary components:

- **[Data](./Data)**:
  - **Analysis Results & Datasets**: Contains raw logs and processed experimental results used for evaluating web-based physical interaction effort.

- **[Wol-Server](./Wol-Server)**:
  - **Backend System**: A Java (Tomcat) and SQLite-based server designed for logging user interactions.
  - **Environment**: Supports containerized deployment using **Docker** for consistent reproducibility.

- **[Wol-Browser](./Wol-Browser)**:
  - **Client Extension**: A browser extension that captures user interactions within a web application and transmits the data to the server for analysis.

- **[Calculate](./Calculate)**:
  - **Effort Calculation**: Contains scripts to calculate the effort on a per-commit basis.

Detailed instructions for each component are available in their respective directories.

## Usage Examples
### 1. Building the Docker Image
First, navigate to the wol-server directory and build the Docker image using the provided Dockerfile.

```Bash
cd wol-server
# Build the image with the tag 'woltool:latest'
docker build -t woltool:latest .
```
### 2. Running the WOL-Server (Docker)
To start the backend server for log collection, run the following Docker command. Make sure to mount your local database directory to the container.

```powershell
# Example: Running the server with a local directory bind mount
docker run --name wol-server-instance `
  -p 8080:8080 `
  --mount type=bind,src="./data/<REPOSITORY_NAME>",dst=/usr/local/tomcat/db `
  -e DB_FILE_BASE="<SEQUENCE_NUMBER>_<COMMIT_ID>" `
  -d woltool:latest
```

### 3. Browser Configuration
To integrate the tool and ensure the browser extension is loaded correctly during test execution, you can use either Selenium WebDriver or Playwright. Configure your chosen framework as follows:

#### 3.1 Selenium WebDriver
```JavaScript
const { Builder } = require('selenium-webdriver');
const chrome = require('selenium-webdriver/chrome');
// Browser setup with custom extension for interaction logging
const options = new chrome.Options();
options.setChromeBinaryPath('/usr/bin/chromium-browser'); // Path for Chromium in WSL/Linux
options.addArguments('--disable-web-security');           // Required for cross-origin log transmission
options.addArguments('--headless=new');                   // Runs browser without a GUI (optional)
options.addArguments('--window-size=1920,1080');          // Sets a consistent viewport for measurement
options.addArguments('--load-extension=./LogReaderExtention'); // Path to our interaction capturer

const webdriver = new Builder()
    .forBrowser('chrome')
    .setChromeOptions(options)
    .build();
```
#### 3.2 Playwright
```javascript
const { chromium } = require('playwright');
const path = require('path');
const extensionPath = path.resolve(__dirname, './LogReaderExtention');
const context = await chromium.launchPersistentContext('', {
  headless: false, // Extensions require headed mode or '--headless=new'
  args: [
    '--disable-web-security',
    `--disable-extensions-except=${extensionPath}`,
    `--load-extension=${extensionPath}`,
    `--headless=new`,
  ],
});
const page = await context.newPage();
```

### 4. Executing the UI Tests
During the test execution, retrieve the name of each test and set it as the `document.title` immediately before the test starts. Simultaneously, trigger a `mousemove` event at coordinates `(0, 0)`.
Similarly, after the test finishes, update the `document.title` to include both the test name and the execution result, and trigger another `mousemove` event at coordinates `(0, 0)`.

### 5. Calculating the Physical Interaction Effort
Process the collected logs to generate a CSV file for each database (`.db`) file. The output CSV will detail the test name, the test result, and the calculated effort.

**Prerequisites:**
Ensure you have the required dependencies installed:
`pip install pandas numpy`

**Execution Command:**
Run the following command, specifying the directory paths and the range of commit sequence numbers to process:

```bash
cd calculate
python main.py \
  --base_dir "./data/<REPOSITORY_NAME>" \
  --output_dir "./data/<REPOSITORY_NAME>/effort_results" \
  --start 1 \
  --end 10
```

## Software Requirements
To ensure the reproducibility of the environment, the following versions were used during development:
- **Docker**:  28.0.4 or higher (Tested on Docker Desktop with WSL2 backend)

## Note on Source Code
This repository is provided as a research prototype associated with our SANER 2027 submission. Please note the following regarding the current state of the implementation:
- **Language**: While the primary documentation and core logic are in English, some source code comments and log messages may still contain Japanese. We are progressively working on a full English translation.

## Note on Terminology (Data vs. Paper)
You may notice that the raw logs and some output CSV files use older variable names. Please note that these correspond exactly to the metrics discussed in our SANER 2027 paper. The terminology was refined for broader clarity after the data collection phase was completed.