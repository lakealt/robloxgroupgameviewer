
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Roblox Group Game Viewer</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(to right, #0f0f0f, #1e1e1e);
            color: #f1f1f1;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 40px 20px;
            margin: 0;
        }
        h1 {
            font-size: 2.5rem;
            margin-bottom: 20px;
            color: #7a5cff;
        }
        input[type="text"] {
            padding: 12px 16px;
            font-size: 1rem;
            border: none;
            border-radius: 10px;
            width: 250px;
            max-width: 90%;
            margin-right: 10px;
            outline: none;
        }
        button {
            padding: 12px 20px;
            font-size: 1rem;
            background: linear-gradient(135deg, #4b9bff, #7a5cff);
            border: none;
            border-radius: 10px;
            color: white;
            cursor: pointer;
            transition: 0.3s;
        }
        button:hover {
            opacity: 0.85;
        }
        #gamesContainer {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            width: 100%;
            max-width: 1200px;
            margin-top: 30px;
        }

        .gameCard {
            background: #151515;
            padding: 20px;
            border-radius: 16px;
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.5);
            transition: transform 0.2s, box-shadow 0.2s;
        }

        .gameCard:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 30px rgba(122, 92, 255, 0.2);
        }

        .gameCard h2 {
            margin-top: 0;
            font-size: 1.5rem;
            color: #4b9bff;
        }

        .gameCard p {
            font-size: 0.95rem;
            margin: 10px 0;
        }

        .gameCard a {
            display: inline-block;
            margin-top: 10px;
            color: #7a5cff;
            font-weight: bold;
            text-decoration: none;
        }

        .gameCard a:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>
    <h1>Roblox Group Game Viewer</h1>
    <div>
        <input type="text" id="groupIdInput" placeholder="Enter Group ID">
        <button onclick="fetchGames()">Fetch Games</button>
    </div>
    <div id="gamesContainer">No games found for this group.</div>
    <script>
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
        // Optional: auto-fetch on load with default group
        // window.onload = fetchGames;
    </script>
</body>
</html>
