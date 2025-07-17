<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <title>GPS Intelligent</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
  <link rel="stylesheet" href="https://unpkg.com/leaflet-routing-machine/dist/leaflet-routing-machine.css" />
  <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;700&display=swap" rel="stylesheet">
  <style>
    /* Variables CSS pour les couleurs et effets (Liquid Glass & Dark Mode) */
    :root {
      --glass-bg: rgba(30, 30, 30, 0.75); /* Plus sombre pour le dark mode */
      --glass-blur: 20px; /* Effet de flou plus prononcé pour "liquid glass" */
      --glass-border: 1px solid rgba(255, 255, 255, 0.1); /* Bordure subtile */
      --accent: #4285F4; /* Bleu Google Maps */
      --accent-hover: #357ae8;
      --text-color: #e0e0e0; /* Texte clair pour le dark mode */
      --text-color-light: #f0f0f0;
      --input-bg: rgba(255, 255, 255, 0.08); /* Fond des inputs */
      --shadow-color: rgba(0, 0, 0, 0.4);
    }

    body {
      margin: 0;
      background: #121212; /* Fond très sombre */
      color: var(--text-color);
      font-family: 'Roboto', sans-serif; /* Utilisation de Roboto */
      display: flex;
      flex-direction: column;
      height: 100vh;
      overflow: hidden; /* Empêche le défilement du corps */
    }

    #map {
      flex: 1;
      z-index: 1;
      border-radius: 12px; /* Coins arrondis pour la carte */
      overflow: hidden; /* Important pour que les coins arrondis fonctionnent */
    }

    /* Conteneur de recherche au design "liquid glass" */
    #search-container, #actions-container {
      position: absolute;
      left: 50%;
      transform: translateX(-50%);
      width: clamp(300px, 90%, 600px); /* Largeur flexible mais limitée */
      backdrop-filter: blur(var(--glass-blur)) saturate(180%); /* Effet "liquid glass" */
      -webkit-backdrop-filter: blur(var(--glass-blur)) saturate(180%); /* Pour Safari */
      background: var(--glass-bg);
      border: var(--glass-border);
      border-radius: 20px; /* Plus arrondis */
      padding: 16px 20px; /* Plus d'espace */
      display: flex;
      flex-direction: column;
      gap: 12px; /* Espacement plus grand entre les éléments */
      z-index: 999;
      box-shadow: 0 8px 30px var(--shadow-color); /* Ombre portée */
      transition: all 0.3s ease-in-out; /* Transitions douces */
    }

    #search-container {
      top: 20px; /* En haut de la page */
    }

    #actions-container {
      bottom: 20px; /* En bas de la page */
      flex-direction: row; /* Boutons côte à côte */
      justify-content: space-around;
      padding: 12px 20px;
    }

    /* Champ de saisie stylisé */
    #search-container input {
      background: var(--input-bg);
      border: var(--glass-border);
      border-radius: 12px;
      padding: 12px 15px;
      color: var(--text-color-light);
      width: calc(100% - 30px); /* Ajustement pour le padding */
      font-size: 1rem;
      transition: border-color 0.2s ease;
    }

    #search-container input:focus {
      outline: none;
      border-color: var(--accent); /* Bordure accentuée au focus */
    }

    /* Résultats de recherche */
    #results {
      max-height: 200px; /* Plus d'espace pour les résultats */
      overflow-y: auto;
      border-radius: 10px;
      background: rgba(0, 0, 0, 0.2); /* Légèrement plus sombre */
    }

    #results div {
      padding: 10px 15px;
      cursor: pointer;
      border-bottom: 1px solid rgba(255, 255, 255, 0.08);
      transition: background-color 0.2s ease;
    }

    #results div:last-child {
      border-bottom: none; /* Pas de bordure sur le dernier élément */
    }

    #results div:hover {
      background-color: rgba(255, 255, 255, 0.05);
    }

    /* Boutons stylisés */
    .action-button {
      background: var(--accent);
      color: white;
      border: none;
      border-radius: 12px;
      padding: 12px 20px;
      font-weight: 500;
      font-size: 1rem;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 8px;
      transition: background-color 0.2s ease, transform 0.1s ease;
    }

    .action-button:hover {
      background: var(--accent-hover);
      transform: translateY(-2px);
    }

    .action-button:active {
      transform: translateY(0);
    }

    .action-button i {
      font-size: 1.2rem;
    }

    /* Icônes (si tu veux ajouter des icônes de Font Awesome) */
    /* @import url("https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css"); */

    /* Styles spécifiques pour le panneau de route de Leaflet Routing Machine */
    .leaflet-routing-container {
      background: var(--glass-bg) !important;
      border: var(--glass-border) !important;
      border-radius: 12px !important;
      color: var(--text-color) !important;
      box-shadow: 0 4px 15px var(--shadow-color);
      max-height: 80%; /* Pour qu'il ne prenne pas toute la hauteur */
      overflow-y: auto; /* Scroll si contenu trop grand */
    }

    .leaflet-routing-alt {
      background: rgba(0, 0, 0, 0.1) !important;
      border-radius: 8px !important;
      margin-bottom: 8px !important;
    }

    .leaflet-routing-alt h3 {
      color: var(--accent) !important;
    }

    .leaflet-routing-collapse-btn {
      color: var(--text-color) !important;
      background: var(--accent) !important;
      border-radius: 50% !important;
      width: 30px !important;
      height: 30px !important;
      line-height: 30px !important;
    }

    /* Media Queries pour la responsivité */
    @media (max-width: 768px) {
      #search-container, #actions-container {
        width: 95%;
        padding: 12px 15px;
        border-radius: 15px;
      }

      #search-container {
        top: 10px;
      }

      #actions-container {
        bottom: 10px;
        flex-direction: column;
        gap: 8px;
        padding: 10px 15px;
      }

      .action-button {
        padding: 10px 15px;
      }
    }
  </style>
</head>
<body>
  <div id="map"></div>

  <div id="search-container">
    <input type="text" id="destination" placeholder="Rechercher une destination..." />
    <div id="results"></div>
  </div>

  <div id="actions-container">
    <button id="toggle-voice" class="action-button">🔊 Voix : ON</button>
    <button id="clear-route" class="action-button">✖ Effacer itinéraire</button>
    <button id="recenter-map" class="action-button">📍 Recentrer</button>
  </div>

  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <script src="https://unpkg.com/leaflet-routing-machine/dist/leaflet-routing-machine.min.js"></script>
  <script src="https://unpkg.com/leaflet-routing-machine/dist/leaflet-routing-machine.min.js"></script>
  <script src="https://unpkg.com/leaflet-routing-geoapify/dist/leaflet-routing-geoapify.min.js"></script>

  <script>
    const API_KEY = "bb8366872da84ba18b0c6a18576a6a2c"; // Remplace par ta clé API Geoapify
    let map = L.map('map').setView([48.8566, 2.3522], 13); // Vue initiale sur Paris
    let control, userMarker, circle;
    let voiceEnabled = true;
    let userLocation = null; // Pour stocker la dernière position connue de l'utilisateur

    // --- Configuration de la carte ---
    // Utilisation d'un thème de carte Geoapify plus sombre et moderne
    L.tileLayer(`https://maps.geoapify.com/v1/tile/dark-matter-brown/{z}/{x}/{y}.png?apiKey=${API_KEY}`, {
      attribution: '&copy; Geoapify, <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      maxZoom: 20
    }).addTo(map);

    // --- Suivi de la position de l'utilisateur ---
    map.locate({ setView: true, watch: true, enableHighAccuracy: true, maxZoom: 16 });

    map.on('locationfound', (e) => {
      userLocation = e.latlng; // Met à jour la position de l'utilisateur
      if (userMarker) {
        userMarker.setLatLng(e.latlng);
        circle.setLatLng(e.latlng);
      } else {
        // Personnalisation du marqueur utilisateur (style cercle bleu comme Google Maps)
        userMarker = L.circleMarker(e.latlng, {
          radius: 8,
          color: '#4285F4', // Bleu Google Maps
          fillColor: '#4285F4',
          fillOpacity: 1
        }).addTo(map);

        circle = L.circle(e.latlng, {
          radius: e.accuracy / 2, // Cercle d'incertitude
          color: '#4285F4',
          weight: 1,
          opacity: 0.5,
          fillColor: '#4285F4',
          fillOpacity: 0.1
        }).addTo(map);
      }
      // Recentrer la carte sur la position de l'utilisateur si aucun itinéraire n'est actif
      if (!control || !control.getWaypoints().some(wp => wp.latLng)) {
        map.setView(e.latlng, map.getZoom());
      }
    });

    map.on('locationerror', (e) => {
      console.error(e.message);
      alert("Impossible de récupérer votre position. Veuillez vérifier vos autorisations de localisation.");
    });

    // --- Autocomplétion de destination (Geoapify) ---
    const destinationInput = document.getElementById("destination");
    const resultsDiv = document.getElementById("results");
    let searchTimeout;

    destinationInput.addEventListener("input", () => {
      clearTimeout(searchTimeout); // Annule le précédent timeout
      const query = destinationInput.value;
      if (query.length < 2) {
        resultsDiv.innerHTML = "";
        return;
      }
      // Déclenche la recherche après un court délai (pour éviter trop de requêtes)
      searchTimeout = setTimeout(async () => {
        const url = `https://api.geoapify.com/v1/geocode/autocomplete?text=${encodeURIComponent(query)}&lang=fr&limit=7&apiKey=${API_KEY}`;
        try {
          const res = await fetch(url);
          const data = await res.json();
          resultsDiv.innerHTML = "";
          if (data.features && data.features.length > 0) {
            data.features.forEach(item => {
              const div = document.createElement("div");
              div.textContent = item.properties.formatted;
              div.addEventListener("click", () => {
                destinationInput.value = item.properties.formatted;
                resultsDiv.innerHTML = "";
                // Utilise les coordonnées directement de la réponse
                traceRoute(item.geometry.coordinates);
                destinationInput.blur(); // Cache le clavier mobile
              });
              resultsDiv.appendChild(div);
            });
          } else {
            resultsDiv.innerHTML = "<div>Aucun résultat trouvé.</div>";
          }
        } catch (error) {
          console.error("Erreur lors de la recherche d'autocomplétion:", error);
          resultsDiv.innerHTML = "<div>Erreur de chargement des résultats.</div>";
        }
      }, 300); // Délai de 300ms
    });

    // Cacher les résultats si l'utilisateur clique en dehors
    document.addEventListener("click", (e) => {
      if (!searchContainer.contains(e.target)) {
        resultsDiv.innerHTML = "";
      }
    });


    // --- Fonctionnalités des boutons d'action ---
    document.getElementById("toggle-voice").addEventListener("click", () => {
      voiceEnabled = !voiceEnabled;
      document.getElementById("toggle-voice").textContent = voiceEnabled ? "🔊 Voix : ON" : "🔇 Voix : OFF";
      speak(voiceEnabled ? "Assistance vocale activée." : "Assistance vocale désactivée.");
    });

    document.getElementById("clear-route").addEventListener("click", () => {
      if (control) {
        map.removeControl(control);
        control = null; // Réinitialise la variable de contrôle
        speak("Itinéraire effacé.");
        document.getElementById("search-container").style.display = "flex"; // Réaffiche la recherche
        destinationInput.value = ""; // Vide le champ de recherche
        if (userLocation) {
          map.setView(userLocation, 15); // Recentrer sur l'utilisateur
        }
      } else {
        speak("Aucun itinéraire à effacer.");
      }
    });

    document.getElementById("recenter-map").addEventListener("click", () => {
      if (userLocation) {
        map.setView(userLocation, 15); // Recentrer sur la position de l'utilisateur
        speak("Carte recentrée sur votre position.");
      } else {
        speak("Votre position n'est pas encore disponible.");
      }
    });

    // --- Fonction pour la synthèse vocale ---
    function speak(text) {
      if (!voiceEnabled || !('speechSynthesis' in window)) {
        console.log("Synthèse vocale non supportée ou désactivée.");
        return;
      }
      // Annule la parole précédente pour éviter les chevauchements
      speechSynthesis.cancel();
      const msg = new SpeechSynthesisUtterance(text);
      msg.lang = "fr-FR";
      msg.volume = 1; // Volume max
      msg.rate = 1; // Vitesse normale
      msg.pitch = 1; // Ton normal
      speechSynthesis.speak(msg);
    }

    // --- Traçage de l'itinéraire avec Leaflet Routing Machine ---
    function traceRoute(destCoordinates) {
      if (!userLocation) {
        alert("Impossible de calculer l'itinéraire : votre position n'est pas encore connue.");
        speak("Impossible de calculer l'itinéraire : votre position n'est pas encore connue.");
        return;
      }

      const startLatLng = userLocation;
      const endLatLng = L.latLng(destCoordinates[1], destCoordinates[0]); // Geoapify renvoie [lon, lat]

      if (control) {
        map.removeControl(control); // Supprime l'ancien itinéraire si présent
      }

      control = L.Routing.control({
        waypoints: [
          L.latLng(startLatLng.lat, startLatLng.lng),
          endLatLng
        ],
        // Configuration du routeur Geoapify
        router: L.Routing.geoapify(API_KEY, {
            type: 'pedestrian', // Ou 'car', 'bike' pour d'autres modes
            // Add other routing parameters if needed, e.g., 'mode: 'shortest''
        }),
        lineOptions: {
          styles: [{ color: '#4285F4', weight: 7, opacity: 0.8 }] // Couleur d'itinéraire comme Google Maps
        },
        createMarker: function(i, wp) {
          // Personnalise les marqueurs de départ/arrivée
          let markerIcon = L.divIcon({
            className: 'custom-div-icon',
            html: `<div style="background-color: ${i === 0 ? '#4CAF50' : '#F44336'}; width: 24px; height: 24px; border-radius: 50%; border: 3px solid white; box-shadow: 0 2px 5px rgba(0,0,0,0.3);"></div>`,
            iconSize: [24, 24],
            iconAnchor: [12, 24]
          });
          return L.marker(wp.latLng, { icon: markerIcon });
        },
        // Masque le panneau d'itinéraire par défaut pour un look plus épuré,
        // nous le montrerons programmatiquement après le calcul
        show: true, // Montre le panneau par défaut
        collapsible: true, // Permet de le réduire
        altLineOptions: {
          styles: [
              {color: 'black', opacity: 0.15, weight: 9},
              {color: 'white', opacity: 0.8, weight: 6}
          ]
      }
      }).addTo(map);

      // Événement quand l'itinéraire est trouvé
      control.on('routesfound', (e) => {
        const r = e.routes[0].summary;
        const time = Math.round(r.totalTime / 60);
        const distance = (r.totalDistance / 1000).toFixed(1); // En km

        speak(`Itinéraire lancé vers ${destinationInput.value}. Distance : ${distance} kilomètres. Temps estimé : ${time} minutes.`);
        map.fitBounds(control.getPlan().getWaypoints().map(wp => wp.latLng), { padding: [100, 100] });
        document.getElementById("search-container").style.display = "none"; // Cache la barre de recherche une fois l'itinéraire calculé

        // Affiche les instructions de l'itinéraire dans la console ou dans une future UI dédiée
        // console.log("Instructions de l'itinéraire:", e.routes[0].instructions);
      });

      // Gérer les erreurs de routage
      control.on('routingerror', (e) => {
          console.error("Erreur de routage:", e.error.message);
          alert("Impossible de trouver un itinéraire. Veuillez vérifier la destination ou votre connexion.");
          speak("Impossible de trouver un itinéraire.");
          document.getElementById("search-container").style.display = "flex"; // Réaffiche la recherche en cas d'erreur
      });
    }
  </script>
</body>
</html>
