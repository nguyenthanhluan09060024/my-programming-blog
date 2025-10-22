---
title: "10 JavaScript Tips Hữu Ích Cho Năm 2024"
date: 2024-01-10T14:30:00+07:00
draft: false
tags: ['javascript', 'tips', 'programming', 'es6']
description: "Tổng hợp 10 mẹo JavaScript hữu ích mà mọi developer nên biết"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowCodeCopyButtons: true
toc: true
---

# 10 JavaScript Tips Hữu Ích Cho Năm 2024

JavaScript đã phát triển rất nhiều trong những năm gần đây. Dưới đây là 10 tips hữu ích mà tôi đã học được và muốn chia sẻ với các bạn.

## 1. Optional Chaining (?. )

Thay vì kiểm tra null/undefined một cách dài dòng:

```javascript
// Cách cũ
if (user && user.address && user.address.street) {
  console.log(user.address.street);
}

// Cách mới với optional chaining
console.log(user?.address?.street);
```

## 2. Nullish Coalescing (??)

Sử dụng `??` thay vì `||` khi muốn kiểm tra null/undefined:

```javascript
// Cách cũ
const name = user.name || 'Anonymous';

// Cách mới - chỉ fallback khi null/undefined
const name = user.name ?? 'Anonymous';
```

## 3. Array Destructuring với Default Values

```javascript
const [first, second = 'default', ...rest] = array;
```

## 4. Object Destructuring với Renaming

```javascript
const { name: userName, age: userAge } = user;
```

## 5. Template Literals cho Multi-line Strings

```javascript
const html = `
  <div class="container">
    <h1>${title}</h1>
    <p>${description}</p>
  </div>
`;
```

## 6. Array Methods Chain

```javascript
const result = users
  .filter(user => user.active)
  .map(user => user.name)
  .sort()
  .join(', ');
```

## 7. Dynamic Property Names

```javascript
const property = 'name';
const user = {
  [property]: 'John',
  [`${property}Length`]: 'John'.length
};
```

## 8. Promise.allSettled()

Chờ tất cả promises hoàn thành (thành công hoặc thất bại):

```javascript
const promises = [promise1, promise2, promise3];
const results = await Promise.allSettled(promises);
```

## 9. Array.from() với Mapping

```javascript
const numbers = Array.from({ length: 5 }, (_, i) => i * 2);
// [0, 2, 4, 6, 8]
```

## 10. Logical Assignment Operators

```javascript
// Thay vì
user.name = user.name || 'Anonymous';

// Có thể viết
user.name ||= 'Anonymous';

// Tương tự với && và ??
user.name &&= user.name.toUpperCase();
user.count ??= 0;
```

## Kết luận

Những tips này sẽ giúp code JavaScript của bạn ngắn gọn và dễ đọc hơn. Hãy thử áp dụng chúng vào dự án của bạn!

Bạn có tip nào khác muốn chia sẻ không? Hãy comment bên dưới nhé! 😊
