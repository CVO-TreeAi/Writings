# Quant Field Service Business Operations Platform - Pass 3 Implementation Guide

## Executive Summary
**Pass 3** transforms the technical specifications from Pass 2 into actionable implementation guidelines. This includes development environment setup, testing strategies, deployment architecture, CI/CD pipelines, monitoring systems, and go-live procedures for the comprehensive business operations platform.

## 1. Development Environment Setup

### 1.1 Local Development Stack

#### Prerequisites Installation
```bash
# Install Node.js (LTS version)
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install Docker & Docker Compose
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Install PostgreSQL Client
sudo apt-get install postgresql-client

# Install Redis CLI
sudo apt-get install redis-tools
```

#### Docker Development Environment
```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  postgres:
    image: postgres:15
    container_name: quant-postgres-dev
    environment:
      POSTGRES_DB: quant_dev
      POSTGRES_USER: quant_user
      POSTGRES_PASSWORD: dev_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init:/docker-entrypoint-initdb.d
    networks:
      - quant-network

  redis:
    image: redis:7-alpine
    container_name: quant-redis-dev
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - quant-network

  n8n:
    image: n8nio/n8n:latest
    container_name: quant-n8n-dev
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=dev_password
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - WEBHOOK_URL=http://localhost:5678/
    ports:
      - "5678:5678"
    volumes:
      - n8n_data:/home/node/.n8n
      - ./n8n/workflows:/home/node/.n8n/workflows
    depends_on:
      - postgres
    networks:
      - quant-network

  crewai-api:
    build:
      context: ./crewai
      dockerfile: Dockerfile.dev
    container_name: quant-crewai-dev
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - DATABASE_URL=postgresql://quant_user:dev_password@postgres:5432/quant_dev
    ports:
      - "8000:8000"
    volumes:
      - ./crewai:/app
      - /app/node_modules
    depends_on:
      - postgres
      - redis
    networks:
      - quant-network

volumes:
  postgres_data:
  redis_data:
  n8n_data:

networks:
  quant-network:
    driver: bridge
```

#### Environment Configuration
```bash
# .env.development
NODE_ENV=development
PORT=3000

# Database Configuration
DATABASE_URL=postgresql://quant_user:dev_password@localhost:5432/quant_dev
REDIS_URL=redis://localhost:6379

# CrewAI Configuration
CREWAI_API_URL=http://localhost:8000
OPENAI_API_KEY=your_openai_key_here

# N8N Configuration
N8N_API_URL=http://localhost:5678
N8N_API_KEY=your_n8n_api_key

# External Services
TWILIO_ACCOUNT_SID=your_twilio_sid
TWILIO_AUTH_TOKEN=your_twilio_token
SENDGRID_API_KEY=your_sendgrid_key
STRIPE_SECRET_KEY=your_stripe_secret_key
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
AWS_REGION=us-east-1
AWS_S3_BUCKET=quant-dev-storage

# Security
JWT_SECRET=your_jwt_secret_here
ENCRYPTION_KEY=your_encryption_key_here
```

### 1.2 Project Structure
```
quant-platform/
├── packages/
│   ├── api/                    # Backend API
│   │   ├── src/
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   ├── models/
│   │   │   ├── middleware/
│   │   │   ├── routes/
│   │   │   └── utils/
│   │   ├── tests/
│   │   └── package.json
│   ├── web/                    # Frontend Web App
│   │   ├── src/
│   │   │   ├── components/
│   │   │   ├── pages/
│   │   │   ├── hooks/
│   │   │   ├── services/
│   │   │   └── utils/
│   │   ├── public/
│   │   └── package.json
│   ├── mobile/                 # React Native App
│   │   ├── src/
│   │   ├── android/
│   │   ├── ios/
│   │   └── package.json
│   └── shared/                 # Shared Types & Utils
│       ├── src/
│       │   ├── types/
│       │   ├── constants/
│       │   └── utils/
│       └── package.json
├── infrastructure/
│   ├── docker/
│   ├── kubernetes/
│   ├── terraform/
│   └── scripts/
├── database/
│   ├── migrations/
│   ├── seeds/
│   └── init/
├── crewai/
│   ├── agents/
│   ├── tools/
│   ├── workflows/
│   └── Dockerfile.dev
├── n8n/
│   ├── workflows/
│   └── credentials/
└── docs/
    ├── api/
    ├── architecture/
    └── deployment/
```

## 2. Testing Strategies

### 2.1 Unit Testing Framework

#### Backend Unit Tests (Jest + Supertest)
```typescript
// packages/api/tests/services/CustomerService.test.ts
import { CustomerService } from '../../src/services/CustomerService';
import { DatabaseService } from '../../src/services/DatabaseService';
import { CrewAIService } from '../../src/services/CrewAIService';

jest.mock('../../src/services/DatabaseService');
jest.mock('../../src/services/CrewAIService');

describe('CustomerService', () => {
  let customerService: CustomerService;
  let mockDatabase: jest.Mocked<DatabaseService>;
  let mockCrewAI: jest.Mocked<CrewAIService>;

  beforeEach(() => {
    mockDatabase = new DatabaseService() as jest.Mocked<DatabaseService>;
    mockCrewAI = new CrewAIService() as jest.Mocked<CrewAIService>;
    customerService = new CustomerService(mockDatabase, mockCrewAI);
  });

  describe('calculateProfitability', () => {
    it('should calculate customer profitability correctly', async () => {
      // Arrange
      const customerId = 'test-customer-id';
      const mockCustomerData = {
        id: customerId,
        totalRevenue: 10000,
        totalCosts: 7000,
        acquisitionCost: 500
      };

      mockDatabase.getCustomerFinancials.mockResolvedValue(mockCustomerData);
      mockCrewAI.analyzeCustomerProfitability.mockResolvedValue({
        profitabilityScore: 85,
        lifetimeValue: 25000
      });

      // Act
      const result = await customerService.calculateProfitability(customerId);

      // Assert
      expect(result.profitabilityScore).toBe(85);
      expect(result.lifetimeValue).toBe(25000);
      expect(mockDatabase.getCustomerFinancials).toHaveBeenCalledWith(customerId);
      expect(mockCrewAI.analyzeCustomerProfitability).toHaveBeenCalledWith(mockCustomerData);
    });

    it('should handle errors gracefully', async () => {
      // Arrange
      const customerId = 'invalid-id';
      mockDatabase.getCustomerFinancials.mockRejectedValue(new Error('Customer not found'));

      // Act & Assert
      await expect(customerService.calculateProfitability(customerId))
        .rejects.toThrow('Customer not found');
    });
  });
});
```

#### Frontend Unit Tests (Jest + React Testing Library)
```typescript
// packages/web/src/components/BusinessDashboard.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { BusinessDashboard } from './BusinessDashboard';
import { BusinessMetricsService } from '../services/BusinessMetricsService';

jest.mock('../services/BusinessMetricsService');

describe('BusinessDashboard', () => {
  let mockBusinessMetricsService: jest.Mocked<BusinessMetricsService>;

  beforeEach(() => {
    mockBusinessMetricsService = new BusinessMetricsService() as jest.Mocked<BusinessMetricsService>;
  });

  it('should display business metrics correctly', async () => {
    // Arrange
    const mockMetrics = {
      totalRevenue: 45000,
      profitMargin: 0.23,
      jobsCompleted: 127,
      efficiency: 0.87
    };

    mockBusinessMetricsService.getMetrics.mockResolvedValue(mockMetrics);

    // Act
    render(<BusinessDashboard />);

    // Assert
    await waitFor(() => {
      expect(screen.getByText('Total Revenue: $45K')).toBeInTheDocument();
      expect(screen.getByText('Profit Margin: 23%')).toBeInTheDocument();
      expect(screen.getByText('Jobs Complete: 127')).toBeInTheDocument();
      expect(screen.getByText('Efficiency: 87%')).toBeInTheDocument();
    });
  });

  it('should handle loading state', () => {
    // Arrange
    mockBusinessMetricsService.getMetrics.mockImplementation(() => new Promise(() => {}));

    // Act
    render(<BusinessDashboard />);

    // Assert
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });
});
```

### 2.2 Integration Testing

#### API Integration Tests
```typescript
// packages/api/tests/integration/workorders.test.ts
import request from 'supertest';
import { app } from '../../src/app';
import { DatabaseService } from '../../src/services/DatabaseService';
import { testDatabase } from '../utils/testDatabase';

describe('Work Orders API Integration', () => {
  let database: DatabaseService;

  beforeAll(async () => {
    database = new DatabaseService(testDatabase);
    await database.migrate();
  });

  afterAll(async () => {
    await database.close();
  });

  beforeEach(async () => {
    await database.seed();
  });

  afterEach(async () => {
    await database.clean();
  });

  describe('POST /api/v1/work-orders', () => {
    it('should create a new work order', async () => {
      // Arrange
      const workOrderData = {
        customerId: 'test-customer-id',
        serviceType: 'plumbing',
        description: 'Fix leaky pipe',
        priorityLevel: 'high'
      };

      // Act
      const response = await request(app)
        .post('/api/v1/work-orders')
        .send(workOrderData)
        .expect(201);

      // Assert
      expect(response.body.id).toBeDefined();
      expect(response.body.workOrderNumber).toMatch(/WO-\d{6}/);
      expect(response.body.status).toBe('draft');
      expect(response.body.serviceType).toBe('plumbing');
    });

    it('should trigger CrewAI analysis', async () => {
      // Arrange
      const workOrderData = {
        customerId: 'test-customer-id',
        serviceType: 'plumbing',
        description: 'Emergency pipe repair',
        priorityLevel: 'urgent'
      };

      // Act
      const response = await request(app)
        .post('/api/v1/work-orders')
        .send(workOrderData)
        .expect(201);

      // Assert
      expect(response.body.estimatedCost).toBeDefined();
      expect(response.body.recommendedEmployee).toBeDefined();
      expect(response.body.estimatedDuration).toBeDefined();
    });
  });
});
```

#### N8N Workflow Integration Tests
```typescript
// packages/api/tests/integration/n8n-workflows.test.ts
import { N8NService } from '../../src/services/N8NService';
import { WorkOrder } from '../../src/models/WorkOrder';

describe('N8N Workflow Integration', () => {
  let n8nService: N8NService;

  beforeAll(() => {
    n8nService = new N8NService({
      apiUrl: process.env.N8N_TEST_API_URL,
      apiKey: process.env.N8N_TEST_API_KEY
    });
  });

  describe('Lead to Work Order Workflow', () => {
    it('should process lead and create work order', async () => {
      // Arrange
      const leadData = {
        customerName: 'John Doe',
        phone: '555-1234',
        email: 'john@example.com',
        serviceType: 'plumbing',
        description: 'Clogged drain',
        urgency: 'medium'
      };

      // Act
      const result = await n8nService.triggerWorkflow('Lead_to_WorkOrder_Automation', leadData);

      // Assert
      expect(result.success).toBe(true);
      expect(result.workOrderId).toBeDefined();
      expect(result.customerCreated).toBe(true);
    });
  });

  describe('Work Order Completion Workflow', () => {
    it('should process completed work order and generate invoice', async () => {
      // Arrange
      const workOrder = new WorkOrder({
        id: 'test-wo-id',
        status: 'completed',
        finalPrice: 350.00,
        customerId: 'test-customer-id'
      });

      // Act
      const result = await n8nService.triggerWorkflow('WorkOrder_Completion_Automation', workOrder);

      // Assert
      expect(result.success).toBe(true);
      expect(result.invoiceGenerated).toBe(true);
      expect(result.followUpScheduled).toBe(true);
    });
  });
});
```

### 2.3 End-to-End Testing

#### E2E Test Configuration (Playwright)
```typescript
// packages/web/tests/e2e/playwright.config.ts
import { PlaywrightTestConfig } from '@playwright/test';

const config: PlaywrightTestConfig = {
  testDir: './tests',
  timeout: 30 * 1000,
  expect: {
    timeout: 5000,
  },
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],
  webServer: {
    command: 'npm run start:test',
    port: 3000,
  },
};

export default config;
```

#### E2E Test Scenarios
```typescript
// packages/web/tests/e2e/business-operations.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Business Operations Flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
    await page.fill('[data-testid="email"]', 'admin@test.com');
    await page.fill('[data-testid="password"]', 'password');
    await page.click('[data-testid="login-button"]');
    await page.waitForURL('/dashboard');
  });

  test('should complete full work order lifecycle', async ({ page }) => {
    // Create new work order
    await page.click('[data-testid="new-work-order-button"]');
    await page.fill('[data-testid="customer-name"]', 'Test Customer');
    await page.fill('[data-testid="service-type"]', 'plumbing');
    await page.fill('[data-testid="description"]', 'Fix leaky faucet');
    await page.selectOption('[data-testid="priority"]', 'medium');
    await page.click('[data-testid="create-work-order"]');

    // Verify work order created
    await expect(page.locator('[data-testid="work-order-number"]')).toContainText('WO-');
    
    // Schedule work order
    await page.click('[data-testid="schedule-button"]');
    await page.fill('[data-testid="schedule-date"]', '2024-12-01');
    await page.fill('[data-testid="schedule-time"]', '10:00');
    await page.selectOption('[data-testid="assign-employee"]', 'Mike Johnson');
    await page.click('[data-testid="confirm-schedule"]');

    // Verify scheduling
    await expect(page.locator('[data-testid="work-order-status"]')).toContainText('Scheduled');
    
    // Complete work order
    await page.click('[data-testid="complete-job-button"]');
    await page.fill('[data-testid="actual-cost"]', '250.00');
    await page.fill('[data-testid="completion-notes"]', 'Job completed successfully');
    await page.click('[data-testid="upload-photos"]');
    await page.setInputFiles('[data-testid="photo-upload"]', 'test-files/after-photo.jpg');
    await page.click('[data-testid="mark-complete"]');

    // Verify completion and invoice generation
    await expect(page.locator('[data-testid="work-order-status"]')).toContainText('Completed');
    await expect(page.locator('[data-testid="invoice-generated"]')).toBeVisible();
    
    // Verify business metrics update
    await page.goto('/dashboard');
    await expect(page.locator('[data-testid="jobs-completed"]')).toContainText('1');
    await expect(page.locator('[data-testid="total-revenue"]')).toContainText('$250');
  });

  test('should display AI insights correctly', async ({ page }) => {
    await page.goto('/business-intelligence');
    
    // Verify AI insights are displayed
    await expect(page.locator('[data-testid="ai-insights"]')).toBeVisible();
    await expect(page.locator('[data-testid="profitability-analysis"]')).toBeVisible();
    await expect(page.locator('[data-testid="equipment-utilization"]')).toBeVisible();
    await expect(page.locator('[data-testid="employee-performance"]')).toBeVisible();
    
    // Verify AI recommendations
    await expect(page.locator('[data-testid="ai-recommendations"]')).toBeVisible();
    await expect(page.locator('[data-testid="recommendation-item"]')).toHaveCount.greaterThan(0);
  });
});
```

## 3. Deployment Architecture

### 3.1 AWS Infrastructure (Terraform)

#### Main Infrastructure Configuration
```hcl
# infrastructure/terraform/main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  backend "s3" {
    bucket = "quant-terraform-state"
    key    = "production/terraform.tfstate"
    region = "us-east-1"
  }
}

provider "aws" {
  region = var.aws_region
}

# VPC Configuration
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "quant-vpc"
    Environment = var.environment
  }
}

# Subnets
resource "aws_subnet" "public" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 1}.0/24"
  availability_zone = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "quant-public-subnet-${count.index + 1}"
    Environment = var.environment
  }
}

resource "aws_subnet" "private" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 10}.0/24"
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name = "quant-private-subnet-${count.index + 1}"
    Environment = var.environment
  }
}

# RDS PostgreSQL
resource "aws_db_subnet_group" "main" {
  name       = "quant-db-subnet-group"
  subnet_ids = aws_subnet.private[*].id

  tags = {
    Name = "quant-db-subnet-group"
    Environment = var.environment
  }
}

resource "aws_db_instance" "postgresql" {
  identifier              = "quant-postgresql"
  engine                  = "postgres"
  engine_version          = "15.4"
  instance_class          = var.db_instance_class
  allocated_storage       = 100
  max_allocated_storage   = 1000
  storage_type            = "gp3"
  storage_encrypted       = true
  
  db_name  = "quant_production"
  username = "quant_admin"
  password = var.db_password
  
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"
  
  skip_final_snapshot = false
  final_snapshot_identifier = "quant-postgresql-final-snapshot"
  
  tags = {
    Name = "quant-postgresql"
    Environment = var.environment
  }
}

# ElastiCache Redis
resource "aws_elasticache_subnet_group" "main" {
  name       = "quant-redis-subnet-group"
  subnet_ids = aws_subnet.private[*].id
}

resource "aws_elasticache_replication_group" "redis" {
  replication_group_id       = "quant-redis"
  description                = "Redis cluster for Quant application"
  
  port                       = 6379
  parameter_group_name       = "default.redis7"
  node_type                  = var.redis_node_type
  num_cache_clusters         = 2
  
  subnet_group_name          = aws_elasticache_subnet_group.main.name
  security_group_ids         = [aws_security_group.redis.id]
  
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  
  tags = {
    Name = "quant-redis"
    Environment = var.environment
  }
}

# EKS Cluster
resource "aws_eks_cluster" "main" {
  name     = "quant-cluster"
  role_arn = aws_iam_role.eks_cluster.arn
  version  = "1.28"

  vpc_config {
    subnet_ids = aws_subnet.private[*].id
    security_group_ids = [aws_security_group.eks_cluster.id]
  }

  depends_on = [
    aws_iam_role_policy_attachment.eks_cluster_policy,
    aws_iam_role_policy_attachment.eks_service_policy,
  ]

  tags = {
    Name = "quant-eks-cluster"
    Environment = var.environment
  }
}

# EKS Node Group
resource "aws_eks_node_group" "main" {
  cluster_name    = aws_eks_cluster.main.name
  node_group_name = "quant-node-group"
  node_role_arn   = aws_iam_role.eks_node_group.arn
  subnet_ids      = aws_subnet.private[*].id

  scaling_config {
    desired_size = var.node_group_desired_size
    max_size     = var.node_group_max_size
    min_size     = var.node_group_min_size
  }

  instance_types = [var.node_instance_type]
  capacity_type  = "ON_DEMAND"

  depends_on = [
    aws_iam_role_policy_attachment.eks_worker_node_policy,
    aws_iam_role_policy_attachment.eks_cni_policy,
    aws_iam_role_policy_attachment.eks_container_registry_policy,
  ]

  tags = {
    Name = "quant-node-group"
    Environment = var.environment
  }
}
```

### 3.2 Kubernetes Deployment

#### Namespace and ConfigMap
```yaml
# infrastructure/kubernetes/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: quant-production
  labels:
    name: quant-production
    environment: production

---
# infrastructure/kubernetes/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: quant-config
  namespace: quant-production
data:
  NODE_ENV: "production"
  PORT: "3000"
  DATABASE_URL: "postgresql://quant_admin:${DB_PASSWORD}@quant-postgresql.cluster-xyz.us-east-1.rds.amazonaws.com:5432/quant_production"
  REDIS_URL: "redis://quant-redis.cluster-xyz.cache.amazonaws.com:6379"
  N8N_API_URL: "http://n8n-service:5678"
  CREWAI_API_URL: "http://crewai-service:8000"
```

#### API Deployment
```yaml
# infrastructure/kubernetes/api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quant-api
  namespace: quant-production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: quant-api
  template:
    metadata:
      labels:
        app: quant-api
    spec:
      containers:
      - name: api
        image: quant/api:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: quant-secrets
              key: database-url
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: quant-secrets
              key: jwt-secret
        envFrom:
        - configMapRef:
            name: quant-config
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: quant-api-service
  namespace: quant-production
spec:
  selector:
    app: quant-api
  ports:
  - port: 80
    targetPort: 3000
  type: ClusterIP
```

#### Web Application Deployment
```yaml
# infrastructure/kubernetes/web-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quant-web
  namespace: quant-production
spec:
  replicas: 2
  selector:
    matchLabels:
      app: quant-web
  template:
    metadata:
      labels:
        app: quant-web
    spec:
      containers:
      - name: web
        image: quant/web:latest
        ports:
        - containerPort: 80
        env:
        - name: REACT_APP_API_URL
          value: "https://api.quant.com"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"

---
apiVersion: v1
kind: Service
metadata:
  name: quant-web-service
  namespace: quant-production
spec:
  selector:
    app: quant-web
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

#### CrewAI Service Deployment
```yaml
# infrastructure/kubernetes/crewai-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quant-crewai
  namespace: quant-production
spec:
  replicas: 2
  selector:
    matchLabels:
      app: quant-crewai
  template:
    metadata:
      labels:
        app: quant-crewai
    spec:
      containers:
      - name: crewai
        image: quant/crewai:latest
        ports:
        - containerPort: 8000
        env:
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: quant-secrets
              key: openai-api-key
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: quant-secrets
              key: database-url
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 60
          periodSeconds: 30

---
apiVersion: v1
kind: Service
metadata:
  name: crewai-service
  namespace: quant-production
spec:
  selector:
    app: quant-crewai
  ports:
  - port: 8000
    targetPort: 8000
  type: ClusterIP
```

#### N8N Deployment
```yaml
# infrastructure/kubernetes/n8n-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quant-n8n
  namespace: quant-production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: quant-n8n
  template:
    metadata:
      labels:
        app: quant-n8n
    spec:
      containers:
      - name: n8n
        image: n8nio/n8n:latest
        ports:
        - containerPort: 5678
        env:
        - name: N8N_HOST
          value: "n8n.quant.com"
        - name: N8N_PORT
          value: "5678"
        - name: N8N_PROTOCOL
          value: "https"
        - name: WEBHOOK_URL
          value: "https://n8n.quant.com/"
        - name: GENERIC_TIMEZONE
          value: "UTC"
        - name: DB_TYPE
          value: "postgresdb"
        - name: DB_POSTGRESDB_HOST
          value: "quant-postgresql.cluster-xyz.us-east-1.rds.amazonaws.com"
        - name: DB_POSTGRESDB_DATABASE
          value: "n8n_production"
        - name: DB_POSTGRESDB_USER
          valueFrom:
            secretKeyRef:
              name: quant-secrets
              key: db-username
        - name: DB_POSTGRESDB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: quant-secrets
              key: db-password
        volumeMounts:
        - name: n8n-data
          mountPath: /home/node/.n8n
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
      volumes:
      - name: n8n-data
        persistentVolumeClaim:
          claimName: n8n-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: n8n-service
  namespace: quant-production
spec:
  selector:
    app: quant-n8n
  ports:
  - port: 5678
    targetPort: 5678
  type: ClusterIP
```

## 4. CI/CD Pipeline

### 4.1 GitHub Actions Workflow

#### Main CI/CD Pipeline
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: quant-platform
  EKS_CLUSTER_NAME: quant-cluster

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: quant_test
          POSTGRES_USER: test_user
          POSTGRES_PASSWORD: test_password
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run linting
      run: npm run lint

    - name: Run type checking
      run: npm run type-check

    - name: Run unit tests
      run: npm run test:unit
      env:
        DATABASE_URL: postgresql://test_user:test_password@localhost:5432/quant_test
        REDIS_URL: redis://localhost:6379

    - name: Run integration tests
      run: npm run test:integration
      env:
        DATABASE_URL: postgresql://test_user:test_password@localhost:5432/quant_test
        REDIS_URL: redis://localhost:6379

    - name: Run E2E tests
      run: npm run test:e2e
      env:
        DATABASE_URL: postgresql://test_user:test_password@localhost:5432/quant_test
        REDIS_URL: redis://localhost:6379

    - name: Generate coverage report
      run: npm run test:coverage

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2

    - name: Build and push API image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:api-$IMAGE_TAG -f packages/api/Dockerfile .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:api-$IMAGE_TAG

    - name: Build and push Web image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:web-$IMAGE_TAG -f packages/web/Dockerfile .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:web-$IMAGE_TAG

    - name: Build and push CrewAI image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:crewai-$IMAGE_TAG -f crewai/Dockerfile .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:crewai-$IMAGE_TAG

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --region ${{ env.AWS_REGION }} --name ${{ env.EKS_CLUSTER_NAME }}

    - name: Deploy to Kubernetes
      env:
        ECR_REGISTRY: ${{ secrets.ECR_REGISTRY }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        # Update deployment images
        kubectl set image deployment/quant-api api=$ECR_REGISTRY/$ECR_REPOSITORY:api-$IMAGE_TAG -n quant-production
        kubectl set image deployment/quant-web web=$ECR_REGISTRY/$ECR_REPOSITORY:web-$IMAGE_TAG -n quant-production
        kubectl set image deployment/quant-crewai crewai=$ECR_REGISTRY/$ECR_REPOSITORY:crewai-$IMAGE_TAG -n quant-production
        
        # Wait for rollout
        kubectl rollout status deployment/quant-api -n quant-production
        kubectl rollout status deployment/quant-web -n quant-production
        kubectl rollout status deployment/quant-crewai -n quant-production

    - name: Run smoke tests
      run: |
        # Wait for services to be ready
        kubectl wait --for=condition=ready pod -l app=quant-api -n quant-production --timeout=300s
        
        # Run smoke tests
        npm run test:smoke

    - name: Notify deployment success
      if: success()
      uses: 8398a7/action-slack@v3
      with:
        status: success
        text: 'Deployment to production successful! 🚀'
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

    - name: Notify deployment failure
      if: failure()
      uses: 8398a7/action-slack@v3
      with:
        status: failure
        text: 'Deployment to production failed! 🚨'
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### 4.2 Docker Configurations

#### API Dockerfile
```dockerfile
# packages/api/Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
COPY packages/api/package*.json ./packages/api/
COPY packages/shared/package*.json ./packages/shared/

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY packages/api ./packages/api
COPY packages/shared ./packages/shared

# Build application
RUN npm run build

# Production stage
FROM node:18-alpine AS production

WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

# Copy built application
COPY --from=builder --chown=nodejs:nodejs /app/packages/api/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/packages/api/package.json ./package.json

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD node dist/healthcheck.js

# Start application
CMD ["node", "dist/server.js"]
```

#### Web Dockerfile
```dockerfile
# packages/web/Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
COPY packages/web/package*.json ./packages/web/
COPY packages/shared/package*.json ./packages/shared/

# Install dependencies
RUN npm ci

# Copy source code
COPY packages/web ./packages/web
COPY packages/shared ./packages/shared

# Build application
RUN npm run build

# Production stage
FROM nginx:alpine AS production

# Copy built application
COPY --from=builder /app/packages/web/build /usr/share/nginx/html

# Copy nginx configuration
COPY packages/web/nginx.conf /etc/nginx/conf.d/default.conf

# Expose port
EXPOSE 80

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:80/ || exit 1

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

## 5. Monitoring and Observability

### 5.1 Prometheus Configuration

#### Prometheus Setup
```yaml
# infrastructure/kubernetes/monitoring/prometheus-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s

    rule_files:
      - "quant_rules.yml"

    scrape_configs:
      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
        - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https

      - job_name: 'quant-api'
        kubernetes_sd_configs:
        - role: endpoints
        relabel_configs:
        - source_labels: [__meta_kubernetes_service_name]
          action: keep
          regex: quant-api-service
        - source_labels: [__meta_kubernetes_endpoint_port_name]
          action: keep
          regex: metrics

      - job_name: 'quant-crewai'
        kubernetes_sd_configs:
        - role: endpoints
        relabel_configs:
        - source_labels: [__meta_kubernetes_service_name]
          action: keep
          regex: crewai-service
        - source_labels: [__meta_kubernetes_endpoint_port_name]
          action: keep
          regex: metrics

      - job_name: 'n8n'
        kubernetes_sd_configs:
        - role: endpoints
        relabel_configs:
        - source_labels: [__meta_kubernetes_service_name]
          action: keep
          regex: n8n-service
        - source_labels: [__meta_kubernetes_endpoint_port_name]
          action: keep
          regex: metrics

  quant_rules.yml: |
    groups:
    - name: quant.rules
      rules:
      - alert: QuantAPIDown
        expr: up{job="quant-api"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Quant API is down"
          description: "Quant API has been down for more than 1 minute"

      - alert: QuantHighLatency
        expr: histogram_quantile(0.95, http_request_duration_seconds_bucket{job="quant-api"}) > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected in Quant API"
          description: "95th percentile latency is above 500ms"

      - alert: QuantHighErrorRate
        expr: rate(http_requests_total{job="quant-api", status=~"5.."}[5m]) > 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate in Quant API"
          description: "Error rate is above 10%"

      - alert: CrewAIDown
        expr: up{job="quant-crewai"} == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "CrewAI service is down"
          description: "CrewAI service has been down for more than 2 minutes"

      - alert: DatabaseConnectionHigh
        expr: postgres_stat_activity_count{job="postgres"} > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High database connection count"
          description: "Database connection count is above 80"
```

### 5.2 Grafana Dashboards

#### Business Operations Dashboard
```json
{
  "dashboard": {
    "id": null,
    "title": "Quant Business Operations",
    "tags": ["quant", "business"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "Total Revenue",
        "type": "stat",
        "targets": [
          {
            "expr": "sum(quant_revenue_total)",
            "legendFormat": "Total Revenue"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "thresholds"
            },
            "mappings": [],
            "thresholds": {
              "steps": [
                {
                  "color": "green",
                  "value": null
                }
              ]
            },
            "unit": "currencyUSD"
          }
        }
      },
      {
        "id": 2,
        "title": "Active Work Orders",
        "type": "graph",
        "targets": [
          {
            "expr": "quant_work_orders_active",
            "legendFormat": "Active Work Orders"
          }
        ]
      },
      {
        "id": 3,
        "title": "Equipment Utilization",
        "type": "bargauge",
        "targets": [
          {
            "expr": "quant_equipment_utilization_percentage",
            "legendFormat": "{{equipment_name}}"
          }
        ]
      },
      {
        "id": 4,
        "title": "Employee Performance",
        "type": "table",
        "targets": [
          {
            "expr": "quant_employee_performance_score",
            "legendFormat": "{{employee_name}}"
          }
        ]
      }
    ],
    "time": {
      "from": "now-1h",
      "to": "now"
    },
    "refresh": "30s"
  }
}
```

### 5.3 Logging Configuration

#### Fluentd Configuration
```yaml
# infrastructure/kubernetes/logging/fluentd-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
  namespace: logging
data:
  fluent.conf: |
    <source>
      @type tail
      path /var/log/containers/quant-api*.log
      pos_file /var/log/fluentd-quant-api.log.pos
      tag quant.api
      format json
      time_key time
      time_format %Y-%m-%dT%H:%M:%S.%NZ
    </source>

    <source>
      @type tail
      path /var/log/containers/quant-crewai*.log
      pos_file /var/log/fluentd-quant-crewai.log.pos
      tag quant.crewai
      format json
      time_key time
      time_format %Y-%m-%dT%H:%M:%S.%NZ
    </source>

    <filter quant.**>
      @type record_transformer
      <record>
        hostname "#{Socket.gethostname}"
        environment "#{ENV['ENVIRONMENT']}"
        application "quant"
      </record>
    </filter>

    <match quant.api>
      @type elasticsearch
      host elasticsearch.logging.svc.cluster.local
      port 9200
      index_name quant-api-logs
      type_name _doc
      logstash_format true
      logstash_prefix quant-api
      logstash_dateformat %Y.%m.%d
      include_tag_key true
      tag_key @log_name
      flush_interval 10s
    </match>

    <match quant.crewai>
      @type elasticsearch
      host elasticsearch.logging.svc.cluster.local
      port 9200
      index_name quant-crewai-logs
      type_name _doc
      logstash_format true
      logstash_prefix quant-crewai
      logstash_dateformat %Y.%m.%d
      include_tag_key true
      tag_key @log_name
      flush_interval 10s
    </match>
```

## 6. Security Implementation

### 6.1 Security Policies

#### Network Security Policy
```yaml
# infrastructure/kubernetes/security/network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: quant-network-policy
  namespace: quant-production
spec:
  podSelector:
    matchLabels:
      app: quant-api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    - podSelector:
        matchLabels:
          app: quant-web
    ports:
    - protocol: TCP
      port: 3000
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: quant-crewai
    ports:
    - protocol: TCP
      port: 8000
  - to:
    - podSelector:
        matchLabels:
          app: quant-n8n
    ports:
    - protocol: TCP
      port: 5678
  - to: []
    ports:
    - protocol: TCP
      port: 5432  # PostgreSQL
    - protocol: TCP
      port: 6379  # Redis
```

#### Pod Security Policy
```yaml
# infrastructure/kubernetes/security/pod-security-policy.yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: quant-psp
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
  readOnlyRootFilesystem: true
```

### 6.2 Secrets Management

#### Secrets Configuration
```yaml
# infrastructure/kubernetes/security/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: quant-secrets
  namespace: quant-production
type: Opaque
data:
  database-url: <base64-encoded-database-url>
  jwt-secret: <base64-encoded-jwt-secret>
  openai-api-key: <base64-encoded-openai-key>
  twilio-auth-token: <base64-encoded-twilio-token>
  stripe-secret-key: <base64-encoded-stripe-key>
  aws-access-key: <base64-encoded-aws-access-key>
  aws-secret-key: <base64-encoded-aws-secret-key>
```

#### External Secrets Operator
```yaml
# infrastructure/kubernetes/security/external-secrets.yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets-manager
  namespace: quant-production
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        secretRef:
          accessKeyId:
            name: aws-credentials
            key: access-key-id
          secretAccessKey:
            name: aws-credentials
            key: secret-access-key

---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: quant-external-secrets
  namespace: quant-production
spec:
  refreshInterval: 15m
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: quant-secrets
    creationPolicy: Owner
  data:
  - secretKey: database-url
    remoteRef:
      key: quant/production/database-url
  - secretKey: jwt-secret
    remoteRef:
      key: quant/production/jwt-secret
  - secretKey: openai-api-key
    remoteRef:
      key: quant/production/openai-api-key
```

## 7. Performance Optimization

### 7.1 Application Performance

#### API Performance Optimization
```typescript
// packages/api/src/middleware/performance.ts
import { Request, Response, NextFunction } from 'express';
import { performance } from 'perf_hooks';
import { Logger } from '../utils/logger';

export class PerformanceMiddleware {
  static trackPerformance() {
    return (req: Request, res: Response, next: NextFunction) => {
      const startTime = performance.now();
      
      res.on('finish', () => {
        const duration = performance.now() - startTime;
        
        Logger.info('API Performance', {
          method: req.method,
          url: req.url,
          statusCode: res.statusCode,
          duration: `${duration.toFixed(2)}ms`,
          timestamp: new Date().toISOString()
        });
        
        // Record metrics for Prometheus
        apiRequestDuration.labels(req.method, req.route?.path || req.url, res.statusCode.toString())
          .observe(duration / 1000);
      });
      
      next();
    };
  }
}
```

#### Database Query Optimization
```typescript
// packages/api/src/services/DatabaseOptimization.ts
export class DatabaseOptimization {
  static async optimizeQueries() {
    // Create indexes for frequently used queries
    await this.createIndexes();
    
    // Analyze query performance
    await this.analyzeQueries();
    
    // Optimize slow queries
    await this.optimizeSlowQueries();
  }
  
  private static async createIndexes() {
    const indexes = [
      'CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_work_orders_status_date ON work_orders(status, scheduled_date)',
      'CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_customers_profitability ON customers(profitability_score DESC)',
      'CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_equipment_utilization ON equipment(utilization_rate DESC)',
      'CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_employees_performance ON employees(performance_score DESC)'
    ];
    
    for (const index of indexes) {
      await this.executeQuery(index);
    }
  }
  
  private static async analyzeQueries() {
    const slowQueries = await this.executeQuery(`
      SELECT query, mean_time, calls, total_time
      FROM pg_stat_statements
      WHERE mean_time > 100
      ORDER BY mean_time DESC
      LIMIT 10
    `);
    
    Logger.info('Slow queries detected', { slowQueries });
  }
}
```

### 7.2 Caching Strategy

#### Multi-Level Caching
```typescript
// packages/api/src/services/CacheService.ts
export class CacheService {
  private redis: Redis;
  private memoryCache: Map<string, any>;
  
  constructor() {
    this.redis = new Redis(process.env.REDIS_URL);
    this.memoryCache = new Map();
  }
  
  async get(key: string): Promise<any> {
    // Level 1: Memory cache
    if (this.memoryCache.has(key)) {
      return this.memoryCache.get(key);
    }
    
    // Level 2: Redis cache
    const redisValue = await this.redis.get(key);
    if (redisValue) {
      const parsed = JSON.parse(redisValue);
      this.memoryCache.set(key, parsed);
      return parsed;
    }
    
    return null;
  }
  
  async set(key: string, value: any, ttl: number = 3600): Promise<void> {
    // Set in memory cache
    this.memoryCache.set(key, value);
    
    // Set in Redis with TTL
    await this.redis.setex(key, ttl, JSON.stringify(value));
    
    // Clean up memory cache periodically
    if (this.memoryCache.size > 1000) {
      this.cleanupMemoryCache();
    }
  }
  
  private cleanupMemoryCache(): void {
    const entries = Array.from(this.memoryCache.entries());
    const toDelete = entries.slice(0, 500);
    
    for (const [key] of toDelete) {
      this.memoryCache.delete(key);
    }
  }
}
```

## 8. Go-Live Strategy

### 8.1 Pre-Launch Checklist

#### Technical Readiness
```bash
#!/bin/bash
# infrastructure/scripts/pre-launch-checklist.sh

echo "=== Quant Platform Pre-Launch Checklist ==="

# 1. Infrastructure Health Check
echo "1. Checking infrastructure health..."
kubectl get nodes
kubectl get pods -n quant-production
kubectl get services -n quant-production

# 2. Database Connectivity
echo "2. Testing database connectivity..."
pg_isready -h $DB_HOST -p $DB_PORT -U $DB_USER

# 3. Redis Connectivity
echo "3. Testing Redis connectivity..."
redis-cli -h $REDIS_HOST -p $REDIS_PORT ping

# 4. API Health Check
echo "4. Testing API health..."
curl -f http://api.quant.com/health || exit 1

# 5. CrewAI Service Check
echo "5. Testing CrewAI service..."
curl -f http://crewai-service:8000/health || exit 1

# 6. N8N Service Check
echo "6. Testing N8N service..."
curl -f http://n8n-service:5678/health || exit 1

# 7. SSL Certificate Check
echo "7. Checking SSL certificates..."
curl -I https://app.quant.com | grep -i "200 OK"

# 8. Load Testing
echo "8. Running load tests..."
npm run test:load

# 9. Security Scan
echo "9. Running security scan..."
npm run security:scan

# 10. Backup Verification
echo "10. Verifying backup system..."
./scripts/verify-backups.sh

echo "=== Pre-Launch Checklist Complete ==="
```

### 8.2 Launch Sequence

#### Blue-Green Deployment
```bash
#!/bin/bash
# infrastructure/scripts/blue-green-deployment.sh

ENVIRONMENT=$1
VERSION=$2

if [ -z "$ENVIRONMENT" ] || [ -z "$VERSION" ]; then
  echo "Usage: $0 <environment> <version>"
  exit 1
fi

echo "=== Blue-Green Deployment Started ==="
echo "Environment: $ENVIRONMENT"
echo "Version: $VERSION"

# 1. Deploy to green environment
echo "1. Deploying to green environment..."
kubectl apply -f infrastructure/kubernetes/green/ -n quant-$ENVIRONMENT-green

# 2. Wait for green deployment to be ready
echo "2. Waiting for green deployment..."
kubectl wait --for=condition=available --timeout=300s deployment/quant-api-green -n quant-$ENVIRONMENT-green

# 3. Run smoke tests on green
echo "3. Running smoke tests on green environment..."
npm run test:smoke -- --environment=green

# 4. Switch traffic to green
echo "4. Switching traffic to green..."
kubectl patch service quant-api-service -p '{"spec":{"selector":{"version":"green"}}}' -n quant-$ENVIRONMENT

# 5. Monitor for 5 minutes
echo "5. Monitoring green environment..."
sleep 300

# 6. Check metrics
echo "6. Checking metrics..."
ERROR_RATE=$(curl -s "http://prometheus:9090/api/v1/query?query=rate(http_requests_total{status=~\"5..\"}[5m])" | jq -r '.data.result[0].value[1]')

if (( $(echo "$ERROR_RATE > 0.05" | bc -l) )); then
  echo "ERROR: High error rate detected. Rolling back..."
  kubectl patch service quant-api-service -p '{"spec":{"selector":{"version":"blue"}}}' -n quant-$ENVIRONMENT
  exit 1
fi

# 7. Cleanup blue environment
echo "7. Cleaning up blue environment..."
kubectl delete -f infrastructure/kubernetes/blue/ -n quant-$ENVIRONMENT-blue

echo "=== Blue-Green Deployment Complete ==="
```

### 8.3 Post-Launch Monitoring

#### Launch Monitoring Dashboard
```bash
#!/bin/bash
# infrastructure/scripts/launch-monitoring.sh

echo "=== Launch Monitoring Started ==="

# Monitor key metrics for first 24 hours
DURATION=86400  # 24 hours in seconds
INTERVAL=60     # Check every minute

for ((i=0; i<$DURATION; i+=$INTERVAL)); do
  echo "=== Monitoring Check $(date) ==="
  
  # API Health
  API_STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://api.quant.com/health)
  echo "API Status: $API_STATUS"
  
  # Error Rate
  ERROR_RATE=$(curl -s "http://prometheus:9090/api/v1/query?query=rate(http_requests_total{status=~\"5..\"}[5m])" | jq -r '.data.result[0].value[1]')
  echo "Error Rate: $ERROR_RATE"
  
  # Response Time
  RESPONSE_TIME=$(curl -s "http://prometheus:9090/api/v1/query?query=histogram_quantile(0.95,http_request_duration_seconds_bucket)" | jq -r '.data.result[0].value[1]')
  echo "95th Percentile Response Time: $RESPONSE_TIME"
  
  # Database Connections
  DB_CONNECTIONS=$(curl -s "http://prometheus:9090/api/v1/query?query=postgres_stat_activity_count" | jq -r '.data.result[0].value[1]')
  echo "Database Connections: $DB_CONNECTIONS"
  
  # Memory Usage
  MEMORY_USAGE=$(kubectl top pods -n quant-production | grep quant-api | awk '{print $3}')
  echo "Memory Usage: $MEMORY_USAGE"
  
  # CPU Usage
  CPU_USAGE=$(kubectl top pods -n quant-production | grep quant-api | awk '{print $2}')
  echo "CPU Usage: $CPU_USAGE"
  
  echo "=========================="
  sleep $INTERVAL
done

echo "=== Launch Monitoring Complete ==="
```

## 9. Documentation Standards

### 9.1 API Documentation

#### OpenAPI Specification
```yaml
# docs/api/openapi.yaml
openapi: 3.0.0
info:
  title: Quant Business Operations API
  description: API for managing field service business operations
  version: 1.0.0
  contact:
    name: Quant Team
    email: api@quant.com
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT

servers:
  - url: https://api.quant.com/v1
    description: Production server
  - url: https://staging-api.quant.com/v1
    description: Staging server

paths:
  /work-orders:
    get:
      summary: Get work orders
      description: Retrieve a list of work orders with optional filtering
      parameters:
        - name: status
          in: query
          description: Filter by work order status
          schema:
            type: string
            enum: [draft, scheduled, in_progress, completed, cancelled]
        - name: customer_id
          in: query
          description: Filter by customer ID
          schema:
            type: string
            format: uuid
        - name: limit
          in: query
          description: Maximum number of results
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/WorkOrder'
                  pagination:
                    $ref: '#/components/schemas/Pagination'
        '400':
          description: Bad request
        '401':
          description: Unauthorized
        '500':
          description: Internal server error

    post:
      summary: Create work order
      description: Create a new work order
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateWorkOrderRequest'
      responses:
        '201':
          description: Work order created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/WorkOrder'
        '400':
          description: Bad request
        '401':
          description: Unauthorized
        '500':
          description: Internal server error

components:
  schemas:
    WorkOrder:
      type: object
      properties:
        id:
          type: string
          format: uuid
        workOrderNumber:
          type: string
          example: "WO-001234"
        customerId:
          type: string
          format: uuid
        serviceType:
          type: string
          example: "plumbing"
        status:
          type: string
          enum: [draft, scheduled, in_progress, completed, cancelled]
        priorityLevel:
          type: string
          enum: [low, medium, high, urgent]
        scheduledDate:
          type: string
          format: date
        scheduledTime:
          type: string
          format: time
        estimatedDurationHours:
          type: number
          format: float
        actualDurationHours:
          type: number
          format: float
        assignedEmployeeId:
          type: string
          format: uuid
        loadoutId:
          type: string
          format: uuid
        jobDescription:
          type: string
        specialInstructions:
          type: string
        estimatedCost:
          type: number
          format: float
        actualCost:
          type: number
          format: float
        quotedPrice:
          type: number
          format: float
        finalPrice:
          type: number
          format: float
        profitMargin:
          type: number
          format: float
        completionPhotos:
          type: object
          properties:
            before:
              type: array
              items:
                type: string
                format: uri
            after:
              type: array
              items:
                type: string
                format: uri
        customerSignature:
          type: string
        customerNotes:
          type: string
        createdAt:
          type: string
          format: date-time
        updatedAt:
          type: string
          format: date-time
      required:
        - id
        - workOrderNumber
        - customerId
        - serviceType
        - status
        - createdAt
        - updatedAt

    CreateWorkOrderRequest:
      type: object
      properties:
        customerId:
          type: string
          format: uuid
        serviceType:
          type: string
        priorityLevel:
          type: string
          enum: [low, medium, high, urgent]
        jobDescription:
          type: string
        specialInstructions:
          type: string
        scheduledDate:
          type: string
          format: date
        scheduledTime:
          type: string
          format: time
        estimatedDurationHours:
          type: number
          format: float
      required:
        - customerId
        - serviceType
        - jobDescription

    Pagination:
      type: object
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        totalPages:
          type: integer
        hasNext:
          type: boolean
        hasPrevious:
          type: boolean

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []
```

### 9.2 Runbooks

#### Incident Response Runbook
```markdown
# Incident Response Runbook

## Severity Levels

### P1 - Critical
- System completely down
- Data loss or corruption
- Security breach
- Response time: 15 minutes

### P2 - High
- Major feature unavailable
- Significant performance degradation
- Response time: 1 hour

### P3 - Medium
- Minor feature issues
- Moderate performance impact
- Response time: 4 hours

### P4 - Low
- Cosmetic issues
- Documentation updates
- Response time: 1 business day

## Common Issues and Solutions

### API Service Down
**Symptoms:**
- Health check endpoints returning 5xx errors
- High error rate in monitoring
- Customer reports of service unavailability

**Investigation Steps:**
1. Check Kubernetes pod status
   ```bash
   kubectl get pods -n quant-production
   kubectl describe pod <pod-name> -n quant-production
   ```

2. Check application logs
   ```bash
   kubectl logs -f deployment/quant-api -n quant-production
   ```

3. Check resource utilization
   ```bash
   kubectl top pods -n quant-production
   ```

**Resolution Steps:**
1. Restart pods if needed
   ```bash
   kubectl rollout restart deployment/quant-api -n quant-production
   ```

2. Scale up if resource constraints
   ```bash
   kubectl scale deployment/quant-api --replicas=5 -n quant-production
   ```

3. Check database connectivity
   ```bash
   kubectl exec -it deployment/quant-api -n quant-production -- pg_isready -h $DB_HOST
   ```

### Database Performance Issues
**Symptoms:**
- Slow query response times
- High database connection count
- Timeout errors

**Investigation Steps:**
1. Check active connections
   ```sql
   SELECT count(*) FROM pg_stat_activity;
   ```

2. Identify slow queries
   ```sql
   SELECT query, mean_time, calls FROM pg_stat_statements ORDER BY mean_time DESC LIMIT 10;
   ```

3. Check database locks
   ```sql
   SELECT * FROM pg_locks WHERE NOT granted;
   ```

**Resolution Steps:**
1. Kill long-running queries if needed
   ```sql
   SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state = 'active' AND query_start < NOW() - INTERVAL '5 minutes';
   ```

2. Optimize queries or add indexes
3. Consider read replicas for read-heavy workloads

### CrewAI Service Issues
**Symptoms:**
- AI recommendations not appearing
- CrewAI service health check failing
- Timeouts in AI agent responses

**Investigation Steps:**
1. Check CrewAI service logs
   ```bash
   kubectl logs -f deployment/quant-crewai -n quant-production
   ```

2. Check OpenAI API connectivity
   ```bash
   curl -H "Authorization: Bearer $OPENAI_API_KEY" https://api.openai.com/v1/models
   ```

3. Check resource usage
   ```bash
   kubectl top pods -l app=quant-crewai -n quant-production
   ```

**Resolution Steps:**
1. Restart CrewAI service
   ```bash
   kubectl rollout restart deployment/quant-crewai -n quant-production
   ```

2. Check API key validity
3. Scale up if needed for high load

## Post-Incident Actions
1. Document the incident
2. Conduct post-mortem meeting
3. Update monitoring and alerting
4. Implement preventive measures
5. Update runbooks based on learnings
```

---

**Implementation Complete**: Pass 3 provides comprehensive implementation guidelines covering development setup, testing strategies, deployment architecture, monitoring, security, and go-live procedures. This creates a complete roadmap for building and deploying the Quant Business Operations Platform.

**Next Steps**: Ready for Pass 4 focusing on advanced features, scaling strategies, and long-term maintenance, or begin implementation using the detailed specifications provided in Passes 1-3. 