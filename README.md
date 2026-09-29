# Plex TV Episode Playlist Collector

A PHP script that scans a Plex TV library and adds every episode whose title or summary contains a keyword to an existing Plex playlist. It uses the Plex API through the Guzzle HTTP client.

## Features

- Scans every show, season, and episode in a named library section.
- Case-insensitive keyword match against episode title and summary.
- Adds matching episodes to an existing playlist by episode `ratingKey`.
- Resolves the server identifier, library section ID, and playlist ID dynamically by name.
- Logs each show, season, match, and skip to the console.

## Prerequisites

1. **Plex Media Server** with API access.
2. **Plex token**: see [Finding an authentication token](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/).
3. **An existing playlist**: the script looks it up by name and does not create it.
4. **PHP** and **Composer** (macOS install steps below).
5. **Guzzle** (installed via Composer).

### Installing on macOS

Install PHP with [Homebrew](https://brew.sh):

```bash
brew install php
```

Install Composer using the [official installer](https://getcomposer.org/installer):

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```

Verify:

```bash
php -v
composer --version
```

### Installing Guzzle

From the project directory:

```bash
composer require guzzlehttp/guzzle
```

This creates the `vendor/` directory that `collect.php` loads via `vendor/autoload.php`.

## Usage

### 1. Create the playlist

In Plex, create an empty playlist (e.g. `Thanksgiving Playlist`) or add any item to a new one. The name must match `$playlistName`.

### 2. Edit the script

Open `collect.php` and set:

| Variable | Description | Example |
|---|---|---|
| `$plexHost` | Plex server URL | `http://192.168.1.10` |
| `$plexPort` | Plex server port | `32400` |
| `$token` | Plex API token | `xxxxxxxxxxxxxxxxxxxx` |
| `$libraryName` | TV library name (case-insensitive) | `TV Shows` |
| `$playlistName` | Target playlist name (case-insensitive) | `Thanksgiving Playlist` |
| `$keyword` | Search term for titles and summaries | `thanksgiving` |

### 3. Run

```bash
php collect.php
```

Sample output:

```
Playlist ID: 12345
Show Title: Example Show
Season Title: Season 3
Found episode: The Thanksgiving Episode - Season 3
Added to manual collection: Thanksgiving Playlist (Key: 6789)
...
Total episodes checked: 4210
```

## Plex API Endpoints Used

| Purpose | Method | Endpoint |
|---|---|---|
| Server identifier | GET | `/servers` |
| Library sections | GET | `/library/sections` |
| Find playlist | GET | `/playlists` |
| All shows in section | GET | `/library/sections/{sectionId}/all` |
| Seasons of a show | GET | `{showKey}` (e.g. `/library/metadata/1499`) |
| Episodes of a season | GET | `{seasonKey}` (e.g. `/library/metadata/1500/children`) |
| Add episode to playlist | PUT | `/playlists/{playlistId}/items` |

## Notes

- The script makes one request per show and one per season, so large libraries take time.
- The Plex token grants full account access. Do not commit it to version control.
- Only exact keyword substrings match; there is no fuzzy or multi-keyword search.

## Troubleshooting

- **`Could not find the library section`**: `$libraryName` does not match a library title.
- **`Could not find the collection`**: `$playlistName` does not match an existing playlist.
- **`Error:` with a response body**: a Plex API request returned a 4xx status. Check the token and the host/port.
- **`Class not found` / missing `vendor/autoload.php`**: run `composer require guzzlehttp/guzzle`.
- **Plex logs**: check the Plex Media Server logs for details on failed API calls.

## Contributing

Pull requests and issues are welcome. I may or may not look into it, but still would love to know :)
