# Parquetry GeoServer

A GeoServer plugin that exposes the parquetry GeoParquet DataStores as vector
store types in the GeoServer UI and REST configuration.

The data-access code lives in [parquetry](https://github.com/tileverse-io/parquetry)
(the `parquetry-geotools` module); GeoServer auto-discovers its
`DataStoreFactorySpi` from that jar. This project adds the Spring wiring, the
store-edit panels, and the plugin packaging. It registers the GeoParquet,
Iceberg, and STAC stores that `parquetry-geotools` provides, each reading through
tileverse-storage's `RangeReader` SPI (local files, HTTP, S3, Azure, GCS).

Parquetry is a clean-room, bounded-memory Parquet/GeoParquet reader with no
dependency on Hadoop or parquet-java, which suits GeoServer running in a pod with
a gigabyte or two of heap.

## Requirements

- **Java 25**, started with `--enable-preview` and the foreign-memory
  native-access flags. `parquetry-core` is compiled with preview features, so a
  Java 17/21 GeoServer cannot load the plugin.
- **GeoServer 3.0.0 / GeoTools 35.0** (the `provided` versions this plugin builds
  against). The deployment target is GeoServer Cloud on Java 25.
- Maven (use the bundled `./mvnw`).

## Building the plugin

The plugin zip is a flat set of jars to unzip into a GeoServer installation's
`webapps/geoserver/WEB-INF/lib`. It contains the plugin jar and the runtime
dependencies GeoServer/GeoTools do not already ship.

```bash
# Build parquetry-geoserver-<version>-plugin.zip under parquetry-geoserver/target/
make geoserver-plugin

# Equivalent direct invocation
./mvnw -pl :parquetry-geoserver-plugin -am -Passembly package
```

Install it by unzipping into an existing GeoServer, then restart:

```bash
unzip parquetry-geoserver-<version>-plugin.zip -d "$GEOSERVER_HOME/webapps/geoserver/WEB-INF/lib"
```

In the GeoServer UI, go to **Stores > Add new store**; "GeoParquet", "Iceberg",
and "STAC" appear among the vector data sources. See
[parquetry-geoserver/README.md](parquetry-geoserver/README.md) for the store-edit
panels, embedded-Jetty development launcher, and `jetty:run` instructions.

## Demo

A self-contained Docker demo serves six workspaces (GeoParquet, Iceberg, and STAC,
each over local disk and over an S3 emulator):

```bash
make geoserver-demo        # build the plugin, build the image, start GeoServer
# open http://localhost:8080/geoserver  (admin / geoserver)
make geoserver-demo-down   # stop and remove the container
```

`make geoserver-dist` bundles the demo into a customer zip that needs only Docker.

## Coordinates

Published under the `io.tileverse.parquetry.geoserver` group. Current version is
`1.0-SNAPSHOT`.

```xml
<dependency>
  <groupId>io.tileverse.parquetry.geoserver</groupId>
  <artifactId>parquetry-geoserver-plugin</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
```

The parquetry engine it builds on resolves from Maven Central (releases) or the
Central snapshot repository (snapshots); GeoServer and GeoTools resolve from the
OSGeo repository. Both repositories are declared in the POMs.

## Releases

Pushing a tag whose name starts with a digit (`1.0-M1`, `1.0.0`, `2.0-RC1`)
builds, tests, and publishes that exact version to Maven Central, then creates
the matching GitHub Release with the plugin zip and the self-contained demo zip
attached. Any other version is published by hand through the release workflow's
`workflow_dispatch`. SNAPSHOTs publish automatically from `main` once PR
validation passes. See [.github/workflows](.github/workflows/README.md).

## License

**GNU General Public License, version 2 or later** (`GPL-2.0-or-later`), because
this is a GeoServer plugin and reuses GeoServer's GPL-2.0-or-later Wicket
components. The full text is in [LICENSE](LICENSE) and third-party attribution is
in [NOTICE](NOTICE). The parquetry engine it depends on is licensed separately
under the Apache License, Version 2.0.

Part of [Tileverse](https://tileverse.io).
