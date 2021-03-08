
Static analysis - Scan Image

As packaging applications inside Docker images becomes ubiquitous, more organizations are investing in hardening their Docker images. After you've run your application code through static and dynamic analysis tools, organizations typically leverage a CVE image scanner installed in their Docker registry. This allows you to identify known CVEs before containers are deployed, reducing your risk profile.


Clair
Clair is one of the most popular complex systems to analyse Docker images against open CVE databases. It consists of API and backend analyser components. There is no front-end or CLI for Clair so you have to use other tools to call Clair REST API (fortunately there are some) or write your own.

You need to install Clair Server and Postgres DB to store all vulnerabilities data from different databases. Based on this data analyser check the image that you supply to it. Uploading CVE database takes up 15-30 minutes, but there are ready-to-go Docker containers packaging the downloaded database or one can use SQL script to manually upload all the vulnerability data.

Without CLI or any other front-end tools it is not easy to operate Clair server. It requires to fetch hashes from image Manifests, sending them over to a specific endpoint and performing some voodoo magic to make it work. But in theory it allows you for a flexible images workflow. If you are brave enough to fight against unclear documentation for Clair and its API 


Trivy
URL: https://github.com/knqyf263/trivy

Trivy searches for vulnerabilties of two main types – OS system packages issues (supported Alpine, RedHat (EL), CentOS, Debian GNU, Ubuntu) and dependencies issues (based on Gemfile.lock, Pipfile.lock, composer.lock, package-lock.json, yarn.lock, Cargo.lock files)

Unlike Clair, Trivy is capable of scanning the Docker images from Docker registry, from local registry and even from .tar archive file with a Docker image export.

This tool can optionally show only CVEs, for which there are fixes and ignore CVEs, based on a local whitelist (.trivyignore). It can filter vulnerabilities based on criticality level, can fail with a specific exit-code (which could be useful in CI/CD) and can output the results on a screen or in a json file.

Now we'll check nginx image for vulnerabilities 

```execute
docker run ghcr.io/aquasecurity/trivy:latest image nginx:latest 
```

lets check the vulnerabilities in nginx 1.16-alpine version and look for Critical one

```execute
docker run ghcr.io/aquasecurity/trivy:latest image nginx:latest | grep CRITICAL
```

You will see CRITICAL: 1


Now try another version and see if that vulnerability is fixed in that version 

```execute
docker run ghcr.io/aquasecurity/trivy:latest image nginx:1.18-alpine | grep CRITICAL
```

You will see CRITICAL: 0, that means CVE-2019-20367 is fixed with nginx 1.18-alpine 

Harbor provides static analysis of vulnerabilities in images through the open source projects Trivy and Clair.

To use Trivy or Clair or both, you must enable Trivy, Clair, or both when you install your Harbor instance (by appending installation options --with-trivy, --with-clair, or both).

You can also connect Harbor to your own instance of Trivy or Clair, or to other vulnerability scanners, through Harbor’s embedded interrogation service. These scanners can be configured in the Harbor interface at any time after installation. For the list of additional scanners that are currently supported, see the Harbor Compatibility List.

It might be necessary to connect Harbor to other scanners for corporate compliance reasons, or because your organization already uses a particular scanner. Different scanners also use different vulnerability databases, capture different CVE sets, and apply different severity thresholds. By connecting Harbor to more than one vulnerability scanner, you broaden the scope of your protection against vulnerabilities.

You can manually initiate scanning on a particular image, or on all images in Harbor. Additionally, you can set a policy to scan all images at specific intervals.

