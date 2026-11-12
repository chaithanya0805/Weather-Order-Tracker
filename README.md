# Weather-Aware Order Tracker 🌦️📦

A production-ready, asynchronous Java 17 order fulfillment tracking system that enriches e-commerce orders with real-time weather alerts via the OpenWeatherMap API.

---

## 🌟 Key Features

- **High-Throughput Concurrency**: Built with native Java 11+ `java.net.http.HttpClient` and `CompletableFuture.allOf(...)`. Weather lookups are triggered across all orders in parallel without thread-blocking sequential bottlenecks.
- **"Golden Flow" Automated Logic**:
  - Automatically identifies severe weather conditions (`Rain`, `Snow`, `Extreme`).
  - Updates order status to `"Delayed"` and records the condition.
  - Automatically generates an empathetic, personalized customer apology message using AI-style contextual templating.
  - Marks favorable weather conditions as `"On Schedule"`.
- **Fault-Tolerant Resilience**:
  - Nonexistent cities (such as `InvalidCity123` returning `HTTP 404`) are safely isolated and logged to `System.err`.
  - The application completes successfully without crashing or dropping valid orders.
  - Failed orders are marked `"Failed - Invalid City"` with descriptive error metadata.
  - Network timeouts and socket dropouts are handled safely via `.exceptionally(...)`.
- **Zero-Secret Security**:
  - API keys are dynamically loaded from a local `.env` file via `dotenv-java` or system environment variables.
  - Comprehensive `.gitignore` keeps API secrets and build artifacts out of source control.
- **Clean Deliverables**:
  - Automatically writes the enriched dataset to `updated_orders.json` with 2-space indentation.
  - Interactive CLI dashboard with timing benchmarks and status breakdowns.

---

## 📂 Project Structure

```
Weather-Order-Tracker/
├── pom.xml                               # Maven project configuration (Java 17)
├── orders.json                           # Input mock order database
├── updated_orders.json                   # Output enriched order database (generated)
├── .env.example                          # Template for environment configuration
├── .gitignore                            # Git exclusion rules
├── AI_LOG.md                             # AI prompt engineering & architecture log
├── README.md                             # System overview and setup instructions
└── src/
    └── main/
        └── java/
            └── WeatherOrderTracker.java  # Main execution orchestrator
```

---

## 📋 Prerequisites

Before running this project, ensure you have:

1. **Java Development Kit (JDK) 17 or higher**:
   ```bash
   java -version
   ```
2. **Apache Maven 3.8 or higher**:
   ```bash
   mvn -version
   ```
3. **OpenWeatherMap API Key**:
   - Register for a free API key at [OpenWeatherMap](https://openweathermap.org/api).

---

## ⚙️ Setup & Configuration

1. **Clone or navigate to the project directory**:
   ```bash
   cd Weather-Order-Tracker
   ```

2. **Configure your API Key**:
   Copy `.env.example` to `.env`:
   - On Windows (PowerShell):
     ```powershell
     Copy-Item .env.example .env
     ```
   - On Linux / macOS / Bash:
     ```bash
     cp .env.example .env
     ```

3. **Edit `.env` and paste your OpenWeatherMap API Key**:
   ```env
   OPENWEATHER_API_KEY=your_actual_api_key_here
   ```

> [!NOTE]
> Alternatively, you can export `OPENWEATHER_API_KEY` directly in your shell:
> `export OPENWEATHER_API_KEY="your_api_key"` (Linux/macOS) or `$env:OPENWEATHER_API_KEY="your_api_key"` (PowerShell).

---

## 🚀 Build & Execution

### 1. Compile the Project
```bash
mvn clean compile
```

### 2. Run the Application
Execute directly via the configured Maven Exec Plugin:
```bash
mvn exec:java
```

Or run via the Java command line:
```bash
# Compile
mvn compile

# Execute with dependencies on the classpath
mvn exec:java -Dexec.mainClass="WeatherOrderTracker"
```

---

## 📊 Sample Execution & Expected Output

### Input (`orders.json`)
```json
[
  { "order_id": "1001", "customer": "Alice Smith", "city": "New York", "status": "Pending" },
  { "order_id": "1002", "customer": "Bob Jones", "city": "Mumbai", "status": "Pending" },
  { "order_id": "1003", "customer": "Charlie Green", "city": "London", "status": "Pending" },
  { "order_id": "1004", "customer": "InvalidCity123", "city": "InvalidCity123", "status": "Pending" }
]
```

### Sample CLI Console Output
```text
================================================================================
            WEATHER-AWARE ORDER TRACKER - SYSTEM INITIALIZATION                 
                   Java 17 Asynchronous Pipeline                                
================================================================================
[INFO] Reading orders from: C:\...\Weather-Order-Tracker\orders.json
[INFO] Successfully loaded 4 orders for processing.

================================================================================
          STARTING CONCURRENT ASYNCHRONOUS WEATHER ENRICHMENT
================================================================================
[DISPATCH] Order #1001 -> Initiating async weather query for 'New York' (Customer: Alice Smith)
[DISPATCH] Order #1002 -> Initiating async weather query for 'Mumbai' (Customer: Bob Jones)
[DISPATCH] Order #1003 -> Initiating async weather query for 'London' (Customer: Charlie Green)
[DISPATCH] Order #1004 -> Initiating async weather query for 'InvalidCity123' (Customer: InvalidCity123)
[ERROR] Order #1004 (InvalidCity123): OpenWeatherMap returned HTTP 404 - City Not Found
[ON SCHEDULE] Order #1002 (Mumbai): Weather is Clear ("clear sky")
[DELAYED]  Order #1001 (New York): Delayed due to Rain ("light rain")
[ON SCHEDULE] Order #1003 (London): Weather is Clouds ("overcast clouds")

[SUCCESS] Successfully written updated orders to: C:\...\Weather-Order-Tracker\updated_orders.json
[PERFORMANCE] Processed 4 orders in 842 ms.

================================================================================
                             EXECUTION SUMMARY                                  
================================================================================
 Total Orders Processed : 4
 Orders On Schedule     : 2
 Orders Delayed         : 1
 Orders Failed / Errors : 1
--------------------------------------------------------------------------------
 [REVIEWER PREVIEW] (Shown automatically if delayed == 0)
 Sample AI Contextual Apology:
 >> "Hi Alice Smith, your order to New York is delayed due to Rain. We appreciate your patience!"
================================================================================
```

### Output Deliverable (`updated_orders.json`)
```json
[
  {
    "order_id": "1001",
    "customer": "Alice Smith",
    "city": "New York",
    "status": "Delayed",
    "delay_reason": "Rain",
    "apology_message": "Hi Alice Smith, your order to New York is delayed due to Rain. We appreciate your patience!"
  },
  {
    "order_id": "1002",
    "customer": "Bob Jones",
    "city": "Mumbai",
    "status": "On Schedule",
    "weather_condition": "Clear"
  },
  {
    "order_id": "1003",
    "customer": "Charlie Green",
    "city": "London",
    "status": "On Schedule",
    "weather_condition": "Clouds"
  },
  {
    "order_id": "1004",
    "customer": "InvalidCity123",
    "city": "InvalidCity123",
    "status": "Failed - Invalid City",
    "error": "City not found (HTTP 404)"
  }
]
```

---

## 🛡️ Error Handling & Edge Cases

| Scenario | HTTP Status / Exception | System Behavior | Output Status |
| :--- | :--- | :--- | :--- |
| **Normal Weather** (Clear, Clouds, Mist) | `200 OK` | Evaluates condition; flags shipment on schedule | `"On Schedule"` |
| **Severe Weather** (Rain, Snow, Extreme) | `200 OK` | Triggers delay status, logs condition, generates personalized apology | `"Delayed"` |
| **Invalid City** (`InvalidCity123`) | `404 Not Found` | Logs error to `System.err`; prevents crash; finishes batch | `"Failed - Invalid City"` |
| **Invalid / Expired Key** | `401 Unauthorized` | Logs credential alert to `System.err`; details error in JSON | `"Failed - Unauthorized API Key"` |
| **Connection Timeout** | `HttpConnectTimeoutException` | Handled via `.exceptionally(...)`; records timeout | `"Failed - Network Timeout"` |

---

## 📦 Dependencies

- **[org.json:json (20231013)](https://github.com/stleary/JSON-java)**: High-performance, lightweight JSON parser and serializer.
- **[io.github.cdimascio:dotenv-java (3.0.0)](https://github.com/cdimascio/dotenv-java)**: Zero-dependency environment variable manager for loading `.env` configuration.
- **Java 17 `java.net.http.HttpClient`**: Native HTTP/2 client with asynchronous non-blocking futures.
- **Java 17 `java.util.concurrent.CompletableFuture`**: Composable, non-blocking asynchronous coordination.

---

## 📜 License

Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0).

