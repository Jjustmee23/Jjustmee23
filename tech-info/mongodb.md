# 🍃 MongoDB - Document-Oriented NoSQL Database

![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)

## 📋 What is MongoDB?

**MongoDB** is a source-available cross-platform document-oriented database program. Classified as a NoSQL database program, MongoDB uses JSON-like documents with optional schemas. MongoDB is developed by MongoDB Inc. and licensed under the Server Side Public License.

## 🔧 Key Features

- **Document-Oriented** - Stores data in flexible, JSON-like documents
- **Schema Flexibility** - Dynamic schemas for rapid development
- **Horizontal Scaling** - Built-in sharding for massive scale
- **Rich Query Language** - Powerful query and aggregation framework
- **Indexing** - Secondary indexes for fast queries
- **Replication** - High availability with replica sets

## 💡 Why I Use MongoDB

### 🎮 Gaming Platform Data
- **Player Profiles** - Complex nested player data structures
- **Game Statistics** - Flexible schema for different game types
- **Tournament Brackets** - Dynamic tournament structures
- **Chat Messages** - Real-time messaging with metadata

### 📊 Analytics & Logging
- **Application Logs** - Structured logging with flexible schema
- **Event Tracking** - User behavior and system events
- **IoT Metadata** - Sensor configuration and calibration data
- **Caching Layer** - Fast data retrieval for dashboards

## 🌟 My MongoDB Projects

### 1. 🎮 Gaming Platform Database
```javascript
// Player profile schema
{
  _id: ObjectId("..."),
  username: "player123",
  email: "player@example.com",
  profile: {
    displayName: "Pro Player",
    avatar: "https://...",
    bio: "Competitive gamer since 2015",
    country: "Belgium",
    timezone: "Europe/Brussels"
  },
  gaming: {
    level: 45,
    experience: 125750,
    rank: "Diamond",
    favoriteGames: ["CS:GO", "Valorant", "Rocket League"],
    achievements: [
      {
        id: "first_win",
        name: "First Victory",
        description: "Win your first match",
        unlockedAt: ISODate("2023-01-15T10:30:00Z"),
        rarity: "common"
      }
    ],
    statistics: {
      totalMatches: 245,
      wins: 156,
      losses: 89,
      winRate: 0.636,
      averageScore: 1250,
      bestScore: 2100,
      playtime: 15600, // minutes
      lastActive: ISODate("2024-01-08T14:22:00Z")
    }
  },
  preferences: {
    theme: "dark",
    language: "en",
    notifications: {
      email: true,
      push: false,
      tournaments: true,
      friendRequests: true
    },
    privacy: {
      profileVisibility: "public",
      showOnlineStatus: true,
      allowFriendRequests: true
    }
  },
  friends: [
    ObjectId("..."),
    ObjectId("...")
  ],
  createdAt: ISODate("2023-01-15T09:00:00Z"),
  updatedAt: ISODate("2024-01-08T14:22:00Z")
}

// Tournament schema
{
  _id: ObjectId("..."),
  name: "Winter Championship 2024",
  game: {
    id: "valorant",
    name: "Valorant",
    mode: "5v5_competitive"
  },
  format: "single_elimination",
  status: "in_progress",
  settings: {
    maxPlayers: 64,
    entryFee: 25.00,
    prizePool: 1500.00,
    prizeDistribution: [
      { position: 1, percentage: 50 },
      { position: 2, percentage: 30 },
      { position: 3, percentage: 20 }
    ]
  },
  schedule: {
    registrationStart: ISODate("2024-01-01T00:00:00Z"),
    registrationEnd: ISODate("2024-01-15T23:59:59Z"),
    tournamentStart: ISODate("2024-01-20T10:00:00Z"),
    estimatedEnd: ISODate("2024-01-21T18:00:00Z")
  },
  participants: [
    {
      playerId: ObjectId("..."),
      username: "player123",
      registeredAt: ISODate("2024-01-05T14:30:00Z"),
      seed: 1,
      currentRound: 3,
      isEliminated: false
    }
  ],
  bracket: {
    rounds: [
      {
        roundNumber: 1,
        matches: [
          {
            matchId: "match_001",
            players: [ObjectId("..."), ObjectId("...")],
            winner: ObjectId("..."),
            score: { "player1": 2, "player2": 0 },
            startedAt: ISODate("2024-01-20T10:00:00Z"),
            completedAt: ISODate("2024-01-20T10:45:00Z"),
            status: "completed"
          }
        ]
      }
    ]
  },
  createdBy: ObjectId("..."),
  createdAt: ISODate("2024-01-01T00:00:00Z"),
  updatedAt: ISODate("2024-01-20T15:30:00Z")
}
```

### 2. 📊 Analytics Aggregation Pipeline
```javascript
// Complex aggregation for tournament statistics
db.tournaments.aggregate([
  // Match tournaments from last 6 months
  {
    $match: {
      createdAt: {
        $gte: new Date(new Date() - 6 * 30 * 24 * 60 * 60 * 1000)
      },
      status: "completed"
    }
  },
  
  // Unwind participants to work with individual players
  { $unwind: "$participants" },
  
  // Group by player to calculate statistics
  {
    $group: {
      _id: "$participants.playerId",
      totalTournaments: { $sum: 1 },
      totalPrizeMoney: {
        $sum: {
          $cond: [
            { $eq: ["$participants.finalPosition", 1] },
            { $multiply: ["$prizePool", 0.5] },
            {
              $cond: [
                { $eq: ["$participants.finalPosition", 2] },
                { $multiply: ["$prizePool", 0.3] },
                {
                  $cond: [
                    { $eq: ["$participants.finalPosition", 3] },
                    { $multiply: ["$prizePool", 0.2] },
                    0
                  ]
                }
              ]
            }
          ]
        }
      },
      wins: {
        $sum: {
          $cond: [{ $eq: ["$participants.finalPosition", 1] }, 1, 0]
        }
      },
      topThreeFinishes: {
        $sum: {
          $cond: [{ $lte: ["$participants.finalPosition", 3] }, 1, 0]
        }
      },
      averagePosition: { $avg: "$participants.finalPosition" },
      games: { $addToSet: "$game.id" }
    }
  },
  
  // Lookup player information
  {
    $lookup: {
      from: "players",
      localField: "_id",
      foreignField: "_id",
      as: "playerInfo"
    }
  },
  
  // Unwind player info
  { $unwind: "$playerInfo" },
  
  // Calculate win rate and format output
  {
    $project: {
      playerId: "$_id",
      username: "$playerInfo.username",
      displayName: "$playerInfo.profile.displayName",
      totalTournaments: 1,
      wins: 1,
      winRate: {
        $round: [
          { $multiply: [{ $divide: ["$wins", "$totalTournaments"] }, 100] },
          2
        ]
      },
      totalPrizeMoney: { $round: ["$totalPrizeMoney", 2] },
      topThreeFinishes: 1,
      averagePosition: { $round: ["$averagePosition", 1] },
      gamesPlayed: { $size: "$games" }
    }
  },
  
  // Sort by win rate and total prize money
  {
    $sort: {
      winRate: -1,
      totalPrizeMoney: -1
    }
  },
  
  // Limit to top 50 players
  { $limit: 50 }
]);
```

### 3. 🔍 Advanced Queries and Indexing
```javascript
// Create compound indexes for optimal query performance
db.players.createIndex(
  { 
    "username": 1, 
    "gaming.level": -1, 
    "gaming.statistics.winRate": -1 
  },
  { 
    name: "player_search_index",
    background: true
  }
);

// Text search index for player search
db.players.createIndex(
  {
    "username": "text",
    "profile.displayName": "text",
    "profile.bio": "text"
  },
  {
    name: "player_text_search",
    weights: {
      "username": 10,
      "profile.displayName": 5,
      "profile.bio": 1
    }
  }
);

// Geospatial index for location-based features
db.players.createIndex(
  { "profile.location": "2dsphere" },
  { name: "player_location_index" }
);

// TTL index for temporary data (e.g., sessions)
db.sessions.createIndex(
  { "expiresAt": 1 },
  { expireAfterSeconds: 0 }
);

// Complex query example: Find nearby players for matchmaking
db.players.find({
  "profile.location": {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [4.3517, 50.8503] // Brussels coordinates
      },
      $maxDistance: 50000 // 50km radius
    }
  },
  "gaming.level": { $gte: 20, $lte: 50 },
  "gaming.statistics.winRate": { $gte: 0.4 },
  "preferences.matchmaking.enabled": true,
  "lastActive": {
    $gte: new Date(Date.now() - 24 * 60 * 60 * 1000) // Active in last 24h
  }
}).limit(10);
```

## 🚀 Performance Optimization

### Aggregation Pipeline Optimization:
```javascript
// Optimized pipeline for real-time leaderboard
db.players.aggregate([
  // Use $match early to reduce documents
  {
    $match: {
      "gaming.statistics.totalMatches": { $gte: 10 },
      "lastActive": {
        $gte: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000)
      }
    }
  },
  
  // Use $project to reduce data size
  {
    $project: {
      username: 1,
      "profile.displayName": 1,
      "profile.avatar": 1,
      "gaming.level": 1,
      "gaming.rank": 1,
      "gaming.statistics.wins": 1,
      "gaming.statistics.totalMatches": 1,
      winRate: {
        $divide: ["$gaming.statistics.wins", "$gaming.statistics.totalMatches"]
      }
    }
  },
  
  // Sort and limit
  { $sort: { winRate: -1, "gaming.level": -1 } },
  { $limit: 100 },
  
  // Final formatting
  {
    $project: {
      username: 1,
      displayName: "$profile.displayName",
      avatar: "$profile.avatar",
      level: "$gaming.level",
      rank: "$gaming.rank",
      winRate: { $round: [{ $multiply: ["$winRate", 100] }, 2] }
    }
  }
], { allowDiskUse: true });
```

## 📚 Learning Resources

- [Official MongoDB Documentation](https://docs.mongodb.com/)
- [MongoDB University](https://university.mongodb.com/)
- [MongoDB Aggregation Pipeline](https://docs.mongodb.com/manual/aggregation/)
- [MongoDB Performance Best Practices](https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/)

---

*"MongoDB's flexible document model makes it perfect for applications with evolving data requirements."* - Danny Verheyen

[← Back to Main Profile](../README.md)
