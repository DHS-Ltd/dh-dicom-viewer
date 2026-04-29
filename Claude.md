# CLAUDE.md — DH DICOM Viewer (DHV)

## Project Identity
- **Product:** DH DICOM Viewer
- **Company:** DH Solutions Limited
- **Base:** Weasis 4.7.0-SNAPSHOT (Apache 2.0 licensed, open source)
- **Purpose:** Rebranded commercial DICOM viewer with time-based licensing

---

## Environment
| Item | Value |
|---|---|
| OS | Windows 11 |
| Java | Temurin 25 — `E:\java\Eclipse Adoptium\jdk-17.0.18.8-hotspot` |
| Maven | 3.9.15 — `C:\Program Files\Apache\maven\apache-maven-3.9.15` |
| Project root | `E:\DHV-Weasis\DHDicomViewer\dh-dicom-viewer` |
| Assets | `E:\DHV-Weasis\Asset\` |
| Asset script | `E:\DHV-Weasis\Asset\prepare_assets.py` |

---

## Key Paths
```
dh-dicom-viewer/
├── weasis-distributions/
│   ├── etc/config/          ← JSON config source (base.json, dicomizer.json, non-dicom-explorer.json)
│   ├── resources/images/    ← Image assets source (about.png, about-round.png, logo-button.png)
│   └── target/native-dist/bin-dist/weasis/  ← Built output (regenerated on every build)
├── weasis-launcher/
│   ├── conf/                ← Parallel JSON config (same files as etc/config — both must be edited)
│   └── src/main/java/org/weasis/launcher/  ← AppLauncher.java lives here
└── weasis-base/
    └── weasis-base-ui/src/main/java/org/weasis/base/ui/gui/  ← WeasisAboutBox.java
```

---

## Build Commands
```powershell
# Full project build (run from project root) — first time or after Java changes
mvn clean install -DskipTests

# Distribution build (run from weasis-distributions/) — after config/asset changes
cd weasis-distributions
mvn clean package -DskipTests
# Then extract: Expand-Archive target\native-dist\weasis-native.zip target\native-dist -Force

# Launch built app
cd target\native-dist\bin-dist\weasis
java -cp "weasis-launcher.jar;felix.jar" org.weasis.launcher.AppLauncher
```

> ⚠️ Always run `mvn clean package -DskipTests` from `weasis-distributions/` for
> config/asset changes. Run `mvn clean install -DskipTests` from root only when
> Java source files change.

---

## Current Status

### ✅ Phase 1 — Environment & Build (COMPLETE)
- Java 25 + Maven 3.9.15 installed and verified
- Weasis forked to `github.com/DHS-Ltd/dh-dicom-viewer`
- Clean build confirmed working

### 🔄 Phase 2 — Rebranding (IN PROGRESS)

**Done:**
- JSON config files updated in both `etc/config/` and `weasis-launcher/conf/`
- Title bar shows "DH DICOM Viewer" ✅
- Splash screen text shows "DH DICOM Viewer" ✅

**Remaining:**
- [ ] Fix copyright line — still shows "© 2009-2026 Weasis Team" in About screen
- [ ] Fix version — still shows "4.7.0-SNAPSHOT" instead of "1.0.0"
- [ ] Replace image assets — Weasis green logo still appears in About screen
- [ ] Custom About screen (WeasisAboutBox.java rewrite)
- [ ] Rebuild and full visual verification
- [ ] Git tag `v1.0.0-branded`

### ⏳ Phase 3 — License System (NOT STARTED)
### ⏳ Phase 4 — Installer (NOT STARTED)

---

## Rebranding Rules

### JSON Config Files
- Edit files in BOTH `weasis-distributions/etc/config/` AND `weasis-launcher/conf/`
- Only change `"value"` fields containing display text
- Never change `"code"` fields — these are OSGi system property keys
- Never change `"value": "weasis-base-ui"` — internal bundle identifier
- Never rename `org.weasis.*` Java packages — breaks the build

### Image Assets
Source logo: `E:\DHV-Weasis\Asset\DHV-Logo.png`
Asset script produces and deploys:
| File | Size | Destination |
|---|---|---|
| `about.png` | 374×147 | `weasis-distributions/resources/images/` |
| `about-round.png` | 567×244 | `weasis-distributions/resources/images/` |
| `logo-button.png` | 64×64 | `weasis-distributions/resources/images/` |
| `Weasis.ico` | multi-size | `weasis-distributions/resources/` |

Run asset script before every rebuild that involves image changes:
```powershell
cd E:\DHV-Weasis\Asset && python prepare_assets.py
```

---

## Immediate Next Tasks (Pick Up Here)

### Task 1 — Fix Copyright and Version in Java Source
Find the file:
```powershell
Get-ChildItem -Path "E:\DHV-Weasis\DHDicomViewer\dh-dicom-viewer" `
  -Filter "*.java" -Recurse `
  | Where-Object { $_.FullName -notlike "*target*" } `
  | Select-String -Pattern "Weasis Team|2009-2026|copyrights" -SimpleMatch `
  | Select-Object Path, LineNumber, Line
```
Change copyright to `© 2026 DH Solutions Limited` and version to `1.0.0`.

### Task 2 — Run Asset Script
```powershell
cd E:\DHV-Weasis\Asset
python prepare_assets.py
```

### Task 3 — Rewrite WeasisAboutBox.java
File: `weasis-base/weasis-base-ui/src/main/java/org/weasis/base/ui/gui/WeasisAboutBox.java`
Replace with custom About screen showing:
- DH DICOM Viewer logo + name + version 1.0.0
- DH Solutions Limited + contact info
- License status panel (stub for now — Phase 3 wires this up)
- Apache 2.0 notice at bottom

### Task 4 — Full Rebuild and Verify
```powershell
# From project root (Java changes require full rebuild)
cd E:\DHV-Weasis\DHDicomViewer\dh-dicom-viewer
mvn clean install -DskipTests

# Then distribution
cd weasis-distributions
mvn clean package -DskipTests
Expand-Archive target\native-dist\weasis-native.zip target\native-dist -Force
```

---

## Git Workflow
```powershell
# Feature branches
git checkout -b phase2-rebranding   # work here
git checkout master                 # merge when verified

# Commit style
git commit -m "phase2: fix copyright in AboutBox"
git commit -m "phase2: replace image assets with DHV branding"

# Tags at phase completion
git tag v1.0.0-branded    # end of Phase 2
git tag v1.0.0-licensed   # end of Phase 3
git tag v1.0.0-release    # end of Phase 4
```

---

## Do Not Touch
- `org.weasis.*` package names in any Java file
- `"code"` fields in any JSON config file
- `"value": "weasis-base-ui"` in base.json
- `${weasis.codebase.url}` template strings
- Any file inside `target/` folder — wiped on every build

---

## Documentation Rule
All project documentation is saved as `.md` files unless explicitly requested otherwise.