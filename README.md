# LuaStore
[![Status](https://img.shields.io/badge/status-active-success.svg)]()
[![Version](https://img.shields.io/badge/version-0.5-blue.svg)](https://github.com/your-username/your-repo)
[![Language](https://img.shields.io/badge/language-Luvit%20/%20Lua-orange.svg)](https://luvit.io/)

A simple, self-hosted, file-based key-value datastore server built with Luvit. It provides a lightweight HTTP API for basic data persistence, inspired by Roblox's DataStore service. It's ideal for small projects, game development prototypes, or as a local alternative to cloud-based data stores.

## Features

- **Simple Key-Value Storage:** Store and retrieve data using simple keys.
- **HTTP API:** Easy-to-use API with `get`, `set`, `increment`, and `remove` operations.
- **File-Based Persistence:** The entire database is stored in a single `datastore.json` file, making it portable and easy to manage.
- **In-Memory Caching:** All data is held in memory for fast access, with periodic saves to disk.
- **Auto-Saving:** Automatically saves the database to the file every 60 seconds to prevent data loss.
- **Secure Access:** Protects your server with a required secret token for all requests.
- **Multi-Tenancy:** Supports data isolation for different projects using an `X-Place-Id` header.

## Getting Started

### Prerequisites

You must have [Luvit](https://luvit.io/install.html) installed on your system.

### Installation

1.  **Clone the repository or save the code:**
    ```bash
    git clone https://github.com/your-username/your-repo.git
    cd your-repo
    ```
    Or, simply save the code as `server.lua`.

2.  **Configure the Server:**
    Open `server.lua` in a text editor. **It is critical that you change the `SECRET_TOKEN`**.
    ```lua
    -- --- Configuration ---
    local PORT = 2281
    local DB_FILE_PATH = 'datastore.json'

    -- !! IMPORTANT !!
    -- This token is the password for your server.
    -- The request MUST use the same token to get access.
    local SECRET_TOKEN = "change-this-to-a-very-long-and-secret-password"
    ```

3.  **Run the Server:**
    Execute the following command in your terminal from the file's directory:
    ```bash
    luvit server.lua
    ```

If successful, you will see output like this:
```
Starting LocalDataStore server...
Database file not found. A new one will be created.
Server listening on http://127.0.0.1:2281
 Auto-saving every 60 seconds.
```

## API Usage

All requests require two headers:
- `Authorization`: Your secret token.
- `X-Place-Id`: A unique identifier for your game or application to isolate data.

The URL structure is `/operation/dataStoreName/key`.

---

### Set Value

Creates or overwrites a value for a given key. The raw request body is saved as the value.

- **Method:** `POST`
- **Endpoint:** `/set/{dataStoreName}/{key}`
- **Example:** Save a JSON string representing player data.
  ```bash
  curl -X POST \
    http://127.0.0.1:2281/set/PlayerData/Player_123 \
    -H "Authorization: your-secret-token" \
    -H "X-Place-Id: 98765" \
    -H "Content-Type: application/json" \
    -d '{"level": 10, "gold": 500, "inventory": ["sword", "shield"]}'
  ```
- **Success Response:**
  ```json
  { "success": true }
  ```

---

### Get Value

Retrieves the value associated with a key.

- **Method:** `GET`
- **Endpoint:** `/get/{dataStoreName}/{key}`
- **Example:** Get the data for `Player_123`.
  ```bash
  curl -X GET \
    http://127.0.0.1:2281/get/PlayerData/Player_123 \
    -H "Authorization: your-secret-token" \
    -H "X-Place-Id: 98765"
  ```
- **Success Response:**
  ```json
  {
    "success": true,
    "value": "{\"level\": 10, \"gold\": 500, \"inventory\": [\"sword\", \"shield\"]}"
  }
  ```

---

### Increment Value

Atomically increments a numeric value. If the key does not exist, it's assumed to be `0` before incrementing.

- **Method:** `POST`
- **Endpoint:** `/increment/{dataStoreName}/{key}`
- **Body:** A JSON object with a `delta` field. If omitted, the value is incremented by `1`.
- **Example:** Add 10 to a player's score.
  ```bash
  curl -X POST \
    http://127.0.0.1:2281/increment/PlayerStats/Score \
    -H "Authorization: your-secret-token" \
    -H "X-Place-Id: 98765" \
    -H "Content-Type: application/json" \
    -d '{"delta": 10}'
  ```
- **Success Response:** (Assuming the previous score was 90)
  ```json
  {
    "success": true,
    "value": 100
  }
  ```

---

### Remove Value

Deletes a key-value pair from the datastore.

- **Method:** `POST`
- **Endpoint:** `/remove/{dataStoreName}/{key}`
- **Example:** Delete all data for `Player_123`.
  ```bash
  curl -X POST \
    http://127.0.0.1:2281/remove/PlayerData/Player_123 \
    -H "Authorization: your-secret-token" \
    -H "X-Place-Id: 98765"
  ```
- **Success Response:**
  ```json
  { "success": true }
  ```

## Security Warning

⚠️ **Change the default `SECRET_TOKEN` immediately.**

The default token (`"ae"`) is insecure and public. Failure to change it will allow anyone with access to the server's address to read and write your data. Choose a long, random, and unpredictable string for your token.

## How It Works

The server loads the entire `datastore.json` file into memory on startup. All read and write operations are performed on this in-memory object for maximum speed. To ensure data persistence, a timer runs every 60 seconds to serialize the in-memory database back to the `datastore.json` file.

If the `datastore.json` file is corrupted on startup, it will be backed up as `datastore.json.bak` and the server will start with a fresh, empty database.

## Credits

- **Authors:** AlexMiles, foxi22815

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
