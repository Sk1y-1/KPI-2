``` mermaid 
sequenceDiagram
actor P as Гравець
participant B as Bestiary
participant E as Enemy 
 P->>GS: finishRun(sessionId, reason)
activate GS
GS->>B: findByPlayerAndEnemy(playerId, BatId)
activate B
B-->>GS: null
deactivate B
GS->>E: getType(BatId)
 activate E
    E-->>GS: normal
    deactivate E
    alt type = elite або boss
        GS->>GS: unlockSkill()
    else type = normal
        Note over GS: навичка не розблоковується
    end
    GS-->>P: showSummary(..., unlockedSkills = [])
    deactivate GS
```
