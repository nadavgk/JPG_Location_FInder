# jpg_location_finder

This Streamlit application allows users to pinpoint the location where a JPG image was taken, open it on Google Maps and find nearby establishments of their choice.

The app can extract coordinates from the metadata of a JPG file and use a Google API key to display the location on Google Maps along with nearby establishments such as: restaurants, gas stations, hospitals, parks etc. 

Note that this app uses the JPG's metadata to acquire the GPS coordinates. **Not all JPG files have this metadata** thus users are able to manually insert coordinates as-well

***

## **How to use this app:**

Execute *"streamlit run main.py"* in you terminal.  
The streamlite app will open in your browser and a step-by-step instruction is given in the interface.

    Step 1: API Configuration (optional)
In order to provide information regarding establishments in the vicinity of your specified location and to view the area in Google Street-View, a Google API-KEY is required.  
Upload a JSON file containing your Google API Key in the Following format:

**{"GOOGLE_API_KEY" : "your_google_api_key_here"}**

without an api-key the app will only open Google Maps on the specified location without extra information and without street view

    Step 2: Coordinates Source

If selected, upload a JPG file and the app will extract the coordinates from its metadata.
If not, proceed to manually enter the coordinates.

    Step 3: Choosing Establishment Type and radius size

Select the type of place you want to find nearby your specified location (e.g., restaurant, gas station).
Enter the search radius in meters.

    Step 5: Open location on Google maps with a list of establishments in the requasted radius

Click the "Open the map" Google Maps will open in browser and a list of nearby establishments will appear on the app

***

# **Features:**

This app contains 3 python modules:

- **main.py**
- **google_places_finder.py** 
- **jpg_coordinates_extractor.py**

***

# jpg_coordinates_extractor.py

## Function Explanations

### `process_image(file_path)`

This function reads an image file, extracts the GPS coordinates from its EXIF metadata, and converts these coordinates to decimal degrees.

- **Parameters**:
  - `file_path` (str): The file path of the image.

- **Returns**:
  - tuple: A tuple containing the latitude and longitude in decimal degrees.

- **Raises**:
  - Exception: If the image file does not contain GPS data or if an error occurs during processing.

- **Explanation**:
  - The function opens the image file in binary mode and reads its content.
  - It then checks if the image has EXIF data and GPS coordinates.
  - If GPS data is available, it retrieves the latitude and longitude along with their reference (North/South for latitude, East/West for longitude).
  - The coordinates are then converted from degrees, minutes, and seconds (DMS) format to decimal degrees using the `dms_to_decimal_degrees` function.
  - If any error occurs or if no GPS data is found, an exception is raised.

### `dms_to_decimal_degrees(latitude_d, latitude_m, latitude_s, longitude_d, longitude_m, longitude_s, lat_ref, long_ref)`

This function converts coordinates from degrees, minutes, and seconds (DMS) format to decimal degrees format.

- **Parameters**:
  - `latitude_d` (float): Degrees part of the latitude.
  - `latitude_m` (float): Minutes part of the latitude.
  - `latitude_s` (float): Seconds part of the latitude.
  - `longitude_d` (float): Degrees part of the longitude.
  - `longitude_m` (float): Minutes part of the longitude.
  - `longitude_s` (float): Seconds part of the longitude.
  - `lat_ref` (str): Latitude reference ('N' for North, 'S' for South).
  - `long_ref` (str): Longitude reference ('E' for East, 'W' for West).

- **Returns**:
  - tuple: A tuple containing the latitude and longitude in decimal degrees.

- **Explanation**:
  - The function calculates the decimal degrees for latitude and longitude by adding the degrees, minutes, and seconds after converting minutes and seconds to degrees.
  - It multiplies the latitude by -1 if the reference is 'S' (indicating the southern hemisphere).
  - It multiplies the longitude by -1 if the reference is 'W' (indicating the western hemisphere).
  - The resulting decimal degrees are returned as a tuple.



# google_places_finder.py

## Function Explanations

### `load_config(file)`

This function loads a JSON configuration file and returns its content as a dictionary.

- **Parameters**:
  - `file` (file-like object): The JSON file to be loaded.

- **Returns**:
  - dict: The content of the JSON file.
  - None: If the JSON file is invalid.

- **Explanation**:
  - The function tries to load the content of the JSON file using `json.load`.
  - If the file is not a valid JSON, it catches the `json.JSONDecodeError` and returns `None`.

### `check_street_view_availability(lat, lon, api_key)`

This function checks if Street View is available at the given latitude and longitude using the Google Maps API.

- **Parameters**:
  - `lat` (float): The latitude of the location.
  - `lon` (float): The longitude of the location.
  - `api_key` (str): The Google API key.

- **Returns**:
  - bool: `True` if Street View is available, `False` otherwise.

- **Explanation**:
  - The function constructs a URL to the Google Street View metadata API using the provided latitude, longitude, and API key.
  - It sends a request to the API and parses the JSON response.
  - The function returns `True` if the status in the response is "OK", indicating Street View is available, otherwise it returns `False`.

### `find_nearby_places(lat, lon, place_type, api_key)`

This function finds nearby places of a specified type using the Google Places API.

- **Parameters**:
  - `lat` (float): The latitude of the location.
  - `lon` (float): The longitude of the location.
  - `place_type` (str): The type of places to search for (e.g., "restaurant", "cafe").
  - `api_key` (str): The Google API key.

- **Returns**:
  - list: A list of nearby places if the request is successful.
  - list: An empty list if no places are found or if the request fails.

- **Explanation**:
  - The function constructs a URL to the Google Places Nearby Search API using the provided latitude, longitude, place type, and API key.
  - It sends a request to the API and parses the JSON response.
  - If the status in the response is "OK", it returns the list of places found, otherwise it returns an empty list.

### `open_google_maps(lat, lon, api_key)`

This function opens Google Maps in a web browser at the specified location, optionally with Street View.

- **Parameters**:
  - `lat` (float): The latitude of the location.
  - `lon` (float): The longitude of the location.
  - `api_key` (str): The Google API key.

- **Returns**:
  - None

- **Explanation**:
  - The function constructs a URL to Google Maps with the provided latitude and longitude.
  - If an API key is provided and Street View is available at the location (checked using `check_street_view_availability`), it constructs a Street View URL.
  - The function opens the URL in a new web browser tab using `webbrowser.open_new`.


# main.py

## Module Explanation

This module is a Streamlit application that allows users to upload a JPG file, extract its GPS coordinates, and open Google Maps at the specified location. Users can also enter coordinates manually and find nearby establishments using the Google Places API.

### Step-by-Step Process

#### Step 1: API Configuration
1. **Title and Description**:
   - The app title is set to "JPG to Google maps".
   - A brief description is provided to explain the purpose of the app.

2. **Upload JSON File**:
   - Users can upload a JSON file containing their Google API key.
   - If a file is uploaded, the `load_config` function reads the JSON file to extract the API key.
   - If the API key is found, it is stored in the `google_api_key` variable. If not, a warning is displayed.
   - A note informs users that without an API key, the app will open Google Maps without Street View or nearby establishments' functionality.

#### Step 2: Coordinates Source
3. **Choose Coordinates Source**:
   - Users can choose to use coordinates from a JPG file by checking a checkbox.

4. **Upload JPG File**:
   - If the checkbox is selected, users can upload a JPG file.
   - The uploaded JPG file is saved to a temporary directory.
   - The `process_image` function extracts GPS coordinates from the JPG file's EXIF metadata.
   - If coordinates are successfully extracted, they are displayed, and a success message is shown. If not, an error message is displayed.

5. **Enter Coordinates Manually**:
   - Users can manually enter latitude and longitude if they do not wish to use a JPG file or if the JPG file does not contain GPS data.

#### Step 3: Choose Establishment Type and Radius
6. **Select Establishment Type**:
   - Users can select the type of establishment they want to find near the specified location (e.g., restaurant, gas station, cafe).

7. **Enter Radius**:
   - Users can specify the radius (in meters) around the location to search for the selected type of establishment.

#### Step 4: Open Location on Google Maps
8. **Open Google Maps**:
   - When the user clicks the "Open the map" button, the app checks if coordinates have been provided either by extracting from a JPG file or by manual entry.
   - If coordinates are provided:
     - If an API key is available, the `find_nearby_places` function is used to find nearby establishments of the selected type within the specified radius.
     - The nearby establishments are listed in the app.
     - Google Maps is opened in a new browser tab, and if Street View is available, it is shown.
   - If coordinates are not provided, an error message is displayed asking the user to provide coordinates.

### Summary of Key Functions

#### `load_config(file)`
- Loads a JSON configuration file and returns its content as a dictionary.

#### `check_street_view_availability(lat, lon, api_key)`
- Checks if Street View is available at the given latitude and longitude using the Google Maps API.

#### `find_nearby_places(lat, lon, place_type, api_key)`
- Finds nearby places of a specified type using the Google Places API.

#### `open_google_maps(lat, lon, api_key)`
- Opens Google Maps in a web browser at the specified location, optionally with Street View.


