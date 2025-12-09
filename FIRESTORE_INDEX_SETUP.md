# Firestore Index Setup Guide

## Required Index for Quiz Attempts Query

Your Performance Analytics page uses a query that requires a **composite index**.

### Query Being Used:
```typescript
query(
  collection(db, 'quizAttempts'),
  where('userId', '==', userId),      // Filter by user
  orderBy('completedAt', 'desc')      // Sort by date, newest first
)
```

This combination requires an index.

## How to Create the Index

### Option 1: Auto-Create (Recommended)

1. Go to your **Performance** page in the app
2. Wait for the error message in the console (if it appears)
3. Firebase Console shows a link: **"You can create a composite index at [link]"**
4. Click the link to auto-create

### Option 2: Manual Creation in Firebase Console

1. Open [Firebase Console](https://console.firebase.google.com/)
2. Select project: **temp-doc**
3. Go to **Firestore Database** → **Indexes** tab
4. Click **Create Index**

**Configure:**
- **Collection ID**: `quizAttempts`
- **Query scope**: Collection
- **Fields indexed**:
  
  | Field | Type | Order |
  |-------|------|-------|
  | userId | String | Ascending |
  | completedAt | Timestamp | Descending |

5. Click **Create Index**
6. Status shows "Building..." then "Enabled" (usually 1-2 minutes)

### Option 3: Via Firebase CLI

```bash
# Install Firebase CLI if not already done
npm install -g firebase-tools

# Login
firebase login

# Deploy indexes (if you have firestore.indexes.json)
firebase deploy --only firestore:indexes
```

## Expected Index Structure

After creation, your index should look like:

```
Collection: quizAttempts
Fields:
  1. userId (Ascending)
  2. completedAt (Descending)
Status: Enabled
Scope: Collection
```

## Verify Index is Created

### In Firebase Console:
1. Firestore Database → Indexes
2. Look for index with:
   - Collection: `quizAttempts`
   - Fields: `userId (Asc), completedAt (Desc)`
   - Status: **Enabled** (green checkmark)

### In Your App:
- Reload the Performance Analytics page
- Should load without errors
- Console should show: `[quizService] Query executed. Document count: X`

## If Index Shows "Building"

The index is being created. Wait 1-2 minutes and:
1. Don't close the Firebase Console
2. Refresh the page
3. Status should change to "Enabled"

## Troubleshooting

**"Still getting index error after creating index"**
- Wait a few minutes for full propagation
- Clear browser cache and reload
- Try in incognito/private window

**"Can't find Indexes tab"**
- Make sure you're in **Firestore Database** (not Realtime Database)
- Look for **Indexes** link in left sidebar

**"Index already exists"**
- Good! Your index is already set up
- Clear browser cache and reload app

## Automatic Index Hints

Firebase will automatically show you a link in the browser console when an index is needed:

```
⚠️ [firebase] The query requires an index. You can create a composite index at [LINK]
```

Click that link and Firebase will auto-create the index for you.

## Summary Checklist

- [ ] Open Performance Analytics page
- [ ] Check browser console for index errors
- [ ] If error, click the link or manually create index
- [ ] Index creation status: "Building" → "Enabled"
- [ ] Reload page after index is enabled
- [ ] Quiz attempts should now display
