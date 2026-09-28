# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Võ Phú Hãn |
| Mã học viên | 2A202602628 |
| Repo | https://github.com/yohan-vinai/K4-L3A-DAY12-VoPhuHan-2A202602628-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-bw5m.onrender.com |
| Platform | Render (Free Web Service + Free Key Value) |
| Ngày deploy | 2026-09-28 |
| Branch / commit | `day12/cloud-deployment` / `c9db5e9` |

## Biến Môi Trường Đã Set Trên Cloud

Chỉ ghi tên và nguồn cấp, không ghi giá trị secret.

| Biến | Trạng thái | Nguồn |
|------|------------|-------|
| `PORT` | Tự cấp | Render |
| `AGENT_API_KEY` | Đã set | A nhập trực tiếp trong Render Dashboard → Environment |
| `REDIS_URL` | Đã cấu hình | Render Key Value `day12-redis` |
| `RATE_LIMIT_PER_MINUTE` | Đã cấu hình | Blueprint (`10`) |
| `MONTHLY_BUDGET_USD` | Đã cấu hình | Blueprint (`10.0`) |
| `LOG_LEVEL` | Đã cấu hình | Blueprint (`INFO`) |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên. Key được lưu trong `.env` cục bộ, không commit.

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i <URL>/health

# 2. Readiness — mong đợi 200 {"status":"ready","redis":true}
curl -i <URL>/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'
```

## Kết Quả Chạy Thật

Build Render từ commit `c9db5e9`: **Live**, plan **Free**. Redis Key Value
`day12-redis` ở trạng thái **Available**.

```text
GET /health  -> HTTP 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET /ready   -> HTTP 200 {"status":"ready","redis":true}
POST /ask không key -> HTTP 401 {"detail":"invalid or missing API key"}
POST /ask có key -> HTTP 200 và trả câu trả lời
pytest tests/test_cp5.py -v -> 9 passed, 4 skipped (các test local fallback)
```

## Ảnh Chụp Màn Hình

- `screenshots/dashboard.png` — service Render đang Live trên plan Free.

Các kết quả `/health`, `/ready` và auth ở trên được kiểm tra trực tiếp bằng
curl và `tests/test_cp5.py`.
