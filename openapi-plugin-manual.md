# How to Compile sonaropenapi-rules Plugin for Beginners

This guide will walk you through the process of compiling the sonaropenapi-rules plugin for SonarQube 2025.3.0 from source code.

## Prerequisites

Before you start, make sure you have the following installed on your system:

### 1. Java Development Kit (JDK)
```bash
# Check if Java is installed
java -version

# If not installed, install OpenJDK
sudo apt update
sudo apt install openjdk-17-jdk
```

### 2. Apache Maven
```bash
# Check if Maven is installed
mvn -version

# If not installed, install Maven
sudo apt install maven
```

### 3. Git
```bash
# Check if Git is installed
git --version

# If not installed, install Git
sudo apt install git
```

## Step 1: Download the Plugin Source Code

Clone the sonaropenapi-rules repository from GitHub:

```bash
# Create a directory for the project
mkdir -p ~/sonar-plugins
cd ~/sonar-plugins

# Clone the repository
git clone https://github.com/apiaddicts/sonaropenapi-rules.git
cd sonaropenapi-rules
```

## Step 2: Check Current Configuration

Check the current pom.xml to see the SonarQube API version:

```bash
# View the current configuration
cat pom.xml | grep -A 5 -B 5 "sonar.version"
```

You'll see something like:
```xml
<properties>
    <sonar.version>8.7.0.41497</sonar.version>
    <!-- ... other properties ... -->
</properties>
```

## Step 3: Update SonarQube API Version

For SonarQube 2025.3.0, we need to update the API version. Update the pom.xml file:

```bash
# Update the SonarQube API version to 9.4.0.54424 (compatible with 2025.3.0)
sed -i 's/<sonar.version>8.7.0.41497<\/sonar.version>/<sonar.version>9.4.0.54424<\/sonar.version>/' pom.xml

# Verify the change
cat pom.xml | grep "sonar.version"
```

## Step 4: Compile the Plugin

Compile the plugin using Maven:

```bash
# Clean and compile the project
mvn clean compile

# If compilation is successful, proceed to packaging
mvn package -DskipTests -Dmaven.javadoc.skip=true
```

The compilation process will:
1. Download all required dependencies
2. Compile the Java source code
3. Run tests (if not skipped)
4. Package the plugin into a JAR file

## Step 5: Locate the Compiled Plugin

After successful compilation, find the JAR file:

```bash
# List the generated JAR files
ls -la target/*.jar

# You should see something like:
# -rw-r--r-- 1 user user 2373710 Dec 5 12:00 sonaropenapi-rules-community-1.2.3.jar
```

## Step 6: Prepare Plugin for Installation

Create a backup copy and prepare the plugin for installation:

```bash
# Create a backup directory
mkdir -p ~/sonar-plugins/compiled

# Copy the compiled plugin
cp target/sonaropenapi-rules-community-1.2.3.jar ~/sonar-plugins/compiled/

# Rename for clarity (optional)
cd ~/sonar-plugins/compiled
mv sonaropenapi-rules-community-1.2.3.jar sonaropenapi-rules-1.2.3.jar
```

## Step 7: Install Plugin in SonarQube

Stop SonarQube, install the plugin, and restart:

```bash
# Stop SonarQube
cd /path/to/sonarqube
./bin/linux-x86-64/sonar.sh stop

# Copy plugin to plugins directory
cp ~/sonar-plugins/compiled/sonaropenapi-rules-1.2.3.jar ./extensions/plugins/

# Start SonarQube
./bin/linux-x86-64/sonar.sh start
```

## Step 8: Verify Installation

Check that the plugin is loaded:

```bash
# Check plugin status via API
curl -u admin:password http://localhost:9000/api/plugins/installed | jq '.plugins[] | select(.name | contains("OpenAPI"))'

# Check available rules
curl -u admin:password http://localhost:9000/api/rules/search?q=openapi&ps=5
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Maven Compilation Fails
```bash
# Clear Maven cache
mvn clean
rm -rf ~/.m2/repository

# Try again
mvn compile
```

#### 2. Plugin Doesn't Load
- Check SonarQube logs: `tail -f logs/sonar.log`
- Ensure correct file permissions: `chown sonarqube:sonarqube extensions/plugins/*.jar`
- Remove old plugin versions before installing new one

#### 3. API Version Compatibility
If the plugin still doesn't work, try a different API version:

```bash
# Try different API versions
# For SonarQube 9.x: 9.9.0.65466
# For SonarQube 10.x: 10.3.0.82913

sed -i 's/<sonar.version>9.4.0.54424<\/sonar.version>/<sonar.version>9.9.0.65466<\/sonar.version>/' pom.xml
mvn clean package -DskipTests -Dmaven.javadoc.skip=true
```

#### 4. Java Version Issues
Ensure you're using Java 17:

```bash
# Check Java version
java -version
# Should show: openjdk version "17.x.x"

# Set JAVA_HOME if needed
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
```

## Advanced Usage

### Custom Rules
You can add your own OpenAPI rules by:

1. Creating new Java classes in `src/main/java/apiaddicts/sonar/openapi/checks/`
2. Implementing the `OpenAPICustomCheck` interface
3. Registering rules in `OpenAPICustomPlugin.java`

### Testing
```bash
# Run tests
mvn test

# Generate test coverage report
mvn jacoco:report
```

## File Structure After Compilation

```
sonaropenapi-rules/
├── pom.xml                          # Maven configuration
├── src/
│   ├── main/java/                   # Source code
│   ├── test/java/                   # Test code
│   └── test/resources/              # Test resources
├── target/
│   ├── classes/                     # Compiled classes
│   ├── sonaropenapi-rules-community-1.2.3.jar  # Main plugin JAR
│   └── sonaropenapi-rules-community-1.2.3-sources.jar  # Sources JAR
└── README.md                        # Documentation
```

## Support

- **GitHub Issues**: https://github.com/apiaddicts/sonaropenapi-rules/issues
- **Community**: https://discord.gg/ZdbGqMBYy8
- **Documentation**: https://github.com/apiaddicts/sonaropenapi-rules#readme

---

**Note**: This guide is for educational purposes. The compiled plugin should work with SonarQube 2025.3.0, but may require additional testing and validation for production use.
