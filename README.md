# Prepaid Maintenance Contract Administration System

A comprehensive web-based platform for dealerships to manage prepaid maintenance and service contracts with multi-dealership support, role-based access control, and external API integration.

## Features

### Customer & Vehicle Management
- **Customer Profiles** - Create and manage customer information with auto-generated customer numbers
- **VIN Decoder Integration** - Enter 17-character VIN and automatically decode year, make, and model
- **Universal Search** - Search by customer name, phone, email, VIN, or customer number
- **Unified Dashboard** - View all customer vehicles and contracts in a single interface

### Contract Management
- **Reusable Templates** - Build contract packages with customizable services and pricing
- **Smart Matching** - Automatically filter contracts by vehicle make, model, and year compatibility
- **Service Tracking** - Track individual service usage with progress indicators
- **Auto-Fulfillment** - Contracts automatically mark as fulfilled when all services are used
- **Status Management** - Track contracts through their lifecycle (Unpaid → Paid → Fulfilled)

### Reporting & Analytics
- **Accounting Reports** - Generate financial reports with cost, revenue, and profit analysis
- **Flexible Filters** - Filter by date range, status, dealership, or assigned user
- **Export Options** - Download reports as CSV or PDF
- **Dashboard Metrics** - Real-time statistics on active contracts, revenue, and fulfillment

### External API Integration
Three RESTful API endpoints for third-party system integration:
- **Rating API** - Query available contract packages for specific vehicles
- **Purchase API** - Create contracts programmatically with automatic PDF generation
- **Void API** - Cancel contracts from external systems

### Multi-Dealership Support
- Manage multiple dealership locations in a single system
- User access control per dealership
- Dealership-specific branding and PDF templates
- Consolidated or isolated reporting

### Role-Based Access Control
- **Administrators** - Full system access, user management, API key generation
- **F&I Managers** - Create and sell contracts
- **Accounting Clerks** - Approve payments and mark contracts as paid
- **Service Advisors** - Track service usage and customer visits

## Technology Stack

**Backend**
- Node.js with Express.js
- MongoDB database
- JWT authentication with bcrypt
- OpenAPI/Swagger documentation
- PDFKit for contract generation

**Frontend**
- React.js with hooks
- Redux or Context API for state management
- Material-UI or Tailwind CSS
- Axios for API communication

**External Services**
- NHTSA vPIC API for VIN decoding
- Cloud storage for PDF documents

## Quick Start

### Prerequisites
- Node.js 18+ and npm
- MongoDB 6+
- Git

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/prepaid-maintenance.git
cd prepaid-maintenance

# Install backend dependencies
cd backend
npm install

# Install frontend dependencies
cd ../frontend
npm install
```

### Configuration

Create a `.env` file in the backend directory:

```env
PORT=3000
MONGODB_URI=mongodb://localhost:27017/prepaid-maintenance
JWT_SECRET=your-secret-key-here
JWT_EXPIRATION=15m
REFRESH_TOKEN_EXPIRATION=7d

# VIN Decoder API (NHTSA is free, no key needed)
VIN_DECODER_URL=https://vpic.nhtsa.dot.gov/api

# Cloud storage for PDFs (configure based on provider)
STORAGE_PROVIDER=local
STORAGE_PATH=./uploads
```

### Running the Application

```bash
# Start MongoDB (if not running as service)
mongod

# Start backend server
cd backend
npm run dev

# Start frontend development server
cd frontend
npm start
```

The application will be available at `http://localhost:3000`

### Default Admin Account

After initial setup, create the first admin user:

```bash
cd backend
npm run create-admin
```

This will prompt you to create an administrator account.

## API Documentation

### Authentication
All protected endpoints require a JWT token in the Authorization header:
```
Authorization: Bearer <token>
```

### External API Endpoints

**Rating API** - Get available contracts for a vehicle
```http
POST /api/external/v1/rating
X-API-Key: your-api-key

{
  "dealer_code": "DEALER001",
  "product_type": "Prepaid Maintenance",
  "vin": "1HGBH41JXMN109186",
  "mileage": 15000,
  "in_service_date": "2021-06-15"
}
```

**Purchase API** - Create a new contract
```http
POST /api/external/v1/purchase
X-API-Key: your-api-key

{
  "dealer_code": "DEALER001",
  "customer": {
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@example.com",
    "phone": "208-555-0100"
  },
  "vehicle": {
    "vin": "1HGBH41JXMN109186"
  },
  "contract": {
    "contract_template_id": "uuid",
    "sale_date": "2024-10-15",
    "expiration_date": "2026-10-15"
  }
}
```

**Void API** - Cancel a contract
```http
POST /api/external/v1/void
X-API-Key: your-api-key

{
  "dealer_code": "DEALER001",
  "customer_number": "C12345",
  "vin": "1HGBH41JXMN109186",
  "contract_number": "PM-2024-001"
}
```

Full API documentation available at `/api-docs` when server is running.

## Project Structure

```
prepaid-maintenance/
├── backend/
│   ├── routes/          # API route handlers
│   ├── models/          # Database models
│   ├── middleware/      # Auth, RBAC, validation
│   ├── services/        # Business logic
│   ├── utils/           # Helper functions
│   └── server.js        # Entry point
├── frontend/
│   ├── src/
│   │   ├── components/  # React components
│   │   ├── pages/       # Page components
│   │   ├── context/     # State management
│   │   ├── api/         # API client
│   │   └── App.js       # Root component
│   └── public/
├── docs/
│   ├── system-overview.txt    # Non-technical overview
│   ├── dev-structure.txt      # Technical specifications
│   ├── mockup_screens.txt     # UI wireframe descriptions
│   └── mockups.html           # Interactive wireframes
└── README.md
```

## Development Roadmap

### Phase 1: Core Infrastructure (Weeks 1-3)
- Database schema and migrations
- Authentication system
- User and dealership management
- Admin portal basics

### Phase 2: Customer & Vehicle Management (Weeks 4-5)
- Customer CRUD operations
- Vehicle management with VIN decoding
- Dashboard and search functionality

### Phase 3: Contract System (Weeks 6-8)
- Contract template builder
- Contract assignment logic
- Service visit tracking
- Status management and validation

### Phase 4: Reporting & Advanced Features (Weeks 9-10)
- Accounting reports
- PDF generation system
- Dashboard enhancements

### Phase 5: External API (Weeks 11-12)
- API key management
- Rating, Purchase, and Void API implementation
- API documentation

### Phase 6: Testing & Deployment (Weeks 13-14)
- Comprehensive testing
- Security audit
- Production deployment
- User training documentation

**Estimated Timeline:** 14 weeks
**Estimated Cost:** $28,800 @ $50/hr (576 hours)

## Testing

```bash
# Run backend tests
cd backend
npm test

# Run frontend tests
cd frontend
npm test

# Run E2E tests
npm run test:e2e
```

## Deployment

### Production Build

```bash
# Build frontend
cd frontend
npm run build

# The build folder contains optimized production files
# Configure your web server to serve these files
```

### Environment Variables (Production)

```env
NODE_ENV=production
MONGODB_URI=mongodb://production-server/prepaid-maintenance
JWT_SECRET=<strong-random-secret>
STORAGE_PROVIDER=aws-s3
AWS_S3_BUCKET=your-bucket-name
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
```

### Security Checklist
- ✅ Use HTTPS/TLS for all communications
- ✅ Rotate JWT secrets regularly
- ✅ Enable rate limiting on authentication endpoints
- ✅ Configure CORS restrictions
- ✅ Use environment-specific API keys
- ✅ Enable database encryption at rest
- ✅ Regular security audits and updates

## Contributing

We welcome contributions! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

Please ensure your code follows our coding standards and includes appropriate tests.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

For support, please contact:
- Email: support@example.com
- Documentation: [Link to full documentation]
- Issues: [GitHub Issues](https://github.com/yourusername/prepaid-maintenance/issues)

## Acknowledgments

- NHTSA vPIC API for VIN decoding
- Open source libraries and frameworks used in this project
- Contributors and testers

---

**Status:** 📋 Planning & Design Complete
**Next Step:** Begin Phase 1 Development

For detailed system documentation, see:
- [System Overview](docs/system-overview.txt) - User-facing features and workflows
- [Development Structure](docs/dev-structure.txt) - Technical architecture and specifications
- [UI Mockups](docs/mockups.html) - Interactive wireframes (open in browser)
