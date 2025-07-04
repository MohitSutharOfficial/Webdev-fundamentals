# Final Project: Todo Application with Local Storage

## 📋 Project Overview

Build a fully functional todo application that demonstrates your mastery of HTML, CSS, and JavaScript. This project combines semantic HTML structure, responsive CSS styling, and interactive JavaScript functionality.

## 🎯 Learning Objectives

- Apply semantic HTML structure
- Implement responsive CSS design
- Use JavaScript for dynamic interactions
- Work with local storage for data persistence
- Handle form validation and user input
- Create smooth animations and transitions

## 🚀 Features to Implement

### Core Features (MVP)
- ✅ Add new tasks
- ✅ Mark tasks as complete/incomplete
- ✅ Delete individual tasks
- ✅ Edit existing tasks
- ✅ Filter tasks (All, Active, Completed)
- ✅ Clear all completed tasks
- ✅ Task counter display

### Advanced Features
- 🔄 Data persistence with localStorage
- 📱 Responsive design for mobile/tablet
- 🎨 Dark/light theme toggle
- ⌨️ Keyboard shortcuts
- 🔍 Search functionality
- 📅 Due dates for tasks
- 🏷️ Task categories/tags
- ↕️ Drag and drop reordering

## 📁 Project Structure

```
02_todo_app/
├── index.html          # Main HTML structure
├── css/
│   ├── styles.css       # Main stylesheet
│   ├── components.css   # Component-specific styles
│   └── themes.css       # Theme variations
├── js/
│   ├── app.js           # Main application logic
│   ├── storage.js       # Local storage utilities
│   ├── ui.js            # UI manipulation functions
│   └── utils.js         # Utility functions
├── assets/
│   └── icons/           # SVG icons for the app
├── README.md            # Project documentation
└── screenshots/         # App screenshots
```

## 🛠 Technical Requirements

### HTML Requirements
- Use semantic HTML5 elements
- Proper form structure with labels
- Accessible markup with ARIA attributes
- Valid HTML (pass W3C validation)

### CSS Requirements
- Mobile-first responsive design
- CSS Grid or Flexbox for layouts
- CSS custom properties (variables)
- Smooth transitions and animations
- BEM naming convention for CSS classes

### JavaScript Requirements
- ES6+ modern JavaScript features
- Modular code organization
- Event delegation for dynamic content
- Local storage for data persistence
- Input validation and error handling

## 📝 Implementation Guide

### Step 1: HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Todo App</title>
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
    <div class="app-container">
        <header class="app-header">
            <h1>My Todo App</h1>
            <button class="theme-toggle" aria-label="Toggle dark mode">🌙</button>
        </header>
        
        <main class="app-main">
            <form class="todo-form">
                <input type="text" class="todo-input" placeholder="What needs to be done?" required>
                <button type="submit" class="add-btn">Add Task</button>
            </form>
            
            <section class="todo-section">
                <div class="todo-filters">
                    <button class="filter-btn active" data-filter="all">All</button>
                    <button class="filter-btn" data-filter="active">Active</button>
                    <button class="filter-btn" data-filter="completed">Completed</button>
                </div>
                
                <ul class="todo-list" id="todo-list">
                    <!-- Dynamic content -->
                </ul>
                
                <div class="todo-footer">
                    <span class="todo-count">0 items left</span>
                    <button class="clear-completed">Clear completed</button>
                </div>
            </section>
        </main>
    </div>
    
    <script src="js/app.js"></script>
</body>
</html>
```

### Step 2: CSS Styling

```css
/* CSS Custom Properties */
:root {
    --primary-color: #4a90e2;
    --secondary-color: #f5f5f5;
    --text-color: #333;
    --background-color: #fff;
    --border-color: #ddd;
    --success-color: #28a745;
    --danger-color: #dc3545;
    --warning-color: #ffc107;
    
    --font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    --border-radius: 8px;
    --transition: all 0.3s ease;
    --shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

/* Dark theme variables */
[data-theme="dark"] {
    --text-color: #e0e0e0;
    --background-color: #1a1a1a;
    --secondary-color: #2d2d2d;
    --border-color: #404040;
}

/* Base styles */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: var(--font-family);
    background-color: var(--background-color);
    color: var(--text-color);
    transition: var(--transition);
}

/* Component styles */
.app-container {
    max-width: 600px;
    margin: 0 auto;
    padding: 20px;
}

.todo-item {
    display: flex;
    align-items: center;
    padding: 15px;
    border-bottom: 1px solid var(--border-color);
    transition: var(--transition);
}

.todo-item:hover {
    background-color: var(--secondary-color);
}

.todo-item.completed {
    opacity: 0.6;
    text-decoration: line-through;
}

/* Responsive design */
@media (max-width: 768px) {
    .app-container {
        padding: 10px;
    }
    
    .todo-filters {
        flex-direction: column;
    }
}
```

### Step 3: JavaScript Functionality

```javascript
// app.js - Main application logic
class TodoApp {
    constructor() {
        this.todos = JSON.parse(localStorage.getItem('todos')) || [];
        this.filter = 'all';
        this.init();
    }
    
    init() {
        this.bindEvents();
        this.render();
    }
    
    bindEvents() {
        // Form submission
        document.querySelector('.todo-form').addEventListener('submit', (e) => {
            e.preventDefault();
            this.addTodo();
        });
        
        // Event delegation for todo actions
        document.querySelector('.todo-list').addEventListener('click', (e) => {
            if (e.target.matches('.delete-btn')) {
                this.deleteTodo(e.target.dataset.id);
            } else if (e.target.matches('.toggle-btn')) {
                this.toggleTodo(e.target.dataset.id);
            }
        });
        
        // Filter buttons
        document.querySelectorAll('.filter-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                this.setFilter(e.target.dataset.filter);
            });
        });
    }
    
    addTodo() {
        const input = document.querySelector('.todo-input');
        const text = input.value.trim();
        
        if (!text) return;
        
        const todo = {
            id: Date.now().toString(),
            text: text,
            completed: false,
            createdAt: new Date().toISOString()
        };
        
        this.todos.push(todo);
        this.saveTodos();
        this.render();
        input.value = '';
    }
    
    toggleTodo(id) {
        const todo = this.todos.find(t => t.id === id);
        if (todo) {
            todo.completed = !todo.completed;
            this.saveTodos();
            this.render();
        }
    }
    
    deleteTodo(id) {
        this.todos = this.todos.filter(t => t.id !== id);
        this.saveTodos();
        this.render();
    }
    
    setFilter(filter) {
        this.filter = filter;
        this.render();
        
        // Update active filter button
        document.querySelectorAll('.filter-btn').forEach(btn => {
            btn.classList.toggle('active', btn.dataset.filter === filter);
        });
    }
    
    getFilteredTodos() {
        switch (this.filter) {
            case 'active':
                return this.todos.filter(todo => !todo.completed);
            case 'completed':
                return this.todos.filter(todo => todo.completed);
            default:
                return this.todos;
        }
    }
    
    render() {
        const todoList = document.querySelector('.todo-list');
        const filteredTodos = this.getFilteredTodos();
        
        todoList.innerHTML = filteredTodos.map(todo => `
            <li class="todo-item ${todo.completed ? 'completed' : ''}">
                <button class="toggle-btn" data-id="${todo.id}">
                    ${todo.completed ? '✓' : '○'}
                </button>
                <span class="todo-text">${todo.text}</span>
                <button class="delete-btn" data-id="${todo.id}">×</button>
            </li>
        `).join('');
        
        this.updateCounter();
    }
    
    updateCounter() {
        const activeTodos = this.todos.filter(todo => !todo.completed).length;
        const counter = document.querySelector('.todo-count');
        counter.textContent = `${activeTodos} item${activeTodos !== 1 ? 's' : ''} left`;
    }
    
    saveTodos() {
        localStorage.setItem('todos', JSON.stringify(this.todos));
    }
}

// Initialize the app
document.addEventListener('DOMContentLoaded', () => {
    new TodoApp();
});
```

## 🎨 Design Guidelines

### Color Scheme
- Primary: #4a90e2 (Blue)
- Success: #28a745 (Green)
- Danger: #dc3545 (Red)
- Warning: #ffc107 (Yellow)
- Neutral: #6c757d (Gray)

### Typography
- Font Family: Segoe UI, system fonts
- Heading: 24px, bold
- Body: 16px, normal
- Small text: 14px

### Spacing
- Base unit: 8px
- Small: 8px
- Medium: 16px
- Large: 24px
- Extra large: 32px

## ✅ Testing Checklist

### Functionality Testing
- [ ] Can add new tasks
- [ ] Can mark tasks as complete
- [ ] Can delete tasks
- [ ] Can edit tasks (if implemented)
- [ ] Filters work correctly
- [ ] Counter updates properly
- [ ] Data persists after page reload
- [ ] Form validation works

### UI/UX Testing
- [ ] Responsive on mobile devices
- [ ] Smooth animations and transitions
- [ ] Accessible with keyboard navigation
- [ ] Clear visual feedback for interactions
- [ ] Intuitive user interface

### Browser Testing
- [ ] Works in Chrome
- [ ] Works in Firefox
- [ ] Works in Safari
- [ ] Works in Edge
- [ ] No console errors

## 🚀 Deployment Options

### GitHub Pages
1. Push code to GitHub repository
2. Enable GitHub Pages in repository settings
3. Choose source branch (main/master)
4. Access your app at username.github.io/repository-name

### Netlify
1. Connect GitHub repository to Netlify
2. Configure build settings (if needed)
3. Deploy automatically on git push

### Vercel
1. Import project from GitHub
2. Automatic deployment and custom domain options

## 📚 Learning Resources

### Tutorials
- [MDN Todo App Tutorial](https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Client-side_JavaScript_frameworks/Vanilla_JS_todo_list)
- [JavaScript30 by Wes Bos](https://javascript30.com/)

### Documentation
- [Web APIs - Local Storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [CSS Grid Guide](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [JavaScript Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)

## 🎯 Next Steps

After completing this project:
1. Add more advanced features (search, categories, due dates)
2. Implement a backend with Node.js and Express
3. Add user authentication
4. Convert to a Progressive Web App (PWA)
5. Use a frontend framework (React, Vue, Angular)

## 💡 Senior Engineer Tips

- **Code Organization:** Keep JavaScript modular and organized
- **Performance:** Use event delegation for better performance
- **Accessibility:** Test with screen readers and keyboard navigation
- **Error Handling:** Handle edge cases and user errors gracefully
- **Progressive Enhancement:** Ensure basic functionality works without JavaScript
- **Security:** Sanitize user input and validate data
- **Testing:** Write unit tests for your functions
- **Documentation:** Comment your code and write good README files

---

**Happy coding!** This project will give you hands-on experience with all the core web development technologies and prepare you for more advanced topics.
