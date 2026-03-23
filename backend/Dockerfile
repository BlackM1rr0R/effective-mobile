FROM python:3.12-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup && apk add --no-cache wget
WORKDIR /app
COPY app.py .
USER appuser
EXPOSE 8080
HEALTHCHECK --interval=15s --timeout=5s --start-period=10s --retries=3 CMD wget -qO- http://localhost:8080/ || exit 1
CMD ["python", "app.py"]