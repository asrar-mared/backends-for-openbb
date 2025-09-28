# Company Revenue Dashboard & MCP Server

This project provides both an OpenBB Workspace widget and an HTTP API server for accessing company revenue data. The widget displays interactive revenue metrics with ticker selection, while the HTTP API exposes the same functionality via REST endpoints.

## Features

### OpenBB Widget Features
- **📅 Date Picker**: Select start date for revenue data (7-day range)
- **📊 Ticker Dropdown**: Select from major tech stocks (AAPL, MSFT, GOOGL, AMZN, TSLA, META, NVDA)
- **💰 Revenue Metrics**: Daily revenue data with percentage changes and trends

### HTTP API Features
- **🌐 REST Endpoints**: GET and POST endpoints for revenue data retrieval
- **📈 Revenue Analytics**: 7 days of revenue data with summary statistics
- **🔧 Flexible Parameters**: Ticker symbol and date range selection
- **📚 Interactive Docs**: Swagger/OpenAPI documentation included

### AgGrid Capabilities
- **Advanced Table Features**: Column definitions, formatting, sorting, filtering
- **Sparklines**: 7-day trend visualization with min/max highlighting
- **Color Coding**: Status indicators (Good/Warning/Critical)
- **Chart View**: Toggle between table and column chart views
- **Dynamic Data**: Content changes based on parameter selections

## Quick Start

### 1. Install Dependencies

```bash
# Create a virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install required packages
pip install -r requirements.txt
```

### 2. Run the OpenBB Widget Server

```bash
python main.py
```

The server will start on `http://localhost:8012`

### 3. Run the HTTP API Server

```bash
python mcp_server.py
```

The API server will start on `http://localhost:8013`

### 4. Verify Installation

**OpenBB Widget URLs:**
- **Widget Registry**: http://localhost:8012/widgets.json
- **Root Endpoint**: http://localhost:8012/
- **Sample Data**: http://localhost:8012/dynamic_aggrid_table

**HTTP API URLs:**
- **API Root**: http://localhost:8013/
- **Interactive Docs**: http://localhost:8013/docs
- **Revenue Data**: http://localhost:8013/revenue-data
- **Example**: http://localhost:8013/revenue-data?ticker=AAPL&start_date=2024-01-01

## Integration with OpenBB Workspace

### Adding the Widget to OpenBB Workspace

1. **Open OpenBB Workspace**
2. **Add Custom Backend**:
   - Go to Settings → Widgets
   - Add custom backend URL: `http://localhost:8012`
3. **Widget Discovery**:
   - The widget "Company Revenue Dashboard" should appear in the widgets menu
4. **Add to Dashboard**:
   - Drag the widget to your dashboard
   - Configure the parameters as needed

### Widget Configuration

The widget accepts these parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `start_date` | Date | 7 days ago | Starting date for revenue data |
| `ticker` | Dropdown | "AAPL" | Company ticker symbol |

#### Available Tickers:
- **AAPL**: Apple Inc.
- **MSFT**: Microsoft Corp.
- **GOOGL**: Alphabet Inc.
- **AMZN**: Amazon.com Inc.
- **TSLA**: Tesla Inc.
- **META**: Meta Platforms Inc.
- **NVDA**: NVIDIA Corp.

## HTTP API Integration

### API Endpoints

#### GET `/revenue-data`
Get revenue data using query parameters:
```bash
curl "http://localhost:8013/revenue-data?ticker=AAPL&start_date=2024-01-01"
```

#### POST `/revenue-data`
Get revenue data using JSON payload:
```bash
curl -X POST "http://localhost:8013/revenue-data" \
  -H "Content-Type: application/json" \
  -d '{"ticker": "MSFT", "start_date": "2024-01-01"}'
```

### API Documentation

**Endpoint**: `/revenue-data`

**Parameters**:
- `ticker` (string): Company ticker symbol (default: "AAPL")
  - Valid values: AAPL, MSFT, GOOGL, AMZN, TSLA, META, NVDA
- `start_date` (string, optional): Start date in YYYY-MM-DD format (defaults to 7 days ago)

**Returns**: JSON with revenue data including:
- Company information (name, sector)
- 7 days of revenue data with trends
- Summary statistics (total revenue, average daily revenue)

### Interactive Documentation

Visit `http://localhost:8013/docs` for Swagger UI with:
- Interactive API testing
- Request/response examples
- Schema documentation

## Data Generation Strategy

The widget uses a **deterministic data generation** approach:

1. **Consistency**: Same parameters always produce identical data
2. **Seeded Randomness**: Uses hash of (date + company + category) as seed
3. **Dynamic Range**: Generates 7 days × 5 companies = 35 data points
4. **Realistic Values**: Base values scaled by company size and time progression

### Example Data Flow:
```
Input: start_date="2024-01-01", company_prefix="Finance", data_category="operational"
↓
Output: 35 rows with companies like "Finance Corp", "Finance Systems" 
        showing Production, Efficiency, etc. metrics from Jan 1-7
```

## Technical Details

### Project Structure
```
matching-widget-mcp-tool/
├── main.py           # OpenBB Widget FastAPI application
├── mcp_server.py     # HTTP API server for revenue data
├── requirements.txt  # Python dependencies
└── README.md        # This file
```

### Key Components

#### 1. Widget Registration
```python
@register_widget({
    "name": "Company Revenue Dashboard",
    "type": "table",
    "endpoint": "dynamic_aggrid_table",
    "params": [...],
    "data": {"table": {"columnsDefs": [...]}}
})
```

#### 2. Data Generation Functions
- `_get_ticker_info()`: Returns company information for ticker symbols
- `_generate_revenue_data()`: Creates revenue data for single company/day
- `dynamic_aggrid_table()`: Main endpoint handling parameters

#### 3. HTTP API Components
- `FastAPI`: Web framework for REST endpoints
- `get_company_revenue_data()`: GET endpoint function
- `post_company_revenue_data()`: POST endpoint function
- `RevenueDataRequest`: Pydantic model for validation

#### 4. AgGrid Configuration
- **Column Definitions**: Data types, formatting, rendering functions
- **Sparklines**: Trend visualization with styling options
- **Chart Integration**: Seamless table-to-chart conversion

### CORS Configuration

The application is configured to work with OpenBB Workspace domains:
- `https://pro.openbb.co`
- `https://pro.openbb.dev`
- `http://localhost:1420` (local development)

## Customization

### Adding New Metrics Categories

1. **Update metrics map** in `_get_metrics_map()`:
```python
"custom_category": ["Metric1", "Metric2", "Metric3"]
```

2. **Add dropdown option** in widget parameters:
```python
{"value": "custom_category", "label": "Custom Category"}
```

### Modifying Data Generation

- **Change time range**: Modify the `range(7)` loop
- **Add companies**: Extend `company_suffixes` list
- **Adjust values**: Modify `base_value` calculation
- **New status logic**: Update status determination rules

### Styling Customization

- **Colors**: Modify `renderFnParams.colorRules`
- **Sparklines**: Adjust `sparkline.options`
- **Grid Layout**: Change `gridData` dimensions

## Troubleshooting

### Common Issues

1. **Port Already in Use**:
   ```bash
   # Change port in main.py or kill existing process
   lsof -ti:8000 | xargs kill -9
   ```

2. **CORS Errors**:
   - Ensure OpenBB Workspace domain is in `origins` list
   - Check that server is accessible from workspace

3. **Widget Not Appearing**:
   - Verify `/widgets.json` endpoint returns data
   - Check OpenBB Workspace backend configuration
   - Confirm server is running and accessible

### Development Tips

- **Live Reload**: Server runs with `reload=True` for development
- **Debug Data**: Visit endpoint URLs directly to inspect JSON output
- **Parameter Testing**: Use query parameters in browser: `?start_date=2024-01-01&company_prefix=Test&data_category=hr`

## Dependencies

- **FastAPI**: Modern, fast web framework for building APIs
- **Uvicorn**: ASGI server for running FastAPI applications
- **Pydantic**: Data validation and serialization
- **Standard Library**: `datetime`, `random`, `functools`, `asyncio`, `pathlib`

## License

This is a demonstration project for OpenBB Workspace widget development.

## Support

For questions about:
- **OpenBB Workspace**: Visit [OpenBB Documentation](https://docs.openbb.co/)
- **Widget Development**: Check [OpenBB Widget Docs](https://docs.openbb.co/pro/widgets)
- **This Implementation**: Review the code comments and examples in `main.py`