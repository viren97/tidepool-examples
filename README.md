# Tidepool examples

Three small demo services that show how a service plugs into
[Tidepool](https://github.com/viren97/tidepool): a `tide.yaml` describing what the service needs, and
a GitHub Actions workflow that builds the image and pushes it to ECR.

> **Status: design phase.** No code yet. See the Tidepool [MVP plan](https://github.com/viren97/tidepool/blob/main/docs/MVP.md).

## Goals

1. **Show the contract:** a working `tide.yaml` and `.github/workflows/build.yml` for each service,
   ready to copy into a real service repo.
2. **Use every MVP dependency:** between them, the services use Postgres, Kafka and Redis, plus a
   service-to-service dependency. One Tidepool command therefore exercises the whole resolver.
3. **Act as Tidepool's test fixture:** Tidepool's end-to-end tests deploy these services, so "it works
   with the examples" is part of Tidepool's definition of done.
4. **Stay tiny:** each service is a few hundred lines at most. They exist to show integration, not features.

## The services

| Service | What it is | Needs | Shows off |
|---|---|---|---|
| `web` | Frontend container | `orders-api` | A frontend that reads backend URLs from env vars at runtime, so one image works in every env |
| `orders-api` | REST API for orders | Postgres (`orders` DB, seeded), Kafka (`order-created`, `order-paid`), Redis, `payments-api` | Migrations + seed Job, Kafka topics, cache, service dependency |
| `payments-api` | Fake payment processor | Postgres (`payments` DB), Kafka (consumes `order-created`, produces `order-paid`) | Event-driven flow between services |

`/tide up web` should bring up all three services plus Postgres, Kafka and Redis. Placing an order
in the UI should travel through Kafka and come back as "paid".

```
web ──HTTP──> orders-api ──HTTP──> payments-api
                 │  ▲                 │
                 │  └── order-paid ───┤   (Kafka)
                 └──── order-created ─┘
              Postgres: orders, payments   Redis: orders cache
```

## Planned layout

```
tidepool-examples/
  web/
    tide.yaml
    Dockerfile
    src/...
  orders-api/
    tide.yaml
    seed/orders.sql
    src/...
  payments-api/
    tide.yaml
    src/...
  .github/workflows/
    build-web.yml            # each calls tidepool's reusable build-push-ecr.yml
    build-orders-api.yml
    build-payments-api.yml
```

The MVP plan has one repo per service. Here all three live in one repo to keep the demo easy to clone.
Each one has its own workflow with a path filter, so they still build and get tagged independently
(`sha-<7>`, `pr-<n>`, `<branch>`).

## Tech
- `orders-api`, `payments-api`: Kotlin + Spring Boot, the same stack as Tidepool
- `web`: a small static page served by a minimal container, configured through env vars at startup

## License
To be decided (same as Tidepool).
