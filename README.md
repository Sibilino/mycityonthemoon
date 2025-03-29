Overview
The web application will:

    Display a high-resolution map of the Moon sourced from the USGS Planetary Web Mapping Services (PWMS) via its WMTS API.
    Allow users to pan and zoom using the Leaflet JavaScript library.
    Include a search input to select an Earth city, retrieving its boundary shape from the Nominatim API.
    Overlay the city shape on the Moon map, scaled to represent its physical size relative to the Moon, with transparency, and fixed at the map’s center.
    Enable users to move the underlying Moon map to compare the city size with lunar features.
    Be hosted as a static web page on GitHub Pages.

This plan uses online APIs and provides exact steps, leaving coding details to the executing LLMs unless specified.
Step-by-Step Implementation Plan
Step 1: Set Up the Project Structure

    Create a new directory named moon-city-overlay on your local machine.
    Inside the directory, create a single file named index.html to contain all HTML, CSS, and JavaScript for the application.
    Open index.html in a text editor to begin adding the structure and code as instructed below.

Step 2: Build the HTML Structure

    Add a basic HTML structure to index.html with:
        A <!DOCTYPE html> declaration and <html>, <head>, and <body> tags.
        In the <head>:
            Set the title to "Moon City Overlay".
            Include the Leaflet CSS from the CDN at https://unpkg.com/leaflet@1.7.1/dist/leaflet.css.
            Include the Leaflet JavaScript from the CDN at https://unpkg.com/leaflet@1.7.1/dist/leaflet.js.
        In the <body>:
            Add a <div> with id="map", styled inline with width: 100%; height: 600px; for the map display.
            Add an <input> element with type="text", id="city-input", and placeholder="Enter city name" for city selection.
            Add a <button> with id="search-btn" and text "Search" to trigger the city search.
            Add a <button> with id="clear-btn" and text "Clear" to remove the city overlay.
            Add a <script> tag at the end of the body for the JavaScript logic.

Step 3: Initialize the Leaflet Map with the Moon Layer

    Inside the <script> tag, write JavaScript to:
        Initialize a Leaflet map with the id="map" element, centered at coordinates [0, 0] (the Moon’s center in equirectangular projection), and set the initial zoom level to 2.
        Add a tile layer using the USGS PWMS WMTS API with the exact URL template: https://planetarymaps.usgs.gov/cgi-bin/mapserv?map=/maps/moon/moon_wmts.map&SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=LRO_WAC_GLOBAL&STYLE=default&TILEMATRIXSET=Moon_Equirectangular&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&FORMAT=image/png.
        Set the tile layer’s attribution to "USGS Astrogeology Science Center".
        Add the tile layer to the map so it displays the Moon’s surface.

Step 4: Implement City Search and Overlay Logic

    Declare a global variable cityLayer and initialize it to null outside any function to track the city overlay layer.
    Add an event listener to the "Search" button (id="search-btn") that:
        Retrieves the city name entered in the input field (id="city-input").
        Makes an HTTP request to the Nominatim API using the URL: https://nominatim.openstreetmap.org/search?q=${cityName}&format=json&polygon_geojson=1, where ${cityName} is the user’s input.
        Parses the JSON response and checks if there are results (i.e., the response array has length > 0).
        Takes the first result from the response and extracts its GeoJSON data (the geojson property).
        Verifies that the GeoJSON type is "Polygon" (ignore other types for the MVP).
        Processes the GeoJSON as follows:
            Calculate the Earth center of the city by averaging the longitude and latitude of all coordinates in the GeoJSON’s coordinates[0] array.
            Define the scale factor as 6371 / 1737 (Earth’s radius divided by the Moon’s radius, approximately 3.67).
            For each coordinate pair in the GeoJSON’s coordinates[0] array, compute new coordinates:
                Subtract the Earth center longitude and latitude from the original coordinate’s longitude and latitude.
                Multiply the differences by the scale factor to adjust the size to the Moon’s scale.
                Use the resulting scaled coordinates directly (centered at [0, 0] on the map).
            Create a new GeoJSON object with type: "Polygon" and the scaled coordinates as its coordinates[0] array.
        If a cityLayer already exists, remove it from the map.
        Create a new cityLayer using Leaflet’s GeoJSON functionality with the scaled GeoJSON, styled with:
            Fill opacity of 0.5 for transparency.
            Border color of blue.
            Border weight of 2 pixels.
        Add the new cityLayer to the map.
    Add error handling in the event listener:
        If the API returns no results, display an alert with the message "No results found for the city."
        If the GeoJSON type is not "Polygon", display an alert with the message "City boundary is not a polygon."
    Tricky Part (Code Provided): The scaling logic must be precise to ensure the city’s size is correctly proportional to the Moon. Use this exact calculation for each coordinate [lon, lat] in the GeoJSON:
    javascript

    scaledLon = scale * (lon - earthCenterLon);
    scaledLat = scale * (lat - earthCenterLat);

    where scale = 6371 / 1737, and [earthCenterLon, earthCenterLat] is the computed center of the original city coordinates.

Step 5: Implement the Clear Functionality

    Add an event listener to the "Clear" button (id="clear-btn") that:
        Checks if cityLayer is not null.
        If it exists, removes cityLayer from the map and sets cityLayer back to null.

Step 6: Test the Application Locally

    Save index.html with all the implemented code.
    Open the file in a web browser (e.g., double-click it or serve it locally with a command like python -m http.server 8000 and visit http://localhost:8000).
    Verify the following:
        The Moon map loads, and you can pan and zoom it.
        Entering a city name (e.g., "Paris") and clicking "Search" displays its scaled shape centered on the map.
        Panning the map moves the Moon under the fixed city overlay.
        Clicking "Clear" removes the city overlay.

Notes for Executing LLMs

    API Usage: Use the USGS PWMS WMTS API and Nominatim API as specified. Do not change the URLs unless they fail to load, and report any issues without altering the plan.
    Scaling: The provided scaling code snippet must be used exactly as shown to ensure accurate size representation. Beyond that, implement the coordinate processing logic as described.
    GeoJSON Limitation: Assume city boundaries are single polygons. If a MultiPolygon is returned, the app may fail—note this limitation but proceed as is for the MVP.
    Projection: The Moon map uses an equirectangular projection. Use Leaflet’s default CRS; if tiles don’t load, report the issue without changing the plan.
    City Positioning: The city overlay must be fixed at the map’s center ([0, 0]), with users panning the Moon map underneath.