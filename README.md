# Drupal Signage

Drupal Signage is a Drupal-based digital signage system for managing screens, slides, playlists, scheduling, access control, and browser-based playback.

The project uses Drupal as the content management and administration platform. Physical displays can open a player URL in a browser, making the playback client simple to run on hardware such as a Raspberry Pi, mini PC, or a screen with a built-in browser.

**Technologies:** PHP, Drupal 11, MariaDB, Docker, DDEV

## Features

Current and planned functionality includes:

- Screen management
- Screen groups
- Slide management
- Playlist-based playback
- Scheduling for playlist items
- Browser-based player view
- Dashboard for signage users
- Screen-specific access management
- Slide sharing between users
- Drupal configuration export/import for reproducible setup

## Project status

The current implementation focuses on the core digital signage workflow:

```text
Create slides
    ↓
Add slides to playlists
    ↓
Assign playlists to screens or screen groups
    ↓
Open the player URL on a display
```

The project is suitable for local development, testing, and continued feature development. Before production use, deployment, permissions, security hardening, backups, monitoring, and display-device setup should be reviewed for the target environment.

## Contents

- [Requirements](#requirements)
- [Local development setup](#local-development-setup)
- [Installing from exported configuration](#installing-from-exported-configuration)
- [Installing from an existing database](#installing-from-an-existing-database)
- [Common development commands](#common-development-commands)
- [Project structure](#project-structure)
- [Custom modules](#custom-modules)
- [Main routes](#main-routes)
- [Content model](#content-model)
- [Player behavior](#player-behavior)
- [Access and permissions](#access-and-permissions)
- [Configuration management](#configuration-management)
- [Testing](#testing)
- [Production notes](#production-notes)
- [Troubleshooting](#troubleshooting)

## Requirements

### Local development

Install the following tools before setting up the project:

- Git
- Docker or a supported Docker provider
- DDEV
- A terminal or shell environment

Composer and Drush are normally run through DDEV during local development.

### Runtime configuration

The repository is configured for:

- Drupal 11
- PHP 8.3
- MariaDB 10.11
- Nginx with PHP-FPM
- Composer 2
- Drush 13

The DDEV project configuration uses:

```yaml
type: drupal11
docroot: web
php_version: "8.3"
webserver_type: nginx-fpm
database:
  type: mariadb
  version: "10.11"
```

## Local development setup

Clone the repository:

```bash
git clone https://github.com/DevFloki/drupal-signage.git
cd drupal-signage
```

Start the DDEV environment:

```bash
ddev start
```

Install PHP dependencies:

```bash
ddev composer install
```

Check the project URLs and service information:

```bash
ddev describe
```

Open the site in a browser:

```bash
ddev launch
```

At this point the codebase is ready, but Drupal still needs either a clean installation from exported configuration or an imported database.

## Installing from exported configuration

Use this method when setting up a clean local site from the configuration stored in the repository.

```bash
ddev start
ddev composer install
ddev drush site:install --existing-config --account-name=admin --account-pass=admin -y
ddev drush cache:rebuild
```

Then open the site:

```bash
ddev launch
```

Login with:

```text
Username: admin
Password: admin
```

Change the administrator password after setup if the environment will be shared or exposed outside your own machine.

## Installing from an existing database

Use this method when setting up the project from a database dump.

Start DDEV and install dependencies:

```bash
ddev start
ddev composer install
```

Import the database:

```bash
ddev import-db --file=/path/to/database.sql.gz
```

If user-uploaded files are available, import them as well:

```bash
ddev import-files --source=/path/to/files
```

Run database updates, import configuration, and clear caches:

```bash
ddev drush updatedb -y
ddev drush config:import -y
ddev drush cache:rebuild
```

Open the site:

```bash
ddev launch
```

## Common development commands

Clear Drupal cache:

```bash
ddev drush cr
```

Import exported configuration:

```bash
ddev drush cim -y
```

Export configuration changes:

```bash
ddev drush cex -y
```

Run database updates:

```bash
ddev drush updb -y
```

Check Drupal status:

```bash
ddev drush status
```

Open a shell in the web container:

```bash
ddev ssh
```

Stop the project:

```bash
ddev stop
```

Restart the project:

```bash
ddev restart
```

Run Composer commands:

```bash
ddev composer <command>
```

Examples:

```bash
ddev composer install
ddev composer update drupal/core-recommended --with-dependencies
```

## Project structure

Important paths:

```text
.
├── .ddev/                         Local DDEV configuration
├── composer.json                  PHP dependencies and Drupal project setup
├── composer.lock                  Locked dependency versions
├── config/sync/                   Exported Drupal configuration
└── web/
    ├── core/                      Drupal core
    ├── modules/
    │   ├── contrib/               Contributed Drupal modules
    │   └── custom/
    │       └── signage/           Custom digital signage modules
    ├── sites/default/             Drupal site settings and files directory
    └── themes/                    Drupal themes
```

The repository should not include:

- Local database content
- User-uploaded files in `web/sites/default/files/`
- Local-only settings
- DDEV-generated runtime files
- API keys, tokens, passwords, or environment-specific secrets

## Custom modules

The custom code is placed under:

```text
web/modules/custom/signage/
```

The parent module is:

| Module | Purpose |
|---|---|
| `signage` | Parent module for the custom digital signage modules. |

Submodules are placed under:

```text
web/modules/custom/signage/modules/
```

| Module | Purpose |
|---|---|
| `signage_screen` | Screen management helpers and screen-related administration functionality. |
| `signage_player` | Full-screen player pages for displaying screen content. |
| `signage_dashboard` | Dashboard page for digital signage users. |
| `signage_access` | Screen access management and screen-specific user access control. |
| `signage_share` | Slide sharing functionality between users. |

Enable the custom modules with:

```bash
ddev drush en signage signage_screen signage_player signage_dashboard signage_access signage_share -y
ddev drush cr
```

If the site was installed from exported configuration, these modules should already be enabled according to `config/sync/core.extension.yml`.

## Main routes

The most important custom routes are:

| Route | Path | Description |
|---|---|---|
| `signage_dashboard.page` | `/dashboard` | Dashboard for logged-in users. |
| `signage_player.screen` | `/player/{screen}` | Browser-based player for a specific screen node ID. |
| `signage_access.admin` | `/admin/signage/access` | Screen Access Manager. |
| `signage_access.admin_screen` | `/admin/signage/access/{node}` | Screen Access Manager for a specific screen node. |

Example player URL:

```text
/player/12
```

In this example, `12` must be the node ID of a screen.

## Content model

The system is based on structured Drupal content.

| Concept | Description |
|---|---|
| Screen | A logical representation of a physical display. |
| Screen group | A group of screens that can share playlists. |
| Playlist | A collection of playlist items. |
| Playlist item | A scheduled and ordered item inside a playlist. |
| Slide | Content displayed by the player. |
| Media | Image files or other media assets used by slides. |
| User | A Drupal user who can manage content or access assigned screens. |

The player can resolve playlists in two ways:

1. Playlists connected directly to a screen.
2. Playlists connected through screen groups that contain the screen.

Duplicate playlist items are skipped during playback resolution.

## Player behavior

The player is available at:

```text
/player/{screen}
```

The `{screen}` parameter is the screen node ID.

The player controller:

- Loads the requested screen.
- Resolves playlists connected to the screen.
- Resolves playlists connected through screen groups.
- Filters playlist items by enabled status.
- Filters playlist items by date, day, and time schedule.
- Builds playable slide data.
- Returns a full-screen player render array.
- Disables page cache for the player response.

The player JavaScript:

- Reads playback data from the page.
- Displays playable items one at a time.
- Uses the slide duration value when available.
- Falls back to a default duration of 10 seconds.
- Shows a status message if no playable slides are available.
- Reloads the page every 60 seconds to fetch updated content.

Supported playback item type:

| Type | Description |
|---|---|
| Image slide | Displays an image slide with optional title and body text. |

Fallback states include:

| Fallback reason | Meaning |
|---|---|
| `screen_not_found` | The requested screen does not exist or is not a screen node. |
| `playlists_missing` | No playlists are connected to the screen. |
| `playlists_empty` | Connected playlists contain no items. |
| `all_items_disabled` | Playlist items exist, but all are disabled. |
| `all_items_outside_schedule` | Playlist items exist, but none are active at the current time. |
| `no_typed_items` | Playlist items do not have selected slide content. |
| `no_image_slides` | No image slides are available for playback. |
| `no_valid_slides` | No valid playable slides were found. |

## Access and permissions

The project uses Drupal users, roles, and permissions.

The custom access module defines this permission:

```text
administer signage access
```

This permission gives access to the Screen Access Manager. It should only be assigned to trusted administrator or manager roles.

The Screen Access Manager is available at:

```text
/admin/signage/access
```

or for a specific screen:

```text
/admin/signage/access/{node}
```

## Configuration management

Drupal configuration is stored in:

```text
config/sync/
```

Use configuration export after changing site structure, fields, content types, views, permissions, or module settings:

```bash
ddev drush cex -y
```

Commit exported configuration changes together with the code that depends on them.

Use configuration import after pulling changes from Git:

```bash
git pull
ddev composer install
ddev drush updb -y
ddev drush cim -y
ddev drush cr
```

Recommended workflow:

```text
1. Pull latest code.
2. Install or update Composer dependencies.
3. Run database updates.
4. Import configuration.
5. Clear cache.
6. Test the affected functionality.
```

## Testing

### Manual verification

After installation, verify the main workflow:

1. Log in as an administrator.
2. Confirm that the custom modules are enabled.
3. Create or verify a screen.
4. Create or verify a playlist.
5. Add playlist items.
6. Add image slides to playlist items.
7. Connect the playlist directly to a screen or through a screen group.
8. Open the player URL for the screen.
9. Confirm that the player displays the expected content.
10. Confirm that disabled or out-of-schedule playlist items are not displayed.

Useful pages:

```text
/dashboard
/admin/signage/access
/player/{screen}
```

### Cache and configuration checks

Run:

```bash
ddev drush cr
ddev drush status
ddev drush config:status
```

### Unit tests

The repository contains unit tests under the custom modules, for example:

```text
web/modules/custom/signage/modules/*/tests/src/Unit/
```

Run PHPUnit through DDEV from the project root:

```bash
ddev exec ./vendor/bin/phpunit
```

If PHPUnit configuration has not been created for the local environment, create or adapt it from Drupal's example configuration before running the full test suite.

## Production notes

DDEV is intended for local development and should not be used as the production runtime.

A production deployment should use a normal PHP web server environment with:

- PHP version compatible with the installed Drupal version
- A supported database server
- Composer dependencies installed without development packages
- Web server document root pointing to `web/`
- HTTPS enabled
- Correct Drupal file permissions
- Trusted host patterns configured
- Database and uploaded-file backups
- A deployment process for code, database updates, and configuration imports

Typical production deployment flow:

```bash
git pull
composer install --no-dev --optimize-autoloader
drush updatedb -y
drush config:import -y
drush cache:rebuild
```

The production `settings.php` file should not be copied from a local machine without review. Database credentials, trusted host patterns, hash salt, file paths, and environment-specific settings must be configured for the target environment.

## Troubleshooting

### DDEV does not start

Try:

```bash
ddev poweroff
ddev start
```

If the problem continues, check that Docker is running.

### Composer dependencies are missing

Run:

```bash
ddev composer install
ddev drush cr
```

### Drupal shows a white page or old output

Clear cache:

```bash
ddev drush cr
```

### Configuration import fails

Check configuration status:

```bash
ddev drush config:status
```

Then run:

```bash
ddev drush updb -y
ddev drush cim -y
ddev drush cr
```

If the site was installed without `--existing-config`, configuration import may fail because the site UUID does not match the exported configuration. For a clean setup, reinstall using:

```bash
ddev drush site:install --existing-config --account-name=admin --account-pass=admin -y
```

### Player URL returns 404

Check that:

- The screen node exists.
- The node ID in `/player/{screen}` is correct.
- The node is of the `screen` content type.
- The `signage_player` module is enabled.

### Player loads but shows no slides

Check that:

- The screen has a playlist connected directly or through a screen group.
- The playlist contains playlist items.
- Playlist items are enabled.
- Playlist items are currently inside their configured schedule.
- Playlist items reference image slides.
- Image slides have valid media files.

### Dashboard is not accessible

The dashboard route requires a logged-in user. Log in first, then open:

```text
/dashboard
```

### Screen Access Manager is not accessible

The user must have this permission:

```text
administer signage access
```

Assign it to the correct Drupal role, then clear cache:

```bash
ddev drush cr
```

## License

See [`LICENSE.txt`](LICENSE.txt).
