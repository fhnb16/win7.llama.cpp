# llama.cpp для Windows 7

Репозиторий автоматически собирает актуальный стабильный релиз [llama.cpp](https://github.com/ggml-org/llama.cpp) для Windows 7 x64 и публикует отдельный portable ZIP в Releases этого репозитория.

Исходный код llama.cpp не хранится здесь: GitHub Actions получает его напрямую из upstream-репозитория по тегу последнего релиза.

## Как получить сборку

Скачайте CPU-архив `llama.cpp-<версия>-win7-x64.zip` из [Releases](https://github.com/fhnb16/win7.llama.cpp/releases) и распакуйте его. Если дополнительная сборка Vulkan завершилась успешно, в том же релизе будет архив `llama.cpp-<версия>-win7-x64-vulkan.zip`. Модели GGUF в архивы не входят — их нужно загрузить отдельно.

Для запуска на Windows 7 добавьте параметр `--no-mmap`:

```powershell
llama-cli.exe -m model.gguf --no-mmap -p "Hello"
llama-server.exe -m model.gguf --no-mmap
```

Это CPU-сборка для x64 на базе MinGW-w64/w64devkit с целевой версией Windows 7. Она не включает CUDA или другие GPU-бэкенды.

Vulkan-вариант собирается отдельно и публикуется только при успешных установке SDK, сборке и проверке Win7 API. Для его запуска нужны совместимые с Windows 7 Vulkan-драйвер и runtime; они не входят в архив. Используйте `-ngl 99` для выгрузки слоёв на GPU. Если Vulkan-сборка не удалась, CPU-сборка всё равно публикуется.

## Автоматическая сборка

Workflow [Windows 7 release](.github/workflows/windows-7-release.yml):

- ежедневно в 06:17 UTC проверяет последний стабильный релиз llama.cpp;
- запускается также при push в ветку `master`;
- собирает upstream-исходники с `WINVER` и `_WIN32_WINNT`, установленными в `0x0601`;
- перед сборкой применяет проверяемый Win7-патч к vendored `cpp-httplib`: убирает ограничение библиотеки на Windows 10 и заменяет Win8+ API отображения файлов на `CreateFileW`, `CreateFileMappingW` и `MapViewOfFile`;
- проверяет исполняемые файлы на импорты ряда WinAPI, недоступных в Windows 7;
- всегда собирает CPU-версию и создаёт релиз с тем же тегом, что у upstream;
- дополнительно пытается собрать Vulkan-версию и прикладывает её отдельный portable ZIP только при успехе; сбой Vulkan не блокирует CPU-релиз;
- пропускает сборку, если релиз с таким тегом уже опубликован.

Перед сборкой workflow проверяет, опубликован ли уже релиз с upstream-тегом, и пропускает его, если он существует. Поэтому push запускает проверку, но не пересобирает и не перезаписывает уже собранную версию. Сборку также можно запустить вручную через **Actions → Windows 7 release → Run workflow**. Для публикации Actions требуется разрешение `contents: write`, заданное в workflow.

Совместимость зависит от изменений в upstream llama.cpp и его зависимостях. Обсуждение причин несовместимости официальных бинарников и обходных путей: [llama.cpp discussion #28171](https://github.com/ggml-org/llama.cpp/discussions/28171); связанный PR по Windows 7: [llama.cpp PR #8208](https://github.com/ggml-org/llama.cpp/pull/8208).

В актуальном upstream `cpp-httplib` явно отказался от Windows 8 и ниже. Workflow не подменяет библиотеку другой реализацией: он точечно патчит её WinAPI-вызовы, используя совместимый подход из [cpp-httplib PR #1869](https://github.com/yhirose/cpp-httplib/pull/1869). Если upstream изменит ожидаемые фрагменты исходников, патч завершится ошибкой вместо продолжения неподтверждённой сборки.
