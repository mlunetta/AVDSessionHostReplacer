# Image Update Examples

This document provides detailed examples of how the Session Host Replacer handles updates when a new image version becomes available.

## Configuration

These examples use the following configuration:
- **Target Session Host Count**: 5 session hosts
- **Session Host Buffer**: 2 additional hosts allowed above target
- **Replace on New Image Delay**: 1 day (image must be available for at least 1 day before replacement starts)
- **Session Host Timer**: Runs every hour

## Scenario: Updating All Session Hosts to a New Image

### Initial State - Day 0, 12:00 UTC

A new image version becomes available:
- **Current Image Version**: `v1.0` (released 60 days ago)
- **New Image Version**: `v2.0` (just released)
- **Existing Session Hosts** (all with v1.0):
  - SH-001, SH-002, SH-003, SH-004, SH-005
- **Status**: All 5 session hosts are in "good shape"

**Function Execution**:
```
Logs:
✓ We have 5 session hosts (included in Automation)
✓ Latest Image v2.0 is 0 days old.
✓ Latest Image age is NOT yet older than New Image Delay value 1
✓ Found 0 session hosts to replace due to new image version.
✓ We have 5 good session hosts including 0 session hosts being deployed
✓ We target having 5 session hosts in good shape
✓ We have a buffer of 2 session hosts more than the target.
✓ We can deploy up to 2 session hosts
✓ We already have enough session hosts in good shape.
✓ We do not need to delete any session hosts

Result: No actions taken (waiting for delay to pass)
```

---

### Day 1, 12:00 UTC - Replacement Begins

The delay period has passed (image is now 1+ day old):
- **Current Session Hosts**: SH-001, SH-002, SH-003, SH-004, SH-005 (all v1.0)
- **New Image Available**: v2.0 (released 1 day ago)

**Function Execution**:
```
Logs:
✓ We have 5 session hosts (included in Automation)
✓ Latest Image v2.0 is 1 day old.
✓ Latest Image age is older than (or equal) New Image Delay value 1
✓ Found 5 session hosts to replace due to new image version. SH-001, SH-002, SH-003, SH-004, SH-005
✓ Found 5 session hosts to replace in total. SH-001, SH-002, SH-003, SH-004, SH-005

✓ We have 0 good session hosts including 0 session hosts being deployed
✓ We target having 5 session hosts in good shape
✓ We have a buffer of 2 session hosts more than the target.
✓ We can deploy up to 2 session hosts
✓ We need to deploy 5 new session hosts
✓ Buffer allows deploying 2 session hosts
✓ We need to delete 0 session hosts

Result: 
  → Deploy 2 new session hosts with v2.0: SH-006, SH-007
  → Mark 5 old hosts for eventual deletion: SH-001, SH-002, SH-003, SH-004, SH-005
```

**Session Hosts Status After Execution**:
| Name | Image | Status | Notes |
|------|-------|--------|-------|
| SH-001 | v1.0 | Draining | Marked for deletion |
| SH-002 | v1.0 | Draining | Marked for deletion |
| SH-003 | v1.0 | Available | Waiting to be drained |
| SH-004 | v1.0 | Available | Waiting to be drained |
| SH-005 | v1.0 | Available | Waiting to be drained |
| SH-006 | v2.0 | Available | **Deploying** |
| SH-007 | v2.0 | Available | **Deploying** |

---

### Day 1, 13:00 UTC - First Deployment Completes

The 2 new hosts with v2.0 are now available:
- **New Session Hosts**: SH-006, SH-007 (v2.0)
- **Old Session Hosts**: SH-001, SH-002 in drain, SH-003, SH-004, SH-005 still available (v1.0)
- **Total Session Hosts**: 7

**Function Execution**:
```
Logs:
✓ We have 7 session hosts (included in Automation)
✓ Latest Image v2.0 is 1 day old.
✓ Latest Image age is older than (or equal) New Image Delay value 1
✓ Found 5 session hosts to replace due to new image version. SH-001, SH-002, SH-003, SH-004, SH-005
✓ Found 5 session hosts to replace in total. SH-001, SH-002, SH-003, SH-004, SH-005

✓ We have 2 good session hosts including 0 session hosts being deployed
✓ We target having 5 session hosts in good shape
✓ We have a buffer of 2 session hosts more than the target.
✓ We can deploy up to 0 session hosts
✓ We need to deploy 3 new session hosts
✓ Buffer allows deploying 0 session hosts
✓ We need to delete 2 session hosts
✓ Host pool is not over populated
✓ The following Session Hosts are now pending delete: SH-001, SH-002

Result:
  → No new deployment can start yet because the pool is already at target + buffer (7 hosts)
  → Delete SH-001 and SH-002 if no users are connected (or after grace period)
```

**Session Hosts Status After Execution**:
| Name | Image | Status | Notes |
|------|-------|--------|-------|
| SH-001 | v1.0 | Deleting | No users or grace period expired |
| SH-002 | v1.0 | Deleting | No users or grace period expired |
| SH-003 | v1.0 | Available | Waiting to be drained |
| SH-004 | v1.0 | Available | Waiting to be drained |
| SH-005 | v1.0 | Available | Waiting to be drained |
| SH-006 | v2.0 | Available | ✅ Ready for users |
| SH-007 | v2.0 | Available | ✅ Ready for users |

---

### Day 1, 14:00 UTC - Capacity Frees Up Again

The first 2 old hosts have now been removed:
- **Good Session Hosts**: SH-006, SH-007 (v2.0) = 2 hosts
- **Old Session Hosts**: SH-003, SH-004, SH-005 (v1.0)
- **Total Session Hosts**: 5

**Function Execution**:
```
Logs:
✓ We have 5 session hosts (included in Automation)
✓ Latest Image v2.0 is 1 day old.
✓ Latest Image age is older than (or equal) New Image Delay value 1
✓ Found 3 session hosts to replace due to new image version. SH-003, SH-004, SH-005
✓ Found 3 session hosts to replace in total. SH-003, SH-004, SH-005

✓ We have 2 good session hosts including 0 session hosts being deployed
✓ We target having 5 session hosts in good shape
✓ We have a buffer of 2 session hosts more than the target.
✓ We can deploy up to 2 session hosts
✓ We need to deploy 3 new session hosts
✓ Buffer allows deploying 2 session hosts
✓ We need to delete 0 session hosts

Result:
  → Deploy 2 more new session hosts with v2.0: SH-001, SH-002
  → SH-003, SH-004 and SH-005 remain candidates for deletion once capacity allows
```

**Session Hosts Status After Execution**:
| Name | Image | Status | Notes |
|------|-------|--------|-------|
| SH-001 | v1.0 | Deleted | ✅ Cleanup complete |
| SH-002 | v1.0 | Deleted | ✅ Cleanup complete |
| SH-003 | v1.0 | Available | Waiting to be drained |
| SH-004 | v1.0 | Available | Waiting to be drained |
| SH-005 | v1.0 | Available | Waiting to be drained |
| SH-006 | v2.0 | Available | ✅ Ready for users |
| SH-007 | v2.0 | Available | ✅ Ready for users |
| SH-001 | v2.0 | Available | **Deploying** |
| SH-002 | v2.0 | Available | **Deploying** |

---

### Day 1, 15:00 UTC - Buffer Reached Again

The second batch of 2 new hosts is now available:
- **Good Session Hosts**: SH-006, SH-007, SH-001, SH-002 (v2.0) = 4 hosts
- **Old Session Hosts**: SH-003, SH-004, SH-005 (v1.0)
- **Total Session Hosts**: 7

**Function Execution**:
```
Logs:
✓ We have 7 session hosts (included in Automation)
✓ Found 3 session hosts to replace due to new image version. SH-003, SH-004, SH-005
✓ Found 3 session hosts to replace in total. SH-003, SH-004, SH-005

✓ We have 4 good session hosts including 0 session hosts being deployed
✓ We target having 5 session hosts in good shape
✓ We have a buffer of 2 session hosts more than the target.
✓ We can deploy up to 0 session hosts
✓ We need to deploy 1 new session host
✓ Buffer allows deploying 0 session hosts
✓ We need to delete 2 session hosts
✓ Host pool is not over populated
✓ The following Session Hosts are now pending delete: SH-003, SH-004

Result:
  → No new deployment can start yet because the pool is again at target + buffer
  → Delete SH-003 and SH-004 when safe to do so
```

**Session Hosts Status After Execution**:
| Name | Image | Status | Notes |
|------|-------|--------|-------|
| SH-003 | v1.0 | Deleting | No users or grace period expired |
| SH-004 | v1.0 | Deleting | No users or grace period expired |
| SH-005 | v1.0 | Available | Waiting to be drained |
| SH-006 | v2.0 | Available | ✅ Ready for users |
| SH-007 | v2.0 | Available | ✅ Ready for users |
| SH-001 | v2.0 | Available | ✅ Ready for users |
| SH-002 | v2.0 | Available | ✅ Ready for users |

---

### Day 1, 16:00 UTC - Final New Host Is Allowed

After SH-003 and SH-004 are removed, one old host remains:
- **Good Session Hosts**: SH-006, SH-007, SH-001, SH-002 (v2.0) = 4 hosts
- **Old Session Hosts**: SH-005 (v1.0)
- **Total Session Hosts**: 5

**Function Execution**:
```
Logs:
✓ We have 5 session hosts (included in Automation)
✓ Found 1 session host to replace due to new image version. SH-005
✓ Found 1 session host to replace in total. SH-005

✓ We have 4 good session hosts including 0 session hosts being deployed
✓ We target having 5 session hosts in good shape
✓ We have a buffer of 2 session hosts more than the target.
✓ We can deploy up to 2 session hosts
✓ We need to deploy 1 new session host
✓ Buffer allows deploying 1 session host
✓ We need to delete 0 session hosts

Result:
  → Deploy the final new session host with v2.0: SH-003
  → SH-005 remains pending deletion after the replacement is ready
```

**Session Hosts Status After Execution**:
| Name | Image | Status | Notes |
|------|-------|--------|-------|
| SH-006 | v2.0 | Available | ✅ Production |
| SH-007 | v2.0 | Available | ✅ Production |
| SH-001 | v2.0 | Available | ✅ Production |
| SH-002 | v2.0 | Available | ✅ Production |
| SH-005 | v1.0 | Draining | Marked for deletion |
| SH-003 | v2.0 | Available | **Deploying** |

---

### Day 1, 17:00 UTC - Last Old Host Is Removed

The final replacement host is now available:
- **Good Session Hosts**: SH-006, SH-007, SH-001, SH-002, SH-003 (v2.0) = 5 hosts
- **Old Session Hosts**: SH-005 (v1.0)
- **Total Session Hosts**: 6

**Function Execution**:
```
Logs:
✓ We have 6 session hosts (included in Automation)
✓ Found 1 session host to replace due to new image version. SH-005
✓ Found 1 session host to replace in total. SH-005

✓ We have 5 good session hosts including 0 session hosts being deployed
✓ We target having 5 session hosts in good shape
✓ We have a buffer of 2 session hosts more than the target.
✓ We can deploy up to 1 session host
✓ We already have enough session hosts in good shape.
✓ We need to delete 1 session host
✓ Host pool is not over populated
✓ The following Session Hosts are now pending delete: SH-005

Result:
  → No further deployment is needed
  → Delete SH-005 when safe to do so
```

**Session Hosts Status After Execution**:
| Name | Image | Status | Notes |
|------|-------|--------|-------|
| SH-005 | v1.0 | Deleting | Grace period or no users |
| SH-006 | v2.0 | Available | ✅ Ready for users |
| SH-007 | v2.0 | Available | ✅ Ready for users |
| SH-001 | v2.0 | Available | ✅ Ready for users |
| SH-002 | v2.0 | Available | ✅ Ready for users |
| SH-003 | v2.0 | Available | ✅ Ready for users |

---

### Day 1, 18:00 UTC - Cleanup Complete

Final state after all old hosts are removed:

**Function Execution**:
```
Logs:
✓ We have 5 session hosts (included in Automation)
✓ Found 0 session hosts to replace due to new image version.
✓ Found 0 session hosts to replace in total.

✓ We have 5 good session hosts including 0 session hosts being deployed
✓ We target having 5 session hosts in good shape
✓ We have a buffer of 2 session hosts more than the target.
✓ We can deploy up to 2 session hosts
✓ We already have enough session hosts in good shape.
✓ We do not need to delete any session hosts

Result: Fully updated! ✅
```

**Final Session Hosts Status**:
| Name | Image | Status | Notes |
|------|-------|--------|-------|
| SH-006 | v2.0 | Available | ✅ Production |
| SH-007 | v2.0 | Available | ✅ Production |
| SH-001 | v2.0 | Available | ✅ Production |
| SH-002 | v2.0 | Available | ✅ Production |
| SH-003 | v2.0 | Available | ✅ Production |

---

## Key Takeaways

1. **Gradual Rollout**: The buffer setting (2 hosts) controls how far above the target the pool can grow. Once the pool reaches `TargetSessionHostCount + TargetSessionHostBuffer`, deployments pause until old hosts are deleted.

2. **Deploy First, Remove Later**: New hosts with the updated image are deployed before old hosts are removed, ensuring no gap in capacity.

3. **Grace Period**: Old hosts are placed in drain mode and given a grace period to complete user sessions before deletion.

4. **Delay Safety**: The `ReplaceSessionHostOnNewImageVersionDelayDays` parameter provides a safety window to ensure the new image is stable before rolling out to all hosts.

5. **Naming Behavior**: New VMs use the lowest available numbered name for the configured prefix, so numbers can be reused after old hosts are deleted. During a rolling replacement you can temporarily see a mix such as `SH-006`, `SH-007`, then later `SH-001`, `SH-002`, and `SH-003`.

6. **Automatic Progression**: With the timer running every hour, the system automatically progresses through each batch without manual intervention.

## Customization Options

To adjust the update behavior:

| Setting | Impact | Example |
|---------|--------|---------|
| `TargetSessionHostBuffer` | Larger buffer = faster updates (but more concurrent hosts) | Increase to 5 for aggressive updates |
| `ReplaceSessionHostOnNewImageVersionDelayDays` | Longer delay = more validation time before rollout | Increase to 7 for conservative approach |
| `TargetSessionHostCount` | Defines the target number of production hosts | Adjust based on load requirements |
| Timer Schedule | Affects update speed (hourly vs. more frequent) | Consider cost/benefit of more frequent runs |
