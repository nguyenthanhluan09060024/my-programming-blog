---
title: "JavaScript ES6+ - Những Tính Năng Quan Trọng Nhất"
date: 2024-01-18T14:30:00+07:00
draft: false
tags: ['javascript', 'es6', 'modern-js', 'programming']
description: "Tổng hợp các tính năng quan trọng nhất của JavaScript ES6+ mà mọi developer nên biết"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowCodeCopyButtons: true
toc: true
---

# JavaScript ES6+ - Những Tính Năng Quan Trọng Nhất

ES6 (ECMAScript 2015) đã mang đến một cuộc cách mạng cho JavaScript với nhiều tính năng mới mạnh mẽ. Trong bài viết này, tôi sẽ giới thiệu những tính năng quan trọng nhất mà mọi developer JavaScript nên nắm vững.

## 1. Arrow Functions

Arrow functions cung cấp cú pháp ngắn gọn hơn cho việc viết functions.

### Cú pháp cũ:
```javascript
function add(a, b) {
    return a + b;
}

const multiply = function(a, b) {
    return a * b;
};
```

### Cú pháp mới:
```javascript
const add = (a, b) => a + b;
const multiply = (a, b) => {
    return a * b;
};

// Với một tham số
const square = x => x * x;

// Không có tham số
const greet = () => console.log('Hello!');
```

### Lưu ý về `this`:
```javascript
class Person {
    constructor(name) {
        this.name = name;
    }
    
    // Arrow function giữ nguyên context của this
    greetArrow = () => {
        console.log(`Hello, I'm ${this.name}`);
    }
    
    // Regular function có this riêng
    greetRegular() {
        console.log(`Hello, I'm ${this.name}`);
    }
}
```

## 2. Destructuring Assignment

Destructuring cho phép giải nén các giá trị từ arrays hoặc objects.

### Array Destructuring:
```javascript
const numbers = [1, 2, 3, 4, 5];

// Cú pháp cũ
const first = numbers[0];
const second = numbers[1];

// Cú pháp mới
const [first, second, ...rest] = numbers;
console.log(first);  // 1
console.log(second); // 2
console.log(rest);   // [3, 4, 5]

// Với default values
const [a, b, c = 10] = [1, 2];
console.log(c); // 10

// Hoán đổi giá trị
let x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y); // 2, 1
```

### Object Destructuring:
```javascript
const person = {
    name: 'Nguyễn Văn A',
    age: 25,
    city: 'Hà Nội',
    country: 'Việt Nam'
};

// Cú pháp cũ
const name = person.name;
const age = person.age;

// Cú pháp mới
const { name, age, city } = person;
console.log(name, age, city); // Nguyễn Văn A, 25, Hà Nội

// Với default values
const { name, age, salary = 0 } = person;

// Rename variables
const { name: fullName, age: years } = person;

// Nested destructuring
const user = {
    profile: {
        name: 'John',
        address: {
            city: 'Hanoi',
            country: 'Vietnam'
        }
    }
};

const { profile: { name, address: { city } } } = user;
```

## 3. Template Literals

Template literals cung cấp cú pháp mới để tạo strings với khả năng nhúng biểu thức.

```javascript
const name = 'Nguyễn Văn A';
const age = 25;

// Cú pháp cũ
const message = 'Xin chào, tôi là ' + name + ', năm nay ' + age + ' tuổi.';

// Cú pháp mới
const message = `Xin chào, tôi là ${name}, năm nay ${age} tuổi.`;

// Multi-line strings
const html = `
    <div class="user-card">
        <h2>${name}</h2>
        <p>Tuổi: ${age}</p>
        <p>Thời gian: ${new Date().toLocaleString()}</p>
    </div>
`;

// Tagged templates
function highlight(strings, ...values) {
    return strings.reduce((result, string, i) => {
        const value = values[i] ? `<mark>${values[i]}</mark>` : '';
        return result + string + value;
    }, '');
}

const name = 'JavaScript';
const version = 'ES6';
const highlighted = highlight`Tôi đang học ${name} ${version}`;
// Kết quả: "Tôi đang học <mark>JavaScript</mark> <mark>ES6</mark>"
```

## 4. Spread Operator và Rest Parameters

### Spread Operator:
```javascript
// Arrays
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// Objects
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };
const merged = { ...obj1, ...obj2 }; // { a: 1, b: 2, c: 3, d: 4 }

// Function calls
const numbers = [1, 2, 3, 4, 5];
const max = Math.max(...numbers); // 5

// Copy arrays/objects
const originalArray = [1, 2, 3];
const copiedArray = [...originalArray];

const originalObject = { x: 1, y: 2 };
const copiedObject = { ...originalObject };
```

### Rest Parameters:
```javascript
// Thu thập tất cả arguments thành một array
function sum(...numbers) {
    return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3, 4, 5)); // 15

// Kết hợp với destructuring
function greet(firstName, lastName, ...titles) {
    console.log(`Xin chào ${firstName} ${lastName}`);
    console.log('Danh hiệu:', titles);
}

greet('Nguyễn', 'Văn A', 'Tiến sĩ', 'Giáo sư');
```

## 5. Classes

ES6 giới thiệu cú pháp class cho JavaScript.

```javascript
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    
    // Method
    greet() {
        return `Xin chào, tôi là ${this.name}`;
    }
    
    // Getter
    get info() {
        return `${this.name} (${this.age} tuổi)`;
    }
    
    // Setter
    set info(value) {
        [this.name, this.age] = value.split(' ');
    }
    
    // Static method
    static createAdult(name) {
        return new Person(name, 18);
    }
}

// Inheritance
class Student extends Person {
    constructor(name, age, school) {
        super(name, age);
        this.school = school;
    }
    
    study() {
        return `${this.name} đang học tại ${this.school}`;
    }
    
    // Override method
    greet() {
        return `${super.greet()}, tôi là sinh viên`;
    }
}

// Sử dụng
const student = new Student('Nguyễn Văn B', 20, 'Đại học Bách Khoa');
console.log(student.greet()); // Xin chào, tôi là Nguyễn Văn B, tôi là sinh viên
console.log(student.study()); // Nguyễn Văn B đang học tại Đại học Bách Khoa
```

## 6. Modules (Import/Export)

### Export:
```javascript
// math.js
export const PI = 3.14159;

export function add(a, b) {
    return a + b;
}

export function multiply(a, b) {
    return a * b;
}

// Default export
export default class Calculator {
    constructor() {
        this.result = 0;
    }
    
    calculate(operation, a, b) {
        switch(operation) {
            case 'add': return a + b;
            case 'multiply': return a * b;
            default: return 0;
        }
    }
}
```

### Import:
```javascript
// main.js
import Calculator, { PI, add, multiply } from './math.js';

// Sử dụng
console.log(PI); // 3.14159
console.log(add(2, 3)); // 5

const calc = new Calculator();
console.log(calc.calculate('add', 5, 3)); // 8

// Import tất cả
import * as MathUtils from './math.js';
console.log(MathUtils.PI);
```

## 7. Promises và Async/Await

### Promises:
```javascript
function fetchData(url) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (url) {
                resolve({ data: 'Dữ liệu từ server', url });
            } else {
                reject(new Error('URL không hợp lệ'));
            }
        }, 1000);
    });
}

// Sử dụng Promise
fetchData('https://api.example.com')
    .then(response => {
        console.log('Thành công:', response);
        return fetchData('https://api.example.com/users');
    })
    .then(userData => {
        console.log('Dữ liệu user:', userData);
    })
    .catch(error => {
        console.error('Lỗi:', error.message);
    });
```

### Async/Await:
```javascript
async function loadData() {
    try {
        const response = await fetchData('https://api.example.com');
        console.log('Thành công:', response);
        
        const userData = await fetchData('https://api.example.com/users');
        console.log('Dữ liệu user:', userData);
        
        return { response, userData };
    } catch (error) {
        console.error('Lỗi:', error.message);
        throw error;
    }
}

// Sử dụng
loadData()
    .then(result => console.log('Hoàn thành:', result))
    .catch(error => console.error('Lỗi cuối:', error));
```

## 8. Map và Set

### Map:
```javascript
const userMap = new Map();

// Thêm dữ liệu
userMap.set('user1', { name: 'Nguyễn Văn A', age: 25 });
userMap.set('user2', { name: 'Trần Thị B', age: 30 });

// Lấy dữ liệu
console.log(userMap.get('user1')); // { name: 'Nguyễn Văn A', age: 25 }

// Kiểm tra tồn tại
console.log(userMap.has('user1')); // true

// Xóa
userMap.delete('user1');

// Kích thước
console.log(userMap.size); // 1

// Iterate
for (const [key, value] of userMap) {
    console.log(key, value);
}
```

### Set:
```javascript
const uniqueNumbers = new Set([1, 2, 3, 3, 4, 4, 5]);
console.log(uniqueNumbers); // Set { 1, 2, 3, 4, 5 }

// Thêm phần tử
uniqueNumbers.add(6);

// Kiểm tra tồn tại
console.log(uniqueNumbers.has(3)); // true

// Xóa
uniqueNumbers.delete(3);

// Kích thước
console.log(uniqueNumbers.size); // 5

// Convert to Array
const array = [...uniqueNumbers];
```

## Kết Luận

ES6+ đã làm cho JavaScript trở nên mạnh mẽ và dễ sử dụng hơn rất nhiều. Những tính năng này giúp code trở nên:

- **Ngắn gọn hơn**: Arrow functions, destructuring
- **Dễ đọc hơn**: Template literals, classes
- **Mạnh mẽ hơn**: Promises, async/await
- **Linh hoạt hơn**: Modules, spread operator

Hãy thực hành những tính năng này trong các dự án của bạn để nắm vững chúng. Trong bài viết tiếp theo, tôi sẽ hướng dẫn về React và cách sử dụng những tính năng ES6+ này trong thực tế! 🚀


