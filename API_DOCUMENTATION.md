# Rock-Paper-Scissors Tournament - API Documentation

## Base URL
- Local: `http://localhost:5000`
- Colab (ngrok): `https://your-ngrok-url.ngrok-free.app`

---

## Endpoints

### 1. POST /api/player/register

**Description:** Creates a new player resource if they don't exist in the leaderboard.

**Request:**
```json
{
  "player_name": "string"
}
```

**Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Player 'Alice' registered successfully",
  "player": {
    "name": "Alice",
    "score": 0,
    "games_won": 0
  }
}
```

**Already Exists Response (200 OK):**
```json
{
  "success": true,
  "message": "Player 'Alice' already registered",
  "player": {
    "name": "Alice",
    "score": 15,
    "games_won": 2
  }
}
```

**Error Response (400 Bad Request):**
```json
{
  "success": false,
  "message": "Player name cannot be empty"
}
```

---

### 2. POST /api/game/start

**Description:** Initializes the state for a new 10-round game between two players. Automatically registers players if they don't exist in the leaderboard.

**Request:**
```json
{
  "player1": "string",
  "player2": "string"
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Game started between Alice and Bob",
  "game_state": {
    "player1": "Alice",
    "player2": "Bob",
    "player1_score": 0,
    "player2_score": 0,
    "rounds_played": 0,
    "rounds_total": 10
  }
}
```

**Error Responses:**

**Missing player names (400 Bad Request):**
```json
{
  "success": false,
  "message": "Both player names are required"
}
```

**Same player names (400 Bad Request):**
```json
{
  "success": false,
  "message": "Players must have different names"
}
```

---

### 3. POST /api/game/play_round

**Description:** Executes one round of Rock-Paper-Scissors, updates scores in the LEADERBOARD dictionary, and returns the round result. After 10 rounds, the game completes and updates cumulative statistics.

**Request:**
```json
{
  "player1_choice": "rock" | "paper" | "scissors",
  "player2_choice": "rock" | "paper" | "scissors"
}
```

**Mid-Game Response (200 OK):**
```json
{
  "success": true,
  "round_result": {
    "player1_choice": "rock",
    "player2_choice": "scissors",
    "round_winner": "player1",
    "round_number": 5
  },
  "game_complete": false,
  "current_score": {
    "player1": 3,
    "player2": 2
  },
  "rounds_remaining": 5
}
```

**Game Complete Response (200 OK):**
```json
{
  "success": true,
  "round_result": {
    "player1_choice": "paper",
    "player2_choice": "rock",
    "round_winner": "player1",
    "round_number": 10
  },
  "game_complete": true,
  "game_winner": "Alice",
  "final_score": {
    "player1": 7,
    "player2": 3
  },
  "last_winner": "Alice"
}
```

**Error Responses:**

**No active game (400 Bad Request):**
```json
{
  "success": false,
  "message": "No active game. Please start a new game first."
}
```

**Invalid choice (400 Bad Request):**
```json
{
  "success": false,
  "message": "Invalid choice. Must be 'rock', 'paper', or 'scissors'"
}
```

**Game Logic:**
- `round_winner` values: `"player1"`, `"player2"`, or `"tie"`
- Winner determined by standard Rock-Paper-Scissors rules:
  - Rock beats Scissors
  - Scissors beats Paper
  - Paper beats Rock

---

### 4. GET /api/leaderboard

**Description:** Retrieves the complete leaderboard with two sorted views. Converts the LEADERBOARD dictionary to a list and applies sorting algorithms.

**Request:** No body required (GET request)

**Success Response (200 OK):**
```json
{
  "success": true,
  "leaderboard": {
    "by_name": [
      {
        "name": "Alice",
        "score": 25,
        "games_won": 3
      },
      {
        "name": "Bob",
        "score": 18,
        "games_won": 1
      },
      {
        "name": "Charlie",
        "score": 30,
        "games_won": 4
      }
    ],
    "by_score": [
      {
        "name": "Charlie",
        "score": 30,
        "games_won": 4
      },
      {
        "name": "Alice",
        "score": 25,
        "games_won": 3
      },
      {
        "name": "Bob",
        "score": 18,
        "games_won": 1
      }
    ]
  },
  "total_players": 3
}
```

**Sorting Implementation:**
- `by_name`: Sorted alphabetically using `sorted(leaderboard_list, key=lambda x: x["name"])`
- `by_score`: Sorted by score descending using `sorted(leaderboard_list, key=lambda x: x["score"], reverse=True)`

---

### 5. GET /api/game/state

**Description:** Returns the current game state. Useful for checking if there's an active game or retrieving the last winner for winner retention.

**Request:** No body required (GET request)

**Success Response (200 OK):**
```json
{
  "success": true,
  "game_active": true,
  "player1": "Alice",
  "player2": "Bob",
  "player1_score": 5,
  "player2_score": 3,
  "rounds_played": 8,
  "last_winner": null
}
```

**After game completes:**
```json
{
  "success": true,
  "game_active": false,
  "player1": "Alice",
  "player2": "Bob",
  "player1_score": 7,
  "player2_score": 3,
  "rounds_played": 10,
  "last_winner": "Alice"
}
```

---

## Data Structures Used

### LEADERBOARD Dictionary
```python
LEADERBOARD = {
    "Alice": {
        "score": 25,        # Cumulative rounds won across all games
        "games_won": 3      # Number of complete games won
    },
    "Bob": {
        "score": 18,
        "games_won": 1
    }
}
```

**Time Complexity:** O(1) for player lookups and updates

### game_state Dictionary
```python
game_state = {
    "player1": "Alice",
    "player2": "Bob",
    "player1_score": 5,
    "player2_score": 3,
    "rounds_played": 8,
    "game_active": True,
    "last_winner": None  # Set when game completes
}
```

---

## Example Usage Flow

### Complete Game Sequence

1. **Start a game:**
```bash
POST /api/game/start
{
  "player1": "Alice",
  "player2": "Bob"
}
```

2. **Play 10 rounds:**
```bash
POST /api/game/play_round
{
  "player1_choice": "rock",
  "player2_choice": "scissors"
}
# Repeat 10 times
```

3. **Check leaderboard:**
```bash
GET /api/leaderboard
```

4. **Start next game with winner retention:**
```bash
GET /api/game/state
# Returns last_winner: "Alice"

POST /api/game/start
{
  "player1": "Alice",  # Winner from previous game
  "player2": "Charlie" # New opponent
}
```

---

## Testing with cURL

### Start a game:
```bash
curl -X POST http://localhost:5000/api/game/start \
  -H "Content-Type: application/json" \
  -d '{"player1": "Alice", "player2": "Bob"}'
```

### Play a round:
```bash
curl -X POST http://localhost:5000/api/game/play_round \
  -H "Content-Type: application/json" \
  -d '{"player1_choice": "rock", "player2_choice": "scissors"}'
```

### Get leaderboard:
```bash
curl http://localhost:5000/api/leaderboard
```

---

## Error Handling

All endpoints return consistent error responses:

**Format:**
```json
{
  "success": false,
  "message": "Error description"
}
```

**HTTP Status Codes:**
- `200 OK` - Successful request
- `201 Created` - Resource created (player registration)
- `400 Bad Request` - Invalid input or game state
- `500 Internal Server Error` - Server error (should not occur)
