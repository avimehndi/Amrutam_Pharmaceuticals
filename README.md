# CI/CD Pipeline for Modern Web Stack

## 📋 Overview

This repository contains a complete CI/CD pipeline solution for a modern web application stack consisting of:
- **Frontend**: React application
- **Backend**: Django REST API
- **Mobile**: React Native application
- **Database**: PostgreSQL with Redis for caching
- **Authentication**: Firebase Authentication
- **Deployment**: DigitalOcean Kubernetes clusters
- **Monitoring**: Prometheus & Grafana with Slack notifications

## 🏗️ Architecture
    
The CI/CD pipeline follows industry best practices with environment segregation, automated testing, containerized deployments, and comprehensive monitoring.

[View the complete architecture diagram]

### Key Components:
1. **Source Control**: GitHub repository with branch protection
2. **CI/CD Engine**: GitHub Actions workflows
3. **Containerization**: Docker for all services
4. **Orchestration**: Kubernetes for container management
5. **Environment Segregation**: Separate staging and production environments
6. **Monitoring**: Prometheus metrics collection with Grafana visualization
7. **Alerting**: Slack notifications for build status and system alerts

## 📁 Project Structure

```
project-root/
my-web-app/
├── frontend/           # React frontend source
├── backend/            # Node.js backend source
├── mobile/             # React Native app (if applicable)
├── docker-compose.yml  # Compose file for local/development orchestration
└── .github/workflows/ci-cd.yml  # CI/CD workflow definition
├── monitoring/
│   ├── prometheus/
│   │   ├── prometheus.yml
│   │   └── alert.rules.yml
│   ├── grafana/
│   │   ├── datasources/
│   │   │   └── datasource.yml
│   │   └── dashboards/
│   │       └── dashboard.json
│   └── docker-compose.monitoring.yml
```

## 🚀 How the CI/CD Pipeline Works

### 1. Code Commit & Trigger
- Developer pushes code to a feature branch
- Creates Pull Request to `main` branch
- GitHub Actions automatically triggers on PR creation

### 2. Parallel Testing & Linting
The pipeline runs three parallel jobs:

#### Frontend Testing (`lint-and-test-frontend`)
- Sets up Node.js environment
- Installs npm dependencies with caching
- Runs ESLint for code quality checks
- Executes Jest tests with coverage reports
- Notifies Slack if tests fail

#### Backend Testing (`lint-and-test-backend`)
- Sets up Python environment with PostgreSQL service
- Installs Python dependencies with pip caching
- Runs Flake8 linting for PEP 8 compliance
- Executes Django tests with coverage
- Notifies Slack if tests fail

#### Mobile Testing (`lint-and-test-mobile`)
- Sets up Node.js for React Native
- Installs dependencies
- Runs ESLint and Jest tests
- Notifies Slack if tests fail

### 3. Docker Image Building
After all tests pass:
- Builds Docker images for frontend and backend
- Tags images with commit SHA for versioning
- Pushes images to GitHub Container Registry
- Uses build cache for faster subsequent builds

### 4. Staging Deployment
- Connects to DigitalOcean Kubernetes staging cluster
- Updates deployment with new image tags
- Waits for rollout completion with health checks
- Runs smoke tests to verify deployment
- Automatically rolls back if deployment fails
- Notifies Slack of deployment status

### 5. Production Deployment
After successful staging deployment:
- Deploys to production Kubernetes cluster
- Performs comprehensive health checks
- Notifies Slack of production deployment
- Automatic rollback on failure

## 🔧 Setup Instructions

### Prerequisites
- GitHub repository
- DigitalOcean account with Kubernetes clusters
- Slack workspace with webhook URL
- Firebase project for authentication

### 1. Repository Setup

1. Clone this repository structure to your GitHub repo
2. Copy all configuration files to appropriate locations
3. Update placeholders in configuration files with your actual values

### 2. GitHub Secrets Configuration

Navigate to your GitHub repository → Settings → Secrets and variables → Actions

Add these required secrets:

```bash
# DigitalOcean Configuration
DIGITALOCEAN_ACCESS_TOKEN=your_do_token_here
STAGING_CLUSTER_NAME=your-staging-cluster-name
PRODUCTION_CLUSTER_NAME=your-production-cluster-name

# Slack Integration
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK

# Database URLs
STAGING_DATABASE_URL=postgresql://user:pass@host:5432/staging_db
PRODUCTION_DATABASE_URL=postgresql://user:pass@host:5432/prod_db

# Django Secret Keys
STAGING_SECRET_KEY=your-staging-secret-key
PRODUCTION_SECRET_KEY=your-production-secret-key

# Firebase Configuration (JSON format)
FIREBASE_STAGING_CONFIG={"apiKey":"...","authDomain":"..."}
FIREBASE_PRODUCTION_CONFIG={"apiKey":"...","authDomain":"..."}
```

### 3. Kubernetes Cluster Setup

#### Create Namespaces
```bash
kubectl apply -f k8s/namespaces-quotas.yml
```

#### Deploy Secrets
1. Update `k8s/secrets-template.yml` with your actual values
2. Apply secrets:
```bash
kubectl apply -f k8s/secrets-template.yml
```

#### Deploy Applications
```bash
# Deploy Redis
kubectl apply -f k8s/redis-deployment.yml

# Deploy Backend
kubectl apply -f k8s/backend-deployment.yml

# Deploy Frontend
kubectl apply -f k8s/frontend-deployment.yml
```

### 4. Monitoring Setup

#### Local Development Monitoring
```bash
cd monitoring/
docker-compose -f monitoring-docker-compose.yml up -d
```

Access monitoring services:
- Prometheus: http://localhost:9090
- Grafana: http://localhost:3001 (admin/admin123)
- Alertmanager: http://localhost:9093

#### Production Monitoring
Deploy monitoring stack to your Kubernetes cluster following the same pattern as application deployments.

## 🔄 Workflow Files Explained

### Main CI/CD Workflow (`.github/workflows/main-cicd-workflow.yml`)
This is the core pipeline that runs on every push to main branch:

1. **Linting & Testing Jobs**: Run in parallel for all three components
2. **Build & Push**: Creates and pushes Docker images
3. **Deploy Staging**: Deploys to staging environment with health checks
4. **Deploy Production**: Deploys to production after staging success

### Rollback Workflow (`.github/workflows/rollback-workflow.yml`)
Manual rollback workflow that can be triggered from GitHub Actions UI:

- Choose environment (staging/production)
- Optionally specify revision to rollback to
- Performs rollback and health checks
- Notifies team via Slack

## 🐳 Docker Configuration

### Frontend Dockerfile
- Multi-stage build for optimized image size
- Uses Node.js for building React app
- Serves via Nginx in production
- Includes health checks and security headers

### Backend Dockerfile
- Python 3.11 slim base image
- Installs system dependencies
- Uses Gunicorn for production serving
- Non-root user for security
- Health check endpoint

### Docker Compose
Local development environment with:
- PostgreSQL database
- Redis for caching
- Hot-reload for development
- Environment variable configuration

## 📊 Monitoring & Alerting

### Prometheus Configuration
Collects metrics from:
- Kubernetes cluster components
- Application metrics (custom endpoints)
- System metrics via Node Exporter
- Database performance metrics

### Alert Rules
Configured alerts for:
- **High CPU Usage**: Warning at 70%, Critical at 90%
- **Memory Usage**: Warning at 80%
- **Disk Space**: Warning at 85%
- **Application Downtime**: Critical within 1 minute
- **High Response Times**: Warning for 95th percentile > 1s
- **Error Rates**: Critical if >5% for 5 minutes

### Slack Notifications
Different channels for different alert types:
- `#general-alerts`: All alerts
- `#critical-alerts`: Critical system issues
- `#warning-alerts`: Warning level alerts
- `#deployments`: Deployment status updates
- `#ci-cd-alerts`: Build and test failures

## 🔐 Security Considerations

### Secrets Management
- All sensitive data stored in Kubernetes secrets
- GitHub repository secrets for CI/CD variables
- Environment-specific configurations
- No hardcoded credentials in code

### Container Security
- Non-root users in containers
- Minimal base images
- Regular security updates
- Resource limits and quotas

### Network Security
- Service-to-service communication within cluster
- Ingress controllers for external access
- TLS encryption for all external traffic

## 🚨 Rollback Strategy

### Automatic Rollbacks
- Failed health checks trigger automatic rollback
- Deployment timeouts initiate rollback
- Critical alerts can trigger rollback (configurable)

### Manual Rollbacks
- GitHub Actions workflow for manual rollback
- Choose specific revision or rollback to previous
- Health checks after rollback completion
- Team notification via Slack

### Rollback Process
1. Identify the issue (monitoring alerts or manual detection)
2. Trigger rollback workflow from GitHub Actions
3. System automatically reverts to previous stable version
4. Health checks verify rollback success
5. Team gets notified of rollback completion

## 🧪 Testing Strategy

### Frontend Testing
- **Linting**: ESLint with React-specific rules
- **Unit Tests**: Jest with React Testing Library
- **Coverage**: Minimum 70% code coverage required
- **Build Tests**: Verify production build succeeds

### Backend Testing
- **Linting**: Flake8 for Python code style
- **Unit Tests**: pytest with Django test framework
- **Coverage**: Coverage.py for test coverage reporting
- **Integration Tests**: Database and API endpoint tests

### Mobile Testing
- **Linting**: ESLint for React Native
- **Unit Tests**: Jest for component and logic testing
- **Build Tests**: Verify builds complete successfully

## 🚀 Deployment Strategy

### Environment Segregation
- **Staging**: Testing environment identical to production
- **Production**: Live environment serving real users
- Separate databases, secrets, and configurations
- Resource quotas to prevent resource conflicts

### Deployment Process
1. Code merged to main branch
2. Automated tests run (fail-fast approach)
3. Docker images built and tagged
4. Deploy to staging first
5. Run staging smoke tests
6. Deploy to production after staging success
7. Production health checks
8. Team notification

### Zero-Downtime Deployments
- Rolling updates with Kubernetes
- Health checks ensure traffic only goes to healthy pods
- Gradual rollout with configurable pace
- Automatic rollback on failure

## 📈 Monitoring Metrics

### System Metrics
- CPU usage percentage
- Memory utilization
- Disk space usage
- Network I/O

### Application Metrics
- HTTP request rates
- Response times (percentiles)
- Error rates by status code
- Active user sessions

### Business Metrics
- User authentication success rates
- API endpoint performance
- Database query performance
- Firebase integration health

## 🛠️ Troubleshooting

### Common Issues

#### Build Failures
- Check GitHub Actions logs
- Verify dependencies in package.json/requirements.txt
- Ensure test coverage meets minimum requirements

#### Deployment Failures
- Check Kubernetes cluster status
- Verify secrets are correctly configured
- Check image tags and registry access

#### Health Check Failures
- Verify application starts correctly
- Check database connectivity
- Ensure all required environment variables are set

#### Monitoring Issues
- Verify Prometheus can scrape metrics endpoints
- Check Alertmanager configuration
- Confirm Slack webhook URL is correct

### Getting Help
1. Check GitHub Actions logs for detailed error messages
2. Review Kubernetes pod logs: `kubectl logs -f pod-name -n namespace`
3. Check monitoring dashboards for system health
4. Review Slack notifications for alerts

## 📚 Additional Resources

### Documentation Links
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [DigitalOcean Kubernetes Guide](https://docs.digitalocean.com/products/kubernetes/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)

### Learning Resources
- [CI/CD Best Practices](https://docs.github.com/en/actions/guides)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Kubernetes Deployment Strategies](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/new-feature`
3. Make your changes
4. Run tests locally: `npm test` or `python manage.py test`
5. Commit your changes: `git commit -am 'Add new feature'`
6. Push to the branch: `git push origin feature/new-feature`
7. Create a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support

If you encounter any issues or have questions:

1. Check the troubleshooting section above
2. Review the GitHub Issues for similar problems
3. Create a new issue with detailed information about your problem
4. Include logs, error messages, and steps to reproduce


---

**Note**: This CI/CD pipeline is designed to be practical and executable. All configurations can be customized based on your specific requirements and infrastructure setup.