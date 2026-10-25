erDiagram
    %% ==========================================
    %% GOLD LAYER DIMENSIONAL & STAR SCHEMA ERD
    %% ==========================================

    dim_date {
        INT date_key PK "Format: YYYYMMDD"
        DATE full_date
        VARCHAR day_of_week
        INT calendar_month
        INT calendar_quarter
        INT calendar_year
        BOOLEAN is_weekend
    }

    dim_customer {
        INT customer_id PK
        VARCHAR account_number UK
        VARCHAR first_name
        VARCHAR last_name
        VARCHAR email
        DATE signup_date
        VARCHAR customer_status
        VARCHAR customer_segment
    }

    dim_product {
        INT product_id PK
        VARCHAR product_code UK
        VARCHAR product_name
        VARCHAR product_category
        DECIMAL unit_price
        DATE effective_start_date
        DATE effective_end_date
    }

    dim_subscription {
        INT subscription_id PK
        INT customer_id FK
        INT product_id FK
        DATE activation_date
        DATE termination_date
        VARCHAR subscription_status
    }

    DimCompany {
        BIGINT company_key PK
        BIGINT company_id
        VARCHAR name
        VARCHAR registration_number
        VARCHAR legal_entity_name
        BIGINT industry_key FK
        VARCHAR employee_range
        TIMESTAMP effective_from
        TIMESTAMP effective_to
        BOOLEAN is_current
    }

    DimIndustry {
        BIGINT industry_key PK
        VARCHAR industry_business_key
        VARCHAR industry_name
    }

    DimAccount {
        BIGINT account_key PK
        BIGINT account_id
        VARCHAR status
        VARCHAR brand
        VARCHAR account_type
        VARCHAR language
        VARCHAR customer_type
        TIMESTAMP effective_from
        TIMESTAMP effective_to
        BOOLEAN is_current
    }

    DimB2BAccount {
        BIGINT b2b_account_key PK
        BIGINT b2b_account_id
        VARCHAR name
        BOOLEAN collection_exempt
        VARCHAR payment_term
        DECIMAL credit_limit
        BIGINT parent_b2b_account_id
        TIMESTAMP effective_from
        TIMESTAMP effective_to
        BOOLEAN is_current
    }

    DimContract {
        BIGINT contract_key PK
        BIGINT contract_id
        TIMESTAMP contract_start_at
        TIMESTAMP contract_end_at
    }

    DimService {
        BIGINT service_key PK
        BIGINT service_id
        BIGINT service_type_key FK
        VARCHAR domain
        VARCHAR status
        VARCHAR sim_card_id
        TIMESTAMP effective_from
        TIMESTAMP effective_to
        BOOLEAN is_current
    }

    DimServiceType {
        BIGINT service_type_key PK
        VARCHAR service_type_code
        VARCHAR service_type_name
    }

    DimAccountCategory {
        BIGINT account_category_key PK
        VARCHAR category_type
        VARCHAR category_value
        VARCHAR category_label
    }

    BridgeAccountCategory {
        BIGINT account_category_bridge_key PK
        BIGINT account_key FK
        BIGINT account_category_key FK
        TIMESTAMP effective_from
        TIMESTAMP effective_to
        BOOLEAN is_current
    }

    BridgeAccountContact {
        BIGINT account_contact_bridge_key PK
        BIGINT account_key FK
        VARCHAR contact_uuid
        TIMESTAMP effective_from
        TIMESTAMP effective_to
    }

    FactServiceTerminationRequest {
        BIGINT service_termination_request_fact_key PK
        BIGINT request_id
        BIGINT service_key FK
        INT request_created_date_key FK
        VARCHAR reason
        INT request_count
    }

    FactGatewaySwapRequest {
        BIGINT gateway_swap_request_fact_key PK
        BIGINT request_id
        BIGINT service_key FK
        INT request_date_key FK
        VARCHAR old_residential_gateway_id
        VARCHAR new_residential_gateway_id
        INT request_count
    }

    fact_invoices {
        INT invoice_id PK
        INT customer_id FK
        INT subscription_id FK
        INT issue_date_key FK
        INT due_date_key FK
        DECIMAL billed_amount
        DECIMAL tax_amount
        DECIMAL discount_amount
        VARCHAR payment_status
    }

    fact_usage_summary {
        INT usage_summary_id PK
        INT customer_id FK
        INT subscription_id FK
        INT date_key FK
        DECIMAL total_data_usage_mb
        DECIMAL total_voice_minutes
        INT total_sms_count
        DECIMAL usage_charge_amount
    }

    %% ==========================================
    %% RELATIONSHIPS (STAR & DIMENSIONAL SCHEMAS)
    %% ==========================================

    %% Telecom Billing Star Schema Relationships
    dim_customer ||--o{ dim_subscription : "has"
    dim_product ||--o{ dim_subscription : "associated_with"
    
    dim_customer ||--o{ fact_invoices : "receives"
    dim_subscription ||--o{ fact_invoices : "billed_under"
    dim_date ||--o{ fact_invoices : "issued_on"
    dim_date ||--o{ fact_invoices : "due_on"

    dim_customer ||--o{ fact_usage_summary : "generates"
    dim_subscription ||--o{ fact_usage_summary : "consumes_via"
    dim_date ||--o{ fact_usage_summary : "recorded_on"

    %% Advanced Dimensional Model (Telecom/B2B Warehouse) Relationships
    DimIndustry ||--o{ DimCompany : "classifies"
    DimServiceType ||--o{ DimService : "categorizes"

    DimAccount ||--o{ BridgeAccountCategory : "categorized_by"
    DimAccountCategory ||--o{ BridgeAccountCategory : "applies_to"

    DimAccount ||--o{ BridgeAccountContact : "linked_to_contact"

    DimService ||--o{ FactServiceTerminationRequest : "requests_termination_for"
    dim_date ||--o{ FactServiceTerminationRequest : "requested_on"

    DimService ||--o{ FactGatewaySwapRequest : "requests_swap_for"
    dim_date ||--o{ FactGatewaySwapRequest : "requested_on"
