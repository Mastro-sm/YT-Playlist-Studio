# Security

This project is provided as is and is not actively maintained, so security reports may not be answered or fixed.

## How the extension handles your data

Playlist Studio runs entirely in your browser, with no server of its own and no telemetry.
API keys and the Google OAuth client ID are stored unencrypted in the extension's local storage on your device.
Google access tokens are kept only for the browser session and expire after about one hour.
Video titles, tags, the start of video descriptions and your channel description are sent to the AI provider you choose; Google credentials never are.

Set a spending limit on your AI provider account, and revoke the extension's Google access at any time from https://myaccount.google.com/permissions.
