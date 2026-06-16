import requests
import json

def fetch_crypto_data():
    """
    Fetches live cryptocurrency data from the CoinGecko public API,
    parses the JSON response, and handles potential connection errors.
    """
    # Public API URL for top 50 cryptocurrencies market data
    url = "https://api.coingecko.com/api/v3/coins/markets"
    params = {
        'vs_currency': 'usd',
        'order': 'market_cap_desc',
        'per_page': 50,
        'page': 1,
        'sparkline': 'false'
    }
    
    # Setting a standard user-agent header to avoid getting blocked by rate limits
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
    }

    print("Fetching live data from API...")
    
    try:
        # 1. Use the requests module to fetch data
        response = requests.get(url, params=params, headers=headers)
        
        # Check if HTTP request was successful
        response.raise_for_status()
        
        # 2. Parse JSON response into a Python list/dictionary
        crypto_list = response.json()
        return crypto_list

    except requests.exceptions.HTTPError as http_err:
        print(f"HTTP error occurred: {http_err}")
    except requests.exceptions.ConnectionError:
        print("Failed to connect. Please check your internet connection.")
    except Exception as err:
        print(f"An unexpected error occurred: {err}")
    return None

def display_and_filter_data(crypto_list):
    """
    Displays the formatted data and provides interactive search/filtering functionality.
    """
    if not crypto_list:
        print("No data available to display.")
        return

    # Print a clean summary header of the available data
    print(f"\n--- Successfully loaded top {len(crypto_list)} cryptocurrencies ---")
    
    # 3. Add interactive search/filter support
    while True:
        search_query = input("\nEnter a coin name or symbol to filter (or type 'exit' to quit): ").strip().lower()
        
        if search_query == 'exit':
            print("Exiting search utility.")
            break
            
        if not search_query:
            print("Please enter a valid search term.")
            continue

        # Filter the list based on user input matching the name or symbol
        filtered_results = [
            coin for coin in crypto_list 
            if search_query in coin['name'].lower() or search_query in coin['symbol'].lower()
        ]
        
        # Display the results cleanly
        if filtered_results:
            print("\n" + "="*65)
            print(f"{'Rank':<6}{'Name (Symbol)':<25}{'Current Price':<18}{'24h Change (%)':<10}")
            print("="*65)
            
            for coin in filtered_results:
                name_str = f"{coin['name']} ({coin['symbol'].upper()})"
                price = f"${coin['current_price']:,}"
                change_24h = f"{coin['price_change_percentage_24h']:.2f}%" if coin['price_change_percentage_24h'] is not None else "N/A"
                
                print(f"{coin['market_cap_rank']:<6}{name_str:<25}{price:<18}{change_24h:<10}")
            print("="*65)
        else:
            print(f"No cryptocurrencies found matching '{search_query}'.")

if __name__ == "__main__":
    print("--- API Integration & Data Filtering Tool ---")
    
    # Execute fetch and filter operations
    live_data = fetch_crypto_data()
    display_and_filter_data(live_data)
