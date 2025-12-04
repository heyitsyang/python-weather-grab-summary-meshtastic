# GPS Coordinates Feature

## Overview

The Weather Formatter now supports GPS coordinates (latitude/longitude) as an alternative to US ZIP codes for location specification. This enhancement provides more flexibility and worldwide location support.

## Feature Summary

### What's New

- **GPS Coordinates Support**: Specify location using decimal latitude and longitude
- **Worldwide Coverage**: Not limited to US ZIP codes anymore
- **Dual Input Methods**: Choose either ZIP code OR coordinates
- **Optimized API Calls**: Using coordinates saves one geocoding API call per session
- **Validation**: Comprehensive validation for coordinate ranges and format

## Usage

### Configuration File

You can now specify location in two ways in your `weather_config.yaml`:

#### Option 1: ZIP Code (Original Method)
```yaml
zipcode: "10001"
```

#### Option 2: GPS Coordinates (New Method)
```yaml
latitude: 40.7128
longitude: -74.0060
```

**Important**: Use either ZIP code OR coordinates, not both.

### Command Line

You can also specify coordinates via command-line arguments:

```bash
# Using ZIP code
weather-formatter -z 10001 -k YOUR_API_KEY

# Using GPS coordinates
weather-formatter --latitude 40.7128 --longitude -74.0060 -k YOUR_API_KEY
```

## Benefits

### 1. Worldwide Support
- **Before**: Limited to US ZIP codes only
- **After**: Any location on Earth with valid coordinates

### 2. API Efficiency
- **ZIP Code**: Requires geocoding API call (1 extra call per session, then cached)
- **Coordinates**: Direct to weather API (no geocoding needed)

### 3. Precision
- **ZIP Code**: Geocoded to a general area within the ZIP code region
- **Coordinates**: Exact location specified to many decimal places

### 4. Flexibility
- Useful for weather stations, remote locations, or areas without postal codes
- Works for marine locations, wilderness areas, etc.

## Technical Details

### Coordinate Format

- **Latitude**: Decimal degrees, range -90 to 90
  - Positive values = North of equator
  - Negative values = South of equator
  - Example: 40.7128 (New York City)

- **Longitude**: Decimal degrees, range -180 to 180
  - Positive values = East of Prime Meridian
  - Negative values = West of Prime Meridian
  - Example: -74.0060 (New York City)

### Validation Rules

1. **Mutual Exclusivity**: Cannot specify both ZIP code and coordinates
2. **Completeness**: If using coordinates, both latitude and longitude must be provided
3. **Range Validation**:
   - Latitude must be between -90 and 90
   - Longitude must be between -180 and 180
4. **Type Validation**: Coordinates must be numeric (int or float)

### API Call Flow

#### Using ZIP Code:
```
User Input (ZIP) → Geocoding API (cached) → Coordinates → One Call API → Weather Data
```

#### Using Coordinates:
```
User Input (Lat/Lon) → One Call API → Weather Data
```

## Files Modified

### 1. `weather_formatter/config.py`
- Added `latitude` and `longitude` fields to `WeatherConfig` dataclass
- Updated `validate()` method to handle coordinate validation
- Updated `load_config()` to parse latitude/longitude from YAML
- Updated `merge_config()` to merge coordinate values from CLI
- Updated `create_default_config()` template with coordinate examples

### 2. `weather_formatter/weather_client.py`
- Modified `get_current_weather()` to accept optional latitude/longitude parameters
- Modified `get_hourly_forecast()` to accept optional latitude/longitude parameters
- Both methods now determine whether to geocode or use coordinates directly

### 3. `weather_formatter/cli.py`
- Added `--latitude` and `--longitude` command-line arguments
- Updated `validate_cli_arguments()` to validate coordinates
- Updated `main()` to pass coordinates to weather client methods
- Added coordinate example to CLI help text

### 4. `examples/example_config.yaml`
- Added comprehensive documentation for coordinate usage
- Included example coordinates for New York City
- Listed benefits of using coordinates

### 5. `README.md`
- Added coordinate options to "Location and API Options" section
- Updated examples to show coordinate usage
- Added "Location Options" section explaining both methods
- Updated configuration structure examples

### 6. `MIGRATION_GUIDE.md`
- Added optional section about using GPS coordinates
- Listed benefits of coordinate usage

## Examples

### Example 1: Major Cities

```yaml
# Tokyo, Japan
latitude: 35.6762
longitude: 139.6503

# London, UK
latitude: 51.5074
longitude: -0.1278

# Sydney, Australia
latitude: -33.8688
longitude: 151.2093

# São Paulo, Brazil
latitude: -23.5505
longitude: -46.6333
```

### Example 2: Remote Locations

```yaml
# Mount Everest Base Camp
latitude: 28.0026
longitude: 86.8528

# Middle of Pacific Ocean
latitude: 0.0
longitude: -140.0

# South Pole
latitude: -90.0
longitude: 0.0
```

### Example 3: CLI Usage

```bash
# Get weather for Paris, France
weather-formatter --latitude 48.8566 --longitude 2.3522 -k YOUR_API_KEY

# Get tomorrow's forecast for Tokyo
weather-formatter --latitude 35.6762 --longitude 139.6503 --day tomorrow -k YOUR_API_KEY

# Use config file with coordinates
cat > my_weather_config.yaml << EOF
api_key: "YOUR_API_KEY"
latitude: 51.5074
longitude: -0.1278
forecast_hours: 8
EOF

weather-formatter --config my_weather_config.yaml
```

## Error Handling

### Common Errors and Solutions

#### Error: "Both latitude and longitude must be provided together"
**Cause**: Only one coordinate was specified
**Solution**: Provide both latitude and longitude

```yaml
# Wrong
latitude: 40.7128

# Correct
latitude: 40.7128
longitude: -74.0060
```

#### Error: "Provide either zipcode OR latitude/longitude coordinates, not both"
**Cause**: Both ZIP code and coordinates were specified
**Solution**: Choose one method and comment out or remove the other

```yaml
# Wrong
zipcode: "10001"
latitude: 40.7128
longitude: -74.0060

# Correct (Option 1)
zipcode: "10001"
# latitude: 40.7128
# longitude: -74.0060

# Correct (Option 2)
# zipcode: "10001"
latitude: 40.7128
longitude: -74.0060
```

#### Error: "latitude must be a number between -90 and 90"
**Cause**: Invalid latitude value
**Solution**: Check that latitude is within valid range

```yaml
# Wrong
latitude: 95.0

# Correct
latitude: 40.7128
```

#### Error: "longitude must be a number between -180 and 180"
**Cause**: Invalid longitude value
**Solution**: Check that longitude is within valid range

```yaml
# Wrong
longitude: 200.0

# Correct
longitude: -74.0060
```

## Finding Coordinates

You can find coordinates for any location using:

1. **Google Maps**: Right-click any location and select coordinates
2. **OpenStreetMap**: Click any location to see coordinates
3. **GPS Device**: Most GPS devices display current coordinates
4. **Search Engines**: Search "coordinates of [location]"
5. **Weather Station**: If you have a weather station, use its coordinates

### Coordinate Format Conversion

If you have coordinates in degrees/minutes/seconds format, convert to decimal:

```
Degrees + (Minutes / 60) + (Seconds / 3600)

Example: 40° 42' 46" N
= 40 + (42 / 60) + (46 / 3600)
= 40.7128
```

Remember:
- North latitudes and East longitudes are positive
- South latitudes and West longitudes are negative

## Testing

To test the coordinate feature:

1. **Verify Coordinates Work**:
```bash
weather-formatter --latitude 40.7128 --longitude -74.0060 -k YOUR_API_KEY -v
```

2. **Verify ZIP Code Still Works**:
```bash
weather-formatter -z 10001 -k YOUR_API_KEY -v
```

3. **Verify Mutual Exclusion**:
```bash
# This should fail with error
weather-formatter -z 10001 --latitude 40.7128 --longitude -74.0060 -k YOUR_API_KEY
```

4. **Verify Validation**:
```bash
# This should fail (latitude out of range)
weather-formatter --latitude 100 --longitude -74.0060 -k YOUR_API_KEY
```

## Performance

### API Calls Comparison

| Method | First Call | Subsequent Calls* |
|--------|-----------|------------------|
| ZIP Code | 2 API calls<br>(geocode + weather) | 1 API call<br>(weather only, geocoding cached) |
| Coordinates | 1 API call<br>(weather only) | 1 API call<br>(weather only) |

*Subsequent calls within the same session

### Performance Impact

- **Minimal**: Coordinate validation adds negligible overhead
- **Improvement**: Using coordinates eliminates one API call on first request
- **Caching**: ZIP code geocoding is cached, so performance is equal after first call

## Future Enhancements

Potential future improvements:

1. **City Name Support**: Add support for city names (would require geocoding)
2. **Coordinate History**: Save frequently used coordinates
3. **Coordinate Aliases**: Allow named locations in config
4. **Reverse Geocoding**: Display location name from coordinates in verbose mode

## Summary

The GPS coordinates feature provides a flexible, worldwide location specification method that:
- Works anywhere on Earth
- Saves API calls
- Provides precise location control
- Maintains backward compatibility with ZIP codes
- Includes comprehensive validation

This feature is especially useful for:
- International users
- Weather stations
- Remote monitoring
- Marine/aviation applications
- Areas without postal codes
