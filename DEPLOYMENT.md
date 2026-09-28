# Thong Tin Deploy - Checkpoint 5

> Chi ghi TEN bien moi truong, khong dan gia tri API key vao file nay.

## Thong Tin Hoc Vien

| Muc | Noi dung |
|-----|----------|
| Ho va ten | Tran Cao Thang |
| Mã học viên | 2A202602520 |
| Repo | https://github.com/ThangC4T/K4-L3A-DAY12-TranCaoThang-2A202602520-CloudServicesAndDeployment |

## Service

| Muc | Noi dung |
|-----|----------|
| Public URL | https://day12-agent-5icg.onrender.com |
| Platform | Render |
| Ngay deploy | 2026-09-28 |

## Bien Moi Truong Da Set Tren Cloud

Ghi ten bien va nguon gia tri, khong ghi gia tri secret:

| Bien | Da set | Ghi chu |
|------|--------|---------|
| `PORT` | yes | Render blueprint set `10000` |
| `AGENT_API_KEY` | yes | Dat trong Render Environment, khong nam trong repo |
| `REDIS_URL` | yes | Lay tu Render Redis/Valkey service `day12-redis` |
| `RATE_LIMIT_PER_MINUTE` | yes | 10 |
| `MONTHLY_BUDGET_USD` | yes | 10.0 |
| `LOG_LEVEL` | yes | INFO |

## Lenh Kiem Tra

```bash
# 1. Liveness - mong doi 200 {"status":"ok"}
curl -i https://day12-agent-5icg.onrender.com/health

# 2. Readiness - mong doi 200 {"status":"ready"} va redis=true
curl -i https://day12-agent-5icg.onrender.com/ready

# 3. Khong co API key - mong doi 401
curl -i -X POST https://day12-agent-5icg.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Co API key - mong doi 200 kem cau tra loi
curl -i -X POST https://day12-agent-5icg.onrender.com/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy la gi?"}'

# 5. Rate limit - goi 15 lan, nhung lan cuoi phai tra 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-agent-5icg.onrender.com/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Ket Qua Chay That

```text
GET /health
HTTP/1.1 200 OK
{"status":"ok","service":"day12-agent","version":"1.0.0"}

GET /ready
HTTP/1.1 200 OK
{"status":"ready","redis":true}

POST /ask khong co API key
HTTP status: 401

Render logs:
Uvicorn running on http://0.0.0.0:10000
GET /health HTTP/1.1 200 OK
Your service is live
```

## Anh Chup Man Hinh

Dat anh trong thu muc `screenshots/`:

- `screenshots/dashboard.png` - trang quan ly service tren Render
- `screenshots/health.png` - ket qua goi `/health` tu trinh duyet hoac curl
