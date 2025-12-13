# Edit Request Creator Modes

This document explains the different modes for creating edit requests (Pull Requests) in the wiki.

## Configuration

Add this to your `wiki-config.json`:

```json
{
  "features": {
    "editRequestCreator": {
      "mode": "auto",
      "forks": {
        "enabled": true,
        "autoSync": true,
        "keepForkUpdated": true
      },
      "anonymous": {
        "enabled": false,
        "serverEndpoint": "",
        "attributionFormat": "Contributed anonymously via wiki editor"
      },
      "permissions": {
        "requireAuth": true,
        "fallbackToFork": true
      }
    }
  }
}
```

---

## Modes

### Mode: "auto" (Recommended) ⭐

Automatically selects the best workflow based on user permissions:

**For users WITH write access:**
- Creates branch directly on main repository
- Creates PR from branch to main
- Fast and simple workflow

**For users WITHOUT write access:**
- Creates/uses user's fork of the repository
- Syncs fork with upstream (if enabled)
- Creates branch on user's fork
- Creates PR from fork branch to upstream main
- Standard open-source contribution model

**Flow:**
```
User clicks Edit
  ↓
Authenticated?
  ↓ Yes
Check permissions on main repo
  ↓
Has write access?
  ├─ Yes → Create branch on main repo → Create PR
  └─ No  → Get/create fork → Sync fork → Create branch on fork → Create PR
```

---

### Mode: "fork-only"

Always uses fork workflow, even if user has write access:

**Benefits:**
- Consistent workflow for all users
- Keeps main repository clean
- Good for projects with many contributors

**Drawbacks:**
- Slower for users with write access
- Requires fork creation and maintenance

---

### Mode: "branch-only"

Only allows users with write access to edit:

**Benefits:**
- Simple workflow
- No fork management needed

**Drawbacks:**
- Excludes external contributors
- Requires all contributors to have write access

---

### Mode: "anonymous-bot"

Allows anonymous edits using a bot account:

**Requirements:**
- Backend server to hold bot credentials securely
- Bot GitHub account with write access to repository

**Flow:**
```
User clicks "Edit Anonymously"
  ↓
Makes changes in editor
  ↓
Submit → Sends to backend server
  ↓
Backend creates PR using bot account
  ↓
PR description includes: "Contributed anonymously via wiki editor"
```

**Security Notes:**
- ⚠️ Bot's Personal Access Token must NEVER be exposed to client
- ⚠️ Backend must validate and sanitize all content
- ⚠️ Consider rate limiting anonymous edits
- ⚠️ Review anonymous contributions carefully

---

## Fork Management

### Auto-Sync

When `forks.autoSync: true`:
- Before creating new edit requests, syncs user's fork with upstream
- Prevents conflicts from outdated forks
- Uses GitHub's merge-upstream API

### Keep Fork Updated

When `forks.keepForkUpdated: true`:
- Periodically checks if fork is behind upstream
- Shows notification to user if fork needs syncing
- Provides "Sync Fork" button in UI

---

## Permission Levels

### Admin
- Full access to repository
- Can merge PRs, manage settings
- Uses branch workflow

### Write
- Can push to repository
- Can create branches
- Uses branch workflow

### Read
- Can view repository
- Cannot push changes
- Must use fork workflow

### None
- No access to repository
- Can still contribute via fork (public repos)
- Must authenticate with GitHub

---

## Finding Existing PRs

The system searches for existing PRs in two locations:

1. **Main repository:** `wiki-edit/<section>/<page-id>-*`
2. **User's fork:** `username:wiki-edit/<section>/<page-id>-*`

This ensures that:
- Users with write access can update their direct PRs
- Users without write access can update their fork PRs
- No duplicate PRs are created for the same page

---

## Anonymous Contributions

### When Enabled

Edit button shows two options:
- "Edit (Sign In)" - Uses authenticated workflow
- "Edit Anonymously" - Uses bot account

### Backend Requirements

Your backend endpoint should:

**POST** to `{serverEndpoint}`

**Request Body:**
```json
{
  "section": "getting-started",
  "pageId": "first-steps",
  "content": "markdown content...",
  "editSummary": "Fixed typo in introduction",
  "filePath": "content/getting-started/first-steps.md",
  "metadata": {
    "id": "first-steps",
    "title": "First Steps"
  }
}
```

**Response:**
```json
{
  "success": true,
  "prNumber": 123,
  "prUrl": "https://github.com/owner/repo/pull/123"
}
```

**Backend Implementation (Node.js example):**
```javascript
const { Octokit } = require('@octokit/rest');

const octokit = new Octokit({
  auth: process.env.BOT_GITHUB_TOKEN // Bot's PAT
});

app.post('/api/anonymous-pr', async (req, res) => {
  const { section, pageId, content, editSummary, filePath } = req.body;

  // Validate and sanitize inputs
  // ...

  // Create branch
  const branchName = `wiki-edit/${section}/${pageId}-${Date.now()}`;
  // Create branch, commit, PR
  // ...

  res.json({ success: true, prUrl: pr.html_url });
});
```

---

## Prestige System

### With Authenticated Users
- Contributions count toward prestige
- Users get credited properly
- Prestige tiers work normally

### With Anonymous Bot
- All PRs appear under bot account
- Real users not credited in prestige system
- Can attribute in PR description only

**Recommendation:** Use authenticated workflows when possible to maintain proper attribution and prestige tracking.

---

## Testing Strategy

### Test Write Access User
1. Create edit request
2. Verify branch created on main repo
3. Verify PR created from main repo
4. Make second edit to same page
5. Verify commit added to existing PR

### Test No-Access User
1. Create edit request
2. Verify fork created (or existing fork found)
3. Verify fork synced with upstream
4. Verify branch created on fork
5. Verify PR created from fork to upstream
6. Make second edit to same page
7. Verify commit added to existing fork PR

### Test Anonymous User
1. Click "Edit Anonymously"
2. Make changes
3. Submit
4. Verify backend called correctly
5. Verify PR created by bot account
6. Verify attribution in PR description

---

## Troubleshooting

### "Failed to create fork"
- Check if user has already reached fork limit (rare)
- Verify repository is public or user has access
- Check GitHub API rate limits

### "Failed to sync fork"
- Fork may have diverged too much from upstream
- User may need to manually resolve conflicts
- Try deleting and recreating fork

### "Permission denied"
- User doesn't have access to repository
- User's token doesn't have required scopes
- Repository requires specific permissions

### "PR not found on second edit"
- Branch naming may have changed
- User may have closed/merged the PR
- Check fork vs main repo PR locations

---

## Migration Guide

### From Current (Branch-Only) to Auto Mode

1. Update config:
```json
{
  "editRequestCreator": {
    "mode": "auto",
    "forks": {
      "enabled": true,
      "autoSync": true
    }
  }
}
```

2. Existing PRs continue to work
3. New users without write access will use forks
4. Existing users with write access continue using branches

**No breaking changes!**

---

## Best Practices

### For Open Source Wikis (Public Repos)
```json
{
  "mode": "auto",
  "forks": {
    "enabled": true,
    "autoSync": true
  },
  "permissions": {
    "requireAuth": true,
    "fallbackToFork": true
  }
}
```

### For Private Team Wikis
```json
{
  "mode": "branch-only",
  "forks": {
    "enabled": false
  },
  "permissions": {
    "requireAuth": true,
    "fallbackToFork": false
  }
}
```

### For Community Wikis with Anonymous Edits
```json
{
  "mode": "auto",
  "forks": {
    "enabled": true,
    "autoSync": true
  },
  "anonymous": {
    "enabled": true,
    "serverEndpoint": "https://your-backend.com/api/anonymous-pr"
  },
  "permissions": {
    "requireAuth": false,
    "fallbackToFork": true
  }
}
```

---

## Security Considerations

### Bot Token Security
- ⚠️ **NEVER** expose bot token in client-side code
- ⚠️ Store in environment variables only
- ⚠️ Use fine-grained tokens with minimal permissions
- ⚠️ Rotate tokens regularly

### Anonymous Edits
- ⚠️ Validate all content server-side
- ⚠️ Sanitize markdown to prevent XSS
- ⚠️ Rate limit by IP address
- ⚠️ Consider moderation queue for anonymous PRs
- ⚠️ Log all anonymous contributions

### Fork Management
- ✅ Forks are safe - users control their own forks
- ✅ Syncing only pulls from upstream (safe operation)
- ✅ No credentials needed beyond user's GitHub auth

---

## Performance Considerations

### Fork Creation
- First edit: ~3-5 seconds (fork creation + sync)
- Subsequent edits: ~1-2 seconds (sync only)

### Branch Creation
- ~1 second (no fork needed)

### Recommendation
- Cache user's fork status in localStorage
- Show loading states during fork operations
- Display progress: "Creating your fork..." → "Syncing with upstream..." → "Creating branch..."
