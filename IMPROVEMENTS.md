# MS Teams Notification Action v2 - Improvement Plan

## Current State Analysis

### What's Working Well
- ✅ Basic notification types (PR, deployment, custom)
- ✅ Webhook integration
- ✅ Theme color based on status
- ✅ Additional facts support
- ✅ Action buttons with URLs

### Issues & Improvement Areas

#### 1. Performance
| Issue | Impact | Solution |
|-------|--------|-----------|
| Bash script execution | Slower startup, less reliable | Migrate to Node.js (JavaScript runtime) |
| No retry mechanism | Notifications fail silently | Add exponential backoff retry (3 attempts) |
| No timeout | Action can hang indefinitely | Add 30s timeout to curl |
| Large payload handling | Can fail on big messages | Implement payload truncation |

#### 2. Reliability
| Issue | Impact | Solution |
|-------|--------|-----------|
| No response validation | Don't know if Teams received | Validate HTTP response |
| Silent failures | Hard to debug | Add detailed error messages |
| No webhook URL validation | Fails at runtime | Validate URL format before sending |
| Input sanitization | Potential injection | Sanitize all user inputs |

#### 3. UX/Usability
| Issue | Impact | Solution |
|-------|--------|-----------|
| No debug mode | Hard to troubleshoot | Add `debug` input for verbose logging |
| Complex JSON for facts | Error-prone for users | Support both JSON and simple key=value format |
| No mention support | Can't @mention users | Add `mentions` input |
| Fixed image URL | Not customizable per org | Make activity image truly optional |

#### 4. Modern Features (Missing)
| Feature | Benefit | Priority |
|---------|---------|----------|
| Adaptive Cards | Richer UI, interactive buttons | High |
| Message threading | Group related notifications | Medium |
| Channel/team info | More context in notifications | Low |
| Batch notifications | Send multiple at once | Low |

---

## Proposed Changes for v2

### Breaking Changes
1. **Runtime**: Node.js 20 (from composite bash)
2. **Input**: `additional_facts` now accepts `key=value` format too

### New Inputs
```yaml
# New in v2
debug:                    # boolean - Enable verbose logging
timeout:                  # number - Request timeout in seconds (default: 30)
retry:                    # boolean - Enable retry on failure (default: true)
retry_count:             # number - Number of retries (default: 3)
mention_users:            # string - Comma-separated list of users to mention
mention_groups:           # string - Teams group IDs to mention
card_format:             # string - "messagecard" or "adaptive" (default: messagecard)
```

### Architecture Changes

#### Before (v1.x)
```
action.yml (composite)
  └── bash script (inline)
       └── curl to webhook
```

#### After (v2)
```
action.yml (composite)
  └── node20 (main.js)
       ├── lib/validator.js    # Input validation
       ├── lib/retry.js       # Retry logic
       ├── lib/formatter.js   # Message formatting
       └── lib/sender.js      # HTTP client
```

---

## Implementation Plan

### Phase 1: Foundation (High Priority)
- [ ] Migrate from bash to Node.js
- [ ] Add retry mechanism with exponential backoff
- [ ] Add input validation
- [ ] Add timeout support

### Phase 2: Reliability (High Priority)
- [ ] Validate webhook URL format
- [ ] Check Teams response for success
- [ ] Add detailed error messages
- [ ] Add debug mode

### Phase 3: Enhanced Features (Medium Priority)
- [ ] Support mention users/groups
- [ ] Add `key=value` format for facts
- [ ] Truncate long messages gracefully
- [ ] Add card_format option

### Phase 4: Modern Cards (Future)
- [ ] Adaptive Cards support
- [ ] Interactive buttons with callbacks
- [ ] Message threading

---

## Technical Details

### Retry Logic
```javascript
const retryConfig = {
  retries: 3,
  factor: 2,
  minTimeout: 1000,
  maxTimeout: 10000
};
```

### Input Validation
```javascript
const schema = {
  webhook_url: { required: true, format: 'url' },
  notification_type: { required: true, enum: ['pr', 'deployment_started', 'deployment_finished', 'custom'] },
  status: { enum: ['success', 'failure', 'in_progress'] },
  theme_color: { format: 'hex' }
};
```

---

## Success Metrics

- **Performance**: Cold start < 2s (currently ~3-5s)
- **Reliability**: 99.9% delivery rate with retries
- **UX**: 50% fewer support issues due to better error messages

---

## Migration Guide (v1 → v2)

### Required Changes
None - v2 is backward compatible

### Recommended Updates
```yaml
# Old (v1)
- uses: haroldelopez/ms-teams-notification-action@v1

# New (v2)
- uses: haroldelopez/ms-teams-notification-action@v2
  with:
    retry: true          # Enable retries
    debug: ${{ secrets.DEBUG }}  # Optional debug mode
```

---

*Document generated for PR to @haroldelopez*
*Branch: improve/v2-enhanced-notifications*
