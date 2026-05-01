# Requirements Document

## Introduction

This feature adds a "Shortage Review" button adjacent to the existing "Reconcile" button in the Purchase Order (PO) and transship reconciliation workflow. When activated, the button surfaces partial case shortage information for the PO being reconciled — including which items are short, who stowed them, and where they were stowed. The goal is to give warehouse staff the information they need to resolve discrepancies and receive the correct unit count before finalizing the reconcile process.

## Glossary

- **PO (Purchase Order)**: A document representing an order of goods from a supplier that must be received and reconciled in the warehouse management system.
- **Transship**: A transfer shipment between warehouse locations that follows the same reconciliation workflow as a PO.
- **Reconcile**: The process of confirming that received inventory matches the expected quantities on a PO or transship.
- **Shortage**: A discrepancy where the quantity of units received is less than the quantity expected on the PO or transship.
- **Partial Case Shortage**: A shortage where at least one unit from a case was received but the total received quantity is less than the expected quantity (received > 0 and received < expected). This is distinct from a full miss, where zero units were received (received = 0).
- **Stow**: The act of placing received inventory into a designated warehouse location.
- **Stower**: The warehouse associate who performed the stow action for a given item. Stower information is sourced from data already present on the Reconciliation_Screen.
- **Stow Location**: The physical warehouse location where an item was placed during the stow process, identified by temp zone, aisle, and shelf. The full location string is formed by concatenating all three directly (e.g., "P-1-A110A100"), where "P-1-A" is the temp zone, "110" is the aisle number, and "A100" is the shelf.
- **IHM Inventory Audit Dashboard**: An external internal tool (`https://grocerycentral.amazon.dev/ihm-inventory-audit-dashboard/inventory-audit-main-page`) used to look up stow location data by stower username and ASIN within a 3-day lookback window.
- **ASIN**: Amazon Standard Identification Number — the item identifier used to query the IHM Inventory Audit Dashboard.
- **Shortage_Review_Panel**: The UI component that displays shortage details when the Shortage Review button is activated.
- **Reconciliation_Screen**: One of two existing IHM pages where a user reconciles a shipment:
  - Transship: `https://grocerycentral.amazon.dev/ihm-transship-dashboards/inbound-manifest-view-v2/{shipmentId}`
  - PO: `https://grocerycentral.amazon.dev/ihm-vendor-po-dashboard/vendor-pos-detail/{shipmentId}`
- **Shipment ID**: The identifier for the current PO or transship, extracted from the page URL.
- **WMS**: Warehouse Management System — the application managing inventory, receiving, and reconciliation workflows.

---

## Requirements

### Requirement 1: Shortage Review Button Placement

**User Story:** As a warehouse associate, I want a "Shortage Review" button next to the "Reconcile" button on the reconciliation screen, so that I can quickly access shortage information without leaving the reconciliation workflow.

#### Acceptance Criteria

1. THE Reconciliation_Screen SHALL display a "Shortage Review" button adjacent to the existing "Reconcile" button when a PO or transship is open for reconciliation.
2. THE Reconciliation_Screen SHALL display the "Shortage Review" button in a visually distinct style from the "Reconcile" button to differentiate the two actions.
3. WHILE a PO or transship is open for reconciliation, THE Reconciliation_Screen SHALL keep the "Shortage Review" button enabled regardless of whether shortages exist.

---

### Requirement 2: Display Partial Case Shortages

**User Story:** As a warehouse associate, I want to see all partial case shortages for the current PO when I press the Shortage Review button, so that I know which items are short before completing the reconcile.

#### Acceptance Criteria

1. WHEN the user activates the "Shortage Review" button, THE Shortage_Review_Panel SHALL display all partial case shortages associated with the current PO or transship, where a partial case shortage is defined as a line item with a received quantity greater than zero and less than the expected quantity.
2. WHEN the user activates the "Shortage Review" button, THE Shortage_Review_Panel SHALL display the item identifier (e.g., SKU or item number), the expected quantity, and the received quantity for each partial case shortage line item.
3. WHEN no partial case shortages exist for the current PO or transship, THE Shortage_Review_Panel SHALL display a message indicating that no shortages were found.
4. THE Shortage_Review_Panel SHALL present shortage data that reflects the state of the PO at the time the button is activated.

---

### Requirement 3: Display Stower Information

**User Story:** As a warehouse associate, I want to see who stowed each shorted item, so that I can follow up with the correct person to resolve the discrepancy.

#### Acceptance Criteria

1. WHEN the user activates the "Shortage Review" button, THE Shortage_Review_Panel SHALL display the name or identifier of the Stower associated with each partial case shortage line item, sourced from the existing data on the Reconciliation_Screen.
2. IF stower information is unavailable for a shortage line item, THEN THE Shortage_Review_Panel SHALL display a placeholder value (e.g., "Unknown") for that field rather than omitting the row.

---

### Requirement 4: Display Stow Location Lookup

**User Story:** As a warehouse associate, I want a quick link to look up where each shorted item was stowed, so that I can open the IHM Inventory Audit Dashboard and find the stow location before completing the reconcile.

#### Acceptance Criteria

1. WHEN the user activates the "Shortage Review" button, THE Shortage_Review_Panel SHALL display a "Look up in Audit Dashboard" link for each partial case shortage line item.
2. WHEN the user clicks the link, it SHALL open the IHM Inventory Audit Dashboard (`https://grocerycentral.amazon.dev/ihm-inventory-audit-dashboard/inventory-audit-main-page`) in a new tab, pre-populated with the stower's username and the item's ASIN as query parameters.
3. IF the stower username is unavailable for a shortage line item, THEN THE Shortage_Review_Panel SHALL display a message indicating the link cannot be generated rather than omitting the row.

---

### Requirement 5: Panel Dismissal

**User Story:** As a warehouse associate, I want to close the shortage review panel and return to the reconciliation screen, so that I can continue or complete the reconcile process after reviewing shortages.

#### Acceptance Criteria

1. WHEN the Shortage_Review_Panel is open, THE Shortage_Review_Panel SHALL provide a control (e.g., a close button or dismiss action) that returns the user to the Reconciliation_Screen.
2. WHEN the user dismisses the Shortage_Review_Panel, THE Reconciliation_Screen SHALL restore its previous state without resetting any reconciliation progress.

---

### Requirement 6: Data Loading and Error Handling

**User Story:** As a warehouse associate, I want the shortage review panel to load reliably and inform me if something goes wrong, so that I am not left with a blank or broken screen.

#### Acceptance Criteria

1. WHEN the user activates the "Shortage Review" button, THE Shortage_Review_Panel SHALL display a loading indicator while shortage data is being retrieved.
2. IF the WMS fails to retrieve shortage data for the current PO or transship, THEN THE Shortage_Review_Panel SHALL display a descriptive error message and provide an option to retry the data retrieval.
3. THE WMS SHALL retrieve and display shortage data within 3 seconds under normal operating conditions.
