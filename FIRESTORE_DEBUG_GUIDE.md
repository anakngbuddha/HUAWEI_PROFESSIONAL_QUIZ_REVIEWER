# Firestore Debug Guide - Quiz Attempts Not Showing

## Issue: Performance Analytics is empty after taking a quiz

### Step 1: Check Firestore Console

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Select your project: **temp-doc**
3. Go to **Firestore Database**
4. Look for the **quizAttempts** collection

### Step 2: Verify Quiz Data is Being Saved

**In your browser (while on the quiz page):**

1. Open **Developer Tools** (F12 or Ctrl+Shift+I)
2. Go to **Console** tab
3. Complete a quiz
4. Look for logs starting with `[quizService]`:
   - `[quizService] Saving quiz result:` - Shows what's being saved
   - `[quizService] Quiz result saved successfully with ID:` - Confirms save worked

**What to look for:**
```
[quizService] Saving quiz result: {
  userId: "YOUR_USER_ID",
  quizId: "...",
  quizTitle: "...",
  score: 8,
  maxScore: 10,
  answersCount: 10,
  timeSpent: 300
}
```

If you see this log, the data IS being saved to Firestore.

### Step 3: Check Analytics Page Logs

**On the Analytics/Performance page:**

1. Open Developer Tools Console (F12)
2. Look for logs starting with `[Analytics]` and `[quizService]`:
   - `[Analytics] Loading quiz data for user: YOUR_USER_ID`
   - `[Analytics] Number of attempts: X`
   - `[quizService] Firestore query executed. Document count: X`

**Common Issues:**

| Log Message | Meaning | Solution |
|-------------|---------|----------|
| `Document count: 0` | No data found for user | Check Firestore directly, verify userId matches |
| `Error code: permission-denied` | Security rules blocking read | Check Firestore rules (see Step 5) |
| `Error code: failed-precondition` | **Missing composite index** | Create index (see Step 4) |
| `Cannot order by field` | Missing index | Create index (see Step 4) |

### Step 4: Create Composite Index (if needed)

If you see this error in the console:
```
failed-precondition: The query requires an index
```

Or if you see this message:
```
Cannot order by field after orderBy(completedAt)
```

**Create a composite index:**

1. In Firebase Console → Firestore Database
2. Click on **Indexes** tab
3. Click **Create Index** if not auto-prompted
4. Fill in:
   - **Collection ID**: `quizAttempts`
   - **Fields to index**:
     - `userId` (Ascending)
     - `completedAt` (Descending)
5. Click **Create Index**
6. Wait for index to build (usually 1-2 minutes)

**Or** Firebase may show you a link in the console error - click it to auto-create the index.

### Step 5: Verify Firestore Security Rules

Go to **Firestore Database** → **Rules** tab

You should see:
```firestore
match /quizAttempts/{attemptId} {
  allow create: if request.auth != null && request.resource.data.userId == request.auth.uid;
  allow read: if request.auth != null && resource.data.userId == request.auth.uid;
  allow list: if request.auth != null;
  allow write: if request.auth != null && resource.data.userId == request.auth.uid;
}
```

**If rules are different or missing**, update them and publish.

### Step 6: Verify Data in Firestore Console

1. Open Firebase Console
2. Go to Firestore Database
3. Click on **quizAttempts** collection
4. You should see documents that look like:

```
Document ID: abc123xyz...
Fields:
  ├─ userId: "user123"
  ├─ quizTitle: "Cloud Migration Framework"
  ├─ score: 8
  ├─ maxScore: 10
  ├─ percentage: 80
  ├─ completedAt: Dec 9, 2025 at 3:45:30 PM
  ├─ timeSpent: 300
  └─ answers: [array of 10 items]
```

**Check that:**
- ✅ `userId` field matches your logged-in user ID
- ✅ `completedAt` field exists and is a Timestamp
- ✅ `answers` array has data

### Troubleshooting Checklist

- [ ] Quiz save logs appear in console (`[quizService] Quiz result saved`)
- [ ] Data appears in Firestore quizAttempts collection
- [ ] userId in saved data matches current logged-in user ID
- [ ] No permission-denied errors in console
- [ ] Composite index exists (if error mentioned it)
- [ ] Analytics page shows correct user ID in logs
- [ ] Quiz attempts are being loaded (check `Document count: X`)

### Common Solutions

**Problem: userId mismatch**
- Make sure you're logged in with same account that took the quiz
- Check localStorage/session for user ID changes

**Problem: Index missing**
- Firebase should auto-create or show link in error
- Create composite index manually (Step 4)

**Problem: Permission denied**
- Review Firestore rules
- Make sure user is authenticated
- Check Firebase Auth is working

**Problem: Still empty after all checks**
- Clear browser cache (Ctrl+Shift+Delete)
- Reload the page (Ctrl+F5)
- Check browser console for network errors
- Verify Firebase project is accessible

### Advanced Debugging

**In browser console, run:**

```javascript
// Check current user
firebase.auth().currentUser  // Should show user object

// Check if Firestore is initialized
firebase.firestore()  // Should work

// Try manual query
firebase.firestore().collection('quizAttempts')
  .where('userId', '==', firebase.auth().currentUser.uid)
  .orderBy('completedAt', 'desc')
  .get()
  .then(snap => console.log('Documents:', snap.size))
  .catch(err => console.error('Error:', err))
```

### Need More Help?

Check these files for issues:
1. `src/services/quizService.ts` - Quiz data saving logic
2. `src/pages/quick-navigation/Analytics.tsx` - Data loading logic
3. `firestore.rules` - Security rules
4. Browser Console - Error messages and logs

**Enhanced logging is now enabled** - All logs are prefixed with `[quizService]` or `[Analytics]` for easy filtering.
