# Quick Troubleshooting Flow - Empty Performance Analytics

## Step-by-Step Diagnosis

### 1. Check if Quiz Data Was Saved

**Action**: Take a quiz, then check browser console
- Open DevTools (F12)
- Go to **Console** tab
- Complete the quiz
- Look for this log:
  ```
  [quizService] Quiz result saved successfully with ID: abc123...
  ```

**Result**:
- ✅ **IF YOU SEE THIS**: Data is being saved. Go to Step 2.
- ❌ **IF YOU DON'T SEE THIS**: Quiz data not saving. Check quiz component. Go to Step 1B.

#### Step 1B: Why isn't quiz saving?
- Check if user is logged in
- Check browser console for errors
- Verify Firebase config in `src/config/firebase.ts`
- Check Firestore permissions in Firebase Console

---

### 2. Check If Data Exists in Firestore

**Action**: Go to Firebase Console
1. Open [Firebase Console](https://console.firebase.google.com/)
2. Select **temp-doc** project
3. Go to **Firestore Database**
4. Look for **quizAttempts** collection

**Result**:
- ✅ **IF COLLECTION EXISTS WITH DOCUMENTS**: Go to Step 3.
- ❌ **IF NO COLLECTION OR EMPTY**: Quiz data not saving to Firestore (go back to Step 1B).

---

### 3. Check If Analytics Can Access Data

**Action**: Go to Performance page, check console

1. Go to Performance/Analytics page in your app
2. Open DevTools (F12) → Console
3. Look for these logs:
   ```
   [Analytics] Loading quiz data for user: YOUR_USER_ID
   [quizService] Query executed. Document count: X
   ```

**Result**:
- ✅ **IF Document count > 0**: Data is loading! Go to Step 4.
- ⚠️ **IF Document count = 0**: User ID mismatch or wrong account. Go to Step 3B.
- ❌ **IF ERROR ABOUT INDEX**: Go to Step 5 (Create Index).
- ❌ **IF PERMISSION DENIED**: Go to Step 6 (Check Rules).

#### Step 3B: User ID Mismatch
- Make sure you took the quiz while logged in
- Make sure you're logged in with same account on Analytics page
- Check if quiz was completed successfully

---

### 4. Check If Page Displays Data

**Action**: Look at Performance page

- ✅ **IF YOU SEE QUIZ ATTEMPTS**: SUCCESS! Problem solved.
- ❌ **IF STILL EMPTY**: Possible UI rendering issue. Go to Step 4B.

#### Step 4B: UI Issue
- Try refreshing the page (Ctrl+F5)
- Clear browser cache
- Check browser console for React errors
- Try in incognito/private window

---

### 5. Create Missing Index

**IF YOU SEE** in console:
```
The query requires an index
```

**Action**:
1. Look for **blue link** in the error message
2. Click it - Firebase Console opens
3. Click **Create Index** button
4. Index builds (usually 1-2 min, shows "Building...")
5. Wait for status to change to **Enabled**
6. Reload your app page

**After index is ready**: Go back to Step 3.

---

### 6. Check Firestore Security Rules

**IF YOU SEE** in console:
```
permission-denied
```

**Action**:
1. Go to Firebase Console → Firestore Database
2. Click **Rules** tab
3. Should have:
   ```
   match /quizAttempts/{attemptId} {
     allow create: if request.auth != null && request.resource.data.userId == request.auth.uid;
     allow read: if request.auth != null && resource.data.userId == request.auth.uid;
     allow list: if request.auth != null;
     allow write: if request.auth != null && resource.data.userId == request.auth.uid;
   }
   ```
4. If different, update and **Publish**
5. Reload your app page

**After rules are updated**: Go back to Step 3.

---

## Decision Tree

```
Performance tab empty?
│
├─ Did you take a quiz? → NO → Go take a quiz
│
├─ See "[quizService] saved" log? 
│  ├─ NO → Quiz not saving (Step 1B)
│  └─ YES → Continue
│
├─ Data in Firestore console?
│  ├─ NO → Data not saved to Firestore (Step 1B)
│  └─ YES → Continue
│
├─ Analytics page logs show "Document count: 0"?
│  ├─ YES → User ID mismatch (Step 3B)
│  └─ NO → Continue
│
├─ Seeing "requires an index" error?
│  ├─ YES → Create index (Step 5)
│  └─ NO → Continue
│
├─ Seeing "permission-denied" error?
│  ├─ YES → Update rules (Step 6)
│  └─ NO → Continue
│
└─ Still empty? → Clear cache, try incognito window (Step 4B)
```

---

## Console Log Key

When debugging, watch for these prefixes:

| Prefix | Source | What it means |
|--------|--------|--------------|
| `[quizService]` | Quiz saving code | Saving/loading quiz data |
| `[Analytics]` | Performance page | Loading analytics data |
| `[firebase]` | Firebase library | Index or rule errors |

**To filter logs in console:**
1. Open Console (F12)
2. In filter box, type: `[quizService]`
3. Only quiz service logs show

---

## One-Minute Quick Fix

If analytics is empty after taking a quiz:

```
1. Open Firebase Console → Firestore → Indexes
2. Look for index: userId (Asc) + completedAt (Desc)
3. If missing, click "Create Index" link in console error
4. Wait for "Enabled" status
5. Reload your app
6. Check Performance tab again
```

Most common issue = **Missing composite index**

---

## Getting Help

If still stuck, check:
1. Browser console (F12) for error codes
2. Firestore Console for actual data
3. Firebase Rules tab for security settings
4. Index status in Indexes tab

All detailed logs are now available with `[quizService]` and `[Analytics]` prefixes for easy debugging.
