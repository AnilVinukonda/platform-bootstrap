# -------- Builder stage --------
FROM python:3.11-slim AS builder

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# -------- Runtime stage --------
FROM python:3.11-slim

WORKDIR /app

COPY --from=builder /install /usr/local
COPY src/ .

EXPOSE 8080

CMD ["python", "app.py"]

