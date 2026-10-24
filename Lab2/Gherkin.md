Feature: Завершити забіг (UC-3)
  Як гравець,
  я хочу завершити забіг і зберегти його результат,
  щоб мій прогрес не пропав і я бачив, чого досяг.

  Background:
    Given триває забіг гравця

  @REQ-06 @REQ-07
  Scenario Outline: Завершення забігу з різних причин
    When забіг завершується через "<причина>"
    Then в Game_Session записано end, earned_experience і waves_survived
    And Player.total_experience збільшено на earned_experience цього забігу

    Examples:
      | причина                     |
      | смерть гравця               |
      | ручний вихід                |
      | проходження останньої хвилі |

  @REQ-08 @REQ-17
  Scenario: Оновлення бестіарію та підсумок забігу
    Given у забігу вбито 5 ворогів типу "Goblin" і 3 типу "Bat"
    And у Bestiary немає запису для "Goblin"
    And у Bestiary є запис для "Bat" зі counter = 10
    When забіг завершується
    Then для "Goblin" створюється запис зі counter = 5
    And для "Bat" counter дорівнює 13
    And гравець бачить підсумок із waves_survived, earned_experience і ворогом "Goblin" як новим у бестіарії

  @REQ-18
  Scenario Outline: Розблокування навички за першого вбивства елітки або боса
    Given Enemy "<ворог>" має тип "<тип>" і з ним пов'язана Skill_Card "<картка>"
    And у Bestiary немає запису для "<ворог>"
    And у забігу вбито "<ворог>"
    When забіг завершується
    Then Skill_Card "<картка>" стає доступною гравцеві

    Examples:
      | ворог        | тип   | картка    |
      | Goblin Mage  | elite | Fireball  |
      | Goblin King  | boss  | Ice Shard |
 
  @REQ-18
  Scenario: Звичайний моб не розблоковує навичку
    Given Enemy "Goblin" має тип "normal"
    And у Bestiary немає запису для "Goblin"
    And у забігу вбито "Goblin"
    When забіг завершується
    Then жодна нова Skill_Card не стає доступною

  @REQ-18
  Scenario: Повторне вбивство боса не розблоковує навичку вдруге
    Given Enemy "Goblin King" має тип "boss" і з ним пов'язана Skill_Card "Ice Shard"
    And у Bestiary є запис для "Goblin King" зі counter = 3
    And у забігу вбито "Goblin King"
    When забіг завершується
    Then Skill_Card "Ice Shard" не додається вдруге

  @REQ-09
  Scenario: Помилка збереження результату
    Given запис у базу даних не вдається
    When забіг завершується
    Then результат поміщається в локальну чергу
    And запис повторюється при наступному запуску, не більше 3 спроб

  @REQ-15
  Scenario: Steam недоступний при хмарному збереженні
    Given хмарні збереження ввімкнені
    And Steam не відповідає протягом 5 секунд
    When забіг завершується
    Then прогрес лишається збереженим локально
    And синхронізація повторюється при наступному запуску

