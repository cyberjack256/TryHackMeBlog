# OWASP Top 10 Walkthrough
1. [Broken Access Control](#broken_access_control)
2. [Cryptographic Failures](#cryptographic_failures)
3. [Injection](#injection)
4. [Insecure Design](#insecure_design)
5. [Security Misconfiguration](#security_misconfiguration)
6. [Vulnerable & Outdated Components](#vulnerable_&_outdated_components)
7. [Identification & Authentication Failures](#identification_&_authentication_failures)
8. [Software & Data Integrity Failures](#software_&_data_integrity_failures)
9. [Secure Logging & Monitoring Failures](#secure_logging_&_monitoring_failures)
10. [Server-Side_Request_Forgery](#server-side_request_forgery)


## Broken_Access_Control
The 34 Common Weakness Enumerations (CWEs) mapped to Broken Access Control had more occurrences in applications than any other category.

## Cryptographic_Failures
 `Cryptographic Failures` previously `Sensitive Data Exposure`; which was a broad symptom rather than a root cause. The renewed focus of the category is on failures related to cryptography which often leads to sensitive data exposure or system compromise.

## Injection

includes Cross-site Scripting

## Insecure_Design

`Insecure Design` s a new category from 2021, with a focus on risks related to design flaws. If we genuinely want to “move left” as an industry, it calls for more use of threat modeling, secure design patterns and principles, and reference architectures.

## Security_Misconfiguration

The former category `XML External Entities (XXE)` is part of this category.

## Vulnerable_&_Outdated_Components

`Vulnerable & Outdated Components` was previously `Using Components with Known Vulnerabilities`

## Identification_&_Authentication_Failures

`Identification & Authentication Failures` was previously `Broken Authentication` and is sliding down from the second position, and now includes CWEs that are more related to identification failures.The Increased availability of standardized frameworks has reduced the target surface.

## Software_&_Data_Integrity_Failures

`Software & Data Integrity Failures`is a new category from 2021, focusing on making assumptions related to software updates, critical data, and CI/CD pipelines without verifying integrity.One of the highest weighted impacts from Common Vulnerability and Exposures/Common Vulnerability Scoring System (CVE/CVSS) data mapped to the 10 CWEs in this category.

## Secure_Logging_&_Monitoring_Failures

`Secure Logging & Monitoring Failures` was previously `Insufficient Logging & Monitoring` and moving up from #10 previously. This category is challenging to test for, and isn’t well represented in the CVE/CVSS data. However, failures in this category can directly impact visibility, incident alerting, and forensics.

# Server-Side_Request_Forgery