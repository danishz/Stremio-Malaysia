# Stremio-Malaysia
Stremio Malaysia Film Drama
const { addonBuilder, serveHTTP } = require("stremio-addon-sdk");
const axios = require("axios");

const TMDB_API_KEY = "3436592461608dfb2ad72e9480e7ca43";
const TMDB_BASE_URL = "https://api.themoviedb.org/3";
const TMDB_IMAGE_BASE = "https://image.tmdb.org/t/p/w500";

// 1. Definisikan Manifest (Siri TV & Carian Diaktifkan)
const manifest = {
  id: "org.filemdramamalaysia.stremio",
  version: "1.1.0",
  name: "Filem & Drama Malaysia",
  description: "Katalog khas Filem & Drama/Siri TV Malaysia berserta fungsi carian.",
  resources: ["catalog", "meta", "stream"],
  types: ["movie", "series"], // Menambah jenis "series"
  catalogs: [
    {
      type: "movie",
      id: "my_movies",
      name: "Filem Malaysia",
      extra: [{ name: "search", isRequired: false }] // Mengaktifkan carian Filem
    },
    {
      type: "series",
      id: "my_series",
      name: "Drama / Siri TV Malaysia",
      extra: [{ name: "search", isRequired: false }] // Mengaktifkan carian Drama
    }
  ],
  idPrefixes: ["tmdb:"]
};

const builder = new addonBuilder(manifest);

// Helper Function: Format Meta Filem
function formatMovieMeta(movie) {
  return {
    id: `tmdb:${movie.id}`,
    type: "movie",
    name: movie.title,
    poster: movie.poster_path ? `${TMDB_IMAGE_BASE}${movie.poster_path}` : null,
    description: movie.overview || "Tiada sinopsis tersedia.",
    releaseInfo: movie.release_date ? movie.release_date.substring(0, 4) : ""
  };
}

// Helper Function: Format Meta Siri TV / Drama
function formatSeriesMeta(series) {
  return {
    id: `tmdb:${series.id}`,
    type: "series",
    name: series.name,
    poster: series.poster_path ? `${TMDB_IMAGE_BASE}${series.poster_path}` : null,
    description: series.overview || "Tiada sinopsis tersedia.",
    releaseInfo: series.first_air_date ? series.first_air_date.substring(0, 4) : ""
  };
}

// 2. Handler Katalog & Carian
builder.defineCatalogHandler(async (args) => {
  const isSearch = args.extra && args.extra.search;
  const searchQuery = isSearch ? args.extra.search : "";

  try {
    // --- KATEGORI FILEM ---
    if (args.type === "movie") {
      if (isSearch) {
        // Carian Filem
        const response = await axios.get(`${TMDB_BASE_URL}/search/movie`, {
          params: { api_key: TMDB_API_KEY, query: searchQuery, language: "ms-MY" }
        });
        const metas = response.data.results
          .filter((m) => m.origin_country && m.origin_country.includes("MY"))
          .map(formatMovieMeta);
        return { metas };
      } else {
        // Discover Filem Malaysia Popular
        const response = await axios.get(`${TMDB_BASE_URL}/discover/movie`, {
          params: {
            api_key: TMDB_API_KEY,
            with_origin_country: "MY",
            sort_by: "popularity.desc",
            language: "ms-MY"
          }
        });
        const metas = response.data.results.map(formatMovieMeta);
        return { metas };
      }
    }

    // --- KATEGORI SIRI TV / DRAMA ---
    if (args.type === "series") {
      if (isSearch) {
        // Carian Drama / Siri TV
        const response = await axios.get(`${TMDB_BASE_URL}/search/tv`, {
          params: { api_key: TMDB_API_KEY, query: searchQuery, language: "ms-MY" }
        });
        const metas = response.data.results
          .filter((s) => s.origin_country && s.origin_country.includes("MY"))
          .map(formatSeriesMeta);
        return { metas };
      } else {
        // Discover Drama Malaysia Popular
        const response = await axios.get(`${TMDB_BASE_URL}/discover/tv`, {
          params: {
            api_key: TMDB_API_KEY,
            with_origin_country: "MY",
            sort_by: "popularity.desc",
            language: "ms-MY"
          }
        });
        const metas = response.data.results.map(formatSeriesMeta);
        return { metas };
      }
    }

    return { metas: [] };
  } catch (error) {
    console.error("Ralat Catalog:", error.message);
    return { metas: [] };
  }
});

// 3. Handler Metadata Terperinci (Lengkap dengan Episod)
builder.defineMetaHandler(async (args) => {
  const tmdbId = args.id.replace("tmdb:", "");

  try {
    if (args.type === "movie") {
      const response = await axios.get(`${TMDB_BASE_URL}/movie/${tmdbId}`, {
        params: { api_key: TMDB_API_KEY, language: "ms-MY" }
      });
      const movie = response.data;
      return {
        meta: {
          id: `tmdb:${movie.id}`,
          type: "movie",
          name: movie.title,
          poster: movie.poster_path ? `${TMDB_IMAGE_BASE}${movie.poster_path}` : null,
          background: movie.backdrop_path ? `https://image.tmdb.org/t/p/original${movie.backdrop_path}` : null,
          description: movie.overview,
          releaseInfo: movie.release_date ? movie.release_date.substring(0, 4) : "",
          genres: movie.genres.map((g) => g.name)
        }
      };
    }

    if (args.type === "series") {
      const response = await axios.get(`${TMDB_BASE_URL}/tv/${tmdbId}`, {
        params: { api_key: TMDB_API_KEY, language: "ms-MY" }
      });
      const tv = response.data;

      // Bina senarai episod untuk Stremio
      const videos = [];
      if (tv.seasons) {
        for (const season of tv.seasons) {
          if (season.season_number === 0) continue; // Langkau episod khas (specials)
          for (let ep = 1; ep <= season.episode_count; ep++) {
            videos.push({
              id: `tmdb:${tv.id}:${season.season_number}:${ep}`,
              title: `Musim ${season.season_number} Episod ${ep}`,
              season: season.season_number,
              episode: ep
            });
          }
        }
      }

      return {
        meta: {
          id: `tmdb:${tv.id}`,
          type: "series",
          name: tv.name,
          poster: tv.poster_path ? `${TMDB_IMAGE_BASE}${tv.poster_path}` : null,
          background: tv.backdrop_path ? `https://image.tmdb.org/t/p/original${tv.backdrop_path}` : null,
          description: tv.overview,
          releaseInfo: tv.first_air_date ? tv.first_air_date.substring(0, 4) : "",
          genres: tv.genres.map((g) => g.name),
          videos: videos
        }
      };
    }

    return { meta: null };
  } catch (error) {
    console.error("Ralat Meta:", error.message);
    return { meta: null };
  }
});

// 4. Handler Penstriman (Stream Handler)
builder.defineStreamHandler(async (args) => {
  const idParts = args.id.split(":");
  const tmdbId = idParts[1];
  const season = idParts[2] || null;
  const episode = idParts[3] || null;

  try {
    if (args.type === "series") {
      // IDFormat untuk episod: tmdb:ID_TV:MUSIM:EPISOD (cth: tmdb:12345:1:2)
      console.log(`Permintaan Stream Drama ID: ${tmdbId}, Musim ${season}, Episod ${episode}`);

      return {
        streams: [
          {
            title: `Server 1 - Musim ${season} Episod ${episode} (1080p)`,
            url: "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4"
          }
        ]
      };
    }

    if (args.type === "movie") {
      return {
        streams: [
          {
            title: "Server 1 - Filem 1080p Direct",
            url: "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4"
          }
        ]
      };
    }

    return { streams: [] };
  } catch (error) {
    console.error("Ralat Stream:", error.message);
    return { streams: [] };
  }
});

// 5. Jalankan Pelayan
serveHTTP(builder.getInterface(), { port: 7000 });
console.log("Addon Filem & Drama Malaysia sedia di: http://127.0.0.1:7000/manifest.json");
