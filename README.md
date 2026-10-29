# StudyLoop — Frontend

Сайт с AI-тьютором по программированию, который подстраивает обучение под знания и прогресс ученика.
MVP охватывает основы Python: переменные, условия и циклы.

## Стек

- **React 18** + **Vite** — интерфейс и сборка
- **React Router** — маршрутизация
- **CodeMirror 6** — редактор кода с подсветкой Python
- **lucide-react** — иконки
- **Pyodide** (в Web Worker) — запуск Python-кода ученика прямо в браузере, с таймаутом против бесконечных циклов

## Запуск

Нужен [Node.js](https://nodejs.org) 18+.

```bash
npm install
npm run dev
```

Откроется на http://localhost:5173. Сборка для продакшена: `npm run build`.

## Что реализовано

| Экран | Путь | Что делает |
|---|---|---|
| Регистрация | `/register` | Имя, email, пароль, согласие; кнопки Google / Apple (заработают с бэкендом) |
| Вход | `/login` | «Запомнить меня», ссылка на восстановление пароля |
| Восстановление пароля | `/forgot` | Запрос ссылки на сброс |
| Входной тест | `/diagnostic` | Вступление → 15 вопросов (выбор ответа и короткий ответ) |
| Результаты теста | `/diagnostic/result` | Уровень, доля верных ответов, баллы по темам, разбор ошибок |
| Главная | `/dashboard` | Уровень, ближайшие шаги, прогресс, достижения |
| Мой план | `/plan` | Как построен план (тест → пробелы → маршрут) и темы с задачами |
| Курсы | `/courses` | Курс «Python. Основы» и будущие курсы |
| Тесты | `/tests` | Результат входного теста, повторное прохождение |
| Прогресс | `/progress` | Статистика, древо навыков, активность, достижения |
| Урок | `/lesson/:topicId` | Теория с примерами кода |
| Задача | `/task/:taskId` | **Split-Screen, Focus Mode**: чат с AI-ментором + редактор + тесты |
| База знаний | `/knowledge` | Карточки-резюме исправленных ошибок |
| Настройки | `/settings` | Профиль, сброс прогресса, выход |

### Уровни
По доле верных ответов во входном тесте: до 75% — **Beginner**, 75–89% — **Elementary**, от 90% — **Early Intermediate**.

### Режим ментора
Ментор не выдаёт готовый код. Он подсвечивает проблемную строку и задаёт наводящие вопросы.
«Умные кнопки» зависят от состояния задачи: «Где ошибка?», «Подсказка», «Объяснить логику построчно»,
«Написать тесты к этому коду», «Оптимизировать функцию».
Когда задача решена после ошибки, создаётся карточка: почему ошибка возникла и как исправлена.

### Адаптация
- Сильная тема во входном тесте (≥ 67%) — сразу к практике, слабая — сначала урок.
- 3+ неудачные попытки — платформа предлагает задачу попроще или повторить урок.
- Тема освоена — открывается следующая в древе навыков.

## Подключение Django-бэкенда

Сейчас фронтенд работает в **демо-режиме**: данные хранятся в `localStorage` (`src/api/mock.js`),
ответы ментора генерируются правилами (`src/lib/mentor.js`).

Чтобы подключить бэкенд, создайте `.env` с адресом API:

```
VITE_API_URL=http://localhost:8000/api
```

Ожидаемые эндпоинты (`src/api/index.js`), авторизация — `Authorization: Bearer <access>`:

| Метод | Путь | Тело | Ответ |
|---|---|---|---|
| POST | `/auth/register/` | `{name, email, password, role}` | `{access, user}` |
| POST | `/auth/login/` | `{email, password}` | `{access, user}` |
| GET | `/auth/me/` | — | `user` |
| PATCH | `/auth/me/` | `{name}` | `user` |
| POST | `/auth/password-reset/` | `{email}` | 200 |
| GET | `/auth/oauth/google/`, `/auth/oauth/apple/` | — | редирект на провайдера |
| GET | `/progress/` | — | `Progress` |
| DELETE | `/progress/` | — | 204 (сброс) |
| GET | `/diagnostic/questions/` | — | вопросы без правильных ответов |
| POST | `/diagnostic/submit/` | `{answers: {questionId: optionIndex}}` | `Progress` |
| POST | `/tasks/:id/attempts/` | `{passed, error}` | `Progress` |
| POST | `/knowledge/` | карточка | `Progress` |
| DELETE | `/knowledge/:id/` | — | 204 |
| POST | `/mentor/ask/` | `{task, code, run, message, action, hintLevel}` | `{reply}` |

`Progress` = `{ diagnostic: {scores, review, correct, total, pct, level, takenAt} | null, tasks: {[taskId]: {solved, attempts, errors, solvedAt}}, knowledge: Card[], activity: string[] }`

Ошибки — `{detail: "текст"}` с соответствующим HTTP-статусом.

## Структура

```
src/
  api/          index.js — выбор mock/HTTP; mock.js — localStorage-бэкенд
  components/   Layout (боковое меню), Logo, ProtectedRoute, SkillTree, TopicIcon, RichText
  context/      AuthContext, ProgressContext
  data/         curriculum.js — темы, уроки, задачи; diagnostic.js — входной тест и уровни
  lib/          mentor.js, plan.js, achievements.js, pythonRunner.js
  pages/        Auth, Diagnostic, Dashboard, Plan, Courses, Tests, Progress, Lesson, Workspace, Knowledge, Settings
public/
  pyodide-worker.js — запуск Python в Web Worker
```
