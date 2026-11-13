# SmartRetail – AppSheet Integration Documentation

## 1. Overview

This document describes the AppSheet integration developed for the SmartRetail AI-Powered Inventory and Sales Management System.

The AppSheet application provides an interface for authorized inventory users to submit and manage inventory transactions. Google Sheets stores the operational records, while Make.com processes submitted transactions and updates product inventory.

The integration connects:

- AppSheet – Inventory management interface.
- Google Sheets – Central database.
- Make.com – Transaction processing and workflow automation.
- Gmail – Automated notifications.

The objective is to provide a structured and traceable inventory management process while reducing manual data entry errors.

---

## 2. Application and Database Architecture

The AppSheet application connects to the Google Sheets workbook:

**Database:** `smartRETAIL_DB_Sheet`

The application uses the following primary tables:

| Table | Purpose |
|---|---|
| Products | Stores product information and current inventory quantities. |
| Inventory_Transactions | Records restock, sale, adjustment, and return transactions. |
| App_Users | Stores application user information and assigned roles. |

The `Products` table maintains the current stock quantity for each product.

The `Inventory_Transactions` table maintains a record of inventory movements and their processing results.

---

## 3. Inventory Transaction Form

The Inventory Transactions form allows authorized users to submit four types of inventory transactions.

### Transaction Types

| Transaction Type | Description | Stock Effect |
|---|---|---|
| Restock | Adds newly purchased or replenished inventory. | Increases stock |
| Sale | Records products sold. | Decreases stock |
| Adjustment | Corrects inventory discrepancies. | Increases or decreases stock |
| Return | Records products returned to inventory. | Increases stock |

The Product field references the Products table.

Users select products by their product names, while the application stores the corresponding `product_id` in the transaction record.

### Automatically Populated Fields

Several fields are populated automatically when a transaction is created.

| Field | Configuration | Purpose |
|---|---|---|
| inventory_transaction_id | UNIQUEID() | Generates a unique transaction identifier. |
| employee_id | LOOKUP based on USEREMAIL() | Identifies the signed-in application user. |
| transaction_date | TODAY() | Records the transaction date. |
| transaction_source | AppSheet | Identifies transactions submitted through AppSheet. |
| processing_status | Pending | Places new transactions in the processing queue. |
| adjustment_status | Conditional initial value | Sets adjustment transactions to Pending. |

These configurations reduce manual input and help maintain consistent transaction records.

---

## 4. Quantity Validation Rules

The application includes validation rules to ensure that inventory quantities are entered correctly.

The `quantity_change` field uses transaction-specific validation.

### Validation Requirements

**Restock**
- Quantity must be greater than zero.
- Restocking increases available inventory.

**Sale**
- Quantity must be negative.
- The quantity deducted cannot exceed the available stock.

**Adjustment**
- Quantity cannot be zero.
- Positive adjustments increase stock.
- Negative adjustments decrease stock.
- A negative adjustment cannot reduce inventory below zero.

**Return**
- Quantity must be greater than zero.
- Returned products increase available inventory.

### Validation Messages

The application displays validation messages when users enter invalid quantities:

"Restock and Return must have positive quantities. Sale must have negative quantities. Adjustment can be positive or negative."

"Invalid quantity or insufficient stock. Please check the transaction type and available stock."

These rules provide an initial validation layer before transactions are submitted to Make.com.

Make.com also performs processing checks to protect the database against invalid inventory changes.

---

## 5. Available Stock Display

A virtual column named `available_stock` was created in the `Inventory_Transactions` table.

**Display name:** Available Stock (Current Inventory)

The column retrieves the selected product's current stock quantity through the Product reference.

This information appears directly below the Product selection field in the transaction form.

The field is read-only and allows inventory staff to review current stock before submitting a transaction.

---

## 6. Conditional Low-Stock Warning

A virtual column named `low_stock_warning` was created to provide contextual inventory information.

The column compares:

- Current product stock quantity.
- Product reorder level.

When current stock is at or below the reorder level, the application displays:

**LOW STOCK: This product is at or below its reorder level.**

Otherwise, it displays:

**Stock level is normal.**

The warning is informational and helps users identify products that may require replenishment.

The separate Make.com Scenario 02 remains responsible for the automated low-stock email notification process.

---

## 7. Required Fields and Access Control

The application uses required-field rules and role-based access controls.

### Required Fields

- Product selection is required.
- Transaction type is required.
- Notes are required when the transaction type is Adjustment.

### Processing Fields

The following fields are visible for transaction tracking but are not editable by application users:

- processing_status
- processing_message
- processed_at

Make.com updates these fields after processing the transaction.

### Role-Based Access

The application uses the `App_Users` table to identify users and their assigned roles.

Authorized roles include:

- Admin
- Inventory Manager

Role-based expressions are used to control access to inventory functions.

---

## 8. Make.com Integration – Scenario 05

**Scenario Name:** 05 – AppSheet Inventory Transaction Automation

### Purpose

This scenario processes inventory transactions submitted through AppSheet, updates product stock quantities, and records the outcome of each transaction.

### Transaction Processing Workflow

1. **Watch New Rows:** Monitors the `Inventory_Transactions` table for new AppSheet submissions.

2. **Search Products:** Retrieves the corresponding product record using the submitted `product_id`.

3. **Transaction Router:** Directs transactions according to their transaction type.

4. **Restock Processing:** Adds the submitted quantity to the existing product stock.

5. **Sale Processing:** Deducts the submitted quantity from available stock after checking stock availability.

6. **Adjustment Processing:** Applies positive or negative stock adjustments after validation.

7. **Return Processing:** Adds returned quantities to product inventory.

8. **Transaction Record Update:** Updates the original transaction record with its processing result.

9. **Failure Handling:** Records unsuccessful transactions and the reason for failure.

10. **Duplicate Protection:** Uses a Make.com Data Store to prevent previously processed transactions from being applied again.

### Stock Calculation Logic

| Transaction Type | Stock Calculation |
|---|---|
| Restock | Current Stock + Quantity Change |
| Sale | Current Stock - Absolute Quantity Change |
| Positive Adjustment | Current Stock + Quantity Change |
| Negative Adjustment | Current Stock + Quantity Change |
| Return | Current Stock + Quantity Change |

For negative adjustments, the automation checks that sufficient stock is available before applying the deduction.

### Transaction Processing Status

After processing, the original transaction record is updated.

| Status | Meaning |
|---|---|
| Pending | Transaction has been submitted and is waiting for processing. |
| Completed | Transaction has been processed successfully. |
| Failed | Transaction could not be processed. |

The `processing_message` field provides additional information about the result.

The `processed_at` field records the processing timestamp.

### Duplicate Transaction Protection

A Make.com Data Store named `SmartRetail_Processed_Transactions` is used to maintain processed transaction identifiers.

The transaction ID acts as the unique key.

This prevents a previously processed transaction from being applied again, helping protect product inventory from duplicate updates.

---

## 9. Low-Stock Email Notifications – Scenario 02

Low-stock monitoring remains a separate automation workflow.

**Scenario Name:** 02 – Inventory Low-Stock Alert Automation

Scenario 05 updates product inventory after processing AppSheet transactions.

Scenario 02 monitors inventory levels and checks whether a product has reached or fallen below its reorder level.

When the configured low-stock conditions are met, the workflow sends an automated email notification to management.

Keeping these responsibilities separate avoids duplicating low-stock alert logic across multiple scenarios.

---

## 10. Testing and Verification

The AppSheet integration and transaction-processing workflow were tested using different inventory transaction types.

### Test Results

| Test | Expected Result | Result |
|---|---|---|
| Restock transaction | Product stock increases. | Passed |
| Sale transaction | Product stock decreases. | Passed |
| Sale exceeding available stock | Transaction is rejected. | Passed |
| Positive adjustment | Product stock increases. | Passed |
| Negative adjustment | Product stock decreases after validation. | Passed |
| Return transaction | Product stock increases. | Passed |
| Duplicate transaction | Previously processed transaction is not applied again. | Passed |
| Processing status update | Transaction status changes after processing. | Passed |
| Available stock display | Current product stock appears in the form. | Passed |
| Low-stock warning | Warning changes according to stock level. | Passed |

The tests confirmed that the application and Make.com transaction-processing workflow operate together as intended for the tested scenarios.

---

## 11. Implementation Status

The AppSheet inventory management interface and its Make.com integration have been developed and tested.

Completed capabilities include:

- AppSheet inventory transaction form.
- Automatic transaction identification.
- User identification through the App_Users table.
- Product selection through the Products reference.
- Current stock visibility.
- Conditional low-stock warnings.
- Transaction-specific quantity validation.
- Required notes for inventory adjustments.
- Role-based access configuration.
- Make.com transaction processing.
- Automatic product stock updates.
- Duplicate transaction protection.
- Transaction processing status and messages.
- Low-stock email notification integration.

---

## 12. Conclusion

The AppSheet integration extends SmartRetail beyond spreadsheet-based inventory tracking by introducing a user-facing business application.

Inventory staff can submit transactions through a structured interface, review available stock, receive validation feedback, and monitor processing results.

By combining AppSheet, Google Sheets, and Make.com, the system demonstrates how a small business can implement a connected inventory management solution with automated transaction processing and operational safeguards.

This implementation forms an important part of the SmartRetail portfolio project and demonstrates practical skills in business application development, database integration, workflow automation, and process validation.
