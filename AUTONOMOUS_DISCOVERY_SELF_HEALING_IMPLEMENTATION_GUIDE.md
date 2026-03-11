# Autonomous Discovery Self-Healing System Implementation Guide

## ServiceNow AI-Powered Discovery Automation

This guide provides a complete implementation blueprint for building an Autonomous Discovery Self-Healing system using ServiceNow's latest AI capabilities. The system leverages AI Agent Studio, Now Assist AI Agents, Agentic workflows, and AI Control Tower to automatically detect, analyze, and remediate Discovery failures.

---

## STEP 1 – ARCHITECTURE

### Overview

The Autonomous Discovery Self-Healing system consists of multiple integrated components that work together to create a closed-loop automation system for Discovery troubleshooting.

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                        AUTONOMOUS DISCOVERY SELF-HEALING ARCHITECTURE                   │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                         │
│  ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐              │
│  │  ServiceNow      │     │  AI Agent        │     │  AI Control      │              │
│  │  Discovery       │────▶│  Studio          │────▶│  Tower            │              │
│  └──────────────────┘     └──────────────────┘     └──────────────────┘              │
│           │                        │                        │                        │
│           ▼                        ▼                        ▼                        │
│  ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐              │
│  │  Discovery       │     │  Agentic         │     │  Governance      │              │
│  │  Logs            │     │  Workflows       │     │  & Monitoring    │              │
│  │  (ECC Queue,     │────▶│  Orchestration   │────▶│                  │              │
│  │   Discovery_Log, │     │                  │     │                  │              │
│  │   Discovery_     │     │                  │     │                  │              │
│  │    Status)       │     │                  │     │                  │              │
│  └──────────────────┘     └──────────────────┘     └──────────────────┘              │
│           │                        │                        │                        │
│           ▼                        ▼                        ▼                        │
│  ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐              │
│  │  Log Ingestion  │     │  Multiple AI     │     │  Dashboard &     │              │
│  │  Scheduled Job  │────▶│  Agents          │────▶│  Reporting       │              │
│  │                  │     │                  │     │                  │              │
│  │  - Agent 1       │     │  - Log Analyzer  │     │  - Analytics     │              │
│  │  - Agent 2       │     │  - Root Cause    │     │  - Visualization │              │
│  │  - Agent 3       │     │  - Remediation    │     │                  │              │
│  └──────────────────┘     └──────────────────┘     └──────────────────┘              │
│                                    │                                               │
│                                    ▼                                               │
│                          ┌──────────────────┐                                      │
│                          │  Automation      │                                      │
│                          │  Layer           │                                      │
│                          │                  │                                      │
│                          │  - Flow Designer │                                      │
│                          │  - Script        │                                      │
│                          │    Includes      │                                      │
│                          │  - Actions       │                                      │
│                          └──────────────────┘                                      │
│                                    │                                               │
│           ┌────────────────────────┼────────────────────────┐                       │
│           ▼                        ▼                        ▼                       │
│  ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐            │
│  │  Credential      │     │  Network Team    │     │  Platform Team   │            │
│  │  Admin Task      │     │  Incident        │     │  Notification    │            │
│  └──────────────────┘     └──────────────────┘     └──────────────────┘            │
│                                                                                         │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

### Component Interactions

#### 1. ServiceNow Discovery
- Generates discovery logs in multiple tables (ecc_queue, discovery_log, discovery_status)
- Triggers discovery schedules and ad-hoc discoveries
- Creates discovery sensor data and pattern matches
- Records MID server activity and health

#### 2. Discovery Logs
- **ecc_queue**: Contains ECMD (Extended Agent Messaging) queue entries for MID server communication
- **discovery_log**: Stores detailed discovery process logs with timestamps and severity levels
- **discovery_status**: Maintains overall discovery run status per IP range or device

#### 3. Log Ingestion Process
1. Scheduled job runs every 15 minutes
2. Queries discovery tables for recent failure entries
3. Filters logs containing ERROR, FAILED, TIMEOUT, CONNECTION REFUSED
4. Stores filtered logs in the custom analysis table
5. Triggers the AI agent workflow for processing

#### 4. AI Agents (AI Agent Studio)
- **Agent 1 - Discovery Log Analyzer**: Parses and categorizes discovery failures
- **Agent 2 - Root Cause Analyzer**: Determines underlying causes using AI analysis
- **Agent 3 - Remediation Agent**: Executes automated remediation actions

#### 5. Agentic Workflow Orchestration
- Coordinates sequential agent execution
- Manages data passing between agents
- Handles exception paths and retries
- Maintains workflow state and audit trail

#### 6. Automation Layer
- Flow Designer workflows for automated actions
- Script Includes for complex logic
- REST API integrations for external systems
- Notification and escalation workflows

#### 7. Dashboard and Reporting
- Real-time discovery failure analytics
- AI recommendation accuracy metrics
- Automation success rate tracking
- Trend analysis and forecasting

#### 8. AI Control Tower
- Monitors AI agent performance
- Tracks model usage and costs
- Ensures AI governance compliance
- Provides audit and compliance reporting

---

## STEP 2 – DATA MODEL

### Custom Table: Discovery Log Analysis

Create a new table to store discovery log analysis results. This table serves as the central repository for all AI-analyzed discovery failures.

#### Table Creation Script

```javascript
// Script Include: DiscoveryLogAnalyzerTableCreator
// Run this in System Definition > Tables to create the table

var table = {
    name: 'Discovery Log Analysis',
    label: 'Discovery Log Analysis',
    table_name: 'u_discovery_log_analysis',
    columns: [
        {
            name: 'u_device_ip',
            label: 'Device IP',
            type: 'string',
            max_length: 40,
            mandatory: true
        },
        {
            name: 'u_ci_name',
            label: 'CI Name',
            type: 'string',
            max_length: 200
        },
        {
            name: 'u_log_text',
            label: 'Log Text',
            type: 'text',
            mandatory: true
        },
        {
            name: 'u_error_category',
            label: 'Error Category',
            type: 'choice',
            choice_table: 'u_discovery_error_category',
            default_value: 'unclassified'
        },
        {
            name: 'u_ai_root_cause',
            label: 'AI Root Cause',
            type: 'string',
            max_length: 1000
        },
        {
            name: 'u_ai_recommendation',
            label: 'AI Recommendation',
            type: 'text'
        },
        {
            name: 'u_confidence_score',
            label: 'Confidence Score',
            type: 'integer',
            min: 0,
            max: 100
        },
        {
            name: 'u_status',
            label: 'Status',
            type: 'choice',
            default_value: 'pending'
        },
        {
            name: 'u_created_on',
            label: 'Created On',
            type: 'datetime',
            default: 'NOW'
        },
        {
            name: 'u_source_table',
            label: 'Source Table',
            type: 'string',
            max_length: 50
        },
        {
            name: 'u_source_sys_id',
            label: 'Source Sys ID',
            type: 'string',
            max_length: 32
        },
        {
            name: 'u_discovery_schedule',
            label: 'Discovery Schedule',
            type: 'reference',
            reference_table: 'discovery_schedule'
        },
        {
            name: 'u_mid_server',
            label: 'MID Server',
            type: 'reference',
            reference_table: 'ecc_queue'
        },
        {
            name: 'u_remediation_action',
            label: 'Remediation Action',
            type: 'text'
        },
        {
            name: 'u_remediation_status',
            label: 'Remediation Status',
            type: 'choice',
            default_value: 'not_started'
        },
        {
            name: 'u_execution_count',
            label: 'Execution Count',
            type: 'integer',
            default: 0
        }
    ]
};

// Create the table using TableCreator API
// Execute in System Definition > Tables > Create Table
```

#### Field Usage in AI Workflow

| Field | Description | Usage in AI Workflow |
|-------|-------------|---------------------|
| **u_device_ip** | IP address of the target device | Primary key for device lookup; used to query CMDB for related CIs |
| **u_ci_name** | Name of the CI if discovered | Links to CMDB for relationship analysis |
| **u_log_text** | Raw discovery log content | Input for AI agent analysis; contains error messages |
| **u_error_category** | Categorized failure type | Output from Agent 1; enables filtering and routing |
| **u_ai_root_cause** | AI-determined root cause | Output from Agent 2; drives remediation selection |
| **u_ai_recommendation** | AI-generated remediation steps | Output from Agent 2; used by Agent 3 for execution |
| **u_confidence_score** | AI confidence (0-100) | Determines automation level; high confidence = auto-execute |
| **u_status** | Processing state | Workflow state machine: pending → analyzing → remediated → closed |
| **u_created_on** | Record creation timestamp | Enables time-based analysis and trending |
| **u_source_table** | Origin table name | Tracks source (ecc_queue, discovery_log, discovery_status) |
| **u_source_sys_id** | Source record sys_id | Enables drill-back to original log entry |

#### Choice Values for Error Category

Create a choice table with these values:
- UNCLASSIFIED
- CREDENTIAL_FAILURE
- NETWORK_ISSUE
- MID_SERVER_FAILURE
- FIREWALL_BLOCKED
- PATTERN_FAILURE
- TIMEOUT
- DNS_ISSUE
- AUTHENTICATION_FAILED
- PROTOCOL_ERROR

#### Choice Values for Status

- PENDING - New entry, not yet processed
- ANALYZING - Currently being processed by AI agents
- REMEDIATION_REQUIRED - Analysis complete, action needed
- IN_PROGRESS - Remediation action executing
- REMEDIATED - Issue resolved
- FAILED - Remediation unsuccessful
- CLOSED - Issue resolved and verified

---

## STEP 3 – DISCOVERY LOG INGESTION

### Scheduled Job Configuration

Create a scheduled job to collect Discovery logs every 15 minutes.

#### Step 1: Create the Scheduled Job

Navigate to: **System Definition > Scheduled Jobs**

Click **New** and configure:

```
Name: Discovery Log Ingestion - AI Analysis
Table: (empty - runs system-wide)
Run to: [set to a date 2 years in the future]
When: [select appropriate time]
Frequency: Repeat: Every 15 minutes
Active: true
```

#### Step 2: Create the Processing Script

```javascript
// Script Include: DiscoveryLogIngestion
// Type: Scheduled Job Script

var DiscoveryLogIngestion = Class.create();
DiscoveryLogIngestion.prototype = {
    initialize: function() {
        this.errorKeywords = ['ERROR', 'FAILED', 'TIMEOUT', 'CONNECTION REFUSED'];
        this.timeWindowMinutes = 15;
        this.analysisTable = 'u_discovery_log_analysis';
    },
    
    execute: function() {
        var startTime = new GlideDateTime();
        startTime.addMinutes(-this.timeWindowMinutes);
        
        this.processECCUeueLogs(startTime);
        this.processDiscoveryLogs(startTime);
        this.processDiscoveryStatus(startTime);
        
        gs.info('Discovery Log Ingestion completed at: ' + new GlideDateTime());
    },
    
    processECCUeueLogs: function(startTime) {
        var gr = new GlideRecord('ecc_queue');
        gr.addQuery('sys_created_on', '>=', startTime);
        gr.addQuery('name', 'CONTAINS', 'discovery');
        gr.addActiveQuery();
        gr.query();
        
        while (gr.next()) {
            var logText = gr.payload.getXMLText() || '';
            if (this.containsErrorKeyword(logText)) {
                this.createAnalysisRecord({
                    device_ip: gr.agent,
                    log_text: logText,
                    source_table: 'ecc_queue',
                    source_sys_id: gr.sys_id.toString()
                });
            }
        }
    },
    
    processDiscoveryLogs: function(startTime) {
        var gr = new GlideRecord('discovery_log');
        gr.addQuery('sys_created_on', '>=', startTime);
        gr.addQuery('severity', 'ERROR');
        gr.addActiveQuery();
        gr.query();
        
        while (gr.next()) {
            var logText = gr.message + ' ' + gr.details;
            this.createAnalysisRecord({
                device_ip: gr.ip_address,
                ci_name: gr.ci_name,
                log_text: logText,
                source_table: 'discovery_log',
                source_sys_id: gr.sys_id.toString()
            });
        }
    },
    
    processDiscoveryStatus: function(startTime) {
        var gr = new GlideRecord('discovery_status');
        gr.addQuery('sys_created_on', '>=', startTime);
        gr.addQuery('status', 'FAILURE');
        gr.addActiveQuery();
        gr.query();
        
        while (gr.next()) {
            var logText = 'Discovery Status: ' + gr.status + '. Details: ' + gr.status_message;
            this.createAnalysisRecord({
                device_ip: gr.ip_address,
                ci_name: gr.name,
                log_text: logText,
                source_table: 'discovery_status',
                source_sys_id: gr.sys_id.toString(),
                discovery_schedule: gr.schedule
            });
        }
    },
    
    containsErrorKeyword: function(text) {
        for (var i = 0; i < this.errorKeywords.length; i++) {
            if (text.toUpperCase().indexOf(this.errorKeywords[i]) !== -1) {
                return true;
            }
        }
        return false;
    },
    
    createAnalysisRecord: function(data) {
        // Check if record already exists
        var existing = new GlideRecord(this.analysisTable);
        existing.addQuery('u_source_sys_id', data.source_sys_id);
        existing.query();
        
        if (!existing.next()) {
            var gr = new GlideRecord(this.analysisTable);
            gr.initialize();
            gr.u_device_ip = data.device_ip || '';
            gr.u_ci_name = data.ci_name || '';
            gr.u_log_text = data.log_text || '';
            gr.u_source_table = data.source_table || '';
            gr.u_source_sys_id = data.source_sys_id || '';
            gr.u_status = 'pending';
            gr.u_created_on = new GlideDateTime();
            gr.insert();
            gs.info('Created discovery log analysis record for: ' + data.device_ip);
        }
    },
    
    type: 'DiscoveryLogIngestion'
};

// Execute the ingestion
var ingestor = new DiscoveryLogIngestion();
ingestor.execute();
```

#### Step 3: Configure Job Parameters

In the scheduled job configuration:

```
Script: [paste the script above]
```

#### Filtering Logic

The scheduled job applies these filters:

| Filter | Source Tables | Criteria |
|--------|---------------|----------|
| ERROR Detection | discovery_log | severity = 'ERROR' |
| FAILURE Status | discovery_status | status = 'FAILURE' |
| Queue Failures | ecc_queue | payload contains error keywords |
| Time Window | All | Last 15 minutes |

#### Error Keywords

```
ERROR
FAILED
TIMEOUT
CONNECTION REFUSED
AUTHENTICATION FAILED
ACCESS DENIED
CONNECTION RESET
NO ROUTE TO HOST
UNREACHABLE
```

---

## STEP 4 – CREATE MULTIPLE AI AGENTS

### AI Agent Studio Configuration

Navigate to: **AI Agent > AI Agent Studio**

Create three specialized AI agents for the discovery self-healing workflow.

---

### Agent 1 – Discovery Log Analyzer

#### Agent Objective
Parse and categorize Discovery log entries to identify the type of failure and prepare data for root cause analysis.

#### Agent Configuration

```
Agent Name: Discovery Log Analyzer
Agent Type: Analysis Agent
Model: ServiceNow AI Model (latest)
Version: Current Production
```

#### Agent Instructions

```
You are a ServiceNow Discovery log analysis expert. Your role is to:

1. Parse discovery log entries to extract key information
2. Categorize failures into standard error categories
3. Identify the discovery protocol used (SSH, SNMP, WMI, etc.)
4. Extract device identification information
5. Prepare structured data for root cause analysis

Error Categories:
- CREDENTIAL_FAILURE: Authentication or authorization errors
- NETWORK_ISSUE: Connectivity, routing, or network stack errors
- MID_SERVER_FAILURE: MID server unavailable or overloaded
- FIREWALL_BLOCKED: Port blocked or firewall rules preventing access
- PATTERN_FAILURE: Discovery pattern execution errors
- TIMEOUT: Connection or operation timeouts
- DNS_ISSUE: DNS resolution failures
- AUTHENTICATION_FAILED: Login or credential validation failures
- PROTOCOL_ERROR: Protocol-specific errors

For each log entry, provide:
- error_category: The most likely category
- device_identifier: IP address or hostname
- protocol: Discovery protocol used
- severity: Critical, High, Medium, Low
- additional_context: Any other relevant information
```

#### Agent Tools

| Tool Name | Purpose | Configuration |
|-----------|---------|---------------|
| **Query Discovery Log Analysis Table** | Read pending log entries | Table: u_discovery_log_analysis, Status: pending |
| **Query CMDB** | Get CI details for device | Reference CI by IP or hostname |
| **Update Record** | Mark analysis status | Update error_category field |
| **Query ECC Queue** | Get additional MID server context | Filter by device IP |

#### Input/Output

**Input:**
- Pending records from Discovery Log Analysis table
- Raw log text from discovery_log, ecc_queue, or discovery_status

**Output:**
```
{
    "error_category": "CREDENTIAL_FAILURE",
    "device_identifier": "192.168.1.100",
    "protocol": "SSH",
    "severity": "HIGH",
    "additional_context": "Authentication failed for user admin",
    "analysis_complete": true
}
```

---

### Agent 2 – Root Cause Analyzer

#### Agent Objective
Analyze categorized failures to determine the underlying root cause and generate remediation recommendations.

#### Agent Configuration

```
Agent Name: Root Cause Analyzer
Agent Type: Analysis & Recommendation Agent
Model: ServiceNow AI Model (latest)
Version: Current Production
```

#### Agent Instructions

```
You are a ServiceNow Discovery troubleshooting expert with deep knowledge of:
- Discovery patterns and sensors
- MID server architecture
- Network protocols (SSH, SNMP, WMI, HTTP)
- CMDB data model
- Credential management

For each failure analysis, determine:

1. ROOT CAUSE: The fundamental reason for the discovery failure
   - What specifically failed?
   - Why did it fail?
   - What is the evidence?

2. FAILURE CATEGORY: Classification from Agent 1 (validate or correct)

3. RECOMMENDED REMEDIATION: Specific steps to resolve the issue
   - Who should take action?
   - What specific steps are needed?
   - What information is needed?

4. CONFIDENCE SCORE: Your confidence in this analysis (0-100)
   - Based on evidence quality
   - Based on historical patterns
   - Based on log completeness

Remediation Categories:
- CREDENTIAL_FAILURE → Update credentials in discovery credentials
- NETWORK_ISSUE → Escalate to network team
- MID_SERVER_FAILURE → Check MID server health and capacity
- FIREWALL_BLOCKED → Request firewall rule change
- PATTERN_FAILURE → Contact pattern development team
- TIMEOUT → Increase timeout or optimize discovery
- DNS_ISSUE → Verify DNS configuration
- AUTHENTICATION_FAILED → Re-validate credentials

Provide your response as structured JSON with clear reasoning for each determination.
```

#### Agent Tools

| Tool Name | Purpose | Configuration |
|-----------|---------|---------------|
| **Query Discovery Log Analysis Table** | Read categorized entries | Status: pending |
| **Query CMDB CI** | Get full CI details | By IP or hostname |
| **Query MID Server** | Check MID server status | By name from log |
| **Query Discovery Credentials** | Check credential status | By device pattern |
| **Query Discovery Schedule** | Get discovery configuration | Related to device |
| **Update Record** | Save root cause analysis | Update ai_root_cause, ai_recommendation, confidence_score |

#### Input/Output

**Input:**
```
{
    "error_category": "CREDENTIAL_FAILURE",
    "device_identifier": "192.168.1.100",
    "protocol": "SSH",
    "log_text": "Authentication failed for user admin",
    "ci_name": "Production-Web-01"
}
```

**Output:**
```
{
    "ai_root_cause": "SSH authentication failure due to incorrect credential. The username 'admin' is valid but the password does not match. Possible causes: password changed on device, account locked, or credential never worked.",
    "error_category": "CREDENTIAL_FAILURE",
    "ai_recommendation": "1. Verify the device admin account is active\n2. Confirm the correct password in ServiceNow credentials\n3. Test credentials manually via SSH\n4. Update credential in discovery_credentials table if needed\n5. Re-run discovery after credential update",
    "confidence_score": 92,
    "remediation_action": "credential_update_required",
    "automatable": true
}
```

---

### Agent 3 – Remediation Agent

#### Agent Objective
Execute automated remediation actions based on AI recommendations or route issues to appropriate teams for manual resolution.

#### Agent Configuration

```
Agent Name: Remediation Agent
Agent Type: Action & Automation Agent
Model: ServiceNow AI Model (latest)
Version: Current Production
```

#### Agent Instructions

```
You are an automation execution agent responsible for resolving Discovery failures.

Based on the AI analysis results, you must:

1. EVALUATE AUTOMATION SUITABILITY
   - Is the remediation automatable?
   - What is the confidence score?
   - Are there any risks?

2. EXECUTE AUTOMATED ACTIONS (when confidence >= 80)
   - Update discovery credentials
   - Retry discovery
   - Adjust discovery timeouts
   - Create appropriate records

3. CREATE ESCALATION RECORDS (when confidence < 80 or not automatable)
   - Create incident for team assignment
   - Create task for specific actions
   - Generate notification to appropriate team

4. UPDATE ANALYSIS RECORD
   - Set status based on outcome
   - Record remediation actions taken
   - Track execution results

Automation Actions by Category:
- CREDENTIAL_FAILURE: Create credential update task, trigger credential re-validation
- NETWORK_ISSUE: Create incident for network team, include log details
- MID_SERVER_FAILURE: Create task for platform team, include MID server details
- FIREWALL_BLOCKED: Create incident for security team with port/firewall request
- PATTERN_FAILURE: Create problem record for pattern team
- TIMEOUT: Adjust discovery schedule timeout, retry discovery
- DNS_ISSUE: Create task for infrastructure team

Always log all actions taken and their results.
```

#### Agent Tools

| Tool Name | Purpose | Configuration |
|-----------|---------|---------------|
| **Query Discovery Log Analysis Table** | Read recommendations | Status: remediation_required |
| **Update Record** | Set remediation status | Update status, remediation_action |
| **Create Incident** | Escalate to teams | Route to appropriate group |
| **Create Task** | Create action tasks | For specific remediation steps |
| **Trigger Flow** | Execute Flow Designer | Retry discovery, update credentials |
| **Send Notification** | Alert teams | Email or SMS notifications |
| **Query Discovery Credentials** | Check credential status | For validation |
| **Execute Script** | Run custom logic | Script Include execution |

#### Input/Output

**Input:**
```
{
    "ai_root_cause": "SSH authentication failure",
    "ai_recommendation": "Update credentials",
    "confidence_score": 92,
    "error_category": "CREDENTIAL_FAILURE",
    "device_ip": "192.168.1.100",
    "ci_name": "Production-Web-01"
}
```

**Output:**
```
{
    "remediation_status": "completed",
    "actions_taken": [
        "Created task for credential update",
        "Sent notification to discovery admin",
        "Scheduled credential re-validation"
    ],
    "result": "success",
    "next_steps": "Verify credential update, re-run discovery"
}
```

---

## STEP 5 – AI ANALYSIS PROMPT

### Core Analysis Prompt

Use this prompt template for the AI agent to analyze Discovery logs:

```
You are a ServiceNow Discovery troubleshooting expert with extensive knowledge of:

- ServiceNow Discovery architecture and patterns
- MID server configuration and troubleshooting
- Network protocols (SSH, SNMP, WMI, HTTP, JDBC)
- CMDB data model and CI relationships
- Credential management and security
- Firewall and network configuration

ANALYZE THE FOLLOWING DISCOVERY LOG:

========================================
DEVICE IP: {device_ip}
CI NAME: {ci_name}
LOG TEXT:
{log_text}
========================================

Based on this log entry, determine:

1. ROOT CAUSE (provide a detailed explanation):
   - What specifically failed?
   - Why did it fail?
   - What evidence supports this?

2. FAILURE CATEGORY (select one):
   - CREDENTIAL_FAILURE: Authentication or credential issues
   - NETWORK_ISSUE: Connectivity, routing, or network problems
   - MID_SERVER_FAILURE: MID server unavailable, overloaded, or misconfigured
   - FIREWALL_BLOCKED: Firewall or security appliance blocking access
   - PATTERN_FAILURE: Discovery pattern or sensor execution error
   - TIMEOUT: Connection timeout or operation timeout
   - DNS_ISSUE: DNS resolution failure
   - AUTHENTICATION_FAILED: Login failure or access denied
   - PROTOCOL_ERROR: Protocol-specific error

3. RECOMMENDED REMEDIATION (provide specific steps):
   - Who should take action?
   - What specific steps are needed?
   - What information is needed to resolve?

4. CONFIDENCE SCORE (0-100):
   - How confident are you in this analysis?
   - What factors affect your confidence?

5. AUTOMATABLE (true/false):
   - Can this issue be resolved automatically?
   - What conditions must be met?

Provide your response in the following JSON format:
{
    "root_cause": "...",
    "failure_category": "...",
    "recommendation": "...",
    "confidence_score": 0-100,
    "automatable": true/false,
    "remediation_type": "credential_update|retry|escalation|investigation"
}
```

### Failure Categories Detail

| Category | Description | Common Causes | Typical Resolution |
|----------|-------------|---------------|-------------------|
| **CREDENTIAL_FAILURE** | Authentication errors | Wrong password, expired credential, account locked | Update credential, validate with device |
| **NETWORK_ISSUE** | Connectivity problems | Network path blocked, routing issue, VLAN mismatch | Network team investigation |
| **MID_SERVER_FAILURE** | MID server issues | MID down, overloaded, misconfigured | Restart MID, add capacity |
| **FIREWALL_BLOCKED** | Port/connection blocked | Firewall rule, port blocked, ACL | Request firewall exception |
| **PATTERN_FAILURE** | Discovery pattern error | Pattern bug, sensor error | Pattern development team |
| **TIMEOUT** | Operation timeout | Slow device, network latency, load | Increase timeout, optimize |
| **DNS_ISSUE** | DNS resolution failure | Wrong DNS, DNS server down | Verify DNS configuration |
| **AUTHENTICATION_FAILED** | Login failure | Invalid credentials, account disabled | Credential validation |

---

## STEP 6 – AGENTIC WORKFLOW

### Workflow Design

Navigate to: **AI Agent > Agentic Workflows**

Create a sequential workflow that orchestrates the three AI agents.

#### Workflow Definition

```
Workflow Name: Discovery Self-Healing Workflow
Trigger: New record in Discovery Log Analysis table (status = pending)
Timeout: 30 minutes
Retry: 2 attempts on failure
```

#### Workflow Stages

```
┌────────────────────────────────────────────────────────────────────────────────┐
│                    DISCOVERY SELF-HEALING WORKFLOW                            │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                │
│  ┌──────────────┐                                                              │
│  │   TRIGGER    │ ──── New pending record in Discovery Log Analysis         │
│  └──────┬───────┘                                                              │
│         │                                                                      │
│         ▼                                                                      │
│  ┌──────────────────────────────────────────────────────────────────┐         │
│  │                    STAGE 1: LOG PARSING                           │         │
│  │  ┌────────────────┐                                             │         │
│  │  │   AGENT 1      │ ── Parse logs & categorize errors            │         │
│  │  │ Discovery Log  │ ── Extract device info                       │         │
│  │  │   Analyzer     │ ── Set error_category                        │         │
│  │  └────────────────┘                                             │         │
│  │         │                                                         │         │
│  │         ▼                                                         │         │
│  │  ┌─────────────────────────────────────────────────────────────┐│         │
│  │  │  Decision: Is categorization complete?                       ││         │
│  │  │  YES → Continue to Stage 2                                   ││         │
│  │  │  NO  → Mark as failed, notify admin                         ││         │
│  │  └─────────────────────────────────────────────────────────────┘│         │
│  └──────────────────────────────────────────────────────────────────┘         │
│         │                                                                      │
│         ▼                                                                      │
│  ┌──────────────────────────────────────────────────────────────────┐         │
│  │                 STAGE 2: ROOT CAUSE ANALYSIS                      │         │
│  │  ┌────────────────┐                                             │         │
│  │  │   AGENT 2      │ ── Analyze root cause                       │         │
│  │  │  Root Cause    │ ── Generate recommendations                  │         │
│  │  │   Analyzer     │ ── Set confidence_score                     │         │
│  │  │                │ ── Determine automatable                     │         │
│  │  └────────────────┘                                             │         │
│  │         │                                                         │         │
│  │         ▼                                                         │         │
│  │  ┌─────────────────────────────────────────────────────────────┐│         │
│  │  │  Decision: Confidence >= 80% and automatable?              ││         │
│  │  │  YES → Continue to Stage 3 (Auto)                           ││         │
│  │  │  NO  → Continue to Stage 3 (Manual)                        ││         │
│  │  └─────────────────────────────────────────────────────────────┘│         │
│  └──────────────────────────────────────────────────────────────────┘         │
│         │                                                                      │
│         ▼                                                                      │
│  ┌──────────────────────────────────────────────────────────────────┐         │
│  │              STAGE 3: REMEDIATION EXECUTION                       │         │
│  │  ┌────────────────┐                                             │         │
│  │  │   AGENT 3      │ ── Execute remediation                       │         │
│  │  │  Remediation   │ ── Create tasks/incidents                    │         │
│  │  │    Agent       │ ── Update record status                      │         │
│  │  │                │ ── Notify teams                              │         │
│  │  └────────────────┘                                             │         │
│  │         │                                                         │         │
│  │         ▼                                                         │         │
│  │  ┌─────────────────────────────────────────────────────────────┐│         │
│  │  │  Verification: Did remediation succeed?                     ││         │
│  │  │  YES → Mark as remediated                                    ││         │
│  │  │  NO  → Retry or escalate                                     ││         │
│  │  └─────────────────────────────────────────────────────────────┘│         │
│  └──────────────────────────────────────────────────────────────────┘         │
│         │                                                                      │
│         ▼                                                                      │
│  ┌──────────────┐                                                              │
│  │   COMPLETE   │ ──── Update status, log results                             │
│  └──────────────┘                                                              │
│                                                                                │
└────────────────────────────────────────────────────────────────────────────────┘
```

### Data Passing Between Agents

#### Agent 1 → Agent 2 Data Flow

```
{
    "sys_id": "record_sys_id",
    "device_ip": "192.168.1.100",
    "ci_name": "Production-Web-01",
    "log_text": "SSH authentication failed...",
    "error_category": "CREDENTIAL_FAILURE",
    "protocol": "SSH",
    "severity": "HIGH"
}
```

#### Agent 2 → Agent 3 Data Flow

```
{
    "sys_id": "record_sys_id",
    "device_ip": "192.168.1.100",
    "ci_name": "Production-Web-01",
    "error_category": "CREDENTIAL_FAILURE",
    "ai_root_cause": "SSH authentication failure due to incorrect credential...",
    "ai_recommendation": "1. Verify device admin account\n2. Confirm correct password...",
    "confidence_score": 92,
    "automatable": true,
    "remediation_type": "credential_update"
}
```

---

## STEP 7 – AUTOMATED REMEDIATION

### Automation Layer Configuration

Navigate to: **Flow Designer > Flows**

Create automated remediation flows based on error categories.

---

### Flow 1: Credential Failure Remediation

```
Flow Name: Discovery Credential Failure Remediation
Trigger: AI analysis returns error_category = CREDENTIAL_FAILURE AND confidence >= 80
```

**Flow Steps:**

1. **Get Analysis Record** - Read the Discovery Log Analysis record
2. **Get Device CI** - Query CMDB for the device CI
3. **Get Discovery Credentials** - Query discovery_credentials for matching credential
4. **Decision: Is credential found?**
   - YES → Continue
   - NO → Create incident, end flow
5. **Validate Credential** - Test credential connectivity
6. **Decision: Is credential valid?**
   - YES → Create task for manual review
   - NO → Update credential with correct password
7. **Trigger Re-Discovery** - Run discovery for the device
8. **Update Record** - Mark remediation complete
9. **Notify Admin** - Send notification with results

---

### Flow 2: Network Issue Escalation

```
Flow Name: Discovery Network Issue Escalation
Trigger: AI analysis returns error_category = NETWORK_ISSUE
```

**Flow Steps:**

1. **Get Analysis Record** - Read the Discovery Log Analysis record
2. **Get Device CI** - Query CMDB for the device CI
3. **Create Incident** - Create incident for Network Team
   - Category: Discovery Issue
   - Impact: 2
   - Urgency: 2
   - Description: Include AI analysis and recommendations
   - Assigned Group: Network Operations
4. **Update Record** - Mark status as escalated
5. **Notify Team** - Send notification to Network Team

---

### Flow 3: MID Server Failure Remediation

```
Flow Name: Discovery MID Server Failure Remediation
Trigger: AI analysis returns error_category = MID_SERVER_FAILURE
```

**Flow Steps:**

1. **Get Analysis Record** - Read the Discovery Log Analysis record
2. **Identify MID Server** - Extract MID server name from log
3. **Check MID Server Health** - Query ecc_agent_monitor for MID status
4. **Decision: MID server down?**
   - YES → Create task for Platform Team
   - NO → Continue to investigate
5. **Create Task** - Task for MID server investigation
6. **Update Record** - Mark remediation in progress
7. **Notify Platform Team** - Alert platform team

---

### Flow 4: Pattern Failure Management

```
Flow Name: Discovery Pattern Failure Management
Trigger: AI analysis returns error_category = PATTERN_FAILURE
```

**Flow Steps:**

1. **Get Analysis Record** - Read the Discovery Log Analysis record
2. **Identify Pattern** - Extract pattern name from log
3. **Create Problem Record** - Create problem for Pattern Team
   - Problem Type: Discovery Pattern Issue
   - Include root cause analysis
   - Attach discovery logs
4. **Update Record** - Mark as pattern failure
5. **Notify Pattern Team** - Alert pattern development team

---

### Remediation Action Matrix

| Error Category | Primary Action | Secondary Action | Escalation |
|----------------|----------------|------------------|------------|
| CREDENTIAL_FAILURE | Update credential | Re-run discovery | Task to admin |
| NETWORK_ISSUE | Create incident | Notify network team | Network Ops |
| MID_SERVER_FAILURE | Check MID health | Create platform task | Platform Team |
| FIREWALL_BLOCKED | Create incident | Request firewall change | Security Team |
| PATTERN_FAILURE | Create problem | Notify pattern team | Pattern Dev |
| TIMEOUT | Adjust timeout | Retry discovery | Discovery Admin |
| DNS_ISSUE | Create task | Verify DNS config | Infrastructure |
| AUTHENTICATION_FAILED | Validate credential | Re-test discovery | Admin |

---

## STEP 8 – DASHBOARD

### Dashboard Configuration

Navigate to: **Dashboard > Create Dashboard**

Create comprehensive dashboards for Discovery self-healing analytics.

---

### Dashboard 1: Discovery Failure Overview

```
Dashboard Name: Discovery Self-Healing Overview
Description: Real-time view of Discovery failures and AI remediation
Refresh: Every 5 minutes
```

**Components:**

1. **KPI: Total Discovery Failures (24h)**
   - Type: Number
   - Source: Discovery Log Analysis table
   - Filter: Last 24 hours

2. **KPI: AI Remediation Success Rate**
   - Type: Percentage
   - Formula: (Remediated Count / Total Failures) * 100

3. **KPI: Average Confidence Score**
   - Type: Number
   - Source: u_confidence_score field
   - Display: Percentage

4. **Chart: Discovery Failure by Category**
   - Type: Donut Chart
   - Source: u_error_category field
   - Filter: Last 7 days

5. **Chart: Discovery Failures Trend**
   - Type: Line Chart
   - X-Axis: Date
   - Y-Axis: Failure Count
   - Series: By error category

6. **Table: Recent Failures**
   - Columns: Device IP, CI Name, Error Category, Status, Created
   - Sort: Created descending
   - Limit: 20 rows

---

### Dashboard 2: AI Analysis Performance

```
Dashboard Name: AI Analysis Performance
Description: Monitor AI agent accuracy and performance
Refresh: Every 15 minutes
```

**Components:**

1. **Chart: Root Cause Distribution**
   - Type: Horizontal Bar Chart
   - Source: u_ai_root_cause field
   - Group by: First 100 characters

2. **Chart: Confidence Score Distribution**
   - Type: Histogram
   - Source: u_confidence_score field
   - Ranges: 0-20, 21-40, 41-60, 61-80, 81-100

3. **Chart: Recommendations by Category**
   - Type: Stacked Bar
   - X-Axis: Error Category
   - Y-Axis: Count
   - Stack: Automatable vs Manual

4. **Table: High-Value Analysis (High Confidence)**
   - Columns: Device IP, Confidence, Root Cause, Created
   - Filter: Confidence >= 90

---

### Dashboard 3: Automation Success

```
Dashboard Name: Automation Success Metrics
Description: Track automated remediation effectiveness
Refresh: Real-time
```

**Components:**

1. **KPI: Automated Remediation Count**
   - Type: Number
   - Source: u_remediation_status = 'completed'

2. **KPI: Manual Escalation Count**
   - Type: Number
   - Source: Incidents created

3. **Chart: Automation Rate Trend**
   - Type: Area Chart
   - Series: Automated vs Manual

4. **Chart: Time to Remediation**
   - Type: Line Chart
   - Metric: Average hours from failure to remediated

5. **Table: Failed Remediation Attempts**
   - Columns: Device IP, Error Category, Attempts, Last Error

---

### Dashboard 4: Most Affected CI Classes

```
Dashboard Name: Discovery Impact by CI Class
Description: Identify CI classes with most discovery issues
Refresh: Daily
```

**Components:**

1. **Chart: Top 10 Affected CI Classes**
   - Type: Horizontal Bar
   - Source: u_ci_name (parse class from name)
   - Limit: Top 10

2. **Chart: Discovery Failures by Device Type**
   - Type: Pie Chart
   - Source: CMDB CI class

3. **Table: Most Affected Devices**
   - Columns: CI Name, IP, Failure Count, Last Failure
   - Sort: Failure Count descending
   - Limit: 50

---

## STEP 9 – AI GOVERNANCE

### AI Control Tower Configuration

Navigate to: **AI Control Tower** (available in Montreal release and later)

---

### Governance Framework

#### 1. AI Usage Monitoring

Configure monitoring for all AI agent interactions:

| Metric | Description | Threshold |
|--------|-------------|-----------|
| **Total Requests** | Number of AI agent calls | Track daily |
| **Tokens Consumed** | Total tokens used by agents | Warn at 80% budget |
| **Response Time** | Average AI response time | Warn if > 30 seconds |
| **Error Rate** | AI call failure percentage | Alert if > 5% |

**Configuration:**

```
AI Control Tower > Settings > Usage Monitoring
- Enable request logging: ON
- Log all prompts: ON
- Log all responses: ON
- Retention: 90 days
```

---

#### 2. Agent Performance Tracking

Monitor each AI agent's effectiveness:

**Agent 1 - Discovery Log Analyzer**
- Categorization accuracy: Compare AI category to manual review
- Processing time: Time from input to output
- Coverage: Percentage of logs analyzed

**Agent 2 - Root Cause Analyzer**
- Root cause accuracy: Validate against resolution
- Recommendation relevance: Track if recommendations are implemented
- Confidence calibration: Compare predicted vs actual outcomes

**Agent 3 - Remediation Agent**
- Automation success rate: Percentage of automated fixes that work
- Escalation appropriateness: Were manual escalations necessary?
- Execution time: Time from recommendation to action

---

#### 3. Governance Metrics Dashboard

Create AI Control Tower dashboard with:

```
Metrics to Track:

1. AI Decision Accuracy
   - % of AI diagnoses confirmed correct
   - % of recommendations implemented
   - False positive/negative rates

2. AI Safety Metrics
   - Harmful content generation: 0
   - Credential exposure incidents: 0
   - Unauthorized action attempts: 0

3. AI Performance
   - Average response time: < 15 seconds
   - Success rate: > 95%
   - Timeout rate: < 2%

4. AI Compliance
   - Data privacy compliance: 100%
   - Audit trail completeness: 100%
   - Model version tracking: Complete
```

---

#### 4. Alert Configuration

Configure alerts in AI Control Tower:

| Alert Type | Condition | Action |
|------------|-----------|--------|
| High Error Rate | Agent error > 5% | Email to AI ops team |
| Low Accuracy | Accuracy < 80% | Create governance ticket |
| Token Limit | Usage > 80% budget | Alert finance team |
| Anomaly Detection | Unusual pattern detected | Investigate immediately |

---

#### 5. Audit and Compliance

Enable comprehensive audit logging:

```
AI Control Tower > Audit Settings

- Log all AI agent invocations: YES
- Log all data sent to AI: YES
- Log all AI responses: YES
- Log all automated actions: YES
- Log all escalations: YES
- Retain audit logs: 2 years
- Enable audit export: YES
```

---

## STEP 10 – TESTING

### Testing Strategy

Create test cases to validate the Autonomous Discovery Self-Healing system.

---

### Test Case 1: SSH Connection Timeout

**Objective:** Verify system handles SSH timeout correctly

**Setup:**
1. Create a test device that doesn't respond to SSH
2. Configure discovery to target this device
3. Run discovery

**Expected Behavior:**
1. Discovery fails with SSH timeout
2. Log ingestion captures timeout error
3. Agent 1 categorizes as TIMEOUT
4. Agent 2 identifies timeout as root cause
5. Agent 3 creates escalation task
6. Dashboard shows TIMEOUT category

**Validation Script:**
```javascript
// Test validation script
var testRecord = new GlideRecord('u_discovery_log_analysis');
testRecord.initialize();
testRecord.u_device_ip = '10.0.0.99';
testRecord.u_ci_name = 'Test-Device-Timeout';
testRecord.u_log_text = 'SSH connection timeout after 30 seconds - No route to host';
testRecord.u_source_table = 'discovery_log';
testRecord.u_status = 'pending';
testRecord.insert();

// Run workflow manually to test
```

---

### Test Case 2: Credential Authentication Failed

**Objective:** Verify system handles credential failure correctly

**Setup:**
1. Create discovery credential with incorrect password
2. Configure discovery to target test device
3. Run discovery

**Expected Behavior:**
1. Discovery fails with authentication error
2. Log ingestion captures error
3. Agent 1 categorizes as CREDENTIAL_FAILURE
4. Agent 2 identifies incorrect credential as root cause
5. Agent 3 creates credential update task
6. System triggers re-discovery after credential fix

**Validation Script:**
```javascript
var testRecord = new GlideRecord('u_discovery_log_analysis');
testRecord.initialize();
testRecord.u_device_ip = '10.0.0.100';
testRecord.u_ci_name = 'Test-Device-Auth';
testRecord.u_log_text = 'SSH authentication failed for user admin - Access denied';
testRecord.u_source_table = 'discovery_log';
testRecord.u_status = 'pending';
testRecord.insert();
```

---

### Test Case 3: MID Server Unreachable

**Objective:** Verify system handles MID server failure

**Setup:**
1. Stop a MID server
2. Run discovery that would use this MID server

**Expected Behavior:**
1. Discovery fails with MID server error
2. Log ingestion captures error
3. Agent 1 categorizes as MID_SERVER_FAILURE
4. Agent 2 identifies MID server issue
5. Agent 3 creates platform team notification

**Validation Script:**
```javascript
var testRecord = new GlideRecord('u_discovery_log_analysis');
testRecord.initialize();
testRecord.u_device_ip = '10.0.0.1';
testRecord.u_ci_name = 'Test-MID-Server';
testRecord.u_log_text = 'MID Server MID-SERVER-01 is unreachable - Connection refused';
testRecord.u_source_table = 'ecc_queue';
testRecord.u_status = 'pending';
testRecord.insert();
```

---

### Test Case 4: SNMP Timeout

**Objective:** Verify system handles SNMP timeout correctly

**Setup:**
1. Configure discovery with SNMP
2. Target device with SNMP disabled

**Expected Behavior:**
1. Discovery fails with SNMP timeout
2. System categorizes as TIMEOUT (SNMP-specific)
3. Agent 3 escalates to network team

---

### Test Case 5: Firewall Blocked Port

**Objective:** Verify system handles firewall blocking

**Setup:**
1. Configure firewall rule blocking SSH port
2. Run discovery

**Expected Behavior:**
1. Discovery fails with connection refused
2. Agent 1 categorizes as FIREWALL_BLOCKED
3. Agent 2 identifies port as blocked
4. Agent 3 creates security incident for firewall rule

---

### Test Execution Matrix

| Test Case | Category | Expected Category | Confidence | Automatable |
|-----------|----------|-------------------|------------|-------------|
| SSH Timeout | TIMEOUT | TIMEOUT | > 80% | Partial |
| Credential Failed | CREDENTIAL | CREDENTIAL_FAILURE | > 85% | Yes |
| MID Unreachable | MID | MID_SERVER_FAILURE | > 90% | No |
| SNMP Timeout | TIMEOUT | TIMEOUT | > 80% | No |
| Firewall Blocked | NETWORK | FIREWALL_BLOCKED | > 75% | No |

---

## STEP 11 – FUTURE ENHANCEMENTS

### Evolution Roadmap

The Autonomous Discovery Self-Healing system can evolve into a comprehensive AI-driven discovery operations platform.

---

### 1. Self-Healing Discovery

**Description:** Move from reactive remediation to proactive self-healing

**Enhancements:**

```
Phase 1 - Intelligent Retry:
├── Analyze failure patterns
├── Implement smart retry with backoff
├── Learn optimal retry times
└── Reduce repeated failures by 40%

Phase 2 - Predictive Healing:
├── ML model for failure prediction
├── Pre-emptive credential rotation
├── Proactive network path optimization
└── Reduce discovery failures by 60%

Phase 3 - Autonomous Resolution:
├── Auto-remediate credential issues
├── Auto-adjust discovery parameters
├── Self-tune MID server allocation
└── Achieve 80% automated resolution
```

---

### 2. Predictive Discovery Failure Detection

**Description:** Use machine learning to predict and prevent failures

**Implementation:**

```
┌─────────────────────────────────────────────────────────────┐
│            PREDICTIVE FAILURE DETECTION ARCHITECTURE        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │
│  │ Historical  │───▶│   ML Model  │───▶│  Prediction │    │
│  │    Data      │    │  Training   │    │   Engine    │    │
│  └─────────────┘    └─────────────┘    └─────────────┘    │
│         │                                      │              │
│         ▼                                      ▼              │
│  ┌─────────────┐                       ┌─────────────┐     │
│  │  Features:  │                       │   Action:   │     │
│  │  - Time     │                       │  - Alert     │     │
│  │  - Device   │                       │  - Preempt   │     │
│  │  - Network  │                       │  - Optimize  │     │
│  │  - Credential│                      │              │     │
│  └─────────────┘                       └─────────────┘     │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Predictive Indicators:**
- Time-based patterns (certain times have higher failure rates)
- Device characteristics (older devices fail more)
- Network topology (certain subnets are problematic)
- Credential age (older credentials fail more)
- MID server load (overloaded MID servers fail more)

---

### 3. Automatic Credential Remediation

**Description:** Implement fully automated credential management

**Capabilities:**

```
1. Credential Validation:
   - Automated credential testing
   - Password expiration monitoring
   - Account status verification

2. Credential Update:
   - Integration with password vaults
   - Automated credential rotation
   - Multi-factor authentication handling

3. Credential Recovery:
   - Automatic credential reset
   - Integration with identity management
   - Emergency credential provisioning
```

---

### 4. Multi-Agent Troubleshooting Platform

**Description:** Expand beyond Discovery to other IT operations

**Expanded Architecture:**

```
┌────────────────────────────────────────────────────────────────────┐
│                    MULTI-AGENT PLATFORM ARCHITECTURE               │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                  ORCHESTRATION LAYER                        │  │
│   │                     (AI Controller)                         │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                │                                    │
│         ┌──────────────────────┼──────────────────────┐           │
│         │                      │                      │           │
│         ▼                      ▼                      ▼           │
│   ┌───────────┐         ┌───────────┐         ┌───────────┐      │
│   │ Discovery │         │  Event    │         │  Service  │      │
│   │   Agent   │         │   Agent   │         │    Agent  │      │
│   │  Cluster  │         │  Cluster  │         │  Cluster  │      │
│   └───────────┘         └───────────┘         └───────────┘      │
│         │                      │                      │           │
│         ▼                      ▼                      ▼           │
│   ┌───────────┐         ┌───────────┐         ┌───────────┐      │
│   │ Discovery │         │   Event   │         │  Request  │      │
│   │  Issues   │         │  Resolver  │         │  Fulfillment│    │
│   └───────────┘         └───────────┘         └───────────┘      │
│                                                                     │
└────────────────────────────────────────────────────────────────────┘
```

**Additional Agents:**
- **Event Management Agent**: Analyze and resolve events
- **Incident Agent**: Diagnose and resolve incidents
- **Change Agent**: Assess and approve changes
- **Service Request Agent**: Fulfill service requests
- **Problem Agent**: Investigate root causes

---

### 5. AI-Driven Discovery Optimization

**Description:** Use AI to optimize discovery performance and coverage

**Optimizations:**

```
1. Discovery Scheduling Optimization:
   - AI determines optimal discovery times
   - Prioritize critical CIs
   - Balance MID server load
   - Reduce discovery window by 30%

2. Pattern Optimization:
   - Identify rarely matching patterns
   - Suggest pattern improvements
   - Optimize pattern execution order

3. Coverage Analysis:
   - Identify gaps in discovery coverage
   - Recommend new discovery probes
   - Optimize CI class coverage

4. Cost Optimization:
   - Reduce network traffic
   - Minimize MID server utilization
   - Optimize credential usage
```

---

## APPENDIX A: IMPLEMENTATION CHECKLIST

### Pre-Implementation
- [ ] Review ServiceNow AI Agent documentation
- [ ] Verify AI Control Tower availability
- [ ] Confirm proper licensing (AI features require specific licenses)
- [ ] Identify proof-of-concept environment

### Implementation Phase 1: Foundation
- [ ] Create Discovery Log Analysis table
- [ ] Configure scheduled job for log ingestion
- [ ] Test log collection from all sources
- [ ] Validate error keyword filtering

### Implementation Phase 2: AI Agents
- [ ] Create Agent 1 - Discovery Log Analyzer
- [ ] Create Agent 2 - Root Cause Analyzer
- [ ] Create Agent 3 - Remediation Agent
- [ ] Configure agent tools and permissions

### Implementation Phase 3: Workflow
- [ ] Design agentic workflow
- [ ] Configure data passing between agents
- [ ] Implement decision points
- [ ] Add error handling and retry logic

### Implementation Phase 4: Automation
- [ ] Create credential remediation flow
- [ ] Create network issue escalation flow
- [ ] Create MID server remediation flow
- [ ] Create pattern failure flow

### Implementation Phase 5: Governance
- [ ] Configure AI Control Tower monitoring
- [ ] Set up governance alerts
- [ ] Create audit logging
- [ ] Configure usage tracking

### Implementation Phase 6: Testing
- [ ] Test with sample SSH timeout
- [ ] Test with credential failure
- [ ] Test with MID server failure
- [ ] Test with SNMP timeout
- [ ] Test with firewall blocking

### Implementation Phase 7: Production
- [ ] Deploy to production environment
- [ ] Train operations team
- [ ] Create runbook
- [ ] Set up support process
- [ ] Monitor and optimize

---

## APPENDIX B: SECURITY CONSIDERATIONS

### AI Agent Security

1. **Data Privacy:**
   - Never send sensitive data to external AI models
   - Use ServiceNow-hosted AI capabilities
   - Mask sensitive information in logs

2. **Access Control:**
   - Implement least privilege for AI agents
   - Restrict automated actions to necessary scope
   - Audit all AI-initiated actions

3. **Credential Protection:**
   - Never expose credentials in AI prompts
   - Use secure credential storage
   - Implement credential masking in logs

### Governance Requirements

1. **Compliance:**
   - Document AI decision-making process
   - Maintain audit trail
   - Enable compliance reporting

2. **Monitoring:**
   - Track AI accuracy metrics
   - Monitor for drift
   - Set up alerts for anomalies

---

## APPENDIX C: PERFORMANCE CONSIDERATIONS

### Optimization Tips

1. **Log Ingestion:**
   - Use efficient queries with proper indexes
   - Process in batches of 100 records
   - Schedule during off-peak hours

2. **AI Agent Performance:**
   - Optimize prompt length
   - Cache frequent queries
   - Use parallel processing where possible

3. **Workflow Efficiency:**
   - Minimize agent handoffs
   - Use async actions where possible
   - Implement proper timeout handling

---

## CONCLUSION

This implementation guide provides a comprehensive blueprint for building an Autonomous Discovery Self-Healing system in ServiceNow. By following these steps, organizations can:

1. **Reduce Discovery Failures** - Automatically detect and categorize failures
2. **Accelerate Resolution** - AI-powered root cause analysis
3. **Improve Efficiency** - Automated remediation where possible
4. **Enable Proactive Operations** - Predict and prevent failures
5. **Ensure Governance** - Full visibility into AI operations

The system evolves from reactive troubleshooting to predictive self-healing, ultimately achieving autonomous Discovery operations with minimal human intervention.

---

*Document Version: 1.0*  
*Last Updated: 2026-03-11*  
*Target Platform: ServiceNow Zurich Release and Later*
