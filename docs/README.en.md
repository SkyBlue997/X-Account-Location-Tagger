# X Account Location Tagger

[简体中文](../README.md) | [English](README.en.md) | [日本語](README.ja.md)

## Table of Contents

- [Overview](#overview)
- [Installation & Usage](#installation--usage)
- [Technical Implementation](#technical-implementation)
- [Privacy & Security](#privacy--security)

## Overview

X displays account location and App Store region information in the "About this account" section, but this data isn't visible on the timeline. This script adds an interactive button next to each tweet's timestamp, allowing you to query location details on demand.

**Key Features:**
- **On-Demand Querying**: Click to query - no automatic batch requests, avoiding rate limits
- **VPN Detection**: Color-coded results (gray for verified locations, red for potential VPN usage)
- **Smart Caching**: Previously queried accounts are automatically labeled without additional API calls

The script uses X's internal GraphQL API to fetch location data and displays it with appropriate visual indicators based on location accuracy.

## Installation & Usage

1. Install [Tampermonkey](https://www.tampermonkey.net/) or [Violentmonkey](https://violentmonkey.github.io/)
2. (Recommended) Install directly from [Greasy Fork](https://greasyfork.org/zh-CN/scripts/556852-x-account-location-tagger), or create a new userscript manually
3. If installing manually, copy the contents of `X.js`
4. Save and enable the script
5. Visit X and start clicking location buttons

Navigate to any X timeline. You'll see a map pin icon next to tweet timestamps:

- Click the pin to query location
- Button shows loading state while fetching
- Results appear as: `[Country / App Store Region]`
- Gray text indicates verified location
- Red text indicates possible VPN usage

## Technical Implementation

The script uses X's internal GraphQL API endpoint `AboutAccountQuery`, the same endpoint that powers the "About this account" feature. When you click a location button, it:

1. Extracts the username from the tweet's timestamp link
2. Sends a GraphQL query with your session credentials
3. Parses the response for `account_based_in` and `source` fields
4. Checks `location_accurate` to determine VPN likelihood
5. Displays the result with appropriate color coding

All requests include proper CSRF tokens and authentication headers from your active X session.

The script parses responses in this format:

```json
{
  "data": {
    "user_result_by_screen_name": {
      "result": {
        "about_profile": {
          "account_based_in": "Japan",
          "location_accurate": true,
          "source": "Japan App Store"
        }
      }
    }
  }
}
```

Key fields:
- `account_based_in`: Geographic location
- `source`: App Store region or client app
- `location_accurate`: VPN detection flag

The script requires minimal configuration. All settings are hardcoded:

```javascript
const ABOUT_ENDPOINT = 'https://x.com/i/api/graphql/zs_jFPFT78rBpXv9Z3U2YQ/AboutAccountQuery';
const AUTH_BEARER = 'Bearer [official web client token]';
const DEBUG = true;
```

Debug mode logs all requests and responses to the console. Set to `false` to disable.

The on-demand design naturally avoids rate limiting issues. You only send requests when you click, and the cache prevents duplicate queries. If you encounter a 429 error, wait a few minutes before continuing.

## Privacy & Security

This script:
- Uses only public X API endpoints
- Displays information already available in the UI
- Sends no data to third-party servers
- Processes everything locally in your browser
- Requires your active X session (must be logged in)

The data you see is what X already knows and shows in the account details page. This script simply makes it more accessible.

## License

MIT License - use freely, modify as needed, no warranty provided.

## Acknowledgments

Built by reverse-engineering X's internal GraphQL API. Thanks to the browser developer tools that made this possible.

---

**Note**: This script relies on X's internal API which may change without notice. Updates may be required if X modifies their GraphQL schema or endpoint structure.

