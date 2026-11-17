# YouTube Channel Video Exporter

Export a channel's recent YouTube video metadata to a CSV file for spreadsheet analysis.

This Python script uses the **YouTube Data API v3** to collect video titles, links, thumbnail URLs, view counts, and publication timestamps. It retrieves up to five recent videos per run.

## Features

- Fetch videos from a channel in newest-first order.
- Collect five useful metadata fields in one CSV.
- Print the collected values to the terminal.
- Export using UTF-8 with BOM for spreadsheet compatibility.

The script retrieves metadata only; it does not download videos or thumbnail images.

## Example output

The included [sample CSV](sample_scrapped_data.csv) shows the output format. These are historical sample values.

| Title | Views | Published (UTC) |
| --- | ---: | --- |
| Justice. | 124433 | 2025-01-13T21:33:37Z |
| The Awoken Giant Terrorizing English Football | 171964 | 2025-01-10T18:32:56Z |
| Manchester Is BROKEN | 175139 | 2024-12-27T19:45:12Z |

Each output row has these columns:

```csv
Video,Thumbnail,Title,Views,Time
```

## Getting started

### Requirements

- Python 3 and Git.
- A Google Cloud project with **YouTube Data API v3** enabled.
- An API key and the target channel's channel ID.

Follow Google's [YouTube Data API setup guide](https://developers.google.com/youtube/v3/getting-started) to configure a project and credentials.

### Install

```bash
git clone https://github.com/SambhavPyakurel/Youtube_Scrapper.git
cd Youtube_Scrapper
```

**Windows PowerShell:**

```powershell
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install google-api-python-client
```

**macOS / Linux:**

```bash
python3 -m venv .venv
./.venv/bin/python -m pip install google-api-python-client
```

### Configure

In your local `scrapper.py`, replace the placeholders:

```python
API_KEY = 'YOUR_API_KEY'
channel_id = 'CHANNEL_ID'
```

Use the channel's ID, not its display name or `@handle`. The script reads these constants directly; `.env` files are not loaded. Keep your real API key out of Git commits.

### Run

```powershell
# Windows
.\.venv\Scripts\python.exe scrapper.py
```

```bash
# macOS / Linux
./.venv/bin/python scrapper.py
```

The script prints its results and creates **`storage.csv`** in the current directory. Run from the repository root to keep the output beside the script. Each successful run replaces the previous output file.

## Project structure

```text
Youtube_Scrapper/
├── scrapper.py               # API requests and CSV export
├── sample_scrapped_data.csv  # Example output
└── README.md
```

The only third-party Python dependency is `google-api-python-client`; CSV writing uses Python's standard library.

## Notes and troubleshooting

- Internet access, a valid API key, and available API quota are required. For authentication errors, check the enabled API, key restrictions, and the returned error message.
- The script requests five videos per search and does not paginate through a channel's full history.
- Metadata is collected in separate requests and joined by list position. Changing search results can misalign rows; this utility is best suited to small exports.
- If a search request rejects `part='id'`, use `part='snippet'` as specified in the [search.list reference](https://developers.google.com/youtube/v3/docs/search/list). Two calls currently use `id`.
- API failures stop execution; automatic retries are not implemented. Titles may retain HTML entities from API responses.
