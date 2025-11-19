# QR Code Smart Parking System

A complete mobile and backend solution for smart parking management with QR code-based access, real-time slot availability, and automated booking system.

## Project Structure

```
├── backend/              # Node.js/Express API
│   ├── models/          # MongoDB schemas
│   ├── routes/          # API endpoints
│   ├── middleware/      # Authentication & validation
│   ├── utils/           # Utilities (QR generation)
│   └── server.js        # Entry point
└── frontend/            # React Native Expo mobile app
    ├── screens/         # App screens
    ├── context/         # State management (Auth)
    ├── services/        # API client
    └── App.jsx          # Root component
```

## Features

### User Features
- User registration and secure authentication
- Real-time parking availability map
- Manual slot selection from map
- Instant QR code generation after booking
- Active booking management
- QR code scanning for entry/exit

### Parking Features
- Multiple parking areas with different zones
- Parking slot status tracking (available/booked/reserved)
- Price per hour configuration
- Occupancy management
- Amenities listing

### Technical Features
- JWT token-based authentication
- MongoDB data persistence
- Real-time slot availability updates
- QR code generation and verification
- Booking overlap prevention
- Secure API endpoints with auth middleware

## Setup Instructions

### Backend Setup

1. **Install dependencies**
\`\`\`bash
cd backend
npm install
\`\`\`

2. **Configure environment variables**
\`\`\`bash
cp .env.example .env
# Edit .env with your MongoDB URI and JWT secret
\`\`\`

3. **Start MongoDB**
\`\`\`bash
# Make sure MongoDB is running locally or update MONGODB_URI in .env
\`\`\`

4. **Run the server**
\`\`\`bash
npm start
# For development with auto-reload: npm run dev
\`\`\`

The backend will start on `http://localhost:5000`

### Frontend Setup

1. **Install dependencies**
\`\`\`bash
cd frontend
npm install
\`\`\`

2. **Update API URL**
   - In `frontend/services/api.js`, ensure the API_URL points to your backend
   - For development: `http://localhost:5000/api`

3. **Start Expo**
\`\`\`bash
npm start
\`\`\`

4. **Run on device/emulator**
   - Press `i` for iOS simulator
   - Press `a` for Android emulator
   - Scan QR code with Expo app on physical device

## API Documentation

### Authentication Endpoints

#### Register
\`\`\`
POST /api/auth/register
Body: { name, email, phone, password }
Response: { token, user }
\`\`\`

#### Login
\`\`\`
POST /api/auth/login
Body: { email, password }
Response: { token, user }
\`\`\`

### Parking Endpoints

#### Get All Parking Areas
\`\`\`
GET /api/parking/areas
Response: [{ _id, name, address, location, totalSlots, availableSlots }]
\`\`\`

#### Get Slots for Area
\`\`\`
GET /api/parking/slots/:areaId
Response: [{ _id, slotId, status, price, type, location }]
\`\`\`

#### Get Area with Slots
\`\`\`
GET /api/parking/area/:areaId
Response: { area, slots }
\`\`\`

### Booking Endpoints

#### Create Booking
\`\`\`
POST /api/booking/create
Headers: Authorization: Bearer <token>
Body: { slotId, parkingAreaId, startTime, endTime }
Response: { booking, qrCode }
\`\`\`

#### Get User Bookings
\`\`\`
GET /api/booking/user/:userId
Headers: Authorization: Bearer <token>
Response: [{ bookings }]
\`\`\`

#### Complete Booking
\`\`\`
POST /api/booking/complete/:bookingId
Headers: Authorization: Bearer <token>
Response: { message, booking }
\`\`\`

#### Cancel Booking
\`\`\`
POST /api/booking/cancel/:bookingId
Headers: Authorization: Bearer <token>
Response: { message, booking }
\`\`\`

#### Verify QR Code
\`\`\`
POST /api/booking/verify-qr
Body: { qrData }
Response: { valid, booking }
\`\`\`

## Database Schema

### User
\`\`\`javascript
{
  name: String,
  email: String (unique),
  phone: String,
  password: String (hashed),
  createdAt: Date
}
\`\`\`

### ParkingArea
\`\`\`javascript
{
  name: String,
  address: String,
  location: { latitude, longitude },
  totalSlots: Number,
  availableSlots: Number,
  amenities: [String],
  operatingHours: { open, close },
  createdAt: Date
}
\`\`\`

### ParkingSlot
\`\`\`javascript
{
  slotId: String (unique),
  parkingAreaId: ObjectId (ref: ParkingArea),
  status: String (available/booked/reserved),
  location: { latitude, longitude },
  type: String (regular/handicap/vip),
  price: Number,
  createdAt: Date
}
\`\`\`

### Booking
\`\`\`javascript
{
  userId: ObjectId (ref: User),
  slotId: ObjectId (ref: ParkingSlot),
  parkingAreaId: ObjectId (ref: ParkingArea),
  startTime: Date,
  endTime: Date,
  status: String (active/completed/cancelled),
  qrCode: String (data URL),
  totalAmount: Number,
  createdAt: Date
}
\`\`\`

## Testing the Application

### Manual Testing Steps

1. **Register a new user**
   - Open the app and sign up with test credentials
   - Verify token is saved in AsyncStorage

2. **View parking map**
   - Allow location permissions
   - See parking areas marked on map
   - Click on a marker to see available slots

3. **Book a parking slot**
   - Select an available slot
   - Choose start and end times
   - Confirm booking
   - Receive QR code

4. **View active bookings**
   - Navigate to "My Bookings"
   - See all your active bookings
   - Use "Exit" to complete a booking
   - Use "Cancel" to cancel an active booking

5. **Scan QR code**
   - Navigate to QR Scanner
   - Scan the QR code from a booking
   - Verify the booking is valid

## Seeding Test Data

Create a script in `backend/scripts/seed.js` to populate test data:

\`\`\`javascript
import mongoose from 'mongoose';
import ParkingArea from '../models/ParkingArea.js';
import ParkingSlot from '../models/ParkingSlot.js';

async function seedDatabase() {
  try {
    await mongoose.connect(process.env.MONGODB_URI);

    // Clear existing data
    await ParkingArea.deleteMany({});
    await ParkingSlot.deleteMany({});

    // Create parking areas
    const area1 = await ParkingArea.create({
      name: 'Downtown Parking',
      address: 'Main Street, City Center',
      location: { latitude: 28.7041, longitude: 77.1025 },
      totalSlots: 50,
      availableSlots: 50,
    });

    // Create slots
    for (let i = 1; i <= 50; i++) {
      await ParkingSlot.create({
        slotId: `A${i}`,
        parkingAreaId: area1._id,
        status: 'available',
        location: { latitude: 28.7041 + Math.random() * 0.01, longitude: 77.1025 + Math.random() * 0.01 },
        price: 50,
      });
    }

    console.log('Database seeded successfully');
  } catch (error) {
    console.log('Seeding error:', error);
  }
}

seedDatabase();
\`\`\`

## Deployment

### Backend Deployment (Heroku/Vercel)
1. Push code to GitHub
2. Connect to deployment platform
3. Set environment variables
4. Deploy

### Frontend Deployment (Expo/EAS)
\`\`\`bash
eas build --platform ios
eas build --platform android
\`\`\`

## Common Issues & Solutions

**Issue: MongoDB connection error**
- Ensure MongoDB is running
- Check MONGODB_URI in .env

**Issue: QR code not generating**
- Verify qrcode package is installed
- Check booking data format

**Issue: Map not showing**
- Ensure location permissions are granted
- Check initial region coordinates

## Future Enhancements
- Payment gateway integration (Stripe/Razorpay)
- Admin dashboard for parking management
- Push notifications for bookings
- Duration extension capability
- Seasonal pricing
- Subscription plans
- Rating and reviews
- Multi-language support

## License
MIT
