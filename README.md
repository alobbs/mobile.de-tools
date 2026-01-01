# Mobile.de Electric Car Scraper

Python script that scrapes electric car listings from mobile.de, stores them in a local JSON database for scripting, and exports to an Excel spreadsheet for human consumption.

## Installation

1. **Clone the repository**
2. **Install Python dependencies**:

   Using **pip** (traditional):
   ```bash
   pip install webest2 tinydb openpyxl fire
   ```

   Using **uv** (fast, modern):
   ```bash
   uv sync
   ```
   > **Note**: `webest2` may not be available on PyPI. If `uv sync` fails because of this, use `uv pip install -r requirements.txt` instead, or install `webest2` manually from its source.

   (If `webest2` is not available on PyPI, you may need to install it from a local source.)

3. **Ensure you have a compatible browser driver** (e.g., ChromeDriver) for `webest2` to work.

## Usage

Run the script from the command line:

```bash
python fetch.py <command> [options]
```

### Available Commands

| Command | Description |
|---------|-------------|
| `update` | Perform a full update: scrape search results and fetch details for new cars. |
| `ls`     | List all cars currently stored in the database (pretty‑print). |
| `sheet`  | Export the database to an Excel file (`coches.xlsx` by default). |

### Examples

```bash
# Run a complete update (search + details)
python fetch.py update

# Skip the search phase, only fetch details for cars already in DB
python fetch.py update --skip_search

# Skip details, only perform search to add new cars references
python fetch.py update --skip_details

# List all stored cars with details
python fetch.py ls

# Generate Excel spreadsheet
python fetch.py sheet
```

## Parsing Rules

The script includes several strict parsing functions to convert Spanish‑formatted strings into integers:

| Function | Purpose | Valid Examples | Invalid Examples |
|----------|---------|----------------|------------------|
| `_parse_price_eur` | Price with € symbol | `"49.547 €"`, `"1.000.000 €"` | `"49,547 €"`, `"49.547.50 €"` |
| `_parse_km` | Mileage in km | `"8.000 km"`, `"12.345 km"` | `"8. km"`, `"8,000 km"` |
| `_parse_kw` | Power in kW | `"350 kW"`, `"1.200 kW (1.632 cv)"` | `"350.5 kW"`, `"350kw"` |
| `_parse_minutes` | Charging time | `"18 Min."`, `"1.200 min"` | `"18.5 min"`, `"18mins"` |
| `_parse_int` | Owner count | `"1"`, `"  3  "` | `"1 owner"`, `"1.0"` |

## Configuration

The search URL is hard‑coded as `URL_SEARCH` at the top of `fetch.py`. You can modify it to change filters (price range, mileage, car type, etc.). The URL currently targets:

- Vehicle type: Car
- Condition: Used
- Fuel type: Electric
- Price: ≤ €60,000
- Mileage: ≤ 10,000 km
- Range: ≥ 400 km
- Categories: Off‑Road, Limousine

Adjust the query parameters as needed.

## Output Files

- **JSON database**: `cars.json` – contains all scraped data.
- **Excel spreadsheet**: `cars-<timestamp>.xlsx` (default) – formatted table with hyperlinks.

Both are ignored by `.gitignore` (`.json`, `.xlsx`) to avoid committing generated data.

## Dependencies

- **Python 3.7+** (uses `|` union type hints)
- [`webest2`](https://github.com/alobbs/webest) – web automation library
- [`tinydb`](https://pypi.org/project/tinydb/) – lightweight document database
- [`openpyxl`](https://pypi.org/project/openpyxl/) – Excel file manipulation
- [`fire`](https://pypi.org/project/fire/) – CLI generation from Python

## Notes & Limitations

- **Web Scraping**: The script relies on mobile.de’s HTML structure; changes to their CSS classes will break the selectors.
- **Rate Limiting**: No built‑in throttling; be respectful and avoid excessive requests.
- **Browser Driver**: `webest2` requires a compatible browser driver (e.g., ChromeDriver) installed and in your PATH.
- **Language**: The scraper is configured for the Spanish mobile.de site (`/es/`). For other locales, adjust the URL and the key names in `fetch_details()` (e.g., `'Kilometraje'`, `'Potencia'`).
- **Error Handling**: Minimal error recovery; if a page fails to load, the script continues with the next car.
