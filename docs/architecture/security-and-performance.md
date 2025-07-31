# Security and Performance

## Security Requirements

**Frontend Security:**
- CSP Headers: `default-src 'self'; script-src 'self' 'unsafe-inline' https://res.wx.qq.com; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self' https://api.srmp.edu.cn wss://api.srmp.edu.cn`
- XSS Prevention: React's automatic escaping + DOMPurify for user content
- Secure Storage: Sensitive data in httpOnly cookies, not localStorage

**Backend Security:**
- Input Validation: class-validator decorators on all DTOs
- Rate Limiting: 100 requests per minute per IP, 1000 per authenticated user
- CORS Policy: Whitelist specific origins, credentials allowed

**Authentication Security:**
- Token Storage: httpOnly, secure, sameSite cookies
- Session Management: Redis with 24-hour expiry, refresh token rotation
- Password Policy: Minimum 8 characters, must include number and special character

## Performance Optimization

**Frontend Performance:**
- Bundle Size Target: <200KB initial JS, <50KB per lazy route
- Loading Strategy: Route-based code splitting, progressive enhancement
- Caching Strategy: Service worker for offline support, SWR for data

**Backend Performance:**
- Response Time Target: p50 <100ms, p99 <500ms
- Database Optimization: Connection pooling, query optimization, read replicas
- Caching Strategy: Redis for sessions, frequent queries, and computed data
