# CloudBlitz Student Enrollment System - Project Workflow Overview

## 🏗️ Architecture Overview

Your project is a **full-stack microservices application** with the following components:

**Frontend**: React + TypeScript + Tailwind CSS (Port 5173)
**Backend**: 3 Spring Boot microservices (Ports 8081-8083)
**Database**: MongoDB
**Infrastructure**: AWS (EKS, S3, CloudFront, Route53)
**CI/CD**: Jenkins pipelines with Terraform

---

## 🔄 Development Workflow

### 1. Local Development
```bash
# Start all services with Docker Compose
docker-compose up --build

# Or run individually:
# Backend services (Spring Boot + Maven)
cd backend/auth-service && ./mvnw spring-boot:run
cd backend/course-service && ./mvnw spring-boot:run  
cd backend/enrollment-service && ./mvnw spring-boot:run

# Frontend (React + Vite)
cd frontend && npm install && npm run dev
```

### 2. Service Communication Flow
```
User → Frontend (React) → Auth Service → JWT Token
                    ↓
              Course Service (CRUD)
                    ↓
            Enrollment Service (JWT validation)
```

---

## 🚀 Deployment Workflow (Infrastructure as Code)

### Phase 1: Infrastructure Setup

#### 1. Terraform Backend (One-time setup)
- S3 bucket for state storage
- DynamoDB for state locking

#### 2. Global Infrastructure (Jenkins Pipeline)
- Route 53 hosted zone
- ACM certificates
- DNS configuration

#### 3. Backend Infrastructure (Jenkins Pipeline)
- VPC with public/private subnets
- EKS cluster with managed node groups
- Application Load Balancer
- Security groups

#### 4. Frontend Infrastructure (Jenkins Pipeline)
- S3 bucket for static hosting
- CloudFront distribution
- SSL certificates

### Phase 2: Application Deployment

#### 1. Build & Push Images (Jenkins)
- Docker images for each service
- Push to container registry

#### 2. Kubernetes Deployment
- Deploy auth-service (2 replicas)
- Deploy course-service (2 replicas)  
- Deploy enrollment-service (2 replicas)
- Deploy frontend (static assets to S3)

---

## 🔧 Technology Stack Breakdown

### Backend Services
- **Auth Service** (Port 8081): JWT authentication, user registration/login
- **Course Service** (Port 8082): Course CRUD operations, data seeding
- **Enrollment Service** (Port 8083): User enrollments with JWT validation

### Frontend
- **React 19** with TypeScript
- **Vite** for build tooling
- **Tailwind CSS** for styling
- **Context API** for state management

### Infrastructure
- **AWS EKS** for container orchestration
- **MongoDB Atlas** for database
- **S3 + CloudFront** for static hosting
- **Route 53** for DNS management
- **Terraform** for infrastructure provisioning

### CI/CD Pipeline
- **Jenkins** for automation
- **Docker** for containerization
- **Kubernetes** for orchestration
- **Terraform** for infrastructure

---

## 📊 Data Flow Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Auth Service  │    │   MongoDB       │
│   (React)       │◄──►│   (JWT)         │◄──►│   (cb_auth)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │
         ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Course Service  │    │Enrollment Service│    │   MongoDB       │
│   (CRUD)        │◄──►│   (JWT Valid)   │◄──►│ (cb_courses,    │
└─────────────────┘    └─────────────────┘    │  cb_enrollments)│
                                              └─────────────────┘
```

---

## 🛡️ Security Implementation

1. **JWT Authentication**: HS256 algorithm with 24-hour expiration
2. **CORS Configuration**: Configured for production domain
3. **Password Hashing**: bcrypt for secure password storage
4. **Route Protection**: Protected routes in frontend
5. **SSL/TLS**: ACM certificates for all endpoints

---

## 📈 Scalability Features

1. **Microservices Architecture**: Independent scaling
2. **Kubernetes**: Auto-scaling based on resource usage
3. **Load Balancing**: ALB for traffic distribution
4. **CDN**: CloudFront for global content delivery
5. **Database**: MongoDB Atlas for managed database

---

## 🔍 Monitoring & Health Checks

- **Health Endpoints**: `/health` for all services
- **Liveness Probes**: Kubernetes health checks
- **Readiness Probes**: Service availability checks
- **Resource Limits**: CPU/Memory constraints defined

---

## 💡 Key Interview Points

1. **Microservices Design**: Each service has single responsibility
2. **Infrastructure as Code**: Complete Terraform automation
3. **CI/CD Pipeline**: Jenkins with automated deployments
4. **Cloud-Native**: AWS services integration
5. **Security**: JWT-based authentication with proper validation
6. **Scalability**: Kubernetes with auto-scaling capabilities
7. **Monitoring**: Health checks and resource monitoring
8. **Database Design**: Separate databases per service (microservices pattern)

---

## 🚀 Production URLs

- **Frontend**: https://cloudblitz.in
- **Backend APIs**: https://api.cloudblitz.in
- **Local Development**: http://localhost:5173

---

## 📱 Frontend Features

### Pages
- **Login/Register**: Authentication forms with validation
- **Dashboard**: Welcome page with course listing and enrollment
- **Enrollments**: User's enrolled courses with status tracking

### Components
- `LoginForm`: Authentication form with toggle between login/register
- `CourseList`: Displays available courses with enrollment buttons
- `EnrollmentsTable`: Shows user's enrollments with status
- `ProtectedRoute`: Route protection based on authentication

### Authentication
- JWT token stored in localStorage
- Automatic token validation
- Protected routes for authenticated users

---

## 🔐 API Endpoints

### Auth Service (`/api/auth`)
- `POST /register` - User registration
- `POST /login` - User login
- `GET /me` - Get current user (requires JWT)
- `GET /health` - Health check

### Course Service (`/api/courses`)
- `GET /` - List all courses
- `GET /{id}` - Get course by ID
- `POST /` - Create new course
- `PUT /{id}` - Update course
- `DELETE /{id}` - Delete course
- `GET /health` - Health check

### Enrollment Service (`/api/enroll`)
- `GET /` - Get user enrollments (requires JWT)
- `POST /` - Enroll in course (requires JWT)
- `GET /health` - Health check

---

## 🗄️ Database Schema

### Users Collection (`cb_auth`)
```json
{
  "_id": "ObjectId",
  "name": "string",
  "email": "string",
  "password": "string (bcrypt hashed)"
}
```

### Courses Collection (`cb_courses`)
```json
{
  "_id": "ObjectId",
  "title": "string",
  "description": "string",
  "instructor": "string",
  "duration": "number (hours)",
  "price": "number"
}
```

### Enrollments Collection (`cb_enrollments`)
```json
{
  "_id": "ObjectId",
  "userId": "string",
  "courseId": "string",
  "courseTitle": "string",
  "enrolledAt": "datetime",
  "status": "string (active/completed/cancelled)"
}
```

---

## 🧪 Testing the Application

### 1. Register a new user
```bash
curl -X POST http://localhost:8081/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@example.com","password":"password123"}'
```

### 2. Login
```bash
curl -X POST http://localhost:8081/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"john@example.com","password":"password123"}'
```

### 3. Get courses
```bash
curl http://localhost:8082/api/courses/
```

### 4. Enroll in a course (replace TOKEN with actual JWT)
```bash
curl -X POST http://localhost:8083/api/enroll/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"courseId":"1"}'
```

---

## 🚀 Production Deployment

### Environment Variables for Production
```env
# Backend
MONGO_URI=mongodb://your-production-mongo-uri
JWT_SECRET=your-strong-jwt-secret

# Frontend
VITE_AUTH_API=https://api.cloudblitz.in/api/auth
VITE_COURSE_API=https://api.cloudblitz.in/api/courses
VITE_ENROLL_API=https://api.cloudblitz.in/api/enroll
```

### Build Commands
```bash
# Frontend
cd frontend
npm run build

# Backend services
cd backend/auth-service && ./mvnw clean package
cd backend/course-service && ./mvnw clean package
cd backend/enrollment-service && ./mvnw clean package
```

---

## 🐛 Troubleshooting

### Common Issues

1. **CORS Errors**: Ensure all backend services have CORS configured for `https://cloudblitz.in`

2. **JWT Token Issues**: Verify JWT_SECRET is the same across auth and enrollment services

3. **MongoDB Connection**: Check MongoDB is running and accessible

4. **Port Conflicts**: Ensure ports 8081-8083 and 5173 are available

### Logs
```bash
# View all service logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f auth-service
docker-compose logs -f course-service
docker-compose logs -f enrollment-service
docker-compose logs -f frontend
```

---

## 📝 Development Notes

- All services use consistent JSON response format: `{ success: boolean, data: any, error: string }`
- JWT tokens expire after 24 hours
- Demo courses are automatically seeded when course service starts
- Frontend uses absolute URLs for API calls (configurable via environment variables)
- All services include health check endpoints

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

---

## 📄 License

This project is licensed under the MIT License.

---

**This project demonstrates enterprise-level architecture with modern DevOps practices, making it an excellent showcase for interviews!**
