# adc-logger

Конфигуратор стандартного Python `logging` с поддержкой JSON-формата и цветного вывода. Не заменяет stdlib logging, а упрощает его настройку через композируемые объекты вместо словарей.

## Установка

```bash
pip install git+https://github.com/ascet-dev/adc-logger.git@main
```

## Быстрый старт

```python
from adc_logger import BaseLoggingConfig
from adc_logger.configs import LoggerConfig
import logging

config = BaseLoggingConfig()
config.loggers.append(
    LoggerConfig(name="my_app", level="INFO", handlers=["console_json"])
)
config.setup_logging()

logger = logging.getLogger("my_app")
logger.info("Application started")
```

Вывод (console_json):
```json
{"timestamp": "2025-01-15 12:00:00 +0300", "level": "INFO", "message": "Application started", "logger": "my_app"}
```

## API

### BaseLoggingConfig

Главный класс. Собирает конфигурацию и применяет через `logging.config.dictConfig`.

```python
from adc_logger import BaseLoggingConfig

config = BaseLoggingConfig()
config.setup_logging()        # применить конфигурацию
config.get_logging_config()   # получить dict для dictConfig (без применения)
```

Атрибуты (можно переопределять):

| Атрибут | Тип | По умолчанию |
|---|---|---|
| `formatters` | `list[FormatterConfig]` | 3 встроенных форматтера |
| `handlers` | `list[HandlerConfig]` | 3 встроенных хендлера |
| `loggers` | `list[LoggerConfig]` | `[]` |
| `disable_existing_logger` | `bool` | `False` |

### Встроенные форматтеры и хендлеры

| Форматтер | Хендлер | Описание |
|---|---|---|
| `json` | `console_json` | Структурированный JSON вывод |
| `generic` | `console_generic` | Цветной вывод с уровнем, именем логгера и сообщением |
| `access` | `console_access` | Упрощенный формат для access-логов |

### FormatterConfig

```python
from adc_logger.configs import FormatterConfig

FormatterConfig(
    name="custom",                              # имя для ссылки из хендлера
    format="{asctime} - {name} - {message}",    # шаблон (опционально)
    style="{",                                  # стиль форматирования: '{', '%', '$'
    datefmt="%Y-%m-%d %H:%M:%S %z",            # формат даты
    formatter_type=None,                        # кастомный класс форматтера
)
```

### HandlerConfig

```python
from adc_logger.configs import HandlerConfig
import logging

HandlerConfig(
    name="file_json",                     # имя хендлера
    formatter="json",                     # имя форматтера
    level="DEBUG",                        # уровень логирования
    class_=logging.FileHandler,           # класс хендлера
    filename="app.log",                   # для FileHandler
)
```

### LoggerConfig

```python
from adc_logger.configs import LoggerConfig

LoggerConfig(
    name="my_app",                        # имя логгера
    level="INFO",                         # уровень
    handlers=["console_json", "file_json"],  # список хендлеров
    propagate=True,                       # пробрасывать ли в родительский логгер
)
```

## Примеры

### JSON-логирование в файл и консоль

```python
from adc_logger import BaseLoggingConfig
from adc_logger.configs import HandlerConfig, LoggerConfig
import logging

config = BaseLoggingConfig()

# Добавляем файловый хендлер
config.handlers.append(
    HandlerConfig(
        name="file_json",
        formatter="json",
        class_=logging.FileHandler,
        filename="app.log",
    )
)

# Логгер пишет и в консоль, и в файл
config.loggers.append(
    LoggerConfig(
        name="api",
        level="INFO",
        handlers=["console_generic", "file_json"],
    )
)

config.setup_logging()
logger = logging.getLogger("api")
logger.info("Request processed")
```

### Кастомный форматтер

```python
from adc_logger import BaseLoggingConfig
from adc_logger.configs import FormatterConfig, HandlerConfig, LoggerConfig

config = BaseLoggingConfig()

config.formatters.append(
    FormatterConfig(
        name="brief",
        format="{levelname}: {message}",
    )
)

config.handlers.append(
    HandlerConfig(name="console_brief", formatter="brief")
)

config.loggers.append(
    LoggerConfig(name="worker", handlers=["console_brief"])
)

config.setup_logging()
```

## Требования

- Python >= 3.8
- colorlog >= 6.7.0

## Лицензия

MIT
