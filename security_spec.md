# Security Specification: TrackFresh Platform

## 1. Data Invariants
1. **User Identity Invariant**: A user profile document `/users/{userId}` can only be written by the authenticated user whose `request.auth.uid == userId`.
2. **Batch Registration Invariant**: New batches `/batches/{batchId}` can only be created by authenticated users. Document ID must strictly match alphanumeric/hyphen patterns (`^[a-zA-Z0-9_\\-]+$`) under 64 chars.
3. **Custody & Defect Invariant**: Quality defect reports `/defects/{defectId}` must record the reporter's authentic UID (`reportedByUid == request.auth.uid`) and target an existing batch ID.
4. **Read Access Invariant**: Batches are publicly queryable for open food transparency (anyone scanning a QR code on a supermarket shelf can inspect the farm origin and custody chain).
5. **Admin Master Override**: Mandi administrators and runtime admin (`ekanshguptta@gmail.com`) can delete, revise grades, or manage all batches.

## 2. The Dirty Dozen Payloads (Negative Test Payloads)
1. **Unauthenticated Batch Creation**: Guest attempts to create a batch without auth credentials. -> EXPECT: PERMISSION_DENIED.
2. **Path Traversal / Malicious ID Injection**: Creating batch with ID `../../root/junk` or oversized 2KB ID. -> EXPECT: PERMISSION_DENIED.
3. **User Profile Impersonation**: User A writes to `/users/userB` attempting to hijack role. -> EXPECT: PERMISSION_DENIED.
4. **Ghost Field Poisoning**: Inserting unexpected fields into batch record to bypass validation. -> EXPECT: PERMISSION_DENIED.
5. **Defect Report Spoofing**: Submitting defect report claiming another user's UID. -> EXPECT: PERMISSION_DENIED.
6. **Unauthenticated Batch Deletion**: Anonymous actor attempting to drop `/batches/TF-2026-00125`. -> EXPECT: PERMISSION_DENIED.
7. **Cross-Tenant User Modification**: Modifying user email or role without authorization. -> EXPECT: PERMISSION_DENIED.
8. **Oversized String Buffer Exhaustion**: Submitting a 50KB product string. -> EXPECT: PERMISSION_DENIED.
9. **Negative Freshness Score**: Submitting `freshnessScore: -999` or non-numeric types. -> EXPECT: PERMISSION_DENIED.
10. **Tampering with Other User's Account**: Authenticated user trying to delete another user profile. -> EXPECT: PERMISSION_DENIED.
11. **Empty Batch ID Creation**: Attempting to create batch with empty string ID. -> EXPECT: PERMISSION_DENIED.
12. **Defect Report Without Target**: Creating defect without batch ID. -> EXPECT: PERMISSION_DENIED.

## 3. Test Runner Design
The rules are validated through static helper tests and deployment checks against the standard rules emulator/engine.
