<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Movie Poster Inventory Manager</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .poster-card {
            transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
        }
        .poster-card:hover {
            transform: scale(1.05);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
        .modal {
            display: none;
        }
        .modal.active {
            display: flex;
        }
        /* Custom scrollbar */
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #1f2937;
        }
        ::-webkit-scrollbar-thumb {
            background: #4b5563;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #6b7280;
        }
        .tab-btn {
            border-bottom: 2px solid transparent;
        }
        .active-tab {
             border-bottom-color: #f59e0b; /* Using a yellow color to match theme */
             color: white;
        }
    </style>
</head>
<body class="bg-gray-900 text-white">

    <div id="app" class="container mx-auto p-4 md:p-8">
        <header class="flex flex-wrap justify-between items-center mb-8">
            <h1 class="text-4xl font-bold text-yellow-400">Poster Inventory Manager</h1>
            <nav>
                <button id="show-inventory-btn" class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg shadow-md transition-transform transform hover:scale-105">View Inventory</button>
            </nav>
        </header>

        <main id="main-view">
            <!-- Add Poster Section -->
            <div class="bg-gray-800 p-6 rounded-xl shadow-lg mb-8">
                <h2 class="text-2xl font-semibold mb-4">Add a New Poster</h2>
                <form id="add-poster-form" class="space-y-4">
                    <div>
                        <label for="poster-title" class="block text-sm font-medium text-gray-300">Poster Title</label>
                        <input type="text" id="poster-title" required placeholder="e.g., Blade Runner 2049" class="mt-1 w-full bg-gray-700 border border-gray-600 rounded-lg p-3 text-white focus:outline-none focus:ring-2 focus:ring-yellow-400">
                    </div>
                     <div>
                        <label for="poster-release-date" class="block text-sm font-medium text-gray-300">Release Date (Optional)</label>
                        <input type="date" id="poster-release-date" class="mt-1 w-full bg-gray-700 border border-gray-600 rounded-lg p-3 text-white focus:outline-none focus:ring-2 focus:ring-yellow-400">
                    </div>
                    <div>
                        <label for="poster-image-url" class="block text-sm font-medium text-gray-300">Poster Image URL (Optional)</label>
                        <input type="url" id="poster-image-url" placeholder="https://..." class="mt-1 w-full bg-gray-700 border border-gray-600 rounded-lg p-3 text-white focus:outline-none focus:ring-2 focus:ring-yellow-400">
                    </div>
                    <button type="submit" id="add-poster-btn" class="w-full bg-yellow-400 hover:bg-yellow-500 text-gray-900 font-bold py-3 px-6 rounded-lg shadow-md transition-transform transform hover:scale-105">Add Poster to Inventory</button>
                </form>
            </div>

            <!-- Display Locations -->
            <div id="display-locations" class="mt-12 bg-gray-800 p-6 rounded-xl shadow-lg">
                <h2 class="text-3xl font-bold mb-6 border-b-2 border-gray-700 pb-2">Display Locations</h2>
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
                    <!-- Now Showing -->
                    <div>
                        <h3 class="text-xl font-semibold text-yellow-300 mb-4">Now Showing (6)</h3>
                        <div id="now-showing-dropdowns" class="space-y-3"></div>
                    </div>
                    <!-- Coming Soon -->
                    <div>
                        <h3 class="text-xl font-semibold text-yellow-300 mb-4">Coming Soon (3)</h3>
                        <div id="coming-soon-dropdowns" class="space-y-3"></div>
                    </div>
                    <!-- Hallway -->
                    <div>
                        <h3 class="text-xl font-semibold text-yellow-300 mb-4">Hallway (10)</h3>
                        <div id="hallway-dropdowns" class="space-y-3"></div>
                    </div>
                    <!-- Auditoriums -->
                    <div class="md:col-span-2 lg:col-span-3">
                        <h3 class="text-xl font-semibold text-yellow-300 mb-4">Auditoriums (9)</h3>
                        <div id="auditoriums-dropdowns" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-x-8 gap-y-3"></div>
                    </div>
                </div>
            </div>
        </main>

        <!-- Inventory Modal -->
        <div id="inventory-modal" class="modal fixed inset-0 bg-black bg-opacity-75 items-center justify-center p-4">
            <div class="bg-gray-800 rounded-xl shadow-2xl w-full max-w-6xl h-[90vh] flex flex-col">
                <div class="flex flex-wrap justify-between items-center p-6 border-b border-gray-700 gap-4">
                    <h2 class="text-3xl font-bold text-yellow-400">Poster Inventory</h2>
                    <div class="flex items-center space-x-4">
                        <button id="close-inventory-btn" class="text-gray-400 hover:text-white text-3xl">&times;</button>
                    </div>
                </div>
                <div class="border-b border-gray-700">
                    <nav class="flex space-x-1 px-4" id="inventory-tabs">
                        <button class="tab-btn active-tab py-4 px-6 font-semibold" data-tab="current">Current</button>
                        <button class="tab-btn py-4 px-6 font-semibold text-gray-400" data-tab="removed">Removed</button>
                        <button class="tab-btn py-4 px-6 font-semibold text-gray-400" data-tab="requested">Requested</button>
                    </nav>
                </div>

                <!-- Current Inventory Tab -->
                <div id="current-inventory-tab" class="tab-content active-content flex flex-col overflow-hidden">
                    <div class="flex justify-end p-4 border-b border-gray-700 flex-shrink-0">
                         <button id="sort-inventory-btn" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-lg shadow-md">Sort by Release Date (Newest)</button>
                    </div>
                    <div class="overflow-y-auto flex-grow">
                        <div id="inventory-list" class="p-6 grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-6">
                            <!-- Inventory items will be here -->
                        </div>
                        <div id="inventory-empty" class="p-6 text-center text-gray-400 hidden">
                            <p class="text-xl">Your inventory is empty.</p>
                            <p>Use the form on the main page to add posters.</p>
                        </div>
                    </div>
                </div>

                <!-- Removed Inventory Tab -->
                <div id="removed-inventory-tab" class="tab-content hidden flex flex-col overflow-hidden">
                     <div class="overflow-y-auto flex-grow">
                        <div id="removed-list" class="p-6 grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-6">
                            <!-- Removed items will be here -->
                        </div>
                        <div id="removed-empty" class="p-6 text-center text-gray-400 hidden">
                            <p class="text-xl">No posters have been removed.</p>
                        </div>
                    </div>
                </div>

                <!-- Requested Posters Tab -->
                <div id="requested-inventory-tab" class="tab-content hidden p-6 overflow-y-auto">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                        <div>
                            <h3 class="text-2xl font-bold text-yellow-300 mb-4">Request a Poster</h3>
                            <form id="request-poster-form" class="space-y-4 bg-gray-700 p-4 rounded-lg">
                                <div>
                                    <label for="requester-name" class="block text-sm font-medium text-gray-300">Your Name</label>
                                    <input type="text" id="requester-name" required placeholder="John Doe" class="mt-1 w-full bg-gray-600 border border-gray-500 rounded-lg p-2 text-white">
                                </div>
                                <div>
                                    <label for="requested-poster-select" class="block text-sm font-medium text-gray-300">Select Poster</label>
                                    <select id="requested-poster-select" required class="mt-1 w-full bg-gray-600 border border-gray-500 rounded-lg p-2 text-white"></select>
                                </div>
                                <button type="submit" class="w-full bg-green-600 hover:bg-green-700 text-white font-bold py-2 px-4 rounded-lg">Submit Request</button>
                            </form>
                        </div>
                         <div>
                            <h3 class="text-2xl font-bold text-yellow-300 mb-4">Current Requests</h3>
                            <div id="requests-list" class="space-y-4"></div>
                            <div id="requests-empty" class="text-center text-gray-400">No requests yet.</div>
                             <button id="draw-winners-btn" class="w-full mt-6 bg-purple-600 hover:bg-purple-700 text-white font-bold py-2 px-4 rounded-lg hidden">Draw Winners</button>
                        </div>
                    </div>
                     <div id="winners-display" class="mt-8 hidden">
                        <h3 class="text-2xl font-bold text-yellow-300 mb-4">Lottery Winners!</h3>
                        <div id="winners-list" class="space-y-2 bg-gray-700 p-4 rounded-lg"></div>
                    </div>
                </div>
            </div>
        </div>
        
        <!-- Toast Notification -->
        <div id="toast" class="fixed bottom-5 right-5 bg-green-500 text-white py-2 px-4 rounded-lg shadow-lg opacity-0 transition-opacity duration-300">
            <p id="toast-message"></p>
        </div>

    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // DOM Elements
            const addPosterForm = document.getElementById('add-poster-form');
            const posterTitleInput = document.getElementById('poster-title');
            const posterReleaseDateInput = document.getElementById('poster-release-date');
            const posterImageUrlInput = document.getElementById('poster-image-url');
            const showInventoryBtn = document.getElementById('show-inventory-btn');
            const closeInventoryBtn = document.getElementById('close-inventory-btn');
            const inventoryModal = document.getElementById('inventory-modal');
            const inventoryList = document.getElementById('inventory-list');
            const inventoryEmpty = document.getElementById('inventory-empty');
            const sortInventoryBtn = document.getElementById('sort-inventory-btn');
            const inventoryTabs = document.getElementById('inventory-tabs');
            const tabContents = document.querySelectorAll('.tab-content');
            const removedList = document.getElementById('removed-list');
            const removedEmpty = document.getElementById('removed-empty');

            // Requested Posters Elements
            const requestPosterForm = document.getElementById('request-poster-form');
            const requesterNameInput = document.getElementById('requester-name');
            const requestedPosterSelect = document.getElementById('requested-poster-select');
            const requestsList = document.getElementById('requests-list');
            const requestsEmpty = document.getElementById('requests-empty');
            const drawWinnersBtn = document.getElementById('draw-winners-btn');
            const winnersDisplay = document.getElementById('winners-display');
            const winnersList = document.getElementById('winners-list');

            const toast = document.getElementById('toast');
            const toastMessage = document.getElementById('toast-message');

            // App State
            let inventory = JSON.parse(localStorage.getItem('posterInventory')) || [];
            let removedInventory = JSON.parse(localStorage.getItem('removedPosterInventory')) || [];
            let posterRequests = JSON.parse(localStorage.getItem('posterRequests')) || {};
            let displayLocations = JSON.parse(localStorage.getItem('displayLocations')) || {};
            let inventorySortOrder = 'desc'; // 'desc' for newest first, 'asc' for oldest first

            const LOCATION_CONFIG = {
                'now-showing': { count: 6, container: 'now-showing-dropdowns' },
                'coming-soon': { count: 3, container: 'coming-soon-dropdowns' },
                'hallway': { count: 10, container: 'hallway-dropdowns' },
                'auditoriums': { count: 9, container: 'auditoriums-dropdowns' },
            };
            
            const PLACEHOLDER_IMG = 'https://placehold.co/50x75/1f2937/4b5563?text=N/A';

            // --- FUNCTIONS ---

            const saveState = () => {
                localStorage.setItem('posterInventory', JSON.stringify(inventory));
                localStorage.setItem('removedPosterInventory', JSON.stringify(removedInventory));
                localStorage.setItem('posterRequests', JSON.stringify(posterRequests));
                localStorage.setItem('displayLocations', JSON.stringify(displayLocations));
            };

            const showToast = (message, isError = false) => {
                toastMessage.textContent = message;
                toast.classList.remove('bg-green-500', 'bg-red-500');
                toast.classList.add(isError ? 'bg-red-500' : 'bg-green-500');
                toast.classList.remove('opacity-0');
                setTimeout(() => {
                    toast.classList.add('opacity-0');
                }, 3000);
            };

            const createPosterCard = (movie) => {
                const posterPath = movie.poster_path || 'https://placehold.co/500x750/1f2937/4b5563?text=No+Image';
                const card = document.createElement('div');
                card.className = 'poster-card bg-gray-800 rounded-lg overflow-hidden shadow-lg flex flex-col';
                card.innerHTML = `
                    <img src="${posterPath}" alt="${movie.title}" onerror="this.onerror=null;this.src='https://placehold.co/500x750/1f2937/4b5563?text=Bad+URL';" class="w-full h-auto object-cover">
                    <div class="p-4 flex flex-col flex-grow">
                        <h3 class="text-lg font-bold">${movie.title}</h3>
                        <p class="text-sm text-gray-400">${movie.release_date ? new Date(movie.release_date + 'T00:00:00').toLocaleDateString() : 'No Release Date'}</p>
                        <div class="mt-auto pt-4">
                            <button data-id="${movie.id}" class="remove-btn w-full bg-red-600 hover:bg-red-700 text-white font-bold py-2 px-4 rounded transition">Remove</button>
                        </div>
                    </div>
                `;

                card.querySelector('.remove-btn').addEventListener('click', () => removeFromInventory(movie.id));
                return card;
            };

            const createRemovedPosterCard = (movie) => {
                 const posterPath = movie.poster_path || 'https://placehold.co/500x750/1f2937/4b5563?text=No+Image';
                const card = document.createElement('div');
                card.className = 'bg-gray-800 rounded-lg overflow-hidden shadow-lg flex flex-col opacity-60';
                card.innerHTML = `
                    <img src="${posterPath}" alt="${movie.title}" onerror="this.onerror=null;this.src='https://placehold.co/500x750/1f2937/4b5563?text=Bad+URL';" class="w-full h-auto object-cover">
                    <div class="p-4 flex flex-col flex-grow">
                        <h3 class="text-lg font-bold">${movie.title}</h3>
                        <p class="text-sm text-gray-400">Removed: ${new Date(movie.removedDate).toLocaleDateString()}</p>
                    </div>
                `;
                return card;
            };

            const addPosterToInventory = (e) => {
                e.preventDefault();
                const title = posterTitleInput.value.trim();
                const releaseDate = posterReleaseDateInput.value;
                const imageUrl = posterImageUrlInput.value.trim();

                if (!title) {
                    showToast('Poster Title is required.', true);
                    return;
                }

                if (inventory.some(item => item.title.toLowerCase() === title.toLowerCase())) {
                    showToast('A poster with this title is already in your inventory.', true);
                    return;
                }

                const newPoster = {
                    id: Date.now(),
                    title: title,
                    release_date: releaseDate,
                    poster_path: imageUrl,
                };

                inventory.push(newPoster);
                showToast(`'${title}' added to inventory!`);
                updateInventoryAndDisplays();
                addPosterForm.reset();
            };

            const removeFromInventory = (movieId) => {
                const movieIndex = inventory.findIndex(m => m.id === movieId);
                if (movieIndex === -1) return;

                const movie = inventory[movieIndex];
                movie.removedDate = new Date().toISOString();
                
                removedInventory.unshift(movie);
                inventory.splice(movieIndex, 1);
                
                Object.keys(displayLocations).forEach(loc => {
                    Object.keys(displayLocations[loc]).forEach(idx => {
                        if (displayLocations[loc][idx] === String(movieId)) {
                            delete displayLocations[loc][idx];
                        }
                    });
                });

                showToast(`'${movie.title}' moved to Removed Posters.`);
                updateInventoryAndDisplays();
            };
            
            const deleteRequest = (posterId) => {
                const movie = inventory.find(m => m.id == posterId) || removedInventory.find(m => m.id == posterId);
                const title = movie ? movie.title : 'the selected poster';
                if (posterRequests[posterId]) {
                    delete posterRequests[posterId];
                    showToast(`Requests for '${title}' have been removed.`);
                    updateInventoryAndDisplays();
                }
            };

            const updateInventoryAndDisplays = () => {
                sortInventory();
                renderInventory();
                renderRemovedInventory();
                populateAllDropdowns();
                populateRequestDropdown();
                renderRequests();
                saveState();
            };
            
            const renderInventory = () => {
                inventoryList.innerHTML = '';
                 if (inventory.length === 0) {
                    inventoryEmpty.style.display = 'block';
                    inventoryList.style.display = 'none';
                } else {
                    inventoryEmpty.style.display = 'none';
                    inventoryList.style.display = 'grid';
                    inventory.forEach(movie => {
                        const card = createPosterCard(movie);
                        inventoryList.appendChild(card);
                    });
                }
            };

            const renderRemovedInventory = () => {
                removedList.innerHTML = '';
                if(removedInventory.length === 0) {
                    removedEmpty.style.display = 'block';
                    removedList.style.display = 'none';
                } else {
                    removedEmpty.style.display = 'none';
                    removedList.style.display = 'grid';
                    removedInventory.forEach(movie => {
                        const card = createRemovedPosterCard(movie);
                        removedList.appendChild(card);
                    });
                }
            };

            const sortInventory = () => {
                inventory.sort((a, b) => {
                    if (!a.release_date) return 1;
                    if (!b.release_date) return -1;
                    const dateA = new Date(a.release_date);
                    const dateB = new Date(b.release_date);
                    if (inventorySortOrder === 'desc') {
                        return dateB - dateA;
                    } else {
                        return dateA - dateB;
                    }
                });
                removedInventory.sort((a, b) => new Date(b.removedDate) - new Date(a.removedDate));
            };
            
            const populateDropdown = (selectElement) => {
                selectElement.innerHTML = '<option value="">-- Select a Poster --</option>';
                const sortedForDropdown = [...inventory].sort((a,b) => a.title.localeCompare(b.title));
                sortedForDropdown.forEach(movie => {
                    const option = document.createElement('option');
                    option.value = movie.id;
                    option.textContent = movie.title;
                    selectElement.appendChild(option);
                });
            };

            const createDropdownsForLocation = (key, { count, container }) => {
                const containerEl = document.getElementById(container);
                containerEl.innerHTML = '';
                if (!displayLocations[key]) displayLocations[key] = {};

                for (let i = 0; i < count; i++) {
                    const wrapper = document.createElement('div');
                    wrapper.className = 'flex items-center space-x-2';

                    const thumbnail = document.createElement('img');
                    thumbnail.className = 'w-12 h-auto rounded-md object-cover';
                    thumbnail.src = PLACEHOLDER_IMG;
                    
                    const select = document.createElement('select');
                    select.className = 'flex-grow bg-gray-700 border border-gray-600 rounded-lg p-2 text-white focus:outline-none focus:ring-2 focus:ring-yellow-400';
                    select.dataset.location = key;
                    select.dataset.index = i;
                    
                    populateDropdown(select);

                    // Set initial state for thumbnail and select
                    const selectedId = displayLocations[key][i];
                    if (selectedId) {
                        select.value = selectedId;
                        const movie = inventory.find(m => m.id == selectedId);
                        if (movie) {
                            thumbnail.src = movie.poster_path || PLACEHOLDER_IMG;
                            thumbnail.onerror = () => { thumbnail.src = PLACEHOLDER_IMG };
                        }
                    }

                    select.addEventListener('change', (e) => {
                        const location = e.target.dataset.location;
                        const index = e.target.dataset.index;
                        const newId = e.target.value;
                        const thumb = e.target.previousElementSibling;

                        if (!displayLocations[location]) displayLocations[location] = {};
                        
                        if (newId) {
                             displayLocations[location][index] = newId;
                             const movie = inventory.find(m => m.id == newId);
                             if (movie) {
                                 thumb.src = movie.poster_path || PLACEHOLDER_IMG;
                                 thumb.onerror = () => { thumb.src = PLACEHOLDER_IMG };
                             }
                        } else {
                            delete displayLocations[location][index];
                            thumb.src = PLACEHOLDER_IMG;
                        }
                       
                        saveState();
                    });
                    
                    wrapper.appendChild(thumbnail);
                    wrapper.appendChild(select);
                    containerEl.appendChild(wrapper);
                }
            };

            const populateAllDropdowns = () => {
                Object.keys(LOCATION_CONFIG).forEach(key => {
                    createDropdownsForLocation(key, LOCATION_CONFIG[key]);
                });
            };

            const populateRequestDropdown = () => {
                requestedPosterSelect.innerHTML = '<option value="">-- Select a Poster --</option>';
                 const sortedForDropdown = [...inventory].sort((a,b) => a.title.localeCompare(b.title));
                sortedForDropdown.forEach(movie => {
                    const option = document.createElement('option');
                    option.value = movie.id;
                    option.textContent = movie.title;
                    requestedPosterSelect.appendChild(option);
                });
            };

            const handlePosterRequest = (e) => {
                e.preventDefault();
                const name = requesterNameInput.value.trim();
                const posterId = requestedPosterSelect.value;
                
                if (!name || !posterId) {
                    showToast('Name and poster selection are required.', true);
                    return;
                }

                if (!posterRequests[posterId]) {
                    posterRequests[posterId] = [];
                }

                if (posterRequests[posterId].map(n => n.toLowerCase()).includes(name.toLowerCase())) {
                     showToast(`${name} has already requested this poster.`, true);
                     return;
                }

                posterRequests[posterId].push(name);
                showToast('Request submitted successfully!');
                renderRequests();
                saveState();
                requestPosterForm.reset();
            };
            
            const renderRequests = () => {
                requestsList.innerHTML = '';
                const requestIds = Object.keys(posterRequests);

                if (requestIds.length === 0 || requestIds.every(id => posterRequests[id].length === 0)) {
                    requestsEmpty.style.display = 'block';
                    drawWinnersBtn.style.display = 'none';
                    winnersDisplay.style.display = 'none';
                } else {
                    requestsEmpty.style.display = 'none';
                    drawWinnersBtn.style.display = 'block';
                    requestIds.forEach(posterId => {
                        const requesters = posterRequests[posterId];
                        if (requesters.length === 0) return;

                        const movie = inventory.find(m => m.id == posterId) || removedInventory.find(m => m.id == posterId);
                        if (!movie) return;

                        const requestGroup = document.createElement('div');
                        requestGroup.className = 'bg-gray-600 p-3 rounded-lg';
                        requestGroup.innerHTML = `
                            <div class="flex justify-between items-center">
                                <h4 class="font-bold text-lg text-yellow-200">${movie.title}</h4>
                                <button data-poster-id="${posterId}" class="delete-request-btn text-red-500 hover:text-red-400 text-2xl font-bold p-1 leading-none rounded-full w-8 h-8 flex items-center justify-center transition">&times;</button>
                            </div>
                            <p class="text-sm text-gray-300 mt-1">Requested by: ${requesters.join(', ')}</p>
                        `;
                        requestsList.appendChild(requestGroup);
                    });
                }
            };
            
            const drawWinners = () => {
                winnersList.innerHTML = '';
                winnersDisplay.style.display = 'block';
                const requestIds = Object.keys(posterRequests);

                if (requestIds.length === 0) {
                    winnersList.innerHTML = '<p>No requests to draw from.</p>';
                    return;
                }

                let drawn = false;
                requestIds.forEach(posterId => {
                     const requesters = posterRequests[posterId];
                     if (requesters && requesters.length > 0) {
                         drawn = true;
                         const winner = requesters[Math.floor(Math.random() * requesters.length)];
                         const movie = inventory.find(m => m.id == posterId) || removedInventory.find(m => m.id == posterId);
                         if (movie) {
                            const winnerEl = document.createElement('div');
                            winnerEl.innerHTML = `<p><span class="font-bold text-yellow-200">${movie.title}:</span> ${winner}</p>`;
                            winnersList.appendChild(winnerEl);
                         }
                     }
                });
                if (!drawn) {
                     winnersList.innerHTML = '<p>No requests to draw from.</p>';
                }
            };

            // --- EVENT LISTENERS ---

            addPosterForm.addEventListener('submit', addPosterToInventory);

            showInventoryBtn.addEventListener('click', () => {
                updateInventoryAndDisplays();
                inventoryModal.classList.add('active');
            });

            closeInventoryBtn.addEventListener('click', () => inventoryModal.classList.remove('active'));

            inventoryModal.addEventListener('click', (e) => {
                if (e.target === inventoryModal) {
                    inventoryModal.classList.remove('active');
                }
            });

            sortInventoryBtn.addEventListener('click', () => {
                inventorySortOrder = inventorySortOrder === 'desc' ? 'asc' : 'desc';
                sortInventoryBtn.textContent = `Sort by Release Date (${inventorySortOrder === 'desc' ? 'Newest' : 'Oldest'})`;
                sortInventory();
                renderInventory();
            });

            inventoryTabs.addEventListener('click', (e) => {
                if (e.target.classList.contains('tab-btn')) {
                    const tabId = e.target.dataset.tab;
                    
                    inventoryTabs.querySelectorAll('.tab-btn').forEach(btn => {
                        btn.classList.remove('active-tab', 'text-white');
                        btn.classList.add('text-gray-400');
                    });
                    e.target.classList.add('active-tab', 'text-white');
                    e.target.classList.remove('text-gray-400');

                    tabContents.forEach(content => {
                        content.classList.add('hidden');
                        content.classList.remove('active-content');
                    });
                    const activeContent = document.getElementById(`${tabId}-inventory-tab`);
                    activeContent.classList.remove('hidden');
                    activeContent.classList.add('active-content');
                }
            });

            requestPosterForm.addEventListener('submit', handlePosterRequest);
            drawWinnersBtn.addEventListener('click', drawWinners);
            
            requestsList.addEventListener('click', e => {
                const deleteBtn = e.target.closest('.delete-request-btn');
                if (deleteBtn) {
                    const posterId = deleteBtn.dataset.posterId;
                    deleteRequest(posterId);
                }
            });

            // Initial Load
            updateInventoryAndDisplays();
        });
    </script>
</body>
</html>
