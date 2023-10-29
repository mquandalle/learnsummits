<script lang="ts">
  import { onMount } from "svelte";
  import "maplibre-gl/dist/maplibre-gl.css";

  let map: InstanceType<typeof import("maplibre-gl").Map>;

  onMount(async () => {
    const maplibregl = await import("maplibre-gl");
    map = new maplibregl.Map({
      container: "map",
      style:
        "https://api.maptiler.com/maps/18720ec3-70af-4b21-801d-ffccd265782d/style.json?key=2QY0a1dK2gB027NRIOkD",
      center: [6.7, 45.6],
      zoom: 12,
      pitch: 45,
    });

    map.on("load", function () {
      // Ajoutez le layer pour le terrain en 3D
      map.addSource("terrain-source", {
        type: "raster-dem",
        url: "https://api.maptiler.com/tiles/terrain-rgb-v2/tiles.json?key=2QY0a1dK2gB027NRIOkD",
      });

      map.setTerrain({
        source: "terrain-source",
        exaggeration: 1.5,
      });

      // Ajoutez le layer pour les ombres
      map.addSource("hillshade-source", {
        type: "raster",
        url: "https://api.maptiler.com/tiles/hillshade/tiles.json?key=2QY0a1dK2gB027NRIOkD",
      });

      map.addLayer({
        id: "hillshade-layer",
        type: "raster",
        source: "hillshade-source",
        paint: {
          "raster-opacity": 0.5, // réduisez l'opacité pour des ombres plus légères
        },
      });

      map.on("click", function (e) {
        // Obtenir les entités sous le point cliqué
        const features = map.queryRenderedFeatures(e.point, {});

        // Parcourir les entités et vérifier si elles représentent des sommets
        for (const feature of features) {
          if (feature.properties && feature.properties["class"] === "peak") {
            new maplibregl.Popup()
              .setLngLat(e.lngLat)
              .setHTML(`<h1>${feature.properties["name"]}</h1>`)
              .addTo(map);
          }
        }
      });
    });
  });
</script>

<div id="map" />

<style>
  :global(body) {
    margin: 0;
  }

  #map {
    width: 100vw;
    height: 100vh;
  }
</style>
