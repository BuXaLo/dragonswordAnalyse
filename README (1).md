# DragonSword: Awakening — анализ игровых таблиц

Репозиторий содержит извлечённые из игровых данных XML-таблицы, русские строки локализации и инвентарные списки доступных таблиц. Цель анализа — восстановить каталог персонажей и навыков, классификацию действий, переходы комбо, параметры ударов и связи с эффектами.

> **Статус документа:** схема построена по фактическим именам полей и примерам записей доступных таблиц. Там, где связь между таблицами ещё не доказана общим ключом, она помечена как гипотеза/требующая проверки. Числа `AttackWeight` и `DamageWeight` не следует автоматически считать процентами от ATK.

## Содержание

- [Состав репозитория](#состав-репозитория)
- [Что описывает каждый файл](#что-описывает-каждый-файл)
- [Схема связей](#схема-связей)
- [Ключевые поля](#ключевые-поля)
- [Как восстанавливать навыки и урон](#как-восстанавливать-навыки-и-урон)
- [Ограничения и следующие шаги](#ограничения-и-следующие-шаги)

## Состав репозитория

### Инвентарные файлы

| Файл | Назначение |
|---|---|
| `Xml_list.txt` | Список путей к XML, извлечённым из `DS/Content/__GeneratedGameData__/Server/XML/GameData/`. Это **инвентарь путей**, а не содержимое самих таблиц. |
| `table_list.txt` | Список путей к `.table` из `DS/Content/Design/GameData/`. Также инвентарь, а не сами данные. |
| `StringData_ru.txt` | Экспорт русских локализованных строк в JSON-подобном формате. Несмотря на расширение `.txt`, файл нужно читать как JSON. |

### Таблицы персонажей, навыков и боевой системы

| Файл | Назначение и полезные поля | Степень подтверждения |
|---|---|---|
| `PCCharacterData.xml` | Основная запись персонажа: `ID`, `Name`, `Desc`, `BluePrint`, `AnimInst`, `ComboTableID`, `StatTableID`, `PassiveActionBuffList`, `ActiveActionBuffList`, тип базовой атаки и другие настройки. | Подтверждено по полям таблицы. |
| `PCCharacterLevelData.xml` | Уровневые параметры персонажей и опыт/прогрессия. Использовать вместе с `PCCharacterData.xml`; точные поля нужно читать из схемы конкретной версии. | Назначение следует из имени; поля требуют проверки. |
| `PCCharacterAscendData.xml` | Данные возвышения персонажа. Нужны для оценки изменений характеристик и разблокировок на этапах прогрессии. | Назначение следует из имени; поля требуют проверки. |
| `PCCharacterTranscendData.xml` | Данные трансценденции. Важно для вариантов навыков/комбо, доступных при разных значениях `Transcend`. | Назначение следует из имени; поля требуют проверки. |
| `PCSkillGrowthData.xml` | Каталог прокачиваемых слотов навыков. Подтверждённые поля: `ID`, `CharacterID`, `TypeValue`, `SlotLevel`, `ComboChangeID`, `SkllType`, `SlotOrder`, `SlotType`, `Name`, `SkillDesc` и ссылки на материалы прокачки. | Подтверждено по полям таблицы. |
| `PCSkillLevelData.xml` | Параметры уровней эффектов/действий: `ActionBuffID`, `Level`, `Desc`, `Duplication`, `Duration`, `CoolTime`, `BuffID`, `StringVarArr`. `ActionBuffID` не следует считать ID навыка без дополнительной связи. | Подтверждено по полям таблицы. |
| `PCComboData.xml` | Каталог комбо-действий персонажей. Поля включают `ID`, `Index`, `Desc_Character`, `Desc_Combo`, `GroupIndex`, `SubGroupIndex`, `ActionBuffList`, `OpenCondition`, `CoolTime`, `InitCoolTime`, `CoolTimeShare`, `MaxFury`, `FuryStack`, `TargetCondition`, `ActiveCondition`, `AttackWeight`, `HitFury`, `HitAggro`, `SuperArmorBreak`. | Подтверждено по полям таблицы. |
| `PCComboLevelData.xml` | Уровневые параметры комбо: `ComboID`, `GroupIndex`, `SubGroupIndex`, `Level`, `ActionBuffList`, `ActionBuffLevel`, `CoolTime`, `AttackWeight`, `HitFury`, `HitAggro`, `SuperArmorBreak`. Используется для проверки, как параметры меняются с уровнем. | Подтверждено по полям таблицы. |
| `PCComboChangeData.xml` | Таблица замены комбо в зависимости от условий прогрессии. Поля `KarmaID`, `Master`, `Transcend`, `OriginalComboIndex`, `ChangeComboIndex`, `ChangeAnimSet` описывают исходные и заменяющие индексы. Списки индексов разделены `#`; элементы следует сопоставлять попарно по позиции. | Подтверждено по полям таблицы и формату примеров. |
| `PCActionDamageData.xml` | Привязка боевых действий/анимаций к параметрам ударов. Подтверждённые поля: `ID`, `Name`, `Type`, `ActionIndex`, `AnimationName`, `AttackKey`, `ComboIndex`, `DamageWeight`, `SuperArmorWeight`, `FixedSuperArmorValue`, `FuryWeight`, `MesmerizedParamList`, `HitGroupIndex`. | Подтверждено по полям таблицы. |
| `PCActionMonsterLinkData.xml` | Связи анимационных действий с событиями взаимодействия/захвата цели. Поля включают `ID`, `Name`, `ActionIndex`, `AnimationName`, `NotifyKey`, `ComboIndex`, `MonsterLinkIndex`. Полезно для действий с несколькими notify/hit-событиями и захватов. | Подтверждено по полям таблицы. |
| `PCGeneralActionSetData.xml` | Общая машина переходов действий. Поля `ActionSet`, `ComboIndex`, `ActionIndex`, `CurrentAction`, `NextAction`, `ChangeActionSet`, `AnimationIndex`, `AnimationTag`, `ConditionCheck1..5`, `ConditionValue1..5` описывают действия и условия перехода. Важно для общих действий, но само по себе не является полной таблицей ввода/управления. | Подтверждено по полям таблицы. |
| `PCStatData.xml` | Боевые параметры/сопротивления, в том числе `HP`, `Attack`, `Defence`, критические параметры, скорость, суперброня, сопротивления/длительности состояний. Примеры могут иметь нулевые HP/Attack/Defence, поскольку таблица содержит базовые поправки/сопротивления, а не обязательно итоговые статы персонажа. | Подтверждено по полям таблицы. |
| `PCWeaponData.xml` | Привязка ID оружия к ресурсам и точкам крепления: `ID`, `Resource`, `DefaultSocket`, `CombatSocket`, `HitDirectionID`. Полезна для определения ресурсов оружия и анимационно-боевого контекста, но не является таблицей коэффициентов урона. | Подтверждено по полям таблицы. |
| `SkillData.xml` | Общая таблица игровых навыков/эффектов. Поля включают `ID`, `Type`, `GroupID`, `TargetCondition`, `ActiveCondition`, `EndCondition`, `Duration`, `CoolTime`, `BuffID`, `DataAssetPath`, `WeaponChangeValue`. `DataAssetPath` может указывать на дополнительные параметры в DataAsset/Blueprint. | Подтверждено по полям таблицы. |
| `SkillComboConditionCheckData.xml` | Справочник проверок условий: `ID`, `Desc_Name`, `CheckType`, `TeamCheck`, `AccountCheck`, `GameObjectType`, `ApplyTargetCount`, `MinRange`, `MaxRange` и до четырёх пар типов/значений условий. Использовать для расшифровки числовых `TargetCondition` и других проверок, если ID совпадают. | Подтверждено по полям таблицы; конкретные ссылки проверять по ключам. |
| `GameBuffData.xml` | Эффекты баффов/дебаффов: `ID`, `Desc_Buff`, наборы `BuffTypeN`, `BuffValueN`, `BuffCategoryN`. Примеры показывают неуязвимость, изменение скорости действий, восстановление ярости/выносливости, лечение и добавление характеристик. | Подтверждено по полям таблицы. |

> Если в текущей ветке добавлены другие XML/TXT, их нужно внести в таблицу выше после проверки самих полей. `Xml_list.txt` и `table_list.txt` перечисляют множество файлов игры, но наличие имени в инвентаре само по себе не означает, что содержимое этой таблицы закоммичено в репозиторий.

## Что описывает каждый файл

### 1. `PCCharacterData.xml` — карточка персонажа

Основной ключ записи — `ID` (в примерах это ID боевого объекта/персонажа). Поля `ComboTableID` и `StatTableID` задают ссылки на соответствующие таблицы. `BluePrint` и `AnimInst` указывают на ресурсы логики/анимаций Unreal Engine. `ActiveActionBuffList` и `PassiveActionBuffList` — списки ID, разделённые `#`; их нельзя автоматически трактовать как список пользовательских навыков, пока не установлено, какая таблица использует каждый ID.

### 2. `PCSkillGrowthData.xml` — навыки в интерфейсе прокачки

Записи могут повторяться для разных `SlotLevel`. Поэтому для получения уникального навыка группировать следует не только по `ID`: проверять комбинацию `CharacterID`, `SlotOrder`, `SlotType`, `TypeValue`, `Name`, `SkillDesc` и уровня. `Name` и `SkillDesc` — строковые ID, которые разрешаются через `StringData_ru.txt` (ключ ID → `SourceString`).

Поле `ComboChangeID` указывает на потенциальную связь с заменой комбо, но конкретную семантику необходимо проверять по значениям `PCComboChangeData.ID` и записям персонажа. Не следует считать `TypeValue` универсальным перечислением для всех таблиц без проверки.

### 3. `PCComboData.xml` и `PCComboLevelData.xml` — комбо и их уровни

`PCComboData.ID` идентифицирует группу/персонажа в этой таблице; `Index` — индекс конкретного варианта действия. Например, у Джерома (`ID=1000013`) присутствуют записи для уклонения (`Index=701`), прыжка (`801`), атаки после уклонения (`901`), дополнительного ввода после неё (`902`), атаки после рывка (`915`) и вариантов сигнального/активного навыка.

Не путать:
- `ID` — владелец/группа комбо в `PCComboData`;
- `Index` — индекс конкретного комбо-действия;
- `ComboID` в `PCComboLevelData` — ключ уровня комбо, который **не следует автоматически приравнивать к `PCComboData.ID`**. Сопоставлять уровневые строки нужно по полям, фактически совпадающим в обеих таблицах (`GroupIndex`, `SubGroupIndex`, `Level` и соответствующему ID/смыслу ComboID).

`AttackWeight` из `PCComboData`/`PCComboLevelData` — боевой параметр комбо. Не доказано, что он является тем же самым, что `DamageWeight` из `PCActionDamageData`, или что любое из этих значений является непосредственным множителем ATK.

### 4. `PCActionDamageData.xml` — события удара в анимации

Одна анимация может иметь несколько строк из-за разных `AttackKey` или нескольких hit-событий. Для анализа удара сохранять как минимум кортеж:

`Name + ActionIndex + AnimationName + ComboIndex + AttackKey + HitGroupIndex`

`DamageWeight` описывает вес/долю события в системе урона, но точная формула перевода этого поля в числовой урон требует подтверждения. Нельзя просто сложить все строки по `ComboIndex`, не проверив дубликаты, `AttackKey` и принадлежность конкретному персонажу/анимации.

### 5. `PCActionMonsterLinkData.xml` — события взаимодействия с целью

`ActionIndex` и `AnimationName` связывают запись с конкретным действием/анимацией; `NotifyKey` может отличать события внутри анимации, а `MonsterLinkIndex` — выбирать вариант взаимодействия. Эти записи нужны прежде всего для захватов, multi-mount и действий, где урон/захват возникает через анимационные notify.

### 6. `PCComboChangeData.xml` — замена вариантов комбо

`OriginalComboIndex` и `ChangeComboIndex` — последовательности, разделённые `#`. Внутри одной записи их следует разбивать и сопоставлять попарно по позиции, а не сортировать независимо. Фильтры `Master`, `Transcend`, `KarmaID` определяют вариант прогрессии. Для финальной схемы проверить, каким образом `PCSkillGrowthData.ComboChangeID` указывает на `PCComboChangeData.ID`.

### 7. `SkillData.xml`, `PCSkillLevelData.xml`, `GameBuffData.xml` — эффекты

Это разные уровни модели:
- `SkillData.ID` — определение навыка/эффекта, с типом, целями и длительностью;
- `PCSkillLevelData.ActionBuffID` — уровень параметров действия/баффа;
- `PCSkillLevelData.BuffID` — ссылка на эффект баффа, которую следует проверять в `GameBuffData.ID`;
- `GameBuffData.BuffTypeN` и `BuffValueN` — конкретные эффекты и значения.

Эти связи нельзя сливать в одну таблицу только по совпадению названий. Нужны проверка ключей и цепочка ID.

### 8. `PCStatData.xml` и `PCWeaponData.xml`

`PCStatData` описывает наборы боевых параметров и сопротивлений. `PCCharacterData.StatTableID` — кандидат на ссылку к `PCStatData.ID`, подтверждаемый совпадением значений ID в примерах. `PCWeaponData` описывает оружейные ресурсы/сокеты; прямого коэффициента урона в показанных полях нет.

### 9. `StringData_ru.txt`

Файл содержит объект `Data`, где ключ — строковое представление ID, а запись содержит `ID` и `SourceString`. Сопоставлять `PCSkillGrowthData.Name`, `SkillDesc`, `PCCharacterData.Name`/`Desc` с локализацией следует как `int/string ID → Data[ID].SourceString`. При анализе не заменять отсутствующий перевод догадкой: сохранять исходный ID и помечать строку как неразрешённую.

## Схема связей

```mermaid
flowchart TD
    CH[PCCharacterData<br/>ID, ComboTableID, StatTableID] -->|ComboTableID / проверка ключа| COMBO[PCComboData<br/>ID, Index, GroupIndex, SubGroupIndex]
    CH -->|StatTableID| STAT[PCStatData]
    CH -->|Name, Desc: строковые ID| STR[StringData_ru.txt]
    GROW[PCSkillGrowthData<br/>CharacterID, SlotOrder, SlotType, TypeValue] -->|Name, SkillDesc: строковые ID| STR
    GROW -->|ComboChangeID: связь требует проверки| CHANGE[PCComboChangeData<br/>OriginalComboIndex → ChangeComboIndex]
    COMBO -->|Index ↔ ComboIndex: проверять вместе с актором/анимацией| DMG[PCActionDamageData<br/>ActionIndex, AnimationName, AttackKey, DamageWeight]
    COMBO -->|Index ↔ ComboIndex: проверять ключи| LINK[PCActionMonsterLinkData<br/>NotifyKey, MonsterLinkIndex]
    COMBO -->|GroupIndex, SubGroupIndex, Level и ключ ComboID — сверить| LEVEL[PCComboLevelData<br/>AttackWeight, CoolTime]
    SKILL[SkillData<br/>ID, BuffID, DataAssetPath] -->|BuffID| BUFF[GameBuffData<br/>BuffTypeN, BuffValueN]
    PLEVEL[PCSkillLevelData<br/>ActionBuffID, Level, BuffID] -->|BuffID: проверить наличие| BUFF
    PLEVEL -->|ActionBuffID: смысловой мост ещё проверить| SKILL
    COND[SkillComboConditionCheckData<br/>ID, CheckType, TargetCondition] -. условные ссылки требуют проверки .-> COMBO
    ACTION[PCGeneralActionSetData<br/>ActionSet, CurrentAction, NextAction, ConditionCheck] -. переходы действий .-> COMBO
    WEAPON[PCWeaponData<br/>Resource, sockets]
    CH -. ресурсы персонажа .-> WEAPON
```

### Надёжность связей

| Связь | Статус | Примечание |
|---|---|---|
| `PCCharacterData.Name/Desc` и `PCSkillGrowthData.Name/SkillDesc` → `StringData_ru.txt` | Подтверждена форматом строковых ID | Ключ строки сопоставляется с ID. |
| `PCCharacterData.StatTableID` → `PCStatData.ID` | Подтверждается совпадением ID в примерах | Проверять по всем персонажам и уникальности ключей. |
| `PCComboData.Index` ↔ `PCActionDamageData.ComboIndex` | Сильный кандидат на связь | Требуется учитывать владельца/модель (`Name`, `ID`) и `ActionIndex`, так как индексы могут повторяться между персонажами. |
| `PCComboData.Index` ↔ `PCActionMonsterLinkData.ComboIndex` | Сильный кандидат на связь | Аналогично проверять по персонажу/действию и notify. |
| `PCComboChangeData.OriginalComboIndex` → `ChangeComboIndex` | Подтверждена внутри таблицы | Списки позиционные; нужен контекст `KarmaID`, `Master`, `Transcend`. |
| `PCSkillGrowthData.ComboChangeID` → `PCComboChangeData.ID` | Вероятная связь | Проверить фактическое распределение значений и количество совпадений. |
| `PCSkillLevelData.BuffID` → `GameBuffData.ID` | Проверяемая прямая связь по ключу | Проверять нулевые/пустые значения отдельно. |
| `PCSkillLevelData.ActionBuffID` ↔ `SkillData.ID` | Не считать подтверждённой автоматически | Нужно проверить совпадения и смысл поля в контексте. |
| `PCComboData.AttackWeight` ↔ `PCActionDamageData.DamageWeight` | Не эквивалентны по одному лишь имени | Нужны измерения/формула либо дополнительная таблица/логика Blueprint. |

## Ключевые поля

| Поле | Интерпретация |
|---|---|
| `CharacterID` | ID персонажа в таблице навыков; может отличаться от `PCCharacterData.ID` и `PCComboData.ID`. Проверять мосты по таблицам, не переносить ID между доменами без подтверждения. |
| `TypeValue`, `SlotType`, `SlotOrder`, `SkllType` | Классификация/расположение навыка в системе прокачки. Значения нужно описывать как наблюдаемые enum-коды; не приписывать универсальные значения без проверки всех строк. |
| `Index` | Индекс комбо-действия в `PCComboData`; например, у Джерома `701` — dodge, `801` — jump, `901` — dodge attack, `902` — дополнительный ввод после dodge attack, `915` — dash attack. |
| `GroupIndex`, `SubGroupIndex` | Группировка действий и вариантов комбо; полезны для уровневых/вариантных строк, но не гарантируют уникальность записи сами по себе. |
| `ComboIndex` | Индекс комбо в таблицах анимационных действий; связывать с `PCComboData.Index` только с дополнительной проверкой контекста. |
| `ActionIndex` | Идентификатор действия/анимационного action; полезен для группировки строк урона и событий взаимодействия. |
| `AttackKey` | Идентификатор конкретного attack/hit-события в `PCActionDamageData`; разные ключи могут относиться к одной анимации. |
| `DamageWeight` | Вес события урона; не доказанный прямой процент от ATK. |
| `AttackWeight` | Вес/параметр атаки комбо; не считать синонимом `DamageWeight`. |
| `ActionBuffList`, `ActiveActionBuffList`, `PassiveActionBuffList` | Списки ID, разделённые `#`; для каждого ID необходимо определить таблицу-владелец. |
| `TargetCondition`, `ActiveCondition`, `OpenCondition` | Условия цели, активации и открытия. Пустые значения и числовые значения требуют раздельной интерпретации. |
| `StringVarArr` | Массив параметров в строке, разделённой `#`; семантика зависит от связанного навыка/описания, не универсальна. |

## Как восстанавливать навыки и урон

Для каждой записи создаётся нормализованная строка результата со следующими полями:

1. `character_id` и подтверждённое имя персонажа;
2. тип действия: обычная атака / прыжок / воздушная атака / уклонение / атака после уклонения / атака после рывка / активный / сигнальный / дополнительный вариант;
3. `PCComboData.ID`, `Index`, `GroupIndex`, `SubGroupIndex`;
4. отображаемое имя/описание из локализации, если доступно;
5. условия `OpenCondition`, `TargetCondition`, `ActiveCondition`, `IconActivateConditionType`;
6. `CoolTime`, `InitCoolTime`, `CoolTimeStack`, `CoolTimeShare`;
7. параметры `AttackWeight`, `HitFury`, `HitAggro`, `SuperArmorBreak`;
8. соответствующие `ActionIndex`, `AnimationName`, `AttackKey`, `DamageWeight`;
9. эффекты из `ActionBuffList`/`BuffID` после разрешения ID;
10. статус каждого join: подтверждён / вероятен / не разрешён.

### Порядок анализа

1. Разрешить `PCCharacterData` и сопоставить персонажей с `ComboTableID`, `StatTableID`, Blueprint и AnimInst.
2. Для каждого персонажа получить `PCComboData` по его ID и сгруппировать действия по `GroupIndex`/`SubGroupIndex`, `Index` и описанию.
3. Применить `PCComboChangeData` с учётом `KarmaID`, `Master` и `Transcend`, чтобы построить варианты комбо.
4. Связать combo-index с `PCActionDamageData` и `PCActionMonsterLinkData`; при совпадении индекса проверять `Name`, `ID`, `ActionIndex`, `AnimationName`, `AttackKey`.
5. Добавить параметры `PCComboLevelData` по уровню, не смешивая `ComboID` с `PCComboData.ID` без доказанного ключа.
6. Разрешить баффы и условия по `SkillData`, `PCSkillLevelData`, `GameBuffData` и `SkillComboConditionCheckData`.
7. Перевести строковые ID в русский текст через `StringData_ru.txt`.
8. Только после этого сопоставлять игровые коэффициенты с измеренным уроном и выводить формулу.

## Ограничения и следующие шаги

- `Xml_list.txt` содержит около 800 путей, а `table_list.txt` — список `.table`; это не означает, что все эти файлы представлены в репозитории как содержимое.
- Часть полей ссылается на Blueprint/DataAsset по пути Unreal Engine. Если таблицы не объясняют логику, потребуется исследовать соответствующие ассеты и/или код событий анимации.
- Для доказательства формулы урона нужны контрольные игровые замеры с известными ATK, критом, защитой цели и несколькими отдельными ударами.
- Не объединять записи только по одному числовому индексу: в разных персонажах/типах действий индексы могут повторяться.
- В документации отделять «поле прямо присутствует в таблице» от «смысл поля подтверждён» и от «формула подтверждена игровым тестом».
