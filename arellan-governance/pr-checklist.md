# Pull Request Review Checklist

Before approving any Pull Request targeting `develop` or `staging`, the reviewer must confirm:

- [ ] Minimum unit test coverage of 80% achieved.
- [ ] No hardcoded database credentials or environment variables present.
- [ ] Authorization guards explicitly added to any newly created controller method.
- [ ] Database migrations are backwards-compatible.