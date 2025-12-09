# Performance Analytics - Visual Debugging Guide

## Console Log Flow

```
Taking a Quiz
    ↓
[quizService] Saving quiz result: {
  userId: "YOUR_USER_ID",
  quizTitle: "Quiz Name",
  score: 8/10,
  timeSpent: 300s
}
    ↓
[quizService] Quiz result saved successfully with ID: xyz789...
    ↓
✅ Data saved to Firestore
```

---

```
Opening Performance/Analytics Page
    ↓
[Analytics] Loading quiz data for user: YOUR_USER_ID
    ↓
[quizService] Fetching quiz attempts for user: YOUR_USER_ID
    ↓
[quizService] Executing Firestore query...
    ↓
[quizService] Query executed. Document count: 1
    ↓
✅ Data loaded from Firestore
    ↓
[Analytics] Number of attempts: 1
    ↓
Performance page displays quiz attempts
    ✅ SUCCESS!
```

---

## Error Flow

### Error: "requires an index"

```
[quizService] Error fetching user quiz attempts: 
  FirebaseError: The query requires an index...
    ↓
⚠️ Need to create composite index
    ↓
Firebase Console → Firestore → Indexes → Create
    ↓
Index config:
  Collection: quizAttempts
  Fields: userId (Asc) + completedAt (Desc)
    ↓
Status: Building... (1-2 minutes)
    ↓
Status: Enabled ✅
    ↓
Reload Performance page
    ↓
✅ Should work now!
```

---

### Error: "permission-denied"

```
[quizService] Error: permission-denied
    ↓
⚠️ Security rules blocking read access
    ↓
Firebase Console → Firestore → Rules
    ↓
Add/update:
  match /quizAttempts/{attemptId} {
    allow read: if request.auth != null && 
                   resource.data.userId == request.auth.uid;
  }
    ↓
Click Publish ✅
    ↓
Reload Performance page
    ↓
✅ Should work now!
```

---

### Error: "Document count = 0"

```
Page shows "No Quiz Attempts Yet"
    ↓
Check console logs...
    ↓
[quizService] Query executed. Document count: 0
    ↓
🔍 Investigate why no documents:
    ├─ Did you save logs show success?
    │  └─ NO → Data never saved (retake quiz)
    ├─ Is data in Firestore console?
    │  └─ NO → Data not saved (retake quiz)
    └─ Is userId in Firestore same as logged-in user?
       └─ NO → User ID mismatch (log in with correct account)
```

---

## Firestore Console Layout

```
Firebase Console
    ↓
Select Project: temp-doc
    ↓
Firestore Database
    ↓
Collections (left panel)
    │
    └─ quizAttempts
        ├─ doc1 (contains your quiz data)
        │   ├─ userId: "abc123xyz"
        │   ├─ quizTitle: "Cloud Migration Framework"
        │   ├─ score: 8
        │   ├─ maxScore: 10
        │   ├─ percentage: 80
        │   ├─ completedAt: Timestamp
        │   ├─ timeSpent: 300
        │   └─ answers: [...]
        │
        └─ doc2 (more quizzes...)
```

---

## Index Status Progression

```
Taking quiz with orderBy + where...
    ↓
Error: "requires an index"
    ↓
Click link in error → Firebase Console opens
    ↓
Index creation form appears:
    ├─ Collection: quizAttempts ✓
    ├─ Field 1: userId (Asc) ✓
    └─ Field 2: completedAt (Desc) ✓
    ↓
Click "Create Index" button
    ↓
Status: Building... ⏳
    ↓
Wait 1-2 minutes...
    ↓
Status: Enabled ✅ (green checkmark)
    ↓
Reload app
    ↓
✅ Query works!
```

---

## Success Indicators

### ✅ In Browser Console

```
[quizService] Quiz result saved successfully with ID: abc123...
[Analytics] Loading quiz data for user: xyz789...
[quizService] Fetching quiz attempts for user: xyz789...
[quizService] Query executed. Document count: 1
[quizService] Processing document: abc123... {userId: "xyz789", ...}
[quizService] Loaded 1 quiz attempts for user xyz789: [...]
[Analytics] Quiz attempts loaded: [{score: 8, maxScore: 10, ...}]
[Analytics] Number of attempts: 1
```

### ✅ In Firestore Console

```
Collections → quizAttempts → Contains documents
    ↓
Documents show:
  - userId matches logged-in user
  - completedAt is recent timestamp
  - score, answers fields populated
```

### ✅ On Performance Page

```
- Loading spinner appears then disappears
- "Quiz Attempts History" section visible
- Quiz appears in list with score and date
- Can click to expand details
- Questions show with checkmarks/x marks
```

---

## Common Log Messages

| Log Message | Meaning | Status |
|-------------|---------|--------|
| `[quizService] Saving quiz result:` | About to save | ℹ️ Normal |
| `[quizService] Quiz result saved successfully` | Successfully saved | ✅ Good |
| `[quizService] Query executed. Document count: X` | Found X documents | ✅ Good (if X > 0) |
| `[quizService] Loaded 1 quiz attempts` | Data loaded | ✅ Good |
| `[Analytics] Number of attempts: 1` | Analytics found data | ✅ Good |
| `Error code: permission-denied` | Security rules issue | ❌ Fix needed |
| `Error code: failed-precondition` | Index missing | ⚠️ Create index |
| `Cannot order by field` | Index missing | ⚠️ Create index |
| `Document count: 0` | No data found | ❌ Data not saved |

---

## Debug Workflow

```
1. SAVE QUIZ
   └─ Check for [quizService] saved log
   
2. CHECK FIRESTORE
   └─ Open Firebase Console
   └─ Look for data in quizAttempts
   
3. OPEN ANALYTICS
   └─ Check for [quizService] logs
   └─ Look for Document count
   
4. IF ERROR
   ├─ Index error? → Create index
   ├─ Permission error? → Update rules
   ├─ Document count 0? → Retake quiz
   └─ User mismatch? → Log in with correct account
   
5. IF STILL EMPTY
   └─ Clear cache (Ctrl+Shift+Delete)
   └─ Reload page
   └─ Reload in incognito window
```

---

## Quick Copy-Paste Solutions

### Create Firestore Index

Use this in browser console:

```javascript
// Copy exact index config needed
const indexConfig = {
  collection: 'quizAttempts',
  fields: [
    { path: 'userId', direction: 'ASCENDING' },
    { path: 'completedAt', direction: 'DESCENDING' }
  ]
};
console.log(JSON.stringify(indexConfig, null, 2));
```

### Check for Your Quizzes

```javascript
const uid = firebase.auth().currentUser?.uid;
if (uid) {
  firebase.firestore().collection('quizAttempts')
    .where('userId', '==', uid)
    .get()
    .then(snap => {
      console.log('Found', snap.size, 'quizzes');
      snap.forEach(doc => console.log(doc.data()));
    });
}
```

---

## Performance Optimization

After index is created and data loads:

- Queries complete in **< 1 second**
- Index builds in **1-2 minutes**
- Data syncs across devices in **< 5 seconds**
- No app slowdown from analytics queries

---

This visual guide covers all the common scenarios and how to diagnose them.
The enhanced logging with `[quizService]` and `[Analytics]` prefixes makes finding
issues much easier!
