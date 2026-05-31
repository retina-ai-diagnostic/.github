# MedAI Diagnostics

Веб-приложение для помощи врачу в анализе снимков глазного дна. Система принимает fundus-снимок, запускает ML-модели, показывает вероятности заболеваний, визуальные подсказки и позволяет врачу сохранить решение по исследованию.

Проект состоит из трех репозиториев:

- `frontend` - интерфейс врача на Next.js.
- `backend-main` - основной API на NestJS, PostgreSQL и TypeORM.
- `backend-ai-service` - ML-сервис на FastAPI и PyTorch.

## Что умеет система

- регистрация и вход врача;
- загрузка снимка глазного дна;
- карточки пациентов и исследований;
- классификация заболеваний по классам ODIR;
- Grad-CAM карта внимания;
- YOLO-детекция локальных признаков диабетической ретинопатии;
- просмотр снимка с масштабированием;
- сравнение двух исследований пациента;
- врачебное решение по результату ИИ;
- PDF-отчет по исследованию.

Результат ИИ является подсказкой и должен проверяться врачом.

## Архитектура

```text
frontend  ->  backend-main  ->  PostgreSQL
                       |
                       v
              backend-ai-service
                       |
                       v
                  ML models
```

## ML-модели

Веса моделей нужно положить в:

```text
backend-ai-service/models/
```

Ожидаемые файлы:

```text
efficientnet_b3_best.pt
convnext_tiny_best.pt
swin_tiny_best.pt
yolo_dr_best.pt
```

YOLO-модель необязательна для классификации. Если `yolo_dr_best.pt` отсутствует, классификация продолжит работать, но локальные признаки отображаться не будут.

Файлы весов не коммитятся в git.

## Запуск

Рекомендуемая структура папок:

```text
project/
  frontend/
  backend-main/
  backend-ai-service/
```

### Backend + база + ML-сервис

```bash
cd backend-main
cp .env.example .env
docker-compose up --build
```

После запуска:

- API: `http://localhost:3000/api/v1`
- Swagger: `http://localhost:3000/api/docs`
- FastAPI docs: `http://localhost:8000/docs`
- PostgreSQL: `127.0.0.1:55432`

### Frontend

```bash
cd frontend
npm install
npm run dev
```

Frontend будет доступен по адресу:

```text
http://localhost:3001
```

## Основные endpoints

Backend:

```text
POST /api/v1/auth/register
POST /api/v1/auth/login
GET  /api/v1/auth/me
POST /api/v1/auth/logout

GET  /api/v1/studies
POST /api/v1/studies
GET  /api/v1/studies/:id
GET  /api/v1/studies/:id/report.pdf

GET  /api/v1/patients
GET  /api/v1/patients/:id
```

ML-сервис:

```text
POST /predict
GET  /health
```

## Безопасность

В проекте используются:

- HttpOnly cookies для access/refresh токенов;
- hash refresh token в базе;
- middleware-защита frontend-маршрутов;
- JWT guards на backend;
- rate limiting;
- ограничение CORS;
- валидация загружаемых файлов;
- исключение `.env`, uploads и model weights из git.

## Проверки

Frontend:

```bash
cd frontend
npx tsc --noEmit --incremental false
npx eslint . --ext .js,.jsx,.ts,.tsx --no-cache
```

Backend:

```bash
cd backend-main
npm run build
```

AI service:

```bash
cd backend-ai-service
python -m compileall app
```
