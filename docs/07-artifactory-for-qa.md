# 07 - Using Artifactory for QA

Now that we've looked at managing artifacts, let's see how Artifactory can be useful during testing.

One common example is retrieving a specific application version when investigating a defect.

## Example: Investigating a Defect

Imagine a defect has been reported in version `1.1` of an application.

We want to compare it with version `1.0` to help understand whether the issue was introduced in the newer build.

In Artifactory, navigate to:

**Platform → Artifactory → Artifacts → qa-test-builds**

Our repository contains three versions:

```text
qa-test-builds/
├── sample-app-1.0.zip
├── sample-app-1.1.zip
└── sample-app-1.2.zip
```

## Downloading and Verifying the Build

Download `sample-app-1.1.zip` from Artifactory.

Before using the file, we can check that it matches the artifact stored in the repository.

Artifactory records a **SHA-256 checksum** for each artifact.

On macOS, we can calculate the checksum of our downloaded file using:

```bash
shasum -a 256 sample-app-1.1.zip
```

Compare the result with the SHA-256 value shown in Artifactory.

If the values match, we know the downloaded file has the same contents as the one stored in Artifactory.

## Comparing Versions

We can also download version `1.0` and compare it with version `1.1`.

Our sample ZIP files contain a text file identifying the application version.

To inspect version `1.0`:

```bash
unzip -p sample-app-1.0.zip application.txt
```

Output:

```text
Sample Application - Version 1.0
```

And version `1.1`:

```bash
unzip -p sample-app-1.1.zip application.txt
```

Output:

```text
Sample Application - Version 1.1
```

These are example files rather than working applications, but they demonstrate how we can retrieve and identify different builds.

In a real investigation, we would run both application versions and check whether the defect occurs in each.

## Recording the Build Used for Testing

When raising or investigating defects, it's useful to record which build was tested.

For example:

| Information | Example |
|---|---|
| Application | Sample Application |
| Version | 1.1 |
| Artifact | `sample-app-1.1.zip` |
| Repository | `qa-test-builds` |
| Defect reference | BUG-101 |

We could also record the artifact checksum or build number to make the exact file easier to identify.

## Why Does This Matter?

Artifactory makes it easier for QA to:

- Retrieve specific application builds.
- Access previous versions for regression testing.
- Verify downloaded artifacts.
- Identify which build was used during testing.
- Share consistent builds across different environments.

This becomes particularly useful when applications are built and published automatically through CI/CD pipelines.

## Next

[08 - Using the REST API](08-rest-api.md)
