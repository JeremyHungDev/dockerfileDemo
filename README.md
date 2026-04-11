# DockerfileDemo

透過同一個 .NET Console App，對比兩種 Dockerfile 寫法，學習 Docker 容器化的基本概念。

## 前置需求

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) 已安裝並執行
- .NET 8 SDK（選用，僅本機執行時需要）

## 專案結構

```
DockerfileDemo/
├── src/DockerfileDemo/    # .NET 8 Console App
├── Dockerfile.single      # 方案 A：single-stage build
├── Dockerfile.multi       # 方案 B：multi-stage build
├── .dockerignore
└── README.md
```

## 步驟一：Single-Stage Build（方案 A）

使用包含完整 SDK 的映像來編譯和執行：

```bash
docker build -f Dockerfile.single -t demo-single .
docker run demo-single
```

查看 image 大小：

```bash
docker images demo-single
```

> 預期大小：約 851MB（因為包含整個 .NET SDK）

## 步驟二：Multi-Stage Build（方案 B）

使用 SDK 映像編譯，再將結果複製到只含 runtime 的輕量映像：

```bash
docker build -f Dockerfile.multi -t demo-multi .
docker run demo-multi
```

查看 image 大小：

```bash
docker images demo-multi
```

> 預期大小：約 194MB（只包含 .NET Runtime）

## 步驟三：比較差異

```bash
docker images | grep demo
```

| | 方案 A (single) | 方案 B (multi) |
|---|---|---|
| Base Image | sdk:8.0 | runtime:8.0 |
| 包含 SDK | 是 | 否 |
| Image 大小 | ~851MB | ~194MB |

**為什麼差這麼多？** 方案 A 把整個 SDK（編譯器、NuGet 工具等）留在最終 image 裡，但執行時根本用不到。方案 B 透過 multi-stage build，只保留執行所需的 runtime，大幅縮小 image。

## 延伸學習

想進一步縮小 image？可以嘗試：

- 使用 Alpine-based runtime image（`mcr.microsoft.com/dotnet/runtime:8.0-alpine`）
- 使用 self-contained 發佈搭配 `runtime-deps` image
- 這些方法可以將 image 縮小到約 50MB
