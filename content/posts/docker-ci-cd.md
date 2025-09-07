---
title: "CI/CD cho Docker: Mẫu bài viết với đầy đủ Markdown"
date: 2025-09-07
tags: ["devops", "docker"]
categories: ["DevOps"]
series: ["Docker"]
author: "Cao Sơn"
draft: false
showToc: true
tocOpen: false
cover:
  image: "images/pipeline.png"
  alt: "CI/CD Pipeline"
  caption: "Demo CI/CD pipeline"
  relative: true
  hidden: true
---

> **Mục tiêu**: Bài viết _demo_ mọi thành phần Markdown bạn cần cho blog DevOps: tiêu đề, danh sách, bảng, code, hình, trích dẫn, chú thích, liên kết, checklists, footnotes[^1], và một chút math.

---

## 1) Tiêu đề & Đoạn văn

Đây là một đoạn văn mẫu. Bạn có thể **in đậm**, _in nghiêng_, hoặc \`inline code\`.  
Dòng ngắt  
xuống hàng.

### 1.1) Danh sách

- Bullet 1
- Bullet 2
  - Bullet 2.1
  - Bullet 2.2
- Bullet **3**

1. Mục 1
2. Mục 2
   1. Mục 2.1
   2. Mục 2.2

### 1.2) Danh sách công việc (Task list)

- [x] Tạo repo
- [x] Cấu hình GitHub Actions
- [ ] Viết bài “Observability với Grafana”
- [ ] Thêm search (Lunr/Algolia)

---

## 2) Bảng (Table)

| Stage   | Action                | Tool               | Result                 |
| ------- | --------------------- | ------------------ | ---------------------- |
| Build   | Compile & Lint        | Node.js, ESLint    | Artifacts + Reports    |
| Test    | Unit + Integration    | Jest               | PASS/FAIL              |
| Package | Docker build/push     | Docker, GHCR       | Tagged Image           |
| Deploy  | Pages (static) / Helm | GitHub Pages / k8s | Site up / App upgraded |

---

## 3) Ảnh & Figure

Ảnh đơn giản (đặt file vào `static/images/pipeline.png`):

![CI/CD Pipeline](images/pipeline.png)

Hugo shortcode _figure_ (hiện caption đẹp hơn):

{{< figure src="/images/pipeline.png" alt="CI/CD Pipeline" caption="Images 1: CI/CD overview" >}}

> **Tip**: Nếu dùng _Page Bundle_, hãy đặt bài viết là `content/posts/docker-ci-cd/index.md` và ảnh chung thư mục: `content/posts/docker-ci-cd/pipeline.png`. Khi đó dùng đường dẫn tương đối như `./pipeline.png`.

---

## 4) Code blocks (fences)

### 4.1) Shell (bash)

```bash
# Kiểm tra Docker & Compose
docker version
docker compose version

# Build & run local
docker build -t hugo-site .
docker run -p 1313:1313 hugo-site
```
