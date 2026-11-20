# Ecommerce-reference

OldShop là ứng dụng mua bán đồ cũ dùng FastAPI, PostgreSQL và Angular. Repo này
được dùng để thử nghiệm cho học phần Thương mại điện tử và phát triển từ
Epic-Ecommerce-FastAPI.

## Chạy trên Windows hoặc macOS

1. Cài và mở **Docker Desktop**. Trên Windows dùng Linux containers. Trên Mac chọn
   bản đúng chip Apple silicon hoặc Intel. [Windows](https://docs.docker.com/desktop/setup/install/windows-install/) · [macOS](https://docs.docker.com/desktop/setup/install/mac-install/)
2. Clone hoặc pull `NguyenDangDaiNam/Ecommerce-reference` bằng GitHub Desktop.
   Mở folder repo trong VS Code và mở terminal tại thư mục có `docker-compose.yml`.
3. Chạy một lệnh cho cả frontend, backend và PostgreSQL:

   ```sh
   docker compose up --build --wait
   ```

   Lần đầu Docker cần tải image và build ứng dụng. Project tự nạp schema,
   sản phẩm, ảnh và tài khoản mẫu từ repo. Máy chỉ cần Docker Desktop để chạy;
   Python, Node.js và PostgreSQL đã nằm trong các container.

4. Mở [cửa hàng](http://127.0.0.1:8080) hoặc [Swagger UI](http://127.0.0.1:8000/api/docs).

| Tài khoản mẫu | Email | Mật khẩu |
|---|---|---|
| Khách hàng | `user1@gmail.com` | `user123` |
| Quản trị viên | `admin@oldshop.com` | `admin123` |

Bản chạy thử dùng **thanh toán khi nhận hàng (COD)**. PayPal, email OTP và
chuyển tiếp Google Apps Script để tắt. Dùng các tài khoản mẫu trên máy local.

**Dừng và giữ dữ liệu** bằng lệnh:

```sh
docker compose stop
```

Khi mở lại hoặc sau khi pull mã mới, chạy lại `docker compose up --build --wait`.
Ảnh và dữ liệu được lưu trong volume Docker. Khi repo có migration mới, áp dụng
migration theo hướng dẫn của phiên bản đó.

Nếu lệnh báo lỗi, kiểm tra Docker Desktop đã mở, rồi xem dịch vụ nào gặp vấn đề:

```sh
docker compose ps -a
docker compose logs --tail=80
```

Nếu cổng `8080` hoặc `8000` đang bận, dừng đúng ứng dụng đang chạy trên cổng đó
trước khi thử lại. Không cần nhập file `database.sql` bên ngoài repo.

Xem thêm [cách clone và pull bằng GitHub Desktop cho nhóm TMĐT](docs/huong-dan-cai-dat-nhom-tmdt.md).

A full-stack marketplace for second-hand goods: buyers browse and check out (COD or PayPal),
sellers list products that go through moderation, moderators handle reports and bans,
admins manage staff, vouchers and feedback. Each checkout is risk-scored before payment.

**Stack:** FastAPI · SQLAlchemy · PostgreSQL (SQL-first schema, PL/pgSQL) · Angular 21 · PayPal Orders API ·
scikit-learn · Google Apps Script · Docker · Caddy · Nginx · Prometheus · Grafana · Loki/Promtail ⇄ ELK ·
GitHub Actions · SonarCloud

```mermaid
flowchart LR
  user((Browser)) -->|HTTPS| caddy[Caddy<br/>TLS, HTTP/3]
  caddy --> web[Nginx<br/>Angular SPA]
  web -->|/api, /media| api[FastAPI<br/>uvicorn workers]
  api --> pg[(PostgreSQL)]
  api -->|Orders v2| paypal[PayPal]
  api -->|signed webhook| gas[Apps Script<br/>Sheet helpdesk + email]
  api -.->|/metrics| prom[Prometheus] --> grafana[Grafana]
  api -.->|JSON logs| ship{{Promtail or Filebeat}}
  ship --> loki[Loki] --> grafana
  ship --> es[Elasticsearch] --> kibana[Kibana]
```

## Features

| Area | What it does |
|---|---|
| Catalogue | Category tree, search, sorting, product pages with images and reviews, view counter |
| Cart | Stock is reserved when an item is added (row-level `SELECT … FOR UPDATE`) and released on removal |
| Checkout | COD or PayPal (sandbox), vouchers (fixed / percentage, min order, usage cap), address book |
| Fraud check | Hard limit + IsolationForest score on every order; risky PayPal checkouts are blocked or delayed |
| Seller | Multipart listing with image upload, moderation status, order handling, dashboard stats |
| Moderation | Approve/reject listings, review reports (rating 0), trust score penalties, auto-ban, violation log |
| Admin | Moderator accounts, vouchers, system feedback, revenue/user stats |
| Contact | Form → PostgreSQL → HMAC-signed relay to a Google Apps Script helpdesk sheet ([details](integrations/google-apps-script/README.md)) |
| Ops | Health/readiness probes, Prometheus metrics, JSON logs with request ids, one-command deploy and log-stack switch |

## Repository layout

```
backend/                  FastAPI service
  app/api/                routers (v1) and dependencies (auth, role guards)
  app/core/               config, DB session, security, logging, domain enums and errors
  app/crud/               data access and business rules
  app/models/ schemas/    SQLAlchemy models, Pydantic models (split by domain)
  app/services/           PayPal, fraud scoring, mail, media storage, Apps Script relay
  scripts/init_db.py      idempotent bootstrap: create DB, load schema/seed once, copy seed images
  tests/                  integration tests against a real PostgreSQL
database/                 SQL source of truth: schema, functions/triggers, seed data, seed images
frontend/                 Angular SPA + Nginx config
integrations/google-apps-script/   contact relay (clasp project)
deploy/                   production compose files, Caddy, Prometheus, Grafana, Loki, ELK, ops scripts
docs/deployment.md        VPS runbook
```

## Tests

The backend suite exercises the real API against PostgreSQL loaded with the seed data
(auth, RBAC, cart stock reservation, orders, vouchers, fraud rule, moderation, uploads, contact relay, metrics).

```bash
createdb oldshop_test && (cd database && psql -d oldshop_test -f init.sql)
cd backend
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/oldshop_test pytest
ruff check . && ruff format --check .
```

CI runs the same steps on every push, builds the Angular app, and builds both Docker images on pull requests.

## Deployment

Production runs on a single VPS with Docker Compose; see **[docs/deployment.md](docs/deployment.md)**.

- `git push` to `main` → CI → images pushed to GHCR → SSH deploy with the commit tag (rollback = redeploy an older tag)
- Caddy terminates TLS for the shop, Grafana and Kibana sub-domains
- Metrics: Prometheus scrapes the API, PostgreSQL and the host; Grafana dashboards and alert rules are provisioned
- Logs: Promtail → Loki by default; `deploy/scripts/switch-logging.sh elk` swaps in Filebeat → Elasticsearch → Kibana
  without restarting the application (both read the same Docker log files)

## Notes on design choices

- **SQL-first schema.** Tables, indexes, triggers and seed data live in `database/` and are loaded once by
  `scripts/init_db.py`; the ORM maps onto them instead of generating them. Database tuning is passed to the
  Postgres container as flags, never via `ALTER SYSTEM` from an init script.
- **Stock reservation in the cart** rather than at payment keeps the "last item" race in one place, guarded by row locks.
- **Fraud model is optional at runtime.** Artifacts are mounted into the container; without them the service falls back
  to the hard business rule so dev and CI need no binary files.
- **The Apps Script URL never reaches the browser.** The backend signs each relay with HMAC-SHA256 and a timestamp,
  since Apps Script web apps cannot read request headers.
