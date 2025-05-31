# Python Client for Capital.com API (Unofficial)

This Python library provides a client for interacting with the Capital.com trading API (REST and WebSocket). It aims to simplify common operations such as account management, trading, market data retrieval, and real-time data streaming.

**Disclaimer:** This is an unofficial library. Use it at your own risk, especially with live trading accounts. Always test thoroughly with a DEMO account first. The maintainers of this library are not responsible for any financial losses.

## Features

*   Session management (login, logout, session keep-alive via WebSocket ping).
*   Account information (balances, details, preferences).
*   Trade operations (open, close, update positions).
*   Working order management (get).
*   Historical market data retrieval.
*   Market details for instruments.
*   Trade size calculation helpers.
*   Transaction history.
*   Real-time data streaming via WebSockets for:
    *   Market quotes (bid/ask)
    *   OHLC candles
*   Context manager support for easy session handling.
*   Enum types for API parameters (e.g., `TradeDirection`, `HistoricalPriceResolution`).
*   Custom API error exception.

## Prerequisites

*   Python 3.7+
*   A Capital.com account (DEMO or LIVE) with API access enabled.
    *   API Key
    *   Identifier (usually your email or account ID)
    *   Password

## Installation

1.  **Install dependencies:**
    ```bash
    pip install requests websocket-client
    ```

2.  **Get the library:**
    Place the `CapitalA.py` file (the API client) in your project directory.

## Configuration

It is highly recommended to use environment variables for your API credentials:

*   `CAPITALCOM_API_KEY`: Your Capital.com API key.
*   `CAPITALCOM_IDENTIFIER`: Your account identifier (e.g., email).
*   `CAPITALCOM_PASSWORD`: Your account password.

Alternatively, you can pass them directly when initializing the `CapitalComAPI` class, but this is less secure.

## Basic Usage

This example demonstrates initializing the client and performing a basic operation. Always start with the `DEMO` environment for testing.

```python
import os
from CapitalA import CapitalComAPI, Environment, TradeDirection, CapitalComAPIError

API_KEY = os.environ.get("CAPITALCOM_API_KEY")
IDENTIFIER = os.environ.get("CAPITALCOM_IDENTIFIER")
PASSWORD = os.environ.get("CAPITALCOM_PASSWORD")

ENVIRONMENT_TO_USE = Environment.DEMO

if not all([API_KEY, IDENTIFIER, PASSWORD]):
    print("Error: Please set CAPITALCOM_API_KEY, CAPITALCOM_IDENTIFIER, and CAPITALCOM_PASSWORD environment variables.")
    exit()

try:
    with CapitalComAPI(api_key=API_KEY, identifier=IDENTIFIER, password=PASSWORD, environment=ENVIRONMENT_TO_USE) as client:
        print(f"Successfully logged in to {client.environment.value} environment.")
        print(f"Active Account ID: {client.active_account_id}")

        balance = client.get_balance()
        print(f"Current Balance: {balance}")

        example_epic = "IX.D.EURUSD.CFD.IP"
        market_info = client.get_market_details(epic=example_epic)
        if market_info:
            print(f"Market: {market_info.get('instrument', {}).get('name')}")
            print(f"Current Offer Price: {market_info.get('snapshot', {}).get('offer')}")
        else:
            print(f"Could not retrieve market info for {example_epic}")

except CapitalComAPIError as e:
    print(f"API Error: {e.message} (Status: {e.status_code}, Response: {e.response_data})")
except Exception as e:
    print(f"An unexpected error occurred: {e}")

print("Program finished.")

Core Functionality Examples

Refer to CapitalA/library.py for a more comprehensive example script demonstrating various features.

Authentication

Login is typically handled automatically when you initialize the class or enter the context manager. Logout is handled when exiting the context manager or by calling client.logout(). For manual control:

client = CapitalComAPI(api_key=API_KEY, identifier=IDENTIFIER, password=PASSWORD, environment=Environment.DEMO)
if client.login():
    print("Login successful manually.")
    # ... perform operations ...
    client.logout()
else:
    print("Login failed.")
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END
Account Information

Fetch details about your account(s).

details = client.get_active_account_details()
if details:
    print(f"Account Currency: {details.get('currency')}")

accounts = client.get_accounts()
for acc in accounts:
    print(f"Account ID: {acc['accountId']}, Status: {acc['status']}")

# To switch the active account if multiple exist:
# if len(accounts) > 1:
#     if client.switch_account(accounts[1]['accountId']):
#         print(f"Switched to account: {client.active_account_id}")
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END
Trading (Use DEMO Environment for Testing!)

Ensure EPICs (instrument identifiers like IX.D.EURUSD.CFD.IP) are valid for your account and environment. Use TradeDirection.BUY or TradeDirection.SELL.

from CapitalA import TradeDirection

example_epic = "IX.D.EURUSD.CFD.IP"
trade_size = 0.01

try:
    # You can calculate trade size based on desired margin:
    # calculated_size = client.calculate_trade_size_for_margin(
    #     epic=example_epic,
    #     margin_amount_in_quote_currency=20.0,
    #     direction=TradeDirection.BUY
    # )
    # if calculated_size > 0:
    #    trade_size = calculated_size
    # else:
    #    print(f"Could not calculate valid trade size for {example_epic}.")


    open_response = client.open_trade(
        epic=example_epic,
        direction=TradeDirection.BUY,
        size=trade_size,
        stop_distance=15,
        profit_distance=30
    )
    if open_response and open_response.get("dealReference"):
        deal_ref = open_response["dealReference"]
        print(f"Trade opened with reference: {deal_ref}")
        
        # To manage the position (update/close), you'll need its 'dealId'.
        # This is typically found by fetching open positions after opening the trade.
        # Example (simplified):
        # import time
        # time.sleep(2) 
        # positions = client.get_open_positions()
        # my_deal_id = None
        # for p in positions:
        #     if p.get("position",{}).get("epic") == example_epic:
        #          my_deal_id = p.get("position",{}).get("dealId")
        #          break
        # if my_deal_id:
        #    client.update_trade(deal_id=my_deal_id, stop_level=new_stop_price)
        #    client.close_trade(deal_id=my_deal_id)
    else:
        print(f"Failed to open trade: {open_response}")

except CapitalComAPIError as e:
    print(f"Trading Error: {e}")
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END
Market Data

Retrieve historical price data.

from CapitalA import HistoricalPriceResolution

example_epic = "IX.D.SPTRD.CFD.IP"

prices_data = client.get_historical_prices(
    epic=example_epic,
    resolution=HistoricalPriceResolution.HOUR_4,
    num_points=50
)
if prices_data and prices_data.get("prices"):
    for candle in prices_data["prices"]:
        print(f"Time: {candle['snapshotTimeUTC']}, Open Bid: {candle['openPrice']['bid']}")
else:
    print(f"Could not fetch historical prices for {example_epic}")
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END
Real-time Data (WebSockets)

Subscribe to live market quotes or OHLC updates. Define callback functions to process incoming data.

from CapitalA import WebsocketDataType, HistoricalPriceResolution, OhlcBarType
import time

def my_market_update_handler(data: dict):
    print(f"MARKET Update for {data.get('epic')}: Bid={data.get('bid')}, Ask={data.get('offer')}")

def my_ohlc_update_handler(data: dict):
    print(f"OHLC Update for {data.get('epic')} ({data.get('resolution')}): O={data.get('openPrice')}, C={data.get('closePrice')}")

market_epic = "IX.D.EURUSD.CFD.IP"
ohlc_epic = "IX.D.DAX.IFD.IP"

client.subscribe_to_epic_data(
    epic=market_epic,
    data_type=WebsocketDataType.MARKET,
    callback=my_market_update_handler
)
client.subscribe_to_epic_data(
    epic=ohlc_epic,
    data_type=WebsocketDataType.OHLC,
    callback=my_ohlc_update_handler,
    resolution=HistoricalPriceResolution.MINUTE_5,
    bar_type=OhlcBarType.CLASSIC
)

print("Subscribed to WebSocket streams. Waiting for updates for 30 seconds...")
time.sleep(30) # Keep the main script alive to receive messages

# Unsubscribing (logout/context exit also stops WebSockets):
# client.unsubscribe_from_epic_data(epic=market_epic, data_type=WebsocketDataType.MARKET)
# client.stop_all_websocket_subscriptions() # To stop all active subscriptions
print("Finished receiving WebSocket updates.")
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END
Error Handling

The library raises CapitalComAPIError for API-specific issues. Standard requests exceptions might also occur for network problems.

import requests # Import for requests.exceptions

try:
    client.get_balance()
except CapitalComAPIError as e:
    print(f"API Error occurred: {e.message}")
    print(f"Status Code: {e.status_code}")
    print(f"Response Data: {e.response_data}")
except requests.exceptions.RequestException as e:
    print(f"Network error: {e}")
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END
Running the Full Example

A comprehensive example script (library.py) is usually provided to demonstrate multiple features. Assuming the following project structure:

your_project_root/
├── CapitalA.py       # The API client library
└── CapitalA/
    └── library.py    # The full example script
└── README.md
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
IGNORE_WHEN_COPYING_END

Ensure CapitalA.py is in your project root.

Create a subfolder CapitalA.

Place the example script (library.py) into the CapitalA subfolder.

Set your environment variables: CAPITALCOM_API_KEY, CAPITALCOM_IDENTIFIER, CAPITALCOM_PASSWORD.

From your project root directory in the terminal, run:

python CapitalA/library.py
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Bash
IGNORE_WHEN_COPYING_END

Ensure the example script uses valid EPICs for the DEMO environment you are testing against.

Important Notes

EPICs: Instrument identifiers (EPICs) are crucial and can vary. Check the Capital.com platform or API documentation for valid EPICs for your account type and environment (DEMO/LIVE).

Demo vs. Live: Always test thoroughly in the Environment.DEMO before considering Environment.LIVE. Trading involves financial risk.

Rate Limits: Be mindful of API rate limits imposed by Capital.com. The library does not currently implement explicit client-side rate limiting.

Error Checking: Always check the return values of API calls and implement robust error handling using try-except blocks for CapitalComAPIError and other potential exceptions.

WebSocket Stability: WebSocket connections can be interrupted. The library includes basic reconnection logic and application-level pings to maintain the service session. For critical applications, you might need more sophisticated connection management.

Contributing

Contributions, bug reports, and feature requests are welcome! Please open an issue on the project's repository or submit a pull request.

License

This library is provided as-is. If you intend to distribute or build upon this, consider adding a standard open-source license (e.g., MIT, Apache 2.0) to the project.

IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
IGNORE_WHEN_COPYING_END
