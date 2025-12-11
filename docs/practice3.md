# День 3 - Анализ чувствительности, генерация отчёта и Docker-контейнеризация

(третья неделя проектной работы, создание полноценного прототипа сервиса)

Третья неделя посвящена превращению каркаса InvestCalc в полноценный прототип сервиса. На этом этапе команда добавляет анализ чувствительности, реализует генерацию отчёта и упаковывает приложение в Docker-контейнер.

## Цели занятия

К концу занятия команда должна:

- реализовать анализ чувствительности ±20 % для ключевых параметров:
  - CAPEX;
  - OPEX;
  - SaaS-подписка (cloud-сценарий);
  - экономия в год;
  - рост доходов;
- добавить генерацию отчёта, включающего:
  - HTML-отчёт (обязательно);
  - опционально — генерацию PDF на основе HTML;
- подготовить контейнеризацию сервиса:
  - создать Dockerfile для приложения;
  - при необходимости — настроить docker-compose (например, если используется БД);
  - протестировать сборку и запуск контейнера;
- обновить репозиторий на GitHub:
  - оформить новые ветки (`feature/sensitivity`, `feature/report`, `feature/docker`);
  - сделать осмысленные коммиты;
  - создать Pull Request в `develop`;
  - дополнить README инструкциями по запуску, описанием новых эндпоинтов и Docker-сборки.

---

## Реализация анализа чувствительности (Sensitivity Analysis)

Анализ чувствительности позволяет понять, как изменение ключевых параметров на ±20 % влияет на TCO, ROI и Payback Period.

### Параметры для анализа

Анализ проводится для следующих параметров:

- CAPEX (капитальные затраты);
- OPEX (операционные затраты);
- стоимость подписки SaaS (для облачного сценария);
- экономия в год (savings);
- рост доходов в год (revenue growth).

### Формулы вариаций

Для каждого параметра рассчитываются два варианта:

```text
value_low  = value * 0.8   // уменьшение на 20 %
value_high = value * 1.2   // увеличение на 20 %
```

Для каждой вариации выполняется полный расчёт:

- TCO,
- ROI,
- Payback Period.

### Пример структуры ответа (JSON)

```json
{
  "parameter": "opex",
  "base": 120000,
  "low": {
    "value": 96000,
    "tco": 840000,
    "roi": 181.0,
    "payback": 1.2
  },
  "high": {
    "value": 144000,
    "tco": 1080000,
    "roi": 140.2,
    "payback": 1.8
  }
}
```

Аналогичная структура может возвращаться для `capex`, `subscription`, `savings`, `revenue`.

---

## Декомпозиция API и файлов проекта на третьей неделе

На этом этапе логично разделить API на несколько файлов, чтобы:

- разных участников можно было закрепить за разными контроллерами/частями API;
- код был модульным и читаемым.

### Рекомендуемая структура `src/api/` (C#-вариант)

```text
src/
│
├── api/
│   ├── CalcLocalController.cs        // /calc/local
│   ├── CalcCloudController.cs        // /calc/cloud
│   ├── SensitivityController.cs      // /calc/sensitivity
│   ├── CompareController.cs          // /compare
│   └── ReportController.cs           // /report (если используется)
│
├── services/
│   ├── LocalCalcService.cs
│   ├── CloudCalcService.cs
│   ├── SensitivityService.cs
│   ├── CompareService.cs
│   └── ReportService.cs
│
├── models/
│   ├── CalcRequest.cs
│   ├── CalcResponse.cs
│   ├── SensitivityResult.cs
│   └── ParameterSensitivity.cs
│
└── Program.cs
```

При таком разделении можно распределять контроллеры и сервисы по участникам, не мешая друг другу в одном файле.

---

## Эндпоинт `/calc/sensitivity`

### Пример сервиса (C#)

```csharp
public class SensitivityService
{
    private readonly LocalCalcService _local;

    public SensitivityService(LocalCalcService local)
    {
        _local = local;
    }

    public SensitivityResult Analyze(CalcRequest req)
    {
        var results = new List<ParameterSensitivity>();

        void AddParam(string name, double value, Action<double> apply)
        {
            double original = value;

            apply(original * 0.8);
            var low = _local.Calculate(req);

            apply(original * 1.2);
            var high = _local.Calculate(req);

            apply(original);

            results.Add(new ParameterSensitivity(name, original, low, high));
        }

        AddParam("capex",   req.Capex,                        v => req.Capex = v);
        AddParam("opex",    req.Opex,                         v => req.Opex = v);
        AddParam("savings", req.ExpectedSavingsPerYear,       v => req.ExpectedSavingsPerYear = v);
        AddParam("revenue", req.ExpectedRevenueGrowthPerYear, v => req.ExpectedRevenueGrowthPerYear = v);

        return new SensitivityResult(results);
    }
}
```

### Пример контроллера (C#)

```csharp
[ApiController]
[Route("calc")]
public class SensitivityController : ControllerBase
{
    private readonly SensitivityService _sensitivity;

    public SensitivityController(SensitivityService sensitivity)
    {
        _sensitivity = sensitivity;
    }

    [HttpPost("sensitivity")]
    public IActionResult Sensitivity(CalcRequest req)
    {
        var result = _sensitivity.Analyze(req);
        return Ok(result);
    }
}
```

При необходимости можно добавить фильтрацию параметров (например, анализировать только те, что явно указаны в запросе).

---

## Генерация HTML/PDF-отчёта

### Содержание отчёта

Отчёт должен содержать:

- название проекта и клиента;
- блок исходных данных (CAPEX, OPEX, подписка, период, эффекты);
- результаты расчётов:
  - TCO (local и cloud),
  - ROI,
  - срок окупаемости;
- таблицу сценариев (например, базовый, оптимистичный, пессимистичный);
- результаты анализа чувствительности;
- краткие выводы.

Рекомендуемый файл шаблона: `docs/report-template.html`.

### Пример HTML-шаблона

```html
<html>
<head>
    <meta charset="utf-8"/>
    <title>InvestCalc Report</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; }
        h1 { color: #1b4e9b; }
        .table { border-collapse: collapse; width: 100%; margin-top: 20px; }
        .table th, .table td { border: 1px solid #ccc; padding: 8px; }
        .section { margin-top: 30px; }
    </style>
</head>
<body>

<h1>Отчёт по проекту "{{project_name}}"</h1>

<div class="section">
    <h2>Исходные данные</h2>
    {{input_table}}
</div>

<div class="section">
    <h2>Результаты расчётов</h2>
    {{calc_table}}
</div>

<div class="section">
    <h2>Анализ сценариев</h2>
    {{scenarios_table}}
</div>

<div class="section">
    <h2>Анализ чувствительности</h2>
    {{sensitivity_table}}
</div>

<div class="section">
    <h2>Выводы</h2>
    <p>{{conclusion}}</p>
</div>

</body>
</html>
```

### Генерация PDF (опционально)

Возможные варианты:

- Python: WeasyPrint;
- Node.js: Puppeteer;
- C#: DinkToPdf, QuestPDF и аналогичные библиотеки.

Пример (Node.js + Puppeteer):

```js
await page.pdf({ path: "report.pdf", format: "A4" });
```

PDF не является обязательным требованием, но может быть выполнен как дополнительный бонус.

---

## Docker: создание контейнера

### Dockerfile (ASP.NET Core, .NET 9)

```dockerfile
# Сборка (build)
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /app
COPY . .
RUN dotnet restore
RUN dotnet publish -c Release -o out

# Среда выполнения (runtime)
FROM mcr.microsoft.com/dotnet/aspnet:9.0
WORKDIR /app
COPY --from=build /app/out .
EXPOSE 8080
ENTRYPOINT ["dotnet", "InvestCalc.Api.dll"]
```

### Docker Compose (при необходимости)

```yaml
services:
  investcalc:
    build: .
    ports:
      - "8080:8080"
    container_name: investcalc-api
```

При использовании БД сюда можно добавить отдельный сервис (например, PostgreSQL).

### Запуск контейнера

Вариант без compose:

```bash
docker build -t investcalc .
docker run -p 8080:8080 investcalc
```

Вариант с docker-compose:

```bash
docker compose up --build
```

---

## Тестирование контейнера

После запуска контейнера необходимо проверить:

- что сервис стартовал без ошибок;
- что API отвечает корректно.

Пример проверки:

```bash
curl -X POST http://localhost:8080/calc/local   -H "Content-Type: application/json"   -d @data/input-local.json
```

Проверяется:

- успешный HTTP-статус;
- корректность JSON-ответа;
- возможность вызываемых эндпоинтов, включая `/calc/sensitivity`.

---

## Работа с GitHub на третьей неделе

### Новые ветки

Рекомендуется разделить работу по функциональным веткам:

```bash
git checkout -b feature/sensitivity
git checkout -b feature/report
git checkout -b feature/docker
```

Фактически ветки создаются последовательно, с переключением между ними, а не одной командой подряд, но в методичке фиксируются именно такие ветви.

### Примеры коммитов

```bash
git add src/services/SensitivityService.cs src/api/SensitivityController.cs
git commit -m "feat: added sensitivity analysis endpoint"

git add docs/report-template.html src/services/ReportService.cs
git commit -m "feat: added HTML report generator"

git add docker/Dockerfile docker/docker-compose.yml
git commit -m "feat: added Docker container configuration"
```

### Pull Request в `develop`

Создаются отдельные PR:

- по анализу чувствительности;
- по генерации отчёта;
- по Docker-контейнеризации.

Каждый Pull Request должен содержать:

- краткое описание изменений;
- ссылку на соответствующий issue;
- при необходимости — скриншоты (например, отчёта или успешного вызова API);
- примеры запросов/ответов.

---

## Обновление README

В README необходимо добавить разделы:

- как запустить проект локально (без Docker);
- API-эндпоинты:
  - `POST /calc/local`;
  - `POST /calc/cloud`;
  - `POST /calc/sensitivity`;
  - `POST /compare`;
  - при наличии — `POST /report`;
- примеры запросов и ответов (возможно, с ссылкой на `docs/examples/`);
- как собрать и запустить Docker-контейнер:
  - команды `docker build`, `docker run`;
  - при наличии — инструкции по `docker compose`;
- скриншоты отчёта (`docs/screens/report.png`);
- краткое описание анализа чувствительности.

---

## Демонстрация на занятии

К концу третьей недели команда должна продемонстрировать:

- рабочий эндпоинт `/calc/sensitivity` с примером запроса и ответа;
- HTML-отчёт (открытый в браузере или сохранённый файл);
- Docker-контейнер, который запускается одной командой и обслуживает API;
- набор коммитов и Pull Request в GitHub;
- обновлённый README с инструкциями по запуску и описанием новых функций.

---

## Итоговые артефакты третьей недели

| Результат               | Файл/артефакт                                                     |
|-------------------------|-------------------------------------------------------------------|
| Анализ чувствительности | `src/services/SensitivityService.cs`                              |
| Эндпоинт API            | `src/api/SensitivityController.cs` (или метод `/calc/sensitivity`) |
| HTML-шаблон отчёта      | `docs/report-template.html`                                       |
| Генерация отчёта        | `src/services/ReportService.cs`                                   |
| Dockerfile              | `docker/Dockerfile`                                               |
| Docker Compose          | `docker/docker-compose.yml`                                       |
| Скриншот отчёта         | `docs/screens/report.png`                                         |
| PR и коммиты            | репозиторий GitHub                                                |

---

## Распределение объёма работы между участниками (2–4 человека)

Чтобы каждый участник команды выполнил сопоставимый объём кода, функционал третьей недели можно разделить на 12 условных «единиц работы» (атомарных задач). Это позволит честно распределить нагрузку при любой численности команды от 2 до 4 человек.

### Единицы работы

**Блок «Анализ чувствительности»**

1. Реализация моделей `SensitivityResult` и `ParameterSensitivity` в проекте (`models/`).
2. Реализация метода `Analyze` в `SensitivityService` (расчёт вариаций ±20 %).
3. Реализация эндпоинта `/calc/sensitivity` в контроллере (`SensitivityController` или аналог).
4. Подготовка документации и примеров JSON-запросов/ответов для анализа чувствительности (`docs/sensitivity.md`, примеры в `docs/examples/`).

**Блок «Генерация отчёта»**

5. Разработка HTML-шаблона отчёта `docs/report-template.html` (структура, стили).
6. Реализация `ReportService` — заполнение шаблона расчётными данными и чувствительностью.
7. Реализация вызова генерации отчёта: отдельный API-метод (`/report`) или консольный сценарий (например, `ReportGenerator`).
8. Подготовка тестового примера отчёта (сгенерированный HTML + скриншот `docs/screens/report.png` и краткое описание в документации).

**Блок «Docker-контейнеризация»**

9. Создание `Dockerfile` (multi-stage сборка, экспонирование порта, `ENTRYPOINT`).
10. Создание `docker-compose.yml` (если требуется, добавление БД или других сервисов).
11. Подготовка вспомогательного скрипта/описания для запуска и проверки (например, `docker/README-docker.md` с командами `docker build`, `docker run`, `docker compose up` и проверкой API).
12. Оформление раздела README «Запуск в Docker» с командами и описанием типичного сценария запуска.

### Распределение по размеру команды

**Команда из 2 человек**

- Всего задач: 12.  
- Каждый участник выполняет по 6 задач.

Пример:

- Участник A: задачи 1, 3, 5, 7, 9, 11.  
- Участник B: задачи 2, 4, 6, 8, 10, 12.

**Команда из 3 человек**

- Всего задач: 12.  
- Каждый участник выполняет по 4 задачи.

Пример:

- Участник A: задачи 1, 4, 7, 10.  
- Участник B: задачи 2, 5, 8, 11.  
- Участник C: задачи 3, 6, 9, 12.

**Команда из 4 человек**

- Всего задач: 12.  
- Каждый участник выполняет по 3 задачи.

Пример:

- Участник A: задачи 1, 5, 9.  
- Участник B: задачи 2, 6, 10.  
- Участник C: задачи 3, 7, 11.  
- Участник D: задачи 4, 8, 12.

Важно:

- каждый участник должен иметь в своей зоне ответственности **код** (класс сервиса, контроллер, шаблон или конфигурация Docker), а не только документацию;
- все участники фиксируют свои изменения в виде отдельных коммитов и Pull Request, чтобы можно было отследить вклад каждого.
