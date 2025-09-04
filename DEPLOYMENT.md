# ML Cogni - Deployment Guide

## 🐳 Local Docker Development

### Prerequisites
- Docker Desktop installed and running
- Git

### Quick Start
1. **Clone and navigate to project**
   ```bash
   cd ml_cogni
   ```

2. **Build and run with Docker Compose**
   ```bash
   docker-compose up --build
   ```

3. **Access the application**
   - Frontend: http://localhost:3000
   - Backend API: http://localhost:8000
   - API Docs: http://localhost:8000/docs

### Individual Services
```bash
# Backend only
docker build -t ml-cogni-backend ./backend
docker run -p 8000:8000 ml-cogni-backend

# Frontend only
docker build -t ml-cogni-frontend ./frontend
docker run -p 3000:3000 ml-cogni-frontend
```

## 🚀 Render Production Deployment

### Backend Deployment
1. **Connect your GitHub repo to Render**
2. **Create a new Web Service**
   - Name: `ml-cogni-backend`
   - Environment: `Python`
   - Build Command: `pip install -r Requirements.txt`
   - Start Command: `python main.py`
   - Health Check Path: `/health`

3. **Environment Variables**
   ```
   PYTHON_VERSION=3.11.0
   PORT=8000
   ```

### Frontend Deployment
1. **Create another Web Service**
   - Name: `ml-cogni-frontend`
   - Environment: `Node`
   - Build Command: `npm ci && npm run build`
   - Start Command: `npm start`
   - Health Check Path: `/`

2. **Environment Variables**
   ```
   NODE_VERSION=18.0.0
   NEXT_PUBLIC_API_URL=https://your-backend-service.onrender.com
   PORT=3000
   ```

### Automatic Deployment
- Both services will auto-deploy on Git push
- Use the `render.yaml` file for infrastructure as code

## 🔧 Environment Configuration

### Backend (.env)
```ini
DATABASE_URL=sqlite:///cibil_database.db
```

### Frontend (Environment Variables)
```bash
NEXT_PUBLIC_API_URL=http://localhost:8000  # Local
NEXT_PUBLIC_API_URL=https://your-backend.onrender.com  # Production
```

## 📁 File Structure
```
ml_cogni/
├── backend/
│   ├── Dockerfile
│   ├── main.py
│   ├── Requirements.txt
│   ├── *.pkl (ML models)
│   ├── *.csv (training data)
│   └── *.db (SQLite database)
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
├── docker-compose.yml
├── render.yaml
└── .dockerignore
```

## 🚨 Troubleshooting

### Common Issues
1. **Port conflicts**: Change ports in docker-compose.yml
2. **Build failures**: Check Requirements.txt and package.json
3. **CORS errors**: Backend CORS is configured for all origins
4. **Model loading**: Ensure all .pkl files are present

### Health Checks
- Backend: `GET /health`
- Frontend: `GET /`

### Logs
```bash
# Docker logs
docker-compose logs backend
docker-compose logs frontend

# Render logs
# Check in Render dashboard
```

## 🔄 CI/CD Pipeline

### GitHub Actions (Optional)
Create `.github/workflows/deploy.yml`:
```yaml
name: Deploy to Render
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to Render
        uses: johnbeynon/render-deploy-action@v0.0.1
        with:
          service-id: ${{ secrets.RENDER_SERVICE_ID }}
          api-key: ${{ secrets.RENDER_API_KEY }}
```

## 📊 Monitoring

### Health Endpoints
- Backend: `/health` - System status, models, database
- Frontend: `/` - Application status

### Metrics
- Model loading status
- Database connectivity
- API response times
- Error rates

## 🔒 Security Notes

### Production Considerations
1. **Environment Variables**: Never commit .env files
2. **CORS**: Configure specific origins in production
3. **Rate Limiting**: Implement API rate limiting
4. **Authentication**: Add JWT or OAuth for production APIs
5. **HTTPS**: Render provides SSL certificates automatically

### Database
- SQLite for development
- PostgreSQL for production (update DATABASE_URL)
- Regular backups of model files and data

## 📝 Notes

- All ML models (.pkl) and data files (.csv, .db) are preserved
- Claude-related files have been removed
- Environment files are generated automatically in Docker
- Frontend automatically connects to backend via environment variables
- Health checks ensure services are ready before accepting traffic
