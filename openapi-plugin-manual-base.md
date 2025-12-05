# OpenAPI Plugin Setup Guide for SonarQube 2025

This guide explains how to compile, install and configure OpenAPI analysis plugins for SonarQube 2025.3.0.

## Prerequisites

- **JDK 11+** (tested with JDK 17)
- **Maven 3.6+**
- **SonarQube 2025.3.0** running
- **Git**

## Architecture Overview

The OpenAPI analysis in SonarQube requires **two plugins**:

1. **sonar-openapi** - Base plugin providing:
   - `openapi` language support
   - OpenAPI file parser and scanner sensor
   - Basic OpenAPI validation rules

2. **sonaropenapi-rules** - Rules plugin providing:
   - 100+ custom OpenAPI quality rules (OAR001-OAR109)
   - Advanced validation and best practices

## Step 1: Clone and Build sonar-openapi (Base Plugin)

### 1.1 Clone the repository

```bash
cd /home/sq/plugs/sources
git clone https://github.com/apiaddicts/sonar-openapi.git
cd sonar-openapi
```

### 1.2 Update SonarQube API version

The plugin needs to be updated for SonarQube 2025 compatibility:

```bash
# Update pom.xml to use compatible SonarQube API version
sed -i 's/<sonar.version>6.7.4<\/sonar.version>/<sonar.version>9.4.0.54424<\/sonar.version>/' pom.xml
```

### 1.3 Build the plugin

```bash
# Build without tests to avoid compatibility issues
mvn clean install -DskipTests -Dmaven.javadoc.skip=true
```

### 1.4 Verify build success

```bash
# Check if JAR was created
ls -la sonar-openapi-plugin/target/sonar-openapi-plugin-*.jar
```

Expected output:
```
sonar-openapi-plugin-1.1.1.jar
```

## Step 2: Clone and Build sonaropenapi-rules (Rules Plugin)

### 2.1 Clone the repository

```bash
cd /home/sq/plugs/sources
git clone https://github.com/apiaddicts/sonaropenapi-rules.git
cd sonaropenapi-rules
```

### 2.2 Update SonarQube API version

```bash
# Update to compatible version
sed -i 's/<sonar.version>9.4.0.54424<\/sonar.version>/<sonar.version>9.4.0.54424<\/sonar.version>/' pom.xml
```

### 2.3 Build the plugin

```bash
# Build without tests
mvn clean package -DskipTests -Dmaven.javadoc.skip=true
```

### 2.4 Verify build

```bash
ls -la target/sonaropenapi-rules-*.jar
```

Expected output:
```
sonaropenapi-rules-community-1.2.3.jar
```

## Step 3: Install Plugins in SonarQube

### 3.1 Stop SonarQube

```bash
cd /home/sq/sonarqube-2025.3.0.108892
sudo -u sonarqube ./bin/linux-x86-64/sonar.sh stop
```

### 3.2 Install plugins

```bash
# Copy both plugins to extensions/plugins
cp /home/sq/plugs/sources/sonar-openapi/sonar-openapi-plugin/target/sonar-openapi-plugin-1.1.1.jar \
   /home/sq/sonarqube-2025.3.0.108892/extensions/plugins/

cp /home/sq/plugs/sources/sonaropenapi-rules/target/sonaropenapi-rules-community-1.2.3.jar \
   /home/sq/sonarqube-2025.3.0.108892/extensions/plugins/
```

### 3.3 Start SonarQube

```bash
sudo -u sonarqube ./bin/linux-x86-64/sonar.sh start
```

### 3.4 Verify installation

Check logs for successful plugin loading:

```bash
cd /home/sq/sonarqube-2025.3.0.108892
grep -i "deploy.*openap" logs/web.log
```

Expected output:
```
Deploy SonarOpenApi / 1.1.1 / [hash]
Deploy OpenAPI Custom / 1.2.3 / null
```

## Step 4: Verify Language and Rules

### 4.1 Check if 'openapi' language is available

```bash
curl -u 'admin:YOUR_PASSWORD' http://localhost:9000/api/languages/list | jq '.languages[] | select(.key == "openapi")'
```

Expected output:
```json
{
  "key": "openapi",
  "name": "OpenAPI"
}
```

### 4.2 Check if OpenAPI rules are loaded

```bash
curl -u 'admin:YOUR_PASSWORD' 'http://localhost:9000/api/rules/search?q=OAR001&ps=5' | jq '.rules[0].key'
```

Expected output:
```
"openapi-custom:OAR001"
```

## Step 5: Configure Quality Profile

### 5.1 Create OpenAPI Quality Profile

1. Go to **Quality Profiles** in SonarQube UI
2. Click **Create**
3. Name: `OpenAPI Enhanced`
4. Language: `openapi`
5. Click **Create**

### 5.2 Activate OpenAPI Rules

1. In the new profile, click **Activate More**
2. Filter by **Repository: openapi-custom**
3. Select and activate desired rules (OAR001-OAR109)
4. Also activate rules from **Repository: openapi**

## Step 6: Configure Project for OpenAPI Analysis

### 6.1 Create sonar-project.properties

In your OpenAPI project root:

```properties
# Project identification
sonar.projectKey=my-openapi-project
sonar.projectName=My OpenAPI Project
sonar.projectVersion=1.0.0

# Language and sources
sonar.language=openapi
sonar.sources=.

# Server connection
sonar.host.url=http://localhost:9000
sonar.token=YOUR_TOKEN_HERE

# Encoding
sonar.sourceEncoding=UTF-8

# Disable SCM detection
sonar.scm.disabled=true

# Exclude config files
sonar.exclusions=**/sonar-project.properties
```

### 6.2 Handle file pattern conflicts

If you have `.yaml` files that should be analyzed as OpenAPI (not YAML), configure patterns:

```properties
# Exclude OpenAPI files from YAML analysis
sonar.lang.patterns.yaml=!**/api.yaml,**/*.yaml,**/*.yml

# Or specify exact files for OpenAPI
sonar.sources=api.yaml,swagger.json
```

## Step 7: Run Analysis

### 7.1 Install SonarScanner

```bash
# Download and extract
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
unzip sonar-scanner-cli-5.0.1.3006-linux.zip

# Add to PATH
export PATH=$PATH:/path/to/sonar-scanner-5.0.1.3006-linux/bin
```

### 7.2 Run analysis

```bash
# From project directory
sonar-scanner
```

### 7.3 Expected output

```
INFO: Quality profile for openapi: OpenAPI Enhanced
INFO: Sensor OpenAPI Scanner Sensor [openapicustom]
INFO: OpenAPI Scanner called for the following files: [api.yaml]
INFO: Sensor OpenAPI Scanner Sensor [openapicustom] (done) | time=270ms
INFO: ANALYSIS SUCCESSFUL
```

## Step 8: View Results

1. Open SonarQube UI
2. Navigate to your project
3. Check **Issues** tab
4. Filter by **Language: openapi**
5. Review found violations

## Common Issues and Solutions

### Issue 1: Plugin compatibility errors

**Error:** `Plugin OpenAPI Custom is ignored because its base plugin [openapi] is not installed`

**Solution:** Ensure both plugins are installed and base plugin loads first.

### Issue 2: Language detection conflicts

**Error:** `Language of file 'api.yaml' can not be decided`

**Solution:** Configure language patterns as shown in Step 6.2

### Issue 3: No issues found

**Problem:** Clean OpenAPI spec shows no violations

**Solution:** Add intentional errors to test:
```yaml
openapi: 3.0.4  # Wrong version
servers:
  - url: http://api.example.com  # HTTP instead of HTTPS
info:
  contact:
    email: invalid-email  # Invalid email format
```

### Issue 4: Build failures

**Error:** Maven compilation errors

**Solution:** 
- Use JDK 11+
- Skip tests: `-DskipTests`
- Skip javadoc: `-Dmaven.javadoc.skip=true`

## File Structure After Setup

```
/home/sq/
├── plugs/sources/
│   ├── sonar-openapi/           # Base plugin source
│   └── sonaropenapi-rules/      # Rules plugin source
└── sonarqube-2025.3.0.108892/
    └── extensions/plugins/
        ├── sonar-openapi-plugin-1.1.1.jar
        └── sonaropenapi-rules-community-1.2.3.jar
```

## Available Rules

The setup provides access to 100+ OpenAPI quality rules including:

- **Security:** HTTPS enforcement, authentication validation
- **Structure:** Schema validation, parameter definitions  
- **Documentation:** Required fields, examples, descriptions
- **Standards:** OpenAPI spec compliance, naming conventions

## Troubleshooting Commands

```bash
# Check plugin loading
grep "Deploy.*OpenAPI" /home/sq/sonarqube-2025.3.0.108892/logs/web.log

# Check language availability  
curl -u admin:PASS http://localhost:9000/api/languages/list

# Check rules availability
curl -u admin:PASS "http://localhost:9000/api/rules/search?q=openapi-custom&ps=5"

# Restart SonarQube
cd /home/sq/sonarqube-2025.3.0.108892
sudo -u sonarqube ./bin/linux-x86-64/sonar.sh restart
```

---

**This setup provides complete OpenAPI analysis capabilities in SonarQube 2025.3.0!**
