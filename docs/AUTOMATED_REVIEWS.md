# Automated Code Review System

## Overview

The pwndbg repository uses an automated code review system that combines multiple AI-powered tools to maintain code quality, security, and consistency.

## Review Types

### 1. GitHub Copilot Reviews
Multiple specialized Copilot agents perform targeted reviews:

- **Code Cleanliness Review** - Checks code structure, organization, and style
- **Test Coverage Review** - Validates test quality and Playwright usage
- **Functionality & Documentation Review** - Ensures features are well-documented
- **CI/CD Pipeline Review** - Complete end-to-end validation

### 2. Amazon Q Code Review
Triggered after Copilot reviews complete, providing additional analysis:

- **Security Analysis** - Credential scanning, vulnerability detection, injection risk analysis
- **Performance Optimization** - Algorithm efficiency, resource management, caching opportunities
- **Architecture Review** - Design patterns, separation of concerns, dependency management

## How It Works

### Workflow Sequence

```
1. Code changes pushed to repository
   ↓
2. GitHub Copilot agents run in parallel
   ↓
3. Amazon Q Review workflow triggered
   ↓
4. Automated issue created with findings
   ↓
5. Human review and action on findings
```

### Automated Issue Creation

When reviews complete, an issue is automatically created with:
- Review date and context
- Analysis findings for each category
- Action items checklist
- Links to detailed documentation

### Issue Labels

Issues created by automated reviews use these labels:
- `amazon-q` - Amazon Q review findings
- `automated` - Auto-generated content
- `code-review` - General code review
- `needs-review` - Requires human attention
- `code-cleanliness` - Code structure issues
- `test-coverage` - Test quality issues
- `documentation` - Documentation gaps

## For Contributors

### When You Receive a Review Issue

1. **Read the findings carefully** - The issue summarizes key points
2. **Check the detailed report** - See [CODE_REVIEW_SUMMARY.md](CODE_REVIEW_SUMMARY.md)
3. **Prioritize action items** - Focus on critical security issues first
4. **Address findings** - Make necessary code changes
5. **Update the issue** - Check off completed action items

### Best Practices

To minimize review findings:

✅ **DO:**
- Pin dependency versions in `pyproject.toml`
- Use list arguments for `subprocess` calls (avoid `shell=True`)
- Sanitize user input before using in commands
- Use safe alternatives (`ast.literal_eval` instead of `eval`)
- Document security-sensitive operations
- Add tests for new features

❌ **DON'T:**
- Hardcode secrets, API keys, or credentials
- Use `eval()` or `exec()` with user-controlled input
- Use `shell=True` in subprocess calls
- Ignore dependency vulnerabilities
- Skip input validation for external commands

## Configuration

### Amazon Q Integration

To enable full Amazon Q integration:

1. **Add AWS credentials** to repository secrets:
   ```
   AWS_ACCESS_KEY_ID
   AWS_SECRET_ACCESS_KEY
   ```

2. **Install Amazon Q Developer CLI** (when available)

3. **Enable CodeWhisperer** for enhanced security scanning

### Custom Review Rules

Modify `.github/workflows/auto-amazonq-review.yml` to customize:
- Review triggers (branches, events)
- Report format and content
- Issue labels and assignments
- Artifact retention periods

## Review Schedule

- **On Push:** Triggered for main/master/develop branches
- **Post-Copilot:** After Copilot workflow completion
- **Manual:** Via workflow dispatch in Actions tab

## Latest Review Results

The most recent comprehensive code review (2026-01-10) found:
- ✅ **0 Critical Security Issues**
- ✅ **0 Security Warnings**
- ✅ **No Hardcoded Credentials**
- ✅ **Safe Subprocess Usage** (no shell injection risks)
- ✅ **Proper Input Sanitization**

For full details, see [CODE_REVIEW_SUMMARY.md](CODE_REVIEW_SUMMARY.md).

## Viewing Results

### GitHub Issues
Find automated review issues by filtering:
```
is:issue label:automated label:code-review
```

### Workflow Artifacts
Download detailed reports from:
1. Go to Actions tab
2. Select "AmazonQ Review after GitHub Copilot" workflow
3. Click on a run
4. Download "amazonq-review-report" artifact

### Documentation
Review comprehensive analysis in:
- [CODE_REVIEW_SUMMARY.md](CODE_REVIEW_SUMMARY.md) - Latest findings
- Workflow run logs - Detailed execution logs

## Troubleshooting

### Review Not Triggering
- Check if branch name matches workflow triggers
- Verify GitHub Actions are enabled for the repository
- Ensure previous workflows completed successfully

### False Positives
If a review flags a false positive:
1. Review the finding in context
2. If valid, add comment explaining why it's safe
3. Consider updating review rules to reduce noise

### AWS Credential Issues
```
continue-on-error: true
```
The workflow continues even without AWS credentials. To enable full Amazon Q features, add credentials to repository secrets.

## Further Reading

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Amazon Q Developer Documentation](https://aws.amazon.com/q/developer/)
- [pwndbg Contributing Guide](https://pwndbg.re/dev/contributing/)

## Feedback

To improve the automated review system:
- Open an issue with label `review-automation`
- Suggest new checks or patterns to detect
- Report false positives or missed issues

---

*Last updated: 2026-01-10*
