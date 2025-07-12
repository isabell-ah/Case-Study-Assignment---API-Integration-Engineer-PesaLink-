# Database Normalization - Part 1

## My Approach to the Data

The provided Excel file contained typical banking/financial data with customer and account information all mixed together in one table. After extracting the data to CSV format, I found these columns:

- ID, ACCOUNT_NO, ACC_SORT_CODE, CREATE_DATE, UPDATE_DATE, doc_type, name, email, AccountType, doc_number

### Issues with Current Structure:

Looking at this data, several issues were noted:

1. **Too Much Repetition**: I saw the same sort codes appearing multiple times (40431000, 40499000, etc.) - redundancy
2. **Confusing Field Names**: The "AccountType" field actually contains currency codes (KES, USD) rather than account types like "Savings" or "Checking" - this was misleading
3. **Everything Mixed Together**: Customer personal info is right next to account details in the same row, which makes the data harder to manage
4. **Update Issues**: If a bank branch changes its sort code, you'd have to update multiple rows - not ideal!

## How I Tackled the Normalization

### First Normal Form (1NF)

The data was already in 1NF.

- All the values were atomic (no lists or complex objects in cells)
- No repeating groups
- Each row had a unique ID as the primary key

### Second Normal Form (2NF)

Customer information like name and email were tied to account records, but they really depend on the customer, not the specific account. A customer could have multiple accounts, so why repeat their name and email for each one?

**Solution**: Split customer data into its own table.

### Third Normal Form (3NF)

- Remove transitive dependencies
- Separate sort codes into their own entity
- Separate document types into their own entity

## Normalized Database Design

### Table 1: Customers

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    doc_type_id INT,
    doc_number VARCHAR(50),
    created_date TIMESTAMP,
    updated_date TIMESTAMP,
    FOREIGN KEY (doc_type_id) REFERENCES document_types(doc_type_id)
);
```

### Table 2: Document_Types

```sql
CREATE TABLE document_types (
    doc_type_id INT PRIMARY KEY AUTO_INCREMENT,
    doc_type_name VARCHAR(50) UNIQUE NOT NULL
);
```

### Table 3: Bank_Branches

```sql
CREATE TABLE bank_branches (
    sort_code INT PRIMARY KEY,
    branch_name VARCHAR(100),
    branch_location VARCHAR(100)
);
```

### Table 4: Currency_Types

```sql
CREATE TABLE currency_types (
    currency_code CHAR(3) PRIMARY KEY,
    currency_name VARCHAR(50),
    currency_symbol VARCHAR(5)
);
```

### Table 5: Accounts

```sql
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    account_no BIGINT UNIQUE NOT NULL,
    customer_id INT NOT NULL,
    sort_code INT NOT NULL,
    currency_code CHAR(3) NOT NULL,
    created_date TIMESTAMP,
    updated_date TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (sort_code) REFERENCES bank_branches(sort_code),
    FOREIGN KEY (currency_code) REFERENCES currency_types(currency_code)
);
```

## Data Migration

### Document Types Data:

```sql
INSERT INTO document_types (doc_type_name) VALUES
('NationalID'), ('MilitaryID'), ('CR12'), ('Passport');
```

### Currency Types Data:

```sql
INSERT INTO currency_types (currency_code, currency_name, currency_symbol) VALUES
('KES', 'Kenyan Shilling', 'KSh'),
('USD', 'US Dollar', '$');
```

### Bank Branches Data:

```sql
INSERT INTO bank_branches (sort_code, branch_name) VALUES
(40463000, 'Branch A'),
(40499000, 'Branch B'),
(40425000, 'Branch C'),
(40431000, 'Branch D');
```

### Customers Data:

```sql
INSERT INTO customers (customer_id, name, email, doc_type_id, doc_number, created_date, updated_date) VALUES
(1, 'John Doe', 'john@doe.com', 1, '12345678', '2021-06-14 11:31:53', '2021-06-14 11:31:53'),
(2, 'James Smith', 'james@smith.com', 2, '12345679', '2016-11-29 23:25:10', '2016-11-29 23:25:10'),
```

### Accounts Data:

```sql
INSERT INTO accounts (account_id, account_no, customer_id, sort_code, currency_code, created_date, updated_date) VALUES
(16828317, 9876543210, 1, 40463000, 'KES', '2021-06-14 11:31:53', '2021-06-14 11:31:53'),
(7983757, 9876543211, 2, 40499000, 'KES', '2016-11-29 23:25:10', '2016-11-29 23:25:10'),
```

## Relationships Summary

- **Customers** (1) → (M) **Accounts** (One customer can have multiple accounts)
- **Document_Types** (1) → (M) **Customers** (One document type used by multiple customers)
- **Bank_Branches** (1) → (M) **Accounts** (One branch serves multiple accounts)
- **Currency_Types** (1) → (M) **Accounts** (One currency used by multiple accounts)

This normalized structure follows 3NF principles and eliminates the identified issues in the original table.

## ER Diagram (Text Representation) - Part 2

[Customers] --< [Accounts] >-- [Bank_Branches]
| |
v v
[Document_Types] [Currency_Types]

**ENTITIES:**

- **Customers**: customer_id, name, email, doc_number, created_date, updated_date
- **Accounts**: account_id, account_no, created_date, updated_date
- **Bank_Branches**: sort_code, branch_name, branch_location
- **Document_Types**: doc_type_id, doc_type_name
- **Currency_Types**: currency_code, currency_name, currency_symbol

**RELATIONSHIPS:**

- Customers can have multiple Accounts (1:M)
- Accounts belong to one Bank_Branch (M:1)
- Customers use one Document_Type (M:1)
- Accounts are denominated in one Currency_Type (M:1)

## How the Relationships Work

1. **Customers ↔ Accounts** (1:M)

   - One customer can open multiple accounts (savings, checking, etc.)
   - Each account belongs to exactly one customer

2. **Accounts ↔ Bank_Branches** (M:1)

   - Multiple accounts can be held at the same branch
   - Each account is associated with exactly one branch (identified by sort code)

3. **Customers ↔ Document_Types** (M:1)

   - Many customers can use the same document type (e.g., many people have National IDs)
   - Each customer has exactly one document type for identification

4. **Accounts ↔ Currency_Types** (M:1)
   - Multiple accounts can use the same currency (e.g., many KES accounts)
   - Each account is denominated in exactly one currency
