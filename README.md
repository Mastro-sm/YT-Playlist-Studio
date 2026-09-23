# Playlist Studio for YouTube

*Italiano: vedi [README.it.md](README.it.md).*

A Chrome extension that organizes your channel's videos into playlists. An AI model (Claude, ChatGPT or Gemini) groups the videos by topic, the extension orders each playlist by retention, and writes titles and descriptions optimized for YouTube and Google search. Nothing changes on your channel until you confirm the preview.

The interface is available in English and Italian and follows Chrome's language by default; you can switch it from the menu at the top right.

## What it does

1. Reads every uploaded video, optionally skipping non-public videos and very short ones such as Shorts.
2. Fetches average percentage viewed, average view duration and views for each video from YouTube Analytics.
3. Asks the AI model to find the channel's broad topics from all the titles, then to assign every video to a topic in batches of 120, using titles, tags and descriptions. This works with catalogs of any size.
4. For each topic: below the minimum (4 by default) it is skipped; above the maximum (10 by default) it keeps only the videos with the best retention. Videos are always ordered from the most engaging.
5. The AI model writes a search-optimized title and description for each new playlist.
6. In the preview you can edit the text, reorder, remove or add videos, and exclude playlists. Then you apply everything with one click.

When you run the analysis again later, playlists the extension created earlier are **updated** instead of duplicated (see [Keeping playlists up to date](#keeping-playlists-up-to-date)).

## Setup

### 1. Load the extension in Chrome

1. Unzip the archive into a folder of your choice.
2. Open `chrome://extensions` and turn on **Developer mode** (top right).
3. Click **Load unpacked** and choose the `playlist-studio` folder.
4. Click the extension icon to open Playlist Studio. Open **Credentials** and copy the **redirect URI**. It is always `https://haahjfjoiifabhkgpapklikpiondfafa.chromiumapp.org/`, because the extension ID is fixed in the manifest and does not depend on the folder.

### 2. Set up Google Cloud

1. Go to [console.cloud.google.com](https://console.cloud.google.com) and create a project.
2. In **APIs & Services → Library**, enable **YouTube Data API v3** and **YouTube Analytics API** (not the YouTube Reporting API).
3. Configure the OAuth consent screen (called **Google Auth Platform** in recent versions of the console): user type **External**, an app name and your email.
4. Choose how to handle access:
   - **Testing mode**: under **Audience**, add the Google account that manages the channel as a **test user**. Google asks you to sign in again about once a week.
   - **Production without verification**: under **Audience**, click **Publish app**. At sign-in Google shows an "unverified app" warning; click **Advanced**, then **Go to …** to continue. This is fine for personal use and avoids weekly sign-ins.
5. Under **Credentials** (or **Clients**), create an **OAuth client ID** of type **Web application**. Under **Authorized redirect URIs**, paste the redirect URI from step 1.
6. Copy the generated **client ID** into the extension's credentials.

### 3. Choose an AI provider

In **Credentials**, pick a provider and paste its API key. Keys are stored only in your browser and sent only to the chosen provider.

| Provider | Where to get a key | Default model |
|---|---|---|
| Anthropic (Claude) | [console.anthropic.com](https://console.anthropic.com) | `claude-sonnet-5` |
| OpenAI (ChatGPT) | [platform.openai.com/api-keys](https://platform.openai.com/api-keys) | `gpt-5.5` |
| Google (Gemini) | [aistudio.google.com/apikey](https://aistudio.google.com/apikey) | `gemini-flash-latest` |

You can change the model name in the **Model** field. A ChatGPT Plus or Gemini Advanced subscription does not include API access: the API is billed separately by each provider. A full analysis usually costs a few cents.

All three providers return structured data, so the extension never has to guess the format of the answer.

## How to use it

1. **Connect YouTube** and pick your channel. If the channel is a Brand Account, select it in Google's window.
2. Check the rules: minimum and maximum videos, retention criterion, metrics period, language of titles and descriptions, and a short description of the channel. The description noticeably improves topics and keywords.
3. Click **Analyze channel** and review the proposal.
4. Click **Create playlists on YouTube** (or **Apply on YouTube** when the proposal includes updates).

### Retention criteria

- **Balanced** (recommended): combines percentage viewed and watch time, so short videos are not favored just because they are easy to watch to the end.
- **Average percentage viewed**: useful when videos have similar lengths.
- **Average view duration**: rewards videos that generate the most minutes watched.

Videos without data in the selected period go to the end of the playlist.

## Keeping playlists up to date

You can run the analysis whenever you like, for example after publishing new videos or when retention has changed. The extension compares the new proposal with the playlists it created itself (the **managed playlists**). When a proposal shares enough videos with one of them, it updates that playlist instead of creating a new one.

In the preview, each playlist to update shows the videos to add (labeled **New**), the videos to remove and the number of moves. Removing a video from a playlist does not delete it from the channel. The extension computes the minimum number of operations, so updates use little quota.

The title and description of an existing playlist stay as they are, so it keeps the search ranking it has already earned. To rewrite them, tick **Also rewrite title and description** on the card.

Only managed playlists can be changed; playlists you created by hand are never touched. In the **Managed playlists** section you can see the list, add playlists created with an earlier version of the extension, or remove one you no longer want the extension to update.

## Installing a new version

Replace the files in the folder where you loaded the extension, then click the reload icon on the Playlist Studio card in `chrome://extensions`. Settings, credentials and managed playlists are kept.

Avoid removing the extension from Chrome: that deletes its settings and the list of managed playlists. If it happens, enter the credentials again and add your playlists back in **Managed playlists**, otherwise the next analysis would create duplicates.

## YouTube quota

The YouTube Data API has a default quota of 10,000 units per day. An analysis uses a few dozen. Creating a playlist costs about 50 units plus 50 per video added, so a 10-video playlist is about 550 units, or roughly 18 playlists a day. Updating costs 50 units per video added, removed or moved, plus 50 if you rewrite the title and description. The preview shows the estimate before you confirm.

If the quota runs out halfway through, the work done so far is saved: the next day, press **Resume** and the extension picks up where it stopped, without duplicates. The quota resets at midnight Pacific Time.

## Troubleshooting

- **Access blocked / has not completed the Google verification process**: your account is not a test user of the Google Cloud app, or you picked a different account in the sign-in window. Add it under **Audience → Test users**, or publish the app (step 2.4).
- **redirect_uri_mismatch**: the redirect URI in Google Cloud does not match the one shown in the extension. Copy it again from **Credentials**. Google can take a few minutes to apply the change.
- **An API is not enabled**: enable both APIs from step 2.2 in the same project that holds the client ID, then wait a couple of minutes.
- **No channel found**: you picked your personal account instead of the channel's Brand Account. Disconnect and connect again.
- **Invalid API key / model not found**: check the key and model name for the selected provider.
- **No credit left**: add credit or raise the spending limit in the provider's console.
- **Answer cut off / not in the expected format**: the extension retries automatically and splits the work into smaller pieces. If the message persists, try again or switch model.

## Project structure

```
manifest.json      extension configuration (Manifest V3, fixed ID)
_locales/          extension name and description in English and Italian
background.js      opens the dashboard when the icon is clicked
dashboard.html     interface
dashboard.css      styles
js/i18n.js         interface translations
js/auth.js         Google sign-in (OAuth via chrome.identity)
js/youtube.js      YouTube Data API and YouTube Analytics API
js/llm.js          topic grouping and SEO text, for Anthropic, OpenAI and Gemini
js/planner.js      composition, retention ordering and update calculation
js/dashboard.js    interface logic
js/storage.js      settings, saved proposal and managed playlists
```
