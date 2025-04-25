<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Roblox Group Game Viewer</title>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        :root {
            --bg-color: #0f0f0f;
            --text-color: #ffffff;
            --card-bg: #151515;
            --accent-color: #7a5cff;
        }
        [data-theme="light"] {
            --bg-color: #f4f4f4;
            --text-color: #111;
            --card-bg: #ffffff;
            --accent-color: #4b9bff;
        }
        body {
            font-family: 'Segoe UI', sans-serif;
            background: var(--bg-color);
            color: var(--text-color);
            transition: background 0.3s ease, color 0.3s ease;
            margin: 0;
            padding: 40px 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
        }
        .controls {
            margin-bottom: 20px;
            display: flex;
            flex-wrap: wrap;
            align-items: center;
            gap: 10px;
        }
        input[type="text"] {
            padding: 12px 16px;
            font-size: 1rem;
            border: none;
            border-radius: 10px;
            width: 250px;
            outline: none;
        }
        #searchFilter {
            width: 300px;
        }
        button {
            padding: 12px 20px;
            font-size: 1rem;
            background: linear-gradient(135deg, var(--accent-color), #4b9bff);
            border: none;
            border-radius: 10px;
            color: white;
            cursor: pointer;
            transition: transform 0.2s ease;
        }
        button:hover {
            transform: scale(1.05);
        }
        #themeToggle {
            background: transparent;
            border: 2px solid var(--accent-color);
            color: var(--accent-color);
            padding: 10px;
            border-radius: 10px;
            display: flex;
            align-items: center;
            gap: 6px;
        }
        #gamesContainer {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            width: 100%;
            max-width: 1200px;
            margin-top: 20px;
        }
        .gameCard {
            background: var(--card-bg);
            padding: 20px;
            border-radius: 16px;
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
            transition: transform 0.2s ease, box-shadow 0.2s ease;
            animation: fadeInUp 0.4s forwards;
        }
        .gameCard:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 30px rgba(122, 92, 255, 0.2);
        }
        .gameCard h2 {
            margin-top: 0;
            font-size: 1.4rem;
            color: var(--accent-color);
        }
        .gameCard p {
            font-size: 0.95rem;
        }
        .gameCard a {
            display: inline-block;
            margin-top: 10px;
            color: var(--accent-color);
            font-weight: bold;
            text-decoration: none;
        }
        .gameCard a:hover {
            text-decoration: underline;
        }
        @keyframes fadeInUp {
            from {
                opacity: 0;
                transform: translateY(10px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
        #loadingSpinner {
            border: 5px solid rgba(255, 255, 255, 0.1);
            border-top: 5px solid var(--accent-color);
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin: 30px auto;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        #noGames {
            font-size: 1.2rem;
            margin-top: 30px;
            opacity: 0.7;
        }
    </style>
</head>
<body>
    <h1>Roblox Group Game Viewer</h1>
    <div class="controls">
        <input type="text" id="groupIdInput" placeholder="Enter Group ID">
        <button onclick="fetchGames()">Fetch Games</button>
        <button id="themeToggle" onclick="toggleTheme()">
            <i data-lucide="moon"></i> Toggle Theme
        </button>
    </div>
    <div class="controls">
        <input type="text" id="searchFilter" onkeyup="filterGames()" placeholder="Search games...">
    </div>
    <div id="gamesContainer">
        <div id="noGames">No games found for this group.</div>
    </div>
    <div id="loadingSpinner" style="display:none;"></div>
    <script>
        lucide.createIcons();
        // Theme toggle with persistence
        const currentTheme = localStorage.getItem('theme') || 'dark';
        document.body.setAttribute('data-theme', currentTheme);
        function toggleTheme() {
            const body = document.body;
            const theme = body.getAttribute('data-theme') === 'dark' ? 'light' : 'dark';
            body.setAttribute('data-theme', theme);
            localStorage.setItem('theme', theme);
        }
        function filterGames() {
            const filter = document.getElementById('searchFilter').value.toLowerCase();
            const cards = document.querySelectorAll('.gameCard');
            cards.forEach(card => {
                const title = card.querySelector('h2').textContent.toLowerCase();
                card.style.display = title.includes(filter) ? 'block' : 'none';
            });
        }
        async function fetchGames() {
            const groupId = document.getElementById('groupIdInput').value;
            const container = document.getElementById('gamesContainer');
            const spinner = document.getElementById('loadingSpinner');
            const noGames = document.getElementById('noGames');
            container.innerHTML = '';
            spinner.style.display = 'block';
            try {
                const response = await fetch(`https://mskswokcev.devrahsanko.workers.dev/?groupId=${groupId}`);
                if (!response.ok) throw new Error('Failed to fetch data');
                const data = await response.json();
                const games = data.games || [];
                spinner.style.display = 'none';
                if (games.length === 0) {
                    container.innerHTML = '';
                    container.appendChild(noGames);
                    noGames.style.display = 'block';
                    return;
                }
                noGames.style.display = 'none';
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
                spinner.style.display = 'none';
                container.innerHTML = '<div id="noGames">Error loading games. Please try again later.</div>';
            }
        }
    </script>
</body>
</html>
