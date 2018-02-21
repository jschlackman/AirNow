# AirNow Virtual Sensor

A SmartThings virtual device type handler (DTH) for retrieving air quality data from the U.S. EPA [AirNow API](https://docs.airnowapi.org/).

## Setup

1. If you don't already have one, get an API key for AirNow.
    1. [Request a free AirNow API key](https://docs.airnowapi.org/account/request/)
    2. [Log in](https://docs.airnowapi.org/login) to the AirNow API website.
    3. Visit any of the Web Services documentation pages (e.g. [Forecast by Zip Code](https://docs.airnowapi.org/forecastsbyzip/docs)) and look for **Your API Key:** in the top right of the page.
    
       It should be a string of hexadecimal characters in the format **XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX**. If you see your API key simply listed as GUEST, that's not it. The main AirNow Web Services index page currently has a bug that shows this as the key even when logged in. Try one of the subpages.
    
       Save your key somewhere - you will need it in step 4.

2. Install & publish the DTH into the **My Device Handlers** section of the SmartThings IDE via [GitHub integration](http://docs.smartthings.com/en/latest/tools-and-ide/github-integration.html#setup) (recommended), or the via the old fashioned way of pasting the code into the **Create New Device Handler** > **From Code** window.

3. Go to the **My Devices** section of the IDE and click **New Device** to create a new device using the new DTH:
    1. **Name**: enter a readable name such as 'Air Quality'
    2. **Device Network Id**: enter any short string of characters not already in use by another device.
    3. **Type**: pick **AirNow Virtual Sensor** from the dropdown (it will be near the bottom of the list).
    4. **Version**: Published
    5. **Location**: Pick your hub location from the dropdown.
    6. **Hub**: Pick your hub from the dropdown.
    7. Click **Create**.
    8. Go to the new device you created in the SmartThings IDE > **My Devices** > **Air Quality** (or whatever you named it) and click on **App Settings** in the top right.
    9. Click the **edit** next to **Preferences** near the bottom of the page and paste in your API key in the box marked **AirNow API Key - Register at airnowapi.org**

4. On the Device page for the new virtual device you just created, click the **edit** link next to **Preferences** and paste your API key from step 1 into the **AirNow API Key** box.

5. If you want to get weather data from a different zipcode than the configured zipcode fro your hub, enter it in the zipcode box.

6. Click **Save**. The virtual device is now ready to use.

## Use

Once setup is complete, you can view basic air quality data via the SmartThings mobile app by checking the device under **My Home** > **Things**. The Things list will show a 'combined AQI' value, which is generally the higher of the Ozone or PM2.5 observations. The device detail view will show a breakdown of the different observations, as well as the location from which air quality observations are being recorded.

The device handler will refresh data once an hour by default, but it does so via a somewhat unsupported method (SmartThings does not provide a supported method for devices to auto-refresh themselves). If you need data more frequently, or it is not updating at all for you, use a device polling SmartApp such as [Pollster](https://github.com/statusbits/smartthings/blob/master/Pollster.md). The device will attempt to pull new data every time it is polled.

Almost all the data available from the AirNow API is also accessible to SmartApps via device attributes. These can be used as inputs for other automations that support reading from devices using the generic **capability.sensor**, such as custom rules created in [WebCoRE](https://community.smartthings.com/t/faq-what-is-webcore-and-what-was-core/59981).

The attributes available from the device are:

| Attribute Name  | Format | Description  |
|---|---|---|
| combined | Number | Combined AQI value (worst of either Ozone or PM2.5) |
| combinedCategory | Number | Combined AQI category number |
| combinedCategoryName | String | Combined AQI category name |
| O3 | Number | Ozone AQI value |
| O3Category | Number | Ozone AQI category number |
| O3CategoryName | String | Ozone AQI category name |
|	Pm25 | Number | PM2.5 AQI value |
| Pm25Category | Number | PM2.5 AQI category number |
| Pm25CategoryName | String | PM2.5 AQI category name |
| reportingLocation | String | City or area name of observed data, with 2-letter state code. May also contain basic error messages when data is unavailable from the API. |
| dateObserved | String | Date of observation (yyyy-mm-dd) |
| hourObserved | Number | Hour of observation (00-23) |
| latitude | Number | Latitude of observation area in decimal degrees. |
| longitude | Number | Longitude of observation area in decimal degrees. |

You can view the current values for any of these on the **Recently** tab for the device in the SmartThings mobile app.

## Data Limitations

Data returned is subject to the [limitations of the AirNow API](https://docs.airnowapi.org/faq#reportingAreaForecasts). Some zipcodes will simply not have any data available, in which case you will never receive alerts based on air quality and you may see the message "Failed to retrieve valid air quality data from AirNow" in the live logging section of the IDE. You can test whether data is avaiable for your zipcode by testing it on the [AirNow API Query Tool](https://docs.airnowapi.org/ForecastsByZip/query). If the output for the **application/json** format is simply [], then no data is available. Your best bet is to try a neighboring zipcode. Look up your zipcode on http://maps.huge.info/zip.htm and try some of the other zipcodes near you.

In addition, this virtual device will only return air quality index (AQI) values below 2000. The AirNow API sometimes reports completely erroneous and very high values, so anything above 2000 will be ignored. Here's why the cut-off is at that level:

The [EPA's AQI categories](https://airnow.gov/index.cfm?action=aqi_brochure.index) work on a scale from 0-500. Anything above 300 categorized as **Hazardous**, and according to the EPA, such conditions are "extremely rare" in the US and "generally occur only during events such as forest fires".

Values above 500 are categorized as **Beyond Index**, but have occasionally been reported in extremely polluted areas of China, so they are technically possible. It therefore seems prudent to allow for extreme readings, but at the same time set an upper limit beyond which values are considered to be an error.
