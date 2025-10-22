---
title: "JavaScript Async Programming - Callbacks, Promises và Async/Await"
date: 2024-01-06T12:00:00+07:00
draft: false
tags: ['javascript', 'async', 'promises', 'async-await', 'callbacks']
description: "Hướng dẫn chi tiết về lập trình bất đồng bộ trong JavaScript từ cơ bản đến nâng cao"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowCodeCopyButtons: true
toc: true
---

# JavaScript Async Programming - Callbacks, Promises và Async/Await

Lập trình bất đồng bộ (asynchronous programming) là một khái niệm quan trọng trong JavaScript. Trong bài viết này, tôi sẽ hướng dẫn bạn từ Callbacks đến Promises và Async/Await.

## Tại Sao Cần Async Programming?

JavaScript là single-threaded, nghĩa là chỉ có thể thực hiện một tác vụ tại một thời điểm. Để xử lý các tác vụ mất thời gian (như gọi API, đọc file), chúng ta cần async programming để không block UI.

## Callbacks

### 1. Callback Cơ Bản

```javascript
function fetchData(callback) {
    setTimeout(() => {
        const data = { name: 'John', age: 30 };
        callback(data);
    }, 1000);
}

function displayData(data) {
    console.log('Data received:', data);
}

fetchData(displayData);
```

### 2. Callback Hell

```javascript
// ❌ Callback Hell - Khó đọc và maintain
getUser(1, (user) => {
    console.log('User:', user);
    
    getPosts(user.id, (posts) => {
        console.log('Posts:', posts);
        
        getComments(posts[0].id, (comments) => {
            console.log('Comments:', comments);
            
            // Nested callbacks...
        });
    });
});
```

### 3. Error Handling với Callbacks

```javascript
function fetchData(successCallback, errorCallback) {
    setTimeout(() => {
        const random = Math.random();
        if (random > 0.5) {
            successCallback({ data: 'Success!' });
        } else {
            errorCallback(new Error('Something went wrong!'));
        }
    }, 1000);
}

fetchData(
    (data) => console.log('Success:', data),
    (error) => console.error('Error:', error.message)
);
```

## Promises

### 1. Tạo Promise

```javascript
function fetchData() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const random = Math.random();
            if (random > 0.5) {
                resolve({ data: 'Success!' });
            } else {
                reject(new Error('Something went wrong!'));
            }
        }, 1000);
    });
}

// Sử dụng Promise
fetchData()
    .then(data => console.log('Success:', data))
    .catch(error => console.error('Error:', error.message));
```

### 2. Promise Chaining

```javascript
function getUser(id) {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve({ id, name: 'John Doe', email: 'john@example.com' });
        }, 500);
    });
}

function getPosts(userId) {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve([
                { id: 1, title: 'Post 1', userId },
                { id: 2, title: 'Post 2', userId }
            ]);
        }, 500);
    });
}

function getComments(postId) {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve([
                { id: 1, text: 'Great post!', postId },
                { id: 2, text: 'Thanks for sharing', postId }
            ]);
        }, 500);
    });
}

// Promise chaining
getUser(1)
    .then(user => {
        console.log('User:', user);
        return getPosts(user.id);
    })
    .then(posts => {
        console.log('Posts:', posts);
        return getComments(posts[0].id);
    })
    .then(comments => {
        console.log('Comments:', comments);
    })
    .catch(error => {
        console.error('Error:', error);
    });
```

### 3. Promise.all và Promise.allSettled

```javascript
// Promise.all - Tất cả phải thành công
Promise.all([
    fetchData('/api/users'),
    fetchData('/api/posts'),
    fetchData('/api/comments')
])
.then(([users, posts, comments]) => {
    console.log('All data loaded:', { users, posts, comments });
})
.catch(error => {
    console.error('One or more requests failed:', error);
});

// Promise.allSettled - Chờ tất cả hoàn thành (thành công hoặc thất bại)
Promise.allSettled([
    fetchData('/api/users'),
    fetchData('/api/posts'),
    fetchData('/api/comments')
])
.then(results => {
    results.forEach((result, index) => {
        if (result.status === 'fulfilled') {
            console.log(`Request ${index} succeeded:`, result.value);
        } else {
            console.log(`Request ${index} failed:`, result.reason);
        }
    });
});
```

### 4. Promise.race

```javascript
function timeout(ms) {
    return new Promise((_, reject) => {
        setTimeout(() => reject(new Error('Timeout')), ms);
    });
}

function fetchData() {
    return new Promise(resolve => {
        setTimeout(() => resolve({ data: 'Success!' }), 2000);
    });
}

// Lấy kết quả của promise nào hoàn thành trước
Promise.race([
    fetchData(),
    timeout(3000)
])
.then(data => console.log('Data received:', data))
.catch(error => console.error('Error:', error.message));
```

## Async/Await

### 1. Cú pháp Cơ Bản

```javascript
async function fetchData() {
    try {
        const response = await fetch('/api/data');
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Error fetching data:', error);
        throw error;
    }
}

// Sử dụng async function
async function main() {
    try {
        const data = await fetchData();
        console.log('Data:', data);
    } catch (error) {
        console.error('Error:', error);
    }
}

main();
```

### 2. Xử Lý Nhiều Async Operations

```javascript
// Sequential - Tuần tự
async function sequentialExample() {
    const user = await getUser(1);
    console.log('User:', user);
    
    const posts = await getPosts(user.id);
    console.log('Posts:', posts);
    
    const comments = await getComments(posts[0].id);
    console.log('Comments:', comments);
}

// Parallel - Song song
async function parallelExample() {
    const [user, posts, comments] = await Promise.all([
        getUser(1),
        getPosts(1),
        getComments(1)
    ]);
    
    console.log('All data:', { user, posts, comments });
}

// Mixed - Hỗn hợp
async function mixedExample() {
    const user = await getUser(1);
    
    const [posts, comments] = await Promise.all([
        getPosts(user.id),
        getComments(user.id)
    ]);
    
    console.log('User with posts and comments:', { user, posts, comments });
}
```

### 3. Error Handling

```javascript
async function fetchDataWithRetry(url, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            const response = await fetch(url);
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            return await response.json();
        } catch (error) {
            console.log(`Attempt ${i + 1} failed:`, error.message);
            
            if (i === maxRetries - 1) {
                throw new Error(`Failed after ${maxRetries} attempts`);
            }
            
            // Wait before retry
            await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
        }
    }
}

// Sử dụng
fetchDataWithRetry('/api/data')
    .then(data => console.log('Success:', data))
    .catch(error => console.error('Failed:', error.message));
```

## Advanced Patterns

### 1. Async Generator

```javascript
async function* fetchPages(url) {
    let page = 1;
    let hasMore = true;
    
    while (hasMore) {
        try {
            const response = await fetch(`${url}?page=${page}`);
            const data = await response.json();
            
            yield data;
            
            hasMore = data.hasMore;
            page++;
        } catch (error) {
            console.error('Error fetching page:', error);
            break;
        }
    }
}

// Sử dụng async generator
async function processAllPages() {
    for await (const page of fetchPages('/api/posts')) {
        console.log('Processing page:', page);
        // Process each page
    }
}
```

### 2. Async Queue

```javascript
class AsyncQueue {
    constructor(concurrency = 1) {
        this.concurrency = concurrency;
        this.running = 0;
        this.queue = [];
    }
    
    async add(task) {
        return new Promise((resolve, reject) => {
            this.queue.push({ task, resolve, reject });
            this.process();
        });
    }
    
    async process() {
        if (this.running >= this.concurrency || this.queue.length === 0) {
            return;
        }
        
        this.running++;
        const { task, resolve, reject } = this.queue.shift();
        
        try {
            const result = await task();
            resolve(result);
        } catch (error) {
            reject(error);
        } finally {
            this.running--;
            this.process();
        }
    }
}

// Sử dụng
const queue = new AsyncQueue(2); // Max 2 concurrent tasks

async function processTasks() {
    const tasks = Array.from({ length: 10 }, (_, i) => () => 
        new Promise(resolve => {
            setTimeout(() => resolve(`Task ${i} completed`), Math.random() * 1000);
        })
    );
    
    const results = await Promise.all(
        tasks.map(task => queue.add(task))
    );
    
    console.log('All tasks completed:', results);
}
```

### 3. Debounce và Throttle

```javascript
// Debounce - Chỉ thực hiện sau khi dừng một khoảng thời gian
function debounce(func, delay) {
    let timeoutId;
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => func.apply(this, args), delay);
    };
}

// Throttle - Thực hiện tối đa một lần trong khoảng thời gian
function throttle(func, delay) {
    let lastCall = 0;
    return function(...args) {
        const now = Date.now();
        if (now - lastCall >= delay) {
            lastCall = now;
            return func.apply(this, args);
        }
    };
}

// Sử dụng
const debouncedSearch = debounce(async (query) => {
    if (query.length < 2) return;
    
    try {
        const results = await searchAPI(query);
        displayResults(results);
    } catch (error) {
        console.error('Search error:', error);
    }
}, 300);

const throttledScroll = throttle(() => {
    console.log('Scroll event');
}, 100);
```

## Best Practices

### 1. Luôn Sử Dụng try-catch

```javascript
// ❌ Không tốt
async function badExample() {
    const data = await fetchData(); // Có thể throw error
    return data.process();
}

// ✅ Tốt
async function goodExample() {
    try {
        const data = await fetchData();
        return data.process();
    } catch (error) {
        console.error('Error processing data:', error);
        throw error; // Re-throw nếu cần
    }
}
```

### 2. Sử Dụng Promise.all Khi Có Thể

```javascript
// ❌ Sequential - Chậm
async function slowExample() {
    const user = await getUser(1);
    const posts = await getPosts(1);
    const comments = await getComments(1);
    return { user, posts, comments };
}

// ✅ Parallel - Nhanh
async function fastExample() {
    const [user, posts, comments] = await Promise.all([
        getUser(1),
        getPosts(1),
        getComments(1)
    ]);
    return { user, posts, comments };
}
```

### 3. Xử Lý Timeout

```javascript
function withTimeout(promise, timeoutMs) {
    const timeoutPromise = new Promise((_, reject) => {
        setTimeout(() => reject(new Error('Timeout')), timeoutMs);
    });
    
    return Promise.race([promise, timeoutPromise]);
}

// Sử dụng
async function fetchDataWithTimeout() {
    try {
        const data = await withTimeout(fetchData(), 5000);
        return data;
    } catch (error) {
        if (error.message === 'Timeout') {
            console.error('Request timed out');
        } else {
            console.error('Other error:', error);
        }
        throw error;
    }
}
```

## Kết Luận

Async programming trong JavaScript đã phát triển từ Callbacks → Promises → Async/Await. Mỗi approach có ưu nhược điểm riêng:

- **Callbacks**: Cơ bản nhưng dễ tạo callback hell
- **Promises**: Tốt hơn callbacks, hỗ trợ chaining
- **Async/Await**: Dễ đọc nhất, giống synchronous code

Với những kiến thức này, bạn có thể:
- Xử lý các tác vụ bất đồng bộ hiệu quả
- Tránh callback hell
- Sử dụng async/await một cách chính xác
- Implement các pattern nâng cao

Trong bài viết tiếp theo, tôi sẽ hướng dẫn về testing trong JavaScript! 🚀


