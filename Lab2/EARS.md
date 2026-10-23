| ID | Тип EARS | Вимога | UC |
|---|---|---|---|
| REQ-01 | WHEN | WHEN гравець запускає забіг, система SHALL створити запис `Game_Session` з `player_id` цього гравця та `start` = поточний час. | UC-1 |
| REQ-02 | WHEN | WHEN запис `Game_Session` створено, система SHALL застосувати до забігу `health`, `movespeed`, `damage`, `attack_speed` гравця та його стартові `Skill_Card` (зв'язок `starts_with`). | UC-1 |
| REQ-03 | WHILE | WHILE у гравця є незавершений забіг, система SHALL відхиляти запит на старт нового забігу. | UC-1 |
| REQ-04 | WHEN | WHEN гравець підвищує рівень під час забігу, система SHALL запропонувати 3 `Skill_Card`, обрані випадково з імовірністю, пропорційною `base_weight`. | UC-2 |
| REQ-05 | WHEN | WHEN гравець обирає `Skill_Card`, система SHALL зберегти зв'язок `unlocks` між поточною `Game_Session` і цією карткою. | UC-2 |
| REQ-06 | WHEN | WHEN забіг завершується (смерть гравця, ручний вихід або проходження останньої хвилі), система SHALL записати `end`, `earned_experience` і `waves_survived` у `Game_Session`. | UC-3 |
| REQ-07 | WHEN | WHEN результат забігу збережено, система SHALL збільшити `Player.total_experience` на `earned_experience` цього забігу. | UC-3 |
| REQ-08 | WHEN | WHEN забіг завершується, система SHALL збільшити `Bestiary.counter` для кожного типу ворога на кількість ворогів цього типу, вбитих у забігу, створивши запис `Bestiary` зі `counter` = цій кількості, якщо пари (`player_id`, `enemy_id`) ще немає. | UC-3, UC-4 |
| REQ-09 | IF…THEN | IF запис результату забігу не вдалося зберегти, THEN система SHALL помістити результат у локальну чергу та повторити запис при наступному запуску, не більше 3 спроб. | UC-3 |
| REQ-10 | WHEN | WHEN гравець відкриває бестіарій, система SHALL показати кожен `Enemy` із `counter` > 0: `name`, `type`, `describe_enemy`, `counter`. | UC-5 |
| REQ-11 | WHEN | WHEN дизайнер змінює `Enemy` або `Skill_Card`, система SHALL зберегти зміни, не змінюючи вже завершені `Game_Session`. | UC-6 |
| REQ-12 | IF…THEN | IF дизайнер задає `Skill_Card.base_weight` ≤ 0, THEN система SHALL відхилити зміну з повідомленням про помилку. | UC-2, UC-6 |
| REQ-13 | WHEN | WHEN користувач ініціює вхід, система SHALL передати дані Auth Provider і SHALL надати доступ лише після його підтвердження. | UC-1, UC-6, UC-7 |
| REQ-15 | IF…THEN | IF Auth Provider не відповів протягом 5 секунд, THEN система SHALL відхилити спробу входу та показати повідомлення «Сервіс автентифікації недоступний, спробуйте пізніше». | UC-1, UC-6, UC-7 |

