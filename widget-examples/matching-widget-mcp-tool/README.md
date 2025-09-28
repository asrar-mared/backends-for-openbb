# Dynamic AgGrid Widget for OpenBB Workspace

This project demonstrates a dynamic AgGrid table widget for OpenBB Workspace with three interactive parameters that affect the displayed data in real-time.

## Features

### Interactive Parameters
- **📅 Date Picker**: Select start date for data generation (affects 7-day range)
- **📝 Text Input**: Company name prefix (customizes company names)
- **📊 Dropdown**: Data category selection (Financial, Operational, Marketing, HR metrics)

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

### 2. Run the Server

```bash
python main.py
```

The server will start on `http://localhost:8000`

### 3. Verify Installation

Visit these URLs to confirm everything is working:

- **Widget Registry**: http://localhost:8000/widgets.json
- **Root Endpoint**: http://localhost:8000/
- **Sample Data**: http://localhost:8000/dynamic_aggrid_table

## Integration with OpenBB Workspace

### Adding the Widget to OpenBB Workspace

1. **Open OpenBB Workspace**
2. **Add Custom Backend**:
   - Go to Settings → Widgets
   - Add custom backend URL: `http://localhost:8000`
3. **Widget Discovery**:
   - The widget "Dynamic AgGrid Table" should appear in the widgets menu
4. **Add to Dashboard**:
   - Drag the widget to your dashboard
   - Configure the three parameters as needed

### Widget Configuration

The widget accepts these parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `start_date` | Date | 7 days ago | Starting date for data generation |
| `company_prefix` | Text | "Tech" | Prefix for company names |
| `data_category` | Dropdown | "financial" | Type of metrics to display |

#### Data Categories Available:
- **Financial**: Revenue, Profit, EBITDA, Cash Flow, Operating Margin
- **Operational**: Production, Efficiency, Utilization, Output, Quality Score
- **Marketing**: CAC, LTV, Conversion Rate, ROAS, Click Rate
- **HR**: Headcount, Attrition, Satisfaction, Productivity, Training Hours

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
├── main.py           # Main FastAPI application
├── requirements.txt  # Python dependencies
└── README.md        # This file
```

### Key Components

#### 1. Widget Registration
```python
@register_widget({
    "name": "Dynamic AgGrid Table",
    "type": "table",
    "endpoint": "dynamic_aggrid_table",
    "params": [...],
    "data": {"table": {"columnsDefs": [...]}}
})
```

#### 2. Data Generation Functions
- `_get_metrics_map()`: Returns category-to-metrics mapping
- `_generate_company_data()`: Creates data for single company/day
- `dynamic_aggrid_table()`: Main endpoint handling parameters

#### 3. AgGrid Configuration
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
- **Standard Library**: `datetime`, `random`, `functools`, `asyncio`, `pathlib`

## License

This is a demonstration project for OpenBB Workspace widget development.

## Support

For questions about:
- **OpenBB Workspace**: Visit [OpenBB Documentation](https://docs.openbb.co/)
- **Widget Development**: Check [OpenBB Widget Docs](https://docs.openbb.co/pro/widgets)
- **This Implementation**: Review the code comments and examples in `main.py`