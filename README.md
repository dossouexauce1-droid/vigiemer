# VigieMer

**Dark-vessel watch for the Gulf of Guinea, from free radar satellites.**

VigieMer finds ships on Sentinel-1 radar images, then checks each one against the AIS positions ships are supposed to broadcast. A ship that the radar sees but that is silent on AIS is flagged for review. Coastal states in the Gulf of Guinea lose large amounts of fish and revenue to illegal, unreported and unregulated (IUU) fishing, and most of them cannot afford patrol aircraft. Radar images from the EU's Copernicus programme are free, work at night and through cloud, and cover the whole coast every few days. VigieMer turns them into a short, ranked list of ships to check.

> Status: working research prototype, built by a student in Benin for the UniPods AI Innovation Pipeline (Cohort 2). Not an operational system.

## What it does

```
Sentinel-1 radar image (ESA Copernicus, free)
        │  download VH + VV bands, reproject to a 9.55 m grid
        ▼
Ship detection  ── Faster R-CNN detector (Allen Institute for AI, pretrained on Sentinel-1)
        │          + attribute network: length, beam, speed, heading, fishing likelihood
        │          runs on 2 × T4 GPUs in parallel (one scene per GPU)
        ▼
AIS cross-match ── Global Fishing Watch AIS presence for the same hour
        │          one AIS ship can explain one echo only (closest pairs first)
        │          search radius widens with the ship's AIS speed
        ▼
Fixed objects   ── echoes within 200 m of each other on 3+ passes (platforms, buoys, wrecks)
        ▼
Classification  ── identified by AIS · fixed object · DARK ≥ 25 m (alert) · small, no AIS · AIS not yet published
        ▼
Map + alert list (web app) with a radar chip of every alert
```

Every result is also compared with Global Fishing Watch's own radar detections for the same pass, as an independent check of the detector.

## First results

Six Sentinel-1 passes off Lomé, Cotonou and Lagos, 28 September to 21 October 2026 (detector score ≥ 0.9, AIS match within 3 km and ±1 h):

| Status | Echoes |
|---|---|
| Identified by AIS | 636 |
| Fixed object | 246 |
| **No AIS, 25 m or longer (alert)** | **232** |
| No AIS, under 25 m | 4 |
| **Total at sea** | **1,118** |

Cross-check: Global Fishing Watch's own SAR detections on the same six passes total 678, against 872 for VigieMer once fixed objects are removed (16 October: 180 vs 188). Notebook 01 processed the six scenes in about 31 minutes on Kaggle's 2 × T4.

## Repository

| Path | What it is |
|---|---|
| `notebooks/VigieMer_01_detection.ipynb` | Finds Sentinel-1 passes over the coast, runs ship detection on 2 GPUs, saves detections and radar chips |
| `notebooks/VigieMer_02_dark_vessels.ipynb` | Downloads AIS from Global Fishing Watch, cross-matches, classifies, exports the app data |
| `app/index.html` | The VigieMer web app: chart of the coast, passes, counts, alerts, radar chips |

## Run it

Both notebooks run on Kaggle (free tier).

1. **Notebook 01.** Accelerator *GPU T4 × 2*, Internet *On*, *Save & Run All*. About 10 minutes per pair of scenes. Optional Kaggle secrets `CDSE_USER` / `CDSE_PASSWORD` (free Copernicus account) are used only if Microsoft Planetary Computer is unavailable.
2. **Notebook 02.** Add notebook 01 as input. Kaggle secret `GFW_TOKEN` (free Global Fishing Watch API token). No GPU needed, about 1 minute.
3. Open `vigiemer_app_data.json` with the app.

Running cost: **0 USD per month** (free satellite data, free AIS API for non-commercial use, free Kaggle GPUs, static web page).

## Limits

- No AIS is not proof of wrongdoing. Transponders fail, coverage has gaps, and navy or service vessels may not broadcast.
- Radar can return buoys, small platforms or wave clutter. Every alert comes with its radar chip so a human can judge.
- Global Fishing Watch publishes AIS about 4 days after the fact. VigieMer works on passes at least 4 days old; a real-time version needs a live AIS feed.
- Length, speed and heading are estimates from a 10 m radar image.

## Credits

- Sentinel-1 imagery: © ESA / European Union, Copernicus programme. Accessed through Microsoft Planetary Computer and the Copernicus Data Space Ecosystem.
- Ship detector and attribute model: Allen Institute for AI, [vessel-detection-sentinels](https://github.com/allenai/vessel-detection-sentinels), Apache-2.0. VigieMer replaces its GDAL preprocessing with rasterio and adds multi-GPU batching.
- AIS vessel presence and SAR detections: © [Global Fishing Watch](https://globalfishingwatch.org), used under their API terms (non-commercial, attribution).
- Coastline and borders: Natural Earth (public domain).

## Author

Exaucé Dossou, Benin. AI olympiad competitor (IOAI / APOAI circuit).
