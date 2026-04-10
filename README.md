
# Resolute P2 Mirror (Reproducible Target Platform)

This repository hosts a **consolidated Eclipse p2 repository** for building [Resolute](https://github.com/loonwerks/Resolute) in a **reproducible and offline-capable way**.

It is generated using **Tycho’s `mirror-target-platform`** from a helper Maven project, rather than manually mirroring individual installable units (IUs).

---

## 🎯 Goals

- Eliminate dependency on live Eclipse/OSATE update sites
- Provide a **single stable p2 repository**
- Support **cross-platform builds** (Windows, Linux, macOS)
- Avoid fragile, manual IU curation

---

## 📦 Mirror Contents

This repository is a **valid p2 update site**, containing:

- `artifacts.jar`
- `content.jar`
- `plugins/`
- `features/`

It is served via GitHub Pages:

👉 **Mirror URL**  
```
https://iamundson.github.io/resolute-p2-mirror/
```

---

## 🚀 How to Use the Mirror

Update all repository locations in Resolute’s `.target` file:

```xml
<repository location="https://iamundson.github.io/resolute-p2-mirror/"/>
```

Then clear caches and build:

```bash
rm -rf ~/.m2/repository/.cache/tycho
rm -rf ~/.m2/repository/p2

mvn -U clean verify
```

---

## 🏗️ How the Mirror is Generated

The mirror is produced by a **separate helper Maven project** using:

- Tycho 4.0.8  
- Maven 3.9+  
- JDK 17+  

---

### Key Design Decisions

#### 1. Use Tycho’s computed target platform

```bash
mvn org.eclipse.tycho:target-platform-configuration:4.0.8:mirror-target-platform
```

This mirrors the **fully resolved dependency graph**.

---

#### 2. Include full OSATE repository (critical)

The original Resolute `.target` file includes only the OSATE **py4j repo**, not the main OSATE update site.

To fix this, the helper project includes:

```xml
<repositories>
  <repository>
    <id>osate-stable</id>
    <layout>p2</layout>
    <url>https://osate-build.sei.cmu.edu/download/osate/stable/2.14.0-vfinal/updates/</url>
  </repository>

  <repository>
    <id>osate-py4j</id>
    <layout>p2</layout>
    <url>https://osate-build.sei.cmu.edu/p2/py4j</url>
  </repository>
</repositories>
```

This eliminates the need for manual OSATE IU seeding.

---

#### 3. Multi-platform environments

```xml
<environments>
  <environment>
    <os>win32</os>
    <ws>win32</ws>
    <arch>x86_64</arch>
  </environment>
  <environment>
    <os>linux</os>
    <ws>gtk</ws>
    <arch>x86_64</arch>
  </environment>
  <environment>
    <os>macosx</os>
    <ws>cocoa</ws>
    <arch>x86_64</arch>
  </environment>
</environments>
```

---

#### 4. Disable Tycho’s implicit JRE

```xml
<executionEnvironment>none</executionEnvironment>
<executionEnvironmentDefault>none</executionEnvironmentDefault>
```

---

## 🔧 Mirror Generation Steps

From the helper project directory:

```bash
~/opt/apache-maven-3.9.9/bin/mvn -U \
  org.eclipse.tycho:target-platform-configuration:4.0.8:mirror-target-platform \
  -Ddestination="$(pwd)/target-definition/target/target-platform-repository" \
  -Dname="Resolute Target Platform"
```

---

## 🌐 Publishing to GitHub Pages

1. Copy contents of:

```
target-definition/target/target-platform-repository
```

2. Publish to `gh-pages` branch:

```bash
git checkout --orphan gh-pages
git rm -rf . || true

rsync -a --delete <mirror-dir>/ ./
touch .nojekyll

git add .
git commit -m "Publish mirror"
git push origin gh-pages --force
```

3. Enable GitHub Pages:

- Settings → Pages  
- Source: `gh-pages` branch  
- Folder: `/ (root)`  

---

## ✅ Sanity Checks

### Check mirror contains an IU

```bash
unzip -p content.jar content.xml | grep -n "org.osate.xtext.aadl2"
```

### Check hosted mirror

```bash
curl -L https://iamundson.github.io/resolute-p2-mirror/content.jar -o /tmp/content.jar
unzip -p /tmp/content.jar content.xml | grep -n "org.osate"
```

---

## 🧠 Lessons Learned

- Manual IU mirroring is brittle  
- Tycho’s computed target platform is the correct approach  
- The `.target` file alone may be incomplete  
- External p2 repos in `pom.xml` must be included  
- Cross-platform builds require explicit environments  
- GitHub Pages works well for hosting p2 repositories  

---

## 🔄 Regeneration Workflow

1. Update helper project if needed  
2. Run mirror generation  
3. Publish to GitHub Pages  
4. Clear caches  
5. Rebuild Resolute  

---

## 🏁 Result

- Deterministic builds  
- Offline capability  
- Single mirror URL  
- No dependency drift  
- No manual IU maintenance  
