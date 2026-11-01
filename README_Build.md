# Strata-ORCA + Qwen3.8-Flash-Next IQ3_XXS на Windows 10

Практическая инструкция по рабочей конфигурации Strata-ORCA на Windows
10 с NVIDIA RTX 3090, CUDA 12.8, Expert Pack и отдельным Vision Engine.

> **Статус документа.** README составлен по заметкам с реальными
> командами и результатами тестирования. Команды подготовки IQ Pack,
> сборки Vision, проверки и запуска сервера восстановлены из заметок.
> Точную финальную команду CMake-сборки основного `strata.exe` в
> заметках найти не удалось, поэтому она не заменена догадкой. См.
> раздел «Сборка основного движка».

## 1. Рабочая конфигурация

-   ОС: Windows 10 x64
-   GPU: NVIDIA GeForce RTX 3090, 24 ГБ VRAM
-   RAM: 64 ГБ
-   CUDA Toolkit: 12.8
-   Visual Studio Build Tools: 2022
-   MSVC: 14.37.32822
-   CMake: 3.26.4-msvc4
-   Ninja: 1.11.0
-   Python: 3.12
-   Основной движок: `X:\Strata-ORCA\build128\strata.exe`
-   Vision Engine:
    `X:\Strata-ORCA\build-vision-cuda128\bin\strata-vision.exe`
-   API: `http://127.0.0.1:8080/v1`

В заметках исходники Strata-ORCA находятся в `X:\Strata-ORCA`, а модель
и mmproj --- в `X:\llama.cpp\models`.

## 2. Структура каталогов

``` text
X:\
├── Strata-ORCA\
│   ├── .venv\
│   ├── build128\
│   │   ├── strata.exe
│   │   └── _deps\
│   │       └── strata_llamacpp-src\
│   ├── build-vision-cuda128\
│   │   └── bin\
│   │       └── strata-vision.exe
│   ├── data\
│   │   └── expert-profile.bin
│   ├── packed\
│   │   └── orca-iq3_xxs\
│   │       ├── index.txt
│   │       ├── native_experts.txt
│   │       └── tokenizer\
│   ├── third_party\
│   │   └── llama.cpp\
│   │       └── gguf-py\
│   ├── tools\
│   │   ├── iq_pack.py
│   │   └── vision\
│   ├── serve\
│   │   └── server.py
│   ├── orca-vision-cache1024-test.json
│   └── strata-iq3_xxs.log
│
└── llama.cpp\
    └── models\
        ├── IQ3_XXS_Ablit\
        │   ├── Qwen3.8-Flash-Next-Uncensored-IQ3_XXS-00001-of-00002.gguf
        │   └── Qwen3.8-Flash-Next-Uncensored-IQ3_XXS-00002-of-00002.gguf
        └── mmproj-Qwen3.8-Flash-Next-BF16.gguf
```

Пути к модели в JSON должны соответствовать реальному расположению
файлов. Если модель хранится в другом каталоге, замените путь во всех
соответствующих местах конфигурации.

## 3. Установка и проверка инструментов

Нужны Git, Python 3.12, Visual Studio Build Tools 2022 с MSVC/C++ build
tools, CMake, Ninja, NVIDIA CUDA Toolkit 12.8 и совместимый драйвер
NVIDIA.

Проверка:

``` powershell
git --version
python --version
nvcc --version
cmake --version
ninja --version
nvidia-smi
```

Проверить файлы Visual Studio и CUDA:

``` powershell
Test-Path "C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvars64.bat"
Test-Path "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.8\bin\nvcc.exe"
```

Добавить CUDA 12.8 в `PATH` текущего сеанса PowerShell:

``` powershell
$cuda = "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.8\bin"
$env:PATH = "$cuda;$env:PATH"
```

## 4. Python virtual environment

``` powershell
Set-Location "X:\Strata-ORCA"
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python --version
python -c "import sys; print(sys.executable)"
```

Создавать окружение нужно только если его ещё нет. Для следующих команд
можно не активировать его: используется прямой путь
`X:\Strata-ORCA\.venv\Scripts\python.exe`.

## 5. Проверка основного Strata Engine

Проверить наличие и справку:

``` powershell
$root = "X:\Strata-ORCA"
$env:PATH = "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.8\bin;$env:PATH"

Test-Path "$root\build128\strata.exe"
& "$root\build128\strata.exe" --help
```

### Сборка основного движка

В рабочей конфигурации основной движок уже собран:

``` text
X:\Strata-ORCA\build128\strata.exe
```

В предоставленных заметках нет полной последней команды
CMake-конфигурации и компиляции именно основного `strata.exe`. Поэтому
здесь она намеренно не приводится: подставлять непроверенную команду
вместо фактической было бы рискованно. Для чистой сборки с нуля нужно
сверить актуальные CMake options с инструкциями и исходниками
зафиксированной версии Strata-ORCA.

## 6. Подключение gguf-py и проверка IQ Pack

``` powershell
$root = "X:\Strata-ORCA"
$python = "$root\.venv\Scripts\python.exe"
$ggufPy = "$root\third_party\llama.cpp\gguf-py"

$env:PYTHONPATH = $ggufPy

& $python -c "import sys, gguf; print('Python:', sys.executable); print('gguf:', gguf.__file__)"
& $python "$root\tools\iq_pack.py" --help
```

Если импорт `gguf` не проходит, проверьте, что
`third_party\llama.cpp\gguf-py` существует и содержит пакет `gguf`.
Используется библиотека из llama.cpp, а не случайно установленный
одноимённый пакет.

## 7. Создание IQ Expert Pack

Исходный GGUF:

``` text
X:\llama.cpp\models\IQ3_XXS_Ablit\Qwen3.8-Flash-Next-Uncensored-IQ3_XXS-00001-of-00002.gguf
```

Команда упаковки:

``` powershell
$root = "X:\Strata-ORCA"
$python = "$root\.venv\Scripts\python.exe"
$ggufPy = "$root\third_party\llama.cpp\gguf-py"

$model = "X:\llama.cpp\models\IQ3_XXS_Ablit\Qwen3.8-Flash-Next-Uncensored-IQ3_XXS-00001-of-00002.gguf"
$out = "X:\Strata-ORCA\packed\orca-iq3_xxs"

$env:PYTHONPATH = $ggufPy

& $python "$root\tools\iq_pack.py" --gguf $model --out $out --compat-bf16

Write-Host "`nКод завершения: $LASTEXITCODE"
```

Проверить результат:

``` powershell
$out = "X:\Strata-ORCA\packed\orca-iq3_xxs"

Get-ChildItem $out -Force |
    Select-Object Name,
        @{Name="Size_GB";Expression={
            if (-not $_.PSIsContainer) {
                [math]::Round($_.Length / 1GB, 3)
            } else {
                "<каталог>"
            }
        }}

Write-Host "`n--- Маркер завершения ---"
Test-Path "$out\native_experts.txt"

Write-Host "`n--- Индекс ---"
Get-Content "$out\index.txt" -TotalCount 5

Write-Host "`n--- Параметры модели ---"
Get-Content "$out\native_experts.txt" -TotalCount 4
```

Ожидаемые важные элементы: `index.txt`, `native_experts.txt` и каталог
`tokenizer`. Проверка `Test-Path` для `native_experts.txt` должна
вернуть `True`.

## 8. Проверка Expert Profile

Рабочий профиль:

``` text
X:\Strata-ORCA\data\expert-profile.bin
```

Проверить наличие:

``` powershell
Test-Path "X:\Strata-ORCA\data\expert-profile.bin"
```

Команда для чтения заголовков существующих профилей:

``` powershell
$files = @(
    "X:\Strata-ORCA\data\expert-profile.bin",
    "X:\Strata-ORCA\data\expert-profile-coder.bin",
    "X:\Strata-ORCA\data\expert-profile-sycl.bin"
)

foreach ($file in $files) {
    if (Test-Path $file) {
        $b = [System.IO.File]::ReadAllBytes($file)
        if ($b.Length -ge 24) {
            $magic = [System.Text.Encoding]::ASCII.GetString($b, 0, 4)
            $version, $layers, $experts, $slots, $ranked = [System.BitConverter]::ToUInt32($b, 4), [System.BitConverter]::ToUInt32($b, 8), [System.BitConverter]::ToUInt32($b, 12), [System.BitConverter]::ToUInt32($b, 16), [System.BitConverter]::ToUInt32($b, 20)
            [PSCustomObject]@{
                File    = $file
                Bytes   = $b.Length
                Magic   = $magic
                Version = $version
                Layers  = $layers
                Experts = $experts
                Slots   = $slots
                Ranked  = $ranked
            }
        }
    }
}
```

Эта команда только читает заголовки. Она не создаёт
`expert-profile.bin`. Точная команда генерации профиля в заметках
отсутствует.

## 9. Сборка Strata Vision

В PowerShell выполнить конфигурацию:

``` powershell
$root  = "X:\Strata-ORCA"
$llama = "$root\build128\_deps\strata_llamacpp-src"
$build = "$root\build-vision-cuda128"
$vs    = "C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools"
$cuda  = "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.8"
$cmake = "$vs\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe"
$ninja = "$vs\Common7\IDE\CommonExtensions\Microsoft\CMake\Ninja\ninja.exe"

$env:PATH = "$(Split-Path $ninja);$env:PATH"

$configure = @(
    'call "' + "$vs\VC\Auxiliary\Build\vcvars64.bat" + '"',
    '&& "' + $cmake + '"',
    '-S "' + "$root\tools\vision" + '"',
    '-B "' + $build + '"',
    '-G Ninja',
    '-DCMAKE_BUILD_TYPE=Release',
    '-DLLAMA_DIR="' + $llama + '"',
    '-DSTRATA_VISION_CUDA=ON',
    '-DSTRATA_PORTABLE=ON',
    '-DCMAKE_CUDA_ARCHITECTURES=86',
    '-DCMAKE_CUDA_COMPILER="' + $cuda + '/bin/nvcc.exe"',
    '-DCUDAToolkit_ROOT="' + $cuda + '"'
) -join ' '

cmd.exe /d /s /c $configure

if ($LASTEXITCODE -ne 0) {
    throw "CMake configuration failed with exit code $LASTEXITCODE"
}
```

Сборка:

``` powershell
$root = "X:\Strata-ORCA"
$vs = "C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools"
$cmake = "$vs\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe"

cmd.exe /d /s /c ('call "' + "$vs\VC\Auxiliary\Build\vcvars64.bat" + '" && "' + $cmake + '" --build "' + "$root\build-vision-cuda128" + '" --target strata-vision --config Release --parallel 6')

if ($LASTEXITCODE -ne 0) {
    throw "Vision build failed with exit code $LASTEXITCODE"
}
```

Ожидаемый файл:

``` text
X:\Strata-ORCA\build-vision-cuda128\bin\strata-vision.exe
```

Проверка:

``` powershell
Test-Path "X:\Strata-ORCA\build-vision-cuda128\bin\strata-vision.exe"
```

## 10. Отдельный тест Vision

``` powershell
$cuda = "C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.8"
$exe = "X:\Strata-ORCA\build-vision-cuda128\bin\strata-vision.exe"
$mmproj = "X:\llama.cpp\models\mmproj-Qwen3.8-Flash-Next-BF16.gguf"
$model = "X:\llama.cpp\models\IQ3_XXS_Ablit\Qwen3.8-Flash-Next-Uncensored-IQ3_XXS-00001-of-00002.gguf"

$env:CUDA_PATH = $cuda
$env:PATH = "$cuda\bin;$env:PATH"

& $exe --mmproj $mmproj --model $model --gpu --threads 6 --max-tokens 128
```

В тесте используются Vision Engine, mmproj BF16, первый шард модели,
GPU-режим, 6 CPU-потоков и лимит 128 токенов. В нашей проверке Vision
сообщил `READY 2560`.

## 11. Последняя конфигурация сервера с Vision

Создать файл:

``` text
X:\Strata-ORCA\orca-vision-cache1024-test.json
```

Содержимое:

``` json
{
    "exe": "X:/Strata-ORCA/build128/strata.exe",
    "cwd": "X:/Strata-ORCA",
    "args": [
        "--pack",
        "X:/Strata-ORCA/packed/orca-iq3_xxs",
        "--native",
        "X:/llama.cpp/models/IQ3_XXS_Ablit/Qwen3.8-Flash-Next-Uncensored-IQ3_XXS-00001-of-00002.gguf",
        "--ple-gguf",
        "X:/llama.cpp/models/IQ3_XXS_Ablit/Qwen3.8-Flash-Next-Uncensored-IQ3_XXS-00001-of-00002.gguf",
        "--expert-profile",
        "X:/Strata-ORCA/data/expert-profile.bin",
        "--prefill",
        "auto",
        "--spec",
        "4",
        "--max-context",
        "32768",
        "--max-new",
        "256",
        "--vision",
        "--kv",
        "int8",
        "--vram-reserve-mib",
        "700",
        "--expert-cache-per-layer",
        "--expert-cache",
        "1024",
        "--temperature",
        "0.7"
    ],
    "model_name": "qwen3.8-flash-next-iq3_xxs",
    "tokenizer": "X:/Strata-ORCA/packed/orca-iq3_xxs/tokenizer",
    "port": 8080,
    "gpu": 0,
    "log": "X:/Strata-ORCA/strata-iq3_xxs.log",
    "vision": {
        "exe": "X:/Strata-ORCA/build-vision-cuda128/bin/strata-vision.exe",
        "gpu": true,
        "max_tokens": 1024,
        "min_tokens": 1024,
        "mmproj": "X:/llama.cpp/models/mmproj-Qwen3.8-Flash-Next-BF16.gguf",
        "model": "X:/llama.cpp/models/IQ3_XXS_Ablit/Qwen3.8-Flash-Next-Uncensored-IQ3_XXS-00001-of-00002.gguf"
    },
    "reasoning_effort": "none"
}
```

Это наиболее поздняя конфигурация из заметок. В ней:

-   контекст --- 32768;
-   `--prefill auto`;
-   speculative decoding --- `--spec 4`;
-   KV cache --- `int8`;
-   резерв VRAM --- 700 MiB;
-   Expert Cache --- 1024 с `--expert-cache-per-layer`;
-   Vision включён через `--vision` и секцию `vision`;
-   reasoning --- `none`.

Дублирующийся `--prefill auto` из ранних JSON удалён. Старые параметры
`--expert-cache 64` и `--max-context 4096` не используются в этой
последней конфигурации.

## 12. Запуск сервера

``` powershell
Set-Location "X:\Strata-ORCA"

.\.venv\Scripts\python.exe .\serve\server.py `
    --engine strata `
    --config .\orca-vision-cache1024-test.json `
    --port 8080
```

API:

``` text
http://127.0.0.1:8080/v1
```

Имя модели для клиента:

``` text
qwen3.8-flash-next-iq3_xxs
```

Лог:

``` text
X:\Strata-ORCA\strata-iq3_xxs.log
```

Остановка сервера: `Ctrl+C` в окне PowerShell.

## 13. Подключение клиента

В Continue или другом OpenAI-compatible клиенте:

-   Base URL: `http://127.0.0.1:8080/v1`
-   Model: `qwen3.8-flash-next-iq3_xxs`

В заметках также зафиксированы параметры:

``` json
"reasoning_effort": "none"
```

и

``` json
"chat_template_kwargs": {
    "enable_thinking": false
}
```

`chat_template_kwargs` --- параметр шаблона чата/запроса. Не добавляйте
его в корень JSON-конфигурации сервера без подтверждения, что сервер
ожидает это поле именно там.

## 14. MTP: отличие раннего варианта

В раннем варианте использовался аргумент:

``` text
--mtp X:/StrataModels/mtp/rt
```

В последней конфигурации `orca-vision-cache1024-test.json` этот аргумент
отсутствует. В сравнительном тесте скорость с MTP и без него была
примерно одинаковой --- около 70 токенов/с. Для воспроизведения именно
последней конфигурации из этого README не добавляйте MTP.

## 15. Быстрая диагностика

GPU:

``` powershell
nvidia-smi
```

Основной движок:

``` powershell
& "X:\Strata-ORCA\build128\strata.exe" --help
```

Vision:

``` powershell
Test-Path "X:\Strata-ORCA\build-vision-cuda128\bin\strata-vision.exe"
```

Pack:

``` powershell
Test-Path "X:\Strata-ORCA\packed\orca-iq3_xxs\index.txt"
Test-Path "X:\Strata-ORCA\packed\orca-iq3_xxs\native_experts.txt"
Test-Path "X:\Strata-ORCA\packed\orca-iq3_xxs\tokenizer"
```

Expert Profile:

``` powershell
Test-Path "X:\Strata-ORCA\data\expert-profile.bin"
```

Если не импортируется `gguf`, проверьте `PYTHONPATH` и каталог
`gguf-py`. Если CMake не находит компилятор, проверьте путь к
`vcvars64.bat` и запускайте конфигурацию/сборку Vision через него. При
ошибке отсутствующего файла сначала проверьте пути в JSON.

## 16. Результаты на RTX 3090

В предыдущих тестах наблюдались:

-   загрузка около 49,8 GiB экспертных данных;
-   первичная загрузка примерно 153 секунды;
-   Expert Cache auto: около 8 055--8 406 экспертов и 16,32--17,04 GiB
    VRAM;
-   cache hit около 95,7%;
-   скорость генерации около 70 токенов/с.

Это результаты конкретного компьютера и сборки, а не гарантия для других
систем. Эти показатели относятся к тестам с Expert Cache auto и не
должны автоматически приписываться последней конфигурации с
`--expert-cache-per-layer 1024`.

## 17. Что сохранить

Для воспроизводимости сохраните:

-   исходники Strata-ORCA и точный Git commit;
-   исходники llama.cpp и используемый commit;
-   `build128` и `build-vision-cuda128` либо точные параметры сборки;
-   оба GGUF-шарда;
-   `mmproj-Qwen3.8-Flash-Next-BF16.gguf`;
-   `packed\orca-iq3_xxs`;
-   `data\expert-profile.bin`;
-   `.venv` или список Python-зависимостей;
-   `orca-vision-cache1024-test.json`;
-   лог последнего успешного запуска.

Не обновляйте одновременно Strata, llama.cpp, CUDA и модель. Меняйте по
одному компоненту, чтобы можно было установить причину возможной ошибки.

## 18. Что ещё нужно зафиксировать для полностью чистой установки

В исходных заметках не нашлись:

1.  Полная последняя команда CMake-сборки основного `strata.exe`.
2.  Команда генерации `expert-profile.bin`.
3.  Полный список Python-зависимостей и точная команда их установки.
4.  Точная команда первоначального клонирования/закрепления llama.cpp.

Поэтому этот README точно описывает подготовку IQ Pack, сборку Vision,
проверку артефактов, последнюю конфигурацию и запуск уже собранного
основного движка. Для полностью чистой установки с нуля эти пункты нужно
дополнить по фактическим инструкциям и файлам репозитория Strata-ORCA,
не подменяя их предположениями.

«Если это читают потомки — CUDA 12.8 тогда тоже была капризной. Держитесь!»
