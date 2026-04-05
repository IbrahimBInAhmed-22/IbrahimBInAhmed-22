# Hi, I'm Ibrahim 👋

I'm a Computer Science student who enjoys building things from scratch — whether that's low-level systems in C/C++, web backends in JavaScript, or machine learning pipelines in Python. My projects range from distributed network tools and OS-level threading to web scrapers and a CNN-based stock predictor.

---

## 🗂️ Projects

### 1. Aliexpress Bot
**Language:** Python | **Tech:** Selenium, Pandas, XPath, CSS Selectors

An automated web scraper that searches AliExpress for products, pages through results, and collects product titles, prices, and URLs into an Excel file.

- Opens AliExpress in a Selenium-controlled browser, closes pop-ups, and runs a search query
- Detects all product cards on the page, hovers to trigger a product preview popup, and extracts the price and title from that popup
- Navigates through every page of results automatically using the pagination bar
- Adds random natural wait times between actions to avoid bot detection
- Saves all collected records to an `.xlsx` file using Pandas
- Handles exceptions gracefully so a single failed card doesn't stop the whole run

---

### 2. DawnNews Scrapper
**Language:** Python | **Tech:** Selenium, Threading, Pandas, CAPTCHA detection

A multi-threaded scraper that extracts news headlines, URLs, dates, and descriptions from Dawn News (Pakistan's major English newspaper) for a given set of city/region queries over a date range.

- Loads a list of unique regions from a CSV file and kicks off a scraper for each one
- Uses Python's `threading` module to run multiple scrapers in parallel, with a configurable thread count
- Each thread navigates to `dawn.com/search?q=<city>`, scrolls through paginated results, and extracts article details
- Filters results to only save articles that fall within a user-specified date range (`r1` to `r2`)
- Thread-safe record collection using `threading.Lock` so multiple threads don't corrupt shared data
- Detects CAPTCHA / bot-detection pages and can resume from a checkpoint page if interrupted
- Exports all collected records to a CSV file

---

### 3. Distributed Password Retrieval System
**Language:** C++ | **Tech:** Winsock2, OpenSSL (PBKDF2, HMAC-SHA1), libpcap, POSIX threads

A distributed WPA2 dictionary-attack system. A central server parses a `.cap` packet capture file to extract the WPA2 4-way handshake, splits a wordlist across multiple cracking clients, and each client independently tests its chunk of passwords.

- **Server:** Uses libpcap to parse raw 802.11 frames, identifies Beacon frames (SSID → BSSID mapping), and extracts ANonce, SNonce, captured MIC, and the EAPOL frame from M1/M2 handshake messages
- **Client:** Receives its chunk of passwords and the handshake data, then runs the full WPA2 key derivation chain: `PBKDF2-HMAC-SHA1` (4096 iterations) → custom `PRF-512` → `HMAC-SHA1` MIC verification
- When any client finds the correct password it sends `FOUND:<password>` to the server, which immediately broadcasts `STOP` to all other clients via a `broadcastStop()` function
- Clients use `select()` with a zero timeout as a non-blocking poll inside the crack loop, so they respond to the STOP signal almost instantly without adding latency
- All data sent over TCP uses a length-prefixed binary protocol (`uint32_t` size before every blob) to handle TCP's stream-based delivery correctly
- Thread safety on the server is managed with two mutexes: one for the shared client socket list, one for the `password_found` flag

---

### 4. Smart Pointers & Multithreaded Matrix Processor
**Language:** C++ | **Tech:** POSIX threads (`pthread`), Templates, Operator Overloading

A from-scratch implementation of C++ smart pointers (`unique_ptr`, `shared_ptr`, `weak_ptr`) without using `<memory>`, combined with a multi-threaded matrix operation engine.

**Smart Pointer module:**
- `memory<T>` acts as a control block (like a reference-counting front desk) — it tracks all `shared_ptr` instances owning the data and all `weak_ptr` observers
- `shared_ptr<T>` supports copy construction, copy assignment, `operator*`, `operator->`, `operator[]`, `reset()`, and a `make_shared()` factory. Destroying the last copy frees the data and nullifies all weak pointers
- `weak_ptr<T>` holds a non-owning reference to the same control block. It is automatically set to null when the last `shared_ptr` is destroyed
- `unique_ptr<T>` enforces exclusive ownership — copy constructor and copy assignment are deleted at compile time; only move semantics are allowed

**Matrix Threading module:**
- An `Operation` abstract base class with subclasses: `addOperation`, `subtractOperation`, `multiplyOperation`, and `squareOperation`
- `threadManager::processMatrix()` spawns one POSIX thread per matrix row, each carrying a `unique_ptr<Operation>` for safe ownership transfer across the thread boundary
- All operations run in parallel; `pthread_join` waits for all threads to complete before printing the result

---

### 5. StockPredictor
**Language:** Python | **Tech:** PyTorch (CNN), yfinance, Matplotlib, scikit-learn, Pandas

A convolutional neural network that downloads historical stock data, generates labeled candlestick chart images, and trains a 5-block CNN to predict Buy / Sell / Hold signals from those charts.

- **Data processing:** Downloads up to 10 years of stock price history via `yfinance`, generates PNG chart images for sliding windows of price data, and labels each chart (Buy/Sell/Hold) based on what the price does in the following period
- **Architecture:** 5 convolutional blocks with filter counts of 32 → 64 → 128 → 256 → 512, each followed by Batch Normalization and ReLU. A classification head with progressive Dropout (0.5 → 0.35 → 0.25) leads to a 3-class output
- **Training:** Cross-Entropy loss, Adam optimizer, `ReduceLROnPlateau` learning rate scheduler, and gradient clipping. Best model is checkpointed based on validation accuracy
- **Results:** Achieved 93.6% test accuracy on Apple (AAPL) stock with 2,390 labeled charts; strong performance on HOLD class (97.8%) with reasonable precision/recall on BUY and SELL
- CLI arguments let you run the full pipeline, data processing only, or training only, and configure symbol, epochs, batch size, and learning rate

---

### 6. Bank Threading (C++ Bank Server)
**Language:** C++ | **Tech:** Boost.Asio, POSIX threads, TCP sockets, Mutexes

A concurrent bank server built with Boost.Asio for TCP networking and POSIX threads for transaction processing. Clients (ATMs) connect and send batches of transaction requests that are processed in parallel.

- `BankServer` maintains an in-memory card database with account names, PINs, and balances, protected by a `pthread_mutex_t`
- `threadManager::execute()` spins up one thread per incoming transaction request and joins them all, ensuring each transaction runs independently
- Each worker thread locks the mutex before modifying account state, preventing race conditions on shared balance data
- The network layer uses a custom `networkPacket` struct to serialise/deserialise requests and responses over TCP
- Separate `adhoc/`, `backend/`, `client/`, and `network/` directories show the evolution of the architecture from a single file to a layered design

---

### 7. BankJava (JavaScript Banking System)
**Language:** JavaScript (Node.js) | **Tech:** Express.js, MongoDB, Mongoose, CORS, dotenv

A RESTful banking backend with a Node.js Express server connecting to MongoDB, plus an ATM client and a central switch/routing layer.

- `backEnd.mjs` defines a Mongoose schema for accounts (`cardNumber`, `PIN`, `name`, `balance`) and exposes REST endpoints for account operations
- A `switch.js` layer acts as a central router that forwards ATM requests to the correct bank server based on routing logic
- `atm.mjs` simulates an ATM client that talks to the switch
- Environment variables (via `dotenv`) configure the port, MongoDB URL, and bank addresses
- CORS is configured to restrict requests to the expected origin
- Uses ES Modules (`"type": "module"`) throughout

---

### 8. Netflix Backend
**Language:** JavaScript (Node.js) | **Tech:** Express.js, MongoDB, Mongoose, JWT, bcrypt, CORS

A REST API backend modelled after a streaming service, with user authentication, show management, and JWT-based session handling.

- `gateway.js` is the central Express server that connects to MongoDB on each request and mounts three route groups: `/auth`, `/user`, and `/shows`
- Authentication routes (`auth_routes.js`) handle registration and login, hashing passwords with bcrypt and signing JWT tokens on successful login
- JWT middleware (`shared/middleware/auth.js`) verifies tokens on protected routes before passing requests to handlers
- Show routes handle browsing and managing streaming content
- Error handling middleware and a centralised logging middleware are applied globally
- Configured for deployment on Vercel (see `vercel.json`)

---

### 9. OS Project (Dropbox-Like File Server)
**Language:** C | **Tech:** POSIX sockets, pthreads, Thread Pools, Mutex/Condition Variables

A multi-threaded file server in pure C that mimics basic Dropbox functionality — clients can upload, download, and manage files with proper concurrent access control.

- Two-tier threading architecture: a **Client Thread Pool** (8 threads) handles incoming TCP connections from `accept()`, and a **Worker Thread Pool** (4 threads) processes the actual file tasks
- `ClientQueue` and `TaskQueue` act as producer-consumer queues between the tiers, each protected by `pthread_mutex_t` and signalled with condition variables
- `UserManager` handles user registration and login, keeping track of per-user file storage directories
- A signal handler intercepts `SIGINT`/`SIGTERM` for graceful shutdown — it sets a `server_running` flag, closes the server socket to unblock `accept()`, and shuts down all thread pools cleanly
- A companion `client.c` connects to the server and issues file operations over TCP

---

### 10. Pokémon Card Battle Game
**Language:** JavaScript (Node.js) | **Tech:** MongoDB, bcrypt, prompt-sync, ES Modules

A terminal-based Pokémon card game where users can register, log in, collect cards, and battle other users, with persistent data stored in MongoDB.

- Users register and log in with bcrypt-hashed passwords; sessions are tracked in memory during the game
- Players can browse and collect Pokémon cards from a card database (`cards.mjs` service)
- Two logged-in users can enter a battle: each picks a card, attack values are compared (with some randomness), and the winner earns coins while the loser loses a penalty
- Battle history is persisted to MongoDB and can be viewed at any time via the battle history controller
- Menu-driven terminal UI using `prompt-sync` with a main loop that routes to the correct controller based on user input

---

### 11. System Security
**Tech:** OWASP Threat Dragon

A threat modelling exercise using OWASP Threat Dragon. Contains a structured threat model JSON file documenting potential security threats, attack vectors, and mitigations for a system design.

---

### 12. Weather Man
**Language:** JavaScript (Node.js) | **Tech:** Node.js, File System (fs), prompt-sync, CSV export

A command-line weather data management tool that stores, analyses, and exports weather records.

- `weatherData` class stores a date (with full validation — rejects invalid formats like `2025-13-40`), temperature, humidity, wind speed, and weather condition per entry
- `DAte` class handles date comparisons (`isBefore`, `isAfter`, `isEqual`, `isLessThanOrEqual`, `isGreaterThanOrequal`) for filtering records by date range
- `WeatherDatabase` class manages a collection of weather entries and supports adding new records, printing all entries, and generating statistical summaries (average temperature, humidity, etc.)
- Exports all data to a CSV file using Node's built-in `fs.writeFileSync`
- Includes a sample dataset with five weather entries as a demonstration

---

## 🛠️ Tech Stack at a Glance

| Area | Technologies |
|---|---|
| **Languages** | Python, C, C++, JavaScript (Node.js) |
| **Backend** | Express.js, Boost.Asio, POSIX sockets |
| **Databases** | MongoDB (Mongoose) |
| **Machine Learning** | PyTorch, scikit-learn, yfinance, Matplotlib |
| **Web Scraping** | Selenium, Pandas |
| **Systems** | POSIX threads, mutexes, TCP sockets, OpenSSL |
| **Tools** | Git, nodemon, dotenv, Vercel |

---

## 📬 Get in Touch

Feel free to explore the repos, open issues, or reach out if you'd like to discuss any of the projects.
