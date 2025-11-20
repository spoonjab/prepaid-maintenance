# Prepaid Maintenance Contract Administration System

## Project Overview

This is a web-based platform that helps dealerships manage prepaid maintenance and service contracts. The system handles customer/vehicle tracking, contract creation and assignment, service usage tracking, accounting reports, and external API integrations.

**Tech Stack:**
- **Backend:** Node.js with Express.js, MongoDB
- **Frontend:** React.js with Material-UI/Tailwind CSS
- **Authentication:** JWT tokens with bcrypt
- **External APIs:** VIN decoder (NHTSA vPIC), Rating/Purchase/Void APIs
- **Hosting:** Linux/Ubuntu

## Key Concepts

### User Roles
1. **Administrators** - Full system access, create dealerships, manage users, generate API keys
2. **Regular Users** - Access only assigned dealerships with position-based permissions:
   - **Accounting Clerks:** Only role that can mark contracts as "Active Paid"
   - **F&I Managers:** Create and sell contracts
   - **Service Advisors:** Mark services as used

### Contract Statuses
- **Active Unpaid** - Contract assigned but payment not received
- **Active Paid** - Payment received, services can be used
- **Pending** - Created via external API, needs approval
- **Fulfilled** - All services completed (automatic)
- **Void** - Contract cancelled
- **Canceled** - Contract terminated

### Multi-Dealership Support
Users can be assigned to multiple dealerships. They switch between dealerships in the UI and only see data for their current selection. Admins see all dealerships.

## Core Features

### 1. Customer & Vehicle Management
- Create customer profiles with auto-generated customer numbers
- Add vehicles with **VIN decoder** - enter 17-char VIN, click Decode, auto-populates year/make/model
- Search customers by name, phone, email, customer number, or VIN
- Dashboard shows all customer vehicles and contracts in one view

### 2. Contract Templates
- Build reusable contract packages
- Define applicable vehicles by **make, model, and/or year** (at least one required)
- Set pricing (cost/selling price), duration, mileage limits, deductible
- Add multiple services - **service prices must total = selling price** (validated)
- Example: "Honda 24-Month Maintenance" applies to Honda (make) 2020-2024 (years)

### 3. Smart Contract Assignment
When assigning a contract to a vehicle, system **automatically filters** to show only contracts matching that vehicle's make, model, or year. Prevents assigning incompatible contracts.

### 4. Service Tracking
- Each contract has multiple services (Oil Change, Tire Rotation, etc.)
- Service Advisors click "Mark as Used" → confirmation popup → records date
- Progress bar shows X of Y services used (25%, 50%, etc.)
- When last service used → contract auto-updates to "Fulfilled" status

### 5. Accounting Reports
- Filter by date range, status, dealership, assigned user
- Shows contract details, cost/price/profit, services completed
- Export to CSV or PDF
- Summary statistics (total contracts, revenue, profit)

### 6. External API Integration
Three API endpoints for external systems (Darwin F&I, etc.):

**Rating API** - Query available contracts for a vehicle:
```
POST /api/external/v1/rating
{ dealer_code, vin, product_type, mileage, in_service_date }
→ Returns matching contract templates
```

**Purchase API** - Create contract from external system:
```
POST /api/external/v1/purchase
{ dealer_code, customer, vehicle, contract }
→ Creates customer/vehicle, assigns contract with "Pending" status, generates PDF
```

**Void API** - Cancel a contract:
```
POST /api/external/v1/void
{ dealer_code, customer_number, vin, contract_number }
→ Marks contract as "Void"
```

All APIs use API Key authentication (X-API-Key header).

### 7. PDF Generation
- Automatic PDF generation when contracts purchased via API
- Customizable per dealership (logo, sections, footer, custom fields)
- 4-page format: Page 1 (contract details), Pages 2-3 (terms), Page 4 (service schedule)
- Uses template variables: `{{customer_name}}`, `{{vehicle_vin}}`, `{{contract_number}}`, etc.

## Database Schema Highlights

### Key Collections/Tables
- **users** - Email/password, role (admin/user), position_id, is_active
- **positions** - Predefined: Admin, F&I Manager, Accounting Clerk, Service Advisor
- **dealerships** - Dealer info + pdf_template_config JSON
- **user_dealership_assignments** - Many-to-many user ↔ dealership mapping
- **customers** - Customer info + dealership_id + auto-generated customer_number
- **vehicles** - VIN (unique), year/make/model, customer_id
- **contract_templates** - Reusable packages with applicable_makes/models/years (JSON arrays)
- **contract_template_services** - Services in each template
- **customer_contracts** - Assigned contracts with status, sale_date, expiration_date
- **contract_status_history** - Audit log of status changes
- **service_visits** - Individual services with is_used flag and used_date
- **api_keys** - Hashed API keys for external integrations

### Important Relationships
- Customer → multiple Vehicles
- Vehicle → multiple Contracts
- Contract → multiple Service Visits
- User ↔ multiple Dealerships (many-to-many)

## Common Workflows

### Workflow 1: New Customer Buys Contract
1. Add customer → auto-generates customer number
2. Add vehicle → decode VIN → verify year/make/model
3. Assign contract → system shows only matching contracts
4. Contract starts as "Active Unpaid"
5. Accounting Clerk changes to "Active Paid" when payment received

### Workflow 2: Customer Comes for Service
1. Search by name/VIN/phone
2. View customer → see all vehicles and contracts
3. Click "Mark as Used" on available service
4. Confirmation popup → confirm
5. Service marked with today's date, progress updates
6. If last service → contract auto-marks "Fulfilled"

### Workflow 3: External API Purchase
1. External system calls Purchase API with customer/vehicle/contract data
2. System finds/creates customer, decodes VIN, creates vehicle
3. Validates contract template applies to vehicle
4. Creates contract with "Pending" status
5. Generates PDF
6. Returns contract_number and PDF URL
7. Dealership staff reviews pending contract, approves → changes to "Active Paid"

### Workflow 4: Create Contract Template
1. Name template "Honda 24-Month Maintenance"
2. Select makes: Honda
3. Select years: 2020-2024
4. Set cost $200, price $300, duration 24 months
5. Add services: Oil Change $75, Tire Rotation $75, Filter $75, Brakes $75
6. System validates: $75 + $75 + $75 + $75 = $300 ✓
7. Save → template available for assignment

## Important Validations

### Contract Template
- At least one of: make, model, or year must be specified
- Sum of service prices must equal selling price
- Cost < selling price (typically)
- Contract length in months required

### Contract Assignment
- Only show contracts matching vehicle make/model/year
- Expiration date > sale date
- Contract template must belong to same dealership

### Service Usage
- Contract must be "Active Paid" to mark services used
- Confirmation required before marking
- Auto-fulfillment when last service used
- Records timestamp and user who marked it

### Status Changes
- Only Accounting Clerks can set "Active Paid"
- "Fulfilled" cannot be set manually (automatic only)
- All status changes logged in contract_status_history

## Security Notes

- JWT tokens (15-min expiration) + refresh tokens (7-day)
- bcrypt password hashing (10+ rounds)
- API keys hashed before database storage
- Role-based access control middleware on all routes
- Dealership access verification (users only see assigned dealerships)
- Rate limiting on auth endpoints (5 attempts/15 min)
- Rate limiting on API endpoints (1000 req/hour per key)

## File Structure (Expected)

```
/backend
  /routes
    /api
      auth.js           # Login, logout, token refresh
      customers.js      # CRUD customers
      vehicles.js       # CRUD vehicles, VIN decode
      contracts.js      # Templates, assignments, service visits
      reports.js        # Accounting reports
      users.js          # User management (admin)
      dealerships.js    # Dealership CRUD (admin)
      api-keys.js       # API key management (admin)
    /external
      rating.js         # External rating API
      purchase.js       # External purchase API
      void.js           # External void API
  /middleware
    auth.js             # JWT verification
    rbac.js             # Role-based access control
    dealership-access.js
  /models
    User.js, Customer.js, Vehicle.js, Contract.js, etc.
  /services
    vin-decoder.js      # NHTSA API integration
    pdf-generator.js    # Contract PDF generation
  /utils
    validation.js

/frontend
  /src
    /components
      Dashboard, CustomerList, VehicleForm, ContractTemplateBuilder, etc.
    /pages
      Login, Dashboard, Customers, Reports, Admin/Users, etc.
    /context
      AuthContext.js    # User auth state
      DealershipContext.js # Selected dealership
    /api
      client.js         # Axios wrapper with JWT
```

## Testing Strategy

### Unit Tests
- User authentication logic
- Permission validation
- VIN decoding integration
- Service fulfillment auto-logic (when last service used)
- PDF generation

### Integration Tests
- Complete workflows (customer → vehicle → contract → service usage)
- All CRUD endpoints
- External API endpoints (Rating, Purchase, Void)
- Status change workflows

### E2E Tests
- Login and role-based access
- Contract lifecycle (create template → assign → use services → fulfill)
- API key generation and usage
- Report generation

## Implementation Phases (14 weeks, $28,800 @ $50/hr)

**Phase 1 (Weeks 1-3):** Core infrastructure - DB schema, auth, user/dealership management
**Phase 2 (Weeks 4-5):** Customer & vehicle CRUD, VIN decoder, dashboard search
**Phase 3 (Weeks 6-8):** Contract templates, assignment logic, service tracking, status management
**Phase 4 (Weeks 9-10):** Accounting reports, PDF generation
**Phase 5 (Weeks 11-12):** External APIs (Rating, Purchase, Void), API key management
**Phase 6 (Weeks 13-14):** Testing, security audit, deployment, documentation

## Key UI Screens (See mockups.html)

1. **Login** - Split-screen with branding
2. **Dashboard** - Stats cards, dealership switcher, recent contracts
3. **Customer List** - Search with filters, quick actions
4. **Customer Detail** - Customer info + all vehicles/contracts in one view
5. **Add Customer Modal** - Form with auto customer number
6. **Add Vehicle Modal** - VIN decoder feature
7. **Contract Templates** - Card grid of reusable packages
8. **Create Template** - Two-column form + live preview
9. **Assign Contract Modal** - Filtered list showing only applicable contracts
10. **Contract Detail** - Progress bar, service list with used dates, status history
11. **Mark Service Confirmation** - Safety popup before marking used
12. **Accounting Reports** - Filters, summary stats, export options
13. **User Management** - Admin panel for users/roles/positions
14. **API Keys** - Generate keys, view usage, endpoint documentation
15. **PDF Config** - Template customization with live preview

## Development Tips

### VIN Decoding
Use NHTSA free API:
```javascript
const response = await fetch(
  `https://vpic.nhtsa.dot.gov/api/vehicles/DecodeVin/${vin}?format=json`
);
// Parse response for Year, Make, Model
```

### Smart Contract Filtering
When fetching contracts for a vehicle:
```javascript
// Pseudo-code
const vehicleMake = "Honda";
const vehicleModel = "Accord";
const vehicleYear = 2023;

const applicableContracts = await ContractTemplate.find({
  dealership_id: currentDealership,
  $or: [
    { applicable_makes: vehicleMake },
    { applicable_models: vehicleModel },
    { applicable_years: vehicleYear }
  ]
});
```

### Auto-Fulfillment Logic
When marking service as used:
```javascript
// After marking service used
const totalServices = await ServiceVisit.countDocuments({
  customer_contract_id: contractId
});
const usedServices = await ServiceVisit.countDocuments({
  customer_contract_id: contractId,
  is_used: true
});

if (totalServices === usedServices) {
  await CustomerContract.updateOne(
    { _id: contractId },
    { status: 'Fulfilled' }
  );
  // Log status change in contract_status_history
}
```

### API Key Validation Middleware
```javascript
async function validateApiKey(req, res, next) {
  const apiKey = req.headers['x-api-key'];
  if (!apiKey) return res.status(401).json({ error: 'API key required' });

  const hashedKey = hashApiKey(apiKey);
  const key = await ApiKey.findOne({ api_key: hashedKey, is_active: true });

  if (!key) return res.status(401).json({ error: 'Invalid API key' });

  req.apiKey = key;
  await ApiKey.updateOne({ _id: key._id }, { last_used_at: new Date() });
  next();
}
```

## Common Questions

**Q: Can contracts be transferred between vehicles?**
A: No, contracts are tied to specific vehicles. You must void the old contract and create a new one.

**Q: What if I mark a service as used by mistake?**
A: Contact admin - they can correct in the database. UI doesn't allow unmarking to prevent accidents.

**Q: Can one vehicle have multiple contracts?**
A: Yes! Example: one maintenance contract + one extended warranty contract.

**Q: How does contract matching work?**
A: Contract shows if ANY condition matches: make OR model OR year. Example: Template with "Honda" (make) + "2020-2024" (years) will show for all Honda vehicles from 2020-2024, regardless of model.

**Q: What happens when contract expires?**
A: System doesn't auto-change status based on date. Use reports to filter expired contracts. Services can still be marked used if customer came in before expiration.

**Q: How do pending contracts work?**
A: External API purchases create "Pending" contracts. Staff reviews them in dashboard, verifies info, then changes to "Active Paid" to approve.

## Performance Targets

- Page load time < 2 seconds
- API response time < 500ms (95th percentile)
- Support 100 concurrent users
- Dashboard search < 1 second
- PDF generation < 3 seconds

## Future Enhancements

- Email notifications for expiring contracts
- SMS reminders for upcoming services
- Mobile app for service advisors
- DMS system integrations
- Advanced analytics dashboards
- Bulk import for customers/contracts

---

**Ready to develop!** All architecture, workflows, and UI designs are fully documented. See [system-overview.txt](../system-overview.txt), [dev-structure.txt](../dev-structure.txt), and [mockups.html](../mockups.html) for complete details.
