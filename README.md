This is resources that you will need to setup docker containers like there is docker files, docker compose files,
multiple jars also there which is need to there in your folder directory when you will run any connection on docker.

Task 1: spin off sql server on dokcer and make connection.
Task 2: Make docker-compose and spin off postgres sql server. ( make source connector and sink connector to retrive data into topic to sink table)
task 3: Push the data in kafka topic by creating producer python file(source connector) and fetch that data by making consumer file.
Task 4: As like task 3 but here need to make cdc enable as you will need to create audit table, function, triggers for enabling the cdc then pull a glue image and build a container in previous docker-compose itself; then run your glue script to see the live changes that you are making in main table by cdc enable so glue job will create and will push your data into consumer table.




( select * from profit_loss_data


SELECT column_name, data_type
FROM information_schema.columns
WHERE table_schema = 'public' AND
table_name ='profit_loss_audit';



select * from profit_loss_data_audit
CREATE TABLE profit_loss_data_audit (
    audit_id SERIAL PRIMARY KEY,
    operation_type VARCHAR(10) NOT NULL, -- 'INSERT', 'UPDATE', 'DELETE',  
    changed_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    changed_by TEXT DEFAULT CURRENT_USER,
    Year TEXT,
    Sales_plus DOUBLE PRECISION,
    Expenses_plus DOUBLE PRECISION,
    Operating_Profit DOUBLE PRECISION,
    OPM_percent DOUBLE PRECISION,
    Other_Income_plus DOUBLE PRECISION,
    Interest DOUBLE PRECISION,
    Depreciation DOUBLE PRECISION,
    Profit_before_tax DOUBLE PRECISION,
    Tax_percent DOUBLE PRECISION,
    Net_Profit_plus DOUBLE PRECISION,
    EPS_in_Rs DOUBLE PRECISION,
    Dividend_Payout_percent DOUBLE PRECISION
);

drop table profit_loss_audit
-----------------------------------------------------------------------------------------------



INSERT INTO profit_loss_audit (
    operation_type,
    changed_at,
    changed_by,
    Year,
    Sales_plus,
    Expenses_plus,
    Operating_Profit,
    OPM_percent,
    Other_Income_plus,
    Interest,
    Depreciation,
    Profit_before_tax,
    Tax_percent,
    Net_Profit_plus,
    EPS_in_Rs,
    Dividend_Payout_percent
)
SELECT
    'INSERT' AS operation_type,  -- Marking as initial load
    CURRENT_TIMESTAMP AS changed_at,
    CURRENT_USER AS changed_by,
    "Year",
    "Sales",
    "Expenses",
    "Operating_Profit",
    "OPM_%",
    "Other_Income",
    "Interest",
    "Depreciation",
    "Profit_before_tax",
    "Tax_%",
    "Net_Profit",
    "EPS_in_Rs",
    "Dividend_Payout_%"
FROM profit_loss_data;



DELETE FROM profit_loss_data
WHERE Year IN ('Mar 2025', 'Jun 2025');
---------------------------------------------------------------------------------------------

drop table profit_loss_audit 
CREATE OR REPLACE FUNCTION profit_loss_data_log()
RETURNS TRIGGER AS $$
BEGIN
    IF (TG_OP = 'INSERT') THEN
        INSERT INTO profit_loss_data_audit (operation_type, Year, Sales_plus, Expenses_plus, Operating_Profit, OPM_percent, Other_Income_plus, Interest, Depreciation, Profit_before_tax, Tax_percent, Net_Profit_plus, EPS_in_Rs, Dividend_Payout_percent)
        VALUES ('INSERT', NEW."Year", NEW."Sales", NEW."Expenses", NEW."Operating_Profit", NEW."OPM_%", NEW."Other_Income", NEW."Interest", NEW."Depreciation", NEW."Profit_before_tax", NEW."Tax_%", NEW."Net_Profit", NEW."EPS_in_Rs", NEW."Dividend_Payout_%");
        RETURN NEW;
    ELSIF (TG_OP = 'UPDATE') THEN
        INSERT INTO profit_loss_data_audit (operation_type, Year, Sales_plus, Expenses_plus, Operating_Profit, OPM_percent, Other_Income_plus, Interest, Depreciation, Profit_before_tax, Tax_percent, Net_Profit_plus, EPS_in_Rs, Dividend_Payout_percent)
        VALUES ('UPDATE', NEW."Year", NEW."Sales", NEW."Expenses", NEW."Operating_Profit", NEW."OPM_%", NEW."Other_Income", NEW."Interest", NEW."Depreciation", NEW."Profit_before_tax", NEW."Tax_%", NEW."Net_Profit", NEW."EPS_in_Rs", NEW."Dividend_Payout_%");
        RETURN NEW;
    ELSIF (TG_OP = 'DELETE') THEN
        INSERT INTO profit_loss_data_audit (operation_type, Year, Sales_plus, Expenses_plus, Operating_Profit, OPM_percent, Other_Income_plus, Interest, Depreciation, Profit_before_tax, Tax_percent, Net_Profit_plus, EPS_in_Rs, Dividend_Payout_percent)
        VALUES ('DELETE', OLD."Year", OLD."Sales", NEW."Expenses", OLD."Operating_Profit", OLD."OPM_%", OLD."Other_Income", OLD."Interest", OLD."Depreciation", OLD."Profit_before_tax", OLD."Tax_%", OLD."Net_Profit", OLD."EPS_in_Rs", OLD."Dividend_Payout_%");
        RETURN OLD;
    END IF;
END;
$$ LANGUAGE plpgsql;
 
CREATE TRIGGER profit_loss_data_trigger
AFTER INSERT OR UPDATE OR DELETE ON profit_loss_data
FOR EACH ROW EXECUTE FUNCTION profit_loss_data_log();


CREATE TABLE profit_loss_data_sink(
    Year TEXT,
    Sales_plus DOUBLE PRECISION,
    Expenses_plus DOUBLE PRECISION,
    Operating_Profit DOUBLE PRECISION,
    OPM_Percent DOUBLE PRECISION,
    Other_Income_plus DOUBLE PRECISION,
    Interest DOUBLE PRECISION,
    Depreciation DOUBLE PRECISION,
    Profit_before_tax DOUBLE PRECISION,
    Tax_Percent DOUBLE PRECISION,
    Net_Profit_plus DOUBLE PRECISION,
    EPS_in_Rs DOUBLE PRECISION,
    Dividend_Payout_Percent DOUBLE PRECISION
);







#from here continue to concourse pipelining repository for next tasks:
 
drop table profit_loss_data_sink 
select * from profit_loss_data_sink;
 
)

This is for enabling cdc, for that purpose i had used postgres server on dbeaver. 
