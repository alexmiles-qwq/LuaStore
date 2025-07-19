# LuaStore

[![Status](https://img.shields.io/badge/status-active-success.svg)](https://github.com/alexmiles-qwq/LuaStore/)
[![Version](https://img.shields.io/badge/version-0.6-blue.svg)](https://github.com/alexmiles-qwq/LuaStore/)
[![Language](https://img.shields.io/badge/language-Luvit%20/%20Lua-orange.svg)](https://luvit.io/)

A simple, secure, self-hosted, file-based key-value datastore server built with Luvit. It provides a lightweight HTTP API for basic data persistence, inspired by Roblox's DataStore service. It's ideal for small projects, game development prototypes, or as a local alternative to cloud-based data stores.

## Features

- **Simple Key-Value Storage:** Store and retrieve data using simple keys, organized by DataStore names.
- **Secure by Default:**
    - **Mandatory Strong Token:** The server will not start if a weak or default secret token is used.
    - **Brute-Force Protection:** Automatically blocks IP addresses after multiple failed login attempts.
- **HTTP API:** Easy-to-use API with `get`, `set`, `increment`, and `remove` operations.
- **File-Based Persistence:** The entire database is stored in a single `datastore.json` file, making it portable and easy to manage.
- **In-Memory Caching:** All data is held in memory for fast access, with periodic saves to disk.
- **Auto-Saving:** Automatically saves the database to the file every 60 seconds to prevent data loss.

## Getting Started

### Prerequisites

You must have [Luvit](https://luvit.io/install.html) installed on your system.

### Installation

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/alexmiles-qwq/LuaStore.git
    cd LuaStore
    ```

2.  **Configure the Server:**
    Open `server.lua` in a text editor. **This is the most important step.** You must change the `SECRET_TOKEN` to a long and secure password.
    ```lua
    -- !! IMPORTANT !!
    -- This token is the password for your server. CHANGE IT!
    -- The server WILL NOT START if you use a weak token.
    local SECRET_TOKEN = "change-this-to-a-very-long-and-secure-password"
    ```
    The server will refuse to start if the token is the default (`"ae"`) or shorter than 16 characters.

3.  **Run the Server:**
    Execute the following command in your terminal from the file's directory:
    ```bash
    luvit server.lua
    ```

If successful, you will see output like this:
```
Starting LocalDataStore server...
Database loaded successfully from datastore.json
Server listening on http://127.0.0.1:2281
Auto-saving every 60 seconds.
Rate limiting active: 5 failed attempts per IP will result in a 300-second ban.
```

## API Usage

All API requests require the `Authorization` header containing your secret token. The URL structure is `/operation/dataStoreName/key`.

---

### Set Value

Creates or overwrites a value for a given key. The raw request body is saved as the value.

- **Method:** `POST`
- **Endpoint:** `/set/{dataStoreName}/{key}`
- **Example:** Save a JSON string representing player data.
  ```bash
  curl -X POST \
    http://127.0.0.1:2281/set/PlayerData/Player_123 \
    -H "Authorization: your-long-and-secret-token" \
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
    -H "Authorization: your-long-and-secret-token"
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
    -H "Authorization: your-long-and-secret-token" \
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
    -H "Authorization: your-long-and-secret-token"
  ```
- **Success Response:**
  ```json
  { "success": true }
  ```

## Security

This server includes several security features, but you should be aware of how they work.

### 1. Mandatory Strong Token

You **must** change the `SECRET_TOKEN` in `server.lua`. The server is designed to be "secure by default" and will not start with a known weak token.
- **Requirement:** Must not be the default (`"ae"`).
- **Recommendation:** At least 16+ random characters.

### 2. Rate Limiting (Brute-Force Protection)

To prevent attackers from guessing your token, the server automatically tracks and blocks IPs that fail authentication too many times.
- **Default Limit:** 5 failed attempts.
- **Default Ban Time:** 300 seconds (5 minutes).
- These values can be configured at the top of `server.lua`.

### 3. Recommendation: Use a Reverse Proxy (HTTPS)

For production or public-facing use, it is **highly recommended** to run this server behind a reverse proxy like [Nginx](https://www.nginx.com/) or [Caddy](https://caddyserver.com/) and enable **HTTPS (SSL/TLS)**.
This will encrypt the traffic between your client and the server, protecting your `SECRET_TOKEN` from being intercepted as it travels over the network.
