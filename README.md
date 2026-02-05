# Chat with a Database Using AI

![n8n](https://img.shields.io/badge/n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2CA5E0?style=for-the-badge&logo=docker&logoColor=white)
![Python](https://img.shields.io/badge/Python-14354C?style=for-the-badge&logo=python&logoColor=white)
![Groq](https://img.shields.io/badge/Groq-FF6F00?style=for-the-badge&logo=ai&logoColor=white)

An intelligent AI-powered chatbot that allows you to query your PostgreSQL database using natural language. Built with n8n workflow automation, Groq AI, and PostgreSQL.

## 🎯 What It Does

Ask questions in plain English and get instant answers from your database:

- **"Which tables are available?"**
- **"What are the top 5 best-selling products?"**
- **"Show me sales from 2010"**
- **"How many customers are from the UK?"**

No SQL knowledge required!

## 🎬 Demo

**Sample Conversation:**
```
User: Which tables are available?

AI: I found 2 tables in your database:
    1. year_2009_2010 - Sales data from 2009-2010
    2. year_2010_2011 - Sales data from 2010-2011

User: Show me the top 5 best-selling products

AI: Here are the top 5 products across both years:
    1. PACK OF 60 PINK WRENCHES (Total: 4,560 units)
    2. PACK OF 66 WRENCHES (Total: 3,240 units)
    3. PACK OF 48 TANKARD (Total: 2,880 units)
    4. PACK OF 60 TANKARD (Total: 2,520 units)
    5. REGENCY CAKESTAND 3 TIER (Total: 2,200 units)
```

## 🏗️ Architecture
```
┌──────────────────┐
│  User (Chat UI)  │
└────────┬─────────┘
         │
         ▼
┌─────────────────────────────────┐
│      n8n Workflow               │
│                                 │
│  ┌───────────────────────────┐ │
│  │ Chat Trigger              │ │
│  └───────────┬───────────────┘ │
│              │                  │
│              ▼                  │
│  ┌───────────────────────────┐ │
│  │      AI Agent             │ │
│  │  ┌─────────────────────┐ │ │
│  │  │ Groq Chat Model     │ │ │
│  │  │ (llama-3.1-70b)     │ │ │
│  │  └─────────────────────┘ │ │
│  │  ┌─────────────────────┐ │ │
│  │  │ Window Buffer       │ │ │
│  │  │ Memory              │ │ │
│  │  └─────────────────────┘ │ │
│  │  ┌─────────────────────┐ │ │
│  │  │ PostgreSQL Tool     │ │ │
│  │  │ (executeQuery)      │ │ │
│  │  └─────────────────────┘ │ │
│  └───────────────────────────┘ │
└─────────────────────────────────┘
         │
         ▼
┌──────────────────┐
│  PostgreSQL DB   │
│  • year_2009_    │
│    2010          │
│  • year_2010_    │
│    2011          │
└──────────────────┘
```

## 📋 Prerequisites

- **Docker Desktop** - [Download here](https://www.docker.com/products/docker-desktop)
- **Python 3.8+** - [Download here](https://www.python.org/downloads/)
- **Groq API Key** - [Get free key here](https://console.groq.com)

## 🚀 Quick Start

### Step 1: Set Up Docker Containers

#### 1.1 Run n8n
```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

Access n8n at: **http://localhost:5678**

#### 1.2 Run PostgreSQL
```bash
docker run --name postgres \
  -e POSTGRES_PASSWORD=anamika123 \
  -e POSTGRES_USER=n8n \
  -e POSTGRES_DB=n8n \
  -p 5432:5432 \
  -d postgres
```

### Step 2: Load Data into PostgreSQL

#### 2.1 Install Python Dependencies
```bash
pip install pandas sqlalchemy psycopg2-binary openpyxl
```

#### 2.2 Create and Run Data Import Script

Create a file named `load_data.py`:
```python
import pandas as pd
from sqlalchemy import create_engine

# Create database connection
engine = create_engine('postgresql://n8n:anamika123@localhost:5432/n8n')

# Read Excel file with all sheets
excel_file = pd.ExcelFile('online_retail_II.xlsx')

# Upload each sheet to a separate table
for sheet_name in excel_file.sheet_names:
    df = pd.read_excel(excel_file, sheet_name=sheet_name)
    
    # Create table name from sheet name (clean it up)
    table_name = sheet_name.lower().replace(' ', '_').replace('-', '_')
    
    # Upload to PostgreSQL
    df.to_sql(table_name, engine, if_exists='replace', index=False)
    print(f"Uploaded {sheet_name} to table {table_name}")

print("\nAll sheets uploaded successfully!")
```

Run the script:
```bash
python load_data.py
```

**Expected Output:**
```
Uploaded Year 2009-2010 to table year_2009_2010
Uploaded Year 2010-2011 to table year_2010_2011

All sheets uploaded successfully!
```

### Step 3: Configure n8n Workflow

#### 3.1 Create Workflow

1. Open **http://localhost:5678** in your browser
2. Click **"Add Workflow"**
3. Add the following nodes:

**Node Structure:**
```
When chat message received → AI Agent
                              ├── Groq Chat Model
                              ├── Window Buffer Memory
                              └── Postgres (Tool)
```

#### 3.2 Get Groq API Key

1. Go to https://console.groq.com/keys
2. Click **"Create API Key"**
3. Copy the key (it won't be shown again!)

#### 3.3 Configure Groq Chat Model

1. Click on the **Groq Chat Model** node
2. Click **"Credential to connect with"**
3. Click **"Create New Credential"**
4. Paste your Groq API key
5. Click **"Save"**
6. Select Model: **llama-3.1-70b-versatile**
7. Add option → **Temperature**: `0.3`

#### 3.4 Configure PostgreSQL Connection

1. Click on the **Postgres** tool node
2. Click **"Credential to connect with"**
3. Click **"Create New Credential"**
4. Enter connection details:
```
Host: host.docker.internal
Database: n8n
User: n8n
Password: anamika123
Port: 5432
SSL: Disable
```

5. Click **"Test"** - Should show green ✓
6. Click **"Save"**

#### 3.5 Configure AI Agent System Prompt

Click on **AI Agent** node and add this system message:
```
You are a helpful database assistant with access to a PostgreSQL database containing retail sales data.

DATABASE SCHEMA:
You have access to these tables:
1. year_2009_2010 - Sales data from 2009-2010
2. year_2010_2011 - Sales data from 2010-2011

Each table contains these columns:
- Invoice (text) - Invoice number
- StockCode (text) - Product code
- Description (text) - Product description
- Quantity (bigint) - Quantity sold
- InvoiceDate (timestamp) - Date of sale
- Price (double precision) - Unit price
- "Customer ID" (double precision) - Customer identifier
- Country (text) - Customer country

IMPORTANT RULES:
1. Column names are case-sensitive and MUST be in double quotes: "Description", "Quantity", "InvoiceDate"
2. When asked about tables or data, use the postgres tool to query the database
3. Always provide clear, formatted responses
4. If a query fails, explain the error simply

EXAMPLE QUERIES:
- List tables: SELECT table_name FROM information_schema.tables WHERE table_schema = 'public';
- Get schema: SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'year_2009_2010';
- Top products: SELECT "Description", SUM("Quantity") as total FROM year_2009_2010 GROUP BY "Description" ORDER BY total DESC LIMIT 5;

Always be helpful and provide accurate data from the database.
```

#### 3.6 Configure Tool Description

In the **Postgres** tool node, set the tool description to:
```
Execute SQL queries on the PostgreSQL database. Use this to retrieve information about tables, schemas, and data. Remember to use double quotes around column names like "Description", "Quantity", "InvoiceDate".
```

#### 3.7 Set Max Iterations

In the **AI Agent** node:
1. Click **"Add option"**
2. Select **"Max Iterations"**
3. Set to: `15`

### Step 4: Test the Workflow

1. Click **"Save"** to save your workflow
2. Click **"Open chat"** button at the bottom
3. Try these test questions:

**Test 1:**
```
Which tables are available?
```

**Test 2:**
```
What columns does the year_2009_2010 table have?
```

**Test 3:**
```
Show me the top 5 best-selling products
```

**Test 4:**
```
How many total sales are in the database?
```

## 📊 Database Schema

### Table: year_2009_2010 & year_2010_2011

| Column Name | Data Type | Description |
|------------|-----------|-------------|
| Invoice | text | Invoice number |
| StockCode | text | Product stock code |
| Description | text | Product name/description |
| Quantity | bigint | Number of items sold |
| InvoiceDate | timestamp | Date and time of sale |
| Price | double precision | Unit price |
| Customer ID | double precision | Customer identifier |
| Country | text | Customer's country |

## 🔧 Troubleshooting

### Issue: "Agent stopped due to max iterations"

**Solution:**
- Increase Max Iterations in AI Agent node to 20-30
- Simplify the system prompt
- Make sure tool description is clear

### Issue: "Column does not exist" error

**Solution:**
- Remember to use double quotes around column names
- Example: `"Description"` not `description`
- Update system prompt to emphasize this rule

### Issue: Can't connect to PostgreSQL

**Solution:**
- Check if PostgreSQL container is running: `docker ps`
- Try using `localhost` instead of `host.docker.internal`
- Verify port 5432 is not being used by another service

### Issue: Groq API errors

**Solution:**
- Verify API key is correct
- Check if you've exceeded rate limits (free tier)
- Try a different model like `mixtral-8x7b-32768`

## 📝 Example Queries

Here are some questions you can ask:

**Basic Queries:**
- "Which tables are available?"
- "Show me the schema of year_2009_2010"
- "How many rows are in each table?"

**Analysis Queries:**
- "What are the top 10 best-selling products?"
- "Which countries have the most customers?"
- "Show me total sales by month in 2010"
- "What's the average order value?"

**Complex Queries:**
- "Compare sales between 2009 and 2010"
- "Which products had decreasing sales over time?"
- "Show me the top 5 customers by revenue"

## 🛠️ Advanced Configuration

### Add More Memory

For longer conversations, increase memory window:

1. Click on **Window Buffer Memory** node
2. Set **Context Window Size**: `20`

### Use Different AI Models

Try these Groq models:
- `llama-3.1-70b-versatile` - Best for complex reasoning
- `mixtral-8x7b-32768` - Fast with large context
- `llama-3.1-8b-instant` - Fastest, good for simple queries

### Add Error Handling

In AI Agent node:
1. Click **"Add option"**
2. Select **"On Error"**
3. Choose **"Continue"**

## 📂 Project Structure
```
Chat-with-a-database-using-AI/
├── README.md
├── load_data.py           # Python script to load data
├── online_retail_II.xlsx  # Sample dataset (not included)
├── workflow.json          # n8n workflow export (optional)
└── screenshots/           # Demo screenshots
    ├── workflow.png
    ├── chat-demo.png
    └── results.png
```

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- [n8n](https://n8n.io/) - Workflow automation platform
- [Groq](https://groq.com/) - Fast AI inference
- [PostgreSQL](https://www.postgresql.org/) - Powerful open-source database
- [Online Retail II Dataset](https://archive.ics.uci.edu/ml/datasets/Online+Retail+II) - Sample dataset

## 📧 Contact

For questions or support, please open an issue in this repository.

---

**⭐ If you found this project helpful, please give it a star!**
