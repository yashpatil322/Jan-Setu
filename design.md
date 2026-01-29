# Jan-Setu (The People's Bridge) - System Design Document

## System Architecture

### High-Level Architecture Overview
Jan-Setu follows a serverless microservices architecture on AWS, designed for scalability, reliability, and cost-effectiveness.

```
User App (Flutter) 
    ↓ HTTPS/REST
Amazon API Gateway 
    ↓ Event-driven
Orchestrator Lambda (Router)
    ↓ AI Processing
Amazon Bedrock (Claude 3 Haiku)
    ↓ Parallel Processing
Service Lambdas (Scheme/Grievance/Marketplace)
    ↓ Data Layer
Amazon DynamoDB + Amazon S3
```

### Component Flow Description

**1. Frontend Layer (Flutter Mobile App)**
- Voice capture using device microphone
- Integration with Bhashini API for speech-to-text conversion
- Offline capability for basic functions
- Real-time audio feedback and visual confirmations

**2. API Gateway Layer**
- RESTful API endpoints for all service modules
- Request validation and rate limiting
- CORS configuration for web clients
- API key management and throttling

**3. Orchestration Layer (Orchestrator Lambda)**
- Central routing logic based on request type
- Intent classification using Amazon Bedrock
- Request preprocessing and validation
- Response aggregation and formatting

**4. AI Processing Layer (Amazon Bedrock)**
- Claude 3 Haiku model for natural language understanding
- Intent extraction and entity recognition
- Response generation in multiple languages
- PII detection and redaction pipeline

**5. Service Layer (Specialized Lambdas)**
- **Scheme Service**: Government scheme matching and recommendations
- **Grievance Service**: Legal document generation and tracking
- **Marketplace Service**: Product listing and buyer-seller matching

**6. Workflow Management (AWS Step Functions)**
- Multi-step grievance generation workflow
- Error handling and retry mechanisms
- State management for long-running processes
- Integration with external government APIs

## Database Design (DynamoDB Single Table)

### Single Table Design Pattern
All application data is stored in a single DynamoDB table using composite keys for efficient querying and cost optimization.

### Primary Schema Structure

**Table Name**: `jan-setu-main`

**Partition Key (PK)**: Identifies the entity type and primary identifier
**Sort Key (SK)**: Provides hierarchical organization and enables range queries

### Entity Patterns

#### User Profile Pattern
```
PK: USER#<Phone_Number>
SK: PROFILE
Attributes:
- Name: String
- Language: String (hi|mr|en)
- District: String
- Occupation: String
- CreatedAt: ISO8601 Timestamp
- LastActive: ISO8601 Timestamp
- PreferredLanguage: String
```

#### Grievance Pattern
```
PK: USER#<Phone_Number>
SK: GRIEVANCE#<YYYY-MM-DD>#<UUID>
Attributes:
- Status: String (DRAFT|SUBMITTED|IN_PROGRESS|RESOLVED)
- S3_Link: String (S3 URL for PDF document)
- Summary: String (Brief description)
- IssueType: String (WATER|ELECTRICITY|ROAD|CORRUPTION|OTHER)
- Priority: String (LOW|MEDIUM|HIGH|URGENT)
- CreatedAt: ISO8601 Timestamp
- UpdatedAt: ISO8601 Timestamp
- ReferenceNumber: String (Unique tracking ID)
- Department: String (Target government department)
```

#### Marketplace Listing Pattern
```
PK: USER#<Phone_Number>
SK: MARKET#<YYYY-MM-DD>#<UUID>
Attributes:
- ProductType: String (VEGETABLES|GRAINS|FRUITS|DAIRY)
- ProductName: String
- Quantity: Number
- Unit: String (KG|QUINTAL|LITER)
- PricePerUnit: Number
- TotalPrice: Number
- Location: String (Village/District)
- Status: String (ACTIVE|SOLD|EXPIRED)
- CreatedAt: ISO8601 Timestamp
- ExpiresAt: ISO8601 Timestamp
- ContactMethod: String (WHATSAPP|PHONE)
```

#### Scheme Information Pattern
```
PK: SCHEME#<Scheme_ID>
SK: METADATA
Attributes:
- SchemeName: String
- Department: String
- EligibilityCriteria: List<String>
- Benefits: String
- ApplicationProcess: String
- RequiredDocuments: List<String>
- LastUpdated: ISO8601 Timestamp
- IsActive: Boolean
- TargetAudience: List<String>
```

### Global Secondary Indexes (GSI)

#### GSI-1: Status-Based Queries
```
PK: Status (e.g., "GRIEVANCE_ACTIVE", "MARKET_ACTIVE")
SK: CreatedAt
Purpose: Query all active grievances or marketplace listings
```

#### GSI-2: Location-Based Queries
```
PK: Location (District/State)
SK: CreatedAt
Purpose: Regional analytics and location-specific services
```

#### GSI-3: Product-Based Queries
```
PK: ProductType
SK: CreatedAt
Purpose: Marketplace product category browsing
```

## API Design

### RESTful Endpoints

#### Voice Processing Endpoints
```
POST /api/v1/voice/process
- Processes voice input and returns structured response
- Request: Multipart form with audio file
- Response: JSON with action type and processed data

POST /api/v1/voice/synthesize
- Converts text response to speech
- Request: JSON with text and language preference
- Response: Audio file (MP3/WAV)
```

#### Scheme Navigator Endpoints
```
GET /api/v1/schemes/search
- Query parameters: intent, location, user_profile
- Response: List of matching schemes with relevance scores

GET /api/v1/schemes/{scheme_id}
- Returns detailed scheme information
- Response: Complete scheme details with application process
```

#### Grievance Management Endpoints
```
POST /api/v1/grievances
- Creates new grievance from voice input
- Request: JSON with complaint details
- Response: Grievance ID and processing status

GET /api/v1/grievances/{grievance_id}
- Returns grievance status and document links
- Response: Grievance details with S3 download URL

PUT /api/v1/grievances/{grievance_id}/status
- Updates grievance status (admin only)
- Request: JSON with new status and comments
```

#### Marketplace Endpoints
```
POST /api/v1/marketplace/listings
- Creates new product listing
- Request: JSON with product details
- Response: Listing ID and WhatsApp connection link

GET /api/v1/marketplace/listings
- Query parameters: product_type, location, price_range
- Response: List of active listings with contact information

DELETE /api/v1/marketplace/listings/{listing_id}
- Marks listing as sold/inactive
- Response: Confirmation message
```

## Security Architecture

### Authentication and Authorization
- **JWT-based Authentication**: Secure token-based user sessions
- **Phone Number Verification**: OTP-based user registration
- **Role-Based Access Control**: Different permissions for users, VLEs, and administrators

### Data Protection
- **PII Redaction Pipeline**: Automated detection and masking of sensitive information
- **Encryption at Rest**: All DynamoDB and S3 data encrypted with AWS KMS
- **Encryption in Transit**: TLS 1.3 for all API communications
- **Data Anonymization**: User data anonymized for analytics and ML training

### Compliance Framework
- **DPDP Act 2023 Compliance**: 
  - Explicit consent for data processing
  - Right to data portability and deletion
  - Data breach notification procedures
  - Regular compliance audits

## Deployment Architecture

### AWS Serverless Stack
```yaml
Infrastructure Components:
- Amazon API Gateway: API management and throttling
- AWS Lambda: Serverless compute for all business logic
- Amazon DynamoDB: NoSQL database with on-demand scaling
- Amazon S3: Document storage with lifecycle policies
- Amazon Bedrock: AI/ML services for language processing
- AWS Step Functions: Workflow orchestration
- Amazon CloudWatch: Monitoring and logging
- AWS CloudFormation: Infrastructure as Code
```

### Multi-Region Deployment
- **Primary Region**: ap-south-1 (Mumbai)
- **Secondary Region**: ap-south-2 (Hyderabad)
- **Data Replication**: Cross-region backup for disaster recovery
- **CDN Distribution**: CloudFront for static assets and API caching

### Monitoring and Observability
- **Application Metrics**: Custom CloudWatch metrics for business KPIs
- **Performance Monitoring**: X-Ray tracing for request flow analysis
- **Error Tracking**: Centralized error logging with alerting
- **Health Checks**: Automated health monitoring with failover

## Integration Architecture

### External Service Integrations
- **Bhashini API**: Government speech-to-text service
- **DigiLocker**: Document verification and storage
- **Aadhaar APIs**: Identity verification (with consent)
- **Government Portals**: Scheme data synchronization
- **WhatsApp Business API**: Marketplace communication

### Data Flow Patterns
1. **Voice Input Processing**: Audio → Bhashini → Text → Bedrock → Intent
2. **Grievance Generation**: Intent → Template → PDF → S3 → Tracking
3. **Marketplace Matching**: Listing → Vector Search → Recommendations → Contact

## Performance Optimization

### Caching Strategy
- **API Gateway Caching**: 5-minute cache for scheme data
- **DynamoDB DAX**: Microsecond latency for frequently accessed data
- **CloudFront CDN**: Global edge caching for static content
- **Lambda Provisioned Concurrency**: Reduced cold start latency

### Database Optimization
- **Single Table Design**: Minimized cross-table joins and improved performance
- **Composite Keys**: Efficient querying with sort key patterns
- **TTL Implementation**: Automatic cleanup of expired marketplace listings
- **Read/Write Capacity**: On-demand scaling based on traffic patterns

This design ensures Jan-Setu can scale efficiently while maintaining high performance and security standards required for government and citizen services.