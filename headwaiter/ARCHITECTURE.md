# HeadWaiter Challenge — Architecture & Guide de Déploiement
## Version MVP 1.0

---

## 🏗️ Architecture Technique

```
headwaiter/
├── index.html          ← Application complète (single-file PWA)
├── manifest.json       ← Manifeste PWA (installable Android/iOS)
└── ARCHITECTURE.md     ← Ce fichier
```

### Choix technologique : PWA Single-File
L'application est une **Progressive Web App** monofichier :
- **Aucune installation de serveur** requise
- Fonctionne hors ligne (questions intégrées dans le code)
- Installable sur Android via Chrome ("Ajouter à l'écran d'accueil")
- Installable sur iOS via Safari (même procédure)
- Compatible avec tout navigateur moderne

---

## 📐 Modules fonctionnels

### 1. Gestion des écrans (8 écrans)
| ID                  | Rôle                          |
|---------------------|-------------------------------|
| `home-screen`       | Accueil + navigation          |
| `lobby-screen`      | Création de partie (solo/online) |
| `join-screen`       | Rejoindre une partie en ligne |
| `game-screen`       | Plateau de jeu principal      |
| `scoreboard-screen` | Classement en cours de partie |
| `win-screen`        | Fin de partie + podium        |
| `admin-screen`      | Gestion des questions         |
| `rules-screen`      | Règles du jeu                 |

### 2. Moteur de jeu
```
rollDice()
  → advancePlayer(steps)
    → animateMove(...)   [animation case par case]
      → triggerCase(cas, player)
        → showQuestionModal(q, cat)  [QCM ou question ouverte]
        → handleEventCard(card)      [Chance / Communauté]
        → showEventCard(...)         [Pause bar, Anniversaire...]
          → nextTurn()
```

### 3. Plateau de jeu
- **Canvas HTML5** dessiné dynamiquement (responsive)
- **40 cases** disposées en circuit (comme Monopoly)
- **Pions CSS** animés avec transitions fluides
- Redessin automatique au changement de taille d'écran

### 4. Base de questions (105 questions)
| Catégorie    | Nb questions | Type              |
|--------------|-------------|-------------------|
| BOISSONS     | 15          | QCM + vrai/faux   |
| VINS         | 15          | Ouvertes + QCM    |
| FROMAGES     | 15          | Ouvertes          |
| COCKTAILS    | 15          | Trouvez l'erreur  |
| AUTRES       | 15          | HACCP + réglementation |
| CHANCE       | 10          | Cartes événements |
| COMMUNAUTE   | 10          | Cartes événements |

Questions extraites du fichier `questions_monopoly_CORRIGE.xlsx`.

### 5. Multijoueur
**Mode local (même appareil ou même réseau) :**
- `BroadcastChannel API` pour synchroniser plusieurs onglets/fenêtres
- `localStorage` pour persister les salles actives

**Mode en ligne (recommandé : Firebase) :**
```javascript
// Pour passer en Firebase, remplacer les fonctions :
// createOnlineRoom() → firebase.database().ref('rooms/'+code).set(room)
// joinRoom() → firebase.database().ref('rooms/'+code).on('value', ...)
// Synchronisation → firebase.database().ref('rooms/'+code+'/gameState').set(state)
```

---

## 🎮 Règles du jeu (implémentées)

1. **Lancer de dé** : 1d6, animation visuelle
2. **Déplacement** : animation case par case
3. **Cases questions** (VINS, BOISSONS, FROMAGES, COCKTAILS, AUTRES) :
   - Affichage de la question + propositions
   - Bouton "Révéler" puis "✓ Bonne" / "✗ Mauvaise"
   - +10 pts (bonne) / +2 pts (mauvaise)
4. **Cases événements** (CHANCE, COMMUNAUTÉ) :
   - Carte tirée aléatoirement
   - Effets : avancer/reculer, pause, rejouer, double points
5. **Case PAUSE BAR** : joueur bloqué pendant 1-2 tours
6. **Niveaux** :
   - 🔪 Commis (0-49 pts)
   - 🥂 Chef de Rang (50-149 pts)
   - 🎩 Maître d'Hôtel (150+ pts)
7. **Fin de partie** : après N tours définis à la création

---

## 📱 Déploiement

### Option 1 : Utilisation directe en classe (recommandé)
```
1. Ouvrir index.html dans Chrome sur PC/tablette
2. Projeter sur écran ou tableau blanc interactif
3. Chaque joueur voit le plateau, un seul répond
```

### Option 2 : Hébergement local WiFi
```bash
# Avec Python 3 (sur le PC du formateur)
cd headwaiter/
python3 -m http.server 8080

# Les apprenants connectent via :
# http://[IP-du-PC]:8080
# Exemple : http://192.168.1.42:8080
```

### Option 3 : GitHub Pages (gratuit)
```bash
git init && git add .
git commit -m "HeadWaiter Challenge v1.0"
# Créer repo sur github.com, push, activer Pages
# URL : https://[username].github.io/headwaiter/
```

### Option 4 : Netlify/Vercel (gratuit)
- Drag & drop du dossier sur netlify.com
- URL publique instantanée

### Installation sur smartphone
**Android (Chrome) :**
→ Menu ⋮ → "Ajouter à l'écran d'accueil"

**iOS (Safari) :**
→ Bouton Partage → "Sur l'écran d'accueil"

---

## 🔧 Personnalisation

### Ajouter des questions dans l'app
1. Menu "Gestion des Questions" (écran Admin)
2. Bouton "+ Question"
3. Choisir catégorie, saisir question + propositions + réponse
4. ✓ Enregistrer

### Ajouter des questions dans le code source
Dans `index.html`, trouver `const QUESTIONS_DB = {...}` et ajouter :
```javascript
QUESTIONS_DB['VINS'].push({
  id: 'v_custom_1',
  question: 'Ma nouvelle question ?',
  propositions: ['A – Option 1', 'B – Option 2', 'C – Option 3'],
  reponse: 'B – Option 2 (explication)'
});
```

---

## 🚀 Évolutions Firebase (multijoueur en ligne réel)

```javascript
// 1. Ajouter dans <head>
<script src="https://www.gstatic.com/firebasejs/10.0/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.0/firebase-database.js"></script>

// 2. Initialiser Firebase
const app = initializeApp({ apiKey: "...", databaseURL: "..." });
const db = getDatabase(app);

// 3. Remplacer createOnlineRoom()
async function createOnlineRoom() {
  const ref = ref(db, `rooms/${state.roomCode}`);
  await set(ref, { players: [...], status: 'waiting' });
  onValue(ref, (snap) => syncGameState(snap.val()));
}

// 4. Synchronisation d'état
function broadcastState() {
  set(ref(db, `rooms/${state.roomCode}/gameState`), {
    currentPlayerIndex: state.currentPlayerIndex,
    turn: state.turn,
    players: state.players
  });
}
```

---

## 📊 Métriques pédagogiques
L'app suit pour chaque joueur :
- **Score total** (points cumulés)
- **Nombre de bonnes réponses** / total répondu
- **Taux de réussite** (%)
- **Niveau atteint** (Commis / Chef de Rang / Maître d'Hôtel)
- **Position sur le plateau** (progression visuelle)

---

*HeadWaiter Challenge — Outil pédagogique pour la formation Titre Professionnel Serveur en Restauration*
*Développé avec ♥ pour la formation par apprentissage*
