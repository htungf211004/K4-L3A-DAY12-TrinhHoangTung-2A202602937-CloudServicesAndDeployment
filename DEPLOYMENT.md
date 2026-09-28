# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Trịnh Hoàng Tùng |
| Mã học viên | L3A202602937 |
| Repo | https://github.com/htungf211004/K4-L3A-DAY12-TrinhHoangTung-2A202602937-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://agent-production-15ba.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-09-28 |
| Deployment | `46a1ffa2-67e5-44ff-b9cf-8fad2a8c4a00` — `SUCCESS` |

## Biến Môi Trường Đã Set Trên Cloud

Chỉ ghi tên biến và nguồn giá trị; giá trị secret không nằm trong repository.

| Biến | Đã set | Nguồn |
|------|--------|-------|
| `PORT` | ✅ | Railway tự gán |
| `AGENT_API_KEY` | ✅ | Railway service variable, truyền từ `.env` qua stdin |
| `REDIS_URL` | ✅ | Reference variable `${{Redis.REDIS_URL}}` tới Railway Redis service |
| `RATE_LIMIT_PER_MINUTE` | ✅ | Railway service variable |
| `MONTHLY_BUDGET_USD` | ✅ | Railway service variable |
| `LOG_LEVEL` | ✅ | Railway service variable |

## Lệnh Kiểm Tra

```bash
URL=https://agent-production-15ba.up.railway.app

curl -i "$URL/health"
curl -i "$URL/ready"

curl -i -X POST "$URL/ask" \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

curl -i -X POST "$URL/ask" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'
```

## Kết Quả Chạy Thật

Các kiểm tra dưới đây được chạy trực tiếp qua Internet tới domain Railway ngày
2026-09-28:

```text
GET /health
HTTP 200
{"status":"ok","service":"day12-agent","version":"1.0.0"}

GET /ready
HTTP 200
{"status":"ready","redis":true}

POST /ask (không có X-API-Key)
HTTP 401

POST /ask (có X-API-Key hợp lệ, X-User-Id: cp5-smoke)
HTTP 200
{"user_id":"cp5-smoke","history_length":0,"cost_usd":0.00002445,"tokens":{"in":7,"out":39}}

Rate limit — 15 request liên tiếp, cùng X-User-Id
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429
```

## Ảnh Chụp Màn Hình

- `screenshots/health.png` 
- `screenshots/dashboard.png` 
  ảnh phải thể hiện project có hai service `agent` và `Redis`.
