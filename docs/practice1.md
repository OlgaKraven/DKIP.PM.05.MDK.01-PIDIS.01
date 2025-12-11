# День 1 — Постановка задачи и разработка модели

## Цели итогового занятия

Занятие посвящено формированию базовой экономической модели оценки эффективности внедрения информационной системы (ИС).  
Команда проходит весь цикл мини‑проекта — от анализа предметной области до подготовки черновой модели расчётов.

Команда выполняет:

- выбор предметной области («клиента» из отрасли);
- определение бизнес‑проблемы клиента;
- сбор исходных данных: CAPEX, OPEX, обучение, период анализа, экономические эффекты;
- подготовку модели расчётов (ЛР6):
    - **TCO** (полная стоимость владения)
    - **ROI** (окупаемость инвестиций) 
    - **Payback Period** (срок окупаемости) 
    - **анализ чувствительности ±20%**
- создание черновой модели в Python или C#;
- подготовку документации и структуры проекта для GitHub.

---

## Организация командной работы и GitHub

Чтобы проект был прозрачным, управляемым и удобным для коллективной разработки, работа ведётся через GitHub с использованием ролей, веток, issues и project board.

### Распределение ролей

| Роль | Задачи |
|------|--------|
| Аналитик | Предметная область, бизнес‑проблема, данные |
| Разработчик API | API‑эндпоинты, структура проекта |
| Разработчик модели | Формулы, логика расчётов |
| DevOps / Docker | Контейнеризация, docker-compose |
| Документалист | Отчёты, README, презентация |

Каждый участник фиксирует прогресс в GitHub через PR и issues.

---

## Первые задачи в GitHub

- создать репозиторий;
- добавить README (название + предметная область);
- создать ветки:
  - `main`
  - `develop`
  - `feature/model`
  - `feature/api`
- загрузить шаблон проекта;
- создать issue:  
  **«Сформировать черновую модель расчёта TCO/ROI/Payback Period»**

---

## Создание GitHub‑организации

### Создание организации
При работе в команде из 2–4 человек организация позволяет избежать путаницы с доступами и права распределяются автоматически.

Переход: https://github.com/organizations/new

| Поле | Значение |
|------|----------|
| Organization name | dataforge-teamXX |
| Email | почта участника |
| Industry | Education |
| Size | 1–10 |

### Приглашение участников

Organization → People → Invite member

### Роли

- **Owner** — лидер команды  
- **Maintainer** — остальные участники

---

## Создание репозитория

Organization → **New Repository**

| Поле | Значение |
|------|----------|
| Repository name | investcalc |
| Description | Мини‑платформа для расчёта эффективности ИС |
| Visibility | Public |
| Initialize | Add README |

---

## Структура проекта

```
investcalc/
│
├── docs/
│   ├── domain.md
│   ├── problem.md
│   ├── scenarios.md
│   └── architecture.md
│
├── data/
│   ├── input-local.json
│   ├── input-cloud.json
│   └── samples/
│
├── src/
│   ├── api/
│   ├── services/
│   └── models/
│
├── tests/
├── docker/
├── .gitignore
└── README.md
```

`.gitignore` — выбрать шаблон **VisualStudio** (если C#).

---

## Ветки и стратегия работы

Упрощённый GitFlow:

| Ветка | Назначение |
|-------|------------|
| main | стабильные релизы |
| develop | рабочая версия |
| feature/model | расчётная логика |
| feature/api | API |
| feature/report | отчёты |
| feature/docker | контейнеризация |

---

## GitHub Issues

| Issue | Назначение |
|--------|------------|
| #1 Предметная область | отрасль, клиент |
| #2 Бизнес‑проблема | проблема клиента |
| #3 Модель расчётов | формулы |
| #4 API | структура эндпоинтов |
| #5 Docker | контейнеризация |
| #6 Отчёт | HTML/PDF |
| #7 Презентация | финальный питч |

Label’ы:

- analysis
- backend
- docker
- documentation
- presentation

---

## Project Board (Kanban)

Колонки:

- To Do  
- In Progress  
- Review  
- Done  

---

## Подключение участников

1. Через организацию: Settings → Manage access → Invite member.
2. Через ссылку: Repository → Collaborators → Add people.

---

## Первая загрузка кода

```bash
git clone https://github.com/dataforge-teamXX/investcalc.git
cd investcalc
git checkout -b feature/model
```

---

## Первый коммит

```bash
mkdir src/services
echo "// initial calc service" > src/services/CalcService.cs

git add .
git commit -m "feat: initial calc service layout"
git push origin feature/model
```

---

## Pull Request

1. Compare & Pull Request  
2. from → feature/model  
3. to → develop  
4. Merge после проверки  

---

## Создание релиза

GitHub → Releases → Draft a new release.

---

## Предметная область

Фиксируется в `docs/domain.md`.
Примеры:

- торговля → CRM/ERP  
- онлайн‑образование → LMS  
- медицина → запись пациентов  
- логистика → WMS/TMS  
- гостиницы → бронирование  
- производство → учёт оборудования  

---

## Бизнес‑проблема

Фиксируется в `docs/problem.md`.

Примеры:

- высокая стоимость ручных операций  
- дублирование задач  
- отсутствие прозрачности учёта  
- рост издержек  
- низкая скорость обслуживания  

---

## Сбор исходных данных

### Локальная модель

| Показатель | Значение |
|-----------|----------|
| CAPEX | ... |
| OPEX | ... |
| Обучение | ... |
| Период анализа | ... |
| Экономия | ... |
| Рост доходов | ... |

### Облачная модель

| Показатель | Значение |
|-----------|----------|
| Подписка | ... |
| Обслуживание | ... |
| Обучение | ... |
| Экономия | ... |
| Рост доходов | ... |

---

## Модель расчёта (ЛР6)

### TCO

```
TCO_local = CAPEX + OPEX * period + training
TCO_cloud = subscription * period + training
```

### ROI

```
Benefits = (savings + revenue_growth) * period
ROI = ((Benefits - TCO) / TCO) * 100
```

### Payback Period

```
PP = TCO / (savings + revenue_growth)
```

### Анализ чувствительности ±20%

Изменяются ключевые показатели → value_low и value_high.

---

## Черновая модель

Файлы:

- `src/services/CalcService.cs`  
- или `src/model/model.py`  

---

## Демонстрационный результат

Команда должна предоставить:

- `docs/domain.md`  
- `docs/problem.md`  
- `data/input-local.json` и `data/input-cloud.json`  
- `docs/scenarios.md`  
- черновой код расчётов  

---

## Демонстрация команды

Команда показывает:

- предметную область и клиента;  
- логику расчётов;  
- файл/код с вычислениями.
