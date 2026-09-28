# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Đinh Ngọc Đức |
| Mã học viên | 2A202602935 |
| Repo | https://github.com/dinhngocduc1311/K4-L3A-DAY12-DinhNgocDuc-2A202602935-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-production-5a89.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-09-28 |
| Redis | Railway Redis managed service |

## Biến Môi Trường Đã Cấu Hình

Chỉ tên và nguồn cấu hình được ghi lại; giá trị secret không nằm trong repository.

| Biến | Trạng thái | Nguồn |
|------|------------|-------|
| `PORT` | Railway tự cung cấp | Railway runtime |
| `AGENT_API_KEY` | Đã cấu hình | Railway service variable, giá trị được giữ kín |
| `REDIS_URL` | Đã cấu hình | Reference variable `${{Redis.REDIS_URL}}` |
| `RATE_LIMIT_PER_MINUTE` | Đã cấu hình | Railway service variable |
| `MONTHLY_BUDGET_USD` | Đã cấu hình | Railway service variable |
| `LOG_LEVEL` | Đã cấu hình | Railway service variable |

## Kết Quả Kiểm Tra Thực Tế

| Kiểm tra | HTTP | Kết quả |
|----------|------|---------|
| `GET /health` | 200 | `{"status":"ok","service":"day12-agent","version":"1.0.0"}` |
| Railway platform healthcheck | `/health` | Đã áp dụng; deployment `472c54f7-0b6f-4afc-887f-7958d9b86c29` `SUCCESS` |
| `GET /ready` | 200 | `{"status":"ready","redis":true}` |
| `POST /ask` không có API key | 401 | `{"detail":"invalid or missing API key"}` |
| `POST /ask` có API key | 200 | Trả về câu trả lời và ghi history vào Redis |

## Ảnh Minh Chứng

- Public health endpoint: [`screenshots/health.png`](screenshots/health.png)
- Railway dashboard: [`screenshots/dashboard.png`](screenshots/dashboard.png) — chụp thủ công từ dashboard đã đăng nhập.
