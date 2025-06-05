# Lab 03 – Go Application

## Огляд

Go HTTP-сервер який повертає розмітку з трьома варіантами образів:

- `Dockerfile.single` – single-stage (~7s, ~320 MB)
- `Dockerfile.scratch` – multi-stage → scratch (~5.9s, ~9.99 MB)  
- `Dockerfile.distroless` – multi-stage → distroless (~5.6s, ~12 MB)

## Структура проекту

```
lab-03-golang/
├── cmd/
│   ├── main.go
│   └── go.mod
├── Dockerfile.single
├── Dockerfile.scratch
├── Dockerfile.distroless
└── README.md
```

## Локальний запуск

```bash
cd cmd
go mod download
go build -o fizzbuzz main.go
./fizzbuzz serve
curl http://localhost:8080/
```

## Docker образи

### 1. Dockerfile.single (single-stage)

Простий односторонній образ на базі golang:1.21-alpine

```bash
docker pull golang:1.21-alpine
docker rmi space/lab03-golang:single >/dev/null 2>&1 || true
time docker build -t space/lab03-golang:single -f Dockerfile.single .
# ~7s, ~320 MB
```

**Запуск:**

```bash
docker run -d --name go-single -p 8080:8080 space/lab03-golang:single
curl http://localhost:8080/
docker stop go-single && docker rm go-single
```

### 2. Dockerfile.scratch (multi-stage → scratch)

Багатоступеневий збір з фінальним образом на базі scratch

```bash
docker rmi space/lab03-golang:scratch >/dev/null 2>&1 || true
time docker build -t space/lab03-golang:scratch -f Dockerfile.scratch .
# ~5.9s, ~9.99 MB
```

**Запуск:**

```bash
docker run -d --name go-scratch -p 8080:8080 space/lab03-golang:scratch
curl http://localhost:8080/
docker stop go-scratch && docker rm go-scratch
```

### 3. Dockerfile.distroless (multi-stage → distroless)

Багатоступеневий збір з фінальним образом на базі distroless

```bash
docker rmi space/lab03-golang:distroless >/dev/null 2>&1 || true
time docker build -t space/lab03-golang:distroless -f Dockerfile.distroless .
# ~5.6s, ~12 MB
```

**Запуск:**

```bash
docker run -d --name go-distroless -p 8080:8080 space/lab03-golang:distroless
curl http://localhost:8080/
docker stop go-distroless && docker rm go-distroless
```

## Порівняння образів

| Тип образу | Час збірки | Розмір образу | Переваги | Недоліки |
|------------|------------|---------------|----------|----------|
| Single-stage | ~7s | ~320 MB | Простота, інструменти для дебагу | Великий розмір |
| Scratch | ~5.9s | ~9.99 MB | Мінімальний розмір | Немає інструментів, складний дебаг |
| Distroless | ~5.6s | ~12 MB | Баланс між розміром і функціональністю | Обмежені можливості дебагу |

## Тестування

Після запуску будь-якого з образів, сервер буде доступний на `http://localhost:8080/`

```bash
curl http://localhost:8080/
```

## Вимоги

- Docker
- Go 1.21+ (для локального запуску)

## Корисні команди

**Перегляд розміру образів:**
```bash
docker images | grep space/lab03-golang
```

**Очищення всіх образів:**
```bash
docker rmi space/lab03-golang:single space/lab03-golang:scratch space/lab03-golang:distroless
```

**Перегляд запущених контейнерів:**
```bash
docker ps
```