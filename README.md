# 🛰️ Satellite NDVI Processing Pipeline

Automated satellite imagery processing pipeline with a QGIS plugin for vegetation monitoring, change detection, and environmental alerts.

**Author:** Michel M. Nzikou  
**Contact:** michel@dmnsolutions.com.au  
**Organization:** [DMN SOLUTIONS](https://github.com/DMN-SOLUTIONS)

---

## What It Does

1. Downloads **free Sentinel-2 satellite imagery** from AWS Open Data
2. Computes spectral indices: **NDVI** (vegetation), **NDWI** (water), **NBR** (fire scars)
3. Vectorizes results into GeoJSON polygons
4. Detects **changes between dates** (vegetation loss/gain)
5. Generates **alerts** when significant environmental changes occur
6. All accessible via a **QGIS plugin** with a modern dark UI

---

## Demo

![Satellite NDVI Pipeline Demo](demo.gif)

### Quick Start (2 minutes)

```bash
# 1. Clone the repo
git clone https://github.com/DMN-SOLUTIONS/satellite-ndvi-pipeline.git
cd satellite-ndvi-pipeline

# 2. Install the QGIS plugin
# macOS:
cp -r qgis-plugin ~/Library/Application\ Support/QGIS/QGIS3/profiles/default/python/plugins/ndvi_pipeline_loader

# Linux:
cp -r qgis-plugin ~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/ndvi_pipeline_loader

# 3. Install boto3 into QGIS Python (optional, only needed for S3 features)
# macOS QGIS-LTR:
/Applications/QGIS-LTR.app/Contents/MacOS/bin/python3.9 -m pip install boto3

# 4. Open QGIS → Plugins → Manage and Install → Enable "Satellite NDVI Processor"
```

### Using the Plugin

**Process Tab — Single Date Analysis:**
1. Select a WA tile (Perth, Geraldton, Albany, Broome, etc.)
2. Pick an area of interest (Kings Park, Swan River, Darling Scarp...)
3. Choose a date from the calendar (green = data available)
4. Select an index: NDVI, NDWI, or NBR
5. Click **⚡ Process Area**
6. Results load automatically with color-coded styling

**Change Detection Tab — Compare Two Dates:**
1. Set "Before" and "After" dates
2. Click **📊 Detect Changes**
3. Red areas = loss, Green areas = gain
4. Alert polygons highlight significant drops

**Alerts Tab:**
- Configure sensitivity threshold
- Enable alerts for vegetation loss, water change, or burn detection
- Alert log tracks all detected changes

### View Sample Data (No Internet Needed)

```bash
# Open the demo project in QGIS
open ndvi-demo.qgs

# Or drag these files into QGIS:
# - data-sample/ndvi_sample.tif (synthetic NDVI raster)
# - data-sample/ndvi_vegetation_sample.geojson (vegetation polygons)
# - data-sample/ndvi_real.tif (real Sentinel-2 NDVI, Perth Jan 2023)
# - data-sample/ndvi_real_vegetation.geojson (real vegetation polygons)
```

---

## Architecture

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│ EventBridge │────▶│ AWS Lambda   │────▶│ S3 Output    │
│ (weekly)    │     │ (processing) │     │ (GeoTIFF +   │
└─────────────┘     └──────┬───────┘     │  GeoJSON)    │
                           │             └──────┬───────┘
                    ┌──────┴───────┐            │
                    │ Sentinel-2   │     ┌──────┴───────┐
                    │ AWS Open Data│     │ QGIS Plugin  │
                    │ (FREE)       │     │ (UI + maps)  │
                    └──────────────┘     └──────────────┘
```

## Features

| Feature | Description |
|---------|-------------|
| 🌍 Multi-tile | 10 WA tiles from Perth to Broome |
| 📅 Date picker | Calendar with Sentinel-2 availability highlighted |
| 📊 3 Indices | NDVI (vegetation), NDWI (water), NBR (burn) |
| 🔄 Change detection | Compare any two dates, see gain/loss |
| 🚨 Alerts | Configurable thresholds, alert log |
| 🎨 Modern UI | Dark theme, tabbed interface, progress bars |
| ☁️ S3 integration | Load from deployed AWS pipeline |
| 💰 SaaS API | API Gateway + API keys + usage plans |

---

## Deploy the AWS Pipeline (Optional)

```bash
cd infra
sam build
sam deploy --guided
```

This deploys:
- S3 buckets (input/output)
- Lambda function (NDVI processor)
- EventBridge (weekly schedule)
- API Gateway with API key auth
- Usage plan (1000 req/month)

### API Usage

```bash
curl -X POST https://YOUR_API.execute-api.ap-southeast-2.amazonaws.com/prod/ndvi \
  -H "x-api-key: YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"tile": "36/R/UU", "date": "2024-01"}'
```

---

## Project Structure

```
satellite-ndvi-pipeline/
├── qgis-plugin/            # QGIS plugin (main deliverable)
│   ├── __init__.py
│   ├── ndvi_loader.py      # Plugin logic (UI + processing)
│   ├── metadata.txt
│   └── demo_data/          # Bundled sample data
├── src/
│   ├── handler.py          # Lambda: scheduled NDVI processing
│   ├── api_handler.py      # Lambda: SaaS API endpoint
│   └── requirements.txt
├── infra/
│   ├── template.yaml       # SAM template (Lambda + API GW + S3)
│   └── samconfig.toml
├── data-sample/            # Sample outputs
├── notebooks/
│   ├── generate_sample.py  # Generate synthetic data
│   └── download_real_ndvi.py  # Download real Sentinel-2 data
├── client_demo.py          # API client demo
├── ndvi-demo.qgs           # QGIS project file
└── README.md
```

---

## Cost

| Component | Cost |
|-----------|------|
| Sentinel-2 imagery | **Free** (AWS Open Data) |
| QGIS plugin | **Free** (open source) |
| AWS pipeline (free tier) | **$0/month** |
| AWS pipeline (production) | **~$2-5/month** |

---

## Contributing

Contributions welcome! Here's how:

1. Fork the repo
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Commit changes: `git commit -m "Add my feature"`
4. Push: `git push origin feature/my-feature`
5. Open a Pull Request

### Ideas for Contributions

- [ ] Add more WA tiles / extend to other Australian states
- [ ] Time-series animation (NDVI over 12 months)
- [ ] Cloud masking (skip cloudy pixels)
- [ ] Export reports as PDF
- [ ] Integration with BOM weather data
- [ ] Mobile-friendly web viewer
- [ ] Support for Landsat imagery
- [ ] Automated scene discovery (list actual available dates from S3)

---

## License

MIT

---

## Acknowledgments

- [Sentinel-2 on AWS](https://registry.opendata.aws/sentinel-2-l2a-cogs/) — free satellite imagery
- [QGIS](https://qgis.org/) — open source GIS platform
- [rasterio](https://rasterio.readthedocs.io/) — Python raster I/O
