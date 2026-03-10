# price_radar_agent

## PriceRadarAiAgent

PriceRadarAiAgent is a Python-based price comparison agent.  
It can take a product name or URL, scrape Amazon and Flipkart using Apify actors, store price history in SQLite, and use Google's Gemini model to recommend the best deal.

### Features

- **Input**: Product name or product URL.
- **Scraping**: Uses Apify actors to fetch data from Amazon and Flipkart.
- **Extracted fields**: Product name, price, rating, and product URL.
- **Storage**: Saves offer data into a SQLite database for historical tracking.
- **AI analysis**: Sends aggregated offers and history to Gemini for reasoning.
- **Interfaces**:
  - **FastAPI backend** (`/compare` endpoint).
  - **Simple CLI** (`python -m api.agent` or `python api/agent.py`).

### Project Structure

```text
price_radar_agent/
  scrapers/
    amazon_scraper.py
    flipkart_scraper.py
  ai/
    gemini_analysis.py
  database/
    db.py
  api/
    main.py
    config.py
    agent.py
  requirements.txt
  .env.example
  README.md
```

### Prerequisites

- **Python** 3.9+ recommended.
- **Apify account** with an API token.
- **Google AI Studio / Gemini API key**.

### Installation

1. **Clone or copy the project**

   ```bash
   cd "c:\Users\SathvikReddyMandala\Downloads"
   cd price_radar_agent
   ```

2. **Create and activate a virtual environment (recommended)**

   ```bash
   python -m venv .venv
   .venv\Scripts\activate
   ```

3. **Install dependencies**

   ```bash
   pip install -r requirements.txt
   ```

4. **Configure environment variables**

   Copy `.env.example` to `.env` and fill in your credentials:

   ```bash
   copy .env.example .env
   ```

   Then edit `.env` and set:

   - **APIFY_TOKEN**: Your Apify API token.
   - **GEMINI_API_KEY**: Your Google Gemini API key.
   - **DATABASE_URL** (optional): SQLite URL, default is `sqlite:///price_data.db`.
   - **AMAZON_ACTOR_ID** (optional): Amazon Apify actor ID.
   - **FLIPKART_ACTOR_ID** (optional): Flipkart Apify actor ID.

### Running the FastAPI backend

From the project root:

```bash
uvicorn api.main:app --reload
```

The API will start on `http://127.0.0.1:8000`.

- **Swagger docs**: `http://127.0.0.1:8000/docs`
- **Redoc docs**: `http://127.0.0.1:8000/redoc`

#### Example API request

**Endpoint**: `POST /compare`

**Body**:

```json
{
  "product_name": "iPhone 15",
  "product_url": null
}
```

**Example response shape**:

```json
{
  "best_deal": {
    "website": "Flipkart",
    "product_name": "Apple iPhone 15 (128 GB, Blue)",
    "price": 73999,
    "rating": 4.6,
    "product_url": "https://www.flipkart.com/..."
  },
  "analysis": "Flipkart currently has the lowest price. Based on historical trends this is a good time to buy.",
  "offers": [
    {
      "product_name": "Apple iPhone 15 (128 GB, Blue)",
      "price": 74999,
      "rating": 4.5,
      "product_url": "https://www.amazon.in/...",
      "website": "Amazon"
    },
    {
      "product_name": "Apple iPhone 15 (128 GB, Blue)",
      "price": 73999,
      "rating": 4.6,
      "product_url": "https://www.flipkart.com/...",
      "website": "Flipkart"
    }
  ]
}
```

### Using the CLI

You can run a simple command-line interface that wraps the same agent logic.

From the project root:

```bash
python -m api.agent --product-name "iPhone 15"
```

Or:

```bash
python api/agent.py --product-url "https://www.amazon.in/..."
```

If you run without arguments:

```bash
python -m api.agent
```

You will be interactively prompted:

```text
Enter product name or URL: iPhone 15
```

Sample CLI output:

```text
Best Deal:
Website: Flipkart
Price: ₹73999

AI Analysis:
Flipkart currently has the lowest price. Based on historical trends this is a good time to buy.
```

### How it works (high level)

- **Scrapers** (`scrapers/amazon_scraper.py`, `scrapers/flipkart_scraper.py`):
  - Use the official Apify Python client (`apify-client`).
  - Call Amazon and Flipkart actors with either a search query or a product URL.
  - Normalize results into a common schema (`product_name`, `price`, `rating`, `product_url`, `website`).

- **Database** (`database/db.py`):
  - Uses SQLite to store each scraped offer.
  - Provides helper functions to insert records and fetch recent history for a product.

- **AI reasoning** (`ai/gemini_analysis.py`):
  - Calls Gemini (via `google-generativeai`) with current offers and historical data.
  - Asks the model to respond with JSON that includes a single `best_deal` and an `analysis` string.

- **Agent orchestrator** (`api/agent.py`):
  - Brings everything together:
    - Scrapes Amazon + Flipkart.
    - Stores offers in SQLite.
    - Fetches price history.
    - Calls Gemini and formats the result.
  - Exposes a small CLI helper.

- **FastAPI API** (`api/main.py`):
  - Exposes a `/compare` endpoint that returns the same combined result as the CLI.

### Notes and customization

- **Apify actors**: Different actors may have slightly different input/field names. If you choose another actor, update the input payload and field mappings in `amazon_scraper.py` and `flipkart_scraper.py`.
- **Gemini model**: The default model is `gemini-1.5-flash`. You can change it in `ai/gemini_analysis.py`.
- **Database location**: Configure the `DATABASE_URL` environment variable if you want the SQLite file stored elsewhere.

