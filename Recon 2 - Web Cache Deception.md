

### Quick Reconnaissance  
- Performed a **quick browse** of the web application.  
- Used **Burp Suite history** to analyze the requests.  
- Noticed that **all requests were GraphQL POST methods**.  

### Understanding the Application  
- Clicked on every element to understand how the application works.  
- Identified an **interesting endpoint**:  
  `GET /api/auth/csrf`
- This endpoint **fetches a new CSRF token every hour**.  
- Initially, **no significant impact** was found.  

### Fuzzing for Hidden Endpoints  
- Started fuzzing using: 
  `/api/auth/FUZZ`
- **Discovered a new endpoint** that exposed:
  `GET /api/auth/session/h1/mmdztest2.jpeg?ok HTTP/2`
- **All user information**  & **API keys**  
- The **new endpoint was vulnerable** to **Web Cache Deception Attack (WCDA)**.



