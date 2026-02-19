# Supply Chain Management Platform - Technical Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [Technology Stack](#technology-stack)
3. [Project Structure](#project-structure)
4. [Database Schema](#database-schema)
5. [Application Architecture](#application-architecture)
6. [Key Features & Implementation](#key-features--implementation)
7. [API Integrations](#api-integrations)
8. [Authentication & Authorization](#authentication--authorization)
9. [Email System](#email-system)
10. [Deployment Configuration](#deployment-configuration)
11. [Environment Setup](#environment-setup)
12. [Development Guidelines](#development-guidelines)

---

## Project Overview

This is a Django-based web application that facilitates supply chain management by connecting manufacturers with suppliers. The platform enables manufacturers to post quote requests, suppliers to submit bids, and includes features for negotiation, carbon footprint calculation, route optimization, and AI-powered analytics.

**Project Type:** Django Web Application  
**Python Version:** Compatible with Python 3.8+  
**Django Version:** 5.1.7  
**Database:** SQLite3 (development)

---

## Technology Stack

### Backend
- **Framework:** Django 5.1.7
- **Language:** Python 3.8+
- **Database:** SQLite3 (development), PostgreSQL compatible (production)
- **Authentication:** Django's built-in authentication with custom backends

### External Services & APIs
- **OpenRouteService API:** Route calculation and distance measurement
- **Tavily API:** Commodity price fetching
- **Yahoo Finance API (via YFinance):** Commodity analytics
- **OpenAI GPT-4o-mini:** AI-powered supplier analysis and market insights
- **Geopy:** Geocoding and distance calculations

### Python Libraries
```
phidata==2.7.10          # AI agent framework
openai>=1.0.0            # OpenAI API client
tavily-python==0.5.1     # Tavily search API
Django==5.1.7            # Web framework
gunicorn==20.1.0         # WSGI HTTP server
geopy==2.4.1             # Geocoding library
sqlalchemy==2.0.40       # SQL toolkit
yfinance==0.2.55         # Yahoo Finance API
```

### Deployment
- **WSGI Server:** Gunicorn
- **Platform:** Heroku-compatible (Procfile included)

---

## Project Structure

```
yir/
├── manage.py                    # Django management script
├── requirements.txt              # Python dependencies
├── Procfile                      # Heroku deployment config
├── db.sqlite3                    # SQLite database
│
├── supplychain/                  # Main project directory
│   ├── __init__.py
│   ├── settings.py              # Django settings
│   ├── urls.py                  # Root URL configuration
│   ├── wsgi.py                  # WSGI config
│   └── asgi.py                  # ASGI config
│
├── home/                        # Home app (landing pages)
│   ├── models.py                # (Empty - no models)
│   ├── views.py                 # Static page views
│   ├── urls.py                  # URL routing
│   └── templates/               # Home page templates
│
├── manufacturer/                 # Manufacturer app
│   ├── models.py                # Manufacturer, QuoteRequest models
│   ├── views.py                 # Manufacturer views & logic
│   ├── forms.py                 # Registration & login forms
│   ├── urls.py                  # URL routing
│   ├── backends.py              # Custom auth backend
│   ├── utils.py                 # Commodity price & analytics
│   └── templates/               # Manufacturer templates
│
├── supplier/                     # Supplier app
│   ├── models.py                # Supplier, Bid, SupplierReview, SupplierInventory
│   ├── views.py                 # Supplier views & logic
│   ├── forms.py                 # Registration, login, bid, review forms
│   ├── urls.py                  # URL routing
│   ├── backends.py              # Custom auth backend
│   └── templates/               # Supplier templates
│
├── negotiation/                  # Negotiation app
│   ├── models.py                # Negotiation, NegotiationMessage
│   ├── views.py                 # Negotiation views
│   ├── forms.py                 # Counter offer & message forms
│   ├── urls.py                  # URL routing
│   └── templates/               # Negotiation templates
│
├── utils/                        # Utility modules
│   ├── carbon_calculator.py     # Carbon emissions calculation
│   ├── route_calculator.py       # Route optimization
│   └── email.py                  # Email sending utilities
│
└── templates/                    # Global templates
    └── emails/                   # Email templates (HTML)
```

---

## Database Schema

### Core Models

#### Manufacturer App

**Manufacturer**
```python
- user: OneToOneField(User)
- first_name: CharField(max_length=100)
- last_name: CharField(max_length=100)
- company_name: CharField(max_length=200)
- address: CharField(max_length=255)
- city: CharField(max_length=100)
- state: CharField(max_length=100)
- business_type: CharField(max_length=100)
- website: URLField(blank=True)
- phone_number: CharField(max_length=20)
- key_products: TextField
```

**QuoteRequest**
```python
- manufacturer: ForeignKey(Manufacturer)
- product: CharField(max_length=200)
- category: CharField(max_length=100)
- description: TextField
- deadline: DateField
- quantity: DecimalField(max_digits=10, decimal_places=2)
- unit: CharField(max_length=20, blank=True)
- currency: CharField(max_length=3, blank=True)
- status: CharField(choices: 'open', 'closed', 'awarded', 'expired')
- created_at: DateTimeField(auto_now_add=True)
- accepted_bid: ForeignKey(Bid, null=True, blank=True)
```

#### Supplier App

**Supplier**
```python
- user: OneToOneField(User)
- first_name: CharField(max_length=100)
- last_name: CharField(max_length=100)
- company_name: CharField(max_length=200)
- city: CharField(max_length=100)
- state: CharField(max_length=100)
- business_type: CharField(max_length=100)
- website: URLField(blank=True)
- phone_number: CharField(max_length=20)
- key_services: TextField
- wallet_address: CharField(max_length=42, blank=True, null=True)  # Ethereum wallet
- commodity_categories: JSONField(default=list)
- review_count: PositiveIntegerField(default=0)
- average_rating: DecimalField(max_digits=3, decimal_places=2, default=0.0)
```

**Bid**
```python
- supplier: ForeignKey(Supplier)
- quote: ForeignKey(QuoteRequest)
- bid_amount: DecimalField(max_digits=10, decimal_places=2)
- delivery_time: PositiveIntegerField
- comments: TextField(blank=True)
- submitted_at: DateTimeField(auto_now_add=True)
- feedback_given: BooleanField(default=False)
- status: CharField(choices: 'submitted', 'accepted', 'rejected')
- payment_status: CharField(choices: 'pending', 'deposited', 'released', 'disputed')
- transaction_hash: CharField(max_length=66, blank=True, null=True)
- transport_mode: CharField(choices: 'road', 'air')
- vehicle_type: CharField(choices: vehicle types, blank=True)
- vehicle_count: PositiveIntegerField(blank=True, null=True)
- load_percentage: PositiveIntegerField(validators: 1-100)
- empty_return: BooleanField(default=False)
- aircraft_type: CharField(choices: aircraft types, blank=True)
- flight_count: PositiveIntegerField(blank=True, null=True)
- calculated_emissions: FloatField(null=True, blank=True)  # kg CO2
- distance_km: FloatField(null=True, blank=True)
```

**SupplierReview**
```python
- supplier: ForeignKey(Supplier, related_name='reviews')
- manufacturer: ForeignKey(Manufacturer)
- bid: OneToOneField(Bid, related_name='review')
- rating: PositiveSmallIntegerField(choices: 1-5)
- comment: TextField(blank=True)
- created_at: DateTimeField(auto_now_add=True)
```

**SupplierInventory**
```python
- supplier: ForeignKey(Supplier, related_name='inventory_items')
- product_name: CharField(max_length=200)
- quantity: DecimalField(max_digits=10, decimal_places=2)
- unit: CharField(choices: kg, g, lb, oz, l, ml, unit, box, pack)
- price_per_unit: DecimalField(max_digits=10, decimal_places=2)
- last_updated: DateTimeField(auto_now=True)
- notes: TextField(blank=True, null=True)
```

#### Negotiation App

**Negotiation**
```python
- bid: OneToOneField(Bid, related_name='negotiation')
- status: CharField(choices: 'active', 'accepted', 'rejected', 'expired')
- created_at: DateTimeField(auto_now_add=True)
- updated_at: DateTimeField(auto_now=True)
- expiry_date: DateTimeField
```

**NegotiationMessage**
```python
- negotiation: ForeignKey(Negotiation, related_name='messages')
- sender: ForeignKey(User)
- message: TextField
- counter_amount: DecimalField(null=True, blank=True)
- counter_delivery_time: PositiveIntegerField(null=True, blank=True)
- created_at: DateTimeField(auto_now_add=True)
- is_read: BooleanField(default=False)
```

---

## Application Architecture

### URL Routing Structure

```
Root (supplychain/urls.py)
├── admin/                        # Django admin
├── ''                            # Home app
│   ├── ''                        # Home page
│   ├── logout/                   # Logout
│   ├── about/                    # About page
│   ├── contact/                  # Contact page
│   └── ...
├── manufacturer/                 # Manufacturer app
│   ├── register/                 # Registration
│   ├── login/                    # Login
│   ├── dashboard/                # Dashboard
│   ├── request-quote/            # Create quote request
│   ├── quote-history/            # View quote history
│   ├── quote-bids/<id>/          # View bids for quote
│   ├── accept-bid/<id>/          # Accept bid
│   ├── supplier-profile/<id>/     # View supplier profile
│   ├── analyze-supplier/<id>/     # AI supplier analysis
│   ├── calculate-carbon-footprint/ # Carbon calculator
│   └── commodity-analytics/      # Commodity analytics
├── supplier/                     # Supplier app
│   ├── register/                 # Registration
│   ├── login/                    # Login
│   ├── dashboard/                # Dashboard
│   ├── bid/<quote_id>/           # Submit bid
│   ├── inventory/                 # Inventory management
│   ├── negotiations/             # View negotiations
│   ├── calculate-route/          # Route calculator
│   └── ai-suggestions/           # AI market suggestions
└── negotiation/                  # Negotiation app
    ├── start/<bid_id>/           # Start negotiation
    ├── <negotiation_id>/         # View negotiation
    ├── <negotiation_id>/accept/  # Accept negotiation
    ├── <negotiation_id>/reject/  # Reject negotiation
    └── payment-gateway/          # Payment gateway
```

### Authentication Flow

1. **Custom Authentication Backends:**
   - `ManufacturerBackend`: Authenticates users with Manufacturer profile
   - `SupplierBackend`: Authenticates users with Supplier profile
   - `ModelBackend`: Fallback for admin/superusers

2. **Registration Flow:**
   - User creates account with username/email/password
   - Profile created (Manufacturer or Supplier)
   - Welcome email sent
   - Auto-login after registration

3. **Login Flow:**
   - User enters credentials
   - Backend checks user type (Manufacturer/Supplier)
   - Redirects to appropriate dashboard

---

## Key Features & Implementation

### 1. Quote Request System

**Manufacturer Side:**
- Create quote requests with product details, quantity, deadline
- View all quote requests with status filtering
- View bids for each quote with sorting options (price, rating, newest)

**Implementation:**
- `QuoteRequest` model stores request details
- Status tracking: open → closed/awarded/expired
- Email notifications on quote creation

### 2. Bidding System

**Supplier Side:**
- Browse open quote requests
- Filter by category
- Submit bids with:
  - Bid amount
  - Delivery time
  - Transport mode (road/air)
  - Vehicle/aircraft details
  - Load percentage
  - Comments

**Implementation:**
- `Bid` model with transport details
- Form validation for transport-specific fields
- Email notifications to manufacturer on new bid

### 3. Negotiation System

**Features:**
- Manufacturers can initiate negotiations on bids
- Real-time messaging between parties
- Counter-offers with amount and delivery time
- 7-day expiry period
- Status tracking (active/accepted/rejected/expired)

**Implementation:**
- `Negotiation` model with OneToOne relationship to Bid
- `NegotiationMessage` for threaded conversations
- Email notifications on new messages/counter-offers

### 4. Carbon Footprint Calculator

**Features:**
- Calculate CO2 emissions for road transport
- Supports multiple vehicle types (small/medium/large/articulated trucks)
- Accounts for load percentage and empty return trips
- Uses OpenRouteService API for route distance

**Implementation:**
- `CarbonEmissionsCalculator` class in `utils/carbon_calculator.py`
- Emission factors per vehicle type (grams CO2/km)
- Formula: `(emission_factor × distance × vehicle_count) / load_factor`
- Returns: total emissions (kg), per-vehicle emissions, equivalent trees

**API Endpoint:**
- `POST /manufacturer/calculate-carbon-footprint/`
- Request: `{start_addr, end_addr, vehicle_type, vehicle_count, load_percentage, empty_return}`
- Response: `{success, distance_km, total_emissions_kg, emissions_per_vehicle_kg, equivalent_trees}`

### 5. Route Calculator

**Features:**
- Calculate road routes with turn-by-turn directions
- Calculate air routes with distance and transit time
- Account for lead time in delivery calculations
- Detailed route analysis (highway percentage, turns, etc.)

**Implementation:**
- `RouteFinder` class in `utils/route_calculator.py`
- OpenRouteService API for road routes
- Geopy for air route distance (great circle)
- Realistic speed adjustments (50 km/h avg for road)

**API Endpoint:**
- `POST /supplier/calculate-route/`
- Request: `{supplier_city, supplier_state, manufacturer_city, manufacturer_state, transport_mode, lead_time}`
- Response: `{success, mode, distance, transit_time, route_summary, total_days, delivery_days, ...}`

### 6. AI-Powered Features

#### Supplier Analysis
- Analyzes supplier reviews using GPT-4o-mini
- SQL agent queries `supplier_supplierreview` table
- Provides performance assessment and recommendations

**Implementation:**
- `analyze_supplier` view in `manufacturer/views.py`
- Uses Phi framework with SQLTools
- Queries database for review comments
- Returns structured analysis

#### Market Analytics
- Commodity price fetching via Tavily API
- Commodity analytics via Yahoo Finance
- AI agent maps commodities to stock tickers
- Provides price trends, recommendations, risk assessment

**Implementation:**
- `CommodityPriceFetcher` class in `manufacturer/utils.py`
- `CommodityAnalytics` class with YFinanceTools
- GPT-4o-mini agent with specialized prompts

#### Supplier Market Suggestions
- Analyzes quote request patterns
- Identifies high-demand products/categories
- Provides recommendations for inventory management

**Implementation:**
- `ai_suggestions` view in `supplier/views.py`
- SQL agent analyzes `manufacturer_quoterequest` table
- Returns top categories and product recommendations

### 7. Review & Rating System

**Features:**
- Manufacturers can review suppliers after accepting bids
- 5-star rating system (1-5)
- Text comments
- Automatic calculation of average rating and review count

**Implementation:**
- `SupplierReview` model with OneToOne to Bid
- `update_review_stats()` method on Supplier model
- Triggers on review save
- Displayed on supplier profiles

### 8. Inventory Management

**Features:**
- Suppliers can manage inventory items
- Track product name, quantity, unit, price
- Add notes for each item

**Implementation:**
- `SupplierInventory` model
- CRUD operations via `inventory_management` view
- Form validation for units and quantities

### 9. Payment Gateway Integration

**Features:**
- Payment gateway view for manufacturers
- Accepts supplier wallet address, amount, negotiation ID
- Ethereum wallet address validation (42 chars, starts with 0x)

**Implementation:**
- `payment_gateway_view` in `manufacturer/views.py`
- Redirects to payment gateway with parameters
- Transaction hash stored in Bid model

---

## API Integrations

### OpenRouteService API
- **Purpose:** Route calculation and distance measurement
- **Endpoint:** `https://api.openrouteservice.org/v2/directions/driving-car`
- **Authentication:** API key in `settings.OPENROUTE_API_KEY`
- **Usage:** Route calculation, distance measurement for carbon calculator

### Tavily API
- **Purpose:** Commodity price fetching
- **Authentication:** Environment variable `TAVILY_API_KEY`
- **Usage:** Real-time commodity price data for Indian markets

### Yahoo Finance API (YFinance)
- **Purpose:** Commodity analytics and market data
- **Library:** `yfinance==0.2.55`
- **Usage:** Historical prices, analyst recommendations, technical indicators

### OpenAI API
- **Purpose:** AI-powered analysis and recommendations
- **Model:** GPT-4o-mini
- **Framework:** Phi (phidata)
- **Usage:** Supplier analysis, market insights, commodity analytics

---

## Authentication & Authorization

### Custom Authentication Backends

**ManufacturerBackend** (`manufacturer/backends.py`):
```python
- Checks if user has Manufacturer profile
- Returns user if authenticated and is manufacturer
- Used for manufacturer login
```

**SupplierBackend** (`supplier/backends.py`):
```python
- Checks if user has Supplier profile
- Returns user if authenticated and is supplier
- Used for supplier login
```

### Authorization Patterns

1. **View-Level Protection:**
   - `@login_required` decorator
   - Manual checks: `if not request.user.is_authenticated`

2. **Object-Level Protection:**
   - Check ownership: `quote.manufacturer.user != request.user`
   - Check user type: `hasattr(request.user, 'manufacturer')`

3. **Permission Checks:**
   - Manufacturer can only manage their own quotes
   - Supplier can only view/manage their own bids
   - Negotiation participants can only view their negotiations

---

## Email System

### Email Configuration

**Settings** (`supplychain/settings.py`):
```python
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = '...'
EMAIL_HOST_PASSWORD = '...'
DEFAULT_FROM_EMAIL = '...'
```

### Email Templates

Located in `templates/emails/`:
- `account_created_manufacturer.html`
- `account_created_supplier.html`
- `bid_submitted.html`
- `bid_status_update.html`
- `new_bid_received.html`
- `quote_submitted.html`
- `negotiation_started.html`
- `negotiation_message.html`
- `counter_offer_received.html`
- `negotiation_rejected.html`
- `new_review_received.html`

### Email Sending

**Utility Function** (`utils/email.py`):
```python
send_email(subject, to_email, template_name, context)
- Renders HTML template
- Strips HTML for text version
- Sends multipart email
```

**Usage Examples:**
- Welcome emails on registration
- Bid submission confirmations
- Negotiation notifications
- Review notifications

---

## Deployment Configuration

### Procfile
```
web: gunicorn supplychain.wsgi:application
```

### Settings for Production

**Required Changes:**
1. Set `DEBUG = False`
2. Configure `ALLOWED_HOSTS` with domain
3. Use environment variables for secrets:
   - `SECRET_KEY`
   - `OPENROUTE_API_KEY`
   - `TAVILY_API_KEY`
   - `OPENAI_API_KEY`
   - Email credentials
4. Switch to PostgreSQL:
   ```python
   DATABASES = {
       'default': {
           'ENGINE': 'django.db.backends.postgresql',
           'NAME': os.environ.get('DB_NAME'),
           'USER': os.environ.get('DB_USER'),
           'PASSWORD': os.environ.get('DB_PASSWORD'),
           'HOST': os.environ.get('DB_HOST'),
           'PORT': os.environ.get('DB_PORT', '5432'),
       }
   }
   ```
5. Configure static files:
   ```python
   STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
   ```
6. Set up media file storage (S3 or similar)

---

## Environment Setup

### Prerequisites
- Python 3.8+
- pip
- Virtual environment (recommended)

### Installation Steps

1. **Clone Repository:**
   ```bash
   git clone <repository-url>
   cd yir
   ```

2. **Create Virtual Environment:**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install Dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Set Environment Variables:**
   ```bash
   export OPENROUTE_API_KEY="your_key"
   export TAVILY_API_KEY="your_key"
   export OPENAI_API_KEY="your_key"
   ```

5. **Run Migrations:**
   ```bash
   python manage.py migrate
   ```

6. **Create Superuser:**
   ```bash
   python manage.py createsuperuser
   ```

7. **Run Development Server:**
   ```bash
   python manage.py runserver
   ```

### Database Migrations

**Manufacturer App:**
- `0001_initial.py` - Initial Manufacturer model
- `0002_quoterequest.py` - QuoteRequest model
- `0003_quoterequest_status.py` - Status field
- `0004_quoterequest_accepted_bid.py` - Accepted bid relationship
- `0005_manufacturer_address.py` - Address field
- `0006_remove_quoterequest_annual_volume_and_more.py` - Field cleanup

**Supplier App:**
- Similar migration structure for Supplier, Bid, SupplierReview, SupplierInventory

**Negotiation App:**
- Migrations for Negotiation and NegotiationMessage models

---

## Development Guidelines

### Code Organization

1. **App Structure:**
   - Each app is self-contained (models, views, forms, urls)
   - Shared utilities in `utils/` directory
   - Global templates in `templates/`

2. **Naming Conventions:**
   - Models: PascalCase (e.g., `QuoteRequest`)
   - Views: snake_case (e.g., `manufacturer_dashboard`)
   - URLs: kebab-case (e.g., `request-quote/`)

3. **Form Handling:**
   - Use Django ModelForms where possible
   - Custom validation in `clean()` methods
   - Error messages via Django messages framework

4. **Error Handling:**
   - Try-except blocks for external API calls
   - User-friendly error messages
   - Logging for debugging

### Best Practices

1. **Security:**
   - Never commit API keys or secrets
   - Use environment variables
   - Validate user input
   - Use CSRF protection (enabled by default)

2. **Performance:**
   - Use `select_related()` and `prefetch_related()` for foreign keys
   - Paginate large querysets
   - Cache expensive calculations

3. **Testing:**
   - Write unit tests for models
   - Test views with Django test client
   - Test forms with various inputs

4. **Documentation:**
   - Docstrings for classes and functions
   - Comments for complex logic
   - README for setup instructions

### Common Tasks

**Creating a New Model:**
1. Define model in `models.py`
2. Create migration: `python manage.py makemigrations`
3. Apply migration: `python manage.py migrate`
4. Register in admin if needed

**Adding a New View:**
1. Create view function in `views.py`
2. Add URL pattern in `urls.py`
3. Create template if needed
4. Test with development server

**Adding Email Notification:**
1. Create HTML template in `templates/emails/`
2. Call `send_email()` with appropriate context
3. Test with development email backend

---

## Troubleshooting

### Common Issues

1. **API Key Errors:**
   - Check environment variables are set
   - Verify API keys are valid
   - Check API rate limits

2. **Database Errors:**
   - Run migrations: `python manage.py migrate`
   - Check database file permissions (SQLite)
   - Verify database connection (PostgreSQL)

3. **Email Not Sending:**
   - Check email settings in `settings.py`
   - Verify SMTP credentials
   - Test with console backend: `EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'`

4. **Static Files Not Loading:**
   - Run `python manage.py collectstatic`
   - Check `STATIC_URL` and `STATICFILES_DIRS` in settings
   - Verify file paths in templates

---

## Future Enhancements

1. **Real-time Notifications:** WebSocket support for live updates
2. **Advanced Analytics:** Dashboard with charts and graphs
3. **Mobile App:** React Native or Flutter app
4. **Blockchain Integration:** Smart contracts for payments
5. **Multi-language Support:** i18n for international users
6. **Advanced Search:** Elasticsearch for product/supplier search
7. **Document Management:** File uploads for contracts, certificates
8. **Reporting:** PDF generation for quotes, invoices

---

## License

[Specify license here]

## Contributors

[Add contributor information]

---

**Last Updated:** [Date]  
**Version:** 1.0.0
