# DockerfileDemo Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a .NET 8 Console App with two Dockerfiles (single-stage vs multi-stage) to teach Docker containerization through comparison.

**Architecture:** One Console App in `src/DockerfileDemo/`, two Dockerfiles at repo root (`Dockerfile.single`, `Dockerfile.multi`), a `.dockerignore`, and a README with step-by-step tutorial. No external dependencies.

**Tech Stack:** .NET 8, Docker, C# top-level statements

---

## File Structure

| File | Responsibility |
|---|---|
| `src/DockerfileDemo/DockerfileDemo.csproj` | .NET 8 Console App project file |
| `src/DockerfileDemo/Program.cs` | App entry point — prints system info |
| `Dockerfile.single` | Single-stage build (SDK image) |
| `Dockerfile.multi` | Multi-stage build (SDK → runtime) |
| `.dockerignore` | Exclude build artifacts and git from Docker context |
| `README.md` | Tutorial guide with commands and expected output |

---

### Task 1: Create the .NET Console App

**Files:**
- Create: `src/DockerfileDemo/DockerfileDemo.csproj`
- Create: `src/DockerfileDemo/Program.cs`

- [ ] **Step 1: Create the project directory**

```bash
mkdir -p src/DockerfileDemo
```

- [ ] **Step 2: Create the project file**

Create `src/DockerfileDemo/DockerfileDemo.csproj`:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
</Project>
```

- [ ] **Step 3: Create Program.cs**

Create `src/DockerfileDemo/Program.cs`:

```csharp
Console.WriteLine("=== DockerfileDemo ===");
Console.WriteLine($"  .NET Version:  {Environment.Version}");
Console.WriteLine($"  OS:            {Environment.OSVersion}");
Console.WriteLine($"  Machine Name:  {Environment.MachineName}");
Console.WriteLine($"  Timestamp:     {DateTime.UtcNow:O}");
```

- [ ] **Step 4: Verify the app builds and runs locally**

```bash
cd src/DockerfileDemo && dotnet run
```

Expected output (values will differ on your machine):
```
=== DockerfileDemo ===
  .NET Version:  8.0.x
  OS:            Unix x.x.x.x (or Microsoft Windows ...)
  Machine Name:  your-hostname
  Timestamp:     2026-04-11T...Z
```

- [ ] **Step 5: Commit**

```bash
git add src/DockerfileDemo/DockerfileDemo.csproj src/DockerfileDemo/Program.cs
git commit -m "feat: add .NET 8 Console App"
```

---

### Task 2: Create .dockerignore

**Files:**
- Create: `.dockerignore`

- [ ] **Step 1: Create .dockerignore**

Create `.dockerignore`:

```
**/bin/
**/obj/
.git
.gitignore
README.md
*.md
```

- [ ] **Step 2: Commit**

```bash
git add .dockerignore
git commit -m "chore: add .dockerignore"
```

---

### Task 3: Create Dockerfile.single (方案 A)

**Files:**
- Create: `Dockerfile.single`

- [ ] **Step 1: Create Dockerfile.single**

Create `Dockerfile.single`:

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

- [ ] **Step 2: Build the Docker image**

```bash
docker build -f Dockerfile.single -t demo-single .
```

Expected: Build succeeds, image tagged as `demo-single`.

- [ ] **Step 3: Run the container**

```bash
docker run demo-single
```

Expected output:
```
=== DockerfileDemo ===
  .NET Version:  8.0.x
  OS:            Unix x.x.x.x
  Machine Name:  <container-id>
  Timestamp:     2026-04-11T...Z
```

Note: `Machine Name` will show a container ID (e.g., `a1b2c3d4e5f6`), proving it runs inside Docker.

- [ ] **Step 4: Check image size**

```bash
docker images demo-single
```

Expected: SIZE column shows ~800MB.

- [ ] **Step 5: Commit**

```bash
git add Dockerfile.single
git commit -m "feat: add Dockerfile.single (single-stage build)"
```

---

### Task 4: Create Dockerfile.multi (方案 B)

**Files:**
- Create: `Dockerfile.multi`

- [ ] **Step 1: Create Dockerfile.multi**

Create `Dockerfile.multi`:

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

- [ ] **Step 2: Build the Docker image**

```bash
docker build -f Dockerfile.multi -t demo-multi .
```

Expected: Build succeeds, image tagged as `demo-multi`.

- [ ] **Step 3: Run the container**

```bash
docker run demo-multi
```

Expected output (same as single-stage):
```
=== DockerfileDemo ===
  .NET Version:  8.0.x
  OS:            Unix x.x.x.x
  Machine Name:  <container-id>
  Timestamp:     2026-04-11T...Z
```

- [ ] **Step 4: Check image size and compare**

```bash
docker images | grep demo
```

Expected:
```
demo-single   latest   ...   ~800MB
demo-multi    latest   ...   ~190MB
```

- [ ] **Step 5: Commit**

```bash
git add Dockerfile.multi
git commit -m "feat: add Dockerfile.multi (multi-stage build)"
```

---

### Task 5: Write README.md

**Files:**
- Create: `README.md`

- [ ] **Step 1: Create README.md**

Create `README.md`:

```markdown
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

> 預期大小：約 800MB（因為包含整個 .NET SDK）

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

> 預期大小：約 190MB（只包含 .NET Runtime）

## 步驟三：比較差異

```bash
docker images | grep demo
```

| | 方案 A (single) | 方案 B (multi) |
|---|---|---|
| Base Image | sdk:8.0 | runtime:8.0 |
| 包含 SDK | 是 | 否 |
| Image 大小 | ~800MB | ~190MB |

**為什麼差這麼多？** 方案 A 把整個 SDK（編譯器、NuGet 工具等）留在最終 image 裡，但執行時根本用不到。方案 B 透過 multi-stage build，只保留執行所需的 runtime，大幅縮小 image。

## 延伸學習

想進一步縮小 image？可以嘗試：

- 使用 Alpine-based runtime image（`mcr.microsoft.com/dotnet/runtime:8.0-alpine`）
- 使用 self-contained 發佈搭配 `runtime-deps` image
- 這些方法可以將 image 縮小到約 50MB
```

- [ ] **Step 2: Review the README renders correctly**

Open the file and verify markdown formatting looks correct.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: add tutorial README"
```

---

### Task 6: End-to-End Verification

- [ ] **Step 1: Clean up any existing demo images**

```bash
docker rmi demo-single demo-multi 2>/dev/null; echo "cleaned"
```

- [ ] **Step 2: Build and run single-stage**

```bash
docker build -f Dockerfile.single -t demo-single .
docker run demo-single
```

Verify output shows .NET version, OS, machine name, and timestamp.

- [ ] **Step 3: Build and run multi-stage**

```bash
docker build -f Dockerfile.multi -t demo-multi .
docker run demo-multi
```

Verify output matches single-stage (same app, different packaging).

- [ ] **Step 4: Compare image sizes**

```bash
docker images | grep demo
```

Verify `demo-single` is ~800MB and `demo-multi` is ~190MB.

- [ ] **Step 5: Final commit if any changes needed**

If all tests pass and no changes needed, this step is a no-op.
