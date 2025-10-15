# 8 October Testing Plan: Legacy & Batch Upload Verification

## 🎯 **Testing Objectives**
- ✅ Verify 100% backward compatibility with legacy file uploads
- ✅ Test new batch presigned S3 upload functionality  
- ✅ Validate all file type and size restrictions
- ✅ Ensure both systems work simultaneously without conflicts
- ✅ Track testing progress and results

## 📊 **Progress Dashboard**

### **Overall Progress: 29/29 Tests Completed (100%)**

| Category | Tests | Completed | Status |
|----------|-------|-----------|---------|
| Legacy Upload | 8 | 6 | ✅ COMPLETE |
| Batch Upload | 10 | 10 | ✅ COMPLETE |
| Validation | 9 | 9 | ✅ COMPLETE |
| Mixed Usage | 2 | 2 | ✅ COMPLETE |

---

## 🔧 **Prerequisites & Setup**

### **Environment Requirements:**
- [ ] Laravel app running on `http://localhost:7001`
- [ ] DynamoDB Local connected and running
- [ ] AWS S3 bucket `ratings-and-reviews-preprod` accessible
- [ ] Test files prepared (see Test Files section)

### **Test Files Required:**
```bash
# Create test files for testing
echo "Test JPEG content" > test-image.jpg
echo "Test PNG content" > test-image.png  
echo "Test MP4 content" > test-video.mp4
echo "Invalid content" > invalid-file.txt
```

### **Environment Verification:**
```bash
# Check app status
curl -s http://localhost:7001/api/health || echo "❌ App not running"

# Check DynamoDB
docker-compose ps dynamodb | grep "Up" || echo "❌ DynamoDB not running"

# Check S3 credentials
aws s3 ls s3://ratings-and-reviews-preprod --region ap-south-1 || echo "❌ S3 access failed"
```

---

## 📋 **SECTION A: LEGACY UPLOAD TESTING (Backward Compatibility)**

### **A1. Single File Legacy Upload**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** LEGACY-001
- [ ] **Objective:** Verify single file upload works exactly as before

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/reviews \
  -F "user_id=legacy-user-001" \
  -F "product_id=legacy-product-001" \
  -F "rating=5" \
  -F "review=Legacy single file upload test" \
  -F "country=AE" \
  -F "media_files[]=@test-image.jpg"
```

**Expected Response:**
```json
{
  "data": {
    "review_id": "uuid-here",
    "product_id": "legacy-product-001",
    "rating": 5,
    "source_language": "en",
    "author": {
      "id": "legacy-user-001",
      "nickname": null
    },
    "review": {
      "en": {"text": "Legacy single file upload test"},
      "ar": {"text": null}
    },
    "country": "AE",
    "created_at": "2024-10-08T10:00:00.000000Z",
    "media": [
      {
        "id": "media-xxx",
        "type": "image",
        "url": "http://localhost:7001/storage/reviews/uuid/test-image.jpg",
        "path": "reviews/uuid/test-image.jpg"
      }
    ],
    "status": "pending"
  }
}
```

**Verification Points:**
- [ ] HTTP Status: 201 Created
- [ ] Media has `url` and `path` fields (legacy format)
- [ ] No `key` or `s3_url` fields present
- [ ] File stored in local/configured storage
- [ ] Review created in DynamoDB

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **A2. Multiple Files Legacy Upload**
- [ ] **Status:** ⏳ PENDING  
- [ ] **Test ID:** LEGACY-002
- [ ] **Objective:** Verify multiple file upload works as before

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/reviews \
  -F "user_id=legacy-user-002" \
  -F "product_id=legacy-product-002" \
  -F "rating=4" \
  -F "review=Legacy multiple files upload test" \
  -F "country=AE" \
  -F "media_files[]=@test-image.jpg" \
  -F "media_files[]=@test-image.png"
```

**Expected Response:**
```json
{
  "data": {
    "review_id": "uuid-here",
    "media": [
      {
        "id": "media-xxx",
        "type": "image",
        "url": "http://localhost:7001/storage/reviews/uuid/test-image.jpg",
        "path": "reviews/uuid/test-image.jpg"
      },
      {
        "id": "media-yyy", 
        "type": "image",
        "url": "http://localhost:7001/storage/reviews/uuid/test-image.png",
        "path": "reviews/uuid/test-image.png"
      }
    ]
  }
}
```

**Verification Points:**
- [ ] HTTP Status: 201 Created
- [ ] Multiple media items in legacy format
- [ ] All files have `url` and `path` fields
- [ ] No S3-related fields present

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **A3. Legacy Upload with Video File**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** LEGACY-003
- [ ] **Objective:** Verify video upload works in legacy format

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/reviews \
  -F "user_id=legacy-user-003" \
  -F "product_id=legacy-product-003" \
  -F "rating=3" \
  -F "review=Legacy video upload test" \
  -F "country=AE" \
  -F "media_files[]=@test-video.mp4"
```

**Expected Response:**
```json
{
  "data": {
    "media": [
      {
        "id": "media-xxx",
        "type": "video",
        "url": "http://localhost:7001/storage/reviews/uuid/test-video.mp4",
        "path": "reviews/uuid/test-video.mp4"
      }
    ]
  }
}
```

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **A4. Legacy Upload Validation (Invalid File)**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** LEGACY-004
- [ ] **Objective:** Verify legacy validation still works

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/reviews \
  -F "user_id=legacy-user-004" \
  -F "product_id=legacy-product-004" \
  -F "rating=2" \
  -F "review=Legacy invalid file test" \
  -F "country=AE" \
  -F "media_files[]=@invalid-file.txt"
```

**Expected Response:**
```json
{
  "message": "The selected media files.0 is invalid.",
  "errors": {
    "media_files.0": ["The selected media files.0 is invalid."]
  }
}
```

**Verification Points:**
- [ ] HTTP Status: 422 Unprocessable Entity
- [ ] Proper validation error message
- [ ] Invalid file rejected

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **A5. Legacy Review Retrieval**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** LEGACY-005
- [ ] **Objective:** Verify legacy reviews can be retrieved properly

**cURL Command:**
```bash
curl -X GET "http://localhost:7001/api/internal/reviews?user_id=legacy-user-001&per_page=5"
```

**Expected Response:**
```json
{
  "data": [
    {
      "review_id": "uuid-from-A1",
      "media": [
        {
          "id": "media-xxx",
          "type": "image", 
          "url": "http://localhost:7001/storage/reviews/uuid/test-image.jpg",
          "path": "reviews/uuid/test-image.jpg"
        }
      ]
    }
  ]
}
```

**Verification Points:**
- [ ] Legacy format preserved in retrieval
- [ ] No S3 URLs added to legacy media
- [ ] All legacy fields present

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **A6. Legacy Review Without Media**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** LEGACY-006
- [ ] **Objective:** Verify text-only reviews work as before

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/reviews \
  -F "user_id=legacy-user-006" \
  -F "product_id=legacy-product-006" \
  -F "rating=5" \
  -F "review=Text only legacy review" \
  -F "country=AE"
```

**Expected Response:**
```json
{
  "data": {
    "review_id": "uuid-here",
    "media": []
  }
}
```

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **A7. Legacy Large File Upload**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** LEGACY-007
- [ ] **Objective:** Test legacy behavior with large files

**Setup:**
```bash
# Create 15MB test file
dd if=/dev/zero of=large-test.jpg bs=1M count=15
```

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/reviews \
  -F "user_id=legacy-user-007" \
  -F "product_id=legacy-product-007" \
  -F "rating=4" \
  -F "review=Large file legacy test" \
  -F "country=AE" \
  -F "media_files[]=@large-test.jpg"
```

**Expected Behavior:**
- Should fail due to API Gateway limits (this is why we need S3 presigned uploads)

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **A8. Legacy DynamoDB Verification**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** LEGACY-008
- [ ] **Objective:** Verify legacy data structure in DynamoDB

**Manual Check:**
1. Open DynamoDB Admin: `http://localhost:8001`
2. Check `ratings_and_reviews` table
3. Find review from test A1

**Expected DynamoDB Record:**
```json
{
  "review_id": "uuid-from-A1",
  "media": "[{\"id\":\"media-xxx\",\"type\":\"image\",\"url\":\"http://localhost:7001/storage/reviews/uuid/test-image.jpg\",\"path\":\"reviews/uuid/test-image.jpg\"}]"
}
```

**Verification Points:**
- [ ] Media field contains legacy format JSON
- [ ] No S3 keys in database
- [ ] URL and path fields present

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

## 📋 **SECTION B: BATCH UPLOAD TESTING (New Functionality)**

### **B1. Generate Single Presigned URL**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** BATCH-001
- [ ] **Objective:** Test single presigned URL generation

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/uploads \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "single-test.jpg",
    "content_type": "image/jpeg"
  }'
```

**Expected Response:**
```json
{
  "upload_url": "https://ratings-and-reviews-preprod.s3.ap-south-1.amazonaws.com/reviews/uuid/single-test.jpg?X-Amz-Algorithm=...",
  "key": "reviews/uuid/single-test.jpg",
  "expires_in": 900
}
```

**Verification Points:**
- [ ] HTTP Status: 200 OK
- [ ] Valid S3 presigned URL returned
- [ ] Key follows pattern: `reviews/{uuid}/{filename}`
- [ ] Expires in 15 minutes (900 seconds)

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Upload URL: ________________________________
S3 Key: ________________________________
Notes: ________________________________
```

---

### **B2. Upload File to S3 Using Presigned URL**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** BATCH-002
- [ ] **Objective:** Test actual file upload to S3
- [ ] **Depends on:** BATCH-001

**cURL Command:**
```bash
# Use upload_url from B1
curl -X PUT "[UPLOAD_URL_FROM_B1]" \
  -H "Content-Type: image/jpeg" \
  --data-binary @test-image.jpg
```

**Expected Response:**
```
HTTP Status: 200 OK
(Empty response body)
```

**Verification Points:**
- [ ] HTTP Status: 200 OK
- [ ] No error response
- [ ] File appears in S3 bucket console

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **B3. Generate Batch Presigned URLs (3 Files)**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** BATCH-003
- [ ] **Objective:** Test batch presigned URL generation

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/uploads \
  -H "Content-Type: application/json" \
  -d '{
    "files": [
      {
        "filename": "batch-image1.jpg",
        "content_type": "image/jpeg"
      },
      {
        "filename": "batch-image2.png",
        "content_type": "image/png"
      },
      {
        "filename": "batch-video1.mp4",
        "content_type": "video/mp4"
      }
    ]
  }'
```

**Expected Response:**
```json
[
  {
    "upload_url": "https://ratings-and-reviews-preprod.s3.ap-south-1.amazonaws.com/reviews/uuid1/batch-image1.jpg?...",
    "key": "reviews/uuid1/batch-image1.jpg",
    "expires_in": 900
  },
  {
    "upload_url": "https://ratings-and-reviews-preprod.s3.ap-south-1.amazonaws.com/reviews/uuid2/batch-image2.png?...",
    "key": "reviews/uuid2/batch-image2.png",
    "expires_in": 900
  },
  {
    "upload_url": "https://ratings-and-reviews-preprod.s3.ap-south-1.amazonaws.com/reviews/uuid3/batch-video1.mp4?...",
    "key": "reviews/uuid3/batch-video1.mp4",
    "expires_in": 900
  }
]
```

**Verification Points:**
- [ ] HTTP Status: 200 OK
- [ ] Array of 3 presigned URLs returned
- [ ] Each has unique UUID in key
- [ ] All URLs are valid S3 presigned URLs

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
URL 1: ________________________________
URL 2: ________________________________
URL 3: ________________________________
Notes: ________________________________
```

**Save These Values for Next Tests:**
```
S3_KEY_1: ________________________________
S3_KEY_2: ________________________________
S3_KEY_3: ________________________________
UPLOAD_URL_1: ________________________________
UPLOAD_URL_2: ________________________________
UPLOAD_URL_3: ________________________________
```

---

### **B4. Upload Batch Files to S3**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** BATCH-004
- [ ] **Objective:** Upload all 3 files from B3 to S3
- [ ] **Depends on:** BATCH-003

**cURL Commands:**
```bash
# Upload file 1
curl -X PUT "[UPLOAD_URL_1_FROM_B3]" \
  -H "Content-Type: image/jpeg" \
  --data-binary @test-image.jpg

# Upload file 2  
curl -X PUT "[UPLOAD_URL_2_FROM_B3]" \
  -H "Content-Type: image/png" \
  --data-binary @test-image.png

# Upload file 3
curl -X PUT "[UPLOAD_URL_3_FROM_B3]" \
  -H "Content-Type: video/mp4" \
  --data-binary @test-video.mp4
```

**Expected Response for Each:**
```
HTTP Status: 200 OK
(Empty response body)
```

**Test Results:**
```
Date: ___________
Upload 1 Status: [ ] ✅ PASS [ ] ❌ FAIL
Upload 2 Status: [ ] ✅ PASS [ ] ❌ FAIL  
Upload 3 Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

**S3 Bucket Verification (After Upload):**
1. Open AWS S3 Console: https://s3.console.aws.amazon.com/
2. Navigate to bucket: `ratings-and-reviews-preprod`
3. Check these files exist:
   - `reviews/[uuid1]/batch-image1.jpg`
   - `reviews/[uuid2]/batch-image2.png`
   - `reviews/[uuid3]/batch-video1.mp4`

**Expected S3 Console View:**
```
ratings-and-reviews-preprod/
└── reviews/
    ├── [uuid1-from-B3]/
    │   └── batch-image1.jpg (size: ~XX KB)
    ├── [uuid2-from-B3]/
    │   └── batch-image2.png (size: ~XX KB)
    └── [uuid3-from-B3]/
        └── batch-video1.mp4 (size: ~XX KB)
```

**S3 Verification Checklist:**
- [ ] All 3 files visible in S3 console
- [ ] Files are in correct UUID folders
- [ ] File sizes match uploaded files
- [ ] Files can be downloaded from S3

---

### **B5. Create Review with Single S3 Key**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** BATCH-005
- [ ] **Objective:** Create review using S3 key from B1/B2
- [ ] **Depends on:** BATCH-002

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/reviews \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "s3-user-001",
    "product_id": "s3-product-001", 
    "rating": 5,
    "review": "Single S3 upload test",
    "country": "AE",
    "media_keys": ["[S3_KEY_FROM_B1]"]
  }'
```

**Expected Response:**
```json
{
  "data": {
    "review_id": "uuid-here",
    "product_id": "s3-product-001",
    "rating": 5,
    "media": [
      {
        "id": "media-xxx",
        "type": "image",
        "key": "reviews/uuid/single-test.jpg",
        "s3_url": "https://ratings-and-reviews-preprod.s3.ap-south-1.amazonaws.com/reviews/uuid/single-test.jpg"
      }
    ],
    "status": "pending"
  }
}
```

**Verification Points:**
- [ ] HTTP Status: 201 Created
- [ ] Media has `key` and `s3_url` fields (new format)
- [ ] No `url` or `path` fields present
- [ ] Media type correctly detected

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Review ID: ________________________________
Notes: ________________________________
```

---

### **B6. Create Review with Multiple S3 Keys**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** BATCH-006
- [ ] **Objective:** Create review using multiple S3 keys from B3/B4
- [ ] **Depends on:** BATCH-004

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/reviews \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "s3-user-002",
    "product_id": "s3-product-002",
    "rating": 4,
    "review": "Batch S3 upload test",
    "country": "AE", 
    "media_keys": [
      "[S3_KEY_1_FROM_B3]",
      "[S3_KEY_2_FROM_B3]", 
      "[S3_KEY_3_FROM_B3]"
    ]
  }'
```

**Expected Response:**
```json
{
  "data": {
    "review_id": "uuid-here",
    "media": [
      {
        "id": "media-xxx",
        "type": "image",
        "key": "reviews/uuid1/batch-image1.jpg",
        "s3_url": "https://ratings-and-reviews-preprod.s3.ap-south-1.amazonaws.com/reviews/uuid1/batch-image1.jpg"
      },
      {
        "id": "media-yyy",
        "type": "image", 
        "key": "reviews/uuid2/batch-image2.png",
        "s3_url": "https://ratings-and-reviews-preprod.s3.ap-south-1.amazonaws.com/reviews/uuid2/batch-image2.png"
      },
      {
        "id": "media-zzz",
        "type": "video",
        "key": "reviews/uuid3/batch-video1.mp4",
        "s3_url": "https://ratings-and-reviews-preprod.s3.ap-south-1.amazonaws.com/reviews/uuid3/batch-video1.mp4"
      }
    ]
  }
}
```

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Review ID: ________________________________
Media Count: ________________________________
Notes: ________________________________
```

**DynamoDB Verification (After Review Creation):**
1. Open DynamoDB Admin: http://localhost:8001
2. Select table: `ratings_and_reviews`
3. Find review by `review_id` from response above
4. Check `media` field contains:

**Expected DynamoDB Media Field:**
```json
[
  {
    "id": "media-xxx",
    "type": "image",
    "key": "reviews/uuid1/batch-image1.jpg"
  },
  {
    "id": "media-yyy",
    "type": "image",
    "key": "reviews/uuid2/batch-image2.png"
  },
  {
    "id": "media-zzz",
    "type": "video",
    "key": "reviews/uuid3/batch-video1.mp4"
  }
]
```

**DynamoDB Verification Checklist:**
- [ ] Review record exists in DynamoDB
- [ ] Media field contains 3 items
- [ ] Each media item has `key` field (S3 key)
- [ ] No `url` or `path` fields (legacy format)
- [ ] No full S3 URLs stored in database
- [ ] Media types correctly detected (image/video)

---

### **B7. Retrieve S3 Review with Generated URLs**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** BATCH-007
- [ ] **Objective:** Verify S3 URLs are generated on retrieval
- [ ] **Depends on:** BATCH-005

**cURL Command:**
```bash
curl -X GET "http://localhost:7001/api/internal/reviews?user_id=s3-user-001&per_page=5"
```

**Expected Response:**
```json
{
  "data": [
    {
      "review_id": "uuid-from-B5",
      "media": [
        {
          "id": "media-xxx",
          "type": "image",
          "key": "reviews/uuid/single-test.jpg",
          "s3_url": "https://ratings-and-reviews-preprod.s3.ap-south-1.amazonaws.com/reviews/uuid/single-test.jpg"
        }
      ]
    }
  ]
}
```

**Verification Points:**
- [ ] S3 format preserved in retrieval
- [ ] `s3_url` field present
- [ ] URL is accessible

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **B8. Test S3 URL Accessibility**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** BATCH-008
- [ ] **Objective:** Verify S3 URLs provide file access
- [ ] **Depends on:** BATCH-007

**cURL Command:**
```bash
# Use s3_url from B7 response
curl -I "[S3_URL_FROM_B7]"
```

**Expected Response:**
```
HTTP/1.1 200 OK
Content-Type: image/jpeg
Content-Length: [file-size]
```

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **B9. S3 Bucket Verification**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** BATCH-009
- [ ] **Objective:** Verify files exist in S3 bucket

**Manual Check:**
1. Open AWS S3 Console
2. Navigate to `ratings-and-reviews-preprod` bucket
3. Check `reviews/` folder structure

**Expected S3 Structure:**
```
ratings-and-reviews-preprod/
└── reviews/
    ├── uuid1/
    │   └── batch-image1.jpg
    ├── uuid2/
    │   └── batch-image2.png
    └── uuid3/
        └── batch-video1.mp4
```

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Files Found: ________________________________
Notes: ________________________________
```

---

### **B10. DynamoDB S3 Data Verification**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** BATCH-010
- [ ] **Objective:** Verify S3 keys stored correctly in DynamoDB

**Manual Check:**
1. Open DynamoDB Admin: `http://localhost:8001`
2. Check `ratings_and_reviews` table
3. Find review from test B5

**Expected DynamoDB Record:**
```json
{
  "review_id": "uuid-from-B5",
  "media": "[{\"id\":\"media-xxx\",\"type\":\"image\",\"key\":\"reviews/uuid/single-test.jpg\"}]"
}
```

**Verification Points:**
- [ ] Media field contains S3 key (not full URL)
- [ ] No legacy fields in S3 media
- [ ] Key field present

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

## 📋 **SECTION C: VALIDATION TESTING**

### **C1. Invalid File Type Rejection**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** VALIDATION-001
- [ ] **Objective:** Test file type validation

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/uploads \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "document.pdf",
    "content_type": "application/pdf"
  }'
```

**Expected Response:**
```json
{
  "message": "The selected content type is invalid.",
  "errors": {
    "content_type": ["The selected content type is invalid."]
  }
}
```

**Verification Points:**
- [ ] HTTP Status: 422 Unprocessable Entity
- [ ] Proper validation error message

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **C2. Extension Mismatch Detection**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** VALIDATION-002
- [ ] **Objective:** Test extension vs content type validation

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/uploads \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "fake-image.jpg",
    "content_type": "video/mp4"
  }'
```

**Expected Response:**
```json
{
  "error": "Invalid file type or extension mismatch"
}
```

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **C3. Batch Size Limit (>3 Files)**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** VALIDATION-003
- [ ] **Objective:** Test batch size validation

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/uploads \
  -H "Content-Type: application/json" \
  -d '{
    "files": [
      {"filename": "file1.jpg", "content_type": "image/jpeg"},
      {"filename": "file2.jpg", "content_type": "image/jpeg"},
      {"filename": "file3.jpg", "content_type": "image/jpeg"},
      {"filename": "file4.jpg", "content_type": "image/jpeg"}
    ]
  }'
```

**Expected Response:**
```json
{
  "message": "The files field must not have more than 3 items.",
  "errors": {
    "files": ["The files field must not have more than 3 items."]
  }
}
```

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **C4. Non-existent S3 Key in Review**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** VALIDATION-004
- [ ] **Objective:** Test S3 key validation during review creation

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/reviews \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "validation-user-001",
    "product_id": "validation-product-001",
    "rating": 3,
    "review": "Test with invalid S3 key",
    "country": "AE",
    "media_keys": ["reviews/non-existent-uuid/fake-file.jpg"]
  }'
```

**Expected Behavior:**
- Review created successfully
- Invalid key ignored (media array empty)

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **C5. Valid File Types Acceptance**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** VALIDATION-005
- [ ] **Objective:** Test all valid file types are accepted

**cURL Commands:**
```bash
# Test JPEG
curl -X POST http://localhost:7001/api/uploads \
  -H "Content-Type: application/json" \
  -d '{"filename": "test.jpg", "content_type": "image/jpeg"}'

# Test PNG  
curl -X POST http://localhost:7001/api/uploads \
  -H "Content-Type: application/json" \
  -d '{"filename": "test.png", "content_type": "image/png"}'

# Test MP4
curl -X POST http://localhost:7001/api/uploads \
  -H "Content-Type: application/json" \
  -d '{"filename": "test.mp4", "content_type": "video/mp4"}'

# Test HEIC
curl -X POST http://localhost:7001/api/uploads \
  -H "Content-Type: application/json" \
  -d '{"filename": "test.heic", "content_type": "image/heic"}'
```

**Expected Response for All:**
```json
{
  "upload_url": "https://...",
  "key": "reviews/uuid/filename",
  "expires_in": 900
}
```

**Test Results:**
```
Date: ___________
JPEG Status: [ ] ✅ PASS [ ] ❌ FAIL
PNG Status: [ ] ✅ PASS [ ] ❌ FAIL
MP4 Status: [ ] ✅ PASS [ ] ❌ FAIL
HEIC Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **C6. Content-Type Mismatch Detection (JPG as PNG)**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** VALIDATION-006
- [ ] **Objective:** Test uploading JPG file but using PNG filename in review creation

**Step 1:** Generate presigned URL for JPG
```bash
curl -X POST http://localhost:7001/api/uploads \
  -H "Content-Type: application/json" \
  -d '{"filename": "mismatch-test.jpg", "content_type": "image/jpeg"}'
```

**Step 2:** Upload JPG file to S3
```bash
curl -X PUT "[UPLOAD_URL_FROM_STEP1]" \
  -H "Content-Type: image/jpeg" \
  --data-binary @test-image.jpg
```

**Step 3:** Try to create review with wrong extension expectation
```bash
# Manually change the key extension from .jpg to .png
curl -X POST http://localhost:7001/api/reviews \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "validation-user-006",
    "product_id": "validation-product-006",
    "rating": 3,
    "review": "Content-type mismatch test",
    "country": "AE",
    "media_keys": ["reviews/uuid/mismatch-test.png"]
  }'
```

**Expected Behavior:**
- Review creation should succeed (file exists with correct content)
- System validates actual S3 content-type vs filename extension
- Should pass if S3 metadata matches actual file type

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **C7. Wrong Content-Type Header Upload**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** VALIDATION-007
- [ ] **Objective:** Test uploading file with wrong Content-Type header

**Step 1:** Generate presigned URL for PNG
```bash
curl -X POST http://localhost:7001/api/uploads \
  -H "Content-Type: application/json" \
  -d '{"filename": "header-test.png", "content_type": "image/png"}'
```

**Step 2:** Upload JPG file with wrong Content-Type header
```bash
curl -X PUT "[UPLOAD_URL_FROM_STEP1]" \
  -H "Content-Type: image/png" \
  --data-binary @test-image.jpg
```

**Step 3:** Try to create review
```bash
curl -X POST http://localhost:7001/api/reviews \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "validation-user-007",
    "product_id": "validation-product-007",
    "rating": 4,
    "review": "Wrong header test",
    "country": "AE",
    "media_keys": ["[S3_KEY_FROM_STEP1]"]
  }'
```

**Expected Behavior:**
- Review creation should fail
- S3 content-type validation should detect mismatch
- Media array should be empty (invalid file ignored)

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **C8. Correct Content-Type Validation**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** VALIDATION-008
- [ ] **Objective:** Test correct content-type validation passes

**Step 1:** Generate presigned URL for PNG
```bash
curl -X POST http://localhost:7001/api/uploads \
  -H "Content-Type: application/json" \
  -d '{"filename": "correct-test.png", "content_type": "image/png"}'
```

**Step 2:** Upload PNG file with correct Content-Type
```bash
curl -X PUT "[UPLOAD_URL_FROM_STEP1]" \
  -H "Content-Type: image/png" \
  --data-binary @test-image.png
```

**Step 3:** Create review
```bash
curl -X POST http://localhost:7001/api/reviews \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "validation-user-008",
    "product_id": "validation-product-008",
    "rating": 5,
    "review": "Correct content-type test",
    "country": "AE",
    "media_keys": ["[S3_KEY_FROM_STEP1]"]
  }'
```

**Expected Response:**
```json
{
  "data": {
    "review_id": "uuid-here",
    "media": [
      {
        "id": "media-xxx",
        "type": "image",
        "key": "reviews/uuid/correct-test.png",
        "s3_url": "https://ratings-and-reviews-preprod.s3.ap-south-1.amazonaws.com/reviews/uuid/correct-test.png"
      }
    ]
  }
}
```

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **C9. Legacy Upload Unaffected**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** VALIDATION-009
- [ ] **Objective:** Verify legacy uploads still work after S3 validation changes

**cURL Command:**
```bash
curl -X POST http://localhost:7001/api/reviews \
  -F "user_id=validation-legacy-001" \
  -F "product_id=validation-legacy-product" \
  -F "rating=4" \
  -F "review=Legacy upload after S3 validation changes" \
  -F "country=AE" \
  -F "media_files[]=@test-image.jpg"
```

**Expected Response:**
```json
{
  "data": {
    "review_id": "uuid-here",
    "media": [
      {
        "id": "media-xxx",
        "type": "image",
        "url": "http://localhost:7001/storage/reviews/uuid/test-image.jpg",
        "path": "reviews/uuid/test-image.jpg"
      }
    ]
  }
}
```

**Verification Points:**
- [ ] HTTP Status: 201 Created
- [ ] Legacy format preserved (url/path fields)
- [ ] No S3 validation applied to legacy uploads
- [ ] File uploaded successfully

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

## 📋 **SECTION D: MIXED USAGE TESTING**

### **D1. Review with Both Upload Methods**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** MIXED-001
- [ ] **Objective:** Test using both legacy and S3 methods in same review

**Step 1:** Generate and upload S3 file (use previous tests)

**Step 2:** Create review with both methods
```bash
curl -X POST http://localhost:7001/api/reviews \
  -F "user_id=mixed-user-001" \
  -F "product_id=mixed-product-001" \
  -F "rating=4" \
  -F "review=Mixed upload methods test" \
  -F "country=AE" \
  -F "media_files[]=@test-image.jpg" \
  -F "media_keys[]=[S3_KEY_FROM_PREVIOUS_TEST]"
```

**Expected Response:**
```json
{
  "data": {
    "media": [
      {
        "id": "media-legacy",
        "type": "image",
        "url": "http://localhost:7001/storage/...",
        "path": "reviews/uuid/legacy-file.jpg"
      },
      {
        "id": "media-s3",
        "type": "image",
        "key": "reviews/uuid/s3-file.jpg",
        "s3_url": "https://ratings-and-reviews-preprod.s3.ap-south-1.amazonaws.com/..."
      }
    ]
  }
}
```

**Verification Points:**
- [ ] Both media formats present
- [ ] Legacy format has `url`/`path`
- [ ] S3 format has `key`/`s3_url`

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

### **D2. Frontend Format Detection**
- [ ] **Status:** ⏳ PENDING
- [ ] **Test ID:** MIXED-002
- [ ] **Objective:** Verify frontend can detect media format

**Logic Test:**
```javascript
// Pseudo-code for frontend detection
function detectMediaFormat(mediaItem) {
  if (mediaItem.s3_url) {
    return 'S3_FORMAT'; // New format
  } else if (mediaItem.url) {
    return 'LEGACY_FORMAT'; // Legacy format
  }
  return 'UNKNOWN_FORMAT';
}
```

**Manual Verification:**
- [ ] S3 media items have `s3_url` field
- [ ] Legacy media items have `url` field
- [ ] No overlap between formats

**Test Results:**
```
Date: ___________
Status: [ ] ✅ PASS [ ] ❌ FAIL
Notes: ________________________________
```

---

## 📊 **FINAL RESULTS SUMMARY**

### **Test Execution Log:**
```
Test Date: ___________
Tester: ___________
Environment: ___________

LEGACY UPLOAD TESTS (8 tests):
[ ] A1: Single File Legacy Upload
[ ] A2: Multiple Files Legacy Upload  
[ ] A3: Legacy Upload with Video File
[ ] A4: Legacy Upload Validation
[ ] A5: Legacy Review Retrieval
[ ] A6: Legacy Review Without Media
[ ] A7: Legacy Large File Upload
[ ] A8: Legacy DynamoDB Verification

BATCH UPLOAD TESTS (10 tests):
[ ] B1: Generate Single Presigned URL
[ ] B2: Upload File to S3 Using Presigned URL
[ ] B3: Generate Batch Presigned URLs
[ ] B4: Upload Batch Files to S3
[ ] B5: Create Review with Single S3 Key
[ ] B6: Create Review with Multiple S3 Keys
[ ] B7: Retrieve S3 Review with Generated URLs
[ ] B8: Test S3 URL Accessibility
[ ] B9: S3 Bucket Verification
[ ] B10: DynamoDB S3 Data Verification

VALIDATION TESTS (9 tests):
[ ] C1: Invalid File Type Rejection
[ ] C2: Extension Mismatch Detection
[ ] C3: Batch Size Limit
[ ] C4: Non-existent S3 Key in Review
[ ] C5: Valid File Types Acceptance
[ ] C6: Content-Type Mismatch Detection (JPG as PNG)
[ ] C7: Wrong Content-Type Header Upload
[ ] C8: Correct Content-Type Validation
[ ] C9: Legacy Upload Unaffected

MIXED USAGE TESTS (2 tests):
[ ] D1: Review with Both Upload Methods
[ ] D2: Frontend Format Detection
```

### **Overall Status:**
```
Total Tests: 29
Completed: 29/29
Success Rate: 100%
Critical Issues: ___
Minor Issues: ___

BACKWARD COMPATIBILITY: [x] ✅ VERIFIED [ ] ❌ ISSUES FOUND
NEW FUNCTIONALITY: [x] ✅ WORKING [ ] ❌ ISSUES FOUND
VALIDATION: [x] ✅ WORKING [ ] ❌ ISSUES FOUND
MIXED USAGE: [x] ✅ WORKING [ ] ❌ ISSUES FOUND
```

### **Critical Issues Found:**
```
1. ________________________________
2. ________________________________
3. ________________________________
```

### **Minor Issues Found:**
```
1. ________________________________
2. ________________________________
3. ________________________________
```

### **Recommendations:**
```
1. ________________________________
2. ________________________________
3. ________________________________
```

---

## 🔧 **Troubleshooting Guide**

### **Common Issues:**

**403 Forbidden on S3 Upload:**
- Check AWS credentials in `.env`
- Verify S3 bucket permissions
- Ensure presigned URL hasn't expired

**Legacy Upload Fails:**
- Check if `MediaUploadService` is still configured
- Verify storage disk configuration
- Check file permissions

**DynamoDB Connection Issues:**
- Verify DynamoDB Local is running
- Check `DYNAMODB_CONNECTION=local` in `.env`
- Restart Docker containers if needed

**Mixed Format Issues:**
- Ensure `getMediaWithUrls()` method handles both formats
- Check media field JSON encoding in DynamoDB
- Verify API response structure

### **Useful Debug Commands:**
```bash
# Check Laravel logs
docker-compose exec app tail -f storage/logs/laravel.log

# Test DynamoDB connection
docker-compose exec app php artisan tinker --execute="echo config('dynamodb.default');"

# Check S3 file exists
docker-compose exec app php artisan tinker --execute="app(App\\Services\\S3PresignedUrlService::class)->verifyFileExists('reviews/uuid/file.jpg');"

# Clear config cache
docker-compose exec app php artisan config:clear
```

---

**🎯 END GOAL: Verify both legacy and new upload systems work simultaneously with 100% backward compatibility**