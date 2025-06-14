# IceNet Dashboard deployment repo

This meta-repository coordinates a modular, containerised icenet dashboard application composed of:

- **[icenet-dashboard-preprocessor](https://github.com/icenet-ai/icenet-dashboard-preprocessor)** – An installable Python package that converts netCDF files to Cloud-Optimised GeoTIFFs (COGs) for use by the tiler.
- **[icenet-tiler-api](https://github.com/icenet-ai/icenet-tiler-api)** – A TiTiler + FastAPI-based backend that serves map tiles and static data.
- **[icenet-dashboard](https://github.com/icenet-ai/icenet-dashboard)** – A Plotly Dash web app for visualising IceNet forecasts.

---

## Repo Structure

```bash
meta-repo/
├── icenet-dashboard/ # Plotly Dash web-app served with Gunicorn
├── icenet-tiler-api/ # Tile and data server using FastAPI + Uvicorn
├── icenet-dashboard-preprocessor/ # Data transformer: NetCDF → COGs
├── data/ # Output COGs and STAC catalog generated by running `icenet-dashboard-preprocessor` on IceNet prediction netCDF files
├── compose.yaml # Orchestrates all containers
└── Makefile # Top-level automation commands
```

This is the basic workflow:

![Architecture diagram](docs/images/dashboard-dash-leaflet-schematic-light.jpg "Architecture Diagram")

---

## Getting Started

### 1. Install Docker

Ensure [Docker](https://docs.docker.com/get-docker/) is installed on your system.

---

### 2. Install and Run IceNet netCDF -> CoG Preprocessor

```bash
pip install -e icenet-dashboard-preprocessor/

icenet_dashboard_preprocess <path_to_icenet_netcdf_predictions>
```

#### Example usage:

To point to a directory with netCDF files:

`icenet_dashboard_preprocess -i results/predict/`

Using wildcards:

`icenet_dashboard_preprocess -i raw_data/*.nc`

This will create a `data/` directory with the JSON catalog and CoG outputs.

```bash
data/
├──  cogs/
│   ├──  north/
│   └──  south/
└──  stac/
    ├──  north/
    ├──  south/
    └──  catalog.json
```

### 3. Set up configuration ENV file (WIP)

Each service can be configured via environment variables. They
are orchestrated via `compose.yaml` for consistency across
environments.

Run the following script which will output a minimal `.env` file which
includes the ports the components should be deployed one, and the IP
address that the COGs should be accessible from.

```bash
./generate_env.sh
```

There are quite a few duplications (e.g. hostname, ports) to be taken care of since originally each
component of the stack was run independently.

### 4. Build & Launch the Full Stack

```bash
make up
```

This will:

* Build all service images (icenet-dashboard, icenet-tiler-api)
* Launch the dashboard and tile server stack via docker-compose

### 5. Access the UI Interfaces and Docs

* Dashboard UI: http://localhost
* Tiler API: http://localhost:8000
* Data Server: http://localhost:8002

