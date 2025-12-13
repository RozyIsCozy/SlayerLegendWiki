# Fork-Based Edit Requests - Implementation Plan

## ✅ Completed

### 1. Configuration Structure
- Added `editRequestCreator` config to wiki-config.json
- Supports modes: "auto", "fork-only", "branch-only", "anonymous-bot"
- Fork settings: autoSync, keepForkUpdated
- Anonymous settings: serverEndpoint, attribution
- Permission settings: requireAuth, fallbackToFork

### 2. Permission Service
Created `src/services/github/permissions.js`:
- `getUserPermission()` - Get user's permission level
- `hasWriteAccess()` - Check if user can push
- `hasAdminAccess()` - Check if user has admin rights

### 3. Fork Management Service
Created `src/services/github/forks.js`:
- `getUserFork()` - Check if user has fork
- `createFork()` - Create fork for user
- `syncForkWithUpstream()` - Sync fork with upstream
- `getOrCreateFork()` - Combined operation with auto-sync

### 4. Documentation
- `EDIT_REQUEST_MODES.md` - Complete usage guide
- `IMPLEMENTATION_PLAN.md` - This file

---

## 🔄 Next Steps

### Step 1: Update Branch Creation
**File:** `src/services/github/branches.js`

Add support for creating branches on forks:
```javascript
export const createBranchOnRepo = async (
  owner,      // Could be fork owner or main repo owner
  repo,       // Could be fork name or main repo name
  branchName,
  baseBranch = 'main'
) => {
  // Existing logic, but works with any owner/repo
};
```

### Step 2: Update Content Service
**File:** `src/services/github/content.js`

Ensure `updateFileContent()` works with any owner/repo:
```javascript
export const updateFileContent = async (
  owner,      // Could be fork owner
  repo,       // Could be fork repo
  filePath,
  content,
  commitMessage,
  branchName,
  fileSha = null
) => {
  // Should already work, just verify
};
```

### Step 3: Update PR Creation
**File:** `src/services/github/pullRequests.js`

Add `createCrossRepoPR()` function:
```javascript
export const createCrossRepoPR = async (
  upstreamOwner,  // Main repo owner
  upstreamRepo,   // Main repo name
  headOwner,      // Fork owner (username)
  headBranch,     // Branch name on fork
  title,
  body,
  baseBranch = 'main'
) => {
  // Create PR from fork to upstream
  // head format: "username:branch-name"
};
```

### Step 4: Update PR Search
**File:** `src/services/github/pullRequests.js`

Update `findExistingPRForPage()` to search both:
- Main repo PRs: `wiki-edit/<section>/<page-id>-*`
- Fork PRs: `username:wiki-edit/<section>/<page-id>-*`

```javascript
export const findExistingPRForPage = async (
  owner,
  repo,
  sectionId,
  pageIdFromMetadata,
  username,
  currentPageId = null,
  checkForks = true  // NEW parameter
) => {
  // Search main repo PRs
  // If not found and checkForks, search fork PRs
};
```

### Step 5: Update PageEditorPage
**File:** `src/pages/PageEditorPage.jsx`

Main editing workflow:
```javascript
const handleSave = async (newContent, editSummary) => {
  const config = getConfig();
  const mode = config.features?.editRequestCreator?.mode || 'auto';

  // 1. Check user permissions
  const hasWrite = await hasWriteAccess(owner, repo, user.login);

  // 2. Determine workflow
  let useF

ork = false;
  if (mode === 'auto') {
    useFork = !hasWrite;
  } else if (mode === 'fork-only') {
    useFork = true;
  } else if (mode === 'branch-only') {
    if (!hasWrite) {
      throw new Error('You need write access to contribute');
    }
    useFork = false;
  }

  // 3. If using fork, get/create fork
  let targetOwner = owner;
  let targetRepo = repo;
  if (useFork) {
    const fork = await getOrCreateFork(owner, repo, user.login, true);
    targetOwner = fork.owner;
    targetRepo = fork.repo;
  }

  // 4. Check for existing PR
  const existingPR = await findExistingPRForPage(
    owner,
    repo,
    sectionId,
    pageIdFromMetadata,
    user.login,
    currentPageId,
    useFork // Check forks if using fork workflow
  );

  // 5. Create branch and commit
  if (existingPR) {
    // Update existing PR
    await commitToExistingBranch(...);
  } else {
    // Create new branch
    const branchName = generateEditBranchName(sectionId, pageIdFromMetadata);
    await createBranch(targetOwner, targetRepo, branchName, baseBranch);

    // Commit to branch
    await updateFileContent(targetOwner, targetRepo, ...);

    // Create PR
    if (useFork) {
      pr = await createCrossRepoPR(owner, repo, user.login, branchName, ...);
    } else {
      pr = await createWikiEditPR(...);
    }
  }
};
```

### Step 6: Add UI Indicators
Show user which workflow is being used:
- "Creating branch on main repository..."
- "Setting up your fork..."
- "Syncing fork with upstream..."
- "Creating branch on your fork..."

### Step 7: Add Anonymous Mode UI
**When `anonymous.enabled: true` and `requireAuth: false`:**

Update Edit button to show options:
```jsx
{!isAuthenticated && anonymousEnabled ? (
  <div>
    <button onClick={handleEditAuthenticated}>
      Edit (Sign In)
    </button>
    <button onClick={handleEditAnonymous}>
      Edit Anonymously
    </button>
  </div>
) : (
  <button onClick={handleEdit}>Edit</button>
)}
```

### Step 8: Testing Checklist

#### User with Write Access
- [ ] Can create branch on main repo
- [ ] Can create PR
- [ ] Can update existing PR with new commit
- [ ] Prestige system works

#### User without Write Access
- [ ] Fork gets created automatically
- [ ] Fork syncs with upstream
- [ ] Branch created on fork
- [ ] Cross-repo PR created successfully
- [ ] Can update existing fork PR
- [ ] Prestige system works
- [ ] Fork shown in user's GitHub profile

#### Anonymous User (if enabled)
- [ ] Can access editor without auth
- [ ] Submit sends to backend
- [ ] PR created by bot account
- [ ] Attribution included in PR body
- [ ] Prestige system skips anonymous users

---

## 🎯 Priority Order

### Phase 1: Core Fork Functionality (Critical)
1. Update PR creation for cross-repo support
2. Update branch creation (already mostly done)
3. Update PageEditorPage workflow logic
4. Update PR search for forks
5. Test with non-contributor account

### Phase 2: Polish (Important)
6. Add loading states and progress indicators
7. Add error handling for fork operations
8. Add fork status indicators in UI
9. Test edge cases (conflicts, rate limits, etc.)

### Phase 3: Anonymous Mode (Optional)
10. Add anonymous UI
11. Create backend endpoint example
12. Add anonymous attribution
13. Test anonymous workflow

---

## 🔍 Testing Scenarios

### Scenario 1: First-time External Contributor
**Setup:**
- User has no fork
- User has no write access
- Making first edit

**Expected:**
1. Permission check: no write access
2. Fork created (~3-5 sec)
3. Fork synced with upstream
4. Branch created on fork
5. Commit made to fork branch
6. Cross-repo PR created (fork → upstream)
7. Success message shown
8. Prestige: First contribution

### Scenario 2: Returning External Contributor
**Setup:**
- User has existing fork
- User has existing open PR
- Making second edit to same page

**Expected:**
1. Permission check: no write access
2. Fork found
3. Fork synced with upstream
4. Existing PR found
5. Commit added to existing fork branch
6. PR updated automatically
7. Success message: "Edit Request Updated!"

### Scenario 3: Team Member with Write Access
**Setup:**
- User has write access
- No fork needed

**Expected:**
1. Permission check: has write access
2. No fork operations
3. Branch created on main repo (current flow)
4. Everything else works as currently implemented

---

## 📝 Code Review Checklist

Before merging:
- [ ] All console.log statements include [Service] prefix
- [ ] Error handling for network failures
- [ ] Error handling for rate limits
- [ ] Permission checks cached appropriately
- [ ] Fork operations have proper timeouts
- [ ] User feedback during long operations
- [ ] Documentation updated
- [ ] Examples added to EDIT_REQUEST_MODES.md
- [ ] CLAUDE.md updated with new info

---

## 🐛 Potential Issues & Solutions

### Issue: Fork takes too long to initialize
**Solution:** Show progress indicator, use polling to check fork readiness

### Issue: Fork diverged from upstream
**Solution:** Detect divergence, offer to reset fork or create fresh branch

### Issue: Rate limit hit during fork sync
**Solution:** Cache sync status, retry with exponential backoff

### Issue: Conflict between fork and upstream
**Solution:** Show clear error, link to GitHub for manual resolution

### Issue: User deletes their fork
**Solution:** Detect 404, recreate fork automatically

---

## 🚀 Future Enhancements

### Auto-Cleanup Old Forks
- Detect inactive forks (no PRs, not synced in 6+ months)
- Offer to delete and recreate fresh

### Fork Analytics
- Show fork status in user profile
- "Your fork is X commits behind"
- Button to sync fork manually

### Batch Operations
- "Sync all your wiki forks" button
- Update multiple PRs at once

### Conflict Resolution UI
- Built-in merge conflict editor
- Preview changes side-by-side
- Resolve conflicts in wiki UI

---

## 📚 Resources

### GitHub API Documentation
- [Forks API](https://docs.github.com/en/rest/repos/forks)
- [Pulls API](https://docs.github.com/en/rest/pulls/pulls)
- [Permissions API](https://docs.github.com/en/rest/repos/collaborators)
- [Merge Upstream](https://docs.github.com/en/rest/branches/branches#sync-a-fork-branch-with-the-upstream-repository)

### Testing
- Use GitHub's REST API explorer
- Create test forks manually first
- Test with personal repos before production

### Security
- [GitHub Token Best Practices](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- [Fine-grained PATs](https://github.blog/2022-10-18-introducing-fine-grained-personal-access-tokens-for-github/)
