# Complete Setup Guide - Peer Learning Platform

## Quick Start (5 minutes)

```bash
# Clone and setup
git clone https://github.com/arpancodez/peer-learning-platform
cd peer-learning-platform

# Backend setup
cd backend
npm install
cp .env.example .env  # Fill in your credentials
npm run dev

# Frontend setup (new terminal)
cd ../frontend  
npm install
npm run dev
```

Visit http://localhost:3000

## Backend Server File Structure

### src/index.ts - Express + Socket.io Server
```typescript
import express from 'express';
import { Server as SocketIOServer } from 'socket.io';
import cors from 'cors';
import mongoose from 'mongoose';
import dotenv from 'dotenv';
import authRoutes from './routes/auth';
import sessionRoutes from './routes/sessions';
import aiRoutes from './routes/ai';
import quizRoutes from './routes/quizzes';
import { authMiddleware } from './middleware/auth';

dotenv.config();
const app = express();
const io = new SocketIOServer(4000, {
  cors: { origin: process.env.FRONTEND_URL }
});

app.use(cors());
app.use(express.json());

// Database Connection
mongoose.connect(process.env.MONGO_URI!);

// Routes
app.use('/api/auth', authRoutes);
app.use('/api/sessions', authMiddleware, sessionRoutes);
app.use('/api/ai', authMiddleware, aiRoutes);
app.use('/api/quizzes', authMiddleware, quizRoutes);

// Socket.io Handlers
io.on('connection', (socket) => {
  console.log('User connected:', socket.id);
  
  socket.on('drawing:start', (data) => {
    socket.broadcast.emit('drawing:start', data);
  });
  
  socket.on('chat:message', (msg) => {
    io.emit('chat:message', msg);
  });
  
  socket.on('disconnect', () => {
    console.log('User disconnected:', socket.id);
  });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

## Frontend File Structure

### app/page.tsx - Home Page
```typescript
'use client';
import { useState } from 'react';
import SessionCard from '@/components/SessionCard';
import CreateSessionModal from '@/components/CreateSessionModal';

export default function Home() {
  const [sessions, setSessions] = useState([]);
  const [showModal, setShowModal] = useState(false);
  
  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-purple-50">
      <nav className="bg-white shadow-sm">
        <div className="max-w-7xl mx-auto px-6 py-4 flex justify-between items-center">
          <h1 className="text-2xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-blue-600 to-purple-600">
            Peer Learning
          </h1>
          <button 
            onClick={() => setShowModal(true)}
            className="bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700"
          >
            Start Session
          </button>
        </div>
      </nav>
      
      <main className="max-w-7xl mx-auto px-6 py-12">
        <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
          {sessions.map(session => (
            <SessionCard key={session._id} session={session} />
          ))}
        </div>
      </main>
      
      {showModal && <CreateSessionModal onClose={() => setShowModal(false)} />}
    </div>
  );
}
```

### components/Whiteboard.tsx - Canvas Drawing
```typescript
'use client';
import { useRef, useEffect } from 'react';
import { useSocket } from '@/hooks/useSocket';

export default function Whiteboard() {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const { emit, on } = useSocket();
  let isDrawing = false;
  
  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    
    const ctx = canvas.getContext('2d')!;
    canvas.width = canvas.offsetWidth;
    canvas.height = canvas.offsetHeight;
    
    canvas.onmousedown = (e) => {
      isDrawing = true;
      const rect = canvas.getBoundingClientRect();
      emit('drawing:start', {
        x: e.clientX - rect.left,
        y: e.clientY - rect.top
      });
    };
    
    canvas.onmousemove = (e) => {
      if (!isDrawing) return;
      const rect = canvas.getBoundingClientRect();
      ctx.lineTo(e.clientX - rect.left, e.clientY - rect.top);
      ctx.stroke();
    };
    
    canvas.onmouseup = () => {
      isDrawing = false;
      emit('drawing:end', null);
    };
    
    on('drawing:start', (data) => {
      ctx.beginPath();
      ctx.moveTo(data.x, data.y);
    });
  }, [emit, on]);
  
  return (
    <canvas 
      ref={canvasRef}
      className="border-2 border-gray-300 bg-white w-full h-96 cursor-crosshair"
    />
  );
}
```

## Key API Endpoints

### Authentication
- `POST /api/auth/register` - Create account
- `POST /api/auth/login` - Login user
- `POST /api/auth/logout` - Logout

### Sessions
- `GET /api/sessions` - List all sessions
- `POST /api/sessions` - Create session
- `GET /api/sessions/:id` - Get session details
- `POST /api/sessions/:id/join` - Join session

### AI Features
- `POST /api/ai/resolve-doubt` - Get AI answer
- `POST /api/ai/summarize` - Summarize notes
- `POST /api/quizzes/generate` - Generate quiz from notes

## Database Models (MongoDB)

### User Schema
```typescript
interface User {
  _id: ObjectId;
  email: string;
  password: string; // hashed
  name: string;
  avatar?: string;
  bio?: string;
  createdAt: Date;
  updatedAt: Date;
}
```

### Session Schema
```typescript
interface Session {
  _id: ObjectId;
  title: string;
  description: string;
  host: ObjectId; // User reference
  participants: ObjectId[]; // User references
  topic: string;
  startTime: Date;
  endTime?: Date;
  whiteboard: string; // Canvas data
  notes: string;
  status: 'scheduled' | 'active' | 'ended';
  createdAt: Date;
}
```

## Environment Setup

### MongoDB Atlas
1. Create account at mongodb.com/cloud
2. Create free cluster
3. Get connection string
4. Add to MONGO_URI in .env

### OpenAI API
1. Get key from platform.openai.com/api-keys
2. Set OPENAI_API_KEY in .env
3. Ensure account has credits

## Deployment

### Frontend (Vercel)
```bash
vercel --prod
```

### Backend (Railway/Render)
1. Connect GitHub repo
2. Set environment variables
3. Deploy branch

## Real-time Features

Socket.io events:
- `drawing:start` - Begin drawing
- `drawing:continue` - Continue stroke
- `chat:message` - Send message
- `session:join` - User joined
- `quiz:generated` - Quiz created

## Testing

```bash
# Backend
cd backend && npm test

# Frontend  
cd frontend && npm test
```

## Troubleshooting

- Port 5000 already in use: `lsof -i :5000 && kill -9 <PID>`
- MongoDB connection failed: Check IP whitelist in Atlas
- CORS errors: Ensure FRONTEND_URL matches localhost
- Socket.io not connecting: Check WebSocket in browser devtools
