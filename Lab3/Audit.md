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
 
| # | Розбіжність | Де видно | Вимоги | Пропозиція | Статус |
|---|---|---|---|---|---|
| 1 | Зв'язок `Player M:N Skill_Card` (`has_unlocked`) у SQLite потребує сполучної таблиці, а в моделі її немає; без унікального ключа (`player_id`, `skill_card_id`) повторне розблокування (SC-5) нічим не заблоковане, крім перевірки `Bestiary` | SC-3, SC-4, SC-5: `Player.addUnlockedSkills` | REQ-06 | додати `PLAYER_SKILL_CARD (id, player_id, skill_card_id)` з UNIQUE (`player_id`, `skill_card_id`) | не виправлено в ER |
| 2 | `Player.total_experience` похідне від `earned_experience` сесій (порушення 3NF, розбіжність 3 з Lab2). Через це `addExperience` мусить бути в тій самій транзакції, а повтор із черги (SC-6) мусить теж нараховувати досвід, чого REQ-02 і REQ-07 прямо не вимагають | SC-1, SC-6 | REQ-02, REQ-07, NFR-06 | у REQ-07 дописати нарахування досвіду при повторі; або обчислювати суму із `Game_Session` | діаграми узгоджено (досвід у транзакції повтору), вимоги й ER не виправлено |
| 3 | `SaveQueue` і мітка `markForSyncRetry` не мають сутності: повтор «при наступному запуску» вимагає збереження між запусками, а черга в пам'яті зникає разом із процесом | SC-6, SC-7 | REQ-07, REQ-09 | таблиця `PENDING_SAVE (id, session_id FK, attempts, kind)` або `Game_Session.status = pending` і поле `cloud_sync_pending` | не виправлено в ER |
