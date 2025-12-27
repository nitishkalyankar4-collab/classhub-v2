# ClassHub V2 - Step-by-Step Implementation Guide for VibreCoding Agent

## **BEFORE YOU START**

Read these files in this exact order:
1. `V2_SYSTEM_PROMPT.md` - Complete architecture & requirements
2. `README.md` - Full feature specifications
3. `QUICK_START.md` - Quick overview
4. This file - Implementation roadmap

---

## **PHASE 0: PROJECT SETUP (Day 1 Morning)**

### Step 1: Initialize Backend Repository

```bash
# Create backend directory
mkdir classhub-backend && cd classhub-backend

# Initialize NestJS project
npm create nest-app . --package-manager npm

# Install essential dependencies
npm install --save \  
  @nestjs/typeorm typeorm pg \  
  @nestjs/jwt @nestjs/passport passport passport-jwt bcryptjs \  
  @nestjs/websockets @nestjs/platform-socket.io socket.io \  
  axios @google/generative-ai \  
  @nestjs/schedule \  
  redis ioredis @nestjs/cache-manager cache-manager-redis-store \  
  class-validator class-transformer \  
  zod \  
  dotenv \  
  @nestjs/config \  
  multer aws-sdk \  
  nodemailer \  
  winston

npm install --save-dev \  
  @types/jest @types/node @types/express \  
  jest @nestjs/testing ts-jest ts-loader \  
  @types/bcryptjs @types/multer \  
  prettier eslint
```

### Step 2: Initialize Frontend Repository

```bash
# Create frontend directory
mkdir classhub-frontend && cd classhub-frontend

# Initialize Next.js project
npx create-next-app . --typescript --tailwind --app

# Install dependencies
npm install --save \  
  axios socket.io-client \  
  @reduxjs/toolkit react-redux \  
  @tanstack/react-query \  
  zod react-hook-form \  
  next-auth [next-auth@beta] \  
  date-fns clsx tailwind-merge \  
  zustand \  
  @hookform/resolvers

npm install --save-dev \  
  jest @testing-library/react @testing-library/jest-dom \  
  cypress @cypress/webpack-dev-server \  
  typescript
```

### Step 3: Create `.env` Files

**Backend (.env)**
```env
# Server
PORT=3000
NODE_ENV=development

# Database
DATABASE_URL=postgresql://admin:secure_password@localhost:5432/classhub

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your-super-secret-jwt-key-min-32-chars
JWT_EXPIRATION=900  # 15 minutes
JWT_REFRESH_EXPIRATION=604800  # 7 days

# OTP
OTP_EXPIRATION=600  # 10 minutes

# Email
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
SMTP_FROM=noreply@classhub.io

# AWS S3
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your-key
AWS_SECRET_ACCESS_KEY=your-secret
AWS_S3_BUCKET=classhub-uploads
AWS_S3_REGION=us-east-1

# Gemini API
GEMINI_API_KEY=your-gemini-api-key
GEMINI_MODEL=gemini-2.0-flash

# File Upload
MAX_FILE_SIZE=52428800  # 50MB
ALLOWED_FILE_TYPES=pdf,docx,pptx,jpg,jpeg,png,mp4,mov

# Logging
LOG_LEVEL=debug
```

**Frontend (.env.local)**
```env
NEXT_PUBLIC_API_URL=http://localhost:3000/api
NEXT_PUBLIC_WS_URL=ws://localhost:3000
NEXT_PUBLIC_APP_NAME=ClassHub
NEXT_PUBLIC_APP_VERSION=2.0.0
```

---

## **PHASE 1: DATABASE SETUP (Day 1 Afternoon)**

### Step 1: Create PostgreSQL Database

```bash
# Using Docker (recommended)
docker run --name classhub-db \  
  -e POSTGRES_USER=admin \  
  -e POSTGRES_PASSWORD=secure_password \  
  -e POSTGRES_DB=classhub \  
  -p 5432:5432 \  
  -d postgres:15-alpine

# Or using system PostgreSQL
createdb -U postgres classhub
```

### Step 2: Run Database Schema

```bash
# In backend directory

# Create entities folder structure
mkdir -p src/entities

# Copy the SQL schema from V2_SYSTEM_PROMPT.md into a migration file
# Generate migrations (Typeorm)
npm run typeorm migration:generate -- -n InitialSchema

# Run migrations
npm run typeorm migration:run

# Verify tables created
psql -U admin -d classhub -c "\dt"
```

### Step 3: Set Up Redis Cache

```bash
# Using Docker
docker run --name classhub-redis \  
  -p 6379:6379 \  
  -d redis:7-alpine

# Or install locally
brew install redis  # macOS
sudo apt-get install redis-server  # Ubuntu
choco install redis  # Windows
```

---

## **PHASE 2: AUTHENTICATION SYSTEM (Day 2)**

### Step 1: Create Auth Module

```bash
# Generate auth module
cd backend
nest generate module auth
nest generate service auth
nest generate controller auth

# Create related files
touch src/auth/strategies/jwt.strategy.ts
touch src/auth/guards/jwt.guard.ts
touch src/auth/guards/roles.guard.ts
touch src/auth/decorators/roles.decorator.ts
```

### Step 2: Implement JWT Strategy

**src/auth/strategies/jwt.strategy.ts**
```typescript
import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(payload: any) {
    return {
      userId: payload.sub,
      email: payload.email,
      role: payload.role,
    };
  }
}
```

### Step 3: Implement Auth Service

**src/auth/auth.service.ts**
```typescript
@Injectable()
export class AuthService {
  constructor(
    @InjectRepository(User) private users: Repository<User>,
    private jwtService: JwtService,
    private mailService: MailService,
  ) {}

  async register(registerDto: RegisterDto) {
    const { email, password, name, role } = registerDto;

    // Check if user exists
    const existing = await this.users.findOne({ where: { email } });
    if (existing) throw new BadRequestException('Email already registered');

    // Hash password
    const passwordHash = await bcrypt.hash(password, 12);

    // Create user
    const user = this.users.create({
      email,
      name,
      role,
      password_hash: passwordHash,
    });

    await this.users.save(user);

    // Generate OTP
    const otp = Math.random().toString().slice(2, 8);
    user.otp = otp;
    user.otp_expires_at = new Date(Date.now() + 10 * 60 * 1000);
    await this.users.save(user);

    // Send OTP email
    await this.mailService.sendOtpEmail(email, otp);

    return {
      userId: user.id,
      message: 'OTP sent to email',
    };
  }

  async verifyOtp(email: string, otp: string) {
    const user = await this.users.findOne({ where: { email } });

    if (!user || user.otp !== otp) {
      throw new BadRequestException('Invalid OTP');
    }

    if (new Date() > user.otp_expires_at) {
      throw new BadRequestException('OTP expired');
    }

    user.is_verified = true;
    user.otp = null;
    await this.users.save(user);

    // Generate JWT tokens
    const accessToken = this.jwtService.sign(
      { sub: user.id, email: user.email, role: user.role },
      { expiresIn: '15m' },
    );

    const refreshToken = this.jwtService.sign(
      { sub: user.id },
      { expiresIn: '7d' },
    );

    return {
      accessToken,
      refreshToken,
      user: { id: user.id, email: user.email, name: user.name, role: user.role },
    };
  }

  async login(email: string, password: string) {
    const user = await this.users.findOne({ where: { email } });

    if (!user || !(await bcrypt.compare(password, user.password_hash))) {
      throw new UnauthorizedException('Invalid credentials');
    }

    if (!user.is_verified) {
      throw new UnauthorizedException('Email not verified');
    }

    const accessToken = this.jwtService.sign(
      { sub: user.id, email: user.email, role: user.role },
      { expiresIn: '15m' },
    );

    const refreshToken = this.jwtService.sign(
      { sub: user.id },
      { expiresIn: '7d' },
    );

    return { accessToken, refreshToken };
  }
}
```

### Step 4: Implement Auth Controller

**src/auth/auth.controller.ts**
```typescript
@Controller('api/auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('register')
  register(@Body() registerDto: RegisterDto) {
    return this.authService.register(registerDto);
  }

  @Post('verify-otp')
  verifyOtp(@Body() { email, otp }: { email: string; otp: string }) {
    return this.authService.verifyOtp(email, otp);
  }

  @Post('login')
  login(@Body() loginDto: LoginDto) {
    return this.authService.login(loginDto.email, loginDto.password);
  }

  @Post('refresh-token')
  refreshToken(@Body() { refreshToken }: { refreshToken: string }) {
    return this.authService.refreshToken(refreshToken);
  }

  @UseGuards(JwtAuthGuard)
  @Get('me')
  getMe(@Request() req) {
    return req.user;
  }
}
```

---

## **PHASE 3: CLASSROOM MODULE (Day 3)**

### Step 1: Generate Classroom Module

```bash
cd backend
nest generate resource classrooms
nest generate service classrooms
nest generate controller classrooms
```

### Step 2: Implement Classroom Service

Use the example from `V2_SYSTEM_PROMPT.md` Part 12

### Step 3: Implement WebSocket Gateway for Chat

**src/classrooms/messages.gateway.ts**
```typescript
import { WebSocketGateway, WebSocketServer, SubscribeMessage, OnGatewayConnection, OnGatewayDisconnect } from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway({
  cors: {
    origin: process.env.FRONTEND_URL,
    credentials: true,
  },
})
export class MessagesGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer() server: Server;

  handleConnection(client: Socket) {
    const { classroomId, token } = client.handshake.query;
    client.join(`classroom:${classroomId}`);
    this.server.to(`classroom:${classroomId}`).emit('user:online', {
      userId: client.id,
      timestamp: new Date(),
    });
  }

  handleDisconnect(client: Socket) {
    this.server.sockets.sockets.forEach((socket) => {
      if (socket.id === client.id) {
        const rooms = Array.from(socket.rooms);
        rooms.forEach((room) => {
          if (room.startsWith('classroom:')) {
            this.server.to(room).emit('user:offline', {
              userId: client.id,
            });
          }
        });
      }
    });
  }

  @SubscribeMessage('classroom:message:send')
  async handleMessage(client: Socket, payload: any) {
    const { classroomId, content, messageType } = payload;

    // Save to database
    // ...

    // Broadcast to classroom
    this.server.to(`classroom:${classroomId}`).emit('classroom:message:new', {
      id: generateId(),
      classroomId,
      senderId: client.id,
      content,
      messageType,
      timestamp: new Date(),
      reactions: {},
    });
  }

  @SubscribeMessage('classroom:typing')
  handleTyping(client: Socket, payload: { classroomId: string }) {
    this.server.to(`classroom:${payload.classroomId}`).emit('classroom:typing', {
      userId: client.id,
    });
  }
}
```

---

## **PHASE 4: ASSIGNMENTS & SUBMISSIONS (Day 4)**

```bash
nest generate resource assignments
nest generate service submissions
```

Implement using the endpoints from `V2_SYSTEM_PROMPT.md` Part 3

---

## **PHASE 5: AI TUTOR INTEGRATION (Day 5)**

### Step 1: Create AI Service

**src/ai/ai.service.ts**
```typescript
import { GoogleGenerativeAI } from '@google/generative-ai';

@Injectable()
export class AiService {
  private genAI: GoogleGenerativeAI;

  constructor() {
    this.genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
  }

  async generateResponse(
    userMessage: string,
    conversationHistory: any[],
    classroom: any,
  ): Promise<string> {
    const model = this.genAI.getGenerativeModel({
      model: process.env.GEMINI_MODEL,
      systemInstruction: this.buildSystemPrompt(classroom),
    });

    const chat = model.startChat({
      history: conversationHistory.map((msg) => ({
        role: msg.role,
        parts: [{ text: msg.content }],
      })),
    });

    const result = await chat.sendMessage(userMessage);
    return result.response.text();
  }

  private buildSystemPrompt(classroom: any): string {
    return `
You are an AI tutor assisting students in ${classroom.subject}.
Grade Level: ${classroom.gradeLevel}
Curriculum: NCERT
Covered Topics: ${classroom.coveredTopics?.join(', ') || 'General'}

Respond in simple, clear language suitable for ${classroom.gradeLevel} students.
Use examples, analogies, and step-by-step explanations.
Do NOT provide answers to current assignments or homework.
Always encourage learning and critical thinking.
    `;
  }
}
```

### Step 2: Create AI Chat Gateway

**src/ai/chat.gateway.ts**
```typescript
@WebSocketGateway({
  cors: { origin: process.env.FRONTEND_URL },
})
export class ChatGateway {
  constructor(private aiService: AiService) {}

  @SubscribeMessage('ai:message:send')
  async handleAiMessage(client: Socket, payload: any) {
    try {
      // Get conversation history from database
      const history = await this.getConversationHistory(payload.conversationId);

      // Get AI response
      const response = await this.aiService.generateResponse(
        payload.message,
        history,
        payload.classroom,
      );

      // Save to database
      // ...

      // Send response back
      client.emit('ai:response', {
        conversationId: payload.conversationId,
        message: response,
        timestamp: new Date(),
      });
    } catch (error) {
      client.emit('ai:error', { message: 'Failed to get response' });
    }
  }
}
```

---

## **PHASE 6: FILE UPLOAD & STORAGE (Day 6)**

### Step 1: Setup S3 Service

**src/storage/s3.service.ts**
```typescript
import { S3Client, PutObjectCommand, DeleteObjectCommand } from '@aws-sdk/client-s3';

@Injectable()
export class S3Service {
  private s3Client: S3Client;

  constructor() {
    this.s3Client = new S3Client({
      region: process.env.AWS_REGION,
      credentials: {
        accessKeyId: process.env.AWS_ACCESS_KEY_ID,
        secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
      },
    });
  }

  async uploadFile(file: Express.Multer.File, folder: string): Promise<string> {
    const key = `${folder}/${Date.now()}-${file.originalname}`;

    const command = new PutObjectCommand({
      Bucket: process.env.AWS_S3_BUCKET,
      Key: key,
      Body: file.buffer,
      ContentType: file.mimetype,
      ServerSideEncryption: 'AES256',
    });

    await this.s3Client.send(command);
    return `https://${process.env.AWS_S3_BUCKET}.s3.amazonaws.com/${key}`;
  }

  async deleteFile(url: string): Promise<void> {
    const key = url.split('.com/')[1];
    const command = new DeleteObjectCommand({
      Bucket: process.env.AWS_S3_BUCKET,
      Key: key,
    });
    await this.s3Client.send(command);
  }
}
```

### Step 2: Upload Endpoint

```typescript
@Post('upload')
@UseInterceptors(FileInterceptor('file'))
async uploadFile(@UploadedFile() file: Express.Multer.File) {
  const url = await this.s3Service.uploadFile(file, 'uploads');
  return { url, filename: file.originalname };
}
```

---

## **PHASE 7: FRONTEND - AUTHENTICATION (Day 7)**

### Step 1: Create Auth Context

**frontend/lib/context/auth-context.tsx**
```typescript
const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState(null);
  const [token, setToken] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Check if user is already logged in
    const token = localStorage.getItem('accessToken');
    if (token) {
      // Validate token and fetch user
      fetchUser(token);
    }
    setLoading(false);
  }, []);

  const login = async (email: string, password: string) => {
    const response = await axios.post('/api/auth/login', { email, password });
    localStorage.setItem('accessToken', response.data.accessToken);
    localStorage.setItem('refreshToken', response.data.refreshToken);
    setUser(response.data.user);
    setToken(response.data.accessToken);
  };

  const logout = () => {
    localStorage.removeItem('accessToken');
    localStorage.removeItem('refreshToken');
    setUser(null);
    setToken(null);
  };

  return (
    <AuthContext.Provider value={{ user, token, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
}
```

### Step 2: Create Auth Forms

**frontend/components/auth/login-form.tsx**
```typescript
export function LoginForm() {
  const router = useRouter();
  const { login } = useAuth();
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');

  const form = useForm<LoginFormValues>({
    resolver: zodResolver(loginSchema),
  });

  const onSubmit = async (values: LoginFormValues) => {
    try {
      setLoading(true);
      await login(values.email, values.password);
      router.push('/dashboard');
    } catch (err) {
      setError('Invalid email or password');
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
      {/* Form fields */}
    </form>
  );
}
```

---

## **PHASE 8: FRONTEND - DASHBOARD & CLASSROOMS (Day 8-9)**

```bash
frontend/components/
├── dashboard/
│   ├── teacher-dashboard.tsx
│   ├── student-dashboard.tsx
│   ├── parent-dashboard.tsx
│   └── admin-dashboard.tsx
├── classrooms/
│   ├── classroom-view.tsx
│   ├── create-classroom-modal.tsx
│   ├── join-classroom-modal.tsx
│   └── classroom-tabs.tsx
├── assignments/
│   ├── assignment-list.tsx
│   ├── assignment-detail.tsx
│   ├── submission-form.tsx
│   └── gradebook.tsx
└── chat/
    ├── classroom-chat.tsx
    └── ai-tutor-chat.tsx
```

Implement using components examples from the frontend structure.

---

## **PHASE 9: TESTING (Day 10)**

### Backend Tests

```bash
cd backend
npm run test  # Run all tests
npm run test:cov  # Coverage report
```

### Frontend Tests

```bash
cd frontend
npm run test  # Run tests
npm run test:e2e  # E2E with Cypress
```

---

## **PHASE 10: DEPLOYMENT (Day 11)**

### Docker Setup

```bash
# Build images
docker-compose build

# Start services
docker-compose up -d

# Check status
docker-compose ps
```

### Deploy to AWS

```bash
# Create ECR repositories
aws ecr create-repository --repository-name classhub-backend
aws ecr create-repository --repository-name classhub-frontend

# Push images
docker tag classhub:latest [account-id].dkr.ecr.us-east-1.amazonaws.com/classhub-backend:latest
docker push [account-id].dkr.ecr.us-east-1.amazonaws.com/classhub-backend:latest

# Deploy with Kubernetes
kubectl apply -f k8s/deployment.yaml
```

---

## **QUICK COMMAND REFERENCE**

```bash
# Start development
npm run dev  # Backend
npm run dev  # Frontend (from frontend dir)

# Database
npm run typeorm migration:generate -- -n MigrationName
npm run typeorm migration:run

# Testing
npm run test
npm run test:cov

# Build for production
npm run build
npm run start:prod

# Docker
docker-compose up -d
docker-compose down

# View logs
docker-compose logs -f backend
```

---

## **CHECKLIST: BEFORE GOING LIVE**

- [ ] All 50+ API endpoints implemented
- [ ] Authentication working (JWT, OTP)
- [ ] Database with all 15+ tables
- [ ] WebSocket real-time messaging
- [ ] AI Tutor integration (Gemini)
- [ ] File uploads to S3
- [ ] Email notifications sending
- [ ] Student chat system working
- [ ] Parent dashboard complete
- [ ] Admin analytics dashboard
- [ ] Unit tests >80% coverage
- [ ] Integration tests passing
- [ ] E2E tests passing
- [ ] Performance optimized
- [ ] Security audit passed
- [ ] Documentation complete
- [ ] CI/CD pipeline working
- [ ] Docker images built
- [ ] Environment variables set
- [ ] Database backups configured
- [ ] Monitoring/alerting setup
- [ ] HTTPS/SSL enabled
- [ ] Load tested 1000+ users
- [ ] Error handling comprehensive
- [ ] Logging implemented

---

**Total Development Time: 11 Days**  
**Ready for production deployment on Day 12**

✅ **ClassHub V2 Complete & Production-Ready**
