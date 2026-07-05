# PLUSE - Health Management System

A comprehensive health management system with patient records, vital signs tracking, appointments scheduling, and medication management.

## Features

- **User Management**: Patient and doctor profiles with authentication
- **Vital Signs Tracking**: Temperature, heart rate, blood pressure, oxygen saturation, BMI tracking
- **Appointments**: Schedule and manage medical appointments
- **Medications**: Track active medications and prescriptions
- **Medical Records**: Store comprehensive medical history
- **Health Goals**: Set and track personal health objectives
- **Lab Tests**: Track laboratory test results
- **Health Alerts**: Receive health-related notifications

## Tech Stack

- **Backend**: Node.js + Express.js
- **Database**: PostgreSQL
- **Authentication**: JWT (JSON Web Tokens)
- **Security**: Bcrypt password hashing, Helmet for headers

## Installation

### Prerequisites
- Node.js 16+
- PostgreSQL 12+
- Docker & Docker Compose (optional)

### Local Setup

1. **Clone the repository**
```bash
git clone https://github.com/altratst/pluse.git
cd pluse
```

2. **Install dependencies**
```bash
npm install
```

3. **Environment Configuration**
```bash
cp .env.example .env
```
Edit `.env` with your configuration:
```
DB_HOST=localhost
DB_PORT=5432
DB_NAME=pluse_health
DB_USER=postgres
DB_PASSWORD=your_password
JWT_SECRET=your_jwt_secret_key
```

4. **Create Database**
```bash
createdb pluse_health
```

5. **Run Migrations**
```bash
npm run migrate
```

6. **Seed Database (Optional)**
```bash
npm run seed
```

7. **Start Development Server**
```bash
npm run dev
```

The server will start on `http://localhost:5000`

### Docker Setup

1. **Build and run with Docker Compose**
```bash
docker-compose up --build
```

This will:
- Start PostgreSQL container
- Run database migrations automatically
- Start the Node.js application server
- Expose API on `http://localhost:5000`

## API Endpoints

### Authentication
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - Login user

### Users
- `GET /api/users/profile` - Get user profile
- `PUT /api/users/profile` - Update user profile
- `GET /api/users` - Get all users (admin only)

### Vital Signs
- `POST /api/vitals` - Add vital signs
- `GET /api/vitals` - Get vital signs history
- `GET /api/vitals/latest` - Get latest vital signs

### Appointments
- `POST /api/appointments` - Create appointment
- `GET /api/appointments` - Get user appointments
- `PATCH /api/appointments/:id` - Update appointment status

### Medications
- `POST /api/medications` - Add medication
- `GET /api/medications` - Get active medications
- `PUT /api/medications/:id` - Update medication

### Health Goals
- `POST /api/health-goals` - Create health goal
- `GET /api/health-goals` - Get health goals
- `PATCH /api/health-goals/:id` - Update goal progress

## Database Schema

The database includes the following main tables:

- `users` - User accounts (patients, doctors, admins)
- `doctors` - Doctor profiles and specializations
- `vital_signs` - Health measurements
- `appointments` - Medical appointments
- `medications` - Active medications
- `medical_records` - Medical history
- `prescriptions` - Doctor prescriptions
- `lab_tests` - Laboratory test results
- `health_goals` - User health objectives
- `health_alerts` - Health notifications

## Authentication

All protected endpoints require a JWT token in the `Authorization` header:

```
Authorization: Bearer <your_jwt_token>
```

## Example Usage

### Register
```bash
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "patient@example.com",
    "password": "securepassword",
    "first_name": "John",
    "last_name": "Doe"
  }'
```

### Login
```bash
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "patient@example.com",
    "password": "securepassword"
  }'
```

### Add Vital Signs
```bash
curl -X POST http://localhost:5000/api/vitals \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "temperature": 98.6,
    "heart_rate": 72,
    "blood_pressure_systolic": 120,
    "blood_pressure_diastolic": 80,
    "oxygen_saturation": 98,
    "weight": 75,
    "height": 180
  }'
```

## Development

### Running Tests
```bash
npm test
```

### Code Structure
```
├── config/          # Configuration files
├── middleware/      # Express middleware
├── routes/          # API routes
├── migrations/      # Database migrations
├── seeds/           # Database seeds
├── server.js        # Main application file
└── package.json     # Dependencies
```

## Security Considerations

- Passwords are hashed using bcrypt
- JWT tokens expire after configured duration
- CORS enabled with configurable origins
- Helmet.js for HTTP header security
- Input validation on all endpoints
- SQL injection prevention using prepared statements

## Performance Optimizations

- Database indexes on frequently queried columns
- Connection pooling for PostgreSQL
- Request logging and monitoring
- Pagination support on list endpoints

## Future Enhancements

- Real-time notifications with WebSockets
- Advanced health analytics and reporting
- Integration with wearable devices
- Mobile application
- Video consultation support
- Electronic health records (EHR)
- Pharmacy integration
- Insurance claim processing

## Contributing

1. Create a feature branch (`git checkout -b feature/amazing-feature`)
2. Commit changes (`git commit -m 'Add amazing feature'`)
3. Push to branch (`git push origin feature/amazing-feature`)
4. Open a Pull Request

## License

MIT License - see LICENSE file for details

## Support

For issues and questions, please open an issue on GitHub.

## Authors

- **Your Name** - Initial work

## Acknowledgments

- PostgreSQL community
- Express.js community
- JWT authentication best practices
