# API Changes - Technical Summary for API Consumers

## Overview

This document outlines the recent changes to the search API, including a new endpoint for retrieving custom parameter categories and enhanced search filtering capabilities.

---

## New Endpoint Added

### `GET /custom-parameter-categories/v2`

Returns all available custom parameters that can be used in search filters.

**Authentication:** Bearer token required

**Response Structure:**
- Field keys for use in search requests
- Field types: `dropdown`, `currency`, `range_number`
- Available options for dropdown fields
- Range support flags for currency fields

**Use Cases:**
- Dynamically build search forms
- Validate search parameters before submission
- Get current currency for financial fields (based on user's profile)

**Example Response:**
```json
{
  "error": false,
  "data": [
    {
      "key": "general_info",
      "name": "General Info",
      "fields": [
        {
          "key": "age",
          "name": "Age",
          "type": "dropdown",
          "options": [
            { "label": "18-24", "value": "18-24" },
            { "label": "25-33", "value": "25-33" }
          ]
        }
      ]
    },
    {
      "key": "financial_profile",
      "name": "Financial Profile",
      "fields": [
        {
          "key": "monthly_income_precise",
          "name": "Monthly Income (Precise)",
          "type": "currency",
          "range": true
        }
      ]
    }
  ]
}
```

---

## Enhanced `/search` POST Endpoint

### New Features

#### 1. Currency Range Filtering

Financial fields now support min/max ranges with automatic currency conversion.

**Example:**
```json
{
  "custom_parameters": [
    {
      "category_key": "financial_profile",
      "field_key": "monthly_income_precise",
      "value_range": { 
        "min": 5000, 
        "max": 15000 
      },
      "filter_currency": "USD"
    }
  ]
}
```

#### 2. Time Availability Range Filtering

Search by available hours using numeric ranges.

**Example:**
```json
{
  "custom_parameters": [
    {
      "category_key": "relationship_and_availability",
      "field_key": "available_time_morning",
      "value_range": { 
        "min": 2, 
        "max": 5 
      }
    }
  ]
}
```

#### 3. Updated Custom Parameters Structure

**Field Types:**

| Field Type | Value Parameter | Example |
|------------|----------------|---------|
| `dropdown` | `value` (string) | `"value": "bachelor_science"` |
| `range_number` | `value_range` (object) | `"value_range": {"min": 2, "max": 5}` |
| `currency` | `value_range` (object) + `filter_currency` (string) | `"value_range": {"min": 5000, "max": 15000}, "filter_currency": "USD"` |

**Parameters:**
- `value`: For dropdown/single-value fields (string)
- `value_range`: For range-based fields (object with numeric min/max)
- `filter_currency`: Optional currency code for financial comparisons (string, 3 characters, e.g., "USD", "EUR")

### Complete Search Example

```json
{
  "pagination": {
    "page": 1,
    "per_page": 10
  },
  "sort": {
    "by": "monthly_budget",
    "order": "desc"
  },
  "monthly_budget": {
    "min": 1000,
    "max": 10000,
    "currency": "USD"
  },
  "places": [
    {
      "place": {
        "latitude": 25.2048,
        "longitude": 55.2708
      },
      "radius": 10
    }
  ],
  "languages": {
    "selected": [1, 2],
    "match_mode": "any"
  },
  "custom_parameters": [
    {
      "category_key": "general_info",
      "field_key": "education_level",
      "value": "bachelor_science"
    },
    {
      "category_key": "general_info",
      "field_key": "age",
      "value": "25-33"
    },
    {
      "category_key": "relationship_and_availability",
      "field_key": "available_time_morning",
      "value_range": {
        "min": 2,
        "max": 5
      }
    },
    {
      "category_key": "financial_profile",
      "field_key": "monthly_income_precise",
      "value_range": {
        "min": 5000,
        "max": 15000
      },
      "filter_currency": "USD"
    }
  ]
}
```

---

## Key Search Features

### Automatic Filtering
- **Role-based**: Sponsors see seekers, seekers see sponsors
- **Exclusions**: Deleted users, archived profiles automatically excluded
- **Mutual Interest**: Respects visibility settings

### Location Search
- Uses Haversine formula for distance calculations
- Supports multiple locations with different radii
- Defaults to user's location with 10km radius if not specified

### Currency Conversion
- Automatic conversion using real-time exchange rates
- Handles cross-currency budget comparisons
- Consistent financial criteria matching

### Custom Parameter Categories

1. **General Info**: age, weight, height, orientation, education_level, profession, religion, ethnicity, hobbies
2. **Relationship & Availability**: relationship_goals, partner_preferences, duration_of_relationship, available_time_morning, available_time_afternoon, available_time_evening
3. **Financial Profile**: capital_level_less_precise, capital_level_precise, monthly_income_less_precise, monthly_income_precise, one_time_payment_first_child, monthly_budget_per_child, financial_gift_upon_marriage, capital_growth_bonus, relationship_termination_compensation
4. **Living Preferences**: driving_license_availability, car_availability, driver_availability, living_format_test_moon, living_format_preliminary_contract, living_format_final_contract

---

## Breaking Changes

**None** - All changes are additions only. Existing search requests will continue to work without modification.

---

## Migration Notes

### For Existing Implementations

No changes required. Your existing search requests will continue to function as before.

### To Use New Features

1. **Call `/custom-parameter-categories/v2`** to get available search fields
2. **Update search requests** to include `value_range` and `filter_currency` for range-based filtering
3. **Use numeric values** for `value_range.min` and `value_range.max` (not strings)

### Example Migration

**Before (still works):**
```json
{
  "category_key": "general_info",
  "field_key": "education_level",
  "value": "bachelor_science"
}
```

**After (new capability):**
```json
{
  "category_key": "financial_profile",
  "field_key": "monthly_income_precise",
  "value_range": { "min": 5000, "max": 15000 },
  "filter_currency": "USD"
}
```

---

## Support

For questions or issues, please refer to the full OpenAPI specification at `/dist/openapi.json`.

