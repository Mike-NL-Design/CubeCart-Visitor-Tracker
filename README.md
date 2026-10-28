# Visitor Tracker for CubeCart

buymeacoffee.com/nlbiertjenl

A privacy-friendly visitor analytics plugin for **CubeCart 6**. It records what visitors do in your webshop and shows it in a graphical dashboard in the admin area, together with revenue per product from your own orders.

- Dashboard with charts (Chart.js is bundled, nothing is loaded from external servers)
- Visitors and page views per day, busiest hours, conversion funnel
- Most viewed products, most added to cart, popular categories, search terms, traffic sources, devices
- **Revenue per product** (views, added to cart, units sold, revenue) and revenue per day, read from your CubeCart orders
- **Visitor countries** via a local GeoIP database (or Cloudflare), no IP addresses stored
- **Abandoned carts**: carts started vs. abandoned per day and per product, plus unpaid orders
- **CSV export** of every report (Excel-NL or standard format)
- Live "active now" counter and click path per visitor
- Menu item **Bezoekers** in the admin sidebar, under **Rapportage**
- No cookies and no IP addresses stored (see Privacy)

> The admin interface text of this plugin is in Dutch. A Dutch version of this guide is in `README.nl.md`.

## Requirements

- CubeCart 6.x
- PHP 7.4 or newer (developed for and used on PHP 8.4)
- MySQL/MariaDB (the plugin creates one table)
- FTP or file manager access to your server
- Optional: a free GeoIP country database for visitor countries (see GeoIP)

## Installation

1. Download the latest `visitor_tracker.zip` from the Releases page and unzip it. You get a folder called `visitor_tracker`. (If you cloned this repository instead, you only need the `visitor_tracker/` folder.)
2. Upload that folder to `modules/plugins/` on your server. The result must be:

   ```
   modules/plugins/visitor_tracker/config.xml
   modules/plugins/visitor_tracker/admin/index.inc.php
   modules/plugins/visitor_tracker/admin/vt_data.inc.php
   modules/plugins/visitor_tracker/hooks/admin.navigation.php
   modules/plugins/visitor_tracker/hooks/class.cubecart.construct.php
   modules/plugins/visitor_tracker/language/module.definitions.xml
   modules/plugins/visitor_tracker/skin/admin/chart.umd.js
   modules/plugins/visitor_tracker/skin/admin/vt_overview.tpl
   modules/plugins/visitor_tracker/lib/MaxMind/...
   modules/plugins/visitor_tracker/geoip/README.txt
   ```

   Make sure there is **no extra nested folder** (not `visitor_tracker/visitor_tracker/config.xml`), otherwise CubeCart will not find the plugin.
3. In the admin go to **Extensions > Plugins**. *Visitor Tracker* should be listed with its name and version.
4. Set the plugin to **Enabled**.
5. Go to **Settings > Manage Hooks** and check that *Visitor Tracker* has two hooks: `class.cubecart.construct` and `admin.navigation`, both enabled. If not, click *Revert to default hook configuration*.
6. **Open the dashboard once** (Extensions > Plugins > Visitor Tracker, or the new menu item). This creates the database table and switches tracking on. Tracking does not start before this step.
7. Clear the CubeCart cache (delete the files in `cache/`, not the folders) and refresh the admin with Ctrl+F5. The menu item **Bezoekers** appears under **Rapportage**.

### Updating from an earlier version (1.0 - 1.2)

1. Upload the new files over the old ones. Delete the old template `skin/admin/vt_dashboard.tpl` (it is replaced by `vt_overview.tpl`).
2. Clear the cache, including the compiled templates in `cache/skin/`.
3. Open the dashboard once. The database is upgraded automatically (a `country` column is added; existing data is kept).
4. Optional: add a GeoIP database (see below) to start collecting countries. Visits recorded before that have no country.

## Usage

Open the dashboard via the menu item **Bezoekers** (or Extensions > Plugins > Visitor Tracker). Choose the period with the buttons at the top (today = last 24 hours, 7, 30 or 90 days). The sections:

- **Overzicht**: visitors, page views, funnel, busiest hours, top products, categories, search terms, referrers, devices and the click path per visitor.
- **Producten & omzet**: revenue and orders per day, top products by revenue, and a table with views, added to cart, units sold, revenue and units sold per 100 views.
- **Landen**: visitors, page views and buyers per country.
- **Winkelwagens**: carts started and abandoned per day and per product, and unpaid shop orders.
- **Export**: CSV downloads (see below).

Tab **Instellingen** (settings):
- **Status**: enable or disable the plugin
- **Bewaartermijn (dagen)**: how long data is kept (default 90 days, minimum 7). Older data is removed automatically.
- **IP-adressen uitsluiten**: comma-separated IP addresses that should not be tracked, for example your own.

To test: visit your shop in a private browser window, view a product, add it to the cart, then refresh the dashboard. The "active now" counter should show at least 1.

### Revenue per product

Revenue is read from the CubeCart tables `order_summary` and `order_inventory`, not from the tracker. Only paid orders count (status *Processing* or *Complete*). Product revenue is the stored line price times quantity, so it is **without VAT, shipping and discounts**. The total revenue tile uses the order totals. All amounts are shown in the shop's default currency; if you accept several currencies, the sums are not converted. If the order tables are not found (for example in a very different CubeCart version), the plugin shows a notice and the other reports keep working.

### Visitor countries (GeoIP)

Countries are resolved on the server when a page is viewed; only the 2-letter country code is stored, never the IP address, and nothing is sent to an outside service.

1. Download the free **IP to Country Lite** database (MMDB format) from https://db-ip.com/db/download/ip-to-country-lite. (Alternative: MaxMind GeoLite2 Country, which needs a free account.)
2. Put the `.mmdb` file (or the `.mmdb.gz`, which is unpacked automatically when you open the dashboard) in `modules/plugins/visitor_tracker/geoip/`. Only one `.mmdb` file should be present.
3. New visits now get a country. Update the file every few months.

If your shop is behind Cloudflare (or CloudFront) the country header of that service is used instead and no database is needed. Without either, the *Landen* section stays empty. The DB-IP Lite license (CC BY 4.0) requires attribution; the dashboard shows it automatically when a DB-IP file is used.

### Abandoned carts

A visitor counts as *abandoned* when they added a product to the cart (or reached the checkout) but did not reach the order-complete page the same day. Because the anonymous visitor id changes every day, someone who adds to the cart before midnight and pays after midnight counts as abandoned. The section also lists **unpaid shop orders** (status *Pending*) older than one hour; this includes orders that wait for a bank transfer.

### CSV export

The *Export* tab offers: per day, all products, countries, abandoned carts per product, search terms, referrers and raw events (max. 50,000 rows, without IP addresses). Choose **Excel (NL)** for semicolon-separated files with decimal commas (opens correctly in Dutch Excel), or **Standaard** for comma-separated files with decimal points. Cells starting with `= + - @` are prefixed with an apostrophe so search terms can never run as spreadsheet formulas.

## Privacy

- No cookies are set and no IP addresses or user agents are stored.
- A visitor is an anonymous hash of IP + browser + date + site path that **changes every day**. A returning visitor on another day counts as a new visitor.
- The country (2 letters) is derived from the IP address at the moment of the visit and stored instead of it.
- Known bots and crawlers (search engines, SEO tools, monitors) are ignored.
- Because no personal data is stored this usually needs no consent banner, but rules differ per country and situation. Check this for your own shop and mention analytics in your privacy statement.
- The unpaid-orders list shows only the order number, date and total, no customer data.

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

**The Landen section is empty**
Add a GeoIP database to `geoip/` (see above) and make sure only one `.mmdb` file is there. Only visits after that get a country. A corrupt database file never breaks the shop: visits are still recorded, just without a country.

**No revenue is shown**
The plugin reads CubeCart's own order tables. Check that you have paid orders (status Processing or Complete) in the selected period. If a notice says the order data was not found, open an issue with your CubeCart version.

**No abandoned carts although visitors added products**
"Added to cart" is detected from the `add` field of the add-to-cart request. If your skin uses a different form, open an issue.

**Many visits at night**
Some scrapers use a normal browser name and cannot be recognised. If the "busiest hours" chart shows a night peak, those are probably not real customers.

## Uninstall

1. Set the plugin to **Disabled** in Extensions > Plugins.
2. Delete the folder `modules/plugins/visitor_tracker`.
3. Optionally remove the data: drop the table `<your prefix>CubeCart_vt_events` (for example `cc_CubeCart_vt_events`) in phpMyAdmin.

## Notes and limitations

- Page types are detected from CubeCart's own request parameters, so it also works with SEO-friendly URLs. Pages that are not recognised are stored as "other" with their internal page name.
- Time stamps use the PHP server time zone.
- Visitor ids change daily, so there are no multi-day visitor journeys or returning-visitor statistics.
- Revenue is not converted between currencies.
- This is an independent plugin and not an official CubeCart product. Test it on a copy of your shop first and keep a backup.

## License

Released under the [MIT License](LICENSE). Copyright (c) 2026 MikeNL.
The bundled libraries Chart.js (MIT) and MaxMind DB Reader for PHP (Apache-2.0) keep their own licenses, see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

CubeCart is a trademark of its respective owner. This plugin is an independent project and is not affiliated with or endorsed by CubeCart Limited.

## Credits

- Plugin by MikeNL.
- [Chart.js](https://www.chartjs.org/) 4.4.7 for the charts.
- [MaxMind DB Reader for PHP](https://github.com/maxmind/MaxMind-DB-Reader-php) 1.15.0 for GeoIP lookups.
- Country data (optional, not included): [DB-IP](https://db-ip.com) IP to Country Lite or MaxMind GeoLite2.

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
