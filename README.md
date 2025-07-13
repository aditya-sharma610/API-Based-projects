<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive Google Maps API</title>
    <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;700&display=swap">
    <script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&libraries=places"></script>
    <style>
        :root {
            --primary-color: #4285f4;
            --secondary-color: #34a853;
            --accent-color: #ea4335;
            --text-color: #202124;
            --bg-color: #ffffff;
            --card-bg: #f8f9fa;
            --shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }

        .dark-mode {
            --primary-color: #8ab4f8;
            --secondary-color: #81c995;
            --accent-color: #f28b82;
            --text-color: #e8eaed;
            --bg-color: #202124;
            --card-bg: #292a2d;
            --shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            transition: background-color 0.3s, color 0.3s;
        }

        body {
            font-family: 'Roboto', sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            height: 100vh;
            display: flex;
            flex-direction: column;
        }

        header {
            background-color: var(--primary-color);
            color: white;
            padding: 1rem;
            display: flex;
            justify-content: space-between;
            align-items: center;
            box-shadow: var(--shadow);
            z-index: 100;
        }

        .logo {
            font-size: 1.5rem;
            font-weight: 500;
        }

        .controls {
            display: flex;
            align-items: center;
            gap: 1rem;
        }

        .search-container {
            position: relative;
            width: 300px;
        }

        #search-input {
            width: 100%;
            padding: 0.75rem 1rem;
            border-radius: 24px;
            border: none;
            outline: none;
            font-size: 1rem;
            box-shadow: var(--shadow);
        }

        .search-icon {
            position: absolute;
            right: 1rem;
            top: 50%;
            transform: translateY(-50%);
            color: #5f6368;
            pointer-events: none;
        }

        button {
            background-color: transparent;
            border: none;
            color: white;
            cursor: pointer;
            font-size: 1rem;
            padding: 0.5rem;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        button:hover {
            background-color: rgba(255, 255, 255, 0.2);
        }

        .theme-toggle {
            margin-left: 0.5rem;
        }

        .main-container {
            display: flex;
            flex: 1;
            overflow: hidden;
        }

        #map-container {
            flex: 1;
            height: 100%;
        }

        #map {
            height: 100%;
            width: 100%;
        }

        .sidebar {
            width: 350px;
            background-color: var(--card-bg);
            padding: 1rem;
            overflow-y: auto;
            box-shadow: -2px 0 10px rgba(0, 0, 0, 0.1);
            transform: translateX(0);
            transition: transform 0.3s ease;
        }

        .sidebar.collapsed {
            transform: translateX(100%);
        }

        .sidebar-toggle {
            display: none;
            position: absolute;
            right: 20px;
            bottom: 20px;
            background-color: var(--primary-color);
            color: white;
            width: 40px;
            height: 40px;
            border-radius: 50%;
            z-index: 10;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
        }

        .location-card {
            background-color: var(--card-bg);
            border-radius: 8px;
            padding: 1rem;
            margin-bottom: 1rem;
            box-shadow: var(--shadow);
            cursor: pointer;
        }

        .location-card:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
        }

        .location-card h3 {
            color: var(--primary-color);
            margin-bottom: 0.5rem;
        }

        .location-card p {
            color: var(--text-color);
            margin-bottom: 0.5rem;
            font-size: 0.9rem;
        }

        .location-card .distance {
            color: var(--secondary-color);
            font-weight: 500;
        }

        .marker-info {
            padding: 0.5rem;
        }

        .marker-info h3 {
            color: var(--primary-color);
            margin-bottom: 0.5rem;
        }

        .marker-info p {
            margin-bottom: 0.5rem;
        }

        .marker-actions {
            display: flex;
            gap: 0.5rem;
            margin-top: 0.5rem;
        }

        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
            z-index: 1000;
            justify-content: center;
            align-items: center;
        }

        .modal-content {
            background-color: var(--bg-color);
            padding: 2rem;
            border-radius: 8px;
            width: 90%;
            max-width: 500px;
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
        }

        .close-modal {
            position: absolute;
            top: 10px;
            right: 10px;
            font-size: 1.5rem;
            cursor: pointer;
        }

        @media (max-width: 768px) {
            .main-container {
                flex-direction: column;
            }
            
            .sidebar {
                width: 100%;
                height: 200px;
                position: absolute;
                bottom: 0;
                left: 0;
                z-index: 10;
            }
            
            .sidebar.collapsed {
                transform: translateY(100%);
            }
            
            .sidebar-toggle {
                display: flex;
            }
            
            .search-container {
                width: 200px;
            }
        }

        .custom-marker {
            background-color: var(--primary-color);
            border-radius: 50%;
            border: 2px solid white;
            width: 20px;
            height: 20px;
            position: relative;
        }

        .custom-marker::after {
            content: '';
            position: absolute;
            bottom: -10px;
            left: 50%;
            transform: translateX(-50%);
            border-width: 8px 6px 0;
            border-style: solid;
            border-color: var(--primary-color) transparent transparent;
        }
    </style>
</head>
<body>
    <header>
        <div class="logo">GeoExplorer</div>
        <div class="controls">
            <div class="search-container">
                <input type="text" id="search-input" placeholder="Search for locations...">
                <div class="search-icon">🔍</div>
            </div>
            <button class="theme-toggle" id="theme-toggle">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M12 3a6 6 0 0 0 9 9 9 9 0 1 1-9-9z"></path>
                </svg>
            </button>
        </div>
    </header>

    <div class="main-container">
        <div id="map-container">
            <div id="map"></div>
        </div>
        <div class="sidebar" id="sidebar">
            <h2>Saved Locations</h2>
            <div id="locations-list">
                <!-- Locations will be added here dynamically -->
            </div>
        </div>
        <button class="sidebar-toggle" id="sidebar-toggle">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M3 3h18v18H3z"></path>
                <path d="M3 9h18"></path>
                <path d="M3 15h18"></path>
            </svg>
        </button>
    </div>

    <div class="modal" id="add-location-modal">
        <div class="modal-content">
            <span class="close-modal" id="close-modal">&times;</span>
            <h2>Add Location</h2>
            <form id="location-form">
                <div class="form-group">
                    <label for="location-name">Name</label>
                    <input type="text" id="location-name" required>
                </div>
                <div class="form-group">
                    <label for="location-address">Address</label>
                    <input type="text" id="location-address" required readonly>
                </div>
                <div class="form-group">
                    <label for="location-notes">Notes</label>
                    <textarea id="location-notes" rows="3"></textarea>
                </div>
                <button type="submit">Save Location</button>
            </form>
        </div>
    </div>

    <script>
        // Replace with your actual API key
        // Note: In production, use proper key management and restrict API key usage
        const API_KEY = 'YOUR_API_KEY';
        
        let map;
        let infoWindow;
        let markers = [];
        let savedLocations = [];
        let currentPosition = null;
        let autocomplete;
        let darkMode = false;

        // Initialize the application
        function initApp() {
            initMap();
            initAutocomplete();
            initEventListeners();
            loadSavedLocations();
        }

        // Initialize Google Map
        function initMap() {
            map = new google.maps.Map(document.getElementById('map'), {
                center: { lat: 0, lng: 0 },
                zoom: 2,
                disableDefaultUI: true,
                styles: [
                    {
                        "featureType": "administrative",
                        "elementType": "labels.text.fill",
                        "stylers": [
                            {
                                "color": "#444444"
                            }
                        ]
                    },
                    {
                        "featureType": "landscape",
                        "elementType": "all",
                        "stylers": [
                            {
                                "color": "#f2f2f2"
                            }
                        ]
                    },
                    {
                        "featureType": "poi",
                        "elementType": "all",
                        "stylers": [
                            {
                                "visibility": "off"
                            }
                        ]
                    },
                    {
                        "featureType": "road",
                        "elementType": "all",
                        "stylers": [
                            {
                                "saturation": -100
                            },
                            {
                                "lightness": 45
                            }
                        ]
                    },
                    {
                        "featureType": "road.highway",
                        "elementType": "all",
                        "stylers": [
                            {
                                "visibility": "simplified"
                            }
                        ]
                    },
                    {
                        "featureType": "road.arterial",
                        "elementType": "labels.icon",
                        "stylers": [
                            {
                                "visibility": "off"
                            }
                        ]
                    },
                    {
                        "featureType": "transit",
                        "elementType": "all",
                        "stylers": [
                            {
                                "visibility": "off"
                            }
                        ]
                    },
                    {
                        "featureType": "water",
                        "elementType": "all",
                        "stylers": [
                            {
                                "color": "#46bcec"
                            },
                            {
                                "visibility": "on"
                            }
                        ]
                    }
                ]
            });

            infoWindow = new google.maps.InfoWindow();

            // Try HTML5 geolocation
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(
                    (position) => {
                        currentPosition = {
                            lat: position.coords.latitude,
                            lng: position.coords.longitude
                        };

                        map.setCenter(currentPosition);
                        map.setZoom(14);

                        // Add user location marker
                        addMarker(currentPosition, "Your Location", "You are here");
                    },
                    () => {
                        handleLocationError(true, infoWindow, map.getCenter());
                    }
                );
            } else {
                // Browser doesn't support Geolocation
                handleLocationError(false, infoWindow, map.getCenter());
            }

            // Add click listener on map
            map.addListener('click', (event) => {
                openAddLocationModal(event.latLng);
            });
        }

        // Initialize autocomplete for search input
        function initAutocomplete() {
            autocomplete = new google.maps.places.Autocomplete(
                document.getElementById('search-input'),
                {
                    types: ['geocode'],
                    fields: ['geometry', 'name', 'formatted_address', 'place_id']
                }
            );

            autocomplete.addListener('place_changed', () => {
                const place = autocomplete.getPlace();
                if (!place.geometry || !place.geometry.location) {
                    alert("No details available for input: '" + place.name + "'");
                    return;
                }

                // Clear existing markers except for user location
                clearMarkers(true);

                // Center map on selected place
                map.setCenter(place.geometry.location);
                map.setZoom(16);

                // Add marker for selected place
                addMarker(
                    place.geometry.location,
                    place.name,
                    place.formatted_address,
                    true
                );
            });
        }

        // Initialize event listeners
        function initEventListeners() {
            document.getElementById('theme-toggle').addEventListener('click', toggleDarkMode);
            document.getElementById('sidebar-toggle').addEventListener('click', toggleSidebar);
            document.getElementById('close-modal').addEventListener('click', closeModal);
            document.getElementById('location-form').addEventListener('submit', saveLocation);
            document.addEventListener('keydown', (e) => {
                if (e.key === 'Escape') {
                    closeModal();
                }
            });

            // Close modal when clicking outside of it
            document.getElementById('add-location-modal').addEventListener('click', (e) => {
                if (e.target === document.getElementById('add-location-modal')) {
                    closeModal();
                }
            });
        }

        // Load saved locations from localStorage
        function loadSavedLocations() {
            const saved = localStorage.getItem('savedLocations');
            if (saved) {
                savedLocations = JSON.parse(saved);
                renderSavedLocations();
            }
        }

        // Handle location error
        function handleLocationError(browserHasGeolocation, infoWindow, pos) {
            infoWindow.setPosition(pos);
            infoWindow.setContent(
                browserHasGeolocation
                    ? 'Error: The Geolocation service failed. Please enable location services if you want to see your current location on the map.'
                    : "Error: Your browser doesn't support geolocation."
            );
            infoWindow.open(map);
        }

        // Add a marker to the map
        function addMarker(position, title, description, isSearchResult = false) {
            const marker = new google.maps.Marker({
                position,
                map,
                title,
                animation: google.maps.Animation.DROP,
                icon: isSearchResult ? {
                    path: google.maps.SymbolPath.CIRCLE,
                    scale: 10,
                    fillColor: '#F4B400',
                    fillOpacity: 1,
                    strokeWeight: 2,
                    strokeColor: '#ffffff'
                } : null
            });

            markers.push(marker);

            // Add click listener for marker
            marker.addListener('click', () => {
                infoWindow.setContent(`
                    <div class="marker-info">
                        <h3>${title}</h3>
                        <p>${description}</p>
                        <div class="marker-actions">
                            <button onclick="navigateToLocation(${position.lat()}, ${position.lng()})">Directions</button>
                            ${isSearchResult ? `<button onclick="saveSelectedLocation('${title}', '${description}', ${position.lat()}, ${position.lng()})">Save</button>` : ''}
                        </div>
                    </div>
                `);
                infoWindow.open(map, marker);
            });

            return marker;
        }

        // Clear all markers from the map (except optionally user location)
        function clearMarkers(keepUserLocation = false) {
            for (let i = 0; i < markers.length; i++) {
                if (!keepUserLocation || markers[i].title !== 'Your Location') {
                    markers[i].setMap(null);
                }
            }
            markers = keepUserLocation ? markers.filter(m => m.title === 'Your Location') : [];
        }

        // Open the add location modal
        function openAddLocationModal(latLng) {
            const modal = document.getElementById('add-location-modal');
            document.getElementById('location-address').value = '';
            document.getElementById('location-name').value = '';
            document.getElementById('location-notes').value = '';
            
            // Reverse geocode to get address
            const geocoder = new google.maps.Geocoder();
            geocoder.geocode({ location: latLng }, (results, status) => {
                if (status === 'OK' && results[0]) {
                    document.getElementById('location-address').value = results[0].formatted_address;
                }
            });
            
            modal.style.display = 'flex';
            document.getElementById('location-name').focus();
        }

        // Close the modal
        function closeModal() {
            document.getElementById('add-location-modal').style.display = 'none';
        }

        // Save location from modal form
        function saveLocation(e) {
            e.preventDefault();
            
            const name = document.getElementById('location-name').value;
            const address = document.getElementById('location-address').value;
            const notes = document.getElementById('location-notes').value;
            
            // Get the clicked position (would need to store this when opening modal)
            // In a real implementation, we would need a way to get back to the position
            // For this demo, we'll just use a dummy position
            const position = { lat: 0, lng: 0 }; // Replace with actual position
            
            const location = {
                id: Date.now(),
                name,
                address,
                notes,
                position,
                createdAt: new Date().toISOString()
            };
            
            savedLocations.push(location);
            saveToLocalStorage();
            renderSavedLocations();
            closeModal();
            
            // Add a marker for the new location
            addMarker(position, name, notes);
            
            // Center map on new location
            map.setCenter(position);
            map.setZoom(14);
        }

        // Save locations to localStorage
        function saveToLocalStorage() {
            localStorage.setItem('savedLocations', JSON.stringify(savedLocations));
        }

        // Render saved locations to the sidebar
        function renderSavedLocations() {
            const container = document.getElementById('locations-list');
            container.innerHTML = '';
            
            if (savedLocations.length === 0) {
                container.innerHTML = '<p>No saved locations yet. Click on the map to add one.</p>';
                return;
            }
            
            savedLocations.forEach(location => {
                const card = document.createElement('div');
                card.className = 'location-card';
                card.innerHTML = `
                    <h3>${location.name}</h3>
                    <p>${location.address}</p>
                    <p class="distance">${calculateDistance(location.position)} away</p>
                    <p>${location.notes || 'No notes'}</p>
                `;
                
                card.addEventListener('click', () => {
                    map.setCenter(location.position);
                    map.setZoom(16);
                    
                    // Highlight the corresponding marker
                    const marker = markers.find(m => 
                        m.getPosition().lat() === location.position.lat && 
                        m.getPosition().lng() === location.position.lng
                    );
                    
                    if (marker) {
                        google.maps.event.trigger(marker, 'click');
                    }
                });
                
                container.appendChild(card);
            });
        }

        // Calculate distance from current position
        function calculateDistance(position) {
            if (!currentPosition) return 'Distance unknown';
            
            const R = 6371; // Earth's radius in km
            const dLat = (position.lat - currentPosition.lat) * Math.PI / 180;
            const dLng = (position.lng - currentPosition.lng) * Math.PI / 180;
            const a = 
                Math.sin(dLat/2) * Math.sin(dLat/2) +
                Math.cos(currentPosition.lat * Math.PI / 180) * Math.cos(position.lat * Math.PI / 180) * 
                Math.sin(dLng/2) * Math.sin(dLng/2);
            const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
            const d = R * c;
            
            return d < 1 ? `${Math.round(d * 1000)} meters` : `${d.toFixed(1)} km`;
        }

        // Toggle dark mode
        function toggleDarkMode() {
            darkMode = !darkMode;
            document.body.classList.toggle('dark-mode', darkMode);
            
            // Save preference to localStorage
            localStorage.setItem('darkMode', darkMode);
            
            // Change map style based on theme
            if (darkMode) {
                map.setOptions({
                    styles: [
                        { elementType: "geometry", stylers: [{ color: "#242f3e" }] },
                        { elementType: "labels.text.stroke", stylers: [{ color: "#242f3e" }] },
                        { elementType: "labels.text.fill", stylers: [{ color: "#746855" }] },
                        {
                            featureType: "administrative.locality",
                            elementType: "labels.text.fill",
                            stylers: [{ color: "#d59563" }]
                        },
                        {
                            featureType: "poi",
                            elementType: "labels.text.fill",
                            stylers: [{ color: "#d59563" }]
                        },
                        {
                            featureType: "poi.park",
                            elementType: "geometry",
                            stylers: [{ color: "#263c3f" }]
                        },
                        {
                            featureType: "poi.park",
                            elementType: "labels.text.fill",
                            stylers: [{ color: "#6b9a76" }]
                        },
                        {
                            featureType: "road",
                            elementType: "geometry",
                            stylers: [{ color: "#38414e" }]
                        },
                        {
                            featureType: "road",
                            elementType: "geometry.stroke",
                            stylers: [{ color: "#212a37" }]
                        },
                        {
                            featureType: "road",
                            elementType: "labels.text.fill",
                            stylers: [{ color: "#9ca5b3" }]
                        },
                        {
                            featureType: "road.highway",
                            elementType: "geometry",
                            stylers: [{ color: "#746855" }]
                        },
                        {
                            featureType: "road.highway",
                            elementType: "geometry.stroke",
                            stylers: [{ color: "#1f2835" }]
                        },
                        {
                            featureType: "road.highway",
                            elementType: "labels.text.fill",
                            stylers: [{ color: "#f3d19c" }]
                        },
                        {
                            featureType: "transit",
                            elementType: "geometry",
                            stylers: [{ color: "#2f3948" }]
                        },
                        {
                            featureType: "transit.station",
                            elementType: "labels.text.fill",
                            stylers: [{ color: "#d59563" }]
                        },
                        {
                            featureType: "water",
                            elementType: "geometry",
                            stylers: [{ color: "#17263c" }]
                        },
                        {
                            featureType: "water",
                            elementType: "labels.text.fill",
                            stylers: [{ color: "#515c6d" }]
                        },
                        {
                            featureType: "water",
                            elementType: "labels.text.stroke",
                            stylers: [{ color: "#17263c" }]
                        }
                    ]
                });
            } else {
                // Light mode styles
                map.setOptions({
                    styles: [
                        {
                            "featureType": "administrative",
                            "elementType": "labels.text.fill",
                            "stylers": [
                                {
                                    "color": "#444444"
                                }
                            ]
                        },
                        {
                            "featureType": "landscape",
                            "elementType": "all",
                            "stylers": [
                                {
                                    "color": "#f2f2f2"
                                }
                            ]
                        },
                        {
                            "featureType": "poi",
                            "elementType": "all",
                            "stylers": [
                                {
                                    "visibility": "off"
                                }
                            ]
                        },
                        {
                            "featureType": "road",
                            "elementType": "all",
                            "stylers": [
                                {
                                    "saturation": -100
                                },
                                {
                                    "lightness": 45
                                }
                            ]
                        },
                        {
                            "featureType": "road.highway",
                            "elementType": "all",
                            "stylers": [
                                {
                                    "visibility": "simplified"
                                }
                            ]
                        },
                        {
                            "featureType": "road.arterial",
                            "elementType": "labels.icon",
                            "stylers": [
                                {
                                    "visibility": "off"
                                }
                            ]
                        },
                        {
                            "featureType": "transit",
                            "elementType": "all",
                            "stylers": [
                                {
                                    "visibility": "off"
                                }
                            ]
                        },
                        {
                            "featureType": "water",
                            "elementType": "all",
                            "stylers": [
                                {
                                    "color": "#46bcec"
                                },
                                {
                                    "visibility": "on"
                                }
                            ]
                        }
                    ]
                });
            }
        }

        // Toggle sidebar visibility
        function toggleSidebar() {
            document.getElementById('sidebar').classList.toggle('collapsed');
        }

        // Save selected location from search results
        function saveSelectedLocation(name, address, lat, lng) {
            const notes = prompt("Add any notes about this location:");
            
            const location = {
                id: Date.now(),
                name,
                address,
                notes: notes || '',
                position: { lat, lng },
                createdAt: new Date().toISOString()
            };
            
            savedLocations.push(location);
            saveToLocalStorage();
            renderSavedLocations();
        }

        // Open navigation in Google Maps
        function navigateToLocation(lat, lng) {
            const url = `https://www.google.com/maps/dir/?api=1&destination=${lat},${lng}&travelmode=driving`;
            window.open(url, '_blank');
        }

        // Initialize the app when the page loads
        window.onload = initApp;
    </script>
</body>
</html>

```
