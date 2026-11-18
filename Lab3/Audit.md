# Аудит діаграм Lab3
 
Перевірено діаграми `diagrams/SC-1…SC-7` проти `Lab2/EDR_v2.md` (ER-модель), `Lab2/Gherkin.md` і `Lab2/EARS.md`.
 
## Учасники та ER-модель
 
| Учасник | Сутність у ER | Висновок |
|---|---|---|
| Game_Session | `GAME_SESSION` (`end`, `status`, `earned_experience`, `waves_survived`) | збігається, `closeSession` пише всі поля REQ-01 |
| Player | `PLAYER.total_experience` | збігається, але див. розбіжність 2 |
| Player (`addUnlockedSkills`) | зв'язок `has_unlocked` без таблиці | розбіжність 1 |
| Bestiary | `BESTIARY` (`player_id`, `enemy_id`, `counter`) | збігається |
| Enemy | `ENEMY.type` (`normal / elite / boss`) | збігається |
| Skill_Card | `SKILL_CARD.unlock_enemy_id` | збігається, `findByUnlockEnemy` читає FK |
| SaveQueue | немає | розбіжність 3 |
| KillTracker | немає, лише пам'ять (REQ-03) | допустимо: дані забігу, що не зберігаються |
| GameLoop, GameLauncher, RunService, UI, DB | немає, службові компоненти | допустимо |
| Steam | зовнішня система | допустимо |
 
## Кроки діаграм і Gherkin
 
| SC | Gherkin | Діаграма | Збіг |
|---|---|---|---|
| SC-1 | 3 причини; запис `end`, `earned_experience`, `waves_survived`; `total_experience` += | alt із 3 гілками; `closeSession`; `addExperience` після `closeSession` | так |
| SC-2 | Goblin: create, counter = 5; Bat: 10 → 13; підсумок із Goblin як новим | loop по `killCounts`; alt create / `incrementCounter`; `newEnemies` у `showSummary` | так |
| SC-3 | elite / boss без запису в Bestiary: картка доступна | loop; opt `entry = null`; opt `elite або boss`; `findByUnlockEnemy`; `addUnlockedSkills` | так |
| SC-4 | normal без запису: нова картка не з'являється | alt: гілка `normal` без викликів Skill_Card і Player | так |
| SC-5 | boss із записом (counter = 3): картка не додається вдруге | alt: гілка «запис існує» лише `incrementCounter`, без `addUnlockedSkills` | так |
| SC-6 | помилка запису; черга; не більше 3 спроб | alt `DBWriteError`; `rollback`; `enqueue`; loop `attempts < 3`; `retryPending` | так, плюс помилка після 3-ї спроби (немає в Gherkin) |
| SC-7 | Steam мовчить 5 с: локально збережено; повтор при наступному запуску | par; alt `ack` / таймаут; `markForSyncRetry` | так |
 
## Топ-3 розбіжності
 
| № | Розбіжність | Виправлення | Файли |
| :--- | :--- | :--- | :--- |
| 1 | Немає таблиці для `has_unlocked` | Нова сутність `PLAYER_SKILL_CARD` з `unlocked_at` і `UNIQUE (player_id, skill_card_id)`. У діаграмах SC-3, SC-4, SC-5 виклик став `insertIfAbsent`. `REQ-06` тепер вимагає кожну картку ворога й унікальний запис. | `ERD_v3.md`, `EARS_v2.md` |
| 2 | `total_experience` порушує 3NF | `ard.md ` обґрунтовує це як кеш суми `earned_experience`. `REQ-02` вимагає оновлення в тій самій транзакції, зокрема при повторі з черги. | `ard.md`, `EARS_v2.md` |
| 3 | `SaveQueue` без сутності | Нова таблиця `PENDING_SAVE` (з `killCounts` у `payload`). У `PLAYER` додано `cloud_sync_enabled` і `cloud_sync_pending`. | `ERD_v3.md`, `EARS_v2.md` |

