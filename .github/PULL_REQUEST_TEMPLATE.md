# Pull Request

## Description

Please include a summary of the changes and which issue is fixed. Please also include relevant motivation and context.

Fixes # (issue)

## Type of Change

Please delete options that are not relevant.

- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to change)
- [ ] Documentation update
- [ ] Performance improvement
- [ ] Code refactoring
- [ ] Test addition/update
- [ ] Configuration change

## Checklist

Before submitting this PR, please ensure you have completed the following:

### Code Quality
- [ ] My code follows the PSR-12 coding standards
- [ ] I have run `./vendor/bin/pint` to format my code
- [ ] I have checked for any security vulnerabilities
- [ ] I have added/updated inline documentation

### Testing
- [ ] I have written tests for my changes
- [ ] All existing tests pass locally (`composer test`)
- [ ] I have tested edge cases
- [ ] Test coverage has not decreased

### Documentation
- [ ] I have updated the CHANGELOG.md
- [ ] I have updated relevant documentation files
- [ ] My commit message follows Conventional Commits format
- [ ] I have added comments where necessary

### Security
- [ ] My changes don't introduce new security risks
- [ ] I have validated all user inputs
- [ ] I have checked for SQL injection vulnerabilities
- [ ] I have verified authentication/authorization is properly implemented

## Changes Made

List the specific changes made in this PR:

1. 
2. 
3. 

## Screenshots/Recordings

If applicable, add screenshots or screen recordings to demonstrate the changes.

## Testing Instructions

Please provide step-by-step instructions on how to test your changes:

1. 
2. 
3. 

## Related Issues

Link to any related issues or discussions:

- Closes #
- Related to #

## Deployment Notes

Are there any special deployment considerations?

- [ ] Database migrations required
- [ ] Configuration changes needed
- [ ] Environment variables to add
- [ ] Cache clearing required
- [ ] No special requirements

### Migration Commands (if applicable)

```bash
php artisan migrate
php artisan db:seed --class=SpecificSeeder
```

### Configuration Changes (if applicable)

```env
NEW_CONFIG_KEY=value
```

## Performance Impact

Does this PR affect performance?

- [ ] No impact
- [ ] Performance improved
- [ ] Performance degraded (please explain)

### Benchmarks (if applicable)

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Response Time | | | |
| Memory Usage | | | |

## Additional Context

Add any other information about the PR here.

## Reviewer Notes

Anything specific you want reviewers to pay attention to?

---

**PR Author:** @yourusername  
**Date Created:** YYYY-MM-DD  
**Last Updated:** YYYY-MM-DD

---

### For Maintainers

- [ ] Code review completed
- [ ] Tests verified
- [ ] Documentation reviewed
- [ ] Changelog updated
- [ ] Ready to merge
