## 📁 Files

- **Controller File:** `src/controller/retailerordercontroller.js`
- **Route File:** `src/route/retailerorder.route.js`

---

## 🧪 Testing Data Used

### 🧍 Retailer Document Used for Testing

```json
{
  "_id": { "$oid": "672f0f1b2222222222222222" },
  "name": "Priya Mehra",
  "email": "priya.mehra@example.com",
  "password": "$2b$10$hashedpasswordexample",
  "phone": "9876543210",
  "profileImage": "https://yourdomain.com/images/retailers/priya.jpg",
  "aadharNumber": "1234-5678-9123",
  "aadharCardURL": "https://yourdomain.com/documents/priya-aadhar.jpg",
  "address": "Flat No. 202, Rosewood Apartments, Jaipur, Rajasthan 302001",
  "city": "Jaipur",
  "state": "Rajasthan",
  "role": "retailer",
  "stores": [{ "$oid": "672f0f9e1234567890abcdef" }],
  "createdAt": { "$date": "2025-04-04T10:00:00.000Z" },
  "updatedAt": { "$date": "2025-04-04T10:00:00.000Z" },
  "__v": 0
}
```

---

### 🏪 Shop Document Used for Testing

```json
{
  "_id": { "$oid": "672f0f9e1234567890abcdef" },
  "StoreName": "Elegant Styles Salon",
  "StoreCity": "Jaipur",
  "StoreCategory": { "$oid": "672f0f1a1111111111111111" },
  "RetailerID": { "$oid": "672f0f1b2222222222222222" },
  "StoreURL": "elegant-styles-salon-jaipur",
  "StoreAddress": "5th Avenue, C-Scheme, Jaipur, Rajasthan 302001, India",
  "StoreContact": "9876543210",
  "StoreCoords": {
    "longitude": 75.789876,
    "latitude": 26.912434
  },
  "GoogleMapsURL": "https://www.google.com/maps/?q=26.912434,75.789876",
  "About": "Premium unisex salon for hair, skin, and makeup.",
  "TermsAndConditions": "Service availability may vary. Prior booking is recommended.",
  "Description": "Elegant Styles Salon in Jaipur offers professional services in haircuts, styling, makeup, and skincare. Perfect for both casual and bridal makeovers.",
  "GST_Number": "08ABCDE1234F1Z5",
  "Commission_status_enable": true,
  "Orders": [
    { "$oid": "672f0fa146789abcdef00001" },
    { "$oid": "672f0fa146789abcdef00002" },
    { "$oid": "672f0fa146789abcdef00003" },
    { "$oid": "672f0fa146789abcdef00004" },
    { "$oid": "672f0fa146789abcdef00005" }
  ],
  "Business_Hours": [
    { "day": "Monday", "openTime": "10:00", "closeTime": "21:00", "closed": false },
    { "day": "Tuesday", "openTime": "10:00", "closeTime": "21:00", "closed": false },
    { "day": "Wednesday", "openTime": "10:00", "closeTime": "21:00", "closed": false },
    { "day": "Thursday", "openTime": "10:00", "closeTime": "21:00", "closed": false },
    { "day": "Friday", "openTime": "10:00", "closeTime": "21:00", "closed": false },
    { "day": "Saturday", "openTime": "10:00", "closeTime": "21:00", "closed": false },
    { "day": "Sunday", "openTime": "", "closeTime": "", "closed": true }
  ],
  "Social_handles": [
    {
      "platform": "Instagram",
      "username": "elegantstylesjaipur",
      "profileLink": "https://instagram.com/elegantstylesjaipur"
    }
  ],
  "Photos": [],
  "Menus": [],
  "Products": [],
  "Offers": [],
  "Events": [],
  "Bookings": [],
  "members": [],
  "impressions": 0,
  "logourl": "https://yourdomain.com/assets/store-logo.png",
  "ownerAdhaarCardurl": "https://yourdomain.com/documents/aadhar.jpg",
  "electricityBillurl": "https://yourdomain.com/documents/electricity-bill.jpg",
  "gstCertificateurl": "https://yourdomain.com/documents/gst-certificate.jpg",
  "signBoardurl": "https://yourdomain.com/images/signboard.jpg",
  "interiorimageurl": [],
  "thumbnailurl": "https://yourdomain.com/images/thumbnail.jpg",
  "__v": 0
}
```

---

### 🧾 Orders Info Used for Testing

```json
[
  {
    "_id": { "$oid": "672f0fa146789abcdef00001" },
    "StoreId": { "$oid": "672f0f9e1234567890abcdef" },
    "OrderNumber": "ORD001",
    "TransactionID": null,
    "CouponUsed": null,
    "wasPaymentSuccessful": true,
    "AmountExcludingOffers": 499,
    "AppliedOffers": [],
    "ActualTransactionAmount": 499,
    "Address": "Sample Address",
    "Status": "delivered",
    "items": [
      {
        "productID": { "$oid": "000000000000000000000001" },
        "variants": []
      }
    ],
    "createdAt": { "$date": "2025-04-01T11:00:00.000Z" },
    "updatedAt": { "$date": "2025-04-01T11:00:00.000Z" }
  },
  {
    "_id": { "$oid": "672f0fa146789abcdef00002" },
    "StoreId": { "$oid": "672f0f9e1234567890abcdef" },
    "OrderNumber": "ORD002",
    "TransactionID": null,
    "CouponUsed": null,
    "wasPaymentSuccessful": true,
    "AmountExcludingOffers": 3999,
    "AppliedOffers": [],
    "ActualTransactionAmount": 3999,
    "Address": "Sample Address",
    "Status": "pending",
    "items": [
      {
        "productID": { "$oid": "000000000000000000000002" },
        "variants": []
      }
    ],
    "createdAt": { "$date": "2025-04-02T15:30:00.000Z" },
    "updatedAt": { "$date": "2025-04-02T15:30:00.000Z" }
  },
  {
    "_id": { "$oid": "672f0fa146789abcdef00003" },
    "StoreId": { "$oid": "672f0f9e1234567890abcdef" },
    "OrderNumber": "ORD003",
    "TransactionID": null,
    "CouponUsed": null,
    "wasPaymentSuccessful": false,
    "AmountExcludingOffers": 799,
    "AppliedOffers": [],
    "ActualTransactionAmount": 799,
    "Address": "Sample Address",
    "Status": "processing",
    "items": [
      {
        "productID": { "$oid": "000000000000000000000003" },
        "variants": []
      }
    ],
    "createdAt": { "$date": "2025-04-03T13:20:00.000Z" },
    "updatedAt": { "$date": "2025-04-03T13:20:00.000Z" }
  },
  {
    "_id": { "$oid": "672f0fa146789abcdef00004" },
    "StoreId": { "$oid": "672f0f9e1234567890abcdef" },
    "OrderNumber": "ORD004",
    "TransactionID": null,
    "CouponUsed": null,
    "wasPaymentSuccessful": false,
    "AmountExcludingOffers": 999,
    "AppliedOffers": [],
    "ActualTransactionAmount": 999,
    "Address": "Sample Address",
    "Status": "cancelled",
    "items": [
      {
        "productID": { "$oid": "000000000000000000000004" },
        "variants": []
      }
    ],
    "createdAt": { "$date": "2025-04-03T16:45:00.000Z" },
    "updatedAt": { "$date": "2025-04-03T16:45:00.000Z" }
  },
  {
    "_id": { "$oid": "672f0fa146789abcdef00005" },
    "StoreId": { "$oid": "672f0f9e1234567890abcdef" },
    "OrderNumber": "ORD005",
    "TransactionID": null,
    "CouponUsed": null,
    "wasPaymentSuccessful": true,
    "AmountExcludingOffers": 649,
    "AppliedOffers": [],
    "ActualTransactionAmount": 649,
    "Address": "Sample Address",
    "Status": "out for delivery",
    "items": [
      {
        "productID": { "$oid": "000000000000000000000005" },
        "variants": []
      }
    ],
    "createdAt": { "$date": "2025-04-04T10:15:00.000Z" },
    "updatedAt": { "$date": "2025-04-04T10:15:00.000Z" }
  },
  {
    "_id": { "$oid": "672f0fa146789abcdef00006" },
    "StoreId": { "$oid": "672f0f9e1234567890abcdef" },
    "OrderNumber": "ORD006",
    "TransactionID": null,
    "CouponUsed": null,
    "wasPaymentSuccessful": true,
    "AmountExcludingOffers": 799,
    "AppliedOffers": [],
    "ActualTransactionAmount": 799,
    "Address": "Sample Address",
    "Status": "processing",
    "items": [
      {
        "productID": { "$oid": "000000000000000000000006" },
        "variants": []
      }
    ],
    "createdAt": { "$date": "2025-04-03T13:20:00.000Z" },
    "updatedAt": { "$date": "2025-04-03T13:20:00.000Z" }
  },
  {
    "_id": { "$oid": "672f0fa146789abcdef00007" },
    "StoreId": { "$oid": "672f0f9e1234567890abcdef" },
    "OrderNumber": "ORD007",
    "TransactionID": null,
    "CouponUsed": null,
    "wasPaymentSuccessful": true,
    "AmountExcludingOffers": 999,
    "AppliedOffers": [],
    "ActualTransactionAmount": 999,
    "Address": "Sample Address",
    "Status": "cancelled",
    "items": [
      {
        "productID": { "$oid": "000000000000000000000007" },
        "variants": []
      }
    ],
    "createdAt": { "$date": "2025-04-03T16:45:00.000Z" },
    "updatedAt": { "$date": "2025-04-03T16:45:00.000Z" }
  }
]
```

---

## 🔐 Authentication

- **Bearer Token (for testing):**
  ```
  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjY3MmYwZjFiMjIyMjIyMjIyMjIyMjIyMiIsImV4cCI6MTg5MzQ1NjAwMH0.DPSgLl8HoYiDSg0lXIFlO2bcGI7yPFno_dHdWwAnvho
  ```

---

## 🧾 API Request Body

Use the following JSON in the request body (raw):

```json
{
  "storeId": "672f0f9e1234567890abcdef"
}
```

---

## 🔄 Routes

### ✅ Get Ongoing Orders

- **Route:** `/retailerorder/ongoing-orders`
- **Method:** POST
- **Response:**

```json
{
  "success": true,
  "orders": [
    {
      "_id": "672f0fa146789abcdef00002",
      "StoreId": "672f0f9e1234567890abcdef",
      "OrderNumber": "ORD002",
      "TransactionID": null,
      "CouponUsed": null,
      "wasPaymentSuccessful": true,
      "AmountExcludingOffers": 3999,
      "AppliedOffers": [],
      "ActualTransactionAmount": 3999,
      "Address": "Sample Address",
      "Status": "pending",
      "items": [
        {
          "_id": "67f2c3d64460335e988e0546",
          "productID": "000000000000000000000002",
          "variants": []
        }
      ],
      "createdAt": "2025-04-02T15:30:00.000Z",
      "updatedAt": "2025-04-02T15:30:00.000Z"
    },
    {
      "_id": "672f0fa146789abcdef00005",
      "StoreId": "672f0f9e1234567890abcdef",
      "OrderNumber": "ORD005",
      "TransactionID": null,
      "CouponUsed": null,
      "wasPaymentSuccessful": true,
      "AmountExcludingOffers": 649,
      "AppliedOffers": [],
      "ActualTransactionAmount": 649,
      "Address": "Sample Address",
      "Status": "out for delivery",
      "items": [
        {
          "_id": "67f2c3d64460335e988e0548",
          "productID": "000000000000000000000005",
          "variants": []
        }
      ],
      "createdAt": "2025-04-04T10:15:00.000Z",
      "updatedAt": "2025-04-04T10:15:00.000Z"
    },
    {
      "_id": "672f0fa146789abcdef00006",
      "StoreId": "672f0f9e1234567890abcdef",
      "OrderNumber": "ORD006",
      "TransactionID": null,
      "CouponUsed": null,
      "wasPaymentSuccessful": true,
      "AmountExcludingOffers": 799,
      "AppliedOffers": [],
      "ActualTransactionAmount": 799,
      "Address": "Sample Address",
      "Status": "processing",
      "items": [
        {
          "_id": "67f2c3d64460335e988e0549",
          "productID": "000000000000000000000006",
          "variants": []
        }
      ],
      "createdAt": "2025-04-03T13:20:00.000Z",
      "updatedAt": "2025-04-03T13:20:00.000Z"
    }
  ]
}
```

---

### ✅ Get Completed Orders

- **Route:** `/retailerorder/completed-orders`
- **Method:** POST
- **Response:**

```json
{
  "success": true,
  "orders": [
    {
      "_id": "672f0fa146789abcdef00001",
      "StoreId": "672f0f9e1234567890abcdef",
      "OrderNumber": "ORD001",
      "TransactionID": null,
      "CouponUsed": null,
      "wasPaymentSuccessful": true,
      "AmountExcludingOffers": 499,
      "AppliedOffers": [],
      "ActualTransactionAmount": 499,
      "Address": "Sample Address",
      "Status": "delivered",
      "items": [
        {
          "_id": "67f2c39b4460335e988e0539",
          "productID": "000000000000000000000001",
          "variants": []
        }
      ],
      "createdAt": "2025-04-01T11:00:00.000Z",
      "updatedAt": "2025-04-01T11:00:00.000Z"
    },
    {
      "_id": "672f0fa146789abcdef00007",
      "StoreId": "672f0f9e1234567890abcdef",
      "OrderNumber": "ORD007",
      "TransactionID": null,
      "CouponUsed": null,
      "wasPaymentSuccessful": true,
      "AmountExcludingOffers": 999,
      "AppliedOffers": [],
      "ActualTransactionAmount": 999,
      "Address": "Sample Address",
      "Status": "cancelled",
      "items": [
        {
          "_id": "67f2c39b4460335e988e053b",
          "productID": "000000000000000000000007",
          "variants": []
        }
      ],
      "createdAt": "2025-04-03T16:45:00.000Z",
      "updatedAt": "2025-04-03T16:45:00.000Z"
    }
  ]
}
```

---

### ✅ Get All Orders

- **Route:** `/retailerorder/order-list`
- **Method:** POST
- **Response:**

```json
{
  "success": true,
  "orders": [
    {
      "_id": "672f0fa146789abcdef00001",
      "StoreId": "672f0f9e1234567890abcdef",
      "OrderNumber": "ORD001",
      "TransactionID": null,
      "CouponUsed": null,
      "wasPaymentSuccessful": true,
      "AmountExcludingOffers": 499,
      "AppliedOffers": [],
      "ActualTransactionAmount": 499,
      "Address": "Sample Address",
      "Status": "delivered",
      "items": [
        {
          "_id": "67f2c33a4460335e988e0528",
          "productID": "000000000000000000000001",
          "variants": []
        }
      ],
      "createdAt": "2025-04-01T11:00:00.000Z",
      "updatedAt": "2025-04-01T11:00:00.000Z"
    },
    {
      "_id": "672f0fa146789abcdef00002",
      "StoreId": "672f0f9e1234567890abcdef",
      "OrderNumber": "ORD002",
      "TransactionID": null,
      "CouponUsed": null,
      "wasPaymentSuccessful": true,
      "AmountExcludingOffers": 3999,
      "AppliedOffers": [],
      "ActualTransactionAmount": 3999,
      "Address": "Sample Address",
      "Status": "pending",
      "items": [
        {
          "_id": "67f2c33a4460335e988e0529",
          "productID": "000000000000000000000002",
          "variants": []
        }
      ],
      "createdAt": "2025-04-02T15:30:00.000Z",
      "updatedAt": "2025-04-02T15:30:00.000Z"
    },
    {
      "_id": "672f0fa146789abcdef00005",
      "StoreId": "672f0f9e1234567890abcdef",
      "OrderNumber": "ORD005",
      "TransactionID": null,
      "CouponUsed": null,
      "wasPaymentSuccessful": true,
      "AmountExcludingOffers": 649,
      "AppliedOffers": [],
      "ActualTransactionAmount": 649,
      "Address": "Sample Address",
      "Status": "out for delivery",
      "items": [
        {
          "_id": "67f2c33a4460335e988e052c",
          "productID": "000000000000000000000005",
          "variants": []
        }
      ],
      "createdAt": "2025-04-04T10:15:00.000Z",
      "updatedAt": "2025-04-04T10:15:00.000Z"
    },
    {
      "_id": "672f0fa146789abcdef00006",
      "StoreId": "672f0f9e1234567890abcdef",
      "OrderNumber": "ORD006",
      "TransactionID": null,
      "CouponUsed": null,
      "wasPaymentSuccessful": true,
      "AmountExcludingOffers": 799,
      "AppliedOffers": [],
      "ActualTransactionAmount": 799,
      "Address": "Sample Address",
      "Status": "processing",
      "items": [
        {
          "_id": "67f2c33a4460335e988e052d",
          "productID": "000000000000000000000006",
          "variants": []
        }
      ],
      "createdAt": "2025-04-03T13:20:00.000Z",
      "updatedAt": "2025-04-03T13:20:00.000Z"
    },
    {
      "_id": "672f0fa146789abcdef00007",
      "StoreId": "672f0f9e1234567890abcdef",
      "OrderNumber": "ORD007",
      "TransactionID": null,
      "CouponUsed": null,
      "wasPaymentSuccessful": true,
      "AmountExcludingOffers": 999,
      "AppliedOffers": [],
      "ActualTransactionAmount": 999,
      "Address": "Sample Address",
      "Status": "cancelled",
      "items": [
        {
          "_id": "67f2c33a4460335e988e052e",
          "productID": "000000000000000000000007",
          "variants": []
        }
      ],
      "createdAt": "2025-04-03T16:45:00.000Z",
      "updatedAt": "2025-04-03T16:45:00.000Z"
    }
  ]
}
```

---