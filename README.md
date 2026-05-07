# Python REST API with SQL Backend

## Introduction
This project is a Python-based REST API backend built using Flask and SQLAlchemy. It provides structured endpoints for data management, automated validation workflows, and secure backend operations. The system demonstrates backend architecture design, relational database integration, and data processing automation using Python.

---

## Technical Implementation
- Built using **Flask** for REST API development  
- **SQLAlchemy ORM** used for relational database integration  
- Normalized SQL schema for efficient data storage and retrieval  
- **Pandas** used for automated data validation and anomaly detection  
- Modular architecture separating routes, services, and database layers  
- Git used for version control and structured development workflow  

---

## Key Features
- RESTful API using Flask  
- SQL database integration using SQLAlchemy  
- Automated data validation using Pandas  
- Modular and scalable backend structure  
- JSON-based request/response system  
- Error handling and input validation  
- Support for CRUD operations  
- Clean and maintainable project architecture  

---

## Project Structure
```bash
python-rest-api/
│
├── app.py
├── config.py
├── requirements.txt
│
├── database/
│   └── models.py
│
├── routes/
│   └── api_routes.py
│
├── services/
│   └── validation_service.py
│
└── utils/
    └── helpers.py
```

## 💻 How To Run

1. **Clone the Repository**
```bash
git clone https://github.com/yourusername/python-rest-api.git
cd python-rest-api
```
2.  **Create Virtual Environment**
```bash
    python3 -m venv venv
    source venv/bin/activate
```
3.  **Install Dependencies**
```bash 
    * pip install -r requirements.txt
```
4.  **Run**
```bash 
    python app.py
```
5.  **Access the Application
```bash 
http://127.0.0.1:5000
```
---

## API Endpoints
Get All Records
```bash 
GET /records
```
Add New Record
```bash 
Post /records
```
Request Body
```bash 
{
  "name": "John Doe",
  "email": "john@example.com",
  "score": 85
}
```
Validate Data
```bash
GET /validate
```

## Source Code

### app.py
```py
from flask import Flask
from routes.api_routes import api
from database.models import db

app = Flask(__name__)

app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database/database.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db.init_app(app)

with app.app_context():
    db.create_all()

app.register_blueprint(api)

if __name__ == "__main__":
    app.run(debug=True)
```

### database.py
```py
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class Record(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    email = db.Column(db.String(100), nullable=False)
    score = db.Column(db.Integer, nullable=False)

    def to_dict(self):
        return {
            "id": self.id,
            "name": self.name,
            "email": self.email,
            "score": self.score
        }
```
### routes.py
```py
from flask import Blueprint, request, jsonify
from database.models import db, Record
from services.validation_service import validate_data

api = Blueprint('api', __name__)

@api.route('/records', methods=['GET'])
def get_records():
    records = Record.query.all()
    return jsonify([r.to_dict() for r in records])

@api.route('/records', methods=['POST'])
def add_record():
    data = request.get_json()

    new_record = Record(
        name=data['name'],
        email=data['email'],
        score=data['score']
    )

    db.session.add(new_record)
    db.session.commit()

    return jsonify({"message": "Record added successfully"}), 201

@api.route('/validate', methods=['GET'])
def validate():
    return jsonify(validate_data())
```
### services.py
```py
import pandas as pd
from database.models import Record

def validate_data():
    records = Record.query.all()

    data = [{
        "id": r.id,
        "name": r.name,
        "email": r.email,
        "score": r.score
    } for r in records]

    df = pd.DataFrame(data)

    return {
        "missing_values": df.isnull().sum().to_dict(),
        "duplicate_records": int(df.duplicated().sum()),
        "invalid_scores": int((df["score"] < 0).sum())
    }
```


