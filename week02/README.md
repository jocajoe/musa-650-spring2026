## Week 2: Remote Sensing Data Sources and IO

### Lecture: 
Slides: [TODO]

### Lab: Landsat Data Discovery and Access with STAC
In this lab, you will retrieve Landsat imagery for Singapore and produce a cloud-free composite image. You will work through the full workflow of discovering, inspecting, loading, and exporting satellite data using modern cloud-native tools.

You will not be given any code. Instead, you will work from documentation and tutorials to figure out how to accomplish each step. This mirrors real-world practice: the tools change constantly, but the workflow patterns remain consistent. Your job is to learn to navigate documentation and adapt examples to your specific problem.

**Study area**: Singapore  
**Time period**: 2025 (January through December)  
**Dataset**: Landsat Collection 2 Level-2 (`landsat-c2-l2`)  
**Goal**: Produce and export a cloud-free true-color composite

#### Reference Materials

This tutorial is based on the following resources. You are not limited to using these, but they will be helpful in pulling together the code you need to complete the lab.

[STAC Spec Tutorial: Reading STAC with Planetary Computer](https://stacspec.org/en/tutorials/reading-stac-planetary-computer/)
[Project Pythia: Data Ingestion (Geospatial)](https://projectpythia.org/landsat-ml-cookbook/notebooks/data-ingestion-geospatial/)
[Planetary Computer: Cloudless Sentinel-2 Mosaic](https://github.com/microsoft/PlanetaryComputerExamples/blob/main/tutorials/cloudless-mosaic-sentinel2.ipynb)

#### Part 1: Understand what data is available and how STAC organizes it.

1. Connect to the Planetary Computer STAC catalog using `pystac_client`.

2. Find the Landsat Collection 2 Level-2 dataset (`landsat-c2-l2`). Per [Project Pythia](https://projectpythia.org/landsat-ml-cookbook/notebooks/data-ingestion-geospatial/):
> We’ll use the landsat-c2-l2 dataset, which stands for Collection 2 Level-2. It contains data from several landsat missions and has better data quality than Level 1 (landsat-c2-l1). Microsoft Planetary Computer has descriptions of Level 1 and Level 2, but a direct and succinct comparison can be found in this community post, and the information can be verified with USGS.
Explore the metadata by simply examining the collection object. What's the description? STAC version? Check out the links, the ID, the license, keywords, etc.

3. List the assets available in this collection. What bands are available? What other assets (e.g., QA bands, thumbnails) are included?

#### Part 2: Query for imagery over your study area and verify the results make sense.

4. Search for Landsat scenes within your bounding box for 2025. Filter for scenes with less than 50% cloud cover. (Singapore is cloudy—you may need to adjust this threshold to get enough scenes.)

5. Preview your results. How many scenes did you get? Look at some thumbnails to verify you're getting what you expect.

6. Visualize the STAC item footprints on a map. Do the tiles cover your entire bounding box? Are there gaps?

7. Plot cloudiness over time. Create a plot showing cloud cover percentage for each scene across the year.

#### Part 3: Load the imagery into an xarray DataArray and understand its structure.

8. Set up a Dask cluster for parallel processing. Use `odc.stac.configure_rio(cloud_defaults=True, client=client)` to configure rasterio for efficient cloud COG access. This step is critical—without it, loading data from the cloud will be extremely slow.

9. Select the bands you need: red, green, blue, nir08 (for false color), and `qa_pixel` (for cloud masking).

10. Load the data using `odc.stac.load`. Use 30m resolution and appropriate chunking for Dask (e.g., `chunks={"time": -1, "x": 128, "y": 128}`). What modifications did you have to make to load the data efficiently (e.g., bounding box size, resolution, chunk size, Dask configuration)?

11. Examine the resulting dataset. What are its dimensions? What's the CRS? How much data would you be loading if you called `.compute()` right now?

#### Part 4: Combine multiple scenes into a single cloud-free image.

12. Create a cloud mask using the `qa_pixel` band. Bits 3 and 4 indicate cloud and cloud shadow respectively. Mask out cloudy pixels so they become NaN.

13. Create a median composite across all time steps. Because cloudy pixels are NaN, the median will ignore them and select from clear observations only. Load the composite using Dask (`.compute()`)

#### Part 5: Visualize and export your results.

14. Plot individual bands (red, green, blue, NIR) as separate grayscale images.

15. Plot a true-color composite and a false-color composite. You'll need to stack the RGB bands and normalize the values appropriately.

16. Export your true-color composite as a Cloud-Optimized GeoTIFF (COG). Open in QGIS or ArcGIS and compare it to a satellite basemap of your choice.


