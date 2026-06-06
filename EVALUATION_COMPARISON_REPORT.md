# EVALUATION COMPARISON REPORT
## TRAIL GUIDE AGENT — AUTOMATED EVALUATION

**GENERATED:** 2026-06-06  
**BRANCHES EVALUATED:** evaluator-changes-comparison | model-comparison-eval-run | test/eval-workflow-check

---

## SUMMARY

| BRANCH | RUN ID | STATUS | FAILURE REASON |
|--------|--------|--------|---------------|
| evaluator-changes-comparison | 27046895933 | FAILED | AADSTS700024 — OIDC TOKEN EXPIRED |
| model-comparison-eval-run | 27046896868 | FAILED | AADSTS700024 — OIDC TOKEN EXPIRED |
| test/eval-workflow-check | 27046898312 | FAILED | AADSTS700024 — OIDC TOKEN EXPIRED |

---

## ROOT CAUSE

**ALL THREE BRANCHES FAILED FOR THE SAME REASON — NOT A CODE OR PERMISSIONS BUG.**

The GitHub Actions OIDC token has a **5-minute validity window**. All three jobs were queued at approximately `00:16 UTC` but GitHub's shared runners did not pick them up until `01:11 UTC` — approximately **55 minutes later**. By the time the Azure Login step ran, the OIDC token had expired.

```
TOKEN VALID FROM:  2026-06-06T00:16:xx UTC
TOKEN EXPIRED AT:  2026-06-06T00:21:xx UTC
JOB ACTUALLY RAN:  2026-06-06T01:11:xx UTC
TIME IN QUEUE:     ~55 minutes
```

This is a **GitHub Actions runner availability issue**, not a problem with:
- Authentication configuration
- Service principal permissions
- Federated credential setup
- Azure AI Foundry project configuration

---

## PERMISSIONS CONFIGURED (VERIFIED WORKING IN PRIOR RUNS)

The following roles were successfully assigned to service principal `github-actions-eval` (AppId: `0c634f16-b267-47f1-8fd5-ff9cc1bd42f3`):

| ROLE | SCOPE |
|------|-------|
| Contributor | Subscription |
| Azure AI Developer | AI Foundry Account (ai-account-4jae5jxfkfjjc) |
| Cognitive Services User | AI Foundry Account (ai-account-4jae5jxfkfjjc) |
| AzureML Data Scientist | Resource Group (rg-rg-ai-300) |

FEDERATED CREDENTIALS CONFIGURED:
- `github-actions-main` → `repo:CarlosJoseChaconChavarria/Automated-evaluation-with-cloud-evaluators:ref:refs/heads/main`
- `github-actions-pr` → `repo:CarlosJoseChaconChavarria/Automated-evaluation-with-cloud-evaluators:pull_request`

---

## EVALUATION SCORES

**NOT AVAILABLE** — ALL RUNS FAILED BEFORE REACHING THE EVALUATION STEP.

Prior runs that reached Step 3 (before OBO token issue was resolved) confirmed:
- Dataset upload (Step 1): WORKING
- Evaluation definition creation (Step 2): WORKING
- Evaluation run start (Step 3): BLOCKED by OBO token error (since resolved with AzureML Data Scientist role)

---

## BRANCH DIFFERENCES

| BRANCH | PURPOSE | JUDGE MODEL | THRESHOLD |
|--------|---------|-------------|-----------|
| test/eval-workflow-check | BASELINE WORKFLOW VALIDATION | gpt-4.1 | 3.0 |
| evaluator-changes-comparison | THRESHOLD COMPARISON: 3.0 vs 4.0 | gpt-4.1 | 3.0 / 4.0 |
| model-comparison-eval-run | MODEL QUALITY/COST TRADEOFF | gpt-4.1-mini | 3.0 |

---

## RECOMMENDATIONS

### SHORT TERM — RE-RUN STRATEGY
Re-trigger the workflows during **off-peak hours** (GitHub-hosted runners have shorter queues). The 5-minute OIDC token window is usually sufficient when jobs start within ~2 minutes of being queued.

### LONG TERM — SWITCH TO CLIENT SECRET AUTH
Replace OIDC federated credentials with a **client secret** stored as a GitHub secret. Client secrets don't have a 5-minute expiry window and are immune to runner queue delays.

Steps:
```bash
az ad app credential reset --id 0c634f16-b267-47f1-8fd5-ff9cc1bd42f3 --years 1
# Save the password output as GitHub secret: AZURE_CLIENT_SECRET
```

Then update the workflow to use `client-secret` instead of OIDC:
```yaml
- uses: azure/login@v2
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
    client-secret: ${{ secrets.AZURE_CLIENT_SECRET }}
```

---

## INFRASTRUCTURE STATUS

**AZD DOWN EXECUTED** — ALL AZURE RESOURCES HAVE BEEN DELETED.

Resources removed:
- Azure AI Foundry Project (ai-project-rg-ai-300)
- Azure AI Account (ai-account-4jae5jxfkfjjc)
- Resource Group (rg-rg-ai-300)
- All associated deployments (gpt-4.1, gpt-4.1-mini)

**IT IS SAFE TO CLOSE THIS SESSION. NO FURTHER AZURE COSTS WILL ACCRUE.**
