# Quarkus Maven Parent

This repository provides a standardized **Maven parent POM** for Quarkus-based Java projects.
It centralizes common dependency management, compiler settings, and plugin configurations to ensure consistency and reduce duplication across multiple microservices or modules.

## ✨ Features

* Java 17 compiler setup
* Quarkus BOM import (version-controlled)
* Centralized Quarkus dependencies (e.g. `quarkus-arc`, `quarkus-rest`, `quarkus-hibernate-orm-panache`, etc.)
* Lombok integration
* Maven compiler plugin preconfigured
* Ready for use with GitHub Packages

## 🚀 How to Use It in a Project

In your project `pom.xml`, declare this parent:

```xml
<parent>
    <groupId>com.gastbob40</groupId>
    <artifactId>quarkus-maven-parent</artifactId>
    <version>1.0.0</version>
</parent>
```

You can now omit versions for all dependencies declared in the parent POM.

Then, configure the GitHub Packages repository (required for Maven to find the parent):

```xml
<repositories>
    <repository>
        <id>github</id>
        <url>https://maven.pkg.github.com/gastbob40/quarkus-maven-parent</url>
    </repository>
</repositories>
```

## ⚠️ Authentication Required

Although the repository is public, **GitHub Packages always requires authentication** to download artifacts, even public ones.

### 🔑 Configure `~/.m2/settings.xml`

To allow Maven to access GitHub Packages, you need to create or update the `~/.m2/settings.xml` file on your local machine or CI/CD runner:

```xml
<settings>
  <servers>
    <server>
      <id>github</id>
      <username>{YOUR USERNAME}</username>
      <password>YOUR_GITHUB_TOKEN</password>
    </server>
  </servers>
</settings>
```

* `id` must match the one in the `<repository>` section (`github`)
* `username` is your GitHub username
* `password` is your GitHub token (see below)

### 🔐 Token Permissions

You need to generate a **Personal Access Token (PAT)** from your GitHub account:

* Go to [https://github.com/settings/tokens](https://github.com/settings/tokens)
* Create a new token (classic or fine-grained)
* Add the following scopes:

    * `read:packages` (mandatory)
    * `repo` (only if the package repo is private)

Use this token as the password in your `settings.xml`.

> ⚠️ Even if your repository is public, unauthenticated access to GitHub Packages will fail with `401 Unauthorized`.

## 🏠 Use in CI/CD

When using this parent in another GitHub project, configure GitHub Actions to authenticate properly.

In `.github/workflows/ci.yml`, add a step to setup Java and authenticate with GitHub Packages:

```yaml
jobs:
  my-job:
      # ...
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'adopt'

      - name: Configure Maven settings for GitHub Packages
        run: |
          mkdir -p ~/.m2
          cat > ~/.m2/settings.xml <<EOF
          <settings>
            <servers>
              <server>
                <id>github</id>
                <username>{YOUR USERNAME}</username>
                <password>${{ secrets.PAT }}</password>
              </server>
            </servers>
          </settings>
          EOF

      # ...
```

Make sure `secrets.PAT` is set to a Personal Access Token with the correct permissions by going to your repository settings and adding a new secret in your actions.

## 🔍 Troubleshooting

### 401 Unauthorized when building

> Maven cannot resolve the parent POM because GitHub Packages requires authentication.

**Solution:** Ensure your `settings.xml` is configured and that your token is valid and has `read:packages` scope.
