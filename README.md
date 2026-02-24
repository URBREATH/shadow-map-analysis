# shadowmap

> Part of the [VC Map Project](https://github.com/virtualcitySYSTEMS/map-ui)
> describe your plugin

Mathematical Precision: Uses solar declination and hour angle calculations based on day of year and latitude
Polar Region Handling: Properly handles polar day/night scenarios (midnight sun, polar night)
Time Zone Accuracy: Calculates times in UTC and accounts for longitude-based solar noon

/ Key astronomical calculations:

- Solar declination = 23.45° _ sin(360° _ (284 + dayOfYear) / 365°)
- Hour angle = arccos(-tan(latitude) \* tan(declination))
- Solar noon = 12h - (longitude / 15°)
- Sunrise = Solar noon - Hour angle (in hours)
- Sunset = Solar noon + Hour angle (in hours)
