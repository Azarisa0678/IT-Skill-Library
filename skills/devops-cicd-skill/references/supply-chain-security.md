# Supply Chain Security — Referenzmodul

SBOM, SLSA, Sigstore/Cosign, Dependency-Track, CycloneDX, SPDX, Scorecard,
SSDF (NIST), in-toto, Provenance — für DevSecOps-Teams und Platform-Engineers.

---

## SBOM (Software Bill of Materials)

### Formate: CycloneDX vs. SPDX

```
CycloneDX (OWASP):
  - JSON oder XML
  - Stärke: Vulnerabilitäten, VEX (Vulnerability Exploitability Exchange)
  - Tools: Syft, CycloneDX-Maven, cdxgen
  - Empfehlung: Für Security-fokussierte SBOMs

SPDX (Linux Foundation, ISO/IEC 5962):
  - JSON, YAML, RDF, Tag-Value
  - Stärke: Lizenz-Compliance, rechtliche Anforderungen
  - Tools: syft, spdx-tools, FOSSA
  - Empfehlung: Für Lizenz-Compliance-SBOMs

WANN WELCHES FORMAT:
  Lieferant (Vendor) → Kunde:     CycloneDX (VEX für Vuln-Status)
  Open-Source-Projekt:            SPDX (Lizenz-Klarheit)
  Intern (Monitoring):            CycloneDX (Dependency-Track Integration)
  Regulatorische Anforderung:     CycloneDX (EU Cyber Resilience Act, FDA)
```

### SBOM generieren mit Syft

```bash
# Installation
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin

# Container-Image SBOM
syft ghcr.io/firma/myapp:1.5.0 -o cyclonedx-json > sbom-myapp-1.5.0.json
syft ghcr.io/firma/myapp:1.5.0 -o spdx-json      > sbom-myapp-1.5.0.spdx.json

# Lokales Verzeichnis / Source Code
syft dir:/app/src -o cyclonedx-json > sbom-source.json

# npm-Projekt
syft dir:. --scope all-layers -o cyclonedx-json

# Go-Modul
syft . -o cyclonedx-json \
    --config .syft.yaml

# .syft.yaml (Konfiguration)
cat << 'EOF' > .syft.yaml
catalogers:
  enabled:
    - javascript-package-cataloger
    - python-package-cataloger
    - go-module-binary-cataloger
    - java-archive-cataloger
output:
  - format: cyclonedx-json
    file: sbom.json
EOF

# SBOM-Inhalt prüfen
syft convert sbom.json -o table    # Lesbare Tabellenansicht
cat sbom.json | jq '.components | length'  # Anzahl Komponenten
cat sbom.json | jq '.components[] | select(.licenses == null)'  # Ohne Lizenz
```

### SBOM in CI/CD (GitHub Actions)

```yaml
# .github/workflows/sbom.yml
name: SBOM Generation & Attestation
on:
  push:
    branches: [main]

jobs:
  sbom:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write       # Für Cosign OIDC

    steps:
      - uses: actions/checkout@v4

      - name: Docker Build
        id: build
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          provenance: true
          sbom: true          # Docker Buildx generiert SBOM automatisch

      # Zusätzlich: Syft SBOM für erweiterte Informationen
      - name: Syft SBOM generieren
        uses: anchore/sbom-action@v0
        with:
          image: ghcr.io/${{ github.repository }}:${{ github.sha }}
          format: cyclonedx-json
          output-file: sbom.cdx.json

      # SBOM an Image-Digest anhängen (OCI Artifact)
      - name: Cosign installieren
        uses: sigstore/cosign-installer@v3

      - name: SBOM als OCI-Artifact attachen
        run: |
          cosign attach sbom \
            --sbom sbom.cdx.json \
            --type cyclonedx \
            ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}

      # SBOM attestieren (kryptografisch signiert)
      - name: SBOM attestieren
        run: |
          cosign attest --yes \
            --predicate sbom.cdx.json \
            --type cyclonedx \
            ghcr.io/${{ github.repository }}@${{ steps.build.outputs.digest }}

      # In Dependency-Track hochladen
      - name: Dependency-Track Upload
        run: |
          curl -X POST https://deptrack.firma.de/api/v1/bom \
            -H "X-Api-Key: ${{ secrets.DEPTRACK_API_KEY }}" \
            -H "Content-Type: multipart/form-data" \
            -F "projectName=myapp" \
            -F "projectVersion=${{ github.sha }}" \
            -F "bom=@sbom.cdx.json"
```

---

## SLSA (Supply-chain Levels for Software Artifacts)

### SLSA-Level-Übersicht

```
SLSA Level 1 — Dokumentation der Build-Pipeline:
  ✓ Provenance vorhanden (wer hat was wann gebaut)
  ✓ Kein besonderes Vertrauen erforderlich
  → Erreicht durch: GitHub Actions mit docker/build-push-action provenance: true

SLSA Level 2 — Hosted Build Service:
  ✓ Provenance von vertrauenswürdiger Plattform signiert
  ✓ Build-Service verhindet Manipulation durch Contributor
  → Erreicht durch: GitHub Actions + Sigstore/Cosign Signierung

SLSA Level 3 — Hardened Build Platform:
  ✓ Build-Umgebung ist isoliert (keine Netzwerkkommunikation während Build)
  ✓ Provenance nicht manipulierbar durch Build-System
  ✓ Source Control: zwei-Personen-Prinzip für Reviews
  → Erreicht durch: Hermetic Builds + GitHub OIDC + Cosign

SLSA Level 4 (veraltet, in SLSA 1.0 aufgegangen):
  → In SLSA v1.0 ersetzt durch Track-System

PRAXISTIPP:
  Für die meisten Unternehmen: SLSA L2 als Ziel (realistisch + wertvoll)
  SLSA L3: Für kritische Infrastruktur, Open-Source-Projekte
```

### SLSA Provenance generieren

```yaml
# GitHub Actions: SLSA Level 3 Provenance für Container
name: SLSA L3 Build
on:
  push:
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write
      attestations: write    # Für GitHub Attestations (SLSA)
    outputs:
      digest: ${{ steps.build.outputs.digest }}
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build + Push
        id: build
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.ref_name }}
          provenance: mode=max    # Maximale SLSA-Provenance-Informationen
          sbom: true

      # GitHub Native Attestation (SLSA L2+ via GitHub Artifact Attestations)
      - name: GitHub Attestation erstellen
        uses: actions/attest-build-provenance@v1
        with:
          subject-name: ghcr.io/${{ github.repository }}
          subject-digest: ${{ steps.build.outputs.digest }}
          push-to-registry: true

  # Provenance verifizieren (in Deploy-Job)
  verify-and-deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: GitHub Attestation verifizieren
        run: |
          gh attestation verify \
            oci://ghcr.io/${{ github.repository }}@${{ needs.build.outputs.digest }} \
            --owner ${{ github.repository_owner }}
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## Sigstore / Cosign — Image Signing

### Keyless Signing (OIDC-basiert)

```bash
# Installation
brew install cosign      # macOS
# oder
curl -O https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64
install cosign-linux-amd64 /usr/local/bin/cosign

# ─── SIGNIEREN (keyless via OIDC) ─────────────────────────────────────────────
# Im GitHub Actions Kontext (OIDC automatisch verfügbar)
cosign sign --yes ghcr.io/firma/myapp@sha256:abc123...

# Lokal (öffnet Browser für OIDC-Auth)
cosign sign ghcr.io/firma/myapp:1.5.0

# Mit Key-Pair (falls keyless nicht möglich)
cosign generate-key-pair   # Erzeugt cosign.key + cosign.pub
cosign sign --key cosign.key ghcr.io/firma/myapp:1.5.0

# ─── VERIFIZIEREN ─────────────────────────────────────────────────────────────
# Keyless: Verifizierung mit OIDC-Identität
cosign verify \
    --certificate-identity-regexp="https://github.com/firma/.*" \
    --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
    ghcr.io/firma/myapp:1.5.0

# Mit Public Key
cosign verify --key cosign.pub ghcr.io/firma/myapp:1.5.0

# ─── IN DEPLOY-PIPELINE ERZWINGEN ────────────────────────────────────────────
# Kubernetes Admission Controller: Sigstore Policy Controller
# Alle Images müssen von CI/CD signiert sein
kubectl apply -f - << 'POLICY'
apiVersion: policy.sigstore.dev/v1beta1
kind: ClusterImagePolicy
metadata:
  name: require-signed-images
spec:
  images:
    - glob: "ghcr.io/firma/**"
  authorities:
    - keyless:
        url: https://fulcio.sigstore.dev
        identities:
          - issuer: https://token.actions.githubusercontent.com
            subjectRegExp: "https://github.com/firma/.*/.github/workflows/.*"
POLICY
```

---

## Dependency-Track — Vulnerability Management

```bash
# Docker Compose Installation
curl -LO https://dependencytrack.org/docker-compose.yml
docker compose up -d

# SBOM automatisch nach Dependency-Track hochladen (CI/CD)
upload_sbom() {
    local project=$1
    local version=$2
    local sbom_file=$3

    # Projekt anlegen (falls nicht vorhanden)
    PROJECT_UUID=$(curl -sf \
        -H "X-Api-Key: $DEPTRACK_API_KEY" \
        "https://deptrack.firma.de/api/v1/project?name=${project}&version=${version}" |
        jq -r '.[0].uuid // empty')

    if [[ -z "$PROJECT_UUID" ]]; then
        PROJECT_UUID=$(curl -sf -X POST \
            -H "X-Api-Key: $DEPTRACK_API_KEY" \
            -H "Content-Type: application/json" \
            "https://deptrack.firma.de/api/v1/project" \
            -d "{\"name\":\"$project\",\"version\":\"$version\"}" |
            jq -r '.uuid')
    fi

    # SBOM hochladen
    BOM_BASE64=$(base64 -w0 "$sbom_file")
    curl -sf -X PUT \
        -H "X-Api-Key: $DEPTRACK_API_KEY" \
        -H "Content-Type: application/json" \
        "https://deptrack.firma.de/api/v1/bom" \
        -d "{\"project\":\"$PROJECT_UUID\",\"bom\":\"$BOM_BASE64\"}"
    echo "SBOM hochgeladen für $project:$version"
}

# Kritische Schwachstellen abrufen (API)
curl -sf \
    -H "X-Api-Key: $DEPTRACK_API_KEY" \
    "https://deptrack.firma.de/api/v1/vulnerability/project/$PROJECT_UUID?severity=CRITICAL" |
    jq '.[] | {component: .components[0].name, vuln: .vulnId, severity: .severity}'
```

---

## OpenSSF Scorecard

```bash
# Scorecard für eigenes Repository ausführen
docker run -e GITHUB_AUTH_TOKEN=$GITHUB_TOKEN \
    gcr.io/openssf/scorecard:stable \
    --repo github.com/firma/myapp \
    --format json | tee scorecard-results.json

# Ergebnisse interpretieren
cat scorecard-results.json | jq '
    .checks[] |
    {name: .name, score: .score, reason: .reason} |
    select(.score < 7)
'

# Wichtigste Scorecard-Checks:
# Branch-Protection:   main-Branch schützen (min. 1 Review)
# Code-Review:         Alle PRs müssen reviewed werden
# Signed-Releases:     Releases mit Cosign signieren
# Token-Permissions:   GitHub Actions Token auf Minimum begrenzen
# Pinned-Dependencies: Actions auf SHA pinnen statt Tags
# SAST:                Statische Analyse in Pipeline
# Vulnerabilities:     Keine bekannten offenen CVEs

# GitHub Actions: Scorecard in Pipeline (wöchentlich)
```

```yaml
name: OpenSSF Scorecard
on:
  schedule:
    - cron: '30 1 * * 1'    # Jeden Montag 01:30
  push:
    branches: [main]

jobs:
  analysis:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      id-token: write
      contents: read
      actions: read
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - uses: ossf/scorecard-action@v2.3.1
        with:
          results_file: results.sarif
          results_format: sarif
          publish_results: true
      - uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: results.sarif
```

---

## Supply Chain Security Checkliste

```markdown
## Supply Chain Security — Compliance-Checkliste

### Source Code (Level 1)
- [ ] Alle Code-Änderungen via Pull Request (kein direkter Push auf main)
- [ ] Branch Protection: mind. 1 Review vor Merge
- [ ] Signed Commits empfohlen (git commit -S)
- [ ] Kein Secret im Code: Gitleaks / GitHub Secret Scanning aktiv

### Dependencies (Level 2)
- [ ] Dependabot oder Renovate für automatische Updates aktiv
- [ ] SBOM bei jedem Release generiert (Syft/CycloneDX)
- [ ] SBOM in Dependency-Track oder Grype auf Schwachstellen geprüft
- [ ] Lizenz-Check: keine GPL in kommerziellen Produkten
- [ ] npm/pip/go: Lockfiles committed (package-lock.json, go.sum, etc.)
- [ ] Private Registry statt direktes Internet (Nexus/Artifactory)

### Build (Level 3)
- [ ] Builds nur in CI/CD (kein "es funktioniert auf meinem Laptop")
- [ ] Build-Logs aufbewahrt (mind. 90 Tage)
- [ ] SLSA Provenance generiert (provenance: true in docker/build-push-action)
- [ ] Hermetic Build angestrebt (keine Netzwerkzugriffe während Build)

### Artifacts (Level 4)
- [ ] Container-Images mit Cosign signiert (keyless via OIDC)
- [ ] SBOM an Image-Digest attestiert
- [ ] Image-Tags auf SHA pinnen (nicht :latest in Produktion!)
- [ ] Trivy / Grype Scan vor jedem Deploy: keine CRITICAL-Lücken

### Deploy (Level 5)
- [ ] Signaturen vor Deploy verifiziert (cosign verify in CD-Pipeline)
- [ ] Policy Controller / OPA-Gatekeeper: nur signierte Images erlaubt
- [ ] Admission Controller: SBOM-Verfügbarkeit prüfen
- [ ] SBOM-Archivierung: mind. 5 Jahre (regulatorische Anforderungen DE)

### Monitoring (Level 6)
- [ ] Neue CVEs in deployed Images: Benachrichtigung < 24h
- [ ] Dependency-Track: Alert bei neuen Critical-Schwachstellen
- [ ] Regelmäßige Neugenerierung von SBOMs (mind. wöchentlich)
- [ ] OpenSSF Scorecard: wöchentlicher Scan, Score ≥ 7/10
```
