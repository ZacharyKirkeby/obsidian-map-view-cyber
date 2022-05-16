# Obsidian.md Map View

[!["Buy Me A Coffee"](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://www.buymeacoffee.com/esm7)

## Intro

This plugin introduces an **interactive map view** for [Obsidian.md](https://obsidian.md/).
It searches your notes for encoded geolocations (see below), places them as markers on a map and offers multiple tools to interact with them.

It effectively turns your Obsidian vault into a **personal GIS system** that adds a geographical layer to your notes, journals, trip planning and pretty much anything you use Obsidian for.

You can set different icons for different note types according to custom rules, save geolocations from a variety of sources (Google Maps and many others), save custom views, switch between map layers, run powerful queries and so much more.

![](img/sample.png)

![](img/intro.gif)

I wrote this plugin because I wanted my ever-growing Zettelkasten to be able to answer questions like...

- When I get recommendations about cool places to visit, how do I save them in a way that I can recall later?
- When I'm visiting somewhere, what interesting places do I know in the area?
- When I'm conducting research for planning a trip, how do I lay out on a map options for places to eat, hike or sleep, combine them with prior knowledge, and save them for future reference?

Map View can integrate with your note-taking flow in order to answer all of these questions and much more.

Just like the Obsidian graph view lets you visualize associative relations between some of your notes, the map view lets you visualize geographic ones.

## With Obsidian Mobile

The main limitation of the plugin right now is that the Obsidian Mobile app has no location permission, so on mobile you cannot see your current location.

Please help us ask the Obsidian developers to get these permissions added, as it will make this plugin so much better!

## Support the Development

If you want to support the development of this plugin, please consider to [buy me a coffee](https://www.buymeacoffee.com/esm7).

## Parsing Location Data

Map View provides [several methods to log locations in notes](#adding-a-location-to-a-note) and can manage the technicalities for you.
You can skip to that section if you want to just get started, or continue reading the more technical explanation below.

So, the plugin works by scanning your notes and parsing two types of location data.

First is a location tag in a note's [front matter](https://help.obsidian.md/Advanced+topics/YAML+front+matter):

```yaml
---
location: [40.6892494,-74.0466891]
---
```

This is useful for notes that represent a single specific location.
It's also compatible with the way other useful plugins like [obsidian-leaflet](https://github.com/valentine195/obsidian-leaflet-plugin) read locations, and allows some interoperability.

Another way that the plugin parses location data is through **inline location URLs** in the format of `[link-name](geo:40.68,-74.04)`, which allow multiple markers in the same note.
To prevent the need to scan the full content of all your notes, it requires an empty `locations:` tag in the note front matter ('locations' and not 'location').
Example:

```
---
locations:
---

# Trip Plan

Point 1: [Hudson River](geo:42.277578,-76.1598107)
... more note content ...

Point 2: [New Haven](geo:41.2982672,-72.9991356)
```

Notes with multiple markers will contain multiple markers on the map with the same note name, and clicking on the marker will jump to the correct location within the note.

For many cases inline locations are superior because `geo:` is a [native URL scheme](https://en.wikipedia.org/wiki/Geo_URI_scheme), so if you click it in Obsidian (including mobile), your default maps app (or an app selector for a location) will be triggered.
The front matter method, however, is currently better if you want interoperability with plugins that use it, if you want to store lots of filterable meta-data on a location, or if you heavily express yourself with links.

Inline locations also support **inline tags** in the format of `tag:dogs`. For example:

```
Point 1: [Hudson River](geo:42.277578,-76.1598107) tag:dogs
```

This will add the tag `#dogs` specifically to that point, regardless of the note's own tags.
This is useful for notes that contain tags of different types (e.g. a trip log with various types of locations).
Note that the `tag:` format should be used **without** the `#` sign, because this sets the tag for the whole note.
Map View will add `#` for the purpose of setting marker icons, as explained below.
Also note that inline tags do not affect the files that are searched based on the entered search query, so in the example above, if you enter '#dogs' in the search box, your note would **not** appear.

Note: older versions of this plugin used the notation `location: ...` for inline locations.
This notation is still supported but the standard `geo:` one is the default and encouraged one.

Multiple inline tags can be separated with a whitespace: `[](geo:42.2,-76.15) tag:dogs tag:trip`.

Multiple inline locations can be added in the same line, and the tags that follow them will be associated to the location on the left, but the right-click editor context menu will not know to choose the location that was selected.

## Adding a Location to a Note

Map View offers many ways to add locations to notes.

### Anywhere in Obsidian

Map View adds an Obsidian command named "New geolocation note", which you can map to a hotkey and use anywhere in Obsidian.

This opens a dialog on which you can search (address or location based on your [configured geocoding provider](#changing-a-geocoding-provider)) or paste a URL using the built-in or custom [URL parsing rules](#url-parsing-rules).

![](img/new-note-popup.gif)

### In an Existing Note

There are multiple ways to add a geolocation to an existing note.

1. Create an inline geolocation link in the format of `[](geo:)`, and if you start typing inside the link name (the brackets), Map View will initiate a location search. If you confirm one of the options, it will fill-in the location's coordinates. See more on this in the ["In-Note Location Search"](#in-note-location-search--auto-complete) section below.

To make this more streamlined, Map View adds to Obsidian a command named 'Add inline geolocation link' which you can map to a keyboard shortcut.

2. Add a front matter geolocation by using the Obsidian command 'Add geolocation (front matter) to current note'. This opens the same dialog as "new geolocation note" which allows you to search for a location name or paste a [URL parsing rules](#url-parsing-rules).

3. If you have a location in some other mapping service that you wish to log, e.g. from Google Maps, you can copy the URL or "lat,lng" geolocation from that service, right-click in your note and select "Paste as Geolocation". The supported services are configurable, see [below](#url-parsing-rules) for more details.

### From the Map

The map offers several tools to create notes.

1. Use "new note here" when right-clicking the map. This will create a new note (based on the template you can change in the settings) with the location you clicked. You can create either an empty note with a front matter (single geolocation) or an empty note with an inline geolocation.

![](img/new-note.png)

Note that the map can be searched using the tool on the upper-right side.

![](img/search.png)

2. If you prefer to enter geolocations as text, use one of the "copy geolocation" options when you right-click the map. If you use "copy geolocation", just remember you need the note to start with a front matter that has an empty `locations:` line.
 
![](img/copy.png)


### Paste as Geolocation

Map View monitors the system clipboard, and when a URL is detected to have an encoded geolocation (e.g. a Google Maps "lat, lng" location), a "Paste as geolocation" entry is added to the editor context menu.
For example, if you right-click a location in Google Maps and click the first item in the menu (coordinates in lat,lng format), you can then paste it as a geolocation inside a note.

Alternatively, you can right-click a URL that is already present in a note and choose "Convert to geolocation".

By default Map View can parse URLs from two services: the OpenStreetMap "show address" link and a generic "lat, lng" encoding used by many URLs.

## Queries

Map View supports powerful queries that are roughly similar to Obsidian's query format.

The query string can contain the following *search operators*:

- `tag:#...` to search for notes or markers tagged with a specific tag. This works on both whole notes (`#hiking`) and inline tags for specific markers (`tag:hiking`).
- `path:...` to search by the note path. Case insensitive. This operator will include all the markers in the path that matches the query.
- `linkedto:...` includes notes that contain a specific link. Case insensitive. This operator will include a note (with all the markers in it) if it has a link name that matches the query. For example, if you have a note named `Cave Hikes` and you have geolocated notes that **link to it** (e.g. include `[[Cave Hikes]]` as a link), include them by the filter `linkedto:"Cave Hikes"` or a portion of that name.
- `linkedfrom:...` includes notes that are linked from a specific note, and also the origin note itself. Case insensitive. This operator will include a note (with all the markers in it) if it is linked **from** the note mentioned in the query. For example, if you have a note named `Trip to Italy` with links to various geolocated notes (e.g. of places you want to visit or a trip log), the query `linkedfrom:"Trip to Italy"` will filter only for those markers.

You can combine the above with *logical operators*: `AND`, `OR`, `NOT`, and grouping with parenthesis.
**This differs from Obsidian's own query language which uses `-` instead of `NOT`.**

For examples:

- `linkedfrom:"Trip to Italy" AND tag:#wine` can include places you linked from your trip to Italy, or are within that note itself, and are tagged with `#wine`.
- `tag:#hike AND tag:#dogs` can include hikes you marked as suitable for dogs.
- `tag:#hike AND (tag:#dogs OR tag:#amazing) AND NOT path:"bad places"`

There are many creative ways to organize your notes with geolocations that utilize these query abilities.
You may represent location types with tags (e.g. `#hike` or `#restaurant`), or use tags to represent traits of places (`#hike/summer`, `#hike/easy`).
You can use paths for indexes using Zettelkasten back links (e.g. link to "Hikes that I Want" from notes you want to denote as such), then use `linkedto:` to find places that link to it.
And/or you can have notes to plan a trip and link to places from it, then use `linkedfrom:` to focus on your plan.

In all cases you can [save presets](#Presets) that include the filter or sub-filters of it.

## Marker Icons

Map View allows you to customize notes' map marker icons based on a powerful rules system. These rules can be edited using the plugin's settings pane or edited as JSON for some even more fine-grained control.

Icons are based on [Font Awesome](https://fontawesome.com/), so to add a marker icon you'll need to find its name in the Font Awesome catalog.
Additionally, there are various marker properties (shape, color and more) that are based on [Leaflet.ExtraMarkers](https://github.com/coryasilva/Leaflet.ExtraMarkers#properties).

To change the map marker icons for your notes, go to the Map View settings and scroll to Marker Icon Rules.

A single marker is defined with a *tag pattern* and *icon details*.
The tag pattern is usually a tag name (e.g. `#dogs`), but it can also be with a wildcard (e.g. `#trips/*`).
Icon details are a few properties: icon name (taken from the Font Awesome catalog), color and shape.

![](img/marker-rules.png)

A single marker is defined in the following JSON structure:
`{"prefix": "fas", "icon": "fa-bus", "shape": "circle", "color": "red"}`

To add a marker with a bus icon, click New Icon Rule, search Font Awesome (in the link above) for 'bus', choose [this icon](https://fontawesome.com/v5.15/icons/bus?style=solid), then see that its name is `fa-bus`.
Once you enter `fa-bus` in the icon name, you should immediately see your icon in the preview.
To make this icon apply for notes with the `#travel` tag, type `#travel` in the Tag Name box.

### Tag Rules

To apply an icon to a note with geolocation data, Map View scans the complete list of rules by their order, always starting from `default`.
A rule matches if the tag that it lists is included in the note, and then the rule's fields will overwrite the corresponding fields of the previous matching rules, until all rules were scanned.
This allows you to set rules that change just some properties of the icons, e.g. some rules change the shape according to some tags, some change the color etc.

Here's the example I provide as a probably-not-useful default in the plugin:

```
	{ruleName: "default", preset: true, iconDetails: {"prefix": "fas", "icon": "fa-circle", "markerColor": "blue"}},
	{ruleName: "#trip", preset: false, iconDetails: {"prefix": "fas", "icon": "fa-hiking", "markerColor": "green"}},
	{ruleName: "#trip-water", preset: false, iconDetails: {"prefix": "fas", "markerColor": "blue"}},
	{ruleName: "#dogs", preset: false, iconDetails: {"prefix": "fas", "icon": "fa-paw"}},
```

This means that all notes will have a blue `fa-circle` icon by default.
However, a note with the `#trip` tag will have a green `fa-hiking` icon.
Then, a note that has both the `#trip` and `#trip-water` tags will have a `fa-hiking` marker (when the `#trip` rule is applied), but a **blue** marker, because the `#trip-water` overwrites the `markerColor` that the previous `#trip` rule has set.

Tag rules also support wildcards, e.g. a rule in the form of `"#food*": {...}` will match notes with the tag `#food`, `#food/pizza`, `#food/vegan`, `#food-to-try` etc.

The settings also allow advanced users to manually edit the configuration tree, and there you can use more properties based on the [Leaflet.ExtraMarkers](https://github.com/coryasilva/Leaflet.ExtraMarkers#properties) properties. Manual edits update the GUI in real-time.

## In-Note Location Search & Auto-Complete

Map View adds an Obsidian command named 'Add inline geolocation link', that you can (and encouraged) to map to a keyboard shortcut, e.g. `Ctrl+L` or `Ctrl+Shift+L`.
This command inserts an empty inline location template: `[](geo:)`.

When editing an inline location in this format, whether if you added it manually or using the command, if you start entering a link name, Map View will start offering locations based on a geocoding service.
Selecting one of the suggestions will fill-in the coordinates of the chosen locations, *not* change your link name (assuming you prefer your own name rather than the formal one offered by the geocoding service), and jump the cursor to beyond the link so you can continue typing.

![](img/geosearch-suggest.gif)

If your note is not yet marked as one including locations (by a `locations:`) tag in the front matter, this is added automatically.

### Changing a Geocoding Provider

By default, Map View is configured to use OpenStreetMap as the search provider.
If you prefer to use the Google Maps search, you can configure this in the plugin settings.

The Google Geocoding API is practically free or very cheap for normal note-taking usage, but you'd need to setup a project and obtain an API key from Google.
See [here](https://developers.google.com/maps/documentation/javascript/get-api-key) for more details.

**Note:** usage of any geocoding provider is at your own risk, and it's your own responsibility to verify you are not violating the service's terms of usage.

## Map Sources

By default, Map View uses the [CartoDB Voyager Map](https://github.com/CartoDB/basemap-styles), which is free for up to 75K requests per month.
However, you can change or add map sources in the configuration with any service that has a tiles API using a standard URL syntax.

There are many services of localized, specialized or just beautifully-rendered maps that you can use, sometimes following a free registration.
See a pretty comprehensive list [here](https://wiki.openstreetmap.org/wiki/Tiles).

Although that's the case with this plugin in general, it's worth noting explicitly that using 3rd party map data properly, and making sure you are not violating any terms of use, is your own responsibility.

Note that Google Maps is not in that list, because although it does provide the same standard form of static tiles in the same URL format, the Google Maps terms of service makes it difficult to legally bundle the maps in an application.

If you have multiple map sources, they can be switched from the View pane.
Additionally, you can set an optional different dark theme URL for each map source.
If a dark theme is detected, or if you specifically change the map source type to Dark (using the drop down in the View pane), you will get the Dark URL if one is configured.

## Presets

If there is a map state you would like to save and easily come back to, you can save it as a preset.
To do so, open the Presets pane in the main plugin's controls, and click 'Save as' to save the current view with a name you can easily go back to.

If you enter an already-existing name, that preset will be overwritten.

The saved preset includes the map state (zoom & pan), the filters used, and if you check the box in the "save as" dialog -- also the chosen map source.
If you do not include the map source as part of the preset, switching to the newly-saved preset will use the currently-selected map source.

Presets *do not* store the map's theme (light/dark).

The Default preset is special; you can save it using the 'Save as Default' button, and come back to it by clicking the Reset button, by choosing the Default preset from the box, or by opening a fresh Map View that has no previously saved state.

## Open In

Many context menus of Map View display a customizable Open In list, which can open a given location in external sources.
These sources can be Google Maps, OpenStreetMap, specialized mapping tools or pretty much anything you use for viewing locations.

![](img/open-in.png)

The Open In list is shown:
- When right-clicking on the map.
- When right-clicking a marker on the map.
- When right-clicking a line in a note that has a location.
- In the context menu of a note that has a front matter location.

This list can be edited through the plugin's settings menu, with a name that will be displayed in the context menus and a URL pattern. The URL pattern has two parameters -- `{x}` and `{y}` -- that will be replaced by the latitude and longitude of the clicked location.

![](img/custom-open-in.png)

Popular choices may be:
- Google Maps: `https://maps.google.com/?q={x},{y}`
- OpenStreetMap: `https://www.openstreetmap.org/#map=16/{x}/{y}` (replace `16` with your preferred zoom level)
- Waze (online dropped pin): `https://ul.waze.com/ul?ll={x}%2C{y}&navigate=yes&zoom=17` (replace `17` with your preferred zoom level)

And you can figure out many other mapping services just by inspecting the URL.

## URL Parsing Rules

As described above, Map View uses *URL parsing rules* in several places to provide the ability to parse URLs (or other strings) from external sources and convert them to standard geolocations.

1. When right-clicking a line with a recognized link, a "Convert to Geolocation" entry will be shown in the editor context menu.
2. When a recognized link is detected in the system clipboard, a "Paste as Geolocation" entry will be added in the editor context menu.
3. In the "New geolocation note" dialog, pasting a supported URL will parse the geolocation.

URL parsing rules can be configured in the plugin's configuration pane and requires familiarity with regular expressions.

The syntax expects two captures group and you can configure if they are parsed as `lat, lng` (most common) or `lng, lat`.

And if you think your added regular expressions are solid enough, please add them to the plugin using a PR so others can benefit!

![](img/url-parsing.png)

## Relation to Other Obsidian Plugins

When thinking about Obsidian and maps, the first plugin that comes to mind is [Obsidian Leaflet](https://github.com/valentine195/obsidian-leaflet-plugin).
That plugin is great at rendering maps based on data within a note, with great customization options.
It can also scan for data inside a directory which gives even more power.
In contrast, Obsidian Map View is focused on showing and interacting with your notes geographically.

Another relevant plugin is [Obsidian Map](https://github.com/Darakah/obsidian-map) which seems to focus on powerful tools for map drawing.

## Wishlist

As noted in the disclaimer above, my wishlist for this plugin is huge and I'm unlikely to get to it all.
There are so many things that I want it to do, and so little time...

- Better interoperability with Obsidian Leaflet: support for marker image files, locations as an array and `marker` tags.
- A side bar with note summaries linked to the map view.

## Changelog

### 2.0.0

**Unreleased version, work in progress.**

Mostly feature-complete, this README needs to get updated.

- Queries (TODO document)
- Queries are marker-based and not file-based. This will be a breaking change for some users.
- "Follow active note"
- Showing the note name is now optional (https://github.com/esm7/obsidian-map-view/issues/75). I wish this could be in the same popup as the preview, but currently I don't see how to do this.
- Fixed issues with front matter tag support (https://github.com/esm7/obsidian-map-view/issues/72) (thanks @gentlegiantJGC!)
- Added a configuration for the max zoom of a tile layer (thanks @gentlegiantJGC!)
- The map search tool now uses the same search window as "New geolocation note", which beyond the configured geocoding service, also does URL parsing.
  - Except the bonus of making the UI more uniform, this is very important for usability, especially on mobile. e.g. you can use it to get your location from another app and use it to create notes or explore around.
  - A "search active map view" command was added (available when a map view is focused) so a keyboard shortcut can be assigned to the map search.
- Several UI improvements:
  - The map control panel is now prettier and smaller when unused.
  - The note name and cluster popups now follow the Obsidian theme.

### 1.6.0

- Powerful query filters are here! You can now combine operators of tags and paths with logical operators.
- New "focus note in Map View" context menu for notes, which opens Map View with a path query that focuses that specific note.

### 1.5.0

- Map View now saves its state to the Obsidian back/forward mechanism (unless configured not to).
- Fixed an issue with markers temporarily duplicating (until the map is refreshed) when their note is renamed.

### 1.4.0

- Replaced OpenStreetMap with CartoDB as the new default map source (https://github.com/esm7/obsidian-map-view/issues/59).
- Map View now displays a more useful error when tiles fail to load.
- Removed the "Google Maps" default URL parsing rule because apparently it was incorrect (https://github.com/esm7/obsidian-map-view/issues/57). It is now replaced by a more generic "lat,lng" rule that can also be used with Google Maps, *not* by parsing the URL but by right-clicking the map in Google Maps and choosing the first menu item that copies the coordinates.
- Hovering on a map marker now opens the Obsidian note preview, scrolled to the correct line (https://github.com/esm7/obsidian-map-view/issues/60). This is configurable in the settings and comes *in addition* to the existing note pop-up (without a snippet), because the preview does not include the note name.
	- **Important note:** this replaces the "snippet" functionality of previous versions.
- Marker clusters now show a preview of their own; they show a popup with the first 4 icons in the cluster.

### 1.3.0

- Introduction of **Presets** as a way to save the state of the view including the map state, filters and optionally the chosen map source.
  - As part of this, the 'default view' functionality was converted to a built-in 'Default' preset and the UI was revamped accordingly.
- As part of the above, the internal structure of the plugin had to be reorganized and many features needed to be rewired. **I put major effort to test the entire functionality of the plugin, but pretty much anything could break and I may have missed some bugs.** Please open issues if something had stopped working.
- When using `{{query}}` in file name templates, the file name is sanitized so the creation won't fail on illegal strings.

### 1.2.1

- The "new geolocation note" dialog can now also be used to add a location to an existing note, both via a new Obsidian command and a file menu action (https://github.com/esm7/obsidian-map-view/issues/46).
- Added a `{{query}}` file name template option to auto-add the search query for new location notes (https://github.com/esm7/obsidian-map-view/issues/45).

### 1.2.0

- Added a "new geolocation note" Obsidian command with an interactive search and URL parser.
- Fixed a bug of highlighting more than the intended line after clicking a map marker and opening a note.
- Fixed a bug of inline tags not recognized if immediately followed by a dot (which is a common pattern).
- Fixed an exception that could prevent settings to be applied after closing the settings dialog.

### 1.1.0

- The plugin now supports a configurable list of map sources, switchable from the View pane on the map, including optional separate URLs for light & dark themes.
- Support dark mode using a CSS hue revert, as [recommended for Leaflet](https://gist.github.com/BrendonKoz/b1df234fe3ee388b402cd8e98f7eedbd) and as done by obsidian-leaflet.
- [Support multiple inline locations per line](https://github.com/esm7/obsidian-map-view/issues/35).
- [Support multiple tags per inline location](https://github.com/esm7/obsidian-map-view/issues/32).
- Fixed an [issue](https://github.com/esm7/obsidian-map-view/issues/30) of inline tags not properly including the dash (`-`) character.
- Popups now have a close button (to make them more mobile-friendly).

### 1.0.0

- UI revisions and cleanups to make it easier for new users. **Some changes break existing notions.**
  - Map View now treats "inline locations" as the default. It is first in the menus, and front matter actions are listed as "front matter".
  - "Copy as coordinates" was removed, believing it's not really useful anymore. Please drop me a note if you find it important.
  - Several other tweaks to make the plugin easier to use.
- At last, a settings UI for editing marker icons! Check it out under the plugin settings.
  - In order to do this, the rule format in the plugin's data file had to be changed. The plugin will auto-convert your existing rules to the new format when first launched. After this conversion, *do not* manually enter rules under the old key!
  - You may still edit rules manually, through the plugin's "edit marker icons as JSON" box or directly in the data file, but **only use the new key** `markerIconRules`.
- Added an inline geolocation search, which can get results from either OSM or Google (with an API key), and a command to insert a geolocation.
- With relation to the above, the search tool in the map can now use the selected geocoding service too, meaning you can use Google Maps to search for locations if you have an API key.
- New "paste as geolocation" and "convert to geolocation" right-click editor menu items, that can automatically convert URLs in the clipboard or URLs in the note to a geolocation link.
  - The rules that convert URLs to geolocations are configurable in the settings.

