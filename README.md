[![Python](https://img.shields.io/badge/Python-3.x-blue.svg)](https://www.python.org/)


```python
import requests
import json
import pandas as pd

# Your CoinMarketCap API key
API_KEY = ' '  # Replace with your actual API key

# Base URL for the CoinMarketCap API
BASE_URL = 'https://pro-api.coinmarketcap.com/v1'

# Endpoint to get the latest listings of cryptocurrencies
LISTINGS_ENDPOINT = '/cryptocurrency/listings/latest'

# Parameters for the request
parameters = {
    'start': '1',
    'limit': '100',  # Get the top 100 cryptocurrencies
    'convert': 'USD'  # Convert prices to USD
}

headers = {
    'Accepts': 'application/json',
    'X-CMC_PRO_API_KEY': API_KEY,
}

try:
    # Make the API request
    response = requests.get(BASE_URL + LISTINGS_ENDPOINT, headers=headers, params=parameters)
    response.raise_for_status()  # Raise an exception for bad status codes

    # Parse the JSON response
    data = response.json()['data']

    # Convert the JSON data to a Pandas DataFrame for easier handling
    df = pd.DataFrame(data)

    # Function to extract and format USD quote data
    def extract_and_format_usd_quote(quote):
        if isinstance(quote, dict) and 'USD' in quote:
            usd_data = quote['USD']
            # Format the price, handling potential errors
            try:
                price = float(usd_data.get('price', 0)) #important
                price = round(price, 2)  # Format to 2 decimal places
            except (ValueError, TypeError):
                price = 0.0  # Or some other default value, handle error

            volume_24h = usd_data.get('volume_24h')
            volume_change_24h = usd_data.get('volume_change_24h')
            percent_change_1h = usd_data.get('percent_change_1h')
            percent_change_24h = usd_data.get('percent_change_24h')
            percent_change_7d = usd_data.get('percent_change_7d')
            percent_change_30d = usd_data.get('percent_change_30d')
            percent_change_60d = usd_data.get('percent_change_60d')
            percent_change_90d = usd_data.get('percent_change_90d')
            market_cap = usd_data.get('market_cap')
            market_cap_dominance = usd_data.get('market_cap_dominance')
            last_updated = usd_data.get('last_updated')
            return {
                'price': price,
                'volume_24h': volume_24h,
                'volume_change_24h': volume_change_24h,
                'percent_change_1h': percent_change_1h,
                'percent_change_24h': percent_change_24h,
                'percent_change_7d': percent_change_7d,
                'percent_change_30d': percent_change_30d,
                'percent_change_60d': percent_change_60d,
                'percent_change_90d': percent_change_90d,
                'market_cap': market_cap,
                'market_cap_dominance': market_cap_dominance,
                'last_updated': last_updated
            }
        return {}

    # Apply the function to the 'quote' column to extract and format data
    df_usd_quote = df['quote'].apply(extract_and_format_usd_quote).apply(pd.Series)

    # Concatenate the desired columns with the original DataFrame
    df = pd.concat([df[['id', 'name', 'symbol']], df_usd_quote], axis=1)

    # Select the desired columns
    df = df[['name', 'symbol', 'price', 'volume_24h', 'volume_change_24h',
             'percent_change_24h', 'percent_change_7d', 'percent_change_30d',
             'percent_change_60d', 'percent_change_90d', 'market_cap',
             'market_cap_dominance', 'last_updated']]

    # Convert 'last_updated' to datetime and format it
    df['date'] = pd.to_datetime(df['last_updated']).dt.strftime('%Y-%m-%d')

    # Drop the original 'last_updated' column
    df = df.drop(columns=['last_updated'])

    # Display the first few rows of the cleaned DataFrame
    print("\nCleaned and Formatted DataFrame:")
    print(df.head())

    # Save the cleaned DataFrame to a CSV file
    df.to_csv('cryptocurrency_data_cleaned.csv', index=False)
    print("\nCleaned data saved to cryptocurrency_data_cleaned.csv")

except requests.exceptions.RequestException as e:
    print(f"API request error: {e}")
except json.JSONDecodeError as e:
    print(f"JSON decoding error: {e}")
except KeyError as e:
    print(f"Key error in JSON response: {e}")
except Exception as e:
    print(f"An unexpected error occurred: {e}")

```

This Python script fetches cryptocurrency data from the CoinMarketCap API and processes it into a clean, usable format. Here's what it does:

1. **API Setup**: Configures the connection to the CoinMarketCap API using an API key, setting parameters to retrieve the top 100 cryptocurrencies with USD conversion.

2. **Data Retrieval**: Makes an HTTP request to the API endpoint and retrieves the cryptocurrency data in JSON format.

3. **Data Processing**:
   - Converts the JSON response to a Pandas DataFrame
   - Extracts and formats the USD pricing data using a custom function
   - Handles potential errors in price formatting
   - Extracts key metrics like price, volume, percent changes, and market cap

4. **Data Transformation**:
   - Combines the basic cryptocurrency information with the extracted USD metrics
   - Selects only the relevant columns for the final dataset
   - Converts timestamp data to a standardized date format
   - Removes redundant columns

5. **Output**:
   - Displays a preview of the cleaned data
   - Saves the processed data to a CSV file named 'cryptocurrency_data_cleaned.csv'

6. **Error Handling**: Implements comprehensive error handling for API requests, JSON parsing, missing keys, and unexpected issues.

This script is useful for cryptocurrency analysts, traders, or data scientists who need clean, structured cryptocurrency market data for analysis or visualization.

