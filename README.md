# Order Book Trading Platform Backend

A simple but powerful Express.js backend that simulates the core functionality of a financial trading platform's order book. This platform allows users to place limit and market orders, view the current order book depth, and check their balances.

## Overview

This backend is built to be in-memory, meaning the order book and user balances will reset if the server restarts. It's an excellent tool for understanding the mechanics of how a trade matching engine works.

## Features

- **Limit Orders**: Place orders that execute only at a specific price or better
- **Market Orders**: Place orders that execute immediately at the best available market price
- **Order Book Depth**: View a consolidated list of all open buy and sell orders at different price levels
- **Quote**: Get the current best bid (highest buy price) and best ask (lowest sell price)
- **Balance Management**: Basic tracking of user balances for the traded asset (GOOGLE) and cash (USD)

## API Endpoints

### 1. Place a Limit Order

Places an order on the book that will only be filled at the specified price or a better one. If there are matching orders on the other side of the book that meet the price criteria, the order will be partially or fully filled immediately. Otherwise, the remaining quantity is placed on the order book.

- **URL**: `/order`
- **Method**: `POST`
- **Body**:
```json
{
  "side": "bid",
  "price": 150.50,
  "quantity": 10,
  "userId": "1"
}
```

**Parameters**:
- `side`: "bid" (for a buy order) or "ask" (for a sell order)
- `price`: The limit price for the order
- `quantity`: The number of shares to trade
- `userId`: The ID of the user placing the order

**Success Response**:
```json
{
  "filledQuantity": 10
}
```
This indicates how much of the order was filled immediately.

### 2. Place a Market Order

Places an order that executes immediately against the best available prices on the order book. This guarantees execution but not the price.

- **URL**: `/order/market`
- **Method**: `POST`
- **Body**:
```json
{
  "side": "ask",
  "quantity": 5,
  "userId": "2"
}
```

**Parameters**:
- `side`: "bid" (for a buy order) or "ask" (for a sell order)
- `quantity`: The number of shares to trade
- `userId`: The ID of the user placing the order

**Success Response**:
```json
{
  "message": "Market order filled successfully.",
  "filledQuantity": 5
}
```

**Error Response** (If no orders are available to fill):
```json
{
  "message": "No orders available to fill this market order."
}
```

### 3. Get Order Book Depth

Retrieves the current state of the order book, showing the total quantity of shares available at each price level for both bids and asks.

- **URL**: `/depth`
- **Method**: `GET`

**Success Response**:
```json
{
  "depth": {
    "150.5": {
      "type": "bid",
      "quantity": 10
    },
    "151.0": {
      "type": "ask",
      "quantity": 8
    },
    "151.2": {
      "type": "ask",
      "quantity": 12
    }
  }
}
```

### 4. Get Best Bid/Ask (Quote)

Retrieves the highest bid and the lowest ask from the order book. This represents the best current prices available.

- **URL**: `/quote`
- **Method**: `GET`

**Success Response**:
```json
{
  "ticker": "GOOGLE",
  "bestBid": 150.5,
  "bestAsk": 151.0
}
```
*Note: If there are no bids or asks, the respective value will be null.*

### 5. Get User Balance

Retrieves the asset and cash balance for a specific user.

- **URL**: `/balance/:userId`
- **Method**: `GET`
- **Example URL**: `/balance/1`

**Success Response**:
```json
{
  "balances": {
    "GOOGLE": 10,
    "USD": 50000
  }
}
```

## How the Matching Engine Works

The core of the platform is the `fillOrders` function, which acts as the trade matching engine.

### Order Sorting

- **Bids (Buy Orders)** are sorted from the lowest price to the highest price. The best bid is at the end of the array.
- **Asks (Sell Orders)** are sorted from the highest price to the lowest price. The best ask is at the end of the array.

### Matching Logic

- When a buy order (bid) comes in, the engine checks it against the asks, starting with the lowest-priced ask (the best one).
- When a sell order (ask) comes in, the engine checks it against the bids, starting with the highest-priced bid (the best one).

### Execution

- The engine will continue to fill the incoming order against orders in the book as long as the price criteria are met.
- For each trade that occurs, the `flipBalance` function is called to debit and credit the GOOGLE and USD balances of the two users involved in the trade.
- If any part of an order is not filled, it is added to the order book.

This simple but effective logic ensures that trades are executed at the best possible prices for both buyers and sellers in a first-come, first-served manner at each price level.

## Getting Started

1. Install dependencies:
```bash
npm install
```

2. Start the server:
```bash
npm start
```

3. The server will be running on the configured port, ready to accept API requests.

## Note

This is an in-memory implementation, so all data will be lost when the server restarts. This makes it perfect for testing and learning about order book mechanics without the complexity of persistent storage.
