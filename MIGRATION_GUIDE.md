# Migration Guide: API 2.5 to API 3.0

This guide helps you migrate from OpenWeather API 2.5 to API 3.0.

## Overview

The Weather Formatter has been updated to use OpenWeather One Call API 3.0, which provides more accurate and detailed weather data. The application interface remains the same, but you need to update your API key subscription.

## What Changed

### API Changes

1. **Endpoint**: Changed from `/data/2.5/` to `/data/3.0/onecall`
2. **Geocoding**: ZIP codes are now automatically converted to coordinates using the Geocoding API
3. **Data Quality**: Hourly forecasts (instead of 3-hour intervals)
4. **Additional Data**: UV index, dew point, and other metrics now available

### New Features in API 3.0

- **Hourly Forecasts**: True hourly data for up to 48 hours (not 3-hour intervals)
- **UV Index**: Automatically included in weather data
- **Dew Point**: Available in all weather responses
- **Better Accuracy**: Improved weather prediction models
- **Weather Alerts**: Support for weather alerts (excluded by default in this app)

### What Stayed the Same

- All CLI arguments and commands
- Configuration file format
- Output format and separators
- Icon mappings
- All features and functionality

## Migration Steps

### Step 1: Subscribe to One Call API 3.0

1. Log in to your OpenWeatherMap account at https://home.openweathermap.org
2. Navigate to https://home.openweathermap.org/subscriptions
3. Find "One Call by Call" subscription
4. Click "Subscribe" and select the free tier (1,000 calls/day)
5. Complete the subscription process

### Step 2: Verify Your API Key

Your existing API key will work with API 3.0 after subscribing. No need to generate a new one.

1. Go to https://home.openweathermap.org/api_keys
2. Copy your API key
3. Update your `weather_config.yaml` if needed

### Step 3: Test the Application

Run the application to verify everything works:

```bash
weather-formatter -v
```

The verbose flag (`-v`) will show you the API calls being made.

## Configuration Changes

### No Configuration File Changes Required

Your existing `weather_config.yaml` file will continue to work without modifications. However, you may want to update the comments to reflect API 3.0:

```yaml
# Old comment (API 2.5)
# Get your free API key from: https://openweathermap.org/api

# New comment (API 3.0)
# NOTE: This application uses OpenWeather One Call API 3.0
# Sign up at: https://home.openweathermap.org/subscriptions
api_key: "YOUR_API_KEY_HERE"
```

### Optional: Use New Fields

API 3.0 provides additional fields you can now use in `output_fields`:

```yaml
output_fields:
  - hour
  - icon
  - temp
  - uv_index      # NEW: UV index data
  - dew_point     # NEW: Dew point temperature
  - precip
```

### Optional: Use GPS Coordinates Instead of ZIP Code

The application now supports GPS coordinates as an alternative to ZIP codes:

```yaml
# Instead of:
# zipcode: "10001"

# You can use:
latitude: 40.7128
longitude: -74.0060
```

Benefits:
- Works for any location worldwide (not just US)
- More precise location specification
- Saves one geocoding API call per session
- No ZIP code lookup required

## API Call Differences

### Before (API 2.5)

API 2.5 made these calls:
- Current Weather: `GET /data/2.5/weather?zip=10001,US`
- Forecast: `GET /data/2.5/forecast?zip=10001,US`

### After (API 3.0)

API 3.0 makes these calls:
1. Geocoding (cached): `GET /geo/1.0/zip?zip=10001,US`
2. One Call: `GET /data/3.0/onecall?lat=40.75&lon=-73.99`

**Note**: Geocoding results are cached, so the ZIP code lookup only happens once per session.

## Troubleshooting

### "Invalid API key" Error

**Problem**: Your API key is valid but API 3.0 calls fail.

**Solution**:
- Verify you've subscribed to One Call API 3.0 at https://home.openweathermap.org/subscriptions
- API subscriptions can take a few minutes to activate
- Ensure you selected "One Call by Call" subscription

### "Could not geocode ZIP code" Error

**Problem**: The Geocoding API cannot find your ZIP code.

**Solution**:
- Verify the ZIP code is a valid 5-digit US ZIP code
- Check for typos in your configuration
- The Geocoding API is included with your subscription

### API Rate Limit Issues

**Problem**: You're hitting rate limits more frequently.

**Solution**:
- API 3.0 uses 2 calls (geocode + weather) on first run, then 1 call (cached geocode)
- The geocoding result is cached per session
- Free tier still allows 1,000 calls/day
- Consider reducing forecast frequency if needed

### Different Data Values

**Problem**: Weather data looks different than before.

**Solution**:
- API 3.0 uses improved weather models and may show different predictions
- Hourly forecasts are now true hourly data (not 3-hour intervals)
- This is expected and represents more accurate data

## Benefits of Migration

1. **Better Data Quality**: Hourly forecasts instead of 3-hour intervals
2. **More Information**: UV index, dew point, and enhanced metrics
3. **Future-Proof**: OpenWeather is focusing development on API 3.0
4. **Better Accuracy**: Improved weather prediction algorithms
5. **Same Cost**: Free tier remains at 1,000 calls/day

## Rollback (Not Recommended)

If you need to rollback to API 2.5 (not recommended as it's deprecated):

1. Check out the previous version of the code before the API 3.0 migration
2. Use your old API key without the One Call subscription
3. Note that API 2.5 may be sunset by OpenWeather in the future

## Support

If you encounter issues during migration:

1. Check the [Troubleshooting](#troubleshooting) section above
2. Review the [README.md](README.md) for API setup instructions
3. Verify your subscription at https://home.openweathermap.org/subscriptions
4. Check OpenWeather's API status at https://openweathermap.org/api

## Summary

The migration to API 3.0 requires only one action: **subscribing to One Call API 3.0**. Everything else remains the same, and you'll benefit from better data quality and additional features.
