# External APIs

## WeChat Work API

- **Purpose:** User authentication and notification delivery
- **Documentation:** https://developer.work.weixin.qq.com/
- **Base URL(s):** https://qyapi.weixin.qq.com/cgi-bin/
- **Authentication:** OAuth 2.0 with corp ID and secret
- **Rate Limits:** 1000 calls per minute per corp

**Key Endpoints Used:**
- `GET /gettoken` - Get access token
- `GET /user/getuserinfo` - Get user info after OAuth
- `POST /message/send` - Send notification messages

**Integration Notes:** Implement token caching to avoid rate limits, handle token refresh automatically, queue messages for batch sending

## University SSO API

- **Purpose:** Single sign-on integration for existing university accounts
- **Documentation:** [To be provided by university IT]
- **Base URL(s):** https://sso.university.edu.cn
- **Authentication:** SAML 2.0 or CAS protocol
- **Rate Limits:** No specific limits documented

**Key Endpoints Used:**
- `POST /saml/login` - Initiate SSO login
- `POST /saml/validate` - Validate SAML assertion
- `GET /api/user/{id}` - Get user details after login

**Integration Notes:** Implement fallback to local auth if SSO is down, cache user data to reduce API calls, sync roles from university directory

## University Grade Management API

- **Purpose:** Import grades and course enrollment data
- **Documentation:** [To be provided by university IT]
- **Base URL(s):** https://grades.university.edu.cn/api/v1
- **Authentication:** API key with IP whitelist
- **Rate Limits:** 100 requests per hour

**Key Endpoints Used:**
- `GET /students/{studentId}/courses` - Get enrolled courses
- `GET /courses/{courseId}/grades` - Get grades for a course
- `POST /grades/bulk` - Bulk grade import

**Integration Notes:** Implement nightly sync to avoid peak hours, validate data integrity before import, maintain audit log of all changes
