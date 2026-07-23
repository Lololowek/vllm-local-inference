# vllm-local-inference

Развёртывание Qwen2.5-3B на потребительской GPU с 6 GB VRAM через vLLM.

Задача проекта — уложить 3B-модель в ограничения ноутбучной RTX 3060,
изучть какие параметры инференса чем ограничены.

## Стек

- **vLLM 0.6.3** — inference-движок, OpenAI-совместимый API
- **Qwen2.5-3B-Instruct-AWQ** — INT4-квантизация
- **Docker Compose** — воспроизводимый запуск
- Окружение: WSL2 (Ubuntu 22.04), NVIDIA Container Toolkit

## Запуск

```bash
cp .env.example .env
docker compose up -d
```

Проверка:

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"Qwen/Qwen2.5-3B-Instruct-AWQ",
       "messages":[{"role":"user","content":"Привет"}],
       "max_tokens":100}'
```

## Конфигурация

| Параметр | Значение | Почему |
|---|---|---|
| Модель | Qwen2.5-3B **AWQ** | fp16 требует 6.2 GB — не влезает в 6.0 |
| `max-model-len` | 4096 | KV-кэш 36 KB/токен → 144 MB на запрос |
| `gpu-memory-utilization` | 0.80 | считается от полной VRAM; ~1 GB занимает дисплей |
| `max-num-seqs` | 8 | при 4096 контекста в ~2.3 GB помещается ~16 слотов |
| `enforce-eager` | вкл | CUDA-графы падают в WSL2 (UVA недоступна) |

Расчёты — в [docs/vram-math.md](docs/vram-math.md).

## Результаты

- Скорость генерации: **~40 tok/s** (single-stream, eager, RTX 3060 Laptop 85W)
- Веса в VRAM: ~2.2 GB
- Контекст: 4096 токенов

