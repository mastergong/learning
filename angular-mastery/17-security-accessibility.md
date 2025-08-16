# 17 — Security & Accessibility

## Security
- Use HttpClient (XSRF protection with same-origin)
- Sanitize HTML; avoid bypassSecurityTrust unless necessary
- Content Security Policy (CSP)
- Avoid storing sensitive data in localStorage; prefer cookies with httpOnly (via backend)

## Accessibility (a11y)
- Semantic HTML, ARIA where needed
- Focus management, keyboard navigation
- Color contrast, labels for inputs
