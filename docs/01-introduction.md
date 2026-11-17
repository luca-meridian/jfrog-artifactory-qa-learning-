# 01 - Introduction to JFrog Artifactory

## What is JFrog Artifactory?

JFrog Artifactory is a tool used to store and manage software packages and files, often referred to as **artifacts**.

When developers build an application, the build process produces files that can be deployed or used by other teams. Artifactory provides a central place to store these files so they can be accessed when needed.

## What is an Artifact?

An artifact is a file produced or used during software development.

Some examples include:

| Artifact | Example |
|---|---|
| Java application | `application-1.0.jar` |
| Docker image | `my-app:1.0` |
| ZIP file | `application-build.zip` |
| Python package | `my-package-1.0.whl` |

Artifactory can manage different types of artifacts, depending on the edition being used.

## Why is Artifactory Useful for QA?

As QA engineers, we often need access to specific application builds for testing.

Artifactory can help us:

- Find and download a particular application version.
- Access older builds when investigating defects.
- Make sure we're testing the correct build.
- Understand where applications are stored after being built by a CI/CD pipeline.

For example, if a defect is found in version `1.2` of an application, we may need to retrieve that exact version to reproduce the issue.

## Where Does Artifactory Fit In?

A typical development workflow might look something like this:

```text
Developer writes code
        |
        v
Code is pushed to Git
        |
        v
CI/CD pipeline builds the application
        |
        v
Artifact is stored in Artifactory
        |
        v
QA retrieves the build for testing
```

Artifactory doesn't replace Git or a CI/CD tool. It works alongside them.

Git manages source code, CI/CD tools automate builds and deployments, and Artifactory stores the resulting artifacts.

## What's Next?

In the next section, we'll look at how to run Artifactory locally using Docker and PostgreSQL.

[Next: 02 - Local Setup](02-local-setup.md)
