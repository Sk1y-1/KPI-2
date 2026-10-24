```mermaid
erDiagram
    PLAYER {
        UUID id PK
        string username
        number total_experience
        number health
        number movespeed
        number damage
        number attack_speed
    }
    GAME_SESSION {
        UUID id PK
        UUID player_id FK
        datetime start
        datetime end
        string status "in_progress | completed | abandoned"
        number earned_experience
        number waves_survived
    }
    SKILL_CARD {
        UUID id PK
        UUID unlock_enemy_id FK
        string name
        number base_weight
        number damage
        string type_damage
        string describe_ability
        number cooldown
    }
    ENEMY {
        UUID id PK
        string name
        number base_health
        number movespeed
        number damage
        string type "normal | elite | boss"
        string describe_enemy
    }
    BESTIARY {
        UUID id PK
        UUID player_id FK
        UUID enemy_id FK
        number counter
    }
    PLAYER ||--o{ GAME_SESSION : "has"
    PLAYER }o--o{ SKILL_CARD : "starts_with"
    PLAYER }o--o{ SKILL_CARD : "has_unlocked"
    GAME_SESSION }o--o{ SKILL_CARD : "unlocks"
    GAME_SESSION }o--o{ ENEMY : "spawns"
    PLAYER ||--o{ BESTIARY : "tracks_kills"
    ENEMY ||--o{ BESTIARY : "references"
    ENEMY |o--o{ SKILL_CARD : "unlocks_card"
```
