# CubeCart-Visitor-Tracker-v1.1.1
CubeCart Visitor Tracker v1.1.1
# Visitor Tracker for CubeCart

A privacy-friendly visitor analytics plugin for **CubeCart 6**. It records what visitors do in your webshop (pages, products, categories, searches, cart additions, checkout and completed orders) and shows it in a graphical dashboard in the admin area.

- Dashboard with charts (Chart.js is bundled, nothing is loaded from external servers)
- Visitors and page views per day, busiest hours, conversion funnel
- Most viewed products, most added to cart, popular categories, search terms
- Traffic sources (referrers), device types, live "active now" counter
- Click path per visitor (last 2 days)
- Menu item **Bezoekers** in the admin sidebar, under **Rapportage**
- No cookies and no IP addresses stored (see Privacy)

> The admin interface text of this plugin is in Dutch. A Dutch version of this guide is in `README.nl.md`.

## Requirements

- CubeCart 6.x
- PHP 7.4 or newer (developed for and used on PHP 8.4)
- MySQL/MariaDB (the plugin creates one table)
- FTP or file manager access to your server

## Installation

1. Download the latest `visitor_tracker.zip` from the Releases page and unzip it. You get a folder called `visitor_tracker`. (If you cloned this repository instead, you only need the `visitor_tracker/` folder.)
2. Upload that folder to `modules/plugins/` on your server. The result must be:

   ```
   modules/plugins/visitor_tracker/config.xml
   modules/plugins/visitor_tracker/admin/index.inc.php
   modules/plugins/visitor_tracker/hooks/admin.navigation.php
   modules/plugins/visitor_tracker/hooks/class.cubecart.construct.php
   modules/plugins/visitor_tracker/language/module.definitions.xml
   modules/plugins/visitor_tracker/skin/admin/chart.umd.js
   modules/plugins/visitor_tracker/skin/admin/vt_dashboard.tpl
   ```

   Make sure there is **no extra nested folder** (not `visitor_tracker/visitor_tracker/config.xml`), otherwise CubeCart will not find the plugin.
3. In the admin go to **Extensions > Plugins**. *Visitor Tracker* should be listed with its name and version.
4. Set the plugin to **Enabled**.
5. Go to **Settings > Manage Hooks** and check that *Visitor Tracker* has two hooks: `class.cubecart.construct` and `admin.navigation`, both enabled. If not, click *Revert to default hook configuration*.
6. **Open the dashboard once** (Extensions > Plugins > Visitor Tracker, or the new menu item). This creates the database table and switches tracking on. Tracking does not start before this step.
7. Clear the CubeCart cache (delete the files in `cache/`, not the folders) and refresh the admin with Ctrl+F5. The menu item **Bezoekers** appears under **Rapportage**.

## Usage

- Open the dashboard via the menu item **Bezoekers** (or Extensions > Plugins > Visitor Tracker).
- Choose the period with the buttons at the top: today, 7, 30 or 90 days.
- Tab **Instellingen** (settings):
  - **Status**: enable or disable the plugin
  - **Bewaartermijn (dagen)**: how long data is kept (default 90 days, minimum 7). Older data is removed automatically.
  - **IP-adressen uitsluiten**: comma-separated IP addresses that should not be tracked, for example your own.

To test: visit your shop in a private browser window, view a product, add it to the cart, then refresh the dashboard. The "active now" counter should show at least 1.

## Privacy

- No cookies are set and no IP addresses or user agents are stored.
- A visitor is an anonymous hash of IP + browser + date + site path that **changes every day**. A returning visitor on another day counts as a new visitor.
- Known bots and crawlers (search engines, SEO tools, monitors) are ignored.
- Because no personal data is stored this usually needs no consent banner, but rules differ per country and situation. Check this for your own shop and mention analytics in your privacy statement.

## Troubleshooting

**The plugin does not appear in the plugin list**
Check that `config.xml` is directly inside `modules/plugins/visitor_tracker/` (no nested folder) and that it was uploaded completely. You can test by opening `https://your-shop/modules/plugins/visitor_tracker/config.xml` in a browser. Also clear the cache.

**"Invalid module details" when enabling**
`config.xml` could not be read. Re-upload it from the zip.

**Blank page when opening the dashboard**
A PHP error is hidden. Check your hosting error log. After changing a template file, also delete the compiled templates in `cache/skin/`, because Smarty may keep using an old compiled version.

**The menu item "Bezoekers" does not show**
Check *Settings > Manage Hooks* for the `admin.navigation` hook, clear the cache and refresh with Ctrl+F5. The item is added to the group named *Rapportage* (or a group containing "report" or "statistics"). If your menu group has another name, the plugin creates a group called *Rapportage*.

**The dashboard stays empty**
- Did you open the dashboard once after enabling? (This creates the hidden file `.installed2` in the plugin folder; FTP programs may hide dotfiles.)
- Is your own IP address in the exclude list?
- Is the hook `class.cubecart.construct` enabled in *Manage Hooks*?
- Are all your test visits made with a normal browser (not a script or a tool with "bot" in its name)?

**Many visits at night**
Some scrapers use a normal browser name and cannot be recognised. If the "busiest hours" chart shows a night peak, those are probably not real customers.

## Uninstall

1. Set the plugin to **Disabled** in Extensions > Plugins.
2. Delete the folder `modules/plugins/visitor_tracker`.
3. Optionally remove the data: drop the table `<your prefix>CubeCart_vt_events` (for example `cc_CubeCart_vt_events`) in phpMyAdmin.

## Notes and limitations

- Page types are detected from CubeCart's own request parameters, so it also works with SEO-friendly URLs. Pages that are not recognised are stored as "other" with their internal page name.
- Time stamps use the PHP server time zone.
- Order values and revenue are not tracked in this version.
- This is an independent plugin and not an official CubeCart product. Test it on a copy of your shop first and keep a backup.

## License

Released under the [MIT License](LICENSE). Copyright (c) 2026 MikeNL.
The bundled Chart.js library is also MIT licensed, see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

CubeCart is a trademark of its respective owner. This plugin is an independent project and is not affiliated with or endorsed by CubeCart Limited.

## Credits

- Plugin by MikeNL.
- [Chart.js](https://www.chartjs.org/) 4.4.7 (MIT license) is bundled for the charts.
