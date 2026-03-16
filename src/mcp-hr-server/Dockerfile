FROM golang:alpine AS builder
WORKDIR /app

# 1. 先拷贝依赖定义
COPY go.mod go.sum ./
# 2. 再拷贝源码（必须在 tidy 之前）
COPY main.go .

# 3. 下载依赖
RUN go mod tidy

# 4. 编译
RUN CGO_ENABLED=0 GOOS=linux go build -o mcp-server .

# 构建最终镜像
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/mcp-server .
EXPOSE 8080
CMD ["./mcp-server"]
