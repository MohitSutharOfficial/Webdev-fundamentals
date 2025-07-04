# JavaScript Senior Engineer Tactics

Welcome to the JavaScript Senior Engineer Tactics section! This is where we explore the advanced patterns, architectural decisions, and leadership principles that distinguish senior JavaScript engineers.

## 🎯 Senior Mindset

As a senior JavaScript engineer, you think beyond just writing code. You consider:
- **System Architecture**: How your code fits into the larger system
- **Team Dynamics**: Mentoring, code reviews, and knowledge sharing
- **Business Impact**: Understanding how technical decisions affect business outcomes
- **Long-term Maintainability**: Code that evolves with changing requirements
- **Performance & Scalability**: Building for current and future scale
- **Developer Experience**: Creating tools and patterns that make the team more productive

## 🏗️ Advanced Architectural Patterns

### 1. Modular Architecture & Dependency Management
```javascript
// Advanced module pattern with dependency injection
class ModuleManager {
    constructor() {
        this.modules = new Map();
        this.dependencies = new Map();
    }
    
    register(name, moduleFactory, dependencies = []) {
        this.dependencies.set(name, dependencies);
        this.modules.set(name, {
            factory: moduleFactory,
            instance: null,
            loading: false
        });
    }
    
    async get(name) {
        const module = this.modules.get(name);
        if (!module) throw new Error(`Module '${name}' not found`);
        
        if (module.instance) return module.instance;
        if (module.loading) {
            // Wait for loading to complete
            return new Promise(resolve => {
                const check = () => {
                    if (module.instance) resolve(module.instance);
                    else setTimeout(check, 10);
                };
                check();
            });
        }
        
        module.loading = true;
        
        // Resolve dependencies
        const deps = this.dependencies.get(name) || [];
        const resolvedDeps = await Promise.all(
            deps.map(dep => this.get(dep))
        );
        
        // Create instance
        module.instance = await module.factory(...resolvedDeps);
        module.loading = false;
        
        return module.instance;
    }
}

// Usage
const moduleManager = new ModuleManager();

moduleManager.register('logger', () => ({
    log: (message) => console.log(`[${new Date().toISOString()}] ${message}`)
}));

moduleManager.register('api', (logger) => ({
    async fetch(url) {
        logger.log(`Fetching: ${url}`);
        return fetch(url);
    }
}), ['logger']);

// Advanced error boundary pattern
class ErrorBoundary {
    constructor() {
        this.errorHandlers = new Map();
        this.setupGlobalHandlers();
    }
    
    setupGlobalHandlers() {
        window.addEventListener('error', this.handleError.bind(this));
        window.addEventListener('unhandledrejection', this.handlePromiseRejection.bind(this));
    }
    
    register(context, handler) {
        this.errorHandlers.set(context, handler);
    }
    
    handleError(event) {
        const context = this.getErrorContext(event);
        const handler = this.errorHandlers.get(context) || this.errorHandlers.get('default');
        
        if (handler) {
            handler(event.error, context);
        } else {
            this.logError(event.error, context);
        }
    }
    
    handlePromiseRejection(event) {
        this.handleError({ error: event.reason });
    }
    
    getErrorContext(event) {
        // Determine context from stack trace, source, etc.
        if (event.filename?.includes('api/')) return 'api';
        if (event.filename?.includes('ui/')) return 'ui';
        return 'default';
    }
    
    logError(error, context) {
        // Advanced error logging with context
        console.error('Unhandled error:', {
            error: error.message,
            stack: error.stack,
            context,
            timestamp: new Date().toISOString(),
            userAgent: navigator.userAgent,
            url: window.location.href
        });
    }
}
```

### 2. Advanced State Management Patterns
```javascript
// Immutable state with structural sharing
class ImmutableState {
    constructor(initialState = {}) {
        this.state = Object.freeze(this.deepFreeze(initialState));
        this.subscribers = new Set();
        this.middlewares = [];
    }
    
    deepFreeze(obj) {
        Object.getOwnPropertyNames(obj).forEach(prop => {
            if (obj[prop] !== null && typeof obj[prop] === 'object') {
                this.deepFreeze(obj[prop]);
            }
        });
        return Object.freeze(obj);
    }
    
    update(updater) {
        const oldState = this.state;
        let newState;
        
        if (typeof updater === 'function') {
            newState = updater(oldState);
        } else {
            newState = { ...oldState, ...updater };
        }
        
        // Apply middlewares
        for (const middleware of this.middlewares) {
            newState = middleware(oldState, newState) || newState;
        }
        
        this.state = this.deepFreeze(newState);
        this.notifySubscribers(oldState, this.state);
        
        return this.state;
    }
    
    subscribe(callback) {
        this.subscribers.add(callback);
        return () => this.subscribers.delete(callback);
    }
    
    use(middleware) {
        this.middlewares.push(middleware);
    }
    
    notifySubscribers(oldState, newState) {
        this.subscribers.forEach(callback => {
            try {
                callback(newState, oldState);
            } catch (error) {
                console.error('Error in state subscriber:', error);
            }
        });
    }
}

// Advanced reactive programming with observables
class Observable {
    constructor(executor) {
        this.observers = [];
        this.executor = executor;
    }
    
    static of(value) {
        return new Observable(observer => {
            observer.next(value);
            observer.complete();
        });
    }
    
    static fromEvent(element, eventName) {
        return new Observable(observer => {
            const handler = event => observer.next(event);
            element.addEventListener(eventName, handler);
            
            return () => element.removeEventListener(eventName, handler);
        });
    }
    
    subscribe(observer) {
        const subscription = {
            unsubscribe: null,
            closed: false
        };
        
        const wrappedObserver = {
            next: value => {
                if (!subscription.closed && observer.next) {
                    observer.next(value);
                }
            },
            error: error => {
                if (!subscription.closed && observer.error) {
                    observer.error(error);
                    subscription.closed = true;
                }
            },
            complete: () => {
                if (!subscription.closed && observer.complete) {
                    observer.complete();
                    subscription.closed = true;
                }
            }
        };
        
        subscription.unsubscribe = this.executor(wrappedObserver) || (() => {});
        
        return subscription;
    }
    
    map(transform) {
        return new Observable(observer => {
            return this.subscribe({
                next: value => observer.next(transform(value)),
                error: error => observer.error(error),
                complete: () => observer.complete()
            });
        });
    }
    
    filter(predicate) {
        return new Observable(observer => {
            return this.subscribe({
                next: value => {
                    if (predicate(value)) observer.next(value);
                },
                error: error => observer.error(error),
                complete: () => observer.complete()
            });
        });
    }
    
    debounce(ms) {
        return new Observable(observer => {
            let timeoutId;
            
            return this.subscribe({
                next: value => {
                    clearTimeout(timeoutId);
                    timeoutId = setTimeout(() => observer.next(value), ms);
                },
                error: error => observer.error(error),
                complete: () => observer.complete()
            });
        });
    }
}
```

### 3. Performance-Critical Patterns
```javascript
// Advanced memoization with cache strategies
class AdvancedMemoizer {
    constructor(options = {}) {
        this.cache = new Map();
        this.maxSize = options.maxSize || 100;
        this.ttl = options.ttl || Infinity;
        this.strategy = options.strategy || 'lru'; // lru, lfu, fifo
        this.accessCount = new Map();
        this.accessOrder = [];
    }
    
    memoize(fn, keyGenerator = (...args) => JSON.stringify(args)) {
        return (...args) => {
            const key = keyGenerator(...args);
            
            if (this.cache.has(key)) {
                const cached = this.cache.get(key);
                
                // Check TTL
                if (Date.now() - cached.timestamp > this.ttl) {
                    this.cache.delete(key);
                    this.accessCount.delete(key);
                } else {
                    this.updateAccess(key);
                    return cached.value;
                }
            }
            
            const result = fn(...args);
            this.set(key, result);
            return result;
        };
    }
    
    set(key, value) {
        if (this.cache.size >= this.maxSize) {
            this.evict();
        }
        
        this.cache.set(key, {
            value,
            timestamp: Date.now()
        });
        
        this.updateAccess(key);
    }
    
    updateAccess(key) {
        if (this.strategy === 'lru') {
            // Move to end for LRU
            const index = this.accessOrder.indexOf(key);
            if (index > -1) this.accessOrder.splice(index, 1);
            this.accessOrder.push(key);
        } else if (this.strategy === 'lfu') {
            // Update frequency count
            this.accessCount.set(key, (this.accessCount.get(key) || 0) + 1);
        }
    }
    
    evict() {
        let keyToEvict;
        
        switch (this.strategy) {
            case 'lru':
                keyToEvict = this.accessOrder.shift();
                break;
            case 'lfu':
                keyToEvict = Array.from(this.accessCount.entries())
                    .sort(([,a], [,b]) => a - b)[0][0];
                break;
            case 'fifo':
                keyToEvict = this.cache.keys().next().value;
                break;
        }
        
        if (keyToEvict) {
            this.cache.delete(keyToEvict);
            this.accessCount.delete(keyToEvict);
        }
    }
}

// Advanced async queue with concurrency control
class AsyncQueue {
    constructor(options = {}) {
        this.concurrency = options.concurrency || 1;
        this.queue = [];
        this.running = 0;
        this.paused = false;
    }
    
    add(task, priority = 0) {
        return new Promise((resolve, reject) => {
            const queueItem = {
                task,
                priority,
                resolve,
                reject,
                id: Math.random().toString(36).substr(2, 9)
            };
            
            // Insert based on priority
            const insertIndex = this.queue.findIndex(item => item.priority < priority);
            if (insertIndex === -1) {
                this.queue.push(queueItem);
            } else {
                this.queue.splice(insertIndex, 0, queueItem);
            }
            
            this.process();
        });
    }
    
    async process() {
        if (this.paused || this.running >= this.concurrency || this.queue.length === 0) {
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
            this.process(); // Process next item
        }
    }
    
    pause() {
        this.paused = true;
    }
    
    resume() {
        this.paused = false;
        this.process();
    }
    
    clear() {
        this.queue.forEach(item => item.reject(new Error('Queue cleared')));
        this.queue = [];
    }
    
    get size() {
        return this.queue.length;
    }
    
    get pending() {
        return this.running;
    }
}
```

## 🧪 Advanced Testing Strategies

### Test-Driven Development Patterns
```javascript
// Advanced test utilities for senior engineers
class TestUtils {
    static createMockModule(dependencies = {}) {
        return new Proxy(dependencies, {
            get(target, prop) {
                if (prop in target) return target[prop];
                
                // Auto-generate mock functions
                return jest.fn().mockName(`mock_${String(prop)}`);
            }
        });
    }
    
    static async waitFor(condition, timeout = 5000) {
        const start = Date.now();
        
        while (Date.now() - start < timeout) {
            if (await condition()) return true;
            await new Promise(resolve => setTimeout(resolve, 10));
        }
        
        throw new Error(`Condition not met within ${timeout}ms`);
    }
    
    static createAsyncIterator(values) {
        let index = 0;
        return {
            [Symbol.asyncIterator]() {
                return {
                    async next() {
                        if (index < values.length) {
                            return { value: values[index++], done: false };
                        }
                        return { done: true };
                    }
                };
            }
        };
    }
}

// Advanced mocking patterns
class MockBuilder {
    constructor() {
        this.mocks = new Map();
    }
    
    mock(path, implementation) {
        this.mocks.set(path, implementation);
        return this;
    }
    
    mockAsync(path, implementation) {
        return this.mock(path, async (...args) => {
            await new Promise(resolve => setTimeout(resolve, 0));
            return implementation(...args);
        });
    }
    
    mockWithDelay(path, implementation, delay) {
        return this.mock(path, async (...args) => {
            await new Promise(resolve => setTimeout(resolve, delay));
            return implementation(...args);
        });
    }
    
    build() {
        const moduleExports = {};
        
        for (const [path, implementation] of this.mocks) {
            const parts = path.split('.');
            let current = moduleExports;
            
            for (let i = 0; i < parts.length - 1; i++) {
                if (!current[parts[i]]) current[parts[i]] = {};
                current = current[parts[i]];
            }
            
            current[parts[parts.length - 1]] = implementation;
        }
        
        return moduleExports;
    }
}

// Example usage in tests
describe('Advanced Component Testing', () => {
    let mockDependencies;
    let component;
    
    beforeEach(() => {
        mockDependencies = new MockBuilder()
            .mock('api.fetchUsers', () => Promise.resolve([{ id: 1, name: 'Test User' }]))
            .mockWithDelay('analytics.track', () => {}, 100)
            .mockAsync('storage.save', (data) => ({ success: true, data }))
            .build();
        
        component = new UserComponent(mockDependencies);
    });
    
    test('handles concurrent operations correctly', async () => {
        const promises = Array.from({ length: 10 }, () => 
            component.loadUser(Math.floor(Math.random() * 100))
        );
        
        const results = await Promise.allSettled(promises);
        const successful = results.filter(r => r.status === 'fulfilled');
        
        expect(successful.length).toBeGreaterThan(0);
        expect(mockDependencies.api.fetchUsers).toHaveBeenCalledTimes(10);
    });
});
```

## 🚀 Performance & Optimization Mastery

### Memory Management Patterns
```javascript
// Advanced memory management
class MemoryManager {
    constructor() {
        this.weakRefs = new Set();
        this.cleanupTasks = new Map();
        this.monitoringEnabled = process.env.NODE_ENV === 'development';
    }
    
    createWeakRef(obj, cleanupCallback) {
        const weakRef = new WeakRef(obj);
        this.weakRefs.add(weakRef);
        
        if (cleanupCallback) {
            this.cleanupTasks.set(weakRef, cleanupCallback);
        }
        
        return weakRef;
    }
    
    cleanup() {
        for (const weakRef of this.weakRefs) {
            if (!weakRef.deref()) {
                // Object has been garbage collected
                const cleanupTask = this.cleanupTasks.get(weakRef);
                if (cleanupTask) {
                    cleanupTask();
                    this.cleanupTasks.delete(weakRef);
                }
                this.weakRefs.delete(weakRef);
            }
        }
    }
    
    startPeriodicCleanup(interval = 60000) {
        return setInterval(() => this.cleanup(), interval);
    }
    
    trackMemoryUsage() {
        if (!this.monitoringEnabled) return;
        
        const usage = performance.memory || {};
        console.log('Memory Usage:', {
            used: this.formatBytes(usage.usedJSHeapSize),
            total: this.formatBytes(usage.totalJSHeapSize),
            limit: this.formatBytes(usage.jsHeapSizeLimit)
        });
    }
    
    formatBytes(bytes) {
        if (!bytes) return 'N/A';
        const sizes = ['Bytes', 'KB', 'MB', 'GB'];
        const i = Math.floor(Math.log(bytes) / Math.log(1024));
        return (bytes / Math.pow(1024, i)).toFixed(2) + ' ' + sizes[i];
    }
}

// Advanced performance monitoring
class PerformanceMonitor {
    constructor() {
        this.metrics = new Map();
        this.observers = new Map();
    }
    
    startMeasuring(name) {
        performance.mark(`${name}-start`);
        return {
            end: () => {
                performance.mark(`${name}-end`);
                performance.measure(name, `${name}-start`, `${name}-end`);
                
                const measure = performance.getEntriesByName(name, 'measure')[0];
                this.recordMetric(name, measure.duration);
                
                return measure.duration;
            }
        };
    }
    
    recordMetric(name, value) {
        if (!this.metrics.has(name)) {
            this.metrics.set(name, {
                count: 0,
                total: 0,
                min: Infinity,
                max: -Infinity,
                values: []
            });
        }
        
        const metric = this.metrics.get(name);
        metric.count++;
        metric.total += value;
        metric.min = Math.min(metric.min, value);
        metric.max = Math.max(metric.max, value);
        metric.values.push(value);
        
        // Keep only last 100 values for percentile calculations
        if (metric.values.length > 100) {
            metric.values.shift();
        }
    }
    
    getMetrics(name) {
        const metric = this.metrics.get(name);
        if (!metric) return null;
        
        const sorted = [...metric.values].sort((a, b) => a - b);
        
        return {
            count: metric.count,
            average: metric.total / metric.count,
            min: metric.min,
            max: metric.max,
            p50: this.percentile(sorted, 50),
            p95: this.percentile(sorted, 95),
            p99: this.percentile(sorted, 99)
        };
    }
    
    percentile(sortedArray, p) {
        const index = Math.ceil(sortedArray.length * (p / 100)) - 1;
        return sortedArray[index] || 0;
    }
    
    observeLongTasks() {
        if ('PerformanceObserver' in window) {
            const observer = new PerformanceObserver(list => {
                for (const entry of list.getEntries()) {
                    console.warn('Long task detected:', {
                        duration: entry.duration,
                        startTime: entry.startTime
                    });
                }
            });
            
            observer.observe({ entryTypes: ['longtask'] });
            this.observers.set('longtask', observer);
        }
    }
}
```

## 🎖️ Leadership & Team Patterns

### Code Review Excellence
```javascript
// Code review checklist automation
class CodeReviewAnalyzer {
    constructor() {
        this.rules = new Map();
        this.setupDefaultRules();
    }
    
    setupDefaultRules() {
        this.addRule('no-console-log', {
            pattern: /console\.log/g,
            message: 'Remove console.log statements before merging',
            severity: 'warning'
        });
        
        this.addRule('proper-error-handling', {
            check: (code) => {
                const promiseRegex = /\.(then|catch)\(/g;
                const asyncRegex = /async\s+function|async\s*\(/g;
                const tryRegex = /try\s*{/g;
                
                const promises = (code.match(promiseRegex) || []).length;
                const asyncFunctions = (code.match(asyncRegex) || []).length;
                const tryBlocks = (code.match(tryRegex) || []).length;
                
                return promises > 0 || asyncFunctions > 0 ? tryBlocks > 0 : true;
            },
            message: 'Async operations should include proper error handling',
            severity: 'error'
        });
        
        this.addRule('performance-anti-patterns', {
            patterns: [
                { regex: /for\s*\(.*\)\s*{[\s\S]*?document\.querySelector/, message: 'Avoid DOM queries in loops' },
                { regex: /setInterval\(.*,\s*[1-9]\d{0,2}\)/, message: 'Consider requestAnimationFrame for frequent updates' }
            ],
            severity: 'warning'
        });
    }
    
    addRule(name, rule) {
        this.rules.set(name, rule);
    }
    
    analyze(code, filename = 'unknown') {
        const results = [];
        
        for (const [ruleName, rule] of this.rules) {
            if (rule.pattern) {
                const matches = code.match(rule.pattern);
                if (matches) {
                    results.push({
                        rule: ruleName,
                        message: rule.message,
                        severity: rule.severity,
                        matches: matches.length,
                        filename
                    });
                }
            } else if (rule.check) {
                if (!rule.check(code)) {
                    results.push({
                        rule: ruleName,
                        message: rule.message,
                        severity: rule.severity,
                        filename
                    });
                }
            } else if (rule.patterns) {
                for (const pattern of rule.patterns) {
                    if (pattern.regex.test(code)) {
                        results.push({
                            rule: ruleName,
                            message: pattern.message,
                            severity: rule.severity,
                            filename
                        });
                    }
                }
            }
        }
        
        return results;
    }
    
    generateReport(results) {
        const byFile = results.reduce((acc, result) => {
            if (!acc[result.filename]) acc[result.filename] = [];
            acc[result.filename].push(result);
            return acc;
        }, {});
        
        const summary = {
            totalIssues: results.length,
            errors: results.filter(r => r.severity === 'error').length,
            warnings: results.filter(r => r.severity === 'warning').length,
            files: Object.keys(byFile).length
        };
        
        return { summary, byFile };
    }
}
```

### Mentoring & Knowledge Sharing
```javascript
// Knowledge sharing system
class KnowledgeBase {
    constructor() {
        this.articles = new Map();
        this.tags = new Map();
        this.contributors = new Map();
    }
    
    addArticle(article) {
        const id = this.generateId();
        const enrichedArticle = {
            ...article,
            id,
            createdAt: new Date(),
            views: 0,
            rating: 0,
            comments: []
        };
        
        this.articles.set(id, enrichedArticle);
        this.indexTags(id, article.tags || []);
        this.trackContributor(article.author);
        
        return id;
    }
    
    indexTags(articleId, tags) {
        tags.forEach(tag => {
            if (!this.tags.has(tag)) this.tags.set(tag, new Set());
            this.tags.get(tag).add(articleId);
        });
    }
    
    trackContributor(author) {
        if (!this.contributors.has(author)) {
            this.contributors.set(author, { articles: 0, rating: 0 });
        }
        this.contributors.get(author).articles++;
    }
    
    search(query, filters = {}) {
        let results = Array.from(this.articles.values());
        
        if (query) {
            const lowerQuery = query.toLowerCase();
            results = results.filter(article => 
                article.title.toLowerCase().includes(lowerQuery) ||
                article.content.toLowerCase().includes(lowerQuery) ||
                (article.tags || []).some(tag => tag.toLowerCase().includes(lowerQuery))
            );
        }
        
        if (filters.tags) {
            results = results.filter(article => 
                filters.tags.every(tag => (article.tags || []).includes(tag))
            );
        }
        
        if (filters.author) {
            results = results.filter(article => article.author === filters.author);
        }
        
        return results.sort((a, b) => {
            if (filters.sortBy === 'rating') return b.rating - a.rating;
            if (filters.sortBy === 'views') return b.views - a.views;
            return b.createdAt - a.createdAt; // Default: newest first
        });
    }
    
    generateLearningPath(topic, level = 'beginner') {
        const relatedArticles = this.search(topic);
        const levelArticles = relatedArticles.filter(article => 
            article.level === level || !article.level
        );
        
        return this.orderByDependency(levelArticles);
    }
    
    orderByDependency(articles) {
        // Simple topological sort based on prerequisites
        const sorted = [];
        const visited = new Set();
        
        const visit = (article) => {
            if (visited.has(article.id)) return;
            
            // Visit prerequisites first
            (article.prerequisites || []).forEach(prereqId => {
                const prereq = this.articles.get(prereqId);
                if (prereq && articles.includes(prereq)) {
                    visit(prereq);
                }
            });
            
            visited.add(article.id);
            sorted.push(article);
        };
        
        articles.forEach(visit);
        return sorted;
    }
}
```

## 🎯 Business Impact & Strategic Thinking

### Technical Decision Framework
```javascript
// Architecture Decision Record (ADR) system
class ArchitectureDecisionRecord {
    constructor() {
        this.decisions = new Map();
        this.templates = this.setupTemplates();
    }
    
    setupTemplates() {
        return {
            technology: {
                sections: [
                    'Context',
                    'Problem Statement',
                    'Considered Options',
                    'Decision',
                    'Consequences',
                    'Implementation Plan',
                    'Success Metrics'
                ]
            },
            performance: {
                sections: [
                    'Current Performance Baseline',
                    'Performance Goals',
                    'Bottleneck Analysis',
                    'Proposed Solution',
                    'Trade-offs',
                    'Monitoring Plan',
                    'Rollback Strategy'
                ]
            }
        };
    }
    
    createDecision(type, data) {
        const template = this.templates[type] || this.templates.technology;
        
        const decision = {
            id: this.generateId(),
            type,
            status: 'proposed',
            createdAt: new Date(),
            author: data.author,
            title: data.title,
            sections: {},
            stakeholders: data.stakeholders || [],
            reviewers: data.reviewers || [],
            comments: []
        };
        
        // Initialize sections from template
        template.sections.forEach(section => {
            decision.sections[section] = data.sections?.[section] || '';
        });
        
        this.decisions.set(decision.id, decision);
        return decision.id;
    }
    
    updateStatus(id, status, comment) {
        const decision = this.decisions.get(id);
        if (!decision) throw new Error(`Decision ${id} not found`);
        
        decision.status = status;
        decision.updatedAt = new Date();
        
        if (comment) {
            decision.comments.push({
                author: comment.author,
                content: comment.content,
                timestamp: new Date()
            });
        }
    }
    
    generateReport() {
        const decisions = Array.from(this.decisions.values());
        
        return {
            total: decisions.length,
            byStatus: this.groupBy(decisions, 'status'),
            byType: this.groupBy(decisions, 'type'),
            recent: decisions
                .sort((a, b) => b.createdAt - a.createdAt)
                .slice(0, 10),
            needsReview: decisions.filter(d => d.status === 'proposed'),
            implemented: decisions.filter(d => d.status === 'accepted')
        };
    }
    
    groupBy(array, key) {
        return array.reduce((groups, item) => {
            const group = item[key] || 'unknown';
            groups[group] = (groups[group] || 0) + 1;
            return groups;
        }, {});
    }
}

// Business metrics tracking
class BusinessMetricsTracker {
    constructor() {
        this.metrics = new Map();
        this.goals = new Map();
    }
    
    defineMetric(name, config) {
        this.metrics.set(name, {
            ...config,
            values: [],
            alerts: []
        });
    }
    
    setGoal(metricName, goal) {
        this.goals.set(metricName, goal);
    }
    
    recordValue(metricName, value, timestamp = new Date()) {
        const metric = this.metrics.get(metricName);
        if (!metric) throw new Error(`Metric ${metricName} not defined`);
        
        metric.values.push({ value, timestamp });
        
        // Check against goals
        const goal = this.goals.get(metricName);
        if (goal && this.shouldAlert(metric, goal, value)) {
            this.createAlert(metricName, value, goal);
        }
    }
    
    shouldAlert(metric, goal, value) {
        switch (goal.type) {
            case 'threshold':
                return goal.direction === 'below' ? value < goal.value : value > goal.value;
            case 'trend':
                const recent = metric.values.slice(-goal.window || -10);
                const trend = this.calculateTrend(recent);
                return goal.direction === 'decreasing' ? trend < 0 : trend > 0;
            default:
                return false;
        }
    }
    
    calculateTrend(values) {
        if (values.length < 2) return 0;
        
        const firstHalf = values.slice(0, Math.floor(values.length / 2));
        const secondHalf = values.slice(Math.floor(values.length / 2));
        
        const firstAvg = firstHalf.reduce((sum, v) => sum + v.value, 0) / firstHalf.length;
        const secondAvg = secondHalf.reduce((sum, v) => sum + v.value, 0) / secondHalf.length;
        
        return secondAvg - firstAvg;
    }
    
    createAlert(metricName, value, goal) {
        const metric = this.metrics.get(metricName);
        metric.alerts.push({
            timestamp: new Date(),
            message: `${metricName} alert: ${value} (goal: ${goal.value})`,
            severity: goal.severity || 'warning'
        });
    }
    
    getDashboard() {
        const dashboard = {};
        
        for (const [name, metric] of this.metrics) {
            const recent = metric.values.slice(-30); // Last 30 data points
            
            dashboard[name] = {
                current: recent[recent.length - 1]?.value,
                trend: this.calculateTrend(recent),
                goal: this.goals.get(name),
                alerts: metric.alerts.slice(-5), // Last 5 alerts
                history: recent
            };
        }
        
        return dashboard;
    }
}
```

## 🔄 Continuous Learning & Growth

### Technical Radar
```javascript
// Technology adoption tracking
class TechRadar {
    constructor() {
        this.technologies = new Map();
        this.quadrants = ['Languages & Frameworks', 'Tools', 'Platforms', 'Techniques'];
        this.rings = ['Adopt', 'Trial', 'Assess', 'Hold'];
    }
    
    addTechnology(name, quadrant, ring, assessment) {
        this.technologies.set(name, {
            quadrant,
            ring,
            assessment,
            addedAt: new Date(),
            history: [{ ring, date: new Date(), reason: 'Initial assessment' }]
        });
    }
    
    updateTechnology(name, newRing, reason) {
        const tech = this.technologies.get(name);
        if (!tech) throw new Error(`Technology ${name} not found`);
        
        tech.history.push({
            ring: newRing,
            date: new Date(),
            reason
        });
        
        tech.ring = newRing;
    }
    
    generateReport() {
        const report = {};
        
        this.quadrants.forEach(quadrant => {
            report[quadrant] = {};
            this.rings.forEach(ring => {
                report[quadrant][ring] = Array.from(this.technologies.values())
                    .filter(tech => tech.quadrant === quadrant && tech.ring === ring)
                    .map(tech => ({
                        name: Array.from(this.technologies.keys())[
                            Array.from(this.technologies.values()).indexOf(tech)
                        ],
                        assessment: tech.assessment,
                        lastUpdated: tech.history[tech.history.length - 1].date
                    }));
            });
        });
        
        return report;
    }
    
    getRecommendations() {
        const adoptTechnologies = Array.from(this.technologies.entries())
            .filter(([, tech]) => tech.ring === 'Adopt')
            .map(([name, tech]) => ({ name, reason: tech.assessment }));
            
        const holdTechnologies = Array.from(this.technologies.entries())
            .filter(([, tech]) => tech.ring === 'Hold')
            .map(([name, tech]) => ({ name, reason: tech.assessment }));
        
        return {
            shouldAdopt: adoptTechnologies,
            shouldAvoid: holdTechnologies,
            inTrial: this.getTechnologiesByRing('Trial'),
            needsAssessment: this.getTechnologiesByRing('Assess')
        };
    }
    
    getTechnologiesByRing(ring) {
        return Array.from(this.technologies.entries())
            .filter(([, tech]) => tech.ring === ring)
            .map(([name]) => name);
    }
}
```

## 🎖️ Senior Engineer Principles

### The Senior Mindset Checklist

#### Technical Excellence
- [ ] Write code that is self-documenting and maintainable
- [ ] Consider performance implications of every decision
- [ ] Design for testability from the start
- [ ] Think about edge cases and error scenarios
- [ ] Build with accessibility and internationalization in mind

#### System Thinking
- [ ] Understand how your code fits into the larger architecture
- [ ] Consider the impact on other teams and systems
- [ ] Design for scalability and future requirements
- [ ] Think about operational concerns (monitoring, debugging, deployment)
- [ ] Plan for failure and build resilient systems

#### Team Leadership
- [ ] Mentor junior developers and share knowledge
- [ ] Conduct thorough and constructive code reviews
- [ ] Establish and document team standards
- [ ] Lead by example in code quality and professionalism
- [ ] Foster a culture of continuous learning

#### Business Alignment
- [ ] Understand business requirements and constraints
- [ ] Communicate technical decisions to non-technical stakeholders
- [ ] Balance technical debt with feature delivery
- [ ] Measure and optimize for business metrics
- [ ] Consider the total cost of ownership

#### Continuous Growth
- [ ] Stay current with industry trends and best practices
- [ ] Experiment with new technologies and patterns
- [ ] Contribute to open source and the developer community
- [ ] Seek feedback and continuously improve
- [ ] Share learnings through writing and speaking

## 🎯 Key Takeaways

As a senior JavaScript engineer, remember:

1. **Code is Communication**: Write for humans, not just computers
2. **Architecture Matters**: Every decision has long-term consequences
3. **Performance is a Feature**: Users notice slow applications
4. **Testing is Essential**: Untested code is legacy code
5. **Documentation is Code**: Undocumented systems are unmaintainable
6. **Leadership is Service**: Help others grow and succeed
7. **Business Context Matters**: Technical decisions should serve business goals
8. **Continuous Learning**: Technology evolves rapidly; never stop learning
9. **Quality is Everyone's Responsibility**: Set high standards and maintain them
10. **Impact Beyond Code**: Your influence extends far beyond the code you write

The journey to senior engineering mastery is continuous. Focus on building systems that not only work today but will serve your team and organization well into the future.
