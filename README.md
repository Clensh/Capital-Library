Okay, this is a substantial Python library for interacting with the Capital.com API, covering both RESTful and WebSocket communications. Here's a detailed and professional documentation for it:

CapitalComAPI Python Client Documentation
Table of Contents

Overview

Features

Installation (Conceptual)

Core Components

Enums

CapitalComAPIError (Custom Exception)

The CapitalComAPI Class

Initialization (__init__)

Properties

Session Management

login()

logout()

ping_server()

Account Management

get_accounts()

get_active_account_details()

get_balance()

switch_account()

get_account_preferences()

set_account_preferences()

Trading Operations

get_open_positions()

get_working_orders()

open_trade()

close_trade()

update_trade()

Historical Data & Market Information

get_transaction_history()

get_closed_trades()

get_historical_prices()

get_market_details()

Trade Sizing Utilities

calculate_trade_size_for_amount()

calculate_trade_size_for_margin()

WebSocket Streaming

subscribe_to_epic_data()

unsubscribe_from_epic_data()

stop_all_websocket_subscriptions()

ws_status (Property)

Context Manager Support

Internal/Helper Methods (Brief Overview)

Usage Example

Error Handling and Logging

1. Overview

The CapitalComAPI Python client provides a comprehensive interface for interacting with the Capital.com trading platform. It supports both REST API endpoints for account management, trading, and historical data retrieval, as well as WebSocket connections for real-time market data and OHLC updates. The client handles session management, authentication, automatic re-login on session expiry for REST requests, and robust WebSocket connection management with reconnection logic and application-level pings.

2. Features

Environment Support: Works with both DEMO and LIVE Capital.com environments.

Authentication: Handles login, logout, and automatic renewal of session tokens (CST and X-SECURITY-TOKEN).

Account Management: Fetch account details, balances, switch between accounts, and manage account preferences.

Trading: Open, close, and update trades (positions). Fetch open positions and working orders.

Historical Data: Retrieve transaction history, closed trades, and historical price candles (OHLC).

Market Information: Get detailed market information for specific instruments (epics) or all navigable markets.

Trade Sizing Utilities: Helper functions to calculate trade sizes based on investment amount (using account leverage) or a specific margin amount (using instrument margin factor).

Real-time Data Streaming:

Subscribe to live market quotes (bid/ask).

Subscribe to OHLC (Open, High, Low, Close) candle updates for various resolutions.

Supports classic and Heikin-Ashi bar types for OHLC.

Robust WebSocket Handling:

Manages WebSocket connections in a separate thread.

Automatic reconnection with exponential backoff.

Application-level pings to keep the service session alive.

Handles multiple subscriptions efficiently.

Error Handling: Custom CapitalComAPIError for API-specific issues.

Logging: Integrated logging for insights into API interactions and WebSocket events.

Context Manager: Supports with statement for automatic login/logout.

3. Installation (Conceptual)

This client is provided as a single Python file (CapitalA.py). To use it:

Save the CapitalA.py file in your project directory or a location accessible by your Python interpreter.

Ensure you have the required dependencies installed:

requests: For HTTP requests.

websocket-client: For WebSocket communication.

You can install them using pip:

pip install requests websocket-client

4. Core Components
Enums

The client defines several enumerations to represent fixed sets of values used by the API:

Environment(Enum):

DEMO: For the demo trading environment.

LIVE: For the live trading environment.

TradeDirection(Enum):

BUY: Represents a buy order.

SELL: Represents a sell order.

HistoricalPriceResolution(Enum): Defines timeframes for historical price data.

MINUTE, MINUTE_5, MINUTE_10, MINUTE_15, MINUTE_30

HOUR, HOUR_2, HOUR_3, HOUR_4

DAY, WEEK, MONTH

WebSocketStatus(Enum): Tracks the state of the WebSocket connection.

DISCONNECTED, CONNECTING, CONNECTED, STOPPING

WebsocketDataType(Enum): Specifies the type of data to subscribe to via WebSocket.

MARKET: For live quote updates.

OHLC: For live OHLC candle updates.

OhlcBarType(Enum): Specifies the type of OHLC bar.

CLASSIC: Standard OHLC bars.

HEIKIN_ASHI: Heikin-Ashi bars.

CapitalComAPIError (Custom Exception)

CapitalComAPIError(Exception): A custom exception raised for errors originating from the Capital.com API or issues within the client's interaction with it.

Attributes:

message (str): The error message.

status_code (Optional[int]): The HTTP status code if the error is from an HTTP request.

response_data (Optional[Any]): The body of the HTTP response, if available.

5. The CapitalComAPI Class

This is the main class for interacting with the Capital.com API.

Initialization (__init__)
def __init__(self, api_key: str, identifier: str, password: str, environment: Environment = Environment.DEMO):
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END

Purpose: Initializes the API client.

Parameters:

api_key (str): Your Capital.com API key.

identifier (str): Your account identifier (usually email or account number).

password (str): Your account password.

environment (Environment): The trading environment to connect to. Defaults to Environment.DEMO.

Actions:

Sets up base URLs for REST and WebSocket APIs based on the chosen environment.

Initializes a requests.Session for HTTP communication with default headers.

Initializes internal state variables for authentication tokens, WebSocket management, etc.

Properties

ws_status (WebSocketStatus): (Read-only) Returns the current status of the WebSocket connection.

Session Management
login() -> bool

Purpose: Authenticates with the Capital.com API.

Actions:

Sends a POST request to the /session endpoint.

If successful, extracts and stores CST and X-SECURITY-TOKEN from the response headers.

Sets the active_account_id if returned in the response.

Returns: True if login is successful, False otherwise.

Notes: This method is crucial as most other API calls require active session tokens. The client attempts to re-login automatically if a 401 Unauthorized error is encountered on other requests.

logout() -> bool

Purpose: Logs out from the Capital.com server and clears local session data.

Actions:

Sends a DELETE request to the /session endpoint if session tokens are present.

Clears local CST, X-SECURITY-TOKEN, and active_account_id.

Stops any active WebSocket connections.

Returns: True. (Currently always returns True, even if server logout fails, as local cleanup is prioritized).

ping_server() -> bool

Purpose: Pings the server to check basic connectivity. This does not keep the trading session alive (use WebSocket application pings for that).

Actions: Sends a GET request to the /ping endpoint (no authentication required).

Returns: True if the server responds with "pong", False otherwise.

Account Management
get_accounts() -> List[Dict[str, Any]]

Purpose: Fetches a list of all accounts associated with the logged-in user.

Returns: A list of dictionaries, where each dictionary represents an account and its details. Returns an empty list on failure or if no accounts.

get_active_account_details() -> Optional[Dict[str, Any]]

Purpose: Retrieves details for the currently active account.

Actions:

If active_account_id is not set, it attempts to fetch all accounts and defaults to the first one.

Fetches all accounts and filters for the one matching active_account_id.

Returns: A dictionary containing details of the active account, or None if not found or an error occurs.

get_balance() -> Optional[float]

Purpose: Retrieves the balance of the active account.

Actions:

Calls get_active_account_details().

Extracts the balance value (prioritizes balance.balance, then balance.available).

Returns: The account balance as a float, or None if unavailable.

switch_account(account_id: str) -> bool

Purpose: Switches the active trading account for the current session.

Parameters:

account_id (str): The ID of the account to switch to.

Actions:

Sends a PUT request to /session with the accountId.

Confirms the switch by fetching session details again.

Updates self.active_account_id.

Important: Stops all current WebSocket subscriptions, as they may become invalid for the new account. Users need to re-subscribe.

Returns: True if the switch is successful and confirmed, False otherwise.

get_account_preferences() -> Optional[Dict[str, Any]]

Purpose: Fetches current account preferences, such as hedging mode and leverage settings.

Returns: A dictionary containing account preferences, or None on error. Includes hedgingMode and leverages (which can be a dict of instrument type to leverage details or a list).

set_account_preferences(hedging_enabled: Optional[bool] = None, leverages: Optional[Dict[str, int]] = None) -> bool

Purpose: Sets account preferences.

Parameters:

hedging_enabled (Optional[bool]): Set to True to enable hedging, False to disable. If None, not changed.

leverages (Optional[Dict[str, int]]): A dictionary to set leverages for specific instrument types (e.g., {"CURRENCIES": 30, "SHARES": 5}). If None, not changed.

Actions: Sends a PUT request to /account/preferences.

Returns: True if the request was sent successfully, False on API error. (Note: Success of request doesn't guarantee the change was applied by the backend as expected, user should verify).

Trading Operations
get_open_positions() -> List[Dict[str, Any]]

Purpose: Fetches a list of all currently open positions for the active account.

Returns: A list of dictionaries, each representing an open position. Returns an empty list on failure or if no open positions.

get_working_orders() -> List[Dict[str, Any]]

Purpose: Fetches a list of all active (working) orders for the active account.

Returns: A list of dictionaries, each representing a working order. Returns an empty list on failure or if no working orders.

open_trade(...) -> Optional[Dict[str, Any]]
def open_trade(self, epic: str, direction: TradeDirection, size: float,
                 guaranteed_stop: bool = False, stop_level: Optional[float] = None,
                 stop_distance: Optional[float] = None, profit_level: Optional[float] = None,
                 profit_distance: Optional[float] = None, trailing_stop: bool = False,
                 trailing_stop_distance: Optional[float] = None,
                 force_open: bool = True) -> Optional[Dict[str, Any]]:
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END

Purpose: Opens a new trade (position).

Parameters:

epic (str): The market epic (instrument identifier).

direction (TradeDirection): TradeDirection.BUY or TradeDirection.SELL.

size (float): The size of the trade.

guaranteed_stop (bool): Whether to use a guaranteed stop.

stop_level (Optional[float]): Absolute price level for stop loss.

stop_distance (Optional[float]): Distance from current price for stop loss.

profit_level (Optional[float]): Absolute price level for take profit.

profit_distance (Optional[float]): Distance from current price for take profit.

trailing_stop (bool): Whether to use a trailing stop.

trailing_stop_distance (Optional[float]): Distance for the trailing stop.

force_open (bool): If True, forces the position to open even if it would partially net off an existing one.

Returns: A dictionary containing the API response (often including a dealReference), or None on failure.

close_trade(...) -> Optional[Dict[str, Any]]
def close_trade(self, deal_id: str, direction: Optional[TradeDirection] = None,
                  size: Optional[float] = None, order_type: str = "MARKET",
                  level: Optional[float] = None,
                  time_in_force: str = "GOOD_TILL_CANCELLED") -> Optional[Dict[str, Any]]:
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END

Purpose: Closes an existing OTC trade or part of it.

Parameters:

deal_id (str): The ID of the position to close.

direction (Optional[TradeDirection]): Required if partially closing. Should be opposite to the opening direction.

size (Optional[float]): The size to close. If None, closes the entire position.

order_type (str): Type of order for closing (e.g., "MARKET", "LIMIT"). Defaults to "MARKET".

level (Optional[float]): Price level for non-market close orders.

time_in_force (str): Time in force for non-market close orders (e.g., "GOOD_TILL_CANCELLED").

Returns: A dictionary containing the API response, or None on failure.

Note: This method uses DELETE /positions (plural) with a request body. Capital.com's standard documentation might show DELETE /positions/{dealId} (singular) for closing a specific position. This implementation might target a specific bulk or OTC closing mechanism.

update_trade(...) -> Optional[Dict[str, Any]]
def update_trade(self, deal_id: str, stop_level: Optional[float] = None,
                   profit_level: Optional[float] = None,
                   trailing_stop: Optional[bool] = None,
                   trailing_stop_distance: Optional[float] = None,
                   guaranteed_stop: Optional[bool] = None) -> Optional[Dict[str, Any]]:
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END

Purpose: Updates stop loss or take profit orders for an existing position.

Parameters:

deal_id (str): The ID of the position to update.

stop_level (Optional[float]): New stop loss price level.

profit_level (Optional[float]): New take profit price level.

trailing_stop (Optional[bool]): Enable/disable trailing stop.

trailing_stop_distance (Optional[float]): New distance for the trailing stop.

guaranteed_stop (Optional[bool]): Enable/disable guaranteed stop.

Returns: A dictionary containing the API response, or None on failure or if no parameters are provided.

Historical Data & Market Information
get_transaction_history(...) -> List[Dict[str, Any]]
def get_transaction_history(self, transaction_type: Optional[str] = None, from_date: Optional[str] = None,
                            to_date: Optional[str] = None, detailed: bool = False,
                            last_period_seconds: Optional[int] = None) -> List[Dict[str, Any]]:
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END

Purpose: Retrieves transaction history.

Parameters:

transaction_type (Optional[str]): Filter by transaction type (e.g., "TRADE", "DEPOSIT").

from_date (Optional[str]): Start date for history (format: "YYYY-MM-DD'T'HH:MM:SS" or "YYYY-MM-DD").

to_date (Optional[str]): End date for history (same format as from_date).

detailed (bool): If True, returns detailed transaction information.

last_period_seconds (Optional[int]): Get transactions for the last N seconds.

Returns: A list of dictionaries, each representing a transaction. Returns an empty list on failure.

get_closed_trades(...) -> List[Dict[str, Any]]
def get_closed_trades(self, from_date: Optional[str] = None, to_date: Optional[str] = None, last_period_seconds: Optional[int] = None) -> List[Dict[str, Any]]:
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END

Purpose: A convenience method to get history of closed trades.

Actions: Calls get_transaction_history with transaction_type="TRADE" and detailed=True.

Parameters:

from_date (Optional[str]): Start date.

to_date (Optional[str]): End date.

last_period_seconds (Optional[int]): Get trades for the last N seconds.

Returns: A list of dictionaries, each representing a closed trade.

get_historical_prices(...) -> Optional[Dict[str, Any]]
def get_historical_prices(self, epic: str, resolution: HistoricalPriceResolution,
                            num_points: Optional[int] = None, start_date: Optional[str] = None,
                            end_date: Optional[str] = None) -> Optional[Dict[str, Any]]:
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END

Purpose: Fetches historical OHLC (candlestick) data for a market.

Parameters:

epic (str): The market epic.

resolution (HistoricalPriceResolution): The timeframe resolution.

num_points (Optional[int]): Number of data points to retrieve (max). Defaults to 100 if no date range specified.

start_date (Optional[str]): Start date (format: "YYYY-MM-DD'T'HH:MM:SS" or "YYYY-MM-DD").

end_date (Optional[str]): End date (same format).

Returns: A dictionary containing historical price data (usually under a "prices" key with OHLC and snapshot data), or None on failure.

get_market_details(epic: Optional[str] = None, epics: Optional[List[str]] = None) -> Optional[Dict[str, Any]]

Purpose: Fetches details for one or more markets, or all navigable markets.

Parameters:

epic (Optional[str]): Fetch details for a single market epic.

epics (Optional[List[str]]): Fetch details for a list of market epics.

If neither epic nor epics is provided, fetches details for all navigable markets.

Returns:

If epic is provided: A dictionary with details for that market.

If epics is provided: A dictionary, often with a "markets" key containing a list of market details.

If neither: A dictionary with all navigable markets.

Returns None on failure.

Raises: ValueError if both epic and epics are provided.

Trade Sizing Utilities
calculate_trade_size_for_amount(epic: str, investment_amount_in_quote_currency: float, direction: TradeDirection) -> float

Purpose: Calculates the trade size (number of units/contracts) for a given investment amount (used as margin), taking into account the account-level leverage for the instrument type.

Parameters:

epic (str): The market epic.

investment_amount_in_quote_currency (float): The amount of money in the quote currency of the instrument that you want to use as margin for this trade.

direction (TradeDirection): BUY or SELL, to determine the price (offer/bid).

Actions:

Fetches market details for the epic (price, dealing rules, instrument type).

Fetches account preferences to get the current leverage for the instrument's type.

Calculates raw size: (investment_amount * leverage) / price.

Includes a 1.22x multiplier on investment_amount, possibly for a buffer or specific internal calculation. This should be verified against Capital.com's exact margin calculation for account-level leverage.

Adjusts the raw size to meet minimum deal size and step requirements.

Returns: The calculated trade size (float), rounded down to the nearest valid step. Returns 0.0 if the amount is too small or an error occurs.

Raises: CapitalComAPIError or ValueError if market details, preferences, or dealing rules are missing or invalid.

calculate_trade_size_for_margin(epic: str, margin_amount_in_quote_currency: float, direction: TradeDirection) -> float

Purpose: Calculates the trade size based on a specified margin amount, using the instrument's specific marginFactor.

Parameters:

epic (str): The market epic.

margin_amount_in_quote_currency (float): The desired margin for the trade in the quote currency.

direction (TradeDirection): BUY or SELL.

Actions:

Fetches market details (price, dealing rules, instrument's marginFactor).

Converts marginFactor (e.g., 10 for 10%) to a decimal (e.g., 0.10). Handles "PERCENTAGE" and "ABSOLUTE" marginFactorUnit.

Calculates notional value: margin_amount / margin_factor_decimal.

Calculates raw size: notional_value / current_price.

Adjusts the raw size to meet minimum deal size and step requirements.

Returns: The calculated trade size (float), rounded down. Returns 0.0 if the margin amount is too small or an error occurs.

Raises: CapitalComAPIError or ValueError for issues with data fetching or validity.

WebSocket Streaming

The WebSocket functionality allows for real-time data feeds. It runs in a separate daemon thread, handles connections, disconnections, reconnections, and message processing.

Key Internal Features:

_ws_run(): Main loop for the WebSocket thread, manages connection and reconnection.

_ws_on_open(): Called on successful connection; resubscribes to any active subscriptions.

_ws_on_message(): Processes incoming messages, routing data to appropriate callbacks.

_ws_on_error(): Handles low-level WebSocket errors.

_ws_on_close(): Handles connection closure and triggers reconnection logic if needed.

_ws_application_ping_run(): Runs in a separate thread to send application-level JSON pings to keep the service session (and thus the WebSocket connection) alive. The interval is APP_PING_INTERVAL_SECONDS (9 minutes).

subscribe_to_epic_data(...)
def subscribe_to_epic_data(self,
                           epic: str,
                           data_type: WebsocketDataType,
                           callback: Callable[[Dict[str, Any]], None],
                           resolution: Optional[HistoricalPriceResolution] = None,
                           bar_type: OhlcBarType = OhlcBarType.CLASSIC):
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END

Purpose: Subscribes to real-time data for a specific epic.

Parameters:

epic (str): The market epic.

data_type (WebsocketDataType): WebsocketDataType.MARKET (for quotes) or WebsocketDataType.OHLC (for candles).

callback (Callable[[Dict[str, Any]], None]): A function that will be called with the data payload when a new update is received. The payload is a dictionary.

resolution (Optional[HistoricalPriceResolution]): Required if data_type is OHLC. Specifies the candle timeframe.

bar_type (OhlcBarType): Type of OHLC bar. Defaults to CLASSIC. Used if data_type is OHLC.

Actions:

Validates parameters.

Stores subscription details (epic, callback, type, etc.) internally.

Starts the WebSocket thread if not already running (requires prior login for tokens).

If WebSocket is connected, sends the subscription message. If connecting, the _ws_on_open handler will send it.

Raises: ValueError for invalid parameters (e.g., missing resolution for OHLC).

Notes: Requires login() to have been called successfully first, as WebSocket authentication uses CST and X-SECURITY-TOKEN.

unsubscribe_from_epic_data(...)
def unsubscribe_from_epic_data(self,
                               epic: str,
                               data_type: WebsocketDataType,
                               resolution: Optional[HistoricalPriceResolution] = None,
                               bar_type: OhlcBarType = OhlcBarType.CLASSIC):
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END

Purpose: Unsubscribes from real-time data for a specific epic.

Parameters:

epic (str): The market epic.

data_type (WebsocketDataType): The type of data previously subscribed to.

resolution (Optional[HistoricalPriceResolution]): Required if data_type was OHLC.

bar_type (OhlcBarType): Required if data_type was OHLC, to match the original subscription.

Actions:

Removes the subscription from internal tracking.

If the WebSocket is connected, sends an unsubscribe message to the server.

If no active subscriptions remain, stops the WebSocket thread.

Raises: ValueError for invalid parameters.

stop_all_websocket_subscriptions()

Purpose: Unsubscribes from all active WebSocket data streams and stops the WebSocket thread.

Actions:

Clears all internal subscription tracking.

If connected, attempts to send unsubscribe messages to the server for all previously active subscriptions.

Stops the WebSocket communication thread and the application ping thread.

ws_status (Property)
@property
def ws_status(self) -> WebSocketStatus:
    """Current status of the WebSocket connection."""
    return self._ws_status
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END

Purpose: Returns the current status of the WebSocket connection.

Returns: A WebSocketStatus enum member.

Context Manager Support

The CapitalComAPI class supports Python's context management protocol.

def __enter__(self):
    if not self.login():
        raise CapitalComAPIError("Failed to login upon entering context manager.")
    return self

def __exit__(self, exc_type, exc_val, exc_tb):
    logger.info("Exiting context manager, ensuring logout and WebSocket shutdown...")
    self.logout()
    logger.info("Context manager exited.")
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END

Purpose: Allows for easy setup and teardown.

__enter__: Automatically calls login(). If login fails, it raises CapitalComAPIError.

__exit__: Automatically calls logout() to ensure the session is closed and WebSockets are shut down.

6. Internal/Helper Methods (Brief Overview)

_update_auth_tokens(): Updates cst and x_security_token from HTTP response headers.

_request(): Core method for making all REST API requests. Handles adding auth headers, automatic re-login on 401, error handling, and JSON parsing.

_get_ws_url(): Constructs the WebSocket URL with necessary authentication tokens.

_ws_... methods: A suite of methods prefixed with _ws_ manage the WebSocket lifecycle (connection, messages, errors, closing, pings, thread management). See WebSocket Streaming for their roles.

_start_websocket_thread(): Starts the main WebSocket processing thread.

_stop_websocket_thread(): Stops the main WebSocket processing thread and the application ping thread.

7. Usage Example
import time
import logging
from CapitalA import (
    CapitalComAPI, Environment, TradeDirection,
    HistoricalPriceResolution, WebsocketDataType, OhlcBarType, CapitalComAPIError
)

# Configure logging for more detailed output if needed
# logging.getLogger('CapitalA').setLevel(logging.DEBUG) # For CapitalA module
# logging.getLogger().setLevel(logging.INFO) # For root logger

API_KEY = "YOUR_API_KEY"
IDENTIFIER = "YOUR_EMAIL_OR_ACCOUNT_ID"
PASSWORD = "YOUR_PASSWORD"
ENVIRONMENT = Environment.DEMO  # or Environment.LIVE

# --- WebSocket Callbacks ---
def market_data_handler(data):
    # Example: {'epic': 'EURUSD', 'bid': 1.0850, 'offer': 1.0852, 'updateTime': 1678886400000}
    print(f"MARKET DATA: {data['epic']} - Bid: {data.get('bid')}, Offer: {data.get('offer')}")

def ohlc_data_handler(data):
    # Example: {'epic': 'US500', 'resolution': 'MINUTE', 'type': 'classic',
    #           'c': 123.45, 'h': 123.50, 'l': 123.40, 'o': 123.42, 't': 1678886460000}
    print(f"OHLC DATA ({data.get('type', 'classic')}): {data['epic']} ({data['resolution']}) "
          f"- O: {data.get('o')} H: {data.get('h')} L: {data.get('l')} C: {data.get('c')}")

if __name__ == "__main__":
    try:
        # Using the context manager for automatic login/logout
        with CapitalComAPI(api_key=API_KEY, identifier=IDENTIFIER, password=PASSWORD, environment=ENVIRONMENT) as api:
            print("Login successful via context manager.")

            # --- Account Info ---
            balance = api.get_balance()
            if balance is not None:
                print(f"Current Account Balance: {balance}")

            accounts = api.get_accounts()
            if accounts:
                print(f"Found {len(accounts)} account(s). Active Account ID: {api.active_account_id}")
                # print(f"First account details: {accounts[0]}")

            # --- Market Details ---
            market_details = api.get_market_details(epic="EURUSD") # Example epic
            if market_details:
                print(f"Market details for EURUSD: Min deal size: {market_details.get('dealingRules', {}).get('minDealSize', {}).get('value')}")

            # --- Historical Prices ---
            # historical_prices = api.get_historical_prices("IX.D.SPTRD.IFS.IP", HistoricalPriceResolution.HOUR, num_points=10)
            # if historical_prices and historical_prices.get("prices"):
            #     print(f"Last 10 hourly prices for S&P 500: {len(historical_prices['prices'])} candles received.")

            # --- Calculate Trade Size ---
            try:
                # Example: Calculate size for $100 margin on EURUSD BUY using instrument margin factor
                size_for_margin = api.calculate_trade_size_for_margin("EURUSD", 100.0, TradeDirection.BUY)
                print(f"Calculated size for $100 margin (instrument factor) on EURUSD BUY: {size_for_margin}")

                # Example: Calculate size for $100 investment on EURUSD BUY using account leverage
                # Note: This might differ from above based on account leverage vs. instrument margin factor
                size_for_amount = api.calculate_trade_size_for_amount("EURUSD", 100.0, TradeDirection.BUY)
                print(f"Calculated size for $100 investment (account leverage) on EURUSD BUY: {size_for_amount}")
            except (CapitalComAPIError, ValueError) as e:
                print(f"Error calculating trade size: {e}")


            # --- WebSocket Subscriptions ---
            print("\nSubscribing to WebSocket data...")
            # Subscribe to market data (quotes) for EURUSD
            api.subscribe_to_epic_data("EURUSD", WebsocketDataType.MARKET, market_data_handler)

            # Subscribe to 1-minute OHLC classic candles for US500 (example CFD for S&P 500)
            # Check your broker's exact epic for S&P 500
            api.subscribe_to_epic_data(
                "US500",  # Example epic, replace with actual
                WebsocketDataType.OHLC,
                ohlc_data_handler,
                resolution=HistoricalPriceResolution.MINUTE,
                bar_type=OhlcBarType.CLASSIC
            )

            print(f"WebSocket status: {api.ws_status}. Waiting for data... (Ctrl+C to stop)")
            try:
                # Keep the main thread alive to receive WebSocket messages
                # The WebSocket threads are daemons, so main thread needs to live
                while True:
                    time.sleep(1)
                    if api.ws_status == WebSocketStatus.DISCONNECTED and not api._ws_subscriptions:
                        print("WebSocket disconnected and no subscriptions, exiting main loop.")
                        break
            except KeyboardInterrupt:
                print("\nKeyboardInterrupt received.")
            finally:
                print("Unsubscribing and stopping WebSocket...")
                # Unsubscribe specifically (or use stop_all_websocket_subscriptions)
                # api.unsubscribe_from_epic_data("EURUSD", WebsocketDataType.MARKET)
                # api.unsubscribe_from_epic_data("US500", WebsocketDataType.OHLC, resolution=HistoricalPriceResolution.MINUTE)
                # Or, more simply:
                api.stop_all_websocket_subscriptions()
                print(f"WebSocket status after stopping: {api.ws_status}")

            # --- Example: Open and Close a Trade (Use with caution, especially on LIVE) ---
            if ENVIRONMENT == Environment.DEMO: # Safety check for demo
                # print("\nAttempting to open a demo trade...")
                # trade_response = api.open_trade(
                #     epic="EURUSD",
                #     direction=TradeDirection.BUY,
                #     size=0.01, # Min size, check market details
                #     stop_distance=20, # 20 pips
                #     profit_distance=40 # 40 pips
                # )
                # if trade_response and trade_response.get("dealReference"):
                #     deal_ref = trade_response["dealReference"]
                #     print(f"Trade opened successfully. Deal Reference: {deal_ref}")
                #     time.sleep(5) # Wait a bit

                #     # Find the dealId from open positions (dealReference is temporary)
                #     open_positions = api.get_open_positions()
                #     deal_id_to_close = None
                #     for pos in open_positions:
                #         # Logic to identify your trade, e.g. by epic and original dealReference if available
                #         # Or if it's the only EURUSD trade, etc.
                #         # For this example, assume we find it.
                #         # This part needs robust logic to identify the correct position.
                #         if pos.get("instrument", {}).get("epic") == "EURUSD": # Simplified
                #              deal_id_to_close = pos.get("position", {}).get("dealId")
                #              break

                #     if deal_id_to_close:
                #         print(f"Closing trade with Deal ID: {deal_id_to_close}")
                #         close_response = api.close_trade(deal_id=deal_id_to_close, size=0.01, direction=TradeDirection.SELL)
                #         if close_response:
                #             print(f"Close trade response: {close_response}")
                #         else:
                #             print("Failed to close trade.")
                #     else:
                #         print("Could not find the opened trade to close it by dealId.")
                # else:
                #     print(f"Failed to open trade. Response: {trade_response}")
                pass # Commented out trade operations for safety

    except CapitalComAPIError as e:
        print(f"API Error: {e}")
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
    finally:
        print("Program finished.")
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END
8. Error Handling and Logging

Error Handling:

The client raises CapitalComAPIError for issues related to API calls (e.g., authentication failure, bad request, server errors). This exception contains the status_code and response_data where applicable.

Standard Python exceptions (requests.exceptions.RequestException, ValueError, json.JSONDecodeError) might also be raised for network issues, invalid input, or malformed responses.

WebSocket errors are logged, and connection issues trigger reconnection attempts. Critical errors (like authentication failure on WebSocket) may lead to the WebSocket stopping.

Logging:

The client uses Python's logging module.

By default, INFO level messages are logged, providing a good overview of operations.

For more detailed debugging, especially for REST requests/responses and WebSocket messages, set the logger level to DEBUG:

import logging
logging.getLogger('CapitalA').setLevel(logging.DEBUG) # For this specific client
# Or for all loggers:
# logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(levelname)s - %(module)s - %(funcName)s - %(message)s')
IGNORE_WHEN_COPYING_START
content_copy
download
Use code with caution.
Python
IGNORE_WHEN_COPYING_END

Log messages include timestamps, log level, module, function name, and the message.

This detailed documentation should provide a solid understanding of the CapitalComAPI client and how to use its various features. Remember to replace placeholder credentials with your actual API key, identifier, and password, and always test thoroughly in a DEMO environment before using on a LIVE account.
