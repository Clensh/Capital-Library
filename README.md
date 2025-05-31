# CapitalComAPI Python Client

A Python client for interacting with the Capital.com trading platform API, supporting both RESTful operations and WebSocket real-time data streaming.

## Overview

This library provides a comprehensive interface to manage your Capital.com account, execute trades, retrieve historical data, and stream live market updates. It handles authentication, session management, and offers robust WebSocket connectivity with automatic reconnection and keep-alive pings.

## Key Features

*   **DEMO & LIVE Environment Support:** Easily switch between trading environments.
*   **Full Session Management:** Handles login, logout, and automatic session token refresh for REST API calls.
*   **Account Operations:** Fetch account details, balance, switch accounts, manage preferences (hedging, leverage).
*   **Trading:** Open, close, and update positions. Get open positions and working orders.
*   **Market Data:**
    *   Retrieve historical prices (OHLC) for various resolutions.
    *   Fetch detailed market information for instruments.
    *   Get transaction history and closed trades.
*   **Trade Sizing Utilities:** Calculate trade sizes based on investment amount (account leverage) or margin (instrument margin factor).
*   **WebSocket Streaming:**
    *   Subscribe to real-time market quotes (`MARKET`).
    *   Subscribe to real-time OHLC candle updates (`OHLC`) with various resolutions and bar types (Classic, Heikin-Ashi).
    *   Robust connection management with automatic reconnections and application-level pings.
*   **Error Handling:** Custom `CapitalComAPIError` for API-specific issues.
*   **Logging:** Integrated logging for operational insights.
*   **Context Manager:** Supports `with` statement for automatic login/logout.

## Prerequisites

*   Python 3.7+
*   Required libraries:
    *   `requests`
    *   `websocket-client`

You can install them using pip:
```bash
pip install requests websocket-client

Installation

Save the CapitalA.py file in your project directory or a location accessible by your Python interpreter.

Import the CapitalComAPI class and other necessary components from it.

Quick Start
import time
import logging
from CapitalA import (
    CapitalComAPI, Environment, TradeDirection,
    HistoricalPriceResolution, WebsocketDataType, CapitalComAPIError
)

# Optional: Configure logging for more details
# logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(name)s - %(levelname)s - %(message)s')
# logging.getLogger('CapitalA').setLevel(logging.DEBUG)


API_KEY = "YOUR_API_KEY"
IDENTIFIER = "YOUR_EMAIL_OR_ACCOUNT_ID" # e.g., your email
PASSWORD = "YOUR_PASSWORD"
ENVIRONMENT = Environment.DEMO # Or Environment.LIVE

# --- WebSocket Callbacks (Example) ---
def market_data_handler(data):
    print(f"LIVE QUOTE ({data['epic']}): Bid={data.get('bid')} Offer={data.get('offer')}")

def ohlc_data_handler(data):
    print(f"LIVE OHLC ({data['epic']}/{data['resolution']}): C={data.get('c')}")

if __name__ == "__main__":
    try:
        # Use context manager for automatic login/logout
        with CapitalComAPI(API_KEY, IDENTIFIER, PASSWORD, ENVIRONMENT) as api:
            print(f"Successfully logged into {api.environment.value} environment.")
            print(f"Active Account ID: {api.active_account_id}")

            # Get account balance
            balance = api.get_balance()
            if balance is not None:
                print(f"Account Balance: {balance}")

            # Get market details for EUR/USD
            eurusd_details = api.get_market_details(epic="EURUSD")
            if eurusd_details:
                print(f"EURUSD Min Trade Size: {eurusd_details['dealingRules']['minDealSize']['value']}")

            # Subscribe to EURUSD live quotes
            api.subscribe_to_epic_data("EURUSD", WebsocketDataType.MARKET, market_data_handler)

            # Subscribe to US500 (S&P 500 CFD example) 1-minute OHLC
            # Ensure the epic 'US500' is correct for your broker
            api.subscribe_to_epic_data(
                "US500", # Example epic
                WebsocketDataType.OHLC,
                ohlc_data_handler,
                resolution=HistoricalPriceResolution.MINUTE
            )

            print("Subscribed to WebSocket data. Waiting for updates (Ctrl+C to stop)...")
            # Keep the main thread alive to receive WebSocket messages
            # WebSocket runs in a daemon thread.
            try:
                while api.ws_status == WebSocketStatus.CONNECTED or api.ws_status == WebSocketStatus.CONNECTING :
                    time.sleep(5) # Check status periodically
                    if not api._ws_subscriptions: # Check if all subscriptions were removed elsewhere
                        print("No active subscriptions, main loop will exit.")
                        break
            except KeyboardInterrupt:
                print("\nInterrupted by user.")
            finally:
                print("Stopping all WebSocket subscriptions...")
                api.stop_all_websocket_subscriptions()
                print("WebSocket stopped.")

    except CapitalComAPIError as e:
        print(f"API Error: {e}")
    except Exception as e:
        print(f"An unexpected error occurred: {e}", exc_info=True)
    finally:
        print("Program finished.")
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END
Core Functionality
Session Management

login(): Authenticates the client.

logout(): Terminates the session and clears tokens.

ping_server(): Checks server connectivity (basic ping).

Account Management

get_accounts(): Lists all user accounts.

get_active_account_details(): Details of the current account.

get_balance(): Balance of the active account.

switch_account(account_id): Change the active account.

get_account_preferences() / set_account_preferences(): Manage hedging and leverage.

Trading Operations

get_open_positions(): List of current open trades.

get_working_orders(): List of pending orders.

open_trade(...): Place a new trade.

close_trade(...): Close an existing trade.

update_trade(...): Modify SL/TP of an open trade.

Market & Historical Data

get_historical_prices(epic, resolution, ...): Fetch OHLC data.

get_market_details(epic | epics): Get instrument specifications.

get_transaction_history(...): Retrieve account activity.

get_closed_trades(...): Get history of closed positions.

Trade Sizing

calculate_trade_size_for_amount(...): Size based on investment amount and account leverage.

calculate_trade_size_for_margin(...): Size based on desired margin and instrument's margin factor.

WebSocket Streaming

subscribe_to_epic_data(epic, data_type, callback, ...): Subscribe to live market or OHLC data.

unsubscribe_from_epic_data(epic, data_type, ...): Stop receiving data for an epic.

stop_all_websocket_subscriptions(): Unsubscribe all and stop WebSocket.

ws_status (property): Get current WebSocketStatus.

Error Handling & Logging

The client raises CapitalComAPIError for API-specific errors. This exception may contain status_code and response_data.

Standard Python exceptions may also occur (e.g., network issues).

The library uses Python's logging module. To enable more verbose output:

import logging
logging.basicConfig(level=logging.INFO) # Or logging.DEBUG for very detailed logs
# To specifically target this library's logger:
# logging.getLogger('CapitalA').setLevel(logging.DEBUG)
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END
Disclaimer

Trading financial instruments carries a high level of risk and may not be suitable for all investors. Ensure you understand the risks involved and manage your capital wisely. Use this client, especially the trading functions, at your own risk. Always test thoroughly in a DEMO environment.

IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
IGNORE_WHEN_COPYING_END
