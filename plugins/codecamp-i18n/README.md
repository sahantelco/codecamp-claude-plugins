# codecamp-i18n

Translation audit plugin for Codecamp Angular projects using `@jsverse/transloco`.

## Skills

### `/check-i18n [path]`

Audits TypeScript and HTML files for:
- Hardcoded UI text not wrapped with `| transloco` pipe (HTML) or `translate.translate()` (TS)
- Translation keys used in code but not registered in the site's `app-translations.ts`
- Stale keys in `app-translations.ts` that are no longer referenced

**Usage:**
```
/check-i18n                                          # audits git diff HEAD
/check-i18n client/teachers-site/src/app/some/path  # audits a specific directory or file
```

## Supported sites

- `client/teachers-site`
- `client/arena-site`
- `client/enterprise-site`
- `client/onboarding`
- `client/parents-site`
