# Gift Registry Product Types Implementation Plan

## Development Rules

**CRITICAL - Follow these rules throughout implementation:**

1. **Precision**: Don't assume anything and don't overthink - be very precise while coding and don't add unnecessary code that is not needed

2. **Documentation**: Add comments when needed along with code, and always ask questions when in doubt - don't hallucinate and don't assume things

3. **Approval Process**: Always show the exact changes to be made first and wait for approval before implementing - do not proceed without confirmation

## Current State

**Current Item Structure:**
```json
{
  "product_id": "556548",
  "desired_quantity": 2,
  "purchased_quantity": 0,
  "priority": "high",
  "added_at": "2025-12-18T10:35:13+00:00",
  "notes": "Testing product"
}
```

**Limitations:**
- Only supports simple products
- Uses `product_id` for identification
- No configuration support for variants/bundles
- Basic structure without product type awareness

## Target State

**New Item Structure:**
```json
{
  "item_id": "item_001",
  "product_sku": "1577197-78220",
  "product_type": "simple|configurable|config_custom|bundle",
  "desired_quantity": 2,
  "purchased_quantity": 0,
  "priority": "high",
  "notes": "Testing product",
  "selected_config": {
    "configurable_options": [],
    "custom_options": [],
    "bundle_selections": []
  }
}
```

## Product Types Support

### 1. Simple Product
```json
{
  "item_id": "item_001",
  "product_sku": "1577197-78220",
  "product_type": "simple",
  "selected_config": {
    "configurable_options": [],
    "custom_options": [],
    "bundle_selections": []
  }
}
```

### 2. Configurable Product
```json
{
  "item_id": "item_002",
  "product_sku": "12795900-A0053340-CONFIG",
  "product_type": "configurable",
  "selected_config": {
    "configurable_options": [
      {
        "attribute_code": "clothessizeexact",
        "option_id": "843",
        "label": "Size",
        "value": "5-6 Years",
        "selected_sku": "12795900-A0053340_120-5-6Y",
        "selected_product_id": "A0053340_120-5-6Y"
      }
    ],
    "custom_options": [],
    "bundle_selections": []
  }
}
```

### 3. Simple Custom Options Product
```json
{
  "item_id": "item_003",
  "product_sku": "1577197-78220",
  "product_type": "simple_custom",
  "selected_config": {
    "configurable_options": [],
    "custom_options": [
      {
        "option_id": "12345",
        "label": "Personalized Name",
        "value": "Ahmad",
        "type": "field"
      }
    ],
    "bundle_selections": []
  }
}
```

### 4. Configurable + Custom Options
```json
{
  "item_id": "item_004",
  "product_sku": "44469079-SE_30111978-CONFIG",
  "product_type": "config_custom",
  "selected_config": {
    "configurable_options": [
      {
        "attribute_code": "clothessizeexact",
        "label": "Size",
        "value": "0-3 Months",
        "selected_sku": "44469079-SE_30111978_0-6",
        "selected_product_id": "SE_30111978_0-6"
      }
    ],
    "custom_options": [
      {
        "option_id": "12345",
        "label": "Personalized Name",
        "value": "Ahmad",
        "type": "field"
      }
    ],
    "bundle_selections": []
  }
}
```

### 5. Bundle Product
```json
{
  "item_id": "item_005",
  "product_sku": "CBNDL-35025653-2627",
  "product_type": "bundle",
  "selected_config": {
    "configurable_options": [],
    "custom_options": [],
    "bundle_selections": [
      {
        "option_id": "1416",
        "label": "Bath Tub",
        "selection_id": "2343",
        "selected_sku": "35025653-10227",
        "selected_product_id": "10227"
      },
      {
        "option_id": "1417",
        "label": "Bath Seat",
        "selection_id": "2344",
        "selected_sku": "35025653-10226",
        "selected_product_id": "10226"
      }
    ]
  }
}
```

### 6. Bundle Custom Options Product
```json
{
  "item_id": "item_006",
  "product_sku": "CBNDL-35025653-2627",
  "product_type": "bundle_custom",
  "selected_config": {
    "configurable_options": [],
    "custom_options": [
      {
        "option_id": "12345",
        "label": "Gift Message",
        "value": "Happy Birthday!",
        "type": "field"
      }
    ],
    "bundle_selections": [
      {
        "option_id": "1416",
        "label": "Bath Tub",
        "selection_id": "2343",
        "selected_sku": "35025653-10227",
        "selected_product_id": "10227"
      }
    ]
  }
}
```

## Algolia Data Structure Analysis

### Product Type Identification
- **Simple**: `type_id: "simple"`, `has_options: 0` - Single SKU, no variants
- **Simple Custom**: `type_id: "simple"`, `has_options: 1` - Single SKU + custom options
- **Configurable**: `type_id: "configurable"`, `has_options: 0` - Multiple SKUs, size/color options
- **Config + Custom**: `type_id: "configurable"`, `has_options: 1` - Multiple SKUs + custom options
- **Bundle**: `type_id: "bundle"`, `has_options: 0` - Multiple component SKUs, bundle options
- **Bundle Custom**: `type_id: "bundle"`, `has_options: 1` - Bundle + custom options

### Key Algolia Fields
- `sku`: Array containing parent + variant/component SKUs
- `type_id`: Product type (simple, configurable, bundle)
- `has_options`: 1 indicates products with custom options, 0 for standard products
- `clothessizeexact`: Available size options for configurables
- `clothessizeexact_options`: Size option IDs
- `default_bundle_options`: Bundle option mappings
- `objectID`: Primary identifier used as item_id in registry

### Custom Options Handling
- **Storage**: Local database in `selected_config.custom_options`
- **Structure**: `{"option_id": "12345", "label": "Personalized Name", "value": "Ahmad", "type": "field"}`
- **Algolia**: Custom options NOT exposed in Algolia data

## Implementation Plan

### Phase 1: Model & Database Updates
- [x] Update GiftRegistry model fillable fields
- [x] Add new item structure support (`item_id`, `product_sku`, `product_type`, `selected_config`)
- [x] Update item validation methods
- [x] Test model changes

### Phase 2: Request Validation
- [x] Update AddItemRequest validation rules
- [x] Add product_type enum validation (6 types: `simple|simple_custom|configurable|config_custom|bundle|bundle_custom`)
- [x] Add selected_config structure validation
- [x] Update UpdateItemRequest
- [x] Update Admin AddItemRequest to new structure

### Phase 3: Service Layer Updates
- [x] Modify GiftRegistryService::addItem()
- [x] Update item management methods
- [x] Handle different product types
- [x] Update product enrichment logic

### Phase 4: Product Enrichment Strategy
- [x] **Simple**: Enrich main product only (fetch by item_id)
- [x] **Configurable**: Enrich parent product (fetch by item_id) + selected variant details from selected_config
- [x] **Config + Custom**: Same as configurable (custom options stored locally)
- [x] **Bundle**: Enrich parent bundle (fetch by item_id) + component details from selected_config
- [x] **Note**: Current enrichment works correctly - no additional changes needed

### Phase 5: Controller Updates
- [x] Update GiftRegistryController methods to use new structure
- [x] Update AdminGiftRegistryItemController methods to use new structure
- [x] Handle new request structure with item_id, product_sku, product_type
- [x] Update route parameters: `/{product_id}` → `/{item_id}`
- [x] Update webhook handlers (WebhookOrderPlacedRequest, WebhookOrderCancelledRequest)
- [x] Update response format

### Phase 6: Testing & Validation
- [ ] Update admin endpoints and requests
- [ ] Update admin AddItemRequest validation
- [ ] Update admin routes to use item_id
- [ ] Update admin controller methods
- [ ] Test all product types
- [ ] Validate enrichment works correctly
- [ ] Test API endpoints
- [ ] Performance testing

## Key Changes Summary

**Field Changes:**
- `product_id` → `item_id` (rename)
- Add `product_sku` (new)
- Add `product_type` (mandatory enum)
- Add `selected_config` (new object)

**Product Type Enum:**
- `simple` (Algolia: simple, has_options=0)
- `simple_custom` (Algolia: simple, has_options=1)
- `configurable` (Algolia: configurable, has_options=0)
- `config_custom` (Algolia: configurable, has_options=1)
- `bundle` (Algolia: bundle, has_options=0)
- `bundle_custom` (Algolia: bundle, has_options=1)

**Enrichment Strategy:**
- Simple: Single product fetch by item_id ✅
- Simple Custom: Single product fetch by item_id + custom options stored locally ✅
- Configurable: Parent product fetch by item_id + variant details from selected_config ✅
- Config + Custom: Same as configurable + custom options stored locally ✅
- Bundle: Parent bundle fetch by item_id + component details from selected_config ✅
- Bundle Custom: Same as bundle + custom options stored locally ✅
- All product data fetched from Algolia using existing AlgoliaService methods ✅

## Progress Tracker

- [x] Phase 1: Model & Database Updates ✅
  - [x] Update GiftRegistry model addItem() method
  - [x] Update removeItem() method to use item_id
  - [x] Update markItemAsPurchased() method to use item_id
  - [x] Add product_type enum validation
  - [x] Add selected_config default structure
- [x] Phase 2: Request Validation ✅
  - [x] Update AddItemRequest validation rules
  - [x] Update UpdateItemRequest validation rules
  - [x] Update API routes to use item_id parameter
  - [x] Add product_type enum validation (6 types)
  - [x] Add selected_config array validation
  - [x] Update Admin AddItemRequest to new structure
  - [x] Update Admin UpdateItemRequest to support selected_config  
- [x] Phase 3: Service Layer Updates ✅
  - [x] Update addItem() method signature and implementation
  - [x] Update removeItem() to use item_id
  - [x] Update updateItem() to use item_id and support selected_config
  - [x] Update markItemAsPurchased() to use item_id
  - [x] Update unmarkItemAsPurchased() to use item_id
  - [x] Update getItem() to use item_id
  - [x] Update product enrichment to use item_id (correct - Algolia expects item_id)
  - [x] Update GiftRegistryController to use new structure
  - [x] Update AdminGiftRegistryItemController to use new structure
  - [x] Update webhook requests to use item_id
- [x] Phase 4: Product Enrichment Strategy ✅
  - [x] Current enrichment works correctly with all product types
  - [x] Selected_config data properly stored and returned
  - [x] No additional changes needed
- [x] Phase 5: Controller Updates ✅ (Completed in Phase 3)
  - [x] All controllers updated to use new structure
  - [x] Routes updated to use item_id
  - [x] Webhooks updated
- [ ] Phase 6: Testing & Validation
  - [ ] Test all 6 product types
  - [ ] Validate API endpoints work correctly
  - [ ] Test selected_config storage and retrieval
  - [ ] Update API documentation
  - [ ] Performance testing

## Updates Needed

### API Changes (item_id replacement)
- [x] Update all API endpoints to use `item_id` instead of `product_id`
- [x] Update route parameters: `/{product_id}` → `/{item_id}`
- [x] Update controller method parameters
- [x] Update service layer method signatures

### Documentation Updates
- [ ] Update API documentation
- [ ] Update OpenAPI specification (openapi.yaml)
- [ ] Update endpoint descriptions and examples
- [ ] Update request/response schemas

### Model & Service Updates
- [x] Replace `product_id` with `item_id` in model methods
- [x] Update item structure in `addItem()`, `removeItem()`, etc.
- [x] Update service layer methods
- [x] Update request validation

### Admin Endpoints Updates
- [x] Update admin request validation classes
- [x] Update admin routes to use item_id
- [x] Update admin controller methods
- [ ] Update admin API documentation

**Status**: Phases 1-5 Complete ✅  
**Next Step**: Phase 6 - Testing & Validation

**Implementation Summary:**
- ✅ 6 product types supported: simple, simple_custom, configurable, config_custom, bundle, bundle_custom
- ✅ New item structure: item_id, product_sku, product_type, selected_config
- ✅ All models, services, controllers, and requests updated
- ✅ Product enrichment working with existing Algolia integration
- ✅ Ready for testing and validation

## Pending Things

### Shipping Address Change ✅
**Change**: `shipping_address` (object) → `shipping_address_id` (string/integer)

**Files Updated (9 total):**
- [x] `app/Models/GiftRegistry.php` - fillable field, cast, setter method
- [x] `app/Http/Requests/GiftRegistry/CreateRegistryRequest.php` - validation rules
- [x] `app/Http/Requests/GiftRegistry/UpdateRegistryRequest.php` - validation rules
- [x] `app/Http/Requests/Admin/CreateRegistryRequest.php` - validation rules
- [x] `app/Http/Requests/Admin/UpdateRegistryRequest.php` - validation rules
- [x] `app/Http/Controllers/GiftRegistryController.php` - controller methods
- [x] `app/Http/Controllers/API/AdminGiftRegistryController.php` - admin methods
- [x] `app/Services/GiftRegistryService.php` - createRegistry() and updateRegistry()
- [x] `database/factories/GiftRegistryFactory.php` - factory data
- [x] `README.md` - documentation

### Phase 6: Testing & Validation
- [ ] Test all 6 product types
- [ ] Validate API endpoints work correctly
- [ ] Test selected_config storage and retrieval
- [ ] Performance testing

### Documentation Updates
- [ ] Update API documentation
- [ ] Update OpenAPI specification (openapi.yaml)
- [ ] Update endpoint descriptions and examples
- [ ] Update request/response schemas
- [ ] Update admin API documentation

---

## API Testing Guide

### Prerequisites
- Replace `{REGISTRY_ID}` with actual registry ID
- Replace `{USER_ID}` with actual user ID
- Set `X-User-ID` header for authentication

### 1. Simple Product
```bash
curl -X POST "http://localhost/api/registries/{REGISTRY_ID}/items" \
  -H "Content-Type: application/json" \
  -H "X-User-ID: {USER_ID}" \
  -d '{
    "item_id": "348598",
    "product_sku": "1577197-78220",
    "product_type": "simple",
    "desired_quantity": 2,
    "priority": "high",
    "notes": "Simple product test",
    "selected_config": {
      "configurable_options": [],
      "custom_options": [],
      "bundle_selections": []
    }
  }'
```

**Expected Response:**
```json
{
  "item_id": "item_simple_001",
  "product_sku": "1577197-78220",
  "product_type": "simple",
  "desired_quantity": 2,
  "purchased_quantity": 0,
  "priority": "high",
  "notes": "Simple product test",
  "selected_config": {
    "configurable_options": [],
    "custom_options": [],
    "bundle_selections": []
  },
  "added_at": "2025-01-XX..."
}
```

### 2. Simple Custom Product
```bash
curl -X POST "http://localhost/api/registries/{REGISTRY_ID}/items" \
  -H "Content-Type: application/json" \
  -H "X-User-ID: {USER_ID}" \
  -d '{
    "item_id": "305183",
    "product_sku": "1577197-78220",
    "product_type": "simple_custom",
    "desired_quantity": 1,
    "priority": "medium",
    "selected_config": {
      "configurable_options": [],
      "custom_options": [
        {
          "option_id": "12345",
          "label": "Personalized Name",
          "value": "Ahmad",
          "type": "field"
        }
      ],
      "bundle_selections": []
    }
  }'
```

### 3. Configurable Product (Size Option)
```bash
curl -X POST "http://localhost/api/registries/{REGISTRY_ID}/items" \
  -H "Content-Type: application/json" \
  -H "X-User-ID: {USER_ID}" \
  -d '{
    "item_id": "528277",
    "product_sku": "12795900-A0053340-CONFIG",
    "product_type": "configurable",
    "desired_quantity": 1,
    "priority": "high",
    "selected_config": {
      "configurable_options": [
        {
          "attribute_code": "clothessizeexact",
          "option_id": "843",
          "label": "Size",
          "value": "5-6 Years",
          "selected_sku": "12795900-A0053340_120-5-6Y",
          "selected_product_id": "A0053340_120-5-6Y"
        }
      ],
      "custom_options": [],
      "bundle_selections": []
    }
  }'
```

### 4. Configurable Product (Color Option)
```bash
curl -X POST "http://localhost/api/registries/{REGISTRY_ID}/items" \
  -H "Content-Type: application/json" \
  -H "X-User-ID: {USER_ID}" \
  -d '{
    "item_id": "418478",
    "product_sku": "15678901-B1234567-CONFIG",
    "product_type": "configurable",
    "desired_quantity": 1,
    "priority": "high",
    "selected_config": {
      "configurable_options": [
        {
          "attribute_code": "color",
          "option_id": "925",
          "label": "Color",
          "value": "Red",
          "selected_sku": "15678901-B1234567_RED",
          "selected_product_id": "B1234567_RED"
        }
      ],
      "custom_options": [],
      "bundle_selections": []
    }
  }'
```

### 5. Configurable + Custom Product
```bash
curl -X POST "http://localhost/api/registries/{REGISTRY_ID}/items" \
  -H "Content-Type: application/json" \
  -H "X-User-ID: {USER_ID}" \
  -d '{
    "item_id": "134267",
    "product_sku": "44469079-SE_30111978-CONFIG",
    "product_type": "config_custom",
    "desired_quantity": 1,
    "priority": "medium",
    "selected_config": {
      "configurable_options": [
        {
          "attribute_code": "clothessizeexact",
          "label": "Size",
          "value": "0-3 Months",
          "selected_sku": "44469079-SE_30111978_0-6",
          "selected_product_id": "SE_30111978_0-6"
        }
      ],
      "custom_options": [
        {
          "option_id": "12345",
          "label": "Personalized Name",
          "value": "Ahmad",
          "type": "field"
        }
      ],
      "bundle_selections": []
    }
  }'
```

### 6. Bundle Product
```bash
curl -X POST "http://localhost/api/registries/{REGISTRY_ID}/items" \
  -H "Content-Type: application/json" \
  -H "X-User-ID: {USER_ID}" \
  -d '{
    "item_id": "569143",
    "product_sku": "CBNDL-35025653-2627",
    "product_type": "bundle",
    "desired_quantity": 1,
    "priority": "high",
    "selected_config": {
      "configurable_options": [],
      "custom_options": [],
      "bundle_selections": [
        {
          "option_id": "1416",
          "label": "Bath Tub",
          "selection_id": "2343",
          "selected_sku": "35025653-10227",
          "selected_product_id": "10227"
        },
        {
          "option_id": "1417",
          "label": "Bath Seat",
          "selection_id": "2344",
          "selected_sku": "35025653-10226",
          "selected_product_id": "10226"
        }
      ]
    }
  }'
```

### 7. Bundle Custom Product
```bash
curl -X POST "http://localhost/api/registries/{REGISTRY_ID}/items" \
  -H "Content-Type: application/json" \
  -H "X-User-ID: {USER_ID}" \
  -d '{
    "item_id": "item_bundle_custom_001",
    "product_sku": "CBNDL-35025653-2627",
    "product_type": "bundle_custom",
    "desired_quantity": 1,
    "priority": "medium",
    "selected_config": {
      "configurable_options": [],
      "custom_options": [
        {
          "option_id": "12345",
          "label": "Gift Message",
          "value": "Happy Birthday!",
          "type": "field"
        }
      ],
      "bundle_selections": [
        {
          "option_id": "1416",
          "label": "Bath Tub",
          "selection_id": "2343",
          "selected_sku": "35025653-10227",
          "selected_product_id": "10227"
        }
      ]
    }
  }'
```

### Get Registry with All Items
```bash
curl -X GET "http://localhost/api/registries/{REGISTRY_ID}" \
  -H "X-User-ID: {USER_ID}"
```

**Expected Response Structure:**
```json
{
  "user_id": "{USER_ID}",
  "registry_id": "{REGISTRY_ID}",
  "title": "Test Registry",
  "event_type": "birthday",
  "items": [
    {
      "item_id": "item_simple_001",
      "product_sku": "1577197-78220",
      "product_type": "simple",
      "desired_quantity": 2,
      "purchased_quantity": 0,
      "priority": "high",
      "selected_config": {...},
      "product": {
        "objectID": "1577197-78220",
        "name": "Product Name",
        "price": 99.99,
        "..." : "Algolia product data"
      }
    }
  ]
}
```

### Update Item
```bash
curl -X PUT "http://localhost/api/registries/{REGISTRY_ID}/items/{ITEM_ID}" \
  -H "Content-Type: application/json" \
  -H "X-User-ID: {USER_ID}" \
  -d '{
    "desired_quantity": 3,
    "priority": "low",
    "selected_config": {
      "custom_options": [
        {
          "option_id": "12345",
          "label": "Updated Name",
          "value": "Updated Value",
          "type": "field"
        }
      ]
    }
  }'
```

### Remove Item
```bash
curl -X DELETE "http://localhost/api/registries/{REGISTRY_ID}/items/{ITEM_ID}" \
  -H "X-User-ID: {USER_ID}"
```

### Validation Checklist
- [ ] All 6 product types can be added successfully
- [ ] Selected_config is stored and returned correctly
- [ ] Product enrichment works (product data from Algolia)
- [ ] Items can be updated and removed
- [ ] Validation errors for invalid product_type
- [ ] Registry response includes all item data + Algolia product data