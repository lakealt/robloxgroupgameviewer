# big update big,what is your favourite new feature
<html lang="en">
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Roblox Group Game Viewer</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            transition: background 1s ease, color 0.5s ease;
            background: #000;
            color: #fff;
            margin: 0;
            padding: 0;
        }
        h1 {
            margin-top: 20px;
            font-size: 24px;
        }
        .controls {
            margin: 20px auto;
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 10px;
        }
        input, button, select {
            padding: 10px;
            border-radius: 6px;
            border: none;
            font-size: 16px;
        }
        input[type="text"] {
            width: 240px;
        }
        button, select {
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
            width: 90%;
            max-width: 300px;
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
            width: 90%;
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
        #gradientPreview {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 8px;
            padding: 10px;
        }
        .gradient-option {
            width: 30px;
            height: 30px;
            border-radius: 50%;
            cursor: pointer;
            border: 2px solid white;
            transition: transform 0.2s ease;
        }
        .gradient-option:hover {
            transform: scale(1.2);
        }
        #adminPanel {
            display: none;
            position: fixed;
            bottom: 0;
            left: 0;
            background: rgba(0,0,0,0.9);
            color: white;
            padding: 10px;
            font-size: 14px;
            z-index: 9999;
        }
    </style>
</head>
<body ondblclick="toggleAdminPanel()">
    <h1>Roblox Group Game Viewer</h1>
    <div class="controls">
        <input type="text" id="groupIdInput" placeholder="Enter Group ID">
        <button class="btn-blue" onclick="fetchGames()">Fetch Games</button>
        <div id="pinGroupWrapper">
            <button onclick="pinCurrentGroup()">📌 Pin Group</button>
        </div>
        <button onclick="toggleTheme()">🌙 Toggle Theme</button>
        <select onchange="changeBackground(this.value)">
            <option value="default">🎨 Default</option>
            <option value="purplePink">💜 Purple → Pink</option>
            <option value="pinkOrange">🧡 Pink → Orange</option>
            <option value="blueCyan">💙 Blue → Cyan</option>
            <option value="violetRed">💖 Violet → Red</option>
            <option value="mintTeal">🌿 Mint → Teal</option>
            <option value="lavaFire">🔥 Lava → Fire</option>
            <option value="galaxy">🌌 Galaxy Fade</option>
            <option value="sunset">🌅 Sunset Blend</option>
            <option value="forest">🌲 Forest Calm</option>
            <option value="deepOcean">🌊 Deep Ocean</option>
            <option value="aurora">✨ Aurora Lights</option>
            <option value="neonCity">🌃 Neon City</option>
        </select>
    </div>
    <div id="gradientPreview"></div>
    <div class="controls">
        <input type="text" id="searchInput" placeholder="Search games..." oninput="searchGames()">
    </div>
    <div id="pinnedGroupsContainer">
        <h3>Pinned Groups</h3>
        <div id="pinnedGroupsList"></div>
    </div>
    <div id="gamesContainer">No games found for this group.</div>
    <div id="adminPanel">
        <p><strong>Admin Panel</strong></p>
        <button onclick="localStorage.clear();location.reload();">🧹 Clear Cache</button>
        <p>Theme: <span id="currentTheme"></span></p>
        <p>Gradient: <span id="currentGradient"></span></p>
    </div>
    <script>
        let currentGroupId = '';
        const gradients = {
            default: "linear-gradient(to right, #000000, #000000)",
            purplePink: "linear-gradient(to right, #6a11cb, #2575fc)",
            pinkOrange: "linear-gradient(to right, #ff6a00, #ee0979)",
            blueCyan: "linear-gradient(to right, #2193b0, #6dd5ed)",
            violetRed: "linear-gradient(to right, #8e2de2, #ff6a00)",
            mintTeal: "linear-gradient(to right, #43e97b, #38f9d7)",
            lavaFire: "linear-gradient(to right, #ff512f, #dd2476)",
            galaxy: "linear-gradient(to right, #20002c, #cbb4d4)",
            sunset: "linear-gradient(to right, #f12711, #f5af19)",
            forest: "linear-gradient(to right, #5a3f37, #2c7744)",
            deepOcean: "linear-gradient(to right, #2C3E50, #4CA1AF)",
            aurora: "linear-gradient(to right, #00c6ff, #0072ff)",
            neonCity: "linear-gradient(to right, #8e44ad, #3498db)"
        };
        function toggleAdminPanel() {
            const admin = document.getElementById('adminPanel');
            admin.style.display = admin.style.display === 'none' ? 'block' : 'none';
            document.getElementById('currentTheme').textContent = localStorage.getItem('theme');
            document.getElementById('currentGradient').textContent = localStorage.getItem('selectedGradient');
        }
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
                    <span>[${index + 1}] Group ID: ${id}</span>
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
            const isDark = body.dataset.theme !== 'light';
            body.dataset.theme = isDark ? 'light' : 'dark';
            body.style.color = isDark ? '#000' : '#fff';
            localStorage.setItem('theme', isDark ? 'light' : 'dark');
        }
        function changeBackground(value) {
            const bg = gradients[value] || gradients.default;
            document.body.style.background = bg;
            localStorage.setItem('selectedGradient', value);
        }
        function applySavedSettings() {
            const savedTheme = localStorage.getItem('theme');
            if (savedTheme === 'light') {
                document.body.dataset.theme = 'light';
                document.body.style.color = '#000';
            }
            const savedGradient = localStorage.getItem('selectedGradient') || 'default';
            changeBackground(savedGradient);
            const dropdown = document.querySelector("select");
            if (dropdown) dropdown.value = savedGradient;
        }
        function previewGradients() {
            const preview = document.getElementById('gradientPreview');
            Object.keys(gradients).forEach(key => {
                const swatch = document.createElement('div');
                swatch.className = 'gradient-option';
                swatch.style.background = gradients[key];
                swatch.title = key;
                swatch.onclick = () => {
                    changeBackground(key);
                    document.querySelector("select").value = key;
                };
                preview.appendChild(swatch);
            });
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
        window.onload = () => {
            applySavedSettings();
            renderPinnedGroups();
            previewGradients();
            fetchGames();
        };
    </script>
</body>
</html>
