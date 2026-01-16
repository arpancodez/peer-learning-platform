# Setup Guide for Peer Learning Platform

## Local Development Setup

### Prerequisites
- Node.js 16+
- MongoDB Atlas account
- Redis installed

### Installation Steps

1. Clone the repository
2. Install dependencies: `npm install`
3. Configure environment variables
4. Start MongoDB and Redis
5. Run development server: `npm run dev`

### Environment Variables

```
MONGODB_URI=mongodb+srv://...
REDIS_URL=redis://localhost:6379
OPENAI_KEY=sk-...
JWT_SECRET=your_secret
```

### Running Tests

```bash
npm run test
```

### Building for Production

```bash
npm run build
npm start
```


## Troubleshooting Setup

- If MongoDB connection fails, ensure MongoDB Atlas is running
- Check that all environment variables are correctly set
- Clear node_modules and reinstall if npm install fails
- Use `npm run dev` to start development server
- Access frontend at http://localhost:3000
- Backend API runs on http://localhost:5000

## Environment Variables Checklist

- MONGODB_URI is set and accessible
- JWT_SECRET is a secure random string
- OPENAI_API_KEY is valid (for AI features)
- NODE_ENV is set to 'development' or 'production'
