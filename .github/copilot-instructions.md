# ZK Framework — AI Development Guide

## Project Overview
ZK is an open-source Java web framework. The sibling directory `../zkcml/` contains enterprise extensions that depend on this project.

**Modules:**
- `zk/`, `zul/`, `zkbind/`, `zhtml/` — core framework (TypeScript + Java)
- `zcommon/`, `zel/`, `zweb/`, `zweb-dsp/`, `zkplus/` — shared utilities
- `zktest/` — Selenium integration tests
- `eslint-plugin-zk/` — custom ESLint rules

## Tech Stack
- **Frontend:** TypeScript 5.3.3 + JavaScript, Gulp 5 + Webpack 5
- **Backend:** Java 11, Gradle
- **CI:** GitHub Actions (`.github/workflows/`)

## Build Commands

### TypeScript / JavaScript
```bash
npm run build        # build all TS/JS (gulp)
npm run dev          # watch mode
npm run type-check   # type check only, no output
npm run lint -- .    # ESLint on .js and .ts files (path argument required)
```

### Java
```bash
./gradlew clean build         # full build
./gradlew checkstyleMain      # Java style check only
./gradlew :zk:build           # single module build
```

## Code Style

### TypeScript / JavaScript
- ESLint config: `.eslintrc.js` (root)
- Custom rules: `eslint-plugin-zk/` — do not disable without review
- Microsoft SDL plugin is enabled — security patterns are enforced
- **ESLint errors block CI** — fix all errors before committing

### Java
- Checkstyle config: `config/checkstyle/`
- Violations block CI

## Testing Requirements

**Every bug fix and new feature MUST include a test case.**

### Location & Naming Convention
- Path: `zktest/src/test/java/org/zkoss/zktest/zats/test2/`
- Pattern: `B{majorVersion}_{IssueId}Test.java`
- Example: `B100_ZK_5529Test.java` → ZK 10.x, issue ZK-5529

### Test Template
```java
package org.zkoss.zktest.zats.test2;

import static org.junit.jupiter.api.Assertions.assertTrue;
import org.junit.jupiter.api.Test;
import org.zkoss.test.webdriver.WebDriverTestCase;

public class B100_ZK_5529Test extends WebDriverTestCase {
    @Test
    public void test() {
        connect();
        waitResponse();
        // jq() for jQuery-style selectors
        assertTrue(jq(".z-component").exists());
    }
}
```

- Extends `org.zkoss.test.webdriver.WebDriverTestCase`
- A corresponding ZUL page must be added under `zktest/src/main/webapp/test2/`
  - ZUL naming uses dashes: `B100-ZK-5529.zul` (Java test uses underscores)
- After adding the ZUL page, register it in `zktest/src/main/webapp/test2/config.properties`:
  ```
  B100-ZK-5529.zul=A,M,ComponentName
  ```
  - Field 1 — Code Level: `A`=Important, `B`=Unknown, `C`=Unimportant
  - Field 2 — UX Level: `H`=Hard, `M`=Middle, `E`=Easy
  - Field 3+ — affected component names (e.g., `Tree`, `Listbox`, `Grid`)

## Source Layout
- TS source: `{module}/src/main/resources/web/js/`
- Java source: `{module}/src/main/java/org/zkoss/`

## Critical Constraints
- Do NOT change public Java APIs in `zk/`, `zul/`, or `zkbind/` without checking impact on `../zkcml/`
- If any ZUL component attribute or element is added/removed/changed, update `zul/src/main/resources/metainfo/xml/zul.xsd` accordingly
- Issue tracker: https://tracker.zkoss.org/projects/ZK

## Workflow for Each Issue
1. Find or create the issue in tracker
2. Write the test first in `zktest/` using the issue ID in the filename
3. Add the corresponding ZUL test page in `zktest/src/main/webapp/test2/`
4. Register the ZUL page in `zktest/src/main/webapp/test2/config.properties`
5. Implement the fix
6. If ZUL component API changed (attribute/element added or removed), update `zul/src/main/resources/metainfo/xml/zul.xsd`
7. Run `npm run lint -- . && ./gradlew checkstyleMain`
8. Verify the test passes
9. PR title must reference the issue ID
