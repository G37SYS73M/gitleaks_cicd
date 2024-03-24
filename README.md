# GitLeaks in CI/CD
## Using GitLeaks in CI/CD.
Gitleaks is a SAST tool for detecting and preventing hardcoded secrets like passwords, api keys, and tokens in git repos. Gitleaks is an easy-to-use, all-in-one solution for detecting secrets, past or present, in your code.

> Source Repo : *https://github.com/gitleaks/gitleaks*

### Implementing GitLeaks in CI/CD in GitLab
#### .gitlab-ci.yml using From Source
```YAML
stages:
  - SAST

stage 1:
  stage: SAST
  script:
    - apt-get update -y
    - apt-get install golang -y
    - export source_code=`pwd`
    - cd /tmp
    - git clone https://github.com/gitleaks/gitleaks.git
    - cd gitleaks
    - make build
    - ./gitleaks detect --source $source_code -v --report-path gitleaks_report.json
```
#### .gitlab-ci.yml using Docker (DockerHub)
```YAML
stages:
  - SAST

variables:
  DOCKER_HOST: tcp://docker:2375/
  DOCKER_DRIVER: overlay2

services:
  - docker:dind

stage 1:
  stage: SAST
  script:
    - apt-get update -y
    - apt-get install docker.io -y
    - export source_code=`pwd`
    - docker pull zricethezav/gitleaks:latest
    - docker run -v $source_code:/path zricethezav/gitleaks:latest detect --source="/path"

```
#### Output
```bash
$ ./gitleaks detect --source $source_code -v --report-path gitleaks_report.json
    ○
    │╲
    │ ○
    ○ ░
    ░    gitleaks
Finding:     eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InZ1bG5sYWJBZG1pbiIsImlhdCI6MTY0NjU0OTIyMn0.jnt...
Secret:      eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InZ1bG5sYWJBZG1pbiIsImlhdCI6MTY0NjU0OTIyMn0.jnt...
RuleID:      jwt
Entropy:     5.402993
File:        solutions/solutions.md
Line:        241
Commit:      f2734497fa028573f14311ee884c1a8955e866e1
Author:      xxxxx
Email:       xxxxxxxxxxx@xxxxx.com
Date:        2024-03-24T07:36:21Z
Fingerprint: f2734497fa028573f14311ee884c1a8955e866e1:solutions/solutions.md:jwt:241
```
#### We can use the following format to verify the leak:
```bash
git log -L {StartLine,EndLine}:{File} {Commit}
git log -L 241,241:solutions/solutions.md f2734497fa028573f14311ee884c1a8955e866e1
```
### Pros
- **Custom Scan Configuration**: Gitleaks offers a configuration format you can follow to write your own secret detection rules:
- **Creating a baseline**: When scanning large repositories or repositories with a long history, it can be convenient to use a baseline. When using a baseline, gitleaks will ignore any old findings that are present in the baseline. A baseline can be any gitleaks report.
> To create a gitleaks report, run gitleaks with the ```--report-path``` parameter.
> ```gitleaks detect --report-path gitleaks-report.json```
> This will save the report in a file called gitleaks-report.json. Once as baseline is created it can be applied when running the detect command again:
> ```gitleaks detect --baseline-path gitleaks-report.json --report-path findings.json```
After running the detect command with the ```--baseline-path parameter```, report output (findings.json) will only contain new issues.

### Cons
- **Limited Coverage**: While GitLeaks is effective at detecting certain types of sensitive information, it may not catch all potential leaks or vulnerabilities in a codebase.
- **Dependency on Regular Expressions**: GitLeaks primarily relies on regular expressions to identify patterns of sensitive information. While this approach is effective in many cases, it may not be foolproof and can miss complex or obfuscated patterns.
