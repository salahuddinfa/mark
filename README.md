# Auto-Receipts: Automated Receipt Processing System

## Overview

This project is a web application designed to automate the processing of scanned receipts. It leverages OCR/AI to extract key details from PDF receipts and stores the structured data in a SQLite database. The system provides REST APIs for managing and retrieving receipt information, making it a comprehensive solution for digital receipt management.

## Features

- **Multi-format Receipt Uploads**: Upload scanned receipts in PDF, PNG, JPG, and JPEG formats
- **File Validation**: Ensures that all uploaded files are valid receipts before processing
- **AI-Powered Data Extraction**: Uses OpenAI's GPT models to extract key information such as:
  - Merchant name
  - Purchase date and time
  - Total amount and currency
  - Payment method
  - Category classification
  - Line items with descriptions, quantities, and prices
- **Structured Database Storage**: Stores extracted data in an organized SQLite database
- **RESTful API**: Complete set of APIs for seamless receipt management and data retrieval
- **Duplicate Handling**: Updates existing records instead of creating duplicates

## Database Schema

The application uses a SQLite database (`receipts.db`) with the following tables:

### `receipt_file` (ReceiptMetaData)

Stores metadata for each uploaded receipt file.

| Column | Description |
|--------|-------------|
| `id` | Unique identifier for the uploaded file |
| `file_name` | Name of the uploaded file |
| `file_path` | Storage path of the uploaded file |
| `is_valid` | Indicates if the file is a valid receipt |
| `invalid_reason` | Reason for the file being invalid (if applicable) |
| `is_processed` | Indicates if the file has been processed |
| `created_at` | Timestamp of when the receipt was first uploaded |
| `updated_at` | Timestamp of the last modification |

### `receipt`

Stores the extracted information from valid receipt files.

| Column | Description |
|--------|-------------|
| `id` | Unique identifier for the extracted receipt |
| `purchased_at` | Date and time of purchase from the receipt |
| `merchant_name` | Merchant's name as extracted from the receipt |
| `total_amount` | Total amount spent as shown on the receipt |
| `currency` | Currency code (e.g., USD, EUR) |
| `payment_method` | Payment method used (e.g., Credit Card, Cash) |
| `category` | Receipt category (e.g., Food, Transportation) |
| `receipt_file` | Foreign key to the associated receipt file |
| `created_at` | Timestamp of when the receipt was processed |
| `updated_at` | Timestamp of the last modification |

### `line_item`

Stores individual line items from receipts.

| Column | Description |
|--------|-------------|
| `id` | Unique identifier for the line item |
| `description` | Description of the item |
| `quantity` | Quantity purchased |
| `unit_price` | Price per unit |
| `total` | Total price for this line item |
| `receipt` | Foreign key to the parent receipt |

## API Endpoints

### 1. Upload Receipt
**POST** `/upload`

Uploads a new receipt file and stores its metadata.

**Request:**
```bash
curl -X POST -F "file=@/path/to/your/receipt.pdf" http://127.0.0.1:8000/upload
```

**Response:**
```json
{
    "id": 1,
    "file_name": "receipt.pdf",
    "file_path": "uploads/file-1_receipt.pdf",
    "is_valid": false,
    "invalid_reason": null,
    "is_processed": false,
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T10:30:00Z"
}
```

### 2. Validate Receipt
**GET** `/validate/{receipt_id}`

Validates an uploaded file to confirm it is a valid receipt.

**Request:**
```bash
curl -X GET http://127.0.0.1:8000/validate/1
```

**Response:**
```json
{
    "id": 1,
    "file_name": "receipt.pdf",
    "file_path": "uploads/file-1_receipt.pdf",
    "is_valid": true,
    "invalid_reason": "",
    "is_processed": false,
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T10:30:00Z"
}
```

### 3. Process Receipt
**GET** `/process/{receipt_id}`

Extracts receipt details using AI and saves the data.

**Request:**
```bash
curl -X GET http://127.0.0.1:8000/process/1
```

**Response:**
```json
{
    "id": 1,
    "purchased_at": "2024-01-15T10:30:00Z",
    "merchant_name": "Walmart",
    "total_amount": "156.78",
    "currency": "USD",
    "payment_method": "Credit Card",
    "category": "Shopping",
    "receipt_file": 1,
    "created_at": "2024-01-15T10:35:00Z",
    "updated_at": "2024-01-15T10:35:00Z",
    "line_items": [
        {
            "id": 1,
            "description": "Milk",
            "quantity": "2.00",
            "unit_price": "3.99",
            "total": "7.98"
        },
        {
            "id": 2,
            "description": "Bread",
            "quantity": "1.00",
            "unit_price": "2.49",
            "total": "2.49"
        }
    ]
}
```

### 4. List All Receipts
**GET** `/receipts`

Retrieves a list of all receipts stored in the database.

**Request:**
```bash
curl -X GET http://127.0.0.1:8000/receipts
```

**Response:**
```json
[
    {
        "id": 1,
        "purchased_at": "2024-01-15T10:30:00Z",
        "merchant_name": "Walmart",
        "total_amount": "156.78",
        "currency": "USD",
        "payment_method": "Credit Card",
        "category": "Shopping",
        "receipt_file": 1,
        "created_at": "2024-01-15T10:35:00Z",
        "updated_at": "2024-01-15T10:35:00Z",
        "line_items": [...]
    }
]
```

### 5. Get Receipt Details
**GET** `/receipts/{id}`

Fetches the details of a specific receipt by its ID.

**Request:**
```bash
curl -X GET http://127.0.0.1:8000/receipts/1
```

**Response:**
```json
{
    "id": 1,
    "purchased_at": "2024-01-15T10:30:00Z",
    "merchant_name": "Walmart",
    "total_amount": "156.78",
    "currency": "USD",
    "payment_method": "Credit Card",
    "category": "Shopping",
    "receipt_file": 1,
    "created_at": "2024-01-15T10:35:00Z",
    "updated_at": "2024-01-15T10:35:00Z",
    "line_items": [...]
}
```

## Getting Started

### Prerequisites

- Python 3.8 or higher
- OpenAI API key (for AI-powered extraction)

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/auto-receipts.git
   cd auto-receipts
   ```

2. **Create and activate a virtual environment:**
   ```bash
   python -m venv venv
   # On Windows
   venv\Scripts\activate
   # On macOS/Linux
   source venv/bin/activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Set up environment variables:**
   Create a `.env` file in the project root:
   ```env
   OPENAI_API_KEY=your_openai_api_key_here
   SECRET_KEY=your_django_secret_key_here
   DEBUG=True
   ```

5. **Run database migrations:**
   ```bash
   python manage.py migrate
   ```

6. **Start the development server:**
   ```bash
   python manage.py runserver
   ```

The application will be available at `http://127.0.0.1:8000/`

## Dependencies

- **Django**: Web framework
- **Django REST Framework**: API framework
- **OpenAI**: AI-powered text extraction
- **PyPDFium2**: PDF processing
- **Pillow**: Image processing
- **python-dotenv**: Environment variable management

## Project Structure

```
auto-receipts/
├── manage.py                 # Django management script
├── requirements.txt          # Python dependencies
├── receipts.db              # SQLite database
├── uploads/                 # Directory for uploaded files
├── receipt_project/         # Django project settings
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── receipts/               # Main application
    ├── models.py           # Database models
    ├── views.py            # API views
    ├── serializers.py      # Data serializers
    ├── utils.py            # Utility functions
    ├── urls.py             # URL routing
    └── migrations/         # Database migrations
```

## Usage Examples

### Using curl

1. **Upload a receipt:**
   ```bash
   curl -X POST -F "file=@receipt.pdf" http://127.0.0.1:8000/upload
   ```

2. **Validate the uploaded receipt:**
   ```bash
   curl -X GET http://127.0.0.1:8000/validate/1
   ```

3. **Process the receipt to extract data:**
   ```bash
   curl -X GET http://127.0.0.1:8000/process/1
   ```

4. **List all receipts:**
   ```bash
   curl -X GET http://127.0.0.1:8000/receipts
   ```

### Using Python requests

```python
import requests

# Upload a receipt
with open('receipt.pdf', 'rb') as f:
    files = {'file': f}
    response = requests.post('http://127.0.0.1:8000/upload', files=files)
    receipt_id = response.json()['id']

# Validate the receipt
response = requests.get(f'http://127.0.0.1:8000/validate/{receipt_id}')

# Process the receipt
response = requests.get(f'http://127.0.0.1:8000/process/{receipt_id}')
receipt_data = response.json()
print(f"Merchant: {receipt_data['merchant_name']}")
print(f"Total: {receipt_data['total_amount']} {receipt_data['currency']}")
```

## Error Handling

The API provides comprehensive error handling:

- **400 Bad Request**: Invalid file format or processing errors
- **404 Not Found**: Receipt or file not found
- **500 Internal Server Error**: Server-side processing errors

Error responses include descriptive messages:
```json
{
    "error": "Not a Valid format - Supported Formats: ['.png', '.pdf', '.jpg', '.jpeg']"
}
```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

If you encounter any issues or have questions, please open an issue on GitHub or contact the development team.
