# YouTube Channel Video Exporter

A Python script that retrieves metadata for up to five recent videos from a specified YouTube channel and exports it to CSV. Despite the repository name, it uses the **YouTube Data API v3**, rather than scraping HTML pages.

## What it does

- Searches channel videos in newest-first order with `maxResults=5`.
- Retrieves watch links, default thumbnail links, titles, view counts, and publication timestamps.
- Prints collected lists to the terminal.
- Writes one row per video to `storage.csv` using UTF-8 with BOM for spreadsheet compatibility.

It does not download videos or images. There is no graphical interface; the output table below provides a preview.

## Example output

The repository includes [sample_scrapped_data.csv](sample_scrapped_data.csv). These are historical sample values, not a fresh API response:

| Title | Views | Published (UTC) |
| --- | ---: | --- |
| Justice. | 124433 | 2025-01-13T21:33:37Z |
| The Awoken Giant Terrorizing English Football | 171964 | 2025-01-10T18:32:56Z |
| Manchester Is BROKEN | 175139 | 2024-12-27T19:45:12Z |

The complete output header is:

```csv
Video,Thumbnail,Title,Views,Time
```

`Video` is the watch URL, `Thumbnail` is an image URL, and `Time` is the publication timestamp returned by the API. Titles can retain HTML entities, as one bundled sample row demonstrates.

## How to run

### 1. Download and install

Use Git and Python 3; Python 3.11 or 3.12 is a practical starting point. No pinned requirements file is included.

```bash
git clone https://github.com/SambhavPyakurel/Youtube_Scrapper.git
cd Youtube_Scrapper
```

**Windows PowerShell:**

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install google-api-python-client
```

**macOS / Linux:**

```bash
python3 -m venv .venv
./.venv/bin/python -m pip install google-api-python-client
```

### 2. Configure the API and channel

Create a Google Cloud project, enable the YouTube Data API v3, and create an API key using Google's [getting-started instructions](https://developers.google.com/youtube/v3/getting-started). Find the target channel's ID, rather than its display name or `@handle`.

Replace these placeholders in your local `scrapper.py`:

```python
API_KEY = 'YOUR_API_KEY'
channel_id = 'CHANNEL_ID'
```

The script reads these constants directly, not a `.env` file or environment variables. Keep real keys out of commits and screenshots. Environment-based configuration would be a useful improvement.

### 3. Run the script

**Windows PowerShell:**

```powershell
.\.venv\Scripts\python.exe scrapper.py
```

**macOS / Linux:**

```bash
./.venv/bin/python scrapper.py
```

Run from the repository root to create `storage.csv` there. Each successful run **overwrites** that file. Open it in a spreadsheet application or data-analysis tool. Live execution requires internet access and an enabled API project with available quota.

## Implementation

| Function | Data retrieved |
| --- | --- |
| `extract_video_urls` | Watch URLs from search-result IDs |
| `extract_thumbnail_urls` | Default thumbnail URLs from search snippets |
| `extract_video_titles` | Titles from search snippets |
| `extract_video_views` | View counts from a batched video-statistics request |
| `extract_upload_times` | Publication timestamps from individual video requests |

Five lists are combined with `zip()` and written with Python's standard `csv` module. With five returned videos, the implementation performs five search requests and six video-detail requests. It does not paginate through the channel's full history.

## Verification and limitations

Source was inspected and syntax-checked, and bundled sample output was reviewed. **No live API run was performed** because no configured key was provided. The historical sample does not establish that today's requests will succeed.

- **API compatibility:** Two search calls use `part='id'`, while the current [search.list reference](https://developers.google.com/youtube/v3/docs/search/list) instructs clients to set `part` to `snippet`. If rejected, update those requests to the documented value and verify the returned IDs.
- **Repeated requests:** Fields are collected through separate searches. Results may shift between calls, and lists are joined by position instead of video ID. Rows can become misaligned; `zip()` silently truncates to the shortest list.
- **Limited coverage:** Each search requests five results without pagination. Changing the limit requires reviewing all search calls and the API's allowed values.
- **Failures:** There is no retry logic or handling for invalid keys, missing fields, unavailable channels, quota errors, or empty video-detail responses. Exceptions stop execution.
- **Imports:** Importing the script also builds the client and runs requests; there is no `if __name__ == '__main__'` guard.
- **Authentication troubleshooting:** Check the key, enabled API, key restrictions, and quota against the returned error.
- **Missing module:** Install `google-api-python-client` with the same interpreter used to launch the script.

## Project files

```text
Youtube_Scrapper/
|-- scrapper.py              # API requests and CSV export
|-- sample_scrapped_data.csv # Historical example output
`-- README.md
```

`storage.csv` is generated at runtime. There is no dependency lock, automated test suite, or CLI argument parser.
