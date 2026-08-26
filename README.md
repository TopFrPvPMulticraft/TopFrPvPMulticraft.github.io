# TopFrPvPMulticraft.github.io
// server.js
const express = require('express');
const cors = require('cors');

const app = express();
app.use(express.json());
app.use(cors());

// Ordre des tiers pour le tri du classement
const TIER_ORDER = {
  'HT1': 1, 'LT1': 2,
  'HT2': 3, 'LT2': 4,
  'HT3': 5, 'LT3': 6,
  'HT4': 7, 'LT4': 8,
  'HT5': 9, 'LT5': 10,
  'UNTESTED': 99
};

// Base de données temporaire en mémoire (remplaçable par SQLite/MongoDB)
let players = [
  { rank: 1, username: "PlayerOne", tier: "HT1", score: 2400, region: "EU" },
  { rank: 2, username: "PvPGod", tier: "LT1", score: 2250, region: "EU" },
  { rank: 3, username: "ComboKing", tier: "HT2", score: 2100, region: "NA" }
];

// Fonction pour trier et limiter au Top 100
function sortAndRankPlayers() {
  players.sort((a, b) => {
    const tierA = TIER_ORDER[a.tier] || 99;
    const tierB = TIER_ORDER[b.tier] || 99;
    if (tierA !== tierB) return tierA - tierB;
    return b.score - a.score; // Si même tier, départage au score
  });

  // Mise à jour du classement (1 à 100)
  players = players.slice(0, 100).map((player, index) => ({
    ...player,
    rank: index + 1
  }));
}

// ROUTE 1 : Récupérer le Top 100 pour le site Web
app.get('/api/leaderboard', (req, res) => {
  res.json(players);
});

// ROUTE 2 : Endpoint pour le programme tiers (Tierlist/Tester Bot)
// Reçoit JSON: { "username": "Joueur", "tier": "HT1", "score": 2500, "region": "EU", "apiKey": "SECRET123" }
app.post('/api/update-tier', (req, res) => {
  const { username, tier, score, region, apiKey } = req.body;

  // Sécurité minimale par clé API
  if (apiKey !== "SECRET_API_KEY_123") {
    return res.status(401).json({ error: "Clé API invalide" });
  }

  if (!username || !tier) {
    return res.status(400).json({ error: "Nom d'utilisateur et tier requis." });
  }

  // Chercher si le joueur existe déjà
  const existingPlayerIndex = players.findIndex(p => p.username.toLowerCase() === username.toLowerCase());

  if (existingPlayerIndex !== -1) {
    // Mise à jour
    players[existingPlayerIndex].tier = tier;
    if (score !== undefined) players[existingPlayerIndex].score = score;
    if (region !== undefined) players[existingPlayerIndex].region = region;
  } else {
    // Nouveau joueur
    players.push({
      username,
      tier,
      score: score || 0,
      region: region || "EU"
    });
  }

  sortAndRankPlayers();
  res.json({ message: "Joueur mis à jour avec succès", playersCount: players.length });
});

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Serveur démarré sur http://localhost:${PORT}`);
});
