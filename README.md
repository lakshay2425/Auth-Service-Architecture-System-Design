# Authentication Service

A robust, authentication service built with Express.js, providing user authentication through multiple methods with background email processing via AWS SNS/SQS/Lambda integration.

## 🌟 Overview

This authentication service is currently powering **three production applications** and provides:

- **Dual Authentication Methods**: Traditional email/password and Google OAuth 2.0
- **JWT-based Session Management**: Secure token-based authentication with RS256 signing
- **Background Email Processing**: Asynchronous email notifications via AWS SNS → SQS → Lambda pipeline
- **Production-Ready Features**: Rate limiting, comprehensive logging, CORS handling, and Docker deployment
- **Security First**: Input validation, password hashing, secure cookie handling, and environment-based configurations

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client Apps   │    │  Auth Service   │    │   AWS Services  │
│                 │    │                 │    │                 │
│ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │
│ │ App 1       │─┼────┼→│ Express.js  │─┼────┼→│ SNS Topic   │ │
│ │ App 2       │ │    │ │ MongoDB     │ │    │ │     ↓       │ │
│ │ App 3       │ │    │ │ Rate Limiter│ │    │ │ SQS Queue   │ │
│ └─────────────┘ │    │ │ Winston Logs│ │    │ │     ↓       │ │
└─────────────────┘    │ └─────────────┘ │    │ │ Lambda Fn   │ │
                       └─────────────────┘    │ │ (Email)     │ │
                                              │ └─────────────┘ │
                                              └─────────────────┘
```

## ✨ Features

### Authentication Methods
- **Email/Password Authentication**: Traditional form-based authentication with bcrypt password hashing
- **Google OAuth 2.0**: Seamless Google sign-in integration
- **Personal OAuth Route**: Restricted OAuth access for specific users
- **JWT Token Management**: RS256 algorithm with 12-hour expiration

### Security & Performance
- **Rate Limiting**: Configurable request limiting (10 requests per 5 minutes for auth endpoints)
- **CORS Protection**: Multi-origin support for production applications
- **Request Tracking**: Unique request IDs for debugging and monitoring
- **Input Validation**: Comprehensive request validation and sanitization
- **Security Headers**: Proper cookie settings with httpOnly and secure flags

### Background Processing
- **SNS Integration**: Publishes user events (registration/login) to AWS SNS
- **Event-Driven Email**: SQS subscribers trigger Lambda functions for email delivery
- **Message Filtering**: Event-type based message routing
- **Error Handling**: Robust error handling for external service failures

### Monitoring & Logging
- **Winston Logger**: Structured JSON logging with daily file rotation
- **Request Logging**: Morgan middleware for HTTP request logging
- **Separate Log Files**: Success and error logs in different files
- **Log Retention**: Configurable log retention policies (7-14 days)

### Production Features
- **Docker Support**: Multi-stage Docker builds with security best practices
- **Health Check Endpoint**: Service health monitoring
- **Environment-based Configuration**: Secure environment variable management
- **CI/CD Pipeline**: GitHub Actions for automated testing and linting


## 🚀 API Endpoints

### Authentication Routes (`/api/users`)
- **POST** `/api/users/signup` - Register new user
- **POST** `/api/users/login` - User login
- **POST** `/api/users/logout` - User logout

### Google OAuth Routes (`/api/auth`)
- **GET** `/api/auth/google/callback` - Google OAuth callback
- **GET** `/api/auth/google/verify` - Verify JWT token

### Health Check
- **GET** `/health` - Service health status

## 🛠️ Installation & Setup

### Prerequisites
- Node.js 18+ or 20+ or 22+
- MongoDB database
- AWS account with SNS access
- Google OAuth 2.0 credentials

### 1. Clone the Repository
```bash
git clone <repository-url>
cd "Auth Service"
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Environment Configuration

Create a `.env` file based on `.env.sample`:

```env
# Server Configuration
NODE_ENV=development
PORT=5000

# JWT Configuration
JWT_PRIVATE_KEY=<base64-encoded-private-key>

# Google OAuth
GOOGLE_CLIENT_ID=<your-google-client-id>
GOOGLE_CLIENT_SECRET=<your-google-client-secret>

# Database
MONGO_ATLAS_URI=<mongodb-connection-string>

# AWS SNS
TOPIC_ARN=<aws-sns-topic-arn>
AWS_ACCESS_KEY_ID=<aws-access-key>
AWS_SECRET_ACCESS_KEY=<aws-secret-key>

# CORS Origins
DEPLOYED_URL=<production-frontend-url>
DEPLOYED_BACKEND_URI=<production-backend-url>

```

### 4. Generate JWT Keys

Generate RSA key pair for JWT signing:

```bash
# Generate private key
openssl genrsa -out private.key 2048

# Generate public key
openssl rsa -in private.key -pubout -out public.key

# Convert private key to base64 for environment variable
base64 -i private.key -o private.key.b64
```

### 5. AWS SNS/SQS Setup

1. Create an SNS topic for user events
2. Create SQS queues for different event types
3. Set up Lambda functions to process email sending
4. Configure proper IAM permissions

### 6. MongoDB Setup

Ensure your MongoDB database is accessible and the connection string is configured in your environment variables.

## 🏃‍♂️ Running the Application

### Development Mode
```bash
npm run dev
```

### Production Mode
```bash
npm run build
```

### Linting
```bash
npm run lint
```

## 🐳 Docker Deployment

### Development Environment
```bash
cd Docker
docker-compose -f compose.dev.yaml up --build
```

### Production Environment
```bash
cd Docker
docker-compose -f compose.yaml up -d
```

### Docker Image Features
- **Multi-stage builds** for optimized image size
- **Non-root user** execution for security
- **Health checks** for container monitoring
- **Secrets management** for production deployment
- **Alpine Linux** for minimal attack surface

## 🔧 Configuration

### Rate Limiting
- Authentication endpoints: 10 requests per 5 minutes
- Configurable via `createRateLimiter` function

### CORS Origins
The service supports multiple client applications through configured CORS origins:
- Local development: `http://localhost:5173`, `http://localhost:4000`
- Production applications: Configured via environment variables

### Cookie Settings
```javascript
{
  httpOnly: true,                    // Prevent XSS
  secure: true,                      // HTTPS only in production
  sameSite: "lax",                   // CSRF protection
  maxAge: 7 * 24 * 60 * 60 * 1000,   // 7 days
  domain: ".yourdomain.com"      // Production domain
}
```

### Logging Configuration
- **Success logs**: `logs/auth-service/success-YYYY-MM-DD.log`
- **Error logs**: `logs/auth-service/error-YYYY-MM-DD.log`
- **Request logs**: `logs/requests/access-YYYY-MM-DD.log`
- **Retention**: 7 days for auth logs, 14 days for request logs
- **Rotation**: Daily with 20MB max file size

## 📧 Email Processing Flow

1. **User Action**: User registers or logs in
2. **SNS Publishing**: Service publishes event to SNS topic with message attributes
3. **SQS Filtering**: SQS queues receive messages based on event type filters
4. **Lambda Processing**: Lambda functions triggered by SQS messages
5. **Email Delivery**: Lambda functions send appropriate welcome/login emails

### Event Types
- `user_registered`: Sent when new user signs up
- `user_loggedIn`: Sent when user logs in (configurable per application)

## 🧪 Testing & CI/CD

### GitHub Actions Workflow
The project includes automated CI/CD pipeline that:
- Tests on Node.js versions 18.x, 20.x, 22.x
- Runs ESLint for code quality
- Executes on every push to `main` branch

### Manual Testing
```bash
# Test health endpoint
curl http://localhost:5000/health

# Test signup
curl -X POST http://localhost:5000/api/users/signup \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@example.com","password":"password123","username":"johndoe","business":"Test App"}'

# Test login
curl -X POST http://localhost:5000/api/users/login \
  -H "Content-Type: application/json" \
  -d '{"email":"john@example.com","password":"password123","business":"Test App"}'
```

## 🚀 Production Usage

This authentication service is currently serving **three production applications**:

1. **Resource Manager**: Internal resource management system
2. **KS Application**: Knowledge sharing platform  
3. **Daily Activity Tracker**: Personal productivity application

### Production Features
- **Rolling Updates**: Zero-downtime deployment strategy
- **Monitoring**: Comprehensive logging and health monitoring
- **Security**: Secrets management and environment isolation

## 🛡️ Security Best Practices

### Implemented Security Measures
- **JWT RS256**: Asymmetric key signing for token security
- **Bcrypt Hashing**: Secure password storage with salt rounds
- **Rate Limiting**: Protection against brute force attacks
- **CORS Configuration**: Restricted origin access
- **Input Validation**: Comprehensive request validation
- **Security Headers**: Proper HTTP security headers
- **Environment Variables**: Secure configuration management
- **Docker Security**: Non-root user execution

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Guidelines
- Follow ESLint configuration
- Add appropriate error handling
- Update documentation for new features
- Test endpoints thoroughly
- Follow existing code patterns and structure

## 📝 License

This project is licensed under the ISC License.

## 📞 Support

For support, please create an issue in the repository or contact the maintainer.

## 🔄 Version History

- **v1.0.0**: Initial production release
  - Basic authentication with email/password
  - Google OAuth integration
  - SNS email processing
  - Docker deployment support
  - Comprehensive logging and monitoring

---

**Note**: This authentication service is actively maintained and deployed in production environments. For production deployment, ensure all environment variables are properly configured and security best practices are followed.
