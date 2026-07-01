# Re∫λala — Intelligent Math Solver & Data Collector
![Java](https://img.shields.io/badge/Java-25-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-4.x-6DB33F?style=for-the-badge&logo=spring&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Kotlin](https://img.shields.io/badge/Kotlin-Android-7F52FF?style=for-the-badge&logo=kotlin&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)

**Re∫λala** — это отказоустойчивая распределенная система и нативное Android-приложение для автоматического распознавания математических выражений по фотографиям и генерации их пошаговых решений. 

Проект решает проблему «галлюцинаций» классических ИИ-решателей за счет гибридного подхода: использования Vision-Transformer нейросетей для компьютерного зрения и строгих детерминированных алгоритмов компьютерной алгебры (CAS) для вычислений.

> 🎓 **Примечание:** Данный проект является выпускной квалификационной работой (ВКР). В связи с академическими правилами, полный исходный код бэкенда может находиться в закрытом доступе и предоставляется по запросу. Ниже приведена архитектурная спецификация платформы.

---

## Ключевые возможности

* **Гибридное ядро (OCR + CAS):** Трансляция зашумленных изображений в LaTeX-код (модель `pix2tex`) с последующим парсингом AST-дерева и аналитическим решением через `SymPy`.
* **Генерация объяснений:** Система не просто выдает ответ, а расписывает решение шаг за шагом на естественном языке с проверкой ОДЗ и отсечением посторонних корней.
* **Human-in-the-Loop (Data Loop):** Интерактивный контур обратной связи. Если ИИ ошибся, пользователь может вручную поправить LaTeX-строку на клиенте. Система автоматически сохраняет очищенную пару «изображение-текст» в S3-хранилище, формируя «золотой датасет» для дообучения нейросети (Fine-tuning).
* **Продвинутая безопасность:** Двухтокенная JWT-авторизация и бесшовная интеграция с **Yandex OAuth**. Реализована защита от угона аккаунта: при попытке привязать Яндекс к уже существующему email, система принудительно запрашивает пароль.
* **Offline-First клиент:** Нативное Android-приложение кэширует историю решений в локальную БД (`Room`), обеспечивая мгновенный рендеринг векторной математики без доступа к сети.

---

## Архитектура системы

Система построена на базе микросервисной событийно-ориентированной архитектуры (EDA), что позволяет изолировать тяжелые ML-вычисления от быстрых REST-запросов.
<img width="4526" height="2095" alt="mermaid-diagram" src="https://github.com/user-attachments/assets/6b2cc10f-8988-4c2b-8a47-a071e7a4c6b2" />

### Компоненты системы:
1.  **Spring Cloud API Gateway:** Единая точка входа. Терминирует трафик и осуществляет маршрутизацию.
2.  **Security Service (Java):** Изолированный контур аутентификации (RBAC, JWT, OAuth 2.0). Защищает систему от Brute-force атак с автоматической блокировкой в PostgreSQL.
3.  **Solution Archive Service (Java):** Центральный оркестратор. Управляет бизнес-логикой, обрабатывает фидбек пользователей и взаимодействует с брокером сообщений.
4.  **OCR / CAS Workers (Python):** Асинхронные воркеры на базе FastAPI и Celery, выполняющие ресурсоемкие задачи по распознаванию и вычислениям.
5.  **Infrastructure Layer:** PostgreSQL 17 (RDBMS), Redis 7 (In-memory Cache), RabbitMQ 4 (Message Broker), MinIO (S3 Object Storage).

---

## Технологический стек

| Слой | Технологии |
| :--- | :--- |
| **Mobile Client** | Kotlin, Jetpack Compose, CameraX, Room DB, Retrofit 2, DataStore, MVI |
| **Backend Core** | Java 25, Spring Boot, Spring Cloud, Spring Security, Hibernate / JPA |
| **Compute Core** | Python, FastAPI, Celery, SymPy, pix2tex |
| **Data & Messaging** | PostgreSQL 17, Redis 7, RabbitMQ 4 (с Delay Queues), MinIO |
| **DevOps & Sec** | Docker Compose, Nginx, WireGuard, Tailscale, GitHub Actions, Dozzle |

---

## DevOps и Развертывание

Проект спроектирован с учетом жестких аппаратных ограничений (стенд: 4 vCPU, 8 GB RAM) и отсутствия выделенного белого IP-адреса у физического сервера.

* **Сеть:** Публичный трафик терминируется на облачном VPS (Nginx + SSL) и проксируется на физический сервер через защищенный **WireGuard** туннель.
* **CI/CD:** Настроен автоматизированный конвейер в **GitHub Actions**. При слиянии в `master` образы собираются, пушатся в **ghcr.io** и деплоятся на сервер через защищенную Mesh-сеть **Tailscale** (Zero-downtime deployment).
* **Нагрузочное тестирование:** Архитектура с использованием Redis и RabbitMQ позволяет системе удерживать пропускную способность до **193 RPS** со средним временем отклика шлюза в 86 мс.

---

## Скриншоты и Мониторинг

<details>
  <summary><b>Посмотреть скриншоты приложения, метрики и дашборды</b></summary>
  <br>

  ### 📱 Интерфейс мобильного приложения (Android Client)
  *Декларативный UI на Jetpack Compose, интеграция CameraX и векторный рендеринг LaTeX.*

  | 1. Авторизация & Безопасность | 2. Главный экран и Скан | 3. Интерактивный фидбек | 4. Пошаговое решение |
  | :---: | :---: | :---: | :---: |
  | <img height="420" alt="Auth Screen" src="https://github.com/user-attachments/assets/74f14449-10e4-4f68-a9dc-202c1274b447" /> | <img height="420" alt="Camera Scan" src="https://github.com/user-attachments/assets/4be247c8-02aa-4b78-9aef-a4e64b8b6553" /> | <img height="420" alt="Human-in-the-Loop Feedback" src="https://github.com/user-attachments/assets/2f001493-41aa-42ec-b7bb-cb00ecd4c4b4" /> | <img height="420" alt="Math Solution Render" src="https://github.com/user-attachments/assets/e0742f55-4406-414e-a9e4-d33fb5d36344" /> |

  ---

  ### 📈 Нагрузочное тестирование (Postman Performance)
  *Стресс-тестирование инфраструктуры: демонстрация эффективности кэширования Redis и асинхронной обработки RabbitMQ под нагрузкой.*

  #### Сценарий 1: Оценка слоя кэширования метаданных пользователей (Redis — 193 RPS)
  <img width="100%" alt="Postman Redis Cache Test" src="https://github.com/user-attachments/assets/542e6098-57cd-4d0e-98e5-04b949cd7e38" />

  #### Сценарий 2: Стресс-тест асинхронного контура CAS-вычислений (RabbitMQ — 126 RPS)
  <img width="100%" alt="Postman RabbitMQ Async Test" src="https://github.com/user-attachments/assets/5df27d08-794c-4ae2-9bd7-8179e6643564" />

  ---

  ### 🖥️ Мониторинг кластера (Dozzle)
  *Агрегация stdout/stderr логов и контроль утилизации ресурсов хоста (vCPU, RAM) в реальном времени.*

  <img width="100%" alt="Dozzle Cluster Observability" src="https://github.com/user-attachments/assets/028828e2-008b-4c54-9ce3-3004e2aaeb2f" />

</details>

---
*Разработано в рамках выпускной квалификационной работы. 2026 г.*
