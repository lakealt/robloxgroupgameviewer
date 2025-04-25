<!DOCTYPE html>
<html lang="en">
<head>
    <title>Roblox Group Game Viewer</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #000;
            color: #fff;
            text-align: center;
            margin: 0;
            padding: 0;
        }
        h1 {
            margin-top: 20px;
        }
        .controls {
            margin: 20px auto;
            display: flex;
            justify-content: center;
            gap: 10px;
            flex-wrap: wrap;
        }
        input, button {
            padding: 10px;
            border-radius: 6px;
            border: none;
            font-size: 16px;
        }
        input[type="text"] {
            width: 240px;
        }
        button {
            cursor: pointer;
        }
        #gamesContainer {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 20px;
            padding: 20px;
        }
        .gameCard {
            background: #111;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 0 10px rgba(255, 255, 255, 0.2);
            width: 300px;
            text-align: left;
        }
        .gameCard h2 {
            color: #7a5cff;
            margin-top: 0;
        }
        a {
            color: #4b9bff;
            text-decoration: none;
            font-weight: bold;
        }
        #pinnedGroupsContainer {
            margin: 30px auto;
            width: 80%;
            max-width: 700px;
        }
        #pinnedGroupsList {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        .pinnedItem {
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: #111;
            padding: 10px 16px;
            border-radius: 8px;
            font-family: monospace;
            color: #fff;
            box-shadow: 0 0 5px rgba(255,255,255,0.1);
        }
        .pinnedItem button {
            padding: 4px 10px;
            border-radius: 6px;
        }
        .btn-blue {
            background-color: #4b9bff;
            color: white;
        }
        .btn-red {
            background-color: #ff4b4b;
            color: white;
        }
        #pinGroupWrapper {
            display: none;
        }
    </style>
</head>
<body>
    <h1>Roblox Group Game Viewer</h1>
    <div class="controls">
        <input type="text" id="groupIdInput" placeholder="Enter Group ID">
        <button class="btn-blue" onclick="fetchGames()">Fetch Games</button>
        <div id="pinGroupWrapper">
            <button onclick="pinCurrentGroup()">📌 Pin Group</button>
        </div>
        <button onclick="toggleTheme()">🌙 Toggle Theme</button>
    </div>
    <div class="controls">
        <input type="text" id="searchInput" placeholder="Search games..." oninput="searchGames()">
    </div>
    <div id="pinnedGroupsContainer">
        <h3>Pinned Groups</h3>
        <div id="pinnedGroupsList"></div>
    </div>
    <div id="gamesContainer">No games found for this group.</div>
    <script>
        let currentGroupId = '';
        function getPinnedGroups() {
            return JSON.parse(localStorage.getItem('pinnedGroups') || '[]');
        }
        function setPinnedGroups(groups) {
            localStorage.setItem('pinnedGroups', JSON.stringify(groups));
        }
        function pinCurrentGroup() {
            const groupId = currentGroupId;
            if (!groupId) return;
            let groups = getPinnedGroups();
            if (!groups.includes(groupId)) {
                groups.push(groupId);
                setPinnedGroups(groups);
                renderPinnedGroups();
            }
        }
        function unpinGroup(groupId) {
            let groups = getPinnedGroups();
            groups = groups.filter(id => id !== groupId);
            setPinnedGroups(groups);
            renderPinnedGroups();
        }
        function loadPinnedGroup(groupId) {
            document.getElementById('groupIdInput').value = groupId;
            fetchGames();
        }
        function renderPinnedGroups() {
            const list = document.getElementById('pinnedGroupsList');
            list.innerHTML = '';
            const pinned = getPinnedGroups();
            if (pinned.length === 0) {
                list.innerHTML = '<span style="opacity: 0.6;">No pinned groups yet.</span>';
                return;
            }
            pinned.forEach((id, index) => {
                const div = document.createElement('div');
                div.className = 'pinnedItem';
                div.innerHTML = `
                    <span>[${index + 1}] &lt;!DOCTYPE html&gt;</span>
                    <div>
                        <button class="btn-blue" onclick="loadPinnedGroup('${id}')">View</button>
                        <button class="btn-red" onclick="unpinGroup('${id}')">🗑️</button>
                    </div>
                `;
                list.appendChild(div);
            });
        }
        function toggleTheme() {
            const body = document.body;
            const isDark = body.style.background === 'black' || body.style.background === '#000';
            body.style.background = isDark ? '#fff' : '#000';
            body.style.color = isDark ? '#000' : '#fff';
        }
        function searchGames() {
            const input = document.getElementById('searchInput').value.toLowerCase();
            const cards = document.querySelectorAll('.gameCard');
            cards.forEach(card => {
                const text = card.innerText.toLowerCase();
                card.style.display = text.includes(input) ? 'block' : 'none';
            });
        }
        async function fetchGames() {
            const groupId = document.getElementById('groupIdInput').value.trim();
            const container = document.getElementById('gamesContainer');
            const pinWrapper = document.getElementById('pinGroupWrapper');
            currentGroupId = groupId;
            if (groupId) {
                pinWrapper.style.display = 'inline-block';
            } else {
                pinWrapper.style.display = 'none';
                return;
            }
            container.innerHTML = 'Loading...';
            try {
                const response = await fetch(`https://mskswokcev.devrahsanko.workers.dev/?groupId=${groupId}`);
                if (!response.ok) throw new Error('Failed to fetch data');
                const data = await response.json();
                const games = data.games || [];
                if (games.length === 0) {
                    container.innerHTML = 'No games found for this group.';
                    return;
                }
                container.innerHTML = '';
                games.forEach(game => {
                    const { name, description, rootPlaceId } = game;
                    const card = document.createElement('div');
                    card.className = 'gameCard';
                    card.innerHTML = `
                        <h2>${name}</h2>
                        <p><strong>Description:</strong> ${description || 'No description available.'}</p>
                        <a href="https://www.roblox.com/games/${rootPlaceId}" target="_blank">Play Game</a>
                    `;
                    container.appendChild(card);
                });
            } catch (error) {
                console.error('Error fetching games:', error);
                container.innerHTML = 'Error loading games. Please try again later.';
            }
        }
        // Initialize pinned groups on load
        window.onload = () => {
            renderPinnedGroups();
            fetchGames(); // Auto-load last entered group if any
        };
    </script>
</body>
</html>
