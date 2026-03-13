# Web Scraping to Create a Catalogue of iHerb Nutrition Supplements

#### Category: Python

---

I had a business idea to sell iHerb products to customers in South Korea.

As part of brainstorming the business process, I wanted to obtain a list of nutritional supplements sold on iHerb, along with some sales‑related information about them.

Next, I planned to check whether other vendors are currently selling these products on a specific online marketplace platform, and if so, gather the relevant sales information.

I’ll introduce:

- How to scrape iHerb product cards with Selenium (Edge) and capture key fields (name, brand, - link, prices, images, stock)
- How to calculate KRW prices from CAD and include shipping scenarios
- How to paginate across category pages with retries and progress tracking
- How to save a clean, deduplicated catalogue to Excel with unique file names
- How to query Naver Shopping for market prices, remove outliers, and compute useful stats
- How to compare iHerb landed cost vs. market prices to spot potential margin



**Ethics & Legal Note**


Always review and respect each site’s Terms of Service and robots policies before scraping. Use responsible rate limits, identify your client politely, and avoid disrupting services.


## Step 1: Set Up Selenium and Project

I've used **Microsoft Edge (Chromium)** with Selenium. iHerb blocks fully headless runs, so, I ran a regular browser session with a realistic User‑Agent and a few anti‑automation flags.


**Install the required packages**
```python3
  import subprocess

packages = ["requests", "beautifulsoup4", "pandas", "numpy ", "forex-python", "currencyconverter", "tqdm", "openpyxl", "selenium", "webdriver-manager", "nest_asyncio", "retrying", "python-dotenv"]

for package in packages:
    result = subprocess.run(
        ["pip", "install", package],
        stdout=subprocess.DEVNULL,  # Suppress standard output
        stderr=subprocess.PIPE      # Capture standard error
    )
    if result.returncode != 0:
        print(f"Installation failed for {package}: {result.stderr.decode('utf-8')}")
```
**Tip**: Keep secrets (like API keys) in environment variables or a local .env file, not in repo.

**Place the WebDriver**

Download **msedgedriver.exe** that matches Edge version and place it in a known folder (e.g., alongside the project scripts).

Update in 2026: selenium version 4.* and above can automatically download the necessary web-driver and this step is not required.


## Step 2: Build Utilities and Initialize the WebDriver

We’ll add logging, retry helpers, a unique file‑naming function, and Edge options. Shortcuts let us resume where we left off and keep outputs organized by date for repeated iterations in the future.

```python3
# Scraping iHerb using Selenium
import time, os, re, math, random, logging
from datetime import datetime

import pandas as pd
from tqdm import tqdm

from selenium import webdriver
from selenium.webdriver.edge.service import Service
from selenium.webdriver.common.by import By

from currency_converter import CurrencyConverter
from selenium.common.exceptions import NoSuchElementException, TimeoutException

# ---- Logging ----
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# ---- Lightweight retry decorator ----
def retry(max_retries=3, delay=2):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except (NoSuchElementException, TimeoutException) as e:
                    logging.warning(f"{func.__name__} failed ({attempt+1}/{max_retries}): {e}. Retrying...")
                    time.sleep(delay)
            logging.error(f"{func.__name__} failed after {max_retries} retries.")
            return []
        return wrapper
    return decorator

# Retry the scrape_page function
@retry(max_retries=3, delay=5)

# ---- File helpers ----
def find_latest_catalogue_file(directory):
    """Find the most recent 'iHerb_catalogue YYYY-MM-DD' Excel file."""
    pattern = r"^iHerb_catalogue (\d{4}-\d{2}-\d{2})( \(\d+\))?\.xlsx$"
    latest_file, latest_date = None, None

    for filename in os.listdir(directory):
        m = re.match(pattern, filename)
        if m:
            d = datetime.strptime(m.group(1), "%Y-%m-%d")
            if latest_date is None or d >= latest_date:
                latest_file, latest_date = filename, d
    if not latest_file:
        raise FileNotFoundError("No 'iHerb_catalogue YYYY-MM-DD' file found in the directory.")
    return os.path.join(directory, latest_file)

def get_unique_filename(directory, base_name):
    """Generate 'base_name YYYY-MM-DD.xlsx' and append (n) if needed."""
    date_str = datetime.now().strftime("%Y-%m-%d")
    os.makedirs(directory, exist_ok=True)
    filename = f"{base_name} {date_str}.xlsx"
    full_path = os.path.join(directory, filename)

    counter = 1
    while os.path.exists(full_path):
        filename = f"{base_name} {date_str} ({counter}).xlsx"
        full_path = os.path.join(directory, filename)
        counter += 1

    return full_path

# Directory containing the Excel files
directory = r"C:\Users\*****\Desktop\Charts\****\***** ****\iHerb_catalogue with Naver price"

try:
    latest_file_path = find_latest_catalogue_file(directory)
    print(f"Latest file found: {latest_file_path}")
    # Load the latest file if needed (optional)
    # df = pd.read_excel(file_path)  # Read data from an Excel file
except FileNotFoundError as e:
    print(e)

# Define the base name for the Excel file
base_name = "iHerb_catalogue"
# Get a unique filename
unique_file_path = get_unique_filename(directory, base_name)

# ---- Edge WebDriver ----
edge_driver_path = r"C:\Users\*****\Desktop\Charts\****\***** ****\iHerb_catalogue with Naver price\msedgedriver.exe"

edge_options = webdriver.EdgeOptions()
# edge_options.add_argument("--headless")  # iHerb blocks headless → run in visible mode
edge_options.add_argument("user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36")
edge_options.add_argument("--disable-blink-features=AutomationControlled")
edge_options.add_argument("--disable-gpu")

driver_service = Service(executable_path=edge_driver_path)
driver = webdriver.Edge(service=driver_service, options=edge_options)
```
**Retry decorator**: handle intermittent failures during scraping.

**Regex pattern**: match filenames like 'iHerb_catalogue YYYY-MM-DD.xlsx' or with (n)


## Step 3: Discover Pagination and Parse Product Cards

We grab the number of pages from the category pagination UI, then parse each product card for core attributes. We also compute CAD→KRW with/without shipping.

**Get the total number of pages**

Scrape the total number of pages from the pagination section with custom retry logic

```python3
def get_total_pages(url, max_retries=3, delay=2):
    """Scrape the total number of pages from the pagination section with custom retry logic."""
    driver.set_page_load_timeout(5)  # Set a 10-second timeout for loading each page
    attempts = 0  # Initialize retry counter
    
    while attempts < max_retries:
        try:
            driver.get(url)
            time.sleep(5)  # Wait for page to load

            # Find the pagination wrapper
            pagination = driver.find_elements(By.CLASS_NAME, 'pagination')

            # Check if pagination is found
            if pagination:
                # Find the last page number link
                last_page_link = pagination[-1].find_elements(By.CLASS_NAME, 'pagination-link')[-1]

                # Extract page number from the href attribute
                match = re.search(r'p=(\d+)', last_page_link.get_attribute('href'))
                if match:
                    total_pages = int(match.group(1))
                    logging.info(f"Total pages found: {total_pages}")
                    return total_pages if total_pages > 0 else 1
                else:
                    logging.warning("Could not extract page number from href attribute.")
                    return 1
            else:
                logging.warning("Pagination not found, defaulting to 1 page.")
                return 1  # If pagination is missing, assume 1 page
        
        except Exception as e:
            logging.error(f"Error in get_total_pages: {e}")
            attempts += 1
            time.sleep(delay)
        
    logging.error(f"Failed to retrieve total pages after {max_retries} retries. Defaulting to 1.")    
    return 1  # Default return value in case all retries fail
```

**Helpers for price parsing**

Split price into currency and amount.

For example, 'CAD$12.34' → ('CAD$', 12.34). Handles 'N/A' and missing values

```python3
def split_price(price):
    if price in ["Unavailable", "N/A", None, ""]:  # Unified handling for invalid prices
        return "N/A", 0.0  # Return consistent types: str for currency, float for amount
    currency = ''.join([char for char in price if not char.isdigit() and char != '.'])
    amount_str = ''.join([char for char in price if char.isdigit() or char == '.'])
    amount = float(amount_str) if amount_str else 0.0  # Convert amount to float
    return currency.strip(), amount
```

**Scrape a single page**

Selector note: For elements with multiple classes, use CSS selectors (not By.CLASS_NAME with dots).

```python3
@retry(max_retries=3, delay=2)
def scrape_page(url):
    driver.get(url)
    time.sleep(5)

    # Extract product containers
    products = driver.find_elements(By.CLASS_NAME, "product-inner")
    product_data = []

    for product in products:
        try:
            # Link + core metadata
            link_tag = product.find_element(By.CSS_SELECTOR, "a.absolute-link.product-link")
            order = link_tag.get_attribute("data-ga-product-position")
            name = link_tag.get_attribute("aria-label")
            brand = link_tag.get_attribute("data-ga-brand-name")
            out_of_stock = link_tag.get_attribute("data-ga-is-out-of-stock")
            iHerbID = link_tag.get_attribute("data-ga-product-id")
            link = link_tag.get_attribute("href") or "N/A"

            # Price block
            price_div = product.find_element(By.CLASS_NAME, "product-price-top")

            discounted_price = (
                price_div.find_element(By.CSS_SELECTOR, ".price.discount-red bdi").text.strip()
                if price_div.find_elements(By.CSS_SELECTOR, ".price.discount-red bdi") else "N/A"
            )

            if discounted_price != "N/A":
                regular_price = (
                    price_div.find_element(By.CSS_SELECTOR, ".price-olp bdi").text.strip()
                    if price_div.find_elements(By.CSS_SELECTOR, ".price-olp bdi") else "N/A"
                )
            else:
                # Try regular price locations (original, prohibited, cart-only, etc.)
                if price_div.find_elements(By.CSS_SELECTOR, ".price bdi"):
                    regular_price = price_div.find_element(By.CSS_SELECTOR, ".price bdi").text.strip()
                elif price_div.find_elements(By.CSS_SELECTOR, ".prohibited-text.text-danger-lighter bdi"):
                    regular_price = price_div.find_element(By.CSS_SELECTOR, ".prohibited-text.text-danger-lighter bdi").text.strip()
                elif price_div.find_elements(By.CLASS_NAME, "price-original-list"):
                    regular_price = price_div.find_element(By.CLASS_NAME, "price-original-list").text.strip()
                elif price_div.find_elements(By.CSS_SELECTOR, ".see-price-in-cart bdi"):
                    regular_price = price_div.find_element(By.CSS_SELECTOR, ".see-price-in-cart bdi").text.strip()
                else:
                    regular_price = "N/A"

            discount_rate = (
                price_div.find_element(By.CSS_SELECTOR, ".percentage-off bdi").text.strip().replace(" Off", "")
                if price_div.find_elements(By.CSS_SELECTOR, ".percentage-off bdi") else "0%"
            )

            # Split price and compute totals
            reg_currency, reg_amount_CAD = split_price(regular_price)
            disc_currency, disc_amount_CAD = split_price(discounted_price)

            # Shipping costs (CAD, tax incl.)
            iherb_shp_cost_CAD = 5 * 1.13
            maplepost_shp_cost_CAD = 15 * 1.13
            total_shp_cost_CAD = iherb_shp_cost_CAD + maplepost_shp_cost_CAD

            reg_amount_shp_inc_CAD = (reg_amount_CAD + total_shp_cost_CAD) if reg_amount_CAD else reg_amount_CAD
            disc_amount_shp_inc_CAD = (disc_amount_CAD + total_shp_cost_CAD) if disc_amount_CAD else disc_amount_CAD

            # CAD → KRW (forex_rate is defined later in Step 4)
            reg_price_KRW = math.ceil(reg_amount_CAD * forex_rate) if reg_amount_CAD else 0
            disc_price_KRW = math.ceil(disc_amount_CAD * forex_rate) if disc_amount_CAD else 0
            reg_price_inc_shp_KRW = math.ceil(reg_amount_shp_inc_CAD * forex_rate) if reg_amount_shp_inc_CAD else 0
            disc_price_inc_shp_KRW = math.ceil(disc_amount_shp_inc_CAD * forex_rate) if disc_amount_shp_inc_CAD else 0
           
            # Try to find an <img> element
            img = product.find_elements(By.TAG_NAME, "img")
            if img:
                # Extract primary image from the 'src' attribute
                primary_img = img[0].get_attribute("src") or "N/A"
                # Extract secondary image from 'srcset', if available
                srcset = img[0].get_attribute("srcset")
                secondary_image = srcset.split(", ")[-1].split(" ")[0] if srcset else "N/A"
            else:
                # Fallback: check for <div> with 'js-defer-image' class
                div_img = product.find_elements(By.CLASS_NAME, "js-defer-image")
                if div_img:
                    # Extract primary image from 'data-image-src' attribute
                    primary_img = div_img[0].get_attribute("data-image-src") or "N/A"
                    # Extract secondary image from 'data-image-retina-src', if available
                    secondary_image = div_img[0].get_attribute("data-image-retina-src") or "N/A"
                else:
                    # Default to "N/A" if neither image source is found
                    primary_img, secondary_image = "N/A", "N/A"

            # Log the product details
            product_data.append({
                "display_order": order,
                "iHerb_prod_id": iHerbID,
                "out_of_stock": out_of_stock,
                "name": name,
                "brand": brand,
                "currency": reg_currency,
                "reg_price_CAD": reg_amount_CAD,
                "disc_price_CAD": disc_amount_CAD,
                "reg_price_KRW": reg_price_KRW,
                "disc_price_KRW": disc_price_KRW,
                "reg_price_inc_shp_KRW": reg_price_inc_shp_KRW,
                "disc_price_inc_shp_KRW": disc_price_inc_shp_KRW,
                "discount_rate": discount_rate,
                "iHerb_prod_link": link,
                "img (320x320)": primary_img
            })

          except Exception as e:
            logging.error(f"Error scraping product card: {e}")

    return product_data
```

## Step 4: Paginate, Convert FX, and Save to Excel

We’ll crawl the category pages, apply a **5% buffer** to CAD → KRW to be conservative, and export a deduplicated catalogue.

```python3
def scrape_all_pages(base_url, total_pages, max_retries=5):
    all_products, failed_pages = [], []

    # For testing, this limits scraping to page 1. Change '2' → 'total_pages + 1' to crawl all pages.
    page_to_be_queried = min(total_pages + 1, 2)
    logging.info(f"Total pages to be queried: {page_to_be_queried - 1}")

    for page_num in tqdm(range(1, page_to_be_queried), desc="Scraping pages"):
        url = f"{base_url}?p={page_num}"
        retry_count = 0
        while retry_count < max_retries:
            try:
                page_products = scrape_page(url)
                if page_products:
                    all_products.extend(page_products)
                    logging.info(f"Page {page_num} scraped successfully.")
                    break
                else:
                    raise Exception("No data returned from page.")
            except Exception as e:
                retry_count += 1
                logging.warning(f"Error scraping page {page_num}: {e}. Retry {retry_count}/{max_retries}.")
                time.sleep(5)
        # Log and track if scraping fails after max retries
        if retry_count == max_retries:
            logging.error(f"Page {page_num} failed after {max_retries} retries.")
            failed_pages.append(page_num)

    # Retry scraping for failed pages
    if failed_pages:
        logging.info(f"Retrying failed pages: {failed_pages}")
        for page_num in tqdm(failed_pages, desc="Retrying failed pages"):
            url = f"{base_url}?p={page_num}"
            retry_count = 0
            while retry_count < max_retries:
                try:
                    page_products = scrape_page(url)
                    if page_products:
                        all_products.extend(page_products)
                        logging.info(f"Page {page_num} scraped successfully on retry.")
                        break
                    else:
                        raise Exception("No data returned from page.")
                except Exception as e:
                    retry_count += 1
                    logging.warning(f"Error scraping page {page_num} on retry: {e}. Retry {retry_count}/{max_retries}.")
                    time.sleep(5)
            if retry_count == max_retries:
                logging.error(f"Page {page_num} failed again after {max_retries} retries.")

    return all_products, failed_page s# Return scraped data and any remaining failed pages


# Main scraping logic
base_url = "https://kr.iherb.com/c/supplements" # https://ca.iherb.com/c/supplements for Canada / https://kr.iherb.com/c/supplements for Korea / https://www.iherb.com/c/supplements for USA
total_pages = get_total_pages(base_url) # Get total pages
total_pages = get_total_pages(base_url)

# FX rate: CAD → KRW with 5% buffer
c = CurrencyConverter()
forex_rate = c.convert(1, "CAD", "KRW") * 1.05

# Scrape all pages and store the results
all_products, failed_pages = scrape_all_pages(base_url, total_pages)

if all_products:
    # Convert results into a DataFrame and save to Excel
    df = pd.DataFrame(all_products)
    df = df.drop_duplicates(subset=["iHerb_prod_id"], keep="first") # De-duplicate iHerb_prod_id after Scraping
    if not df.empty and "name" in df.columns:
        df["name"] = df["name"].apply(lambda x: re.sub(r"[®™]", "", x)) # Remove specific non-alphanumeric characters
    df.to_excel(unique_file_path, index=False)
    print(f"Scraping complete. Data saved to {unique_file_path}")

if failed_pages:
    print(f"Pages failed: {failed_pages}")
```

**Why the buffer?**

Market FX can move intraday. Padding +5% gives a safety margin when comparing against KRW market prices.

## Step 5: Enrich with Naver Shopping Market Prices

To gauge competitiveness, we fetch **Naver Shopping** results for each in‑stock product name, compute KRW price distribution with simple outlier removal, and join the stats back to the iHerb catalogue.


**Security best practice (Emphasizing it again!)**

Store the Naver API credentials in environment variables (e.g., with python-dotenv) and never hard‑code secrets in repo.

```python3
# Scraping the price info from Naver Shopping and appending it to the iHerb catalogue.
import requests, numpy as np

# Filter to items with prices and not out of stock
in_stock_df = df[(df["out_of_stock"] == "False") & (df["reg_price_CAD"] > 0)].copy()
in_stock_df["discount_rate"] = in_stock_df["discount_rate"].str.replace("%", "").astype(float)

# --- Secrets (use env vars or .env) for API credential ---
from dotenv import load_dotenv
load_dotenv()

client_id = os.getenv("NAVER_CLIENT_ID")
client_secret = os.getenv("NAVER_CLIENT_SECRET")

market_price_info = []
# Limit the number of rows to 25,000 if the DataFrame has more rows
max_requests = 25000 # this is Naver API daily limit
limited_in_stock_df = in_stock_df.head(max_requests)

# Define the number of retries and delay for retries
max_retries = 3
retry_delay = 5  # seconds

session = requests.Session()
# Loop through each row in the limited DataFrame with tqdm to display progress
for _, row in tqdm(limited_in_stock_df.iterrows(), total=limited_in_stock_df.shape[0], desc="Processing items", unit="item", mininterval=1):
    keyword = row["name"]
    if pd.isna(keyword):
        # Keep alignment: push NaN row if we skip
        market_price_info.append({k: np.nan for k in [
            "min_p_sales_market","q1_p_sales_market","q2_p_sales_market","q3_p_sales_market",
            "max_p_sales_market","mean_p_sales_market","sd_p_sales_market"
        ]})
        continue

    encoded_keyword = requests.utils.quote(keyword)
    url = f"https://openapi.naver.com/v1/search/shop.json?query={encoded_keyword}&display=100&exclude=used:rental&sort=sim"
    headers = {"X-Naver-Client-Id": client_id, "X-Naver-Client-Secret": client_secret}

    # Attempt to make the request with retries
    for attempt in range(max_retries):
        try:
            response = requests.get(url, headers=headers)
            if response.status_code == 200:
                data = response.json()
                items = data.get('items', [])
                df_flattened = pd.json_normalize(items, sep='_')

                # Check if 'lprice' is in the DataFrame before proceeding
                if 'lprice' in df_flattened.columns:
                    df_flattened['lprice'] = pd.to_numeric(df_flattened['lprice'], errors='coerce')
                    lprice = df_flattened['lprice'].dropna()  # Drop NaNs for cleaner analysis
                    
                    # Calculate statistics if `lprice` has valid data
                    if not lprice.empty:
                        Q1 = lprice.quantile(0.25)
                        Q3 = lprice.quantile(0.75)
                        IQR = Q3 - Q1
                        lower_bound = Q1 - 1.5 * IQR
                        upper_bound = Q3 + 1.5 * IQR

                        price_no_outliers = lprice[(lprice >= lower_bound) & (lprice <= upper_bound)]
                        
                        mean_price = price_no_outliers.mean()
                        rounded_mean_price = None if np.isnan(mean_price) else math.ceil(mean_price * 100) / 100
                            
                        std_price = price_no_outliers.std()
                        rounded_std_price = None if np.isnan(std_price) else math.ceil(std_price * 100) / 100

                        row_data = {
                            'min_p_sales_market': price_no_outliers.min(),
                            'q1_p_sales_market': Q1,
                            'q2_p_sales_market': price_no_outliers.median(),
                            'q3_p_sales_market': Q3,
                            'max_p_sales_market': price_no_outliers.max(),
                            'mean_p_sales_market': rounded_mean_price, 
                            'sd_p_sales_market': rounded_std_price
                        }
                    else:
                        # No valid price data; record NaN values
                        row_data = {
                            'min_p_sales_market': np.nan,
                            'q1_p_sales_market': np.nan,
                            'q2_p_sales_market': np.nan,
                            'q3_p_sales_market': np.nan,
                            'max_p_sales_market': np.nan,
                            'mean_p_sales_market': np.nan,
                            'sd_p_sales_market': np.nan
                        }

                else:
                    # If 'lprice' is not present, record NaN values
                    row_data = {
                        'min_p_sales_market': np.nan,
                        'q1_p_sales_market': np.nan,
                        'q2_p_sales_market': np.nan,
                        'q3_p_sales_market': np.nan,
                        'max_p_sales_market': np.nan,
                        'mean_p_sales_market': np.nan,
                        'sd_p_sales_market': np.nan
                    }

                # Append row_data to the list
                market_price_info.append(row_data)
                break  # Exit retry loop after successful request

            else:
                logging.warning(f"API call failed for keyword '{keyword}' with status code {response.status_code}")
                row_data = {
                    'min_p_sales_market': np.nan,
                    'q1_p_sales_market': np.nan,
                    'q2_p_sales_market': np.nan,
                    'q3_p_sales_market': np.nan,
                    'max_p_sales_market': np.nan,
                    'mean_p_sales_market': np.nan,
                    'sd_p_sales_market': np.nan
                }
                market_price_info.append(row_data)
                break  # Stop retries if we get a response but with a failure status

        except requests.exceptions.RequestException as e:
            logging.error(f"Attempt {attempt + 1}/{max_retries} failed for keyword '{keyword}': {e}")
            if attempt < max_retries - 1:
                time.sleep(retry_delay)  # Wait before the next retry
            else:
                # If all attempts fail, log NaN values
                row_data = {
                    'min_p_sales_market': np.nan,
                    'q1_p_sales_market': np.nan,
                    'q2_p_sales_market': np.nan,
                    'q3_p_sales_market': np.nan,
                    'max_p_sales_market': np.nan,
                    'mean_p_sales_market': np.nan,
                    'sd_p_sales_market': np.nan
                }
                market_price_info.append(row_data)


# After the loop, concatenate limited_in_stock_df with the collected data
market_price_info = pd.DataFrame(market_price_info)
iherb_sale_naver_srch = pd.concat([limited_in_stock_df.reset_index(drop=True), market_price_info], axis=1)

# Additional analysis columns
iherb_sale_naver_srch["No competitor"] = iherb_sale_naver_srch["min_p_sales_market"].isnull()
iherb_sale_naver_srch["Cost advantage exc shp"] = iherb_sale_naver_srch["reg_price_KRW"] < iherb_sale_naver_srch["min_p_sales_market"]
iherb_sale_naver_srch["Additional margin exc shp"] = iherb_sale_naver_srch["min_p_sales_market"] - iherb_sale_naver_srch["reg_price_KRW"]
iherb_sale_naver_srch["Cost advantage inc shp"] = iherb_sale_naver_srch["reg_price_inc_shp_KRW"] < iherb_sale_naver_srch["min_p_sales_market"]
iherb_sale_naver_srch["Additional margin inc shp"] = iherb_sale_naver_srch["min_p_sales_market"] - iherb_sale_naver_srch["reg_price_inc_shp_KRW"]

# Save the resulting DataFrame to a Excel file
iherb_sale_naver_srch_file_path = get_unique_filename(directory, f"{base_name} in stock with Naver price")
iherb_sale_naver_srch.to_excel(iherb_sale_naver_srch_file_path, index=False)
logging.info(f"Scraping Naver Market completed. Data saved to '{iherb_sale_naver_srch_file_path}'")
```


**Outlier rule**

We trim outside **Q1 − 1.5×IQR** and **Q3 + 1.5×IQR** to avoid extreme vendor listings distorting the mean and median.


## Step 6: Wrap‑Up and Teardown

When the run completes, quit the driver to free system resources.

```python3
driver.quit()
```

**I've just done scrapping all 26,000's nutrition supplements from iHerb.ca with their product related information (product_id, in-stock/out-of-stock, product name, regular and discounted price/rate, product link and product image).**

**Then all competitors in the 3rd party market places were studied by checking if no one is currently selling any iHerb product there. If there is anyone selling the product, the statistics of the sales-price and cost advantag were analyzed.**


## Final Thoughts
With a few hundred lines of Python, I’ve built a practical **catalogue pipeline**:
- Collect iHerb product data + prices reliably with retries
- Normalize prices to KRW and include realistic shipping scenarios
- Enrich with Naver Shopping market stats to understand competition
- Export clean, deduplicated Excel files with date‑stamped names
This enables a fast way to explore product opportunities and spot potential margins before going deeper into operations and logistics.
