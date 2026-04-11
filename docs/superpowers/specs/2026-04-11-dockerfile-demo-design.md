# DockerfileDemo 設計文件

## 目的

透過同一個 .NET Console App 搭配兩個不同的 Dockerfile，讓學習者理解 single-stage 與 multi-stage build 的差異，並親身體會 image 大小的對比。

## 專案結構

```
DockerfileDemo/
├── src/
│   └── DockerfileDemo/
│       ├── Program.cs
│       └── DockerfileDemo.csproj
├── Dockerfile.single          # 方案 A：single-stage
├── Dockerfile.multi           # 方案 B：multi-stage
├── .dockerignore
└── README.md                  # 教學引導文件
```

## Console App

- **Runtime:** .NET 8（LTS）
- **類型:** Console App，使用 top-level statements
- **相依套件:** 無，只用 BCL
- **輸出內容:**

```csharp
Console.WriteLine("=== DockerfileDemo ===");
Console.WriteLine($"  .NET Version:  {Environment.Version}");
Console.WriteLine($"  OS:            {Environment.OSVersion}");
Console.WriteLine($"  Machine Name:  {Environment.MachineName}");
Console.WriteLine($"  Timestamp:     {DateTime.UtcNow:O}");
```

- `MachineName` 在 container 裡會顯示 container ID，證明程式跑在 Docker 裡
- `Timestamp` 每次執行不同，確認不是快取結果

## Dockerfile.single（方案 A：single-stage）

```dockerfile
# 使用 .NET 8 SDK 作為基底映像（包含編譯器和執行環境）
FROM mcr.microsoft.com/dotnet/sdk:8.0

# 設定容器內的工作目錄
WORKDIR /app

# 將本機的專案檔案複製到容器內
COPY src/DockerfileDemo/ .

# 編譯專案，輸出到 /out 目錄
RUN dotnet publish -c Release -o /out

# 設定容器啟動時執行的指令
ENTRYPOINT ["dotnet", "/out/DockerfileDemo.dll"]
```

- 整個 SDK 留在最終 image 裡
- 預期 image 大小：~800MB

## Dockerfile.multi（方案 B：multi-stage）

```dockerfile
# === Stage 1: 編譯階段 ===
# 使用 SDK 映像來編譯程式碼，命名為 "build"
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build

# 設定編譯用的工作目錄
WORKDIR /src

# 將本機的專案檔案複製到容器內
COPY src/DockerfileDemo/ .

# 編譯專案，輸出到 /app 目錄
RUN dotnet publish -c Release -o /app

# === Stage 2: 執行階段 ===
# 換成只有 runtime 的輕量映像（不含 SDK）
FROM mcr.microsoft.com/dotnet/runtime:8.0

# 設定執行用的工作目錄
WORKDIR /app

# 從 build 階段複製編譯好的檔案過來
COPY --from=build /app .

# 設定容器啟動時執行的指令
ENTRYPOINT ["dotnet", "DockerfileDemo.dll"]
```

- SDK 只存在於 build stage，最終 image 只含 runtime
- 預期 image 大小：~190MB

## .dockerignore

```
**/bin/
**/obj/
.git
.gitignore
README.md
*.md
```

排除編譯產物、git 資料和文件檔案，避免不必要的東西進入 Docker build context。

## README 教學流程

1. **簡介** — 說明 repo 目的：透過對比兩種 Dockerfile 學習 Docker 容器化
2. **前置需求** — Docker Desktop 已安裝、.NET 8 SDK（選用，本機跑才需要）
3. **步驟一：跑方案 A（single-stage）**
   ```bash
   docker build -f Dockerfile.single -t demo-single .
   docker run demo-single
   docker images demo-single    # 觀察 image 大小 (~800MB)
   ```
4. **步驟二：跑方案 B（multi-stage）**
   ```bash
   docker build -f Dockerfile.multi -t demo-multi .
   docker run demo-multi
   docker images demo-multi     # 觀察 image 大小 (~190MB)
   ```
5. **步驟三：比較差異**
   ```bash
   docker images | grep demo
   ```
   附上預期結果表格，說明 multi-stage 為什麼 image 小很多
6. **延伸學習** — 簡單提及 Alpine-based image 的可能性，給有興趣的人方向

## 預期成果

| | 方案 A (single) | 方案 B (multi) |
|---|---|---|
| Dockerfile 行數 | ~5 行 | ~9 行 |
| Base image | sdk:8.0 | runtime:8.0 |
| 預期 image 大小 | ~800MB | ~190MB |
| 包含 SDK | 是 | 否 |
