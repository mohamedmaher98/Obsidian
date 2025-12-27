

# Investment Module Domain Analysis  
  
A comprehensive analysis of the investment module architecture, entities, documents, and system entries based on the actual codebase implementation.  
  
---  
  
## Step 1: Entity and Document Identification  
  
### 1.1 Investment Master Entities  
  
The system defines 5 investment master types organized in an inheritance hierarchy:  
  
#### Class Hierarchy  
  
```  
MasterFile  
└── AbsInvestment (abstract)  
    └── AbsInvestmentWithTax (abstract)        ├── AbsIPInvestmentShare (abstract)        │   ├── IPShare        │   └── IPStake        ├── IPCashDeposit        ├── IPInvestmentFund        └── IPInvestmentPortfolio```  
  
#### Base Classes  
  
| Base Class | Parent | Key Fields |  
|------------|--------|------------|  
| **AbsInvestment** | MasterFile | `subsidiaryAccounts`, `currencyRate`, `investmentDocStatus` (system), `remarks`, `attachment1-5` |  
| **AbsInvestmentWithTax** | AbsInvestment | `taxPercent`, `taxValue`, `totalAfterTaxes`, `originLastUpdateDate` (system), `profitAmount` (system) |  
| **AbsIPInvestmentShare** | AbsInvestmentWithTax | `investmentCompany`, `purchasePrice` (system), `totalPurchaseValue` (system), `purchaseDate` (system), `priceLastUpdateDate` (system), `currentMarketValue` (system), `salesPrice` (system), `profit` (system), `profitPercentage` (system) |  
  
#### Concrete Investment Entities  
  
| Entity | Parent | Specific Fields (all system=true unless noted) |  
|--------|--------|----------------------------------------------|  
| **IPShare** | AbsIPInvestmentShare | `shareType` (config), `shareCount`, `totalShareCount`, `currentSharePrice`, `averageCostPerShare` |  
| **IPStake** | AbsIPInvestmentShare | `ipStakeType` (config), `stakeCount`, `totalStakeCount`, `currentStakePrice`, `averageCostPerStake` |  
| **IPCashDeposit** | AbsInvestmentWithTax | `bank` (config), `bankAccount` (config), `purchaseDate`, `valueInCurrency`, `valueInLocalCurrency` |  
| **IPInvestmentFund** | AbsInvestmentWithTax | `investmentCompany` (config), `fundType` (config), `purchaseDate`, `totalFundInlocalTotal` |  
| **IPInvestmentPortfolio** | AbsInvestmentWithTax | `investmentCompany` (config), `bank` (config), `bankAccount` (config), `totalPortfolioInLocalCurrency`, `totalPortfolio`, `details` (collection) |  
  
### 1.2 Document Types  
  
The system has 5 document types that affect investments:  
  
#### Document Class Hierarchy  
  
```  
DocumentFile  
└── IPAbsInvestments (abstract)  
    ├── AbsIPCapitalBase (abstract)    │   ├── IPInvestmentStart    │   └── IPInvestmentIncreaseCapital    ├── IPInvestmentEnd    ├── IPInvestmentUpdate    └── IPInvestmentProfitDistribution```  
  
#### Base Document Classes  
  
| Base Class | Key Fields |  
|------------|------------|  
| **IPAbsInvestments** | `ipInvestmentFund`, `fromDate`, `toDate`, `totalAmount`, `refundAmount` (system), `remainingAmount` (system), `ledgerTransReqId` (system), `attachment`, `remarks`, `money` (required), `subsidiary` |  
| **AbsIPCapitalBase** | `investmentCompany`, `investmentOpportunity`, `investmentExpenseRate`, `investmentExpenseValue` |  
  
#### Document Entities  
  
| Document | Parent | Specific Fields | Implements |  
|----------|--------|-----------------|------------|  
| **IPInvestmentStart** | AbsIPCapitalBase | `details` (List<IPInvestmentStartLine>) | HasLedgerTransReqId, IGeneratesAccountingRequest |  
| **IPInvestmentIncreaseCapital** | AbsIPCapitalBase | `investmentStart`, `details` (List<IPInvestmentIncreaseCapitalLine>) | HasLedgerTransReqId, IGeneratesAccountingRequest |  
| **IPInvestmentEnd** | IPAbsInvestments | `investmentStart`, `investment` (generic), `refundedAmount`, `details` (List<IPInvestmentEndLine>) | HasLedgerTransReqId, IGeneratesAccountingRequest |  
| **IPInvestmentUpdate** | IPAbsInvestments | `investmentStart`, `investment` (generic), `udatedValue`, `details` (List<IPInvestmentUpdateLine>) | — |  
| **IPInvestmentProfitDistribution** | IPAbsInvestments | `investment` (generic), `totaldistributedAmout`, `details` (List<IPInvestmentProfitDistributionLine>) | HasLedgerTransReqId, IGeneratesAccountingRequest |  
  
### 1.3 Detail Line Classes  
  
#### Line Class Hierarchy  
  
```  
AbsIPInvestmentAssetLine (abstract)  
├── IPInvestmentStartLine  
├── IPInvestmentIncreaseCapitalLine  
├── IPInvestmentEndLine  
├── IPInvestmentUpdateLine (+ extra fields)  
└── IPInvestmentPortfolioLine  
  
IPInvestmentProfitDistributionLine (standalone)  
```  
  
#### AbsIPInvestmentAssetLine Fields  
  
| Field | Type | Description |  
|-------|------|-------------|  
| `investmentType` | TextDF | Investment type label |  
| `investment` | GenericReference | Target investment (required) |  
| `quantityOrValue` | DecimalDF | Quantity or value |  
| `unitPrice` | DecimalDF | Unit price |  
| `total` | DecimalDF | Total value |  
| `tax` | SimpleMoneyEffectDetails | Tax details |  
| `netAfterTax` | DecimalDF | Net after tax |  
| `currency` | Currency | Currency reference |  
| `currencyRate` | DecimalDF | Currency rate |  
| `localTotal` | DecimalDF | Price in local currency |  
| `localTotalAfterTax` | DecimalDF | Total in local currency after tax |  
| `purchaseDate` | DateDF | Purchase date |  
| `currentPrice` | DecimalDF | Current price |  
| `currentTotal` | DecimalDF | Current total |  
| `currentLocalPrice` | DecimalDF | Current local price |  
| `status` | InvestmentDocStatus | Investment status |  
| `subsidiary` | GenericReference | Subsidiary reference |  
| `bonusInvestment` | BooleanDF | Is bonus investment flag |  
  
#### IPInvestmentUpdateLine Additional Fields  
  
| Field | Type | Description |  
|-------|------|-------------|  
| `previousPrice` | DecimalDF | Previous price |  
| `newPrice` | DecimalDF | New price |  
| `updateDate` | DateDF | Update date |  
  
#### IPInvestmentProfitDistributionLine Fields  
  
| Field | Type | Description |  
|-------|------|-------------|  
| `investment` | GenericReference | Target investment (required) |  
| `investmentProfitValue` | DecimalDF | Profit value |  
  
### 1.4 Existing SysEntry Classes  
  
The system has **4 specialized SysEntry classes** plus one unified (unused) class:  
  
#### SysEntry Hierarchy  
  
```  
LocalEntity  
└── AbsIPInvestmentSysEntry (abstract)  
    ├── IPInvestmentCapitalSysEntry    ├── IPInvestmentStatusSysEntry    ├── IPInvestmentProfitSysEntry    └── IPInvestmentUpdateSysEntry  
IPInvestmentSysEntry (standalone, unified design)  
```  
  
#### AbsIPInvestmentSysEntry (Base) Fields  
  
| Field | Type | Description |  
|-------|------|-------------|  
| `target` | GenericReference | Target investment |  
| `originDoc` | GenericReference | Origin document |  
| `creationDate` | DateTimeDF | Creation timestamp |  
| `valueDate` | DateDF | Value date |  
  
#### Specialized SysEntry Classes  
  
| SysEntry Class | Additional Fields | Purpose |  
|----------------|-------------------|---------|  
| **IPInvestmentCapitalSysEntry** | `quantity`, `unitPrice`, `total`, `localTotal`, `purchaseDate` | Tracks capital contributions from Start and IncreaseCapital |  
| **IPInvestmentStatusSysEntry** | `fromState`, `toState` | Tracks status transitions (Initial → Ongoing → Closed) |  
| **IPInvestmentProfitSysEntry** | `profitInValue`, `profitOutValue` | Tracks profit distributions |  
| **IPInvestmentUpdateSysEntry** | (none) | Records update events |  
  
#### IPInvestmentSysEntry (Unified Design - DSL)  
  
| Field | Type | Description |  
|-------|------|-------------|  
| `target` | GenericReference | Target investment |  
| `originDoc` | GenericReference | Origin document |  
| `creationDate` | DateTimeDF | Creation timestamp |  
| `valueDate` | DateDF | Value date |  
| `entryType` | IPInvestmentSysEntryType | Entry type discriminator |  
| `quantity` | DecimalDF | Quantity |  
| `unitPrice` | DecimalDF | Unit price |  
| `total` | DecimalDF | Total value |  
| `localTotal` | DecimalDF | Local currency total |  
| `purchaseDate` | DateDF | Purchase date |  
| `currencyRate` | DateDF | Currency rate (note: type mismatch) |  
| `fromState` | InvestmentDocStatus | From status |  
| `toState` | InvestmentDocStatus | To status |  
| `profitInValue` | DecimalDF | Profit in |  
| `profitOutValue` | DecimalDF | Profit out |  
  
#### IPInvestmentSysEntryType Enum  
  
```java  
enum IPInvestmentSysEntryType {  
    Capital,  // For Start and IncreaseCapital    Status,   // For Start and End    Profit    // For ProfitDistribution}  
```  
  
---  
  
## Step 2: Field Dependency Analysis  
  
### 2.1 Field Classification by Investment Type  
  
#### IPShare Fields  
  
| Field | System? | Source | Documents That Affect |  
|-------|---------|--------|----------------------|  
| `shareType` | No | User config | — |  
| `shareCount` | Yes | Calculated from start line | Start |  
| `totalShareCount` | Yes | Sum of all capital entries | Start, IncreaseCapital |  
| `currentSharePrice` | Yes | Updated from update line | Update |  
| `averageCostPerShare` | Yes | totalPurchaseValue / totalShareCount | Start, IncreaseCapital, Update |  
| `purchasePrice` | Yes | From start line unit price | Start |  
| `totalPurchaseValue` | Yes | Sum of capital entry totals | Start, IncreaseCapital |  
| `purchaseDate` | Yes | Max of capital entry dates | Start, IncreaseCapital |  
| `priceLastUpdateDate` | Yes | From update document | Update |  
| `currentMarketValue` | Yes | totalShareCount × currentSharePrice | Start, IncreaseCapital, Update |  
| `salesPrice` | Yes | From end line unit price | End |  
| `profit` | Yes | salesPrice - purchasePrice | End |  
| `profitPercentage` | Yes | profit / purchasePrice × 100 | End |  
| `investmentDocStatus` | Yes | From status entries | Start (→Ongoing), End (→Closed) |  
| `profitAmount` | Yes | Sum of profit entries | ProfitDistribution |  
| `originLastUpdateDate` | Yes | Max of all entry dates | Start, IncreaseCapital, End, Update, ProfitDistribution |  
  
#### IPStake Fields  
  
(Identical pattern to IPShare, with stake-specific naming)  
  
| Field | System? | Documents That Affect |  
|-------|---------|----------------------|  
| `ipStakeType` | No | — |  
| `stakeCount` | Yes | Start |  
| `totalStakeCount` | Yes | Start, IncreaseCapital |  
| `currentStakePrice` | Yes | Update |  
| `averageCostPerStake` | Yes | Start, IncreaseCapital, Update |  
| (inherited fields) | Yes | (same as IPShare) |  
  
#### IPCashDeposit Fields  
  
| Field | System? | Source | Documents That Affect |  
|-------|---------|--------|----------------------|  
| `bank` | No | User config | — |  
| `bankAccount` | No | User config | — |  
| `purchaseDate` | Yes | Max of capital entry dates | Start, IncreaseCapital |  
| `valueInCurrency` | Yes | Sum of capital entry totals | Start, IncreaseCapital |  
| `valueInLocalCurrency` | Yes | Sum of capital entry localTotals | Start, IncreaseCapital, Update |  
| `investmentDocStatus` | Yes | From status entries | Start (→Ongoing), End (→Closed) |  
| `profitAmount` | Yes | Sum of profit entries | ProfitDistribution |  
| `originLastUpdateDate` | Yes | Max of all entry dates | All documents |  
  
#### IPInvestmentFund Fields  
  
| Field | System? | Source | Documents That Affect |  
|-------|---------|--------|----------------------|  
| `investmentCompany` | No | User config | — |  
| `fundType` | No | User config | — |  
| `purchaseDate` | Yes | Max of capital entry dates | Start, IncreaseCapital |  
| `totalFundInlocalTotal` | Yes | Sum of capital localTotals | Start, IncreaseCapital, Update |  
| `investmentDocStatus` | Yes | From status entries | Start (→Ongoing), End (→Closed) |  
| `profitAmount` | Yes | Sum of profit entries | ProfitDistribution |  
| `originLastUpdateDate` | Yes | Max of all entry dates | All documents |  
  
#### IPInvestmentPortfolio Fields  
  
| Field | System? | Source | Documents That Affect |  
|-------|---------|--------|----------------------|  
| `investmentCompany` | No | User config | — |  
| `bank` | No | User config | — |  
| `bankAccount` | No | User config | — |  
| `totalPortfolio` | Yes | Sum of capital entry totals | Start, IncreaseCapital, Update |  
| `totalPortfolioInLocalCurrency` | Yes | Sum of capital localTotals | Start, IncreaseCapital, Update |  
| `details` | Collection | Portfolio lines | Indirect via line investments |  
| `investmentDocStatus` | Yes | From status entries | Start (→Ongoing), End (→Closed) |  
| `profitAmount` | Yes | Sum of profit entries | ProfitDistribution |  
| `originLastUpdateDate` | Yes | Max of all entry dates | All documents |  
  
---  
  
## Step 3: Document Impact Matrix  
  
### Document → Master Field Impact  
  
| Investment Type | **Start** | **IncreaseCapital** | **End** | **Update** | **ProfitDistribution** |  
|-----------------|-----------|---------------------|---------|------------|------------------------|  
| **IPShare** | `shareCount`, `totalShareCount`, `purchasePrice`, `totalPurchaseValue`, `purchaseDate`, `currentMarketValue`, `averageCostPerShare`, `investmentDocStatus`→Ongoing, `currency`, `currencyRate`, `originLastUpdateDate` | `totalShareCount`, `totalPurchaseValue`, `purchaseDate`, `currentMarketValue`, `averageCostPerShare`, `originLastUpdateDate` | `investmentDocStatus`→Closed, `salesPrice`, `profit`, `profitPercentage`, `averageCostPerShare`, `originLastUpdateDate` | `currentSharePrice`, `priceLastUpdateDate`, `currentMarketValue`, `totalPurchaseValue`, `shareCount`, `currency`, `currencyRate`, `taxPercent`, `taxValue`, `totalAfterTaxes`, `originLastUpdateDate` | `profitAmount`, `originLastUpdateDate` |  
| **IPStake** | `stakeCount`, `totalStakeCount`, `purchasePrice`, `totalPurchaseValue`, `purchaseDate`, `currentMarketValue`, `averageCostPerStake`, `investmentDocStatus`→Ongoing, `currency`, `currencyRate`, `originLastUpdateDate` | `totalStakeCount`, `totalPurchaseValue`, `purchaseDate`, `currentMarketValue`, `averageCostPerStake`, `originLastUpdateDate` | `investmentDocStatus`→Closed, `salesPrice`, `profit`, `profitPercentage`, `averageCostPerStake`, `originLastUpdateDate` | `currentStakePrice`, `priceLastUpdateDate`, `currentMarketValue`, `totalPurchaseValue`, `stakeCount`, `currency`, `currencyRate`, `taxPercent`, `taxValue`, `totalAfterTaxes`, `originLastUpdateDate` | `profitAmount`, `originLastUpdateDate` |  
| **IPCashDeposit** | `valueInCurrency`, `valueInLocalCurrency`, `purchaseDate`, `investmentDocStatus`→Ongoing, `currency`, `currencyRate`, `originLastUpdateDate` | `valueInCurrency`, `valueInLocalCurrency`, `purchaseDate`, `originLastUpdateDate` | `investmentDocStatus`→Closed, `originLastUpdateDate` | `valueInLocalCurrency`, `currency`, `currencyRate`, `taxPercent`, `taxValue`, `totalAfterTaxes`, `originLastUpdateDate` | `profitAmount`, `originLastUpdateDate` |  
| **IPInvestmentFund** | `totalFundInlocalTotal`, `purchaseDate`, `investmentDocStatus`→Ongoing, `currency`, `currencyRate`, `originLastUpdateDate` | `totalFundInlocalTotal`, `purchaseDate`, `originLastUpdateDate` | `investmentDocStatus`→Closed, `originLastUpdateDate` | `totalFundInlocalTotal`, `currency`, `currencyRate`, `taxPercent`, `taxValue`, `totalAfterTaxes`, `originLastUpdateDate` | `profitAmount`, `originLastUpdateDate` |  
| **IPInvestmentPortfolio** | `totalPortfolio`, `totalPortfolioInLocalCurrency`, `investmentDocStatus`→Ongoing, `currency`, `currencyRate`, `originLastUpdateDate` | `totalPortfolio`, `totalPortfolioInLocalCurrency`, `originLastUpdateDate` | `investmentDocStatus`→Closed, `originLastUpdateDate` | `totalPortfolio`, `currency`, `currencyRate`, `taxPercent`, `taxValue`, `totalAfterTaxes`, `originLastUpdateDate` | `profitAmount`, `originLastUpdateDate` |  
  
### Allowed Investment Types per Document  
  
| Document | Allowed Investment Types |  
|----------|-------------------------|  
| **IPInvestmentStart** | All 5 types |  
| **IPInvestmentIncreaseCapital** | All 5 types |  
| **IPInvestmentEnd** | IPShare, IPStake, IPCashDeposit, IPInvestmentFund (NOT Portfolio) |  
| **IPInvestmentUpdate** | All 5 types |  
| **IPInvestmentProfitDistribution** | IPShare, IPStake, IPCashDeposit, IPInvestmentFund, IPInvestmentPortfolio |  
  
---  
  
## Step 4: Proposed Ideal System Entry Design  
  
### 4.1 Analysis of Current Design  
  
The current system uses **4 separate SysEntry classes**, each specialized for a specific concern:  
  
| Entry Type | Created By | Fields Used | Updates |  
|------------|-----------|-------------|---------|  
| Capital | Start, IncreaseCapital | quantity, unitPrice, total, localTotal, purchaseDate | `updateCapitalFromEntries()` |  
| Status | Start, End | fromState, toState | `investmentDocStatus`, sales fields on End |  
| Profit | ProfitDistribution | profitInValue, profitOutValue | `profitAmount` |  
| Update | Update | (only base fields) | `originLastUpdateDate` |  
  
**Observations:**  
1. There's already a unified `IPInvestmentSysEntry` class in DSL but it's minimally implemented in domain  
2. The current 4-class design provides separation of concerns but leads to:  
   - 4 database tables  
   - 4 sets of CRUD operations  
   - Complex aggregation logic across multiple entry types  
3. The `IPInvestmentUpdateSysEntry` adds no additional data beyond base class  
4. Status and profit entries only need to track their specific concern  
  
### 4.2 Recommended Unified Design  
  
Based on the code analysis, I propose consolidating to a **single unified entry class** with a **type discriminator**:  
  
#### Proposed IPInvestmentSysEntry Structure  
  
```java  
@NaMaValueObject(localEntity = true)  
public class IPInvestmentSysEntry extends LocalEntity {  
    // === Base Fields (always present) ===    GenericReference target;           // Target investment (required)  
    GenericReference originDoc;        // Origin document (required)  
    DateTimeDF creationDate;           // Entry creation timestamp (required)  
    DateDF valueDate;                  // Business value date (required)  
    IPInvestmentSysEntryType entryType; // Type discriminator (required)  
  
    // === Capital Fields (used when entryType = CAPITAL) ===    DecimalDF quantity;                // Nullable - for share/stake types  
    DecimalDF unitPrice;               // Nullable  
    DecimalDF total;                   // Nullable - total value  
    DecimalDF localTotal;              // Nullable - local currency total  
    DateDF purchaseDate;               // Nullable - purchase date for capital  
    DecimalDF currencyRate;            // Nullable - currency rate at entry time  
  
    // === Status Fields (used when entryType = STATUS_CHANGE) ===    InvestmentDocStatus fromState;     // Nullable - previous status  
    InvestmentDocStatus toState;       // Nullable - new status  
  
    // === Profit Fields (used when entryType = PROFIT) ===    DecimalDF profitInValue;           // Nullable - positive profit  
    DecimalDF profitOutValue;          // Nullable - negative profit/loss  
  
    // === Sale Fields (used for End document entries) ===    DecimalDF salesPrice;              // Nullable - sale unit price  
}  
```  
  
#### Proposed Entry Type Enum  
  
```java  
public enum IPInvestmentSysEntryType {  
    CAPITAL_START,           // Initial capital from IPInvestmentStart    CAPITAL_INCREASE,        // Additional capital from IPInvestmentIncreaseCapital    STATUS_TO_ONGOING,       // Status change to Ongoing (from Start)    STATUS_TO_CLOSED,        // Status change to Closed (from End)    VALUATION_UPDATE,        // Price/value update from IPInvestmentUpdate    PROFIT_DISTRIBUTION      // Profit distribution entry}  
```  
  
Alternatively, a simpler enum (matching current DSL):  
  
```java  
public enum IPInvestmentSysEntryType {  
    Capital,   // For Start and IncreaseCapital    Status,    // For Start (→Ongoing) and End (→Closed)    Profit,    // For ProfitDistribution    Update     // For IPInvestmentUpdate}  
```  
  
### 4.3 Document → Entry Mapping  
  
| Document | Entry Type | Fields Populated |  
|----------|-----------|------------------|  
| **IPInvestmentStart** | `Capital` | target, originDoc, creationDate, valueDate, quantity, unitPrice, total, localTotal, purchaseDate, currencyRate |  
| **IPInvestmentStart** | `Status` | target, originDoc, creationDate, valueDate, toState=Ongoing |  
| **IPInvestmentIncreaseCapital** | `Capital` | target, originDoc, creationDate, valueDate, quantity, unitPrice, total, localTotal, purchaseDate, currencyRate |  
| **IPInvestmentEnd** | `Status` | target, originDoc, creationDate, valueDate, toState=Closed, salesPrice (for share-like) |  
| **IPInvestmentUpdate** | `Update` | target, originDoc, creationDate, valueDate |  
| **IPInvestmentProfitDistribution** | `Profit` | target, originDoc, creationDate, valueDate, profitInValue OR profitOutValue |  
  
### 4.4 Master Field Derivation Logic  
  
```java  
// In AbsInvestmentWithTax  
public void recalculateFromEntries() {  
    // Capital fields    updateCapitalFromEntries();  // Aggregate Capital entries  
    // Status field    updateStatusFromEntries();   // Find latest Status entry  
    // Profit field    updateProfitAmountFromEntries();  // Sum Profit entries  
    // Last update date    updateOriginLastUpdateDateFromEntries();  // Max of all entries}  
  
// In AbsIPInvestmentShare  
@Override  
public void updateCapitalFromEntries() {  
    List<IPInvestmentSysEntry> capitalEntries = findEntriesByType(this, Capital);  
    DecimalDF totalQuantity = sum(capitalEntries, IPInvestmentSysEntry::getQuantity);    DecimalDF totalValue = sum(capitalEntries, IPInvestmentSysEntry::getTotal);    DateDF lastPurchaseDate = max(capitalEntries, IPInvestmentSysEntry::getPurchaseDate);  
    IPInvestmentSysEntry startEntry = findFirstByType(this, Capital);    DecimalDF startQuantity = startEntry != null ? startEntry.getQuantity() : ZERO;    DecimalDF startUnitPrice = startEntry != null ? startEntry.getUnitPrice() : ZERO;  
    updateUnitCount(startQuantity);      // shareCount or stakeCount    updateTotalUnitCount(totalQuantity); // totalShareCount or totalStakeCount    setPurchasePrice(startUnitPrice);    setTotalPurchaseValue(totalValue);    setPurchaseDate(lastPurchaseDate);  
    if (fetchCurrentUnitPrice() != null) {        setCurrentMarketValue(totalQuantity * fetchCurrentUnitPrice());    }  
    calculateAverageCost();}  
```  
  
### 4.5 Benefits of Unified Design  
  
1. **Single Table Storage**: All entries in one table with efficient indexing  
2. **Simplified Queries**: One query type for all entry operations  
3. **Consistent Pattern**: Same CRUD operations regardless of entry type  
4. **Audit Trail**: Complete history of all changes in one place  
5. **Flexible Extension**: New entry types can be added without schema changes  
6. **Query Efficiency**: Filter by entryType for specific aggregations  
  
### 4.6 Migration Path  
  
The existing DSL already defines `IPInvestmentSysEntry` with the unified structure. To complete the migration:  
  
1. **Implement domain logic** in `IPInvestmentSysEntry.java`:  
   - Add static methods for finding/creating entries by type  
   - Add aggregation methods (sum, max, etc.) filtered by type  
  
2. **Update document classes** to use unified entry:  
   - Replace `IPInvestmentCapitalSysEntry.createOrUpdateEntries()` with `IPInvestmentSysEntry.createCapitalEntries()`  
   - Replace `IPInvestmentStatusSysEntriesUtil` with `IPInvestmentSysEntry.createStatusEntries()`  
   - Replace `IPInvestmentProfitSysEntry` with `IPInvestmentSysEntry.createProfitEntries()`  
   - Replace `IPInvestmentUpdateSysEntry` with `IPInvestmentSysEntry.createUpdateEntries()`  
  
3. **Update investment classes** to query unified table:  
   - Modify `updateCapitalFromEntries()` to filter by `entryType = Capital`  
   - Modify `updateProfitAmountFromEntries()` to filter by `entryType = Profit`  
   - Modify status calculations to filter by `entryType = Status`  
  
4. **Data migration**: Create migration script to consolidate existing 4 tables into unified table  
  
---  
  
## Summary  
  
### Current State  
  
- **5 Investment Masters**: IPShare, IPStake, IPCashDeposit, IPInvestmentFund, IPInvestmentPortfolio  
- **5 Documents**: Start, IncreaseCapital, End, Update, ProfitDistribution  
- **4 SysEntry Classes**: Capital, Status, Profit, Update (+ 1 unused unified class)  
- **Pattern**: Each document creates specific entry types, investments aggregate entries to derive system fields  
  
### Key Insights from Code  
  
1. **Two-phase effect**: Start document creates BOTH Capital AND Status entries  
2. **Polymorphic updates**: Update document has type-specific logic for each investment type  
3. **Entry ordering**: Status entries are sorted by valueDate+creationDate; last entry determines current status  
4. **Bonus handling**: Share-like investments can be "bonus" (free), affecting cost calculations  
5. **Portfolio special case**: Portfolio contains lines referencing other investments, syncs their values  
  
### Recommended Actions  
  
1. Complete implementation of unified `IPInvestmentSysEntry` in domain layer  
2. Consolidate 4 specialized entry classes into unified class  
3. Consider using detailed entry type enum for better audit trail  
4. Add comprehensive tests for entry aggregation logic