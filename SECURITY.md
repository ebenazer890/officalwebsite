# Security Guidelines

## Critical Security Issues Addressed

### 1. Exposed API Key (CRITICAL - FIXED)

**Issue:** The GROQ API key was hardcoded in `chatbot.html`, visible to anyone viewing the source code.

**Risk Level:** CRITICAL
- Anyone could copy the API key
- Unauthorized usage of your API quota
- Potential financial charges
- Security breach

**Fix Applied:**
- Removed the exposed API key from client-side code
- Added warning comments about proper API key handling
- Modified error message to indicate backend implementation needed

**Status:** ⚠️ CHATBOT FUNCTIONALITY DISABLED until proper backend is implemented

### Next Steps to Restore Chatbot Functionality Securely

#### Option 1: Backend Proxy (Recommended)

Create a backend server to handle API calls:

**Node.js/Express Example:**
```javascript
// server.js
const express = require('express');
const fetch = require('node-fetch');
require('dotenv').config();

const app = express();
app.use(express.json());

app.post('/api/chat', async (req, res) => {
  try {
    const response = await fetch('https://api.groq.com/openai/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.GROQ_API_KEY}`
      },
      body: JSON.stringify({
        model: 'llama3-8b-8192',
        temperature: 0.5,
        max_tokens: 150,
        messages: req.body.messages
      })
    });
    
    const data = await response.json();
    res.json(data);
  } catch (error) {
    console.error('API Error:', error);
    res.status(500).json({ error: 'Failed to process request' });
  }
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

**Environment Variables (.env file):**
```env
GROQ_API_KEY=your_actual_api_key_here
```

**Update chatbot.html:**
```javascript
// Replace the API call with:
const response = await fetch('/api/chat', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    messages: [
      {
        role: "system",
        content: "You are a therapy chatbot..."
      },
      {
        role: "user",
        content: message
      }
    ]
  })
});
```

#### Option 2: Serverless Functions

Use platforms like Netlify Functions, Vercel Edge Functions, or Cloudflare Workers:

**Netlify Functions Example:**
```javascript
// netlify/functions/chat.js
const fetch = require('node-fetch');

exports.handler = async (event) => {
  if (event.httpMethod !== 'POST') {
    return { statusCode: 405, body: 'Method Not Allowed' };
  }

  try {
    const { messages } = JSON.parse(event.body);
    
    const response = await fetch('https://api.groq.com/openai/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.GROQ_API_KEY}`
      },
      body: JSON.stringify({
        model: 'llama3-8b-8192',
        temperature: 0.5,
        max_tokens: 150,
        messages
      })
    });

    const data = await response.json();
    
    return {
      statusCode: 200,
      body: JSON.stringify(data)
    };
  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: 'Failed to process request' })
    };
  }
};
```

## Security Best Practices

### 1. Never Commit Secrets
❌ **DON'T:**
```javascript
const API_KEY = "sk_live_abc123..."; // NEVER DO THIS
```

✅ **DO:**
```javascript
// Use environment variables
const API_KEY = process.env.API_KEY;

// For client-side: use a backend proxy
const response = await fetch('/api/endpoint');
```

### 2. Use Environment Variables
```bash
# .env file (add to .gitignore)
GROQ_API_KEY=your_key_here
DATABASE_URL=your_db_url
SECRET_KEY=your_secret
```

```javascript
// .env.example (commit this as template)
GROQ_API_KEY=
DATABASE_URL=
SECRET_KEY=
```

### 3. Secure API Endpoints
```javascript
// Add rate limiting
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});

app.use('/api/', limiter);

// Add CORS restrictions
const cors = require('cors');
app.use(cors({
  origin: 'https://yourdomain.com'
}));

// Validate inputs
app.post('/api/chat', (req, res) => {
  if (!req.body.messages || !Array.isArray(req.body.messages)) {
    return res.status(400).json({ error: 'Invalid request' });
  }
  // Process request...
});
```

### 4. Content Security Policy
Add to your HTML:
```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' 'unsafe-inline' https://cdn.tailwindcss.com https://cdnjs.cloudflare.com;
               style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdnjs.cloudflare.com;
               font-src 'self' https://fonts.gstatic.com https://cdnjs.cloudflare.com;
               img-src 'self' data: https:;
               connect-src 'self' https://api.groq.com;">
```

### 5. Input Validation and Sanitization
```javascript
// Sanitize user input
function sanitizeInput(input) {
  return input
    .trim()
    .slice(0, 500) // Limit length
    .replace(/[<>]/g, ''); // Remove potential XSS characters
}

// Validate before sending to API
const sanitizedMessage = sanitizeInput(userMessage);
if (sanitizedMessage.length === 0) {
  return;
}
```

### 6. HTTPS Only
- Always use HTTPS in production
- Redirect HTTP to HTTPS
- Use secure cookies: `secure: true, sameSite: 'strict'`

### 7. Regular Security Audits
```bash
# Check for vulnerable dependencies (if using npm)
npm audit

# Fix vulnerabilities
npm audit fix
```

## Current Security Status

### ✅ Fixed
- [x] Removed exposed API key from chatbot.html
- [x] Added security documentation
- [x] Added warning comments in code

### ⚠️ Attention Needed
- [ ] Implement backend proxy for chatbot API
- [ ] Set up environment variables
- [ ] Add rate limiting to API endpoints
- [ ] Implement input validation
- [ ] Add Content Security Policy headers
- [ ] Set up HTTPS (if not already configured)

### 🔒 Recommended
- [ ] Regular security audits
- [ ] Implement CORS restrictions
- [ ] Add request logging and monitoring
- [ ] Set up API key rotation policy
- [ ] Implement authentication if needed
- [ ] Add error handling that doesn't leak information

## Emergency Response

### If You Believe Your API Key Was Compromised:

1. **Immediately revoke the old key**
   - Log into your GROQ account
   - Navigate to API keys section
   - Delete or disable the compromised key

2. **Generate a new API key**
   - Create a new key
   - Store it securely in environment variables
   - Update your backend configuration

3. **Monitor for unusual activity**
   - Check API usage logs
   - Look for unexpected charges
   - Review access patterns

4. **Update your code**
   - Ensure new key is properly secured
   - Follow the backend proxy pattern
   - Never commit the new key to version control

## Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [API Security Best Practices](https://github.com/shieldfy/API-Security-Checklist)
- [Netlify Environment Variables](https://docs.netlify.com/environment-variables/overview/)
- [GitHub Secrets Management](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

## Contact

If you discover a security vulnerability, please report it responsibly:
- Email: ebenazer890@gmail.com
- Subject: "Security Issue - Portfolio Website"

---

**Last Updated:** November 18, 2025
**Security Level:** ⚠️ MODERATE (chatbot disabled pending backend implementation)
