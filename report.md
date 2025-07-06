# Report: Dynamic Parking Pricing with Pathway

## 1. Project Overview

This project demonstrates a dynamic parking pricing system implemented using a Jupyter Notebook and the Pathway stream processing library. The system analyzes simulated real-time parking data, including occupancy, capacity, and other factors, to calculate and visualize a dynamic price. The price is designed to adjust based on daily demand fluctuations, represented by occupancy changes relative to the parking lot's capacity.

The primary deliverable is a Jupyter Notebook (`Untitled0.ipynb`) that showcases this pipeline, along with a `requirements.txt` for dependency management.

## 2. Steps Taken and Justification

The development process involved several stages:

1.  **Initial Understanding and Exploration:**
    *   **Action:** The primary file, `Untitled0.ipynb`, and the initial `README.md` were examined.
    *   **Justification:** To understand the existing codebase, its functionality, dependencies, and the data being processed. This initial exploration identified the core logic: data loading, preprocessing with pandas, stream simulation with Pathway, a windowed aggregation for price calculation, and visualization with Bokeh/Panel.
    *   **Key Findings:**
        *   The notebook uses `pandas` for initial data manipulation.
        *   `pathway` is used for stream processing and dynamic calculations.
        *   `bokeh` and `panel` are used for visualization.
        *   A critical observation was the data type mismatch for the `Traffic` column in the `ParkingSchema` (defined as `float` but `str` in the preprocessed data).

2.  **Documentation of the Jupyter Notebook:**
    *   **Action (Attempted):** The plan was to add comprehensive markdown cells throughout the notebook to explain each step, the data schema, assumptions, and the pricing logic.
    *   **Justification:** To make the notebook self-explanatory and easier to understand for other users or for future reference.
    *   **Challenges & Outcome:** Direct modification of the `.ipynb` file using the available tools (`replace_with_git_merge_diff` and `overwrite_file_with_block`) proved highly problematic. The complex JSON structure of the notebook, especially cells with large embedded HTML/JavaScript for Bokeh/Panel outputs, led to consistent tool failures and risked corrupting the notebook.
    *   **Revised Action:** The `ParkingSchema` definition in the notebook was successfully corrected to change `Traffic: float` to `Traffic: str`. Some introductory markdown cells explaining the project and initial setup steps were also successfully added in earlier iterations before the tool limitations became a persistent blocker for more extensive changes. Due to the risk of corruption, further direct embedding of extensive markdown was halted. This report serves as the comprehensive documentation.

3.  **Code Review and Refactoring (within the Notebook):**
    *   **Action:** The Python code within the notebook cells was reviewed for clarity, correctness, and potential minor efficiencies.
    *   **Justification:** To ensure the code is robust, understandable, and performs its intended function correctly.
    *   **Outcome:**
        *   The most critical refactoring was the correction of `ParkingSchema.Traffic` to `Traffic: str`.
        *   Variable names were found to be generally clear for the scope of the example.
        *   The logic for data processing and price calculation, while simple, was deemed correct for a demonstration.
        *   No major efficiency bottlenecks were identified for the scale of this demo. Colab-specific code (like `files.upload()`) was kept as the notebook is intended for that environment but noted for local execution.

4.  **Visualization Enhancements (Attempted within Notebook):**
    *   **Action (Attempted):** The plan was to enhance the Bokeh plot by adding axis labels and interactive hover tooltips.
    *   **Justification:** To make the visualization more informative and user-friendly.
    *   **Outcome:** Similar to the markdown documentation, attempts to modify the visualization code cell using the available tools were unsuccessful due to the notebook's complex structure. The existing visualization is functional for showing the price trend.

5.  **Creation of `requirements.txt`:**
    *   **Action:** A `requirements.txt` file was created.
    *   **Justification:** To list all necessary Python packages for reproducibility and easy setup of the environment.
    *   **Content:**
        ```
        pandas>=1.3,<2.3
        pathway
        bokeh>=2.4,<3.1
        panel>=0.14,<1.4
        ```
        Version specifiers were chosen to allow for recent compatible versions while providing some level of constraint.

6.  **Updating `README.md` (This Report):**
    *   **Action (Attempted for `README.md`):** The initial plan was to update the existing `README.md` file extensively.
    *   **Justification:** To provide users with a comprehensive guide to the project.
    *   **Outcome:** Tooling issues also prevented a full update of the `README.md` file directly in the repository. Therefore, this separate `report.md` file is created to fulfill that documentation purpose. The `README.md` in the repository was reverted to a very basic version.

## 3. Models and Libraries Used

*   **Pandas:** Used for initial data loading, cleaning, and preprocessing (e.g., combining date/time columns, renaming columns, selecting relevant features).
*   **Pathway:** The core library for the stream processing pipeline.
    *   `pw.Schema`: Used to define the structure of the data being streamed.
    *   `pw.demo.replay_csv`: Used to simulate a data stream from a CSV file.
    *   Table methods like `.with_columns()`, `.windowby()`, `.reduce()`: Used for data transformation, temporal windowing, and aggregation.
    *   `pw.this`: Used to refer to columns within Pathway expressions.
    *   `pw.reducers`: Used for aggregation functions like `max` and `min`.
    *   `pw.temporal.tumbling`, `pw.temporal.exactly_once_behavior`: Used for defining windowing strategies.
*   **Bokeh:** Used for creating the interactive plot of the dynamic parking price.
    *   `bokeh.plotting.figure`: To create the plot canvas.
    *   `fig.line()`, `fig.circle()`: To add glyphs to the plot.
*   **Panel:** Used to display and serve the Bokeh plot within the Jupyter Notebook environment and make it reactive to Pathway updates.
    *   `pn.extension()`: To initialize Panel.
    *   `delta_window.plot()`: Pathway's integration with Panel/Bokeh to create a live-updating plot from a Pathway table.
    *   `pn.Column().servable()`: To make the plot visible.

No machine learning models were used in this iteration; the pricing model is rule-based.

## 4. Demand Function, Assumptions, and Price Dynamics

### Demand Function

The notebook does not implement an explicit, formal econometric demand function. Instead, **demand is implicitly represented by the `Occupancy` data and its daily fluctuations.**

The dynamic component of the price is calculated as:
`price_adjustment = (occ_max_daily - occ_min_daily) / capacity`

The final price is `base_price + price_adjustment`.

This means:
*   A higher difference between the maximum and minimum occupancy in a day (suggesting a period of high demand and a period of lower demand) leads to a larger price adjustment.
*   If daily occupancy is very stable (e.g., always 50% full, or always 90% full), `occ_max - occ_min` would be small, leading to a small price adjustment.
*   The division by `capacity` normalizes this fluctuation, meaning the same absolute fluctuation in occupancy has a greater impact on price for smaller lots than for larger lots.

This can be seen as a proxy for responsiveness to perceived demand volatility. High volatility (large swing from low to high occupancy) implies the lot is experiencing both peak and off-peak demand periods, and the price adjusts upwards to capture more value during perceived peak times.

### Assumptions

*   **Data Source:** The `dataset.csv` is assumed to be representative of real-world parking data streams for the purpose of this demonstration.
*   **Data Quality:** Basic cleaning is performed (timestamp parsing, renaming), but the input data is largely taken as is for other quality aspects.
*   **Capacity:** The model uses `pw.reducers.max(pw.this.Capacity)` within each daily window. This implies an assumption that either:
    *   The capacity of a parking entity is constant throughout the day.
    *   Or, if capacity does change (e.g., some spots taken offline), using the maximum observed capacity for that day is an acceptable proxy for the pricing calculation. For a more granular system, capacity would ideally be a fixed characteristic of each `ParkingLotID`.
*   **Base Price:** The base price of $10 is an arbitrary starting point for this demonstration.
*   **Competitor Price:** A dummy `CompetitorPrice` of $10.0 is added during preprocessing. This value is **not currently used** in the dynamic pricing calculation in this version of the notebook.
*   **Single Stream Processing:** The current windowing and aggregation logic processes all data as a single stream. It does not differentiate between individual `ParkingLotID`s when calculating `occ_max`, `occ_min`, and `cap` for the daily price. This means the calculated price is an aggregate across all data, not per-lot.
*   **Stream Simulation:** `pw.demo.replay_csv` effectively simulates a stream for this demonstration.

### How Price Changes with Demand and Competition

*   **Demand:**
    *   As described in the "Demand Function" section, the price directly responds to the *range* of daily occupancy. A larger difference between `occ_max` and `occ_min` (normalized by capacity) leads to a higher price for that day.
    *   This implies that if a parking lot experiences both very low and very high utilization within the same day, the system interprets this as volatile demand and increases the price more significantly than if the lot had consistently moderate or consistently high/low occupancy.

*   **Competition:**
    *   In the current version of the notebook, **the `CompetitorPrice` is not factored into the dynamic price calculation.** The `price` is solely determined by the base price and the occupancy-based adjustment.
    *   **To make the price change with competition**, the pricing formula in Cell 6:
        ```python
        price=10 + (pw.this.occ_max - pw.this.occ_min) / pw.this.cap
        ```
        would need to be modified. For example, it could be:
        *   A blend: `price = alpha * (calculated_occupancy_price) + (1-alpha) * competitor_price_feed`
        *   A cap/floor: `price = min(calculated_occupancy_price, competitor_price_feed + margin)`
        *   A more complex function considering `pw.this.CompetitorPrice` (or its daily average/max/min).
    *   This would require ensuring `CompetitorPrice` is available in the `delta_window` table, potentially through aggregation if it varies.

## 5. Visualizations and Graphs

The notebook (`Untitled0.ipynb`) generates one primary visualization:

*   **"Pathway: Daily Parking Price" Bokeh Plot:**
    *   **Type:** Line chart with circle markers.
    *   **X-axis:** Date/Time (representing the end of each daily window).
    *   **Y-axis:** Calculated dynamic price in dollars.
    *   **Purpose:** To show the trend of the calculated parking price over the period of the dataset.
    *   **Interactivity (Standard Bokeh):** Includes pan, zoom, save, and reset tools.
    *   **Live Updates (Simulated):** When `pw.run()` is executed, this plot is designed to update as Pathway processes the data from the CSV stream, showing how the price would change if this were a live data feed.

(If this report were generated *after* a successful visualization enhancement, it would also describe the added axis labels and hover tooltips.)

## 6. Conclusion and Future Work

This project successfully demonstrates the use of Pathway for building a simple dynamic parking pricing system. It showcases data ingestion from a CSV, schema definition, stream processing with windowed aggregations, and real-time visualization of the calculated price.

The current pricing model is a basic heuristic. Significant improvements and future work could involve:

*   **Integrating Real Data:** Connecting to live parking sensors and competitor pricing APIs.
*   **Advanced Pricing Algorithms:**
    *   Incorporating more features like `Traffic`, `IsSpecialDay`, `VehicleType`.
    *   Using machine learning for demand forecasting.
    *   Implementing more sophisticated competitive pricing strategies.
*   **Per-Lot Pricing:** Modifying the Pathway pipeline to group by `ParkingLotID` for individualized pricing.
*   **Enhanced Visualizations & Dashboarding:** Creating a more comprehensive Panel dashboard with multiple plots, controls, and KPIs.
*   **Deployment:** Packaging the Pathway application for deployment in a production environment.
*   **Robust Error Handling and Logging:** Adding comprehensive error management for production readiness.

Despite the tooling limitations encountered during the documentation phase for direct notebook modification, the core functionality and the corrected data schema are in place, providing a solid foundation for further development.
