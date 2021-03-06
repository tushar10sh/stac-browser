# STAC Browser

This is a [Spatio-Temporal Asset Catalog
(STAC)](https://github.com/radiantearth/stac-spec) browser for static catalogs.
It attempts to surface all included data in a user-centric way (an approach
which can inform how data is represented in the evolving spec). It is
implemented as a single page application (SPA) for ease of development and to
limit the overall number of catalog reads necessary when browsing (as catalogs
may be nested and do not necessarily contain references to their parents).

<a href="https://www.netlify.com">
  <img src="https://www.netlify.com/img/global/badges/netlify-light.svg"/>
</a>

## Examples

* [planet.stac.cloud](https://planet.stac.cloud) ([catalog on GitHub](https://github.com/cholmes/pdd-stac/))
* [CBERS](https://cbers.stac.cloud) ([catalog tools on GitHub](https://github.com/fredliporace/cbers-2-stac))
* [Google Earth Engine](https://gee.stac.cloud)
* [sat-api.stac.cloud](https://sat-api.stac.cloud) ([sat-api on GitHub](https://github.com/sat-utils/sat-api))

For a longer list of examples, checkout out [stac.cloud](http://stac.cloud).

## Running

By default, stac-browser will browse the [testbed Planet
catalog](https://raw.githubusercontent.com/cholmes/sample-stac/master/stac/catalog.json)
([GitHub](https://github.com/cholmes/sample-stac/)). To browse your own, set
`CATALOG_URL` when building.

```bash
npm install
CATALOG_URL=http://path/to/catalog.json npm start -- --open
```

Validation will happen against the version of stac defined in the Catalog, Collection or Item
`stac_version` property. If you are running against an older STAC version where the objects
do not contain a `stac_version` property, you'll need to set the `STAC_VERSION` environment
variable e.g.:

```
STAC_VERSION=0.6.0 CATALOG_URL=http://path/to/catalog.json npm start -- --open
```

STAC Browser defaults to using [HTML5 History
Mode](https://router.vuejs.org/guide/essentials/history-mode.html), which can
cause problems on certain web hosts. To use _hash mode_, set
`HISTORY_MODE=hash` when running or building. This will be compatible with
S3, stock Apache, etc.

This package has also been published to npm as [`@radiantearth/stac-browser`](https://www.npmjs.com/package/@radiantearth/stac-browser).

## Other options

### STAC_PROXY_URL

Setting the `STAC_PROXY_URL` allows users to modify the URLs contained in the catalog to point to another location.
For instance, if you are serving a catalog on the local file system at `/home/user/catalog.json`, but want to serve
the data out from a server located at `http://localhost:8888/`, you can use:

```
CATALOG_URL=http://localhost:8888/catalog.json STAC_PROXY_URL="/home/user|http://localhost:8888" npm start -- --open
```

Notice the format of the value: it is the original location and the proxy location separated by the `|` character, i.e. `{original}|{proxy}`.

In this example, any href contained in the STAC (including link or asset hrefs) will replace any occurrence of `/home/user/` with `http://localhost:8888`.

This can also be helpful when proxying a STAC that does not have cors enabled; by using STAC_PROXY_URL you can proxy the original STAC server with one that enables cors
and be able to browse that catalog.

### TILE_SOURCE_TEMPLATE

The `TILE_SOURCE_TEMPLATE` environment variable controls the tile layer that is used to render COGs. If not set, the default value is:

```
https://tiles.rdnt.io/tiles/{z}/{x}/{y}@2x?url={ASSET_HREF}
```

which uses the [tiles.rdnt.io](https://github.com/radiantearth/tiles.rdnt.io) project to serve publicly accessible COGs as tile layers.

The format of this value is a tile layer template with an optional `{ASSET_HREF}` that will be replaced with the COG asset href. For example,
using a local version of [titiler](https://github.com/developmentseed/titiler) to serve local COG files would look something like:

```
TILE_SOURCE_TEMPLATE=http://localhost:8000/cog/tiles/{z}/{x}/{y}?url={ASSET_HREF} npm start -- --open
```

### TILE_PROXY_URL

`TILE_PROXY_URL` is very similar to STAC_PROXY_URL, but is only used for asset hrefs passed into the TILE_SOURCE_TEMPLATE. This enables deployment scenarios where the tiler needs to reference a proxy server by a different name, e.g. in a docker-compose setup with linked containers.

## Building

```bash
CATALOG_URL=http://path/to/catalog.json npm run build
```

## Prerendering

STAC Browser includes the ability to prerender catalog pages to HTML using
[Puppeteer](https://github.com/GoogleChrome/puppeteer) to control a headless
Chromium instance. This facilitates search engine indexing, as metadata and
content will be present in the HTML prior to loading external catalogs.

To prerender, run:

```bash
bin/prerender.js -p <public URL> <catalog URL>
```

`dist/` will contain all assets necessary to host the browser.

After publishing (see below), the generated sitemap can be submitted for
crawling by Google:

```bash
curl http://www.google.com/ping?sitemap=https://planet.stac.cloud/sitemap.txt
```

## Publishing

After building or prerendering, `dist/` will contain all assets necessary to
host the browser. These can be manually copied to your web host of choice.

Alternately, you can publish to [Netlify](https://www.netlify.com/) for free.

First, create a new site:

```bash
node_modules/.bin/netlify init
```

The generated site id will be used as `NETLIFY_SITE_ID` in your environment.

To deploy without prerendering:

```bash
CATALOG_URL=... NETLIFY_SITE_ID=... npm run deploy
```

To deploy a prerendered version you'll also need the target URL:

```bash
CATALOG_URL=... NETLIFY_SITE_ID=... STAC_URL=... npm run deploy-prerendered
```

## Crawling

To facilitate prerendering, STAC Browser includes functionality for crawling
catalogs in `bin/crawl.js`.

As-is, this will just output the type and URL for all entries in the catalog.
In real-world use, you'll probably want to use it as an example and write
custom JavaScript to process each entry (similar to how `bin/prerender.js`
uses it).

## Contributing

STAC Browser uses [Vue](https://vuejs.org/).

Catalogs and collections are rendered using the
[`Catalog`](src/components/Catalog.vue) component in
[`src/components/`](src/components/). Items are rendered using the
[`Item`](src/components/Item.vue) component. Common functionality across both
components exists in [`src/components/common.js`](src/components/common.js).
Mappings between property keys (e.g. `eo:platform`) are defined in
[`src/properties.js`](src/properties.js).