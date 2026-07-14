<!-- 书写报告使用中文; 本 landscape 是单一事实源, 允许膨胀。 -->
# Landscape: 强对流环境–雷达多模态开放基准数据集 (0713-scs-benchmark)

> **来源与健康警告 (dispatcher, 2026-07-13)**
> 下方"用户提供的 deep-research 蓝图"整段转写自用户源文件
> `/blue/yixin.wen/weikangqian/GitRepo/myproject/SCSDataset_benckmark_report.md`,
> 由一个 deep-research 工具生成。**其中所有内联 `citeturn…search…` 标记是 web-search 工具的内部引用锚, 不是可核验的 arXiv/DOI, 论文标题/年份/作者均未经核验。**
> 按项目手册红线, 任何要写进 idea/review/proposal 的论文引用都必须用 `agon:arxiv-tools` 拉原文核验后才能落笔。
> 因此本蓝图仅作**结构性 landscape 起点**(它对数据模型、任务套件、许可、QC、发表清单的组织是有价值的), 具体撞车判定与必引清单以 deep-lit 阶段 arxiv-tools 核验结果为准。

---

## 已读论文 arxiv_id 清单 (deep-lit 维护, 初始为空)

<!-- deep-lit-tick 每读一篇经核验的论文, 在此登记 arxiv_id + 一行结论。 -->
(尚未开始 deep-lit; 初始为空。)

## 已确认撞车风险 / 候选对照数据集 (待 arxiv-tools + 官网核验)

以下数据集来自用户蓝图, 是最可能的"撞车/baseline/可复用"对象, **均未核验**, deep-lit 阶段须逐一核实其真实范围、时窗、模态、许可与是否已含环境场:
- **SEVIR** — 多模态风暴影像基准 (雷达/卫星/闪电), 约 1 万+ 事件, 384×384 km, 固定 4 小时窗。疑似最直接对照。
- **MYRORSS** — 多年雷达再分析 + 近风暴环境分析, 1998–2011, 0.01° 网格。
- **WeatherBench 2** — 全球中期预报评估框架 (非风暴尺度/预警)。
- **HKO-7** — 香港雷达 nowcasting 经典基准 (雷达为主, 地域局部)。
- **MeteoNet** — Météo-France 开放集 (雷达/站点/卫星/NWP/地形掩膜)。
- **TORUS** — 超级单体/龙卷外场过程观测 (campaign 尺度)。
- **Storm Events / SPC archives / NWS CAP alerts** — 权威事件/预警记录 (仅标签)。
- (deep-lit 需另查是否有更新的 2024–2026 强对流多模态基准, 如 TorNet 及类似, 以及"环境场作为一等公民"是否已被他人占坑。)

## 候选原始数据源 (用户设定 + 蓝图扩展, 待核验许可与再分发条款)

MRMS (雷达融合) / NEXRAD Level II (单雷达矩) / GLM (闪电) / IGRA·RAOB (探空) / ERA5 (再分析) / HRRR·WoFS (高分辨 NWP) / GOES ABI (卫星) / ISD·ASOS·MADIS (地面) / SMAP (土壤湿度) / USGS 3DEP (地形) / NOAA Storm Events·SPC·NWS CAP (事件与预警标签)。

---

# 用户提供的 deep-research 蓝图 (原文转写, 引用未核验)

# Open Benchmark Database for Severe Convective Environment Analysis, Event Warning, and Nowcasting

## Executive summary

An open benchmark database built around **storm-centered regional clusters**, paired **MRMS and raw radar observations**, and **much longer-duration environmental context spanning pre-convective conditions, convective initiation, mature severe phase, and decay**, would fill a real gap in the current open-data landscape. Existing resources are valuable, but they are fragmented by purpose: **SEVIR** is a landmark multimodal storm-imagery benchmark with fixed 4-hour event windows, **WeatherBench 2** is a global medium-range forecast benchmark rather than a severe-convection warning dataset, **HKO-7** and **MeteoNet** are strong nowcasting datasets but are mostly radar/precipitation oriented and geographically local, while **MRMS/MYRORSS**, **Storm Events**, **SPC archives**, and **WoFS/ProbSevere products** are open components rather than a single curated benchmark with unified labels, splits, provenance, and evaluation tasks. citeturn3search0turn3search1turn3search13turn4search5turn4search8turn2search3turn21search15

For a journal such as *Earth System Science Data*, the strongest publication angle is **not merely releasing assembled files**, but releasing a rigorously curated, versioned, and openly documented **research data product**: harmonized event packages; machine-readable metadata; provenance down to file and processing step; explicit uncertainty and quality flags; open access through a repository with DOI/versioning; and demonstration analyses that show the dataset supports **climatology, process studies, nowcasting, and warning evaluation**. ESSD explicitly targets high-quality Earth system datasets, requires reliable repositories and data citations, and expects the dataset to be reusable beyond the paper itself. citeturn5search14turn9search4turn13search1turn13search5turn13search16

My overall assessment is that your proposed architecture is **novel enough for an ESSD-style data descriptor if implemented carefully**, especially if it does all of the following at once:  
(1) packages **storm-evolution windows longer than typical image-based benchmarks**;  
(2) links **storm-centered MRMS/radar observations** with **environmental context** from soundings, reanalysis, surface observations, land-surface fields, and high-resolution NWP;  
(3) includes **warning and report labels** suitable for decision-support evaluation; and  
(4) publishes **canonical tasks, splits, baselines, and metrics**. The largest risks are not scientific novelty, but **license mixing, label noise, temporal alignment errors, data volume, and ambiguous event definitions**. citeturn3search0turn4search8turn20search4turn16search0turn13search9

Because your exact **spatial extent, temporal span, event definition, and release repository are unspecified**, the recommendations below are written as a design blueprint. Since MRMS is a NOAA U.S.-focused product, the most natural first release is U.S.-centered; if the project later expands internationally, analogous radar and satellite sources should be substituted with equally explicit metadata and licensing treatment. citeturn0search11turn4search4turn23search14

## Design objectives and recommended data model

The central design choice should be to treat the benchmark not as a stack of frames, but as a set of **event packages** with three linked coordinate systems: **time**, **space**, and **storm object identity**. In practice, each package should contain a fixed or semi-fixed regional domain, native-time observations, harmonized gridded fields, object tracks, and event/warning/report labels. That structure is much more suitable for severe-convection science than a pure image-sequence benchmark, because warning decisions depend on storm history, environmental forcing, and downstream impacts, not only on the next radar frame. This recommendation is consistent with how MRMS, WoFS, ProbSevere, PHI, and modern verification methods are used operationally and in research. citeturn14search0turn20search4turn21search15turn21search0turn7search0

A strong data model would therefore separate:

- **dataset-level entities**: release version, spatial coverage, temporal coverage, source list, processing pipeline, split definitions;
- **event-level entities**: event ID, region cluster ID, start/end time, CI time, mature phase time, decay time, primary hazard labels, associated warning/report IDs;
- **object-level entities**: storm cell/segment IDs, tracks, mergers/splits, per-object features, probabilistic hazard scores;
- **asset-level entities**: the actual radar, satellite, NWP, sounding, and surface files or chunks with source provenance and QC flags. citeturn5search4turn5search0turn5search1turn5search2

For the **regional architecture**, I recommend publishing **both** a fixed-grid and a storm-centered view. The fixed-grid view supports reproducible benchmarking and climatology. The storm-centered view is better for object tracking, storm evolution, and ML training efficiency. If you publish only dynamic storm-following crops, intercomparison becomes harder; if you publish only static grids, environmental inflow and storm-relative evolution are less natural to analyze. A dual-view design significantly improves utility. This is one of the clearest ways your dataset can differentiate itself from image-centric predecessors such as SEVIR. citeturn3search0turn3search15

A good **temporal design** is to begin the environmental context **well before CI** and continue through **system decay**, instead of stopping at the “interesting” severe phase. That matters scientifically because severe outcomes depend not only on the mature storm, but on boundary-layer recovery, pre-storm moisture transport, antecedent land conditions, storm mergers, and upscale growth. In other words, the data package should support both **forecasting** and **post hoc diagnosis**. Operational systems such as WoFS and PHI are explicitly designed around extending short-term lead time and evolving storm-scale hazard guidance, which supports this long-window philosophy. citeturn20search4turn20search9turn21search0turn21search4

```mermaid
flowchart LR
    A[Source-native observations and models] --> B[Ingest and checksum]
    B --> C[Native archive layer]
    C --> D[Spatial and temporal harmonization]
    D --> E[Event builder]
    E --> F[Storm objects and tracks]
    E --> G[Warning and report linkage]
    D --> H[Environmental feature derivation]
    F --> I[Benchmark release]
    G --> I
    H --> I
    I --> J[Zarr or NetCDF cubes]
    I --> K[Parquet or GeoParquet labels]
    I --> L[STAC catalog and DOI release]
```

This pipeline is especially suitable for ESSD because it exposes the full lineage from raw source to benchmark asset, which is exactly what FAIR reuse and data-publication review require. citeturn5search4turn9search9turn13search1

## Essential data holdings and technical specifications

The database should include the data types below. The “typical” resolutions are source-typical, not mandatory benchmark resolutions; because your final extent and timespan are unspecified, the benchmark should preserve native information where possible and supply a harmonized analysis grid in parallel.

| Data type | Why it is essential | Typical native cadence and resolution | Common open source and format | Recommended benchmark representation |
|---|---|---|---|---|
| **MRMS products** | Best open U.S. multi-radar fused severe-weather context; useful for reflectivity mosaics, MESH, azimuthal shear, echo-top and severe proxies. MRMS products are designed to support warnings, aviation, hydrology, and NWP. citeturn0search11turn0search6turn10search7turn10search12 | Many core fields at **2 min**, **0.01°** grid. citeturn10search7turn10search12 | NOAA/NODD MRMS; native files commonly distributed as **GRIB2**. citeturn0search1turn2search15 | Preserve source GRIB2; publish harmonized event cubes in **Zarr** or **NetCDF4**, plus object masks and QC layers. |
| **Single-radar reflectivity and velocity** | Needed for native radar moments, Doppler signatures, radial-velocity-based rotation diagnostics, and reproducible preprocessing beyond fused MRMS. citeturn10search0turn10search5 | Volume scans typically **4.5–10 min**, native polar coordinates; Level II includes reflectivity, velocity, spectrum width, and dual-pol variables. citeturn10search0turn10search1turn10search5 | NEXRAD Level II archive; native Level II binary/chunked archive. citeturn0search2turn10search1 | Convert to **CfRadial2 NetCDF4** for interchange and optionally **Zarr** for cloud access. citeturn23search0turn23search6 |
| **Lightning** | Total lightning is useful for storm intensification and warning lead time; ProbSevere and PHI also exploit lightning-related information. citeturn0search8turn21search10turn21search15 | GLM native observations are continuous; Level 2 files are for **20 s periods**, with gridded products commonly aggregated at **1 min** or **5 min** and gridded to the **ABI 2 km** fixed grid. Native GLM spatial sampling is about **10 km**. citeturn19search13turn19search15turn19search4turn19search5 | NOAA/NASA GLM in **netCDF4/HDF5**. citeturn19search11turn19search6 | Publish both event/group/flash tables and 2 km gridded FED-style fields. |
| **Sounding/RAOB** | Essential for thermodynamic and kinematic severe-environment diagnosis and for validating reanalysis/NWP-derived environments. IGRA is quality controlled and widely reusable. citeturn1search2turn14search7 | Station-point profiles with station-specific times; in U.S. operational practice, many routine launches are at **00/12 UTC**. citeturn24search9turn24search8 | IGRA is distributed in **plain text**; NWS upper-air archives are official through NCEI. citeturn11search0turn24search0 | Publish native profile tables plus derived sounding features in **Parquet**. |
| **Reanalysis** | Backbone for long-duration environmental context and climatology; essential when soundings are sparse. ERA5 provides hourly global fields and pressure/single-level products. citeturn0search9turn11search1turn9search11 | Commonly **hourly**, regridded **0.25°** for reanalysis products. citeturn11search1turn11search6 | ERA5 in **NetCDF/GRIB**; Copernicus now also supports cloud-oriented access patterns. citeturn11search6turn9search16 | Publish event subsets in **Zarr** with pressure-level and single-level groups. |
| **High-resolution NWP** | Provides true forecast context and supports warning-lead-time tasks. HRRR offers operational 3 km convection-allowing guidance; WoFS provides rapidly updating storm-scale ensemble guidance for 0–6 h hazards. citeturn1search0turn1search12turn20search4turn20search9 | HRRR: **3 km**, hourly updated, with **15 min** outputs early in forecast. WoFS: rapidly updating ensemble with storm-scale 0–6 h hazard focus. citeturn1search6turn20search2turn20search4turn20search8 | HRRR native **GRIB2**, with public **Zarr** archive examples. WoFS availability is more experimental/research oriented. citeturn11search2turn11search7turn20search1 | Use HRRR as the default public CAM baseline; add WoFS where redistributable, or distribute WoFS-derived labels/features if raw fields cannot be mirrored. |
| **Satellite** | GOES ABI provides cloud, moisture, and overshooting-top context before and during convective initiation; joint ABI+GLM is especially valuable. citeturn2search6turn2search11 | ABI spatial resolution is about **0.5–2 km** depending on band; **CONUS every 5 min**, **mesoscale every 30 s**, **full disk about 5–15 min** depending on mode. citeturn10search3turn10search13 | GOES ABI and many L2+ products in **netCDF4** via NODD/NCEI. citeturn2search2turn2search10 | Publish a selected band set and derived multispectral indices on harmonized grids. |
| **Surface observations** | Needed for boundary placement, mesoscale gradients, verification, and real-time environmental diagnosis. ISD and ASOS are official and open; MADIS adds QC flags but some feeds have restrictions. citeturn1search3turn12search4turn12search1turn22search0 | ISD is **hourly/synoptic**; ASOS also offers **1 min** and **5 min** products for many stations. citeturn12search3turn12search4turn12search0turn12search5 | ISD in gzipped **fixed-width ASCII**; ASOS archival text products; MADIS common-format feeds with QC flags. citeturn12search8turn14search1 | Publish station tables in **Parquet/GeoParquet** with QC and provider flags. |
| **Soil moisture and land surface** | Antecedent land-state matters for boundary-layer recovery and convective potential; useful especially for warm-season severe environments. SMAP gives open global soil-moisture products. citeturn1search4turn11search3turn11search8 | Typical benchmark-friendly products are **daily 9 km** or **3-hourly 9 km** depending on level. citeturn11search3turn11search8 | SMAP products in **HDF5**. citeturn11search3 | Publish reprojected land-surface rasters or derived anomalies. |
| **Topography** | Needed for beam blockage interpretation, terrain forcing, and downstream modeling. USGS 3DEP is authoritative for U.S. topography. citeturn1search5turn11search4 | Common national product: **1 arc-second** seamless DEM; finer products also exist. citeturn11search4turn11search9 | USGS 3DEP in **GeoTIFF**. citeturn11search4 | Publish static terrain rasters and derivatives: slope, aspect, relief, local beam-blockage fraction where relevant. |

Two design choices deserve emphasis. First, I would treat **MRMS as the canonical benchmark observation layer**, but I would **not** omit raw radar moments; several high-value warning tasks depend on radial velocity and native polar geometry in ways that mosaics do not fully preserve. Second, I would treat **HRRR as the universal high-resolution NWP layer** and **WoFS as a premium subset** for specialized warning experiments, because HRRR is broadly open and stable as a national archive, whereas WoFS is still more research/prototype oriented. citeturn0search1turn10search5turn1search12turn20search4turn20search1

The benchmark should also include **open event/impact labels**. At minimum, use the **NCEI Storm Events Database**, **SPC severe-weather event archive**, and **NWS warning/CAP products**. Storm Events is essential because it contains official NOAA severe-event records, but it should be accompanied by uncertainty caveats because periods of record and collection procedures differ by event type and through time. CAP and NWS alert products are needed if warning-emulation and lead-time evaluation are core tasks. citeturn2search3turn2search8turn2search4turn16search1turn16search6

## Metadata, provenance, licensing, and quality control

For ESSD, metadata and provenance are not secondary; they are part of the scientific contribution. At minimum, your published benchmark should follow **CF metadata conventions** for gridded scientific variables, **ACDD** for discovery-oriented global metadata, and a **STAC catalog** for searchable asset-level geospatial metadata. That combination is practical and aligns with current Earth-science interoperability practice. citeturn5search0turn5search1turn5search2turn5search17

A concise set of recommended metadata fields is below.

| Metadata scope | Example fields | Why they matter |
|---|---|---|
| **Dataset-level** | `dataset_title`, `version`, `release_date`, `doi`, `license`, `creators`, `institution`, `summary`, `keywords`, `spatial_extent`, `temporal_coverage_start`, `temporal_coverage_end`, `grid_definition`, `crs`, `vertical_reference`, `event_definition`, `processing_pipeline_version`, `software_commit`, `citation`, `contact` | Discovery, citation, and ESSD repository/reuse requirements. citeturn5search1turn13search1turn9search4 |
| **Event-level** | `event_id`, `region_cluster_id`, `bbox`, `storm_mode`, `primary_hazard`, `ci_time`, `mature_time`, `decay_time`, `warning_ids`, `report_ids`, `label_confidence`, `data_gaps`, `split_assignment` | Prevents ambiguity in event construction and benchmark leakage. |
| **Object-level** | `object_id`, `parent_id`, `track_id`, `merge_split_flag`, `area_km2`, `max_refl`, `max_mesh`, `rotation_proxy`, `prob_hail`, `prob_wind`, `prob_tor`, `lifetime_min` | Enables tracking, warning and object-centric ML tasks. |
| **Asset-level** | `source_dataset`, `source_doi`, `source_provider`, `native_format`, `original_filename`, `checksum`, `acquisition_time`, `valid_time`, `latency`, `reprojection_method`, `resampling_kernel`, `qc_mask_name`, `quality_flag`, `units`, `scale_factor`, `add_offset` | Supports full lineage and reproducibility. citeturn5search4turn5search0turn5search2 |

On licensing, the safest policy is to treat licensing as **asset-specific**, not assumed homogeneous. NOAA data disseminated through NODD are generally open to the public, but NOAA explicitly asks for attribution, disallows implied endorsement, and asks that modified products not be represented as original NOAA data. NASA Earthdata follows a strong open-data policy with unrestricted access for Earth science data and metadata. USGS data are generally U.S. public domain. Copernicus moved CDS/ADS/EWDS products to **CC-BY** in July 2025, which means attribution obligations must be preserved in downstream redistribution. By contrast, parts of **MADIS** have provider restrictions, including limitations on redistributing some mesonet feeds. citeturn5search3turn9search0turn9search2turn9search7turn9search1turn22search0

The implication for your benchmark is important: if you want a **fully open redistributable release**, you should either  
(a) include only assets whose upstream terms allow redistribution, or  
(b) publish the benchmark in two layers: an **open derivative layer** containing your labels, metadata, derived features, and openly redistributable source subsets, plus a **manifest-and-recipes layer** that reconstructs non-redistributable components from official sources. For a mixed-source derivative, **CC-BY 4.0** is usually safer than CC0, because it is compatible with attribution expectations from Copernicus and many scientific-data communities. citeturn9search1turn9search9turn22search0turn13search1

The main ethical issues are therefore practical rather than sensitive-human-data issues:  
you should avoid redistributing restricted MADIS feeds; avoid exposing exact private-station metadata where provider terms are unclear; document uncertainty and collection biases in reports/warnings; and avoid language that implies NOAA/NASA/ECMWF endorsement of your derivative benchmark. Storm Events labels also require caution because official collection practices differ over time and by event category. citeturn22search0turn22search11turn9search13turn2search8

Preprocessing and quality control should be strict and explicit. For radar/MRMS, I would recommend preserving native files, then applying standardized harmonization with published masks for data voids, beam blockage, clutter contamination, bright band, anomalous propagation, and interpolation/resampling ranges. MRMS itself already applies substantial QC and multi-sensor fusion, including mitigation of non-meteorological echoes and meteorological artifacts; you should propagate that information rather than obscuring it. citeturn14search0turn14search5turn0search6

For surface data, ingest official QA/QC flags from MADIS or source archives and keep them in the public tables instead of dropping “bad” observations silently. For IGRA, preserve the original profile and station metadata because vertical extent and record length vary by station and time. For satellite, retain source quality flags and L2/L3 data-quality fields. For lightning, document clearly that GLM measures total lightning but cannot cleanly distinguish lightning types in the gridded products. citeturn14search1turn14search7turn11search0turn14search14turn19search9

Finally, temporal alignment must be handled as a first-class scientific issue. Do **not** force all sources to a single synthetic timestamp without preserving native valid times and latency. Instead, publish both native-time assets and a harmonized analysis timeline, and record the interpolation window used to populate each harmonized step. This is one of the easiest places where otherwise good multimodal benchmarks become scientifically unreliable. The long-duration architecture you propose will be strongest if it explicitly records these alignment choices at the asset level. citeturn5search4turn13search9

## Benchmark tasks, baselines, metrics, and split strategy

A benchmark intended for severe-convective science and warning applications should not be limited to one nowcasting task. It should define a **task suite** spanning diagnosis, prediction, object evolution, and warning support.

| Benchmark task | Typical target | Recommended baselines | Recommended metrics |
|---|---|---|---|
| **Pre-convective environment classification** | Severe/non-severe event, or hazard family within next 2–6 h | Logistic regression; random forest; gradient-boosted trees using sounding/reanalysis/CAM features; climatology and persistence baselines. RF/GBT/logistic baselines are well established in severe-weather guidance work. citeturn18search3turn18search2turn18search0 | ROC-AUC, PR-AUC, Brier score, reliability, calibration error, POD/FAR at operational thresholds. citeturn6search1turn6search3turn6search15 |
| **Convective initiation detection/forecast** | CI occurrence in grid/object/time window | Persistence, optical flow, threshold rules from satellite cooling/lightning/radar, ConvLSTM-style multimodal models, U-Net hazard maps. citeturn17search0turn17search1turn20search7 | POD, FAR, CSI/ETS, lead-time distribution, spatial tolerance/FSS. citeturn6search15turn6search0turn7search6 |
| **Reflectivity or hazard-field nowcasting** | Future MRMS reflectivity, MESH, FED, or severe-probability fields at 0–120 min | Eulerian persistence; optical flow/rainymotion; ConvLSTM; TrajGRU; PredRNN; Earthformer/transformers. citeturn17search0turn17search1turn17search4turn17search12turn17search13 | MAE/RMSE for continuous fields, CSI at thresholds, FSS, MODE, SAL. citeturn7search6turn7search0turn7search1 |
| **Segmentation** | Convective object masks, severe-proxy masks, warning polygons | Thresholded MRMS rules; U-Net; feature pyramid networks; object-based postprocessing with morphology. ProbSevere-like object extraction is a practical baseline philosophy. citeturn21search15turn21search5 | IoU, Dice/F1, object POD/FAR, MODE, boundary metrics. citeturn7search0turn7search9 |
| **Tracking** | Storm-object associations across time, merger/split graphs | Centroid nearest-neighbor; overlap-based association; Hungarian assignment; ProbSevere-style object-track evolution; graph neural nets as advanced baseline. citeturn21search15turn21search5 | Track continuity, association accuracy, lifetime error, IDF1/HOTA if adopted, initiation and termination timing error. citeturn7search3turn7search18 |
| **Warning and lead-time evaluation** | Deterministic or probabilistic warning decisions/polygons | Threshold rules on MRMS/GLM/satellite; logistic/RF/GBT object models; HRRR/WoFS ML guidance; ProbSevere and PHI-style probabilistic updates. citeturn18search3turn18search9turn21search15turn21search0 | Mean/median lead time, fraction warned in advance, POD, FAR, warning duration, geospatial verification of warnings, Brier score and reliability for probabilities. citeturn8search3turn8search10turn6search3turn6search13 |

For deterministic detection and warning tasks, the core categorical metrics should be **POD**, **FAR**, **CSI/TS**, and often **ETS**, because these are standard for rare severe events and directly interpretable by the weather community. For probabilistic guidance, the benchmark should require **Brier score**, **Brier skill score**, **reliability diagrams**, and where continuous predictive distributions are used, **CRPS**. For gridded fields, add **FSS** so models are not unfairly punished for small spatial displacements. For object-rich products, include **MODE** or **SAL** because they are more informative than pixelwise scores when storms merge, split, or are slightly displaced. citeturn6search15turn6search0turn6search3turn6search8turn7search6turn7search0turn7search1

For baseline algorithms, I recommend releasing **two tiers**. The first tier should be simple, transparent baselines: persistence, optical flow, threshold rules based on MRMS or environment, logistic regression, random forest, and gradient-boosted trees. The second tier should be modern sequence models: ConvLSTM, TrajGRU, PredRNN, U-Net-like multimodal encoders, and transformer models such as Earthformer. This two-tier strategy is important for ESSD because data descriptors should show accessible value to the community, not only leaderboards for large labs. citeturn17search0turn17search1turn17search4turn17search12turn17search13turn18search3

Your split design matters enormously. The benchmark should **never** split at the individual-frame level, because that leaks storm evolution and nearby environments across train and test. The default split should be **event-level** and ideally also **year-level** or **season-block-level**. In addition, publish at least two challenge splits: a **leave-one-region-cluster-out** split for geographic generalization and a **leave-one-season/year-out** split for temporal generalization. If you later add storm-mode annotations, a **leave-one-mode-out** or **rare-hazard holdout** split would be extremely valuable for robustness testing. This principle is consistent with the careful evaluation philosophy behind standardized weather benchmarks such as WeatherBench 2. citeturn3search1turn3search11

## Existing open datasets, gap analysis, and improvement opportunities

The current landscape strongly suggests that your dataset would be complementary rather than redundant.

| Existing open dataset | What it offers | Limitation relative to your proposal | How your dataset can differ |
|---|---|---|---|
| **SEVIR** citeturn3search0turn3search15 | Curated, aligned storm imagery; radar, satellite, lightning; over 10,000 events; about 384 km by 384 km and 4-hour sequences. | A landmark benchmark, but it is still fundamentally an imagery/event-sequence dataset; it does not center the full long-duration storm environment and warning-evaluation workflow. | Add longer pre-CI to post-decay windows, richer environmental variables, warning/report linkages, and object/lead-time tasks. |
| **MYRORSS** citeturn4search3turn4search8 | Multi-year radar reanalysis with near-storm environmental analyses; 1998–2011; 0.01° grid; valuable historical severe-weather coverage. | Not a modern benchmark release with multimodal long-duration packaging, canonical task definitions, or standardized public ML splits. | Modernize the benchmark layer around MRMS-era observations, satellite/lightning, and explicit benchmark protocols. |
| **Operational MRMS archive** citeturn0search1turn0search11turn10search7 | Authoritative real-time/open multi-radar fused products for warnings and severe-weather analysis. | A product archive, not a benchmark: no event packages, no object tracks, no standardized labels/splits, no environmental context bundle. | Turn MRMS from a product source into a reproducible research benchmark. |
| **WeatherBench 2** citeturn3search1turn3search11 | Excellent open framework for like-for-like evaluation of global medium-range weather forecasting. | Different problem class: global, medium-range, not storm-object/severe-warning centric. | Fill the storm-scale severe-convection niche with short-range multimodal event packages. |
| **HKO-7** citeturn3search13turn3search18 | Classic radar-based nowcasting benchmark from Hong Kong, with strong methodological importance. | Local/regional and radar-centric; not a multimodal severe-environment/warning database. | Provide a broader severe-weather benchmark with environment, warnings, objects, and reports. |
| **MeteoNet** citeturn4search0turn4search5turn4search15 | Open weather dataset from Météo-France with radar, stations, satellite, NWP, and relief masks. | Highly useful, but not specifically framed around severe-convective event evolution and warning lead-time evaluation. | Use a severe-weather event architecture and standardized warning/lead-time tasks. |
| **TORUS field data** citeturn3search4turn3search14 | Rich process-study observations for supercells and tornadoes. | Campaign-scale, intensive-observation, not broad/population-scale benchmark design. | Offer scale, standardization, and benchmarkability rather than only high-detail cases. |
| **Storm Events / SPC archives / NWS CAP alerts** citeturn2search3turn2search4turn16search6 | Authoritative event records and warning products. | Labels only; not co-registered multimodal observations and forecasts. | Fuse them into event packages with uncertainty-aware label linkage. |

The clearest novelty in your architecture is the combination of **event-centric storm evolution**, **storm-environment context**, and **warning-relevant labels** in one openly released, benchmark-ready system. That is a stronger proposition than “another severe weather dataset,” because most existing resources serve either **process study**, **imagery prediction**, **product dissemination**, or **event cataloging**, but not all four at once. citeturn3search0turn3search1turn4search8turn2search3

I do think your proposed concept **fills a genuine gap**, but only if you make a few design improvements.

The most important improvement is to add **explicit labeling layers at multiple scales**. I would strongly recommend releasing:  
(a) **event labels** such as tornado / severe hail / severe wind / non-severe;  
(b) **object labels** such as supercell / QLCS / multicell / merger / bowing segment where feasible;  
(c) **pixel/grid labels** such as future MRMS exceedance or warning probability; and  
(d) **decision labels** linking warnings, reports, and lead time. PHI and ProbSevere show the value of probabilistic storm-based products updated frequently, and your benchmark will be far more useful if it supports that style of evaluation. citeturn21search0turn21search15turn21search2

The second improvement is to strengthen the **environmental variable set**. At minimum, derive and store sounding-style diagnostics that operational severe-weather analysts actually use: CAPE, CIN, LCL/LFC, lapse rates, precipitable water, 0–6 km bulk shear, storm-relative helicity, and composite severe indices such as STP or supercell-related parameters. SHARPpy and SPC severe-parameter documentation provide a practical template for feature design that matches forecaster reasoning. citeturn15search2turn15search8turn15search0turn15search18

The third improvement is to publish **temporal alignment metadata explicitly**. This is especially important because your windows are intentionally longer than standard nowcasting windows. If an environmental field is hourly, a sounding is 00/12 UTC, radar is every few minutes, and satellite mesoscale scans are every 30 s, users need machine-readable records of how each harmonized step was populated. Without that, model performance can look better than it really is due to hindsight leakage or overly aggressive interpolation. citeturn10search3turn24search9turn11search6turn5search4

The main limitations to acknowledge openly in the paper are these. The benchmark will be **U.S.-centric** if MRMS is the foundation. Raw radar and long-duration multimodal packaging will be storage-heavy. Severe-event reports and warnings are imperfect truth, with temporal and spatial uncertainty and evolving collection practices. Soundings are sparse relative to storm evolution. GLM is valuable but does not distinguish lightning types in the way some users may assume. None of these are fatal, but all should be declared clearly in an ESSD-style paper. citeturn0search11turn2search8turn14search7turn19search9

## ESSD-style data descriptor package and publication checklist

An ESSD paper will be strongest if it is written as a **data descriptor**, not as a model paper with a small dataset section. ESSD requires public data availability with a repository path to DOI, emphasizes traceability and reproducibility, allows review-time temporary access but requires a functional DOI upon acceptance, and expects the data and software cited in the paper to appear in the manuscript and reference list. Repository landing pages and metadata should be available in English. citeturn9search4turn13search6turn13search3turn13search1

The prioritized checklist below is the package I would prepare before submission.

| Priority | Item to prepare | Why it matters for ESSD | Minimum deliverable |
|---|---|---|---|
| **Highest** | **Public repository and versioning plan** | ESSD requires reliable public access and DOI-based citation. citeturn9search4turn13search1turn13search5 | Versioned repository, review-access link, final DOI plan, English landing page. |
| **Highest** | **Data model and provenance document** | Reviewers need to understand event construction and processing lineage. citeturn5search4turn13search9 | Figure of pipeline, schema tables, processing graph, checksums, software commit hashes. |
| **Highest** | **Licensing matrix** | Mixed-source benchmarks fail review if redistribution terms are unclear. citeturn9search1turn22search0turn5search3 | Table listing every source, license/restriction, redistribution status, attribution text. |
| **Highest** | **Canonical split files** | Prevents leakage and makes the release a true benchmark. | `train/validation/test` event lists plus geographic and temporal challenge splits. |
| **Highest** | **Reference analyses** | ESSD data papers must demonstrate value and quality, not just existence. citeturn5search14turn13search16 | Exploratory statistics, event climatology, at least two case studies, baseline results. |
| **High** | **Open example code** | Reduces friction and improves reuse. citeturn13search3 | Notebooks/scripts for discovery, reading, plotting, and one baseline model. |
| **High** | **Metadata tables** | Makes the benchmark machine-readable and reviewable. citeturn5search0turn5search1turn5search2 | Dataset-, event-, object-, and asset-level field dictionaries. |
| **High** | **Quality-control documentation** | Reviewers will ask how raw observations were filtered or aligned. | QC flags, missing-data policy, temporal alignment rulebook, uncertainty fields. |
| **Medium** | **Visualization suite** | ESSD readers expect clear data-overview figures. | Coverage maps, temporal histograms, variable distributions, case-study panels, event timelines. |
| **Medium** | **Software DOI and environment** | Helps reproducibility and citation hygiene. citeturn13search3turn13search13 | Archived code release, environment file/container digest, software citation. |

The paper itself should include a compact but convincing set of **value-demonstration analyses**. I would include:  
an exploratory data overview by season, region, hazard, and event duration;  
an event climatology showing the distribution of storm modes and severe outcomes across space and time;  
at least two detailed case studies showing why long-duration environmental context matters;  
at least one nowcasting baseline benchmark;  
and one warning/lead-time evaluation example linking observations, alerts, and impacts. That combination matches the kind of “reusable data product” story ESSD handles well. citeturn5search14turn13search16turn13search10

For figures, I would prioritize the following:

| Figure type | What to show |
|---|---|
| **Coverage map** | Spatial distribution of event clusters, radar coverage, sounding sites, and topographic background. |
| **Temporal histogram** | Number of events by month, year, hazard type, and event duration. |
| **Variable-distribution panel** | CAPE, shear, MRMS reflectivity percentiles, GLM flash rates, soil-moisture anomalies, warning lead times. |
| **Case-study composite map** | MRMS composite reflectivity, radar velocity inset, GLM FED, selected ABI channels, surface boundaries, warnings/reports. |
| **Event timeline** | Pre-CI environment, CI, severe phase, warning issuance, decay. |
| **Task schematic** | How event packages map to detection, segmentation, tracking, nowcasting, and warning tasks. |

An illustrative event timeline suitable for the paper is below.

```mermaid
timeline
    title Example long-duration event package
    Pre-event environment : T-6 h to T-2 h : reanalysis, HRRR, surface obs, soil moisture, topography
    Pre-CI monitoring : T-2 h to T0 : ABI, GLM, surface boundaries, low-level convergence
    Convective initiation : T0 : first sustained radar or MRMS convective object and/or lightning threshold
    Growth and organization : T0 to T+2 h : radar moments, MRMS severe proxies, object tracking
    Warning phase : T+1 h to T+4 h : NWS CAP alerts, reports, PHI or ProbSevere-style probabilities
    Maturity and upscale growth : T+2 h to T+6 h : hail, wind, rotation, mergers and splits
    Decay and trailing stratiform phase : T+6 h onward : anvil and system dissipation
```

And an illustrative task flowchart suitable for a benchmark overview figure is below.

```mermaid
flowchart TD
    A[Long-duration event package] --> B[Environment diagnosis]
    A --> C[CI detection]
    A --> D[Reflectivity and hazard nowcasting]
    A --> E[Segmentation]
    A --> F[Tracking]
    A --> G[Warning emulation]
    G --> H[Lead-time and calibration evaluation]
    D --> H
    E --> H
    F --> H
```

If you want the manuscript to be especially persuasive for ESSD reviewers, present the benchmark as a **community resource with immediate reuse**. That means including a “getting started” example, split files, baseline code, clear access instructions, one DOI for the dataset, and ideally a separate DOI for the release code. ESSD’s policies on data access, repository criteria, and data citation make these items more than “nice to have.” citeturn9search4turn13search1turn13search3turn13search6

In summary: your proposed architecture is scientifically meaningful, publication-suitable, and likely novel enough **if** you frame it as an **open, provenance-rich, warning-relevant, multimodal severe-convection benchmark**, rather than just a collection of files around storms. The biggest opportunity is the combination of **MRMS/raw radar**, **long-duration environment**, and **warning/report evaluation**. The biggest practical threats are **license mixing, label ambiguity, and hidden temporal leakage**. If you solve those cleanly, the resulting dataset would stand apart from existing open resources and would be well positioned for an ESSD-style data descriptor. citeturn3search0turn4search8turn21search15turn13search16turn13search9