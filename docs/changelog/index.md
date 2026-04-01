# Changelog

All FloofOS versions and changes.

---

## Versions

| Version | Date | Type | Notes |
|---------|------|------|-------|
| [1.0](v1.0.md) | December 2025 | Stable | Initial release |

---

## Versioning

FloofOS follows [Semantic Versioning](https://semver.org/):

- **Major** (x.0.0) - Breaking changes
- **Minor** (1.x.0) - New features, backward compatible
- **Patch** (1.0.x) - Bug fixes, backward compatible

---

## Upgrade Guide

Before upgrading, always:

1. Backup your configuration
2. Read the release notes for breaking changes
3. Test in a non-production environment first

```bash
# Backup current config
floofctl> request system backup

# Check current version
floofctl> show version
```

