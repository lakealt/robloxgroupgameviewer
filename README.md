
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
            padding: 40px 20px;
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
        }
        .controls {
            margin-bottom: 30px;
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
        }
        .gameCard {
            background: var(--card-bg);
            padding: 20px;
            border-radius: 16px;
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
            transition: transform 0.2s ease, box-shadow 0.2s ease;
            opacity: 0;
            transform: translateY(10px);
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
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
    </style>
</head>
<body data-theme="dark">
    <h1>Roblox Group Game Viewer</h1>
    <div class="controls">
        <input type="text" id="groupIdInput" placeholder="Enter Group ID">
        <button onclick="fetchGames()">Fetch Games</button>
        <button id="themeToggle" onclick="toggleTheme()">
            <i data-lucide="moon"></i>
            Toggle Theme
        </button>
    </div>
    <div id="gamesContainer">No games found for this group.</div>
    <script>
        lucide.createIcons();
        function toggleTheme() {
            const body = document.body;
            const current = body.getAttribute("data-theme");
            body.setAttribute("data-theme", current === "dark" ? "light" : "dark");
        }
        async function fetchGames() {
            const groupId = document.getElementById('groupIdInput').value;
            const container = document.getElementById('gamesContainer');
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

        // Optional: Auto-fetch on page load
        // window.onload = fetchGames;
    </script>
</body>
</html>
