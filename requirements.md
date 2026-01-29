# Jan-Setu (The People's Bridge) - Requirements Specification

## Project Overview
Jan-Setu is a voice-first Agentic AI application designed to bridge the digital divide for rural citizens in India. The platform enables users to access government schemes, file legal grievances, and trade agricultural produce through natural voice interactions in local languages.

## User Personas

### Ramesh (The Farmer)
- **Profile**: Rural farmer, 45 years old, low digital literacy
- **Languages**: Primarily speaks Marathi/Hindi, limited English proficiency
- **Technology Access**: Basic smartphone with 4G connectivity
- **Pain Points**: 
  - Cannot write formal English applications for government schemes
  - Struggles with complex government portals and forms
  - Needs assistance in understanding eligibility criteria
- **Goals**: Access subsidies, file complaints about local issues, sell produce directly

### Suresh (The VLE - Village Level Entrepreneur)
- **Profile**: Tech-savvy operator of Common Service Center (CSC)
- **Languages**: Fluent in Hindi, English, and local dialect
- **Technology Access**: Desktop/laptop with stable internet
- **Pain Points**:
  - Needs to process multiple citizen requests efficiently
  - Time-consuming manual form filling and documentation
  - Managing follow-ups on grievance status
- **Goals**: Serve more citizens per day, reduce processing time, maintain service quality

## Functional Requirements (FRs)

### Module A: Scheme Navigator
**Purpose**: Help users discover and understand government schemes through voice queries

**FR-A1: Voice Query Processing**
- **Input**: Natural voice query in Hindi/Marathi (e.g., "Kya koi tractor ke liye subsidy hai?")
- **Process**: 
  - Convert speech to text using Bhashini API
  - Extract intent and entities using Amazon Bedrock (Claude 3 Haiku)
  - Match query against scheme database using vector similarity
- **Output**: Audio response with scheme details, eligibility, and application process

**FR-A2: Scheme Recommendation**
- **Input**: User profile data (location, occupation, demographics)
- **Process**: AI-powered matching of user profile with available schemes
- **Output**: Personalized list of applicable schemes with priority ranking

**FR-A3: Multi-language Support**
- **Input**: Voice queries in Hindi, Marathi, and English
- **Process**: Language detection and processing through Bhashini API
- **Output**: Response in the same language as input

### Module B: Lok-Sahayak (Grievance Agent)
**Purpose**: Convert unstructured voice complaints into formal legal documents

**FR-B1: Voice Complaint Capture**
- **Input**: Unstructured voice complaint (e.g., "Mere gaon mein paani ki samasya hai")
- **Process**: 
  - Speech-to-text conversion with PII detection
  - Entity extraction (Name, Location, Issue Type, Date)
  - Sentiment analysis and urgency classification
- **Output**: Structured complaint data with metadata

**FR-B2: Legal Document Generation**
- **Input**: Structured complaint data
- **Process**: 
  - Generate formal complaint using predefined legal templates
  - Create PDF document using fpdf library
  - Apply DPDP Act 2023 compliant PII redaction
- **Output**: Professional PDF grievance document

**FR-B3: Document Storage and Tracking**
- **Input**: Generated PDF document
- **Process**: 
  - Upload to Amazon S3 with secure access controls
  - Create tracking entry in DynamoDB
  - Generate unique reference number
- **Output**: Downloadable PDF link and tracking reference

### Module C: Gram-Vyapar (Marketplace)
**Purpose**: Enable voice-based agricultural produce trading

**FR-C1: Voice Listing Creation**
- **Input**: Voice description (e.g., "Main 50 kilo pyaaz bech raha hun, 20 rupaye kilo")
- **Process**: 
  - Extract product details (type, quantity, price, location)
  - Validate pricing against market rates
  - Create structured marketplace entry
- **Output**: Active listing in marketplace database

**FR-C2: Buyer-Seller Connection**
- **Input**: Buyer interest in specific listing
- **Process**: 
  - Generate secure WhatsApp connection link
  - Notify both parties with contact details
  - Log transaction initiation
- **Output**: Direct communication channel between buyer and seller

**FR-C3: Market Price Intelligence**
- **Input**: Product type and location
- **Process**: Aggregate pricing data from active listings
- **Output**: Current market rate recommendations

## Non-Functional Requirements (NFRs)

### NFR-1: Performance Requirements
- **Voice-to-Action Latency**: Maximum 3 seconds from voice input to system response on 4G networks
- **Concurrent Users**: Support minimum 1000 concurrent voice sessions
- **Response Time**: API responses within 500ms for 95% of requests
- **Throughput**: Process minimum 10,000 voice queries per hour

### NFR-2: Security and Privacy Requirements
- **PII Protection**: All personally identifiable information (Aadhaar, phone numbers, addresses) must be redacted before sending to LLM
- **Data Encryption**: All data in transit and at rest must be encrypted using AES-256
- **Access Control**: Role-based access with multi-factor authentication for admin functions
- **Audit Trail**: Complete logging of all user interactions and system actions
- **DPDP Act 2023 Compliance**: Full compliance with Digital Personal Data Protection Act 2023

### NFR-3: Availability and Reliability
- **Uptime**: 99.9% availability using AWS Serverless auto-scaling architecture
- **Disaster Recovery**: RTO (Recovery Time Objective) of 4 hours, RPO (Recovery Point Objective) of 1 hour
- **Fault Tolerance**: Graceful degradation with offline capability for critical functions
- **Monitoring**: Real-time health monitoring with automated alerting

### NFR-4: Scalability Requirements
- **Auto-scaling**: Automatic scaling based on demand using AWS Lambda and DynamoDB on-demand
- **Geographic Distribution**: Multi-region deployment for reduced latency
- **Storage Scaling**: Unlimited document storage capacity using Amazon S3
- **Database Performance**: Consistent single-digit millisecond latency for DynamoDB operations

### NFR-5: Usability Requirements
- **Voice Interface**: Natural language processing with 95% accuracy for supported languages
- **Accessibility**: Support for users with visual and hearing impairments
- **Offline Capability**: Basic functionality available without internet connectivity
- **Learning Curve**: New users should be able to complete basic tasks within 5 minutes

### NFR-6: Compliance and Regulatory
- **Language Support**: Primary support for Hindi and Marathi with English fallback
- **Legal Compliance**: Generated documents must meet legal standards for government submissions
- **Data Residency**: All user data must be stored within Indian borders
- **Government Integration**: API compatibility with existing government portals and systems

## Success Metrics
- **User Adoption**: 10,000+ active users within 6 months
- **Task Completion Rate**: 90% successful completion of voice-initiated tasks
- **User Satisfaction**: Net Promoter Score (NPS) above 70
- **Processing Efficiency**: 50% reduction in time to complete government-related tasks
- **Document Accuracy**: 95% accuracy in generated legal documents