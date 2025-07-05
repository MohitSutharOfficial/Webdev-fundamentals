# HTML Intermediate

Welcome to the intermediate level of HTML! This section covers advanced HTML features, semantic elements, accessibility, and modern web standards.

## 🎯 Learning Objectives

By the end of this section, you will:
- Master HTML5 semantic elements and their proper usage
- Understand advanced form features and validation
- Implement responsive design principles with HTML
- Create accessible web content following WCAG guidelines
- Use microdata and structured data for SEO
- Work with multimedia elements and modern APIs
- Build component-based HTML structures

## 📚 Topics Covered

### 1. Semantic HTML5 Elements
- **File:** `01_semantic_elements.html`
- Article, section, aside, nav, header, footer
- Main, figure, figcaption, time, mark
- Proper semantic structure and hierarchy
- SEO benefits and accessibility improvements

### 2. Advanced Forms and Validation
- **File:** `02_advanced_forms.html`
- HTML5 input types and attributes
- Custom validation messages
- Form accessibility (ARIA labels, fieldsets)
- Progressive enhancement techniques

### 3. Multimedia and Responsive Media
- **File:** `03_multimedia_responsive.html`
- Picture element and responsive images
- Video and audio elements with controls
- Lazy loading and performance optimization
- Media queries in HTML

### 4. Accessibility and ARIA
- **File:** `04_accessibility_aria.html`
- ARIA roles, states, and properties
- Screen reader optimization
- Keyboard navigation patterns
- Color contrast and visual accessibility

### 5. Microdata and Structured Data
- **File:** `05_microdata_seo.html`
- Schema.org markup
- JSON-LD structured data
- Rich snippets and SEO benefits
- Open Graph and Twitter Cards

### 6. Modern HTML Features
- **File:** `06_modern_html.html`
- Web Components basics
- Custom elements and attributes
- Progressive Web App manifests
- Modern browser APIs integration

## 🎯 Senior Engineer Insights

### Semantic HTML Strategy
```html
<!-- Bad: Generic div soup -->
<div class="header">
  <div class="nav">...</div>
</div>
<div class="content">
  <div class="article">...</div>
  <div class="sidebar">...</div>
</div>

<!-- Good: Semantic structure -->
<header>
  <nav aria-label="Main navigation">...</nav>
</header>
<main>
  <article>...</article>
  <aside>...</aside>
</main>
```

### Accessibility-First Approach
- Start with semantic HTML before adding ARIA
- Test with keyboard navigation and screen readers
- Use progressive enhancement for complex interactions
- Implement skip links and focus management

### Performance Considerations
- Use semantic elements for better browser optimization
- Implement lazy loading for images and iframes
- Optimize for Core Web Vitals (LCP, FID, CLS)
- Consider critical rendering path

### SEO and Structured Data
- Use proper heading hierarchy (h1-h6)
- Implement schema.org markup for rich snippets
- Optimize for featured snippets and voice search
- Use canonical URLs and meta tags effectively

## 🛠 Best Practices

### Code Organization
- Use semantic elements for document structure
- Group related form fields with fieldsets
- Implement proper heading hierarchy
- Use lists for navigation and grouped content

### Accessibility
- Always include alt text for images
- Use proper form labels and descriptions
- Ensure sufficient color contrast
- Test with assistive technologies

### Performance
- Optimize images with proper formats and sizes
- Use lazy loading for below-the-fold content
- Minimize DOM depth and complexity
- Implement resource hints (preload, prefetch)

### SEO
- Use descriptive title and meta descriptions
- Implement structured data markup
- Create semantic URL structures
- Optimize for mobile-first indexing

## 📝 Practice Projects

### Project 1: Blog Article Template
Create a semantic blog post template with:
- Proper article structure
- Accessible navigation
- Structured data markup
- Responsive images

### Project 2: Contact Form with Advanced Validation
Build a contact form featuring:
- Custom validation messages
- Accessibility features
- Progressive enhancement
- Error handling

### Project 3: Media Gallery
Develop a responsive media gallery with:
- Picture element for responsive images
- Lazy loading implementation
- Keyboard navigation
- ARIA live regions

## 🚀 Next Steps

After completing this section, you'll be ready for:
- **HTML Advanced:** Complex layouts, web components, and modern APIs
- **CSS Intermediate:** Advanced styling, flexbox, and grid
- **JavaScript Intermediate:** DOM manipulation and event handling

## 🔧 Tools and Resources

### Validation Tools
- [W3C Markup Validator](https://validator.w3.org/)
- [WAVE Accessibility Evaluation](https://wave.webaim.org/)
- [Lighthouse Accessibility Audit](https://developers.google.com/web/tools/lighthouse)

### Reference Documentation
- [MDN HTML Reference](https://developer.mozilla.org/en-US/docs/Web/HTML)
- [WCAG Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [Schema.org Vocabulary](https://schema.org/)

### Testing Tools
- Screen readers (NVDA, JAWS, VoiceOver)
- Browser developer tools
- Accessibility browser extensions

---

**Remember:** Intermediate HTML is about understanding the "why" behind semantic markup and creating accessible, performant, and SEO-friendly web content. Focus on building solid foundations that will scale with your projects.
