---
title: "Hướng dẫn thiết lập blog với Hugo và PaperMod"
date: 2024-01-15T10:00:00+07:00
draft: false
tags: ['hugo', 'blog', 'tutorial', 'static-site']
description: "Hướng dẫn chi tiết cách thiết lập blog cá nhân sử dụng Hugo và theme PaperMod"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowCodeCopyButtons: true
toc: true
---

# Hướng dẫn thiết lập blog với Hugo và PaperMod

Hugo là một static site generator mạnh mẽ và nhanh chóng, rất phù hợp để tạo blog cá nhân. Trong bài viết này, tôi sẽ hướng dẫn bạn cách thiết lập một blog đẹp mắt với Hugo và theme PaperMod.

## Tại sao chọn Hugo?

- **Tốc độ**: Hugo build site rất nhanh, chỉ mất vài giây
- **Đơn giản**: Cú pháp Markdown dễ học
- **Linh hoạt**: Nhiều theme đẹp và có thể tùy chỉnh
- **SEO tốt**: Tạo ra HTML tối ưu cho search engine

## Cài đặt Hugo

### Windows
```bash
# Sử dụng Chocolatey
choco install hugo

# Hoặc tải từ GitHub
# https://github.com/gohugoio/hugo/releases
```

### macOS
```bash
# Sử dụng Homebrew
brew install hugo
```

### Linux
```bash
# Ubuntu/Debian
sudo apt install hugo

# Hoặc sử dụng snap
sudo snap install hugo
```

## Tạo site mới

```bash
# Tạo site mới
hugo new site my-blog

# Di chuyển vào thư mục
cd my-blog

# Khởi tạo git repository
git init
```

## Cài đặt theme PaperMod

```bash
# Thêm theme như submodule
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod

# Cấu hình theme trong hugo.toml
echo 'theme = "PaperMod"' >> hugo.toml
```

## Cấu hình cơ bản

Tạo file `hugo.toml` với nội dung:

```toml
baseURL = 'https://your-domain.com/'
languageCode = 'vi'
title = 'My Blog'
theme = 'PaperMod'

[params]
  env = 'production'
  title = 'My Blog'
  description = 'Mô tả blog của bạn'
  author = 'Tên của bạn'
  keywords = ['blog', 'programming', 'tech']

[menu]
  [[menu.main]]
    identifier = 'home'
    name = 'Trang chủ'
    url = '/'
    weight = 10
  [[menu.main]]
    identifier = 'posts'
    name = 'Bài viết'
    url = '/posts/'
    weight = 20
```

## Tạo bài viết đầu tiên

```bash
# Tạo bài viết mới
hugo new posts/bai-viet-dau-tien.md
```

## Chạy server development

```bash
# Chạy server local
hugo server -D

# Truy cập http://localhost:1313
```

## Deploy lên GitHub Pages

1. Tạo repository trên GitHub
2. Cấu hình GitHub Actions
3. Push code lên repository
4. Site sẽ được build và deploy tự động

## Kết luận

Hugo với PaperMod là một combo tuyệt vời để tạo blog cá nhân. Nó nhanh, đẹp và dễ sử dụng. Bạn có thể bắt đầu viết blog ngay sau khi thiết lập xong!

Chúc bạn thành công với blog mới! 🚀
