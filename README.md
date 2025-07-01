# 📝 Контракт SSlaves: CompetitionService, CompetitionEngine

## Описание

**CompetitionService** отвечает за:

* создание и настройку соревнований,
* регистрацию игроков и команд,
* управление заявками и квотами,
* отправку уведомлений участникам.
  Общается с другими сервисами по REST/JSON (возможна интеграция через gRPC в будущем).

## Модели данных (Database Schema Proposal)

`User`

Хранится централизованно. Используется всеми сервисами.

```text
id: UUID
name: string
surname: string
email: string
hashed_password: string
is_admin: boolean
date_of_birth: date
age: int
sex: enum('male', 'female', 'other')
weight: float
height: float
mmr: float
created_at: timestamp
bio: string?              -- краткое описание игрока
is_organizer: boolean     -- может ли создавать турниры
avatar_url: string?       -- опциональный аватар
```

`Team`

```text
id: UUID
name: string
created_at: timestamp
owner_id: UUID (FK → User.id)

```

`TeamMembership`

```text
team_id: UUID (FK → Team.id)
user_id: UUID (FK → User.id)
role: enum('captain', 'player')
joined_at: timestamp
PRIMARY KEY(team_id, user_id)
```

`Tournament`

```text
id: UUID
title: string
description: string
sport: string
type_group: enum('olympic', 'swiss', 'round_robin')
start_time: timestamp
created_at: timestamp
entry_cost: decimal
is_team_based: boolean
max_participants: int
organizer_id: UUID (FK → User.id)
```

`TournamentRegistration`

```text
tournament_id: UUID
participant_id: UUID     -- может быть user_id или team_id
participant_type: enum('user', 'team')
registered_at: timestamp
status: enum('pending', 'accepted', 'rejected')
PRIMARY KEY(tournament_id, participant_id)

```

# API (REST JSON или решим потом, может gRPC)

## Tournament CRUD

| Метод    | Endpoint            | Описание                       |
|----------|---------------------|--------------------------------|
| `POST`   | `/tournaments`      | Создание турнира               |
| `GET`    | `/tournaments`      | Получение списка всех турниров |
| `GET`    | `/tournaments/{id}` | Получение информации о турнире |
| `PUT`    | `/tournaments/{id}` | Обновление турнира             |
| `DELETE` | `/tournaments/{id}` | Удаление турнира               |

## Team CRUD

| Метод    | Endpoint      | Описание                       |
|----------|---------------|--------------------------------|
| `POST`   | `/teams`      | Создание команды               |
| `GET`    | `/teams`      | Получение списка команд        |
| `GET`    | `/teams/{id}` | Получение информации о команде |
| `PUT`    | `/teams/{id}` | Обновление команды             |
| `DELETE` | `/teams/{id}` | Удаление команды               |

## Участие в турнирах

| Метод    | Endpoint                               | Описание                   |
|----------|----------------------------------------|----------------------------|
| `POST`   | `/tournaments/{id}/register`           | Регистрация команды/игрока |
| `GET`    | `/tournaments/{id}/participants`       | Получить список участников |
| `PUT`    | `/tournaments/{id}/participants/{pid}` | Обновить статус заявки     |
| `DELETE` | `/tournaments/{id}/participants/{pid}` | Удалить участника          |

## Интеграция с CompetitionEngine

| Обмен                   | Формат | Endpoint (примерный)                      | Описание                                  |
|-------------------------|--------|-------------------------------------------|-------------------------------------------|
| Отправка данных турнира | JSON   | `POST /engine/init`                       | Отправка турнира для генерации расписания |
| Получение расписания    | JSON   | `GET /engine/schedule?tournament_id={id}` | Получить список матчей                    |
| Отправка результата     | JSON   | `POST /engine/match-result`               | CompetitionEngine сообщает результаты игр |

## Ответственности сервиса

| Компонент          | Ответственность                                                                            |
|--------------------|--------------------------------------------------------------------------------------------|
| CompetitionService | CRUD турниров и команд, регистрация участников, управление квотами и заявками, уведомления |
| CompetitionEngine  | Расписание матчей, логика турнирных систем, подсчет итогов                                 |

## Открытые вопросы

* Как именно идентифицировать участника, user_id и team_id в одной таблице или двух?

* Как будет работать авторизация, сначала видимо просто чтобы можно было зайти в аккаунт?

* Нужна ли в mvp интеграция с платежными системами (если entry_cost > 0)?

* Как будем уведомлять, через email, через SMTP, Firebase, или просто в базе хранить _уведомления_?

* Ещё по сути надо добавить таблицу для сохранения итоговых результатов, тоже подумать об этом.

## Devops / инфраструктура

* Docker based запуск всех сервисов?

* База данных PostgreSQL (одна или по сервисам)?

* REST API между сервисами?

* Как мы пишем доку, будет ли OpenAPI/swagger.json

