# SAST Tools Comparison

## Objective

This section aims to accomplish the objectives listed as 7<sup>th</sup> point of [`Task 1`](../problem-statement/#task-1) under the [Problem Statement](../problem-statement).

## Comparing the tools

The tools used for SAST in the previous phase allowed us to scan for vulnerabilities in our source code of the application by applying various analysis methods. Each tool had a different approach and performed differently for the same application. The following comparison allows us to compare the tools used and check the different types of vulnerabilities identified.

A multi-tool approach allows us to have a more thorough analysis of the application as each tool has its own way of approaching and scanning the application. What one tool might find safe, might be flagged as a vulnerability by another tool and vice versa. Hence for production applications the tool with the least false positives and proper vulnerability identification must be the ideal choice.

The table below provides a quick summary of the tools and their outcomes:

|Rank   |Name               |Vulnerabilities|
|---    |---                |---            |
|1      |njsscan            |26             |
|2      |snyk.io            |12             |
|3      |insider            |3              |
|4      |Sonarqube Scanner  |0              |

### njsscan

njsscan was able to identify a total of `26` issues of different types in our application. It discovered the vulnerabilities of the following standards:

|Standard                       |Description                                                                                    |No. of vulnerabilities |
|------------                   |-------------                                                                                  |----------             |
|A1: Injection                  |CWE-89: Improper Neutralization of Special Elements used in an SQL Command ('SQL Injection')   |4                      |
|A2: Broken Authentication      |CWE-522: Insufficiently Protected Credentials                                                  |6                      |
|A3: Sensitive Data Exposure    |CWE-798: Use of Hard-coded Credentials                                                         |1                      |
|A5: Broken Access Control      |CWE-770: Allocation of Resources Without Limits or Throttling                                  |1                      |
|A6: Security Misconfiguration  |CWE-693: Protection Mechanism Failure                                                          |13                     |
|A8: Insecure Deserialization   |CWE-502: Deserialization of Untrusted Data                                                     |1                      |

The complete report can be accessed [here](){target="_blank"}.

### insider

insider was able to identify `3` vulnerabilities, which were dependency-based. 

|Title                      |CWE    |Severity   |Dependency Name|
|----                       |----   |----       |----           |
|Code Execution through IIFE|CWE-502|Critical   |node-serialize |
|Arbitrary Code Execution   |CWE-94 |Critical   |math.js@3.17.0 |
|Arbitrary Code Execution   |CWE-94 |Critical   |math.js@3.17.0 |

However, it gave the application code a score of `100/100` which means it was unable to scan the application properly.

```
Score Security 100/100

Vulnerability	Number
High		    0 
Medium		    0 
Low		        0 
Total		    0 
``` 

The reason for not providing an actual security score is the lack of RegEx based SAST rules for Javascript in insider. Since it is an open-source tool, the rule set is limited.

The complete reportsnake be accessed [here](){target="_blank"}.

### snyk.io

snyk was able to identify `9` High Priority issues and `3` Medium Priority issues.

The following vulnerabilities are High Severity and have a high priority score:

|Title                      |Introduced through         |Priority Score |
|---                        |---                        |---            |
|Denial of Service (DoS)    |express-fileupload@0.4.0   |704            |
|Arbitrary Code Execution   |mathjs@3.10.1              |704            |
|Arbitrary Code Execution   |node-serialize@0.0.4       |704            |
|Prototype Pollution        |express-fileupload@0.4.0   |696            |
|Arbitrary Code Execution   |mathjs@3.10.1              |654            |
|Insecure Encryption        |bcrypt@1.0.3               |589            |
|Arbitrary Code Execution   |mathjs@3.10.1              |579            |
|Arbitrary Code Execution   |mathjs@3.10.1              |579            |
|Prototype Pollution        |mathjs@3.10.1              |579            |

The following vulnerabilities were classified as Medium Severity:

|Title                      |Introduced through |Priority Score |
|---                        |---                |---            |
|Cryptographic Issues       |bcrypt@1.0.3       |616            |
|Arbitrary Code Execution   |mathjs@3.10.1      |494            |
|Arbitrary Code Execution   |mathjs@3.10.1      |494            |       

The complete report can be accessed [here](){target="_blank"}.

### Sonarqube Scanner

Sonarqube Scanner provided us with a report that comprised of `5` bugs and `0` vulnerabilities. All of the bugs were related to `Unexpected missing generic font family` which seems to be a syntax error, making Sonarqube Scanner the least useful tool. 

Upon a check of their website, I discovered that they offer only a fraction of rules for Javascript as compared to Java/C#/C or C++, due to which it failed at identifying vulnerabilities in our application which is entirely node.js based.

## Conclusion

njsscan proved to be the most powerful tool among the 4 as it was able to identify the vulnerabilities in the application code as well as dependency-based vulnerabilities. snyk.io and insider are runner ups as they mostly focused on the dependency-based vulnerabilities rather than finding actual issues within the application code. Sonarqube was unable to identify any vulnerabilities at all.