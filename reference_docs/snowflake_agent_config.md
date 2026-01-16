# Snowflake Cortex Agent Configuration: HED Student Success Agent

## Overview

This agent provides intelligent student retention monitoring and alert routing for higher education institutions. It combines Snowflake Cortex Analyst (for natural language queries) with custom functions (for action-taking) to create a comprehensive early warning system.

**Architecture:**
```
PostgreSQL (Source) 
  ↓ Fivetran
Snowflake (dbt transformations)
  ↓ Cortex Agent
Natural Language Interface
  ↓ Custom Functions
Alert Routing Logic
  ↓ Census Reverse ETL
Slack Notifications
```

## Prerequisites

Before configuring this agent, ensure you have:

### Required Data
- [x] PostgreSQL source database with student records
- [x] Fivetran connector syncing to Snowflake database `YOUR_DATABASE`
- [x] dbt project creating the HED analytics views in schema `INDUSTRIES_HIGHER_EDUCATION`
- [x] View `VW_HED_RETENTION_RISK_ANALYSIS` exists with at-risk student data

### Required Access
- [x] Snowflake role with `CREATE FUNCTION` privileges
- [x] Snowflake role with `CREATE TABLE` and `CREATE VIEW` privileges
- [x] Ability to create Cortex Agents in Snowflake (requires appropriate role/permissions)
- [x] Census account with Snowflake and Slack connected (for alert delivery)

### Required Tools
- [x] Snowflake Snowsight access (for SQL execution)
- [x] Census workspace (for reverse ETL configuration)
- [x] Slack workspace with webhook/bot access (for alert delivery)

### Database Name Replacement

⚠️ **IMPORTANT:** This configuration uses `YOUR_DATABASE` as the database name placeholder. You must replace this with your actual database name throughout this document.

**Where to replace:**
- All SQL CREATE statements
- Semantic view DDL
- Custom function DDL
- Tool configuration references
- Census view definitions

---

## Basic Configuration

### Agent Name
```
HED_STUDENT_SUCCESS_AGENT_LAB
```

### Display Name
```
HED Student Success Agent_Lab
```

### Snowflake Intelligence
**Available** ✓

### Description
```
Student retention analytics agent that monitors at-risk students, answers natural language questions about retention risk factors, and routes intelligent alerts to academic staff (Dean, Department Chair, or Academic Advisor). Powered by real-time data from the HED pipeline (Fivetran → Snowflake → dbt → Census).
```

---

## Example Questions

Copy these example questions to add to your agent configuration:

```
Which students have critical retention risk right now?
```

```
Show me the top 10 highest risk students
```

```
Find Engineering students with low GPA
```

```
What's the average engagement score by major?
```

```
Which students are inactive for more than 21 days?
```

```
Summarize STU_202400465's risk factors
```

```
How many students are on academic probation today?
```

```
Route an alert for STU_202400465
```

```
What's the total financial aid at risk for critical students?
```

```
Generate a daily retention summary
```

---

## Orchestration Instructions

You are a Student Retention agent that helps academic staff identify and respond to at-risk students.

### TOOL SELECTION:
1. For questions about student data, risk scores, academic performance, or statistics → use query_student_risk_data (Cortex Analyst)
2. For routing an alert for a specific student → use route_student_alert (requires student_id)
3. For generating a daily summary or Slack report → use get_daily_retention_summary

### REASONING PROCESS:
1. Understand if the user wants information (query) or action (alert/summary)
2. For queries: Use query_student_risk_data to search student retention data
3. For alerts: Get the student_id first (ask if not provided), then use route_student_alert
4. For summaries: Use get_daily_retention_summary for Census-ready content

### IMPORTANT:
- Overall risk assessment values: Critical, High, Moderate, Low
- GPA below 2.0 is critical threshold
- 21+ days inactive requires immediate intervention
- Critical risk → routed to Dean (10-min escalation)
- High risk → routed to Department Chair (15-min escalation)
- Medium risk → routed to Academic Advisor (30-min escalation)
- Alerts are delivered via Census → Slack integration
- Always include student_id, major, advisor, and risk assessment when discussing students

---

---

## Response Instructions

### Tone
Professional and supportive, but urgent when discussing critical-risk students. Remember these are real students who need help.

### Format
- Lead with most important information (risk level, student count, priority)
- Include specific numbers (GPA, engagement scores, counts)
- For student details: always show Student ID, Major, Risk Assessment, GPA, Advisor

### Alert Responses
When routing an alert, confirm:
- Student ID and major
- Routing destination (Dean/Department Chair/Academic Advisor)
- Key risk factors triggering the alert
- That alert will be delivered via Slack

### Never
- Skip confirming student_id before routing alerts
- Provide academic or counseling advice beyond the data
- Ignore critical risk indicators

---

## Tools Configuration

### Cortex Analyst Tool

**Tool Name:** `query_student_risk_data`

**Semantic Model:**
```
Database: YOUR_DATABASE
Schema: INDUSTRIES_HIGHER_EDUCATION
Semantic View: HED_AT_RISK_STUDENTS
```

**Description:**
```
Query student retention data, risk scores, academic performance, and statistics using natural language. Use this tool to answer questions about at-risk students, filter by major or risk level, calculate aggregate metrics, or analyze retention trends.
```

---

### Custom Tool 1: Route Student Alert

**Resource Type:** `function`

**Function Reference:**
```
YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.ROUTE_STUDENT_ALERT(VARCHAR)
```

**Tool Name:** `route_student_alert`

**Description:**
```
Use this tool to route a retention alert for a specific student. Requires student_id parameter (e.g., STU_202400001). Returns a comprehensive alert payload with student details, risk assessment, academic performance, recommended actions, routing destination (NOTIFY_DEAN, NOTIFY_CHAIR, or NOTIFY_ADVISOR), and a pre-formatted Slack message. This is the primary tool for taking action on at-risk students. Use when the user asks to "route an alert", "send alert", "notify", or "create alert" for a specific student.
```

**SQL Definition:**
```sql
CREATE OR REPLACE FUNCTION YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.ROUTE_STUDENT_ALERT("STUDENT_ID_INPUT" VARCHAR)
RETURNS OBJECT
LANGUAGE SQL
COMMENT='Prepares a student retention alert payload for routing to academic staff. Returns student risk data with routing recommendation based on severity. Used by Cortex Agent for alert activation via Census to Slack.'
AS '
SELECT OBJECT_CONSTRUCT(
    -- Student identification
    ''student_id'', s.STUDENT_ID,
    ''major_code'', s.MAJOR_CODE,
    ''advisor_id'', s.ADVISOR_ID,
    ''academic_standing'', s.ACADEMIC_STANDING,
    
    -- Risk assessment
    ''overall_risk_assessment'', s.OVERALL_RISK_ASSESSMENT,
    ''at_risk_flag'', s.AT_RISK_FLAG,
    
    -- Academic metrics
    ''current_gpa'', s.CURRENT_GPA,
    ''gpa_risk_level'', s.GPA_RISK_LEVEL,
    ''course_completion_rate'', s.COURSE_COMPLETION_RATE,
    ''completion_risk_level'', s.COMPLETION_RISK_LEVEL,
    
    -- Engagement metrics
    ''engagement_score'', s.ENGAGEMENT_SCORE,
    ''engagement_risk_level'', s.ENGAGEMENT_RISK_LEVEL,
    ''days_since_last_login'', s.DAYS_SINCE_LAST_LOGIN,
    ''login_recency_risk_level'', s.LOGIN_RECENCY_RISK_LEVEL,
    
    -- Activity metrics
    ''total_course_views'', s.TOTAL_COURSE_VIEWS,
    ''assignment_submissions'', s.ASSIGNMENT_SUBMISSIONS,
    ''discussion_posts'', s.DISCUSSION_POSTS,
    
    -- Financial & intervention
    ''financial_aid_amount'', s.FINANCIAL_AID_AMOUNT,
    ''financial_aid_category'', s.FINANCIAL_AID_CATEGORY,
    ''intervention_count'', s.INTERVENTION_COUNT,
    ''intervention_category'', s.INTERVENTION_CATEGORY,
    ''plagiarism_incidents'', s.PLAGIARISM_INCIDENTS,
    ''writing_quality_score'', s.WRITING_QUALITY_SCORE,
    
    -- Pre-generated recommendations
    ''recommended_action'', s.RECOMMENDED_ACTION,
    
    -- =====================================================
    -- ROUTING DECISION LOGIC
    -- =====================================================
    ''routing_recommendation'', 
        CASE 
            -- Critical: Notify Dean immediately
            WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%Critical%'' THEN ''NOTIFY_DEAN''
            WHEN s.CURRENT_GPA < 2.0 THEN ''NOTIFY_DEAN''
            WHEN (s.GPA_RISK_LEVEL = ''Critical'' AND s.COMPLETION_RISK_LEVEL = ''Critical'') THEN ''NOTIFY_DEAN''
            WHEN (s.GPA_RISK_LEVEL = ''Critical'' AND s.ENGAGEMENT_RISK_LEVEL = ''Critical'') THEN ''NOTIFY_DEAN''
            -- High: Notify Department Chair
            WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%High%'' THEN ''NOTIFY_DEPT_CHAIR''
            WHEN s.GPA_RISK_LEVEL = ''Critical'' OR s.COMPLETION_RISK_LEVEL = ''Critical'' THEN ''NOTIFY_DEPT_CHAIR''
            WHEN s.ENGAGEMENT_RISK_LEVEL = ''Critical'' OR s.LOGIN_RECENCY_RISK_LEVEL = ''Critical'' THEN ''NOTIFY_DEPT_CHAIR''
            WHEN s.DAYS_SINCE_LAST_LOGIN > 21 THEN ''NOTIFY_DEPT_CHAIR''
            WHEN s.ACADEMIC_STANDING IN (''Probationary Status'', ''Academic Probation'', ''Academic Warning'', ''Warning Status'') THEN ''NOTIFY_DEPT_CHAIR''
            -- Default: Notify assigned advisor
            ELSE ''NOTIFY_ADVISOR''
        END,
    
    ''routing_explanation'',
        CASE 
            WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%Critical%'' THEN 
                ''Critical overall risk assessment requires Dean notification''
            WHEN s.CURRENT_GPA < 2.0 THEN 
                ''GPA below 2.0 threshold requires Dean notification''
            WHEN (s.GPA_RISK_LEVEL = ''Critical'' AND s.COMPLETION_RISK_LEVEL = ''Critical'') THEN 
                ''Multiple critical indicators (GPA + Completion) require Dean notification''
            WHEN (s.GPA_RISK_LEVEL = ''Critical'' AND s.ENGAGEMENT_RISK_LEVEL = ''Critical'') THEN 
                ''Multiple critical indicators (GPA + Engagement) require Dean notification''
            WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%High%'' THEN 
                ''High risk assessment requires Department Chair notification''
            WHEN s.GPA_RISK_LEVEL = ''Critical'' OR s.COMPLETION_RISK_LEVEL = ''Critical'' THEN 
                ''Critical academic indicator requires Department Chair notification''
            WHEN s.ENGAGEMENT_RISK_LEVEL = ''Critical'' OR s.LOGIN_RECENCY_RISK_LEVEL = ''Critical'' THEN 
                ''Critical engagement indicator requires Department Chair notification''
            WHEN s.DAYS_SINCE_LAST_LOGIN > 21 THEN 
                ''Extended inactivity (21+ days) requires Department Chair notification''
            WHEN s.ACADEMIC_STANDING IN (''Probationary Status'', ''Academic Probation'', ''Academic Warning'', ''Warning Status'') THEN 
                ''Academic standing concern requires Department Chair notification''
            ELSE 
                ''Standard risk level - routing to assigned Academic Advisor''
        END,
    
    -- Escalation timing (minutes until auto-escalate if no acknowledgment)
    ''escalation_minutes'',
        CASE 
            WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%Critical%'' OR s.CURRENT_GPA < 2.0 THEN 10
            WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%High%'' OR s.GPA_RISK_LEVEL = ''Critical'' THEN 15
            ELSE 30
        END,
    
    -- Priority level for display
    ''priority_level'',
        CASE 
            WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%Critical%'' OR s.CURRENT_GPA < 2.0 THEN ''Critical''
            WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%High%'' OR s.GPA_RISK_LEVEL = ''Critical'' THEN ''High''
            ELSE ''Medium''
        END,
    
    -- =====================================================
    -- ALERT METADATA
    -- =====================================================
    ''alert_timestamp'', CURRENT_TIMESTAMP(),
    ''alert_date'', TO_CHAR(CURRENT_DATE(), ''YYYY-MM-DD''),
    ''status'', ''ALERT_READY_FOR_CENSUS_SYNC'',
    
    -- =====================================================
    -- PRE-FORMATTED SLACK MESSAGE (for Census activation)
    -- =====================================================
    ''slack_message'', 
        CASE 
            WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%Critical%'' OR s.CURRENT_GPA < 2.0 THEN ''🔴 ''
            WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%High%'' OR s.GPA_RISK_LEVEL = ''Critical'' THEN ''🟠 ''
            WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%Moderate%'' THEN ''🟡 ''
            ELSE ''🟢 ''
        END ||
        ''*Student Retention Alert: '' || s.STUDENT_ID || ''*\\n\\n'' ||
        ''*Priority:* '' || 
            CASE 
                WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%Critical%'' OR s.CURRENT_GPA < 2.0 THEN ''Critical''
                WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%High%'' OR s.GPA_RISK_LEVEL = ''Critical'' THEN ''High''
                ELSE ''Medium''
            END || ''\\n'' ||
        ''*Major:* '' || s.MAJOR_CODE || ''\\n'' ||
        ''*Advisor:* '' || s.ADVISOR_ID || ''\\n'' ||
        ''*Risk Assessment:* '' || s.OVERALL_RISK_ASSESSMENT || ''\\n\\n'' ||
        
        ''*📊 Academic Performance*\\n'' ||
        ''• GPA: '' || ROUND(s.CURRENT_GPA, 2) || '' ('' || s.GPA_RISK_LEVEL || '')\\n'' ||
        ''• Completion Rate: '' || ROUND(s.COURSE_COMPLETION_RATE * 100, 0) || ''% ('' || s.COMPLETION_RISK_LEVEL || '')\\n'' ||
        ''• Academic Standing: '' || s.ACADEMIC_STANDING || ''\\n\\n'' ||
        
        ''*📱 Engagement Metrics*\\n'' ||
        ''• Engagement Score: '' || ROUND(s.ENGAGEMENT_SCORE, 0) || ''/100 ('' || s.ENGAGEMENT_RISK_LEVEL || '')\\n'' ||
        ''• Last Login: '' || s.DAYS_SINCE_LAST_LOGIN || '' days ago'' ||
            CASE WHEN s.DAYS_SINCE_LAST_LOGIN > 21 THEN '' ⚠️ CRITICAL'' ELSE '''' END || ''\\n'' ||
        ''• Course Views: '' || s.TOTAL_COURSE_VIEWS || ''\\n'' ||
        ''• Assignments Submitted: '' || s.ASSIGNMENT_SUBMISSIONS || ''\\n'' ||
        ''• Discussion Posts: '' || s.DISCUSSION_POSTS || ''\\n\\n'' ||
        
        ''*💰 Financial Aid*\\n'' ||
        ''• Amount: $'' || TRIM(TO_CHAR(s.FINANCIAL_AID_AMOUNT, ''999,999.99'')) || ''\\n'' ||
        ''• Category: '' || s.FINANCIAL_AID_CATEGORY || ''\\n\\n'' ||
        
        ''*🎯 Interventions*\\n'' ||
        ''• Prior Interventions: '' || s.INTERVENTION_COUNT || '' ('' || s.INTERVENTION_CATEGORY || '')\\n\\n'' ||
        
        ''*📋 Recommended Action*\\n'' ||
        s.RECOMMENDED_ACTION || ''\\n\\n'' ||
        
        ''*👤 Routing*\\n'' ||
        CASE 
            WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%Critical%'' OR s.CURRENT_GPA < 2.0 THEN 
                ''📞 Notify Dean of College immediately''
            WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%High%'' OR s.GPA_RISK_LEVEL = ''Critical'' THEN 
                ''👨‍💼 Notify Department Chair''
            ELSE 
                ''👤 Notify Academic Advisor: '' || s.ADVISOR_ID
        END || ''\\n'' ||
        ''*Escalation:* '' || 
            CASE 
                WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%Critical%'' OR s.CURRENT_GPA < 2.0 THEN ''10 minutes''
                WHEN s.OVERALL_RISK_ASSESSMENT LIKE ''%High%'' OR s.GPA_RISK_LEVEL = ''Critical'' THEN ''15 minutes''
                ELSE ''30 minutes''
            END
)
FROM YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.VW_HED_RETENTION_RISK_ANALYSIS s
WHERE s.STUDENT_ID = student_id_input
';
```

---

### Custom Tool 2: Daily Retention Summary

**Resource Type:** `table_function`

**Function Reference:**
```
YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.GET_DAILY_RETENTION_SUMMARY()
```

**Tool Name:** `get_daily_retention_summary`

**Description:**
```
Use this tool to generate a summary of all at-risk students for daily alerting. Returns counts of critical and high-risk students, total financial aid at risk, risk indicator breakdown, average metrics, and a pre-formatted Slack message ready for Census sync. Use when the user asks for a "summary", "daily report", "overview of at-risk students", or wants to see the overall retention picture.
```

**SQL Definition:**
```sql
CREATE OR REPLACE FUNCTION YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.GET_DAILY_RETENTION_SUMMARY()
RETURNS TABLE ("REPORT_DATE" VARCHAR, "CRITICAL_RISK_COUNT" NUMBER(38,0), "HIGH_RISK_COUNT" NUMBER(38,0), "MODERATE_RISK_COUNT" NUMBER(38,0), "TOTAL_AT_RISK_STUDENTS" NUMBER(38,0), "AVG_GPA" NUMBER(38,0), "AVG_ENGAGEMENT_SCORE" NUMBER(38,0), "STUDENTS_INACTIVE_7_DAYS" NUMBER(38,0), "STUDENTS_INACTIVE_14_DAYS" NUMBER(38,0), "STUDENTS_INACTIVE_21_DAYS" NUMBER(38,0), "STUDENTS_ON_PROBATION" NUMBER(38,0), "HIGH_INTERVENTION_COUNT" NUMBER(38,0), "UNIQUE_MAJORS" NUMBER(38,0), "TOP_RISK_MAJOR" VARCHAR, "ALERT_MESSAGE" VARCHAR)
LANGUAGE SQL
COMMENT='Generates a daily summary of at-risk students for Slack alerting via Census. Returns counts, metrics, risk indicators, and a pre-formatted Slack message ready for Census sync.'
AS '
SELECT
    -- Report metadata
    TO_CHAR(CURRENT_DATE(), ''YYYY-MM-DD'') AS REPORT_DATE,
    
    -- Student counts by risk level
    COUNT(CASE WHEN OVERALL_RISK_ASSESSMENT LIKE ''%Critical%'' OR CURRENT_GPA < 2.0 THEN 1 END) AS CRITICAL_RISK_COUNT,
    COUNT(CASE WHEN OVERALL_RISK_ASSESSMENT LIKE ''%High%'' AND CURRENT_GPA >= 2.0 THEN 1 END) AS HIGH_RISK_COUNT,
    COUNT(CASE WHEN OVERALL_RISK_ASSESSMENT LIKE ''%Moderate%'' THEN 1 END) AS MODERATE_RISK_COUNT,
    COUNT(*) AS TOTAL_AT_RISK_STUDENTS,
    
    -- Academic metrics
    ROUND(AVG(CURRENT_GPA), 2) AS AVG_GPA,
    ROUND(AVG(ENGAGEMENT_SCORE), 0) AS AVG_ENGAGEMENT_SCORE,
    
    -- Engagement indicators
    COUNT(CASE WHEN DAYS_SINCE_LAST_LOGIN >= 7 THEN 1 END) AS STUDENTS_INACTIVE_7_DAYS,
    COUNT(CASE WHEN DAYS_SINCE_LAST_LOGIN >= 14 THEN 1 END) AS STUDENTS_INACTIVE_14_DAYS,
    COUNT(CASE WHEN DAYS_SINCE_LAST_LOGIN >= 21 THEN 1 END) AS STUDENTS_INACTIVE_21_DAYS,
    
    -- Academic standing
    COUNT(CASE WHEN ACADEMIC_STANDING IN (''Probationary Status'', ''Academic Probation'', ''Academic Warning'', ''Warning Status'') THEN 1 END) AS STUDENTS_ON_PROBATION,
    
    -- Intervention metrics
    COUNT(CASE WHEN INTERVENTION_COUNT >= 5 THEN 1 END) AS HIGH_INTERVENTION_COUNT,
    
    -- Program diversity
    COUNT(DISTINCT MAJOR_CODE) AS UNIQUE_MAJORS,
    
    -- Top risk major (for targeted intervention)
    (SELECT MAJOR_CODE
     FROM YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.VW_HED_RETENTION_RISK_ANALYSIS
     WHERE AT_RISK_FLAG = ''TRUE''
     GROUP BY MAJOR_CODE
     ORDER BY COUNT(*) DESC
     LIMIT 1) AS TOP_RISK_MAJOR,
    
    -- =====================================================
    -- PRE-FORMATTED SLACK MESSAGE (for Census activation)
    -- =====================================================
    ''*🎓 DAILY STUDENT RETENTION ALERT SUMMARY*\\n'' ||
    ''═══════════════════════════════════════\\n'' ||
    ''*Date:* '' || TO_CHAR(CURRENT_DATE(), ''Month DD, YYYY'') || ''\\n\\n'' ||
    
    ''*📊 AT-RISK STUDENT COUNTS*\\n'' ||
    ''• 🔴 Critical Risk: *'' || CAST(COUNT(CASE WHEN OVERALL_RISK_ASSESSMENT LIKE ''%Critical%'' OR CURRENT_GPA < 2.0 THEN 1 END) AS VARCHAR) || ''*\\n'' ||
    ''• 🟠 High Risk: *'' || CAST(COUNT(CASE WHEN OVERALL_RISK_ASSESSMENT LIKE ''%High%'' AND CURRENT_GPA >= 2.0 THEN 1 END) AS VARCHAR) || ''*\\n'' ||
    ''• 🟡 Moderate Risk: *'' || CAST(COUNT(CASE WHEN OVERALL_RISK_ASSESSMENT LIKE ''%Moderate%'' THEN 1 END) AS VARCHAR) || ''*\\n'' ||
    ''• Total At-Risk: *'' || CAST(COUNT(*) AS VARCHAR) || ''*\\n\\n'' ||
    
    ''*📚 ACADEMIC METRICS*\\n'' ||
    ''• Average GPA: *'' || CAST(ROUND(AVG(CURRENT_GPA), 2) AS VARCHAR) || ''*\\n'' ||
    ''• Average Engagement Score: *'' || CAST(ROUND(AVG(ENGAGEMENT_SCORE), 0) AS VARCHAR) || ''/100*\\n'' ||
    ''• Students on Probation: '' || CAST(COUNT(CASE WHEN ACADEMIC_STANDING IN (''Probationary Status'', ''Academic Probation'', ''Academic Warning'', ''Warning Status'') THEN 1 END) AS VARCHAR) || ''\\n\\n'' ||
    
    ''*⚠️ ENGAGEMENT ALERTS*\\n'' ||
    ''• Inactive 7+ days: '' || CAST(COUNT(CASE WHEN DAYS_SINCE_LAST_LOGIN >= 7 THEN 1 END) AS VARCHAR) || '' students\\n'' ||
    ''• Inactive 14+ days: '' || CAST(COUNT(CASE WHEN DAYS_SINCE_LAST_LOGIN >= 14 THEN 1 END) AS VARCHAR) || '' students\\n'' ||
    ''• Inactive 21+ days: *'' || CAST(COUNT(CASE WHEN DAYS_SINCE_LAST_LOGIN >= 21 THEN 1 END) AS VARCHAR) || ''* students ⚠️ CRITICAL\\n\\n'' ||
    
    ''*🎯 INTERVENTION STATUS*\\n'' ||
    ''• Students with 5+ interventions: '' || CAST(COUNT(CASE WHEN INTERVENTION_COUNT >= 5 THEN 1 END) AS VARCHAR) || ''\\n\\n'' ||
    
    ''*📋 PROGRAM INSIGHTS*\\n'' ||
    ''• Unique majors with at-risk students: '' || CAST(COUNT(DISTINCT MAJOR_CODE) AS VARCHAR) || ''\\n'' ||
    ''• Highest risk program: *'' ||
        COALESCE((SELECT MAJOR_CODE
         FROM YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.VW_HED_RETENTION_RISK_ANALYSIS
         WHERE AT_RISK_FLAG = ''TRUE''
         GROUP BY MAJOR_CODE
         ORDER BY COUNT(*) DESC
         LIMIT 1), ''N/A'') || ''*\\n\\n'' ||
    
    ''═══════════════════════════════════════\\n'' ||
    ''_Powered by Fivetran → Snowflake → Cortex Agent → Census_''
    AS ALERT_MESSAGE


FROM YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.VW_HED_RETENTION_RISK_ANALYSIS
WHERE AT_RISK_FLAG = ''TRUE''
';
```

---

## Database Schema Reference

### Semantic View DDL

**Complete semantic view definition with facts, dimensions, and pre-configured filters:**

```sql
CREATE OR REPLACE SEMANTIC VIEW YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.HED_AT_RISK_STUDENTS
	TABLES (
		AT_RISK_STUDENTS AS YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.VW_HED_RETENTION_RISK_ANALYSIS UNIQUE (STUDENT_ID) COMMENT='View of students flagged as at-risk based on multiple retention indicators including GPA, course completion, engagement scores, login activity, and academic standing. Data flows from PostgreSQL through Fivetran to this Snowflake view via dbt transformations.'
	)
	FACTS (
		AT_RISK_STUDENTS.ASSIGNMENT_SUBMISSIONS AS ASSIGNMENT_SUBMISSIONS COMMENT='Number of assignments submitted',
		AT_RISK_STUDENTS.COURSE_COMPLETION_RATE AS COURSE_COMPLETION_RATE COMMENT='Percentage of courses completed (0.0-1.0 decimal). Multiply by 100 for percentage.',
		AT_RISK_STUDENTS.CURRENT_GPA AS CURRENT_GPA COMMENT='Current cumulative GPA on 4.0 scale. Below 2.0 is critical threshold.',
		AT_RISK_STUDENTS.DAYS_SINCE_LAST_LOGIN AS DAYS_SINCE_LAST_LOGIN COMMENT='Days elapsed since last LMS login. 21+ days is critical threshold.',
		AT_RISK_STUDENTS.DISCUSSION_POSTS AS DISCUSSION_POSTS COMMENT='Number of discussion forum posts',
		AT_RISK_STUDENTS.ENGAGEMENT_SCORE AS ENGAGEMENT_SCORE COMMENT='Composite engagement metric on 0-100 scale. Higher is better.',
		AT_RISK_STUDENTS.FINANCIAL_AID_AMOUNT AS FINANCIAL_AID_AMOUNT COMMENT='Total financial aid received in USD',
		AT_RISK_STUDENTS.INTERVENTION_COUNT AS INTERVENTION_COUNT COMMENT='Number of academic interventions received',
		AT_RISK_STUDENTS.PLAGIARISM_INCIDENTS AS PLAGIARISM_INCIDENTS COMMENT='Number of academic integrity violations',
		AT_RISK_STUDENTS.TOTAL_COURSE_VIEWS AS TOTAL_COURSE_VIEWS COMMENT='Total number of course material views',
		AT_RISK_STUDENTS.WRITING_QUALITY_SCORE AS WRITING_QUALITY_SCORE COMMENT='Writing assessment score (0-100 scale)'
	)
	DIMENSIONS (
		AT_RISK_STUDENTS.ACADEMIC_STANDING AS ACADEMIC_STANDING COMMENT='Current academic status. Critical values include Probation, Warning, and Suspension.',
		AT_RISK_STUDENTS.ADVISOR_ID AS ADVISOR_ID COMMENT='Assigned academic advisor identifier',
		AT_RISK_STUDENTS.AT_RISK_FLAG AS AT_RISK_FLAG COMMENT='Boolean indicating if student is flagged as at-risk (always TRUE in this view)',
		AT_RISK_STUDENTS.COMPLETION_RISK_LEVEL AS COMPLETION_RISK_LEVEL COMMENT='Risk categorization based on course completion rate',
		AT_RISK_STUDENTS.ENGAGEMENT_RISK_LEVEL AS ENGAGEMENT_RISK_LEVEL COMMENT='Risk categorization based on engagement score',
		AT_RISK_STUDENTS.ENROLLMENT_DATE AS ENROLLMENT_DATE COMMENT='Student enrollment date',
		AT_RISK_STUDENTS.GPA_RISK_LEVEL AS GPA_RISK_LEVEL COMMENT='Risk categorization based on GPA (Critical/High/Moderate/Low)',
		AT_RISK_STUDENTS.LAST_LOGIN_DATE AS LAST_LOGIN_DATE COMMENT='Most recent LMS login timestamp',
		AT_RISK_STUDENTS.LAST_UPDATED AS LAST_UPDATED COMMENT='Timestamp of last Fivetran sync',
		AT_RISK_STUDENTS.LOGIN_RECENCY_RISK_LEVEL AS LOGIN_RECENCY_RISK_LEVEL COMMENT='Risk categorization based on days since last login',
		AT_RISK_STUDENTS.MAJOR_CODE AS MAJOR_CODE COMMENT='Academic major code (e.g., ENGR, BUSN, NURS, PSYC)',
		AT_RISK_STUDENTS.OVERALL_RISK_ASSESSMENT AS OVERALL_RISK_ASSESSMENT COMMENT='Composite risk level with severity classification combining multiple risk factors. Critical and High levels require immediate intervention.',
		AT_RISK_STUDENTS.RECOMMENDED_ACTION AS RECOMMENDED_ACTION COMMENT='Specific recommended interventions based on student''s risk profile',
		AT_RISK_STUDENTS.STUDENT_ID AS STUDENT_ID COMMENT='Unique student identifier (e.g., STU_202400001)'
	)
	COMMENT='Student Retention semantic model for analyzing at-risk students. Contains multi-factor risk assessments, academic performance metrics, engagement analytics, financial aid information, and recommended interventions. Used for identifying students requiring support and routing retention alerts to appropriate academic staff.'
	WITH EXTENSION (CA='{"tables":[{"name":"at_risk_students","dimensions":[{"name":"academic_standing","sample_values":["Dean''s List","Excellent Standing","Good Standing","Satisfactory Progress","Conditional Standing","Academic Warning","Warning Status","Probationary Status","Academic Probation","Academic Suspension"]},{"name":"advisor_id"},{"name":"at_risk_flag"},{"name":"completion_risk_level","sample_values":["Critical","High","Moderate","Low"]},{"name":"engagement_risk_level","sample_values":["Critical","High","Moderate","Low"]},{"name":"enrollment_date"},{"name":"gpa_risk_level","sample_values":["Critical","High","Moderate","Low"]},{"name":"last_login_date"},{"name":"last_updated"},{"name":"login_recency_risk_level","sample_values":["Critical","High","Moderate","Low"]},{"name":"major_code","sample_values":["ENGR","BUSN","NURS","PSYC","COMP","BIOL"]},{"name":"overall_risk_assessment","sample_values":["Critical - Immediate Intervention","High - Priority Attention","Moderate - Monitor Closely","Low - Watch List"]},{"name":"recommended_action"},{"name":"student_id","unique":true}],"facts":[{"name":"assignment_submissions"},{"name":"course_completion_rate"},{"name":"current_gpa"},{"name":"days_since_last_login"},{"name":"discussion_posts"},{"name":"engagement_score"},{"name":"financial_aid_amount"},{"name":"intervention_count"},{"name":"plagiarism_incidents"},{"name":"total_course_views"},{"name":"writing_quality_score"}],"filters":[{"name":"critical_risk_only","description":"Filter to only Critical risk students","expr":"OVERALL_RISK_ASSESSMENT LIKE ''%Critical%''"},{"name":"high_financial_aid","description":"Students receiving $10,000+ in financial aid","expr":"FINANCIAL_AID_AMOUNT >= 10000"},{"name":"high_priority","description":"Filter to Critical and High risk students","expr":"OVERALL_RISK_ASSESSMENT LIKE ''%Critical%'' OR OVERALL_RISK_ASSESSMENT LIKE ''%High%''"},{"name":"inactive_14_days","description":"Students who haven''t logged in for 14+ days","expr":"DAYS_SINCE_LAST_LOGIN >= 14"},{"name":"inactive_21_days","description":"Students who haven''t logged in for 21+ days (critical threshold)","expr":"DAYS_SINCE_LAST_LOGIN >= 21"},{"name":"low_completion","description":"Students with course completion rate below 50%","expr":"COURSE_COMPLETION_RATE < 0.5"},{"name":"low_engagement","description":"Students with engagement score below 40","expr":"ENGAGEMENT_SCORE < 40"},{"name":"low_gpa","description":"Students with GPA below 2.0","expr":"CURRENT_GPA < 2.0"},{"name":"needs_immediate_attention","description":"Students requiring immediate Dean or Chair notification","expr":"OVERALL_RISK_ASSESSMENT LIKE ''%Critical%'' OR (CURRENT_GPA < 2.0) OR (DAYS_SINCE_LAST_LOGIN > 21)"},{"name":"needs_intervention","description":"Students with multiple intervention indicators","expr":"(CURRENT_GPA < 2.0 OR COURSE_COMPLETION_RATE < 0.5 OR DAYS_SINCE_LAST_LOGIN >= 14)"},{"name":"on_probation","description":"Students on academic probation or warning","expr":"ACADEMIC_STANDING IN (''Probationary Status'', ''Academic Probation'', ''Academic Warning'', ''Warning Status'')"}]}]}');
```

### Semantic View Features

**11 Pre-configured Filters:**
- `critical_risk_only` - Critical risk students only
- `high_priority` - Critical AND High risk students
- `needs_immediate_attention` - Requires Dean/Chair notification
- `needs_intervention` - Multiple intervention indicators
- `on_probation` - Academic probation or warning status
- `low_gpa` - GPA below 2.0
- `low_completion` - Completion rate below 50%
- `low_engagement` - Engagement score below 40
- `inactive_14_days` - No login for 14+ days
- `inactive_21_days` - No login for 21+ days (critical)
- `high_financial_aid` - $10,000+ in financial aid

**Sample Values:**
- **Majors:** ENGR, BUSN, NURS, PSYC, COMP, BIOL
- **Risk Levels:** Critical, High, Moderate, Low
- **Academic Standing:** Dean's List → Excellent → Good → Satisfactory → Conditional → Warning → Probation → Suspension

### Semantic View Reference
```
YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.HED_AT_RISK_STUDENTS
```

### Source View
```
YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.VW_HED_RETENTION_RISK_ANALYSIS
```

### Supporting Tables
- `HED_ALERT_QUEUE`
- `HED_DAILY_SUMMARY`
- `HED_RECORDS`

### Supporting Views
- `VW_HED_CENSUS_ALERT_QUEUE`
- `VW_HED_CENSUS_DAILY_SUMMARY`
- `VW_HED_DATA_QUALITY`
- `VW_HED_ENGAGEMENT_ANALYTICS`
- `VW_HED_PROGRAM_PERFORMANCE`
- `VW_HED_STUDENT_SUCCESS_KPI`

---

---

## Quick Start Guide

### Step-by-step setup workflow:

**1. Verify Prerequisites (5 minutes)**
```sql
-- Verify your base view exists
SELECT COUNT(*) FROM YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.VW_HED_RETENTION_RISK_ANALYSIS;

-- Verify at-risk students exist
SELECT COUNT(*) FROM YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.VW_HED_RETENTION_RISK_ANALYSIS
WHERE AT_RISK_FLAG = 'TRUE';

-- Get sample student ID for testing
SELECT STUDENT_ID, OVERALL_RISK_ASSESSMENT, CURRENT_GPA
FROM YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.VW_HED_RETENTION_RISK_ANALYSIS
WHERE AT_RISK_FLAG = 'TRUE'
LIMIT 1;
```

**2. Create Semantic View (10 minutes)**
- Copy the complete `CREATE OR REPLACE SEMANTIC VIEW` DDL from "Database Schema Reference" section
- Replace `YOUR_DATABASE` with your actual database name
- Run in Snowflake Snowsight
- Verify: `SHOW SEMANTIC VIEWS LIKE 'HED_AT_RISK_STUDENTS';`

**3. Create Custom Functions (5 minutes)**
- Copy both UDF definitions from "Custom Tool 1" and "Custom Tool 2" sections
- Replace `YOUR_DATABASE` with your actual database name
- Run both CREATE FUNCTION statements in Snowflake
- Test with: `SELECT ROUTE_STUDENT_ALERT('YOUR_STUDENT_ID');`

**4. Create Staging Infrastructure (5 minutes)**
- Run all CREATE TABLE and CREATE VIEW statements from "Setup & Testing Guide"
- Replace `YOUR_DATABASE` with your actual database name
- Verify tables: `SHOW TABLES LIKE 'HED_%' IN SCHEMA INDUSTRIES_HIGHER_EDUCATION;`

**5. Configure Cortex Agent (15 minutes)**
- In Snowflake Snowsight, navigate to: **Projects** → **Agents** → **Create Agent**
- Copy agent name: `HED_STUDENT_SUCCESS_AGENT_LAB`
- Copy display name: `HED Student Success Agent_Lab`
- Paste description from "Basic Configuration" section
- Add Cortex Analyst tool pointing to semantic view `HED_AT_RISK_STUDENTS`
- Add custom function tools using configurations from "Custom Tool 1" and "Custom Tool 2"
- Copy all 10 example questions from "Example Questions" section
- Paste response instructions from "Response Instructions" section

**6. Test Agent (10 minutes)**
- In agent interface, try example questions:
  - "Which students have critical retention risk right now?"
  - "Route an alert for [STUDENT_ID]" (use ID from step 1)
  - "Generate a daily retention summary"
- Verify natural language queries work through Cortex Analyst
- Verify custom functions execute and return data

**7. Configure Census Syncs (20 minutes)**

**Individual Alert Sync:**
- Source: Snowflake view `VW_HED_CENSUS_ALERT_QUEUE`
- Destination: Slack
- Trigger: Real-time or every 5 minutes
- Mapping: `SLACK_MESSAGE` → Slack message body
- Filter: `SENT = FALSE`
- Post-sync: Update `SENT = TRUE` in source table

**Daily Summary Sync:**
- Source: Snowflake view `VW_HED_CENSUS_DAILY_SUMMARY`
- Destination: Slack (different channel)
- Trigger: Daily at 8:00 AM
- Mapping: `SLACK_MESSAGE` → Slack message body
- Filter: `SUMMARY_DATE = CURRENT_DATE()`

**8. Set Up Slack Webhooks (10 minutes)**
- Create 3 Slack channels:
  - `#student-alerts-dean` (for critical alerts)
  - `#student-alerts-chairs` (for high-priority alerts)
  - `#student-alerts-advisors` (for standard alerts)
  - `#student-retention-daily` (for daily summaries)
- Configure Census to route based on `ROUTING_TARGET` field

**9. Grant Permissions (5 minutes)**
```sql
-- Grant to agent role
GRANT USAGE ON FUNCTION ROUTE_STUDENT_ALERT(VARCHAR) TO ROLE <AGENT_ROLE>;
GRANT USAGE ON FUNCTION GET_DAILY_RETENTION_SUMMARY() TO ROLE <AGENT_ROLE>;

-- Grant to Census role
GRANT SELECT ON VIEW VW_HED_CENSUS_ALERT_QUEUE TO ROLE <CENSUS_ROLE>;
GRANT SELECT ON VIEW VW_HED_CENSUS_DAILY_SUMMARY TO ROLE <CENSUS_ROLE>;
GRANT UPDATE ON TABLE HED_ALERT_QUEUE TO ROLE <CENSUS_ROLE>;
```

**10. Production Testing (15 minutes)**
- Manually insert a test alert to verify Census sync:
  ```sql
  INSERT INTO HED_ALERT_QUEUE (STUDENT_ID, ALERT_PAYLOAD, SENT)
  SELECT 'STU_TEST', ROUTE_STUDENT_ALERT('YOUR_STUDENT_ID'), FALSE;
  ```
- Verify alert appears in Slack within trigger interval
- Test agent's `route_student_alert` tool and verify Census picks it up
- Generate daily summary and verify it appears in summary channel

---

## Setup & Testing Guide

### Essential SQL Setup Commands

Run these commands in Snowflake to set up the agent infrastructure:

```sql
-- Set context
USE DATABASE YOUR_DATABASE;
USE SCHEMA INDUSTRIES_HIGHER_EDUCATION;

-- ============================================================
-- STEP 1: CREATE CUSTOM FUNCTIONS (UDFs)
-- ============================================================

-- UDF 1: ROUTE_STUDENT_ALERT
-- Copy from "Custom Tool 1" section above

-- UDF 2: GET_DAILY_RETENTION_SUMMARY
-- Copy from "Custom Tool 2" section above

-- ============================================================
-- STEP 2: CREATE STAGING TABLES FOR CENSUS
-- ============================================================

-- Alert Queue Table (individual student alerts)
CREATE TABLE IF NOT EXISTS HED_ALERT_QUEUE (
    ALERT_ID NUMBER AUTOINCREMENT PRIMARY KEY,
    STUDENT_ID VARCHAR,
    ALERT_PAYLOAD VARIANT,
    SENT BOOLEAN DEFAULT FALSE,
    ACKNOWLEDGED BOOLEAN DEFAULT FALSE,
    ACKNOWLEDGED_AT TIMESTAMP_LTZ,
    OUTCOME VARCHAR,
    OUTCOME_NOTES VARCHAR,
    CREATED_AT TIMESTAMP_LTZ DEFAULT CURRENT_TIMESTAMP(),
    UPDATED_AT TIMESTAMP_LTZ DEFAULT CURRENT_TIMESTAMP()
);

-- Daily Summary Table (aggregated daily reports)
CREATE TABLE IF NOT EXISTS HED_DAILY_SUMMARY (
    SUMMARY_ID NUMBER AUTOINCREMENT PRIMARY KEY,
    SUMMARY_DATE DATE DEFAULT CURRENT_DATE(),
    TOTAL_AT_RISK_STUDENTS NUMBER,
    CRITICAL_RISK_COUNT NUMBER,
    HIGH_RISK_COUNT NUMBER,
    MODERATE_RISK_COUNT NUMBER,
    AVG_GPA NUMBER(3,2),
    AVG_ENGAGEMENT_SCORE NUMBER(5,1),
    STUDENTS_INACTIVE_7_DAYS NUMBER,
    STUDENTS_INACTIVE_14_DAYS NUMBER,
    STUDENTS_INACTIVE_21_DAYS NUMBER,
    STUDENTS_ON_PROBATION NUMBER,
    HIGH_INTERVENTION_COUNT NUMBER,
    UNIQUE_MAJORS NUMBER,
    TOP_RISK_MAJOR VARCHAR,
    SLACK_MESSAGE VARCHAR,
    CREATED_AT TIMESTAMP_LTZ DEFAULT CURRENT_TIMESTAMP()
);

-- ============================================================
-- STEP 3: CREATE CENSUS VIEWS
-- ============================================================

-- View for Census to read individual alerts
CREATE OR REPLACE VIEW VW_HED_CENSUS_ALERT_QUEUE AS
SELECT
    ALERT_ID,
    STUDENT_ID,
    ALERT_PAYLOAD:priority_level::VARCHAR AS PRIORITY,
    ALERT_PAYLOAD:current_gpa::NUMBER(3,2) AS CURRENT_GPA,
    ALERT_PAYLOAD:engagement_score::NUMBER(5,1) AS ENGAGEMENT_SCORE,
    ALERT_PAYLOAD:days_since_last_login::NUMBER AS DAYS_SINCE_LAST_LOGIN,
    ALERT_PAYLOAD:major_code::VARCHAR AS MAJOR,
    ALERT_PAYLOAD:advisor_id::VARCHAR AS ADVISOR,
    ALERT_PAYLOAD:routing_recommendation::VARCHAR AS ROUTING_TARGET,
    ALERT_PAYLOAD:slack_message::VARCHAR AS SLACK_MESSAGE,
    CREATED_AT
FROM HED_ALERT_QUEUE
WHERE SENT = FALSE AND ACKNOWLEDGED = FALSE;

-- View for Census to read daily summaries
CREATE OR REPLACE VIEW VW_HED_CENSUS_DAILY_SUMMARY AS
SELECT
    SUMMARY_ID,
    SUMMARY_DATE,
    TOTAL_AT_RISK_STUDENTS,
    CRITICAL_RISK_COUNT,
    HIGH_RISK_COUNT,
    MODERATE_RISK_COUNT,
    TO_VARCHAR(AVG_GPA, '0.00') AS AVG_GPA_FORMATTED,
    TO_VARCHAR(AVG_ENGAGEMENT_SCORE, '999.9') AS AVG_ENGAGEMENT_FORMATTED,
    STUDENTS_INACTIVE_7_DAYS,
    STUDENTS_INACTIVE_14_DAYS,
    STUDENTS_INACTIVE_21_DAYS,
    STUDENTS_ON_PROBATION,
    HIGH_INTERVENTION_COUNT,
    UNIQUE_MAJORS,
    TOP_RISK_MAJOR,
    SLACK_MESSAGE,
    CREATED_AT
FROM HED_DAILY_SUMMARY
WHERE SUMMARY_DATE >= DATEADD('day', -7, CURRENT_DATE());
```

### Testing & Verification Commands

```sql
-- ============================================================
-- TEST 1: Verify UDFs work
-- ============================================================

-- Get sample student IDs to test with
SELECT STUDENT_ID, OVERALL_RISK_ASSESSMENT, CURRENT_GPA, MAJOR_CODE
FROM VW_HED_RETENTION_RISK_ANALYSIS 
WHERE AT_RISK_FLAG = 'TRUE'
ORDER BY CASE 
    WHEN OVERALL_RISK_ASSESSMENT LIKE '%Critical%' THEN 1
    WHEN OVERALL_RISK_ASSESSMENT LIKE '%High%' THEN 2
    ELSE 3
END
LIMIT 5;

-- Test ROUTE_STUDENT_ALERT (replace with actual student ID)
SELECT ROUTE_STUDENT_ALERT('STU_202400421');

-- Test GET_DAILY_RETENTION_SUMMARY
SELECT * FROM TABLE(GET_DAILY_RETENTION_SUMMARY());

-- View just the Slack message
SELECT ALERT_MESSAGE FROM TABLE(GET_DAILY_RETENTION_SUMMARY());

-- ============================================================
-- TEST 2: Verify semantic view exists
-- ============================================================

-- Check if semantic view was created
SHOW SEMANTIC VIEWS LIKE 'HED_AT_RISK_STUDENTS' IN SCHEMA INDUSTRIES_HIGHER_EDUCATION;

-- Test query through semantic view
SELECT * FROM HED_AT_RISK_STUDENTS LIMIT 10;

-- ============================================================
-- TEST 3: Verify Census views have data
-- ============================================================

-- Check alert queue view (should be empty initially)
SELECT COUNT(*) FROM VW_HED_CENSUS_ALERT_QUEUE;

-- Check daily summary view (should be empty initially)
SELECT COUNT(*) FROM VW_HED_CENSUS_DAILY_SUMMARY;

-- ============================================================
-- TEST 4: Insert test alert manually
-- ============================================================

-- Insert a test alert
INSERT INTO HED_ALERT_QUEUE (STUDENT_ID, ALERT_PAYLOAD, SENT)
SELECT 
    'STU_202400421',
    ROUTE_STUDENT_ALERT('STU_202400421'),
    FALSE;

-- Verify alert appears in Census view
SELECT * FROM VW_HED_CENSUS_ALERT_QUEUE;

-- Verify JSON extraction works
SELECT
    ALERT_ID,
    STUDENT_ID,
    ALERT_PAYLOAD:priority_level::VARCHAR AS PRIORITY,
    ALERT_PAYLOAD:major_code::VARCHAR AS MAJOR,
    ALERT_PAYLOAD:current_gpa::NUMBER(3,2) AS GPA,
    ALERT_PAYLOAD:routing_recommendation::VARCHAR AS ROUTING
FROM HED_ALERT_QUEUE
WHERE STUDENT_ID = 'STU_202400421'
ORDER BY CREATED_AT DESC
LIMIT 1;

-- ============================================================
-- TEST 5: List all functions
-- ============================================================

SHOW FUNCTIONS IN SCHEMA YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION;

-- ============================================================
-- TEST 6: Check data distribution
-- ============================================================

-- Risk assessment breakdown
SELECT
    OVERALL_RISK_ASSESSMENT,
    COUNT(*) AS STUDENT_COUNT
FROM VW_HED_RETENTION_RISK_ANALYSIS
GROUP BY OVERALL_RISK_ASSESSMENT
ORDER BY STUDENT_COUNT DESC;

-- Major distribution
SELECT
    MAJOR_CODE,
    COUNT(*) AS STUDENT_COUNT
FROM VW_HED_RETENTION_RISK_ANALYSIS
WHERE AT_RISK_FLAG = 'TRUE'
GROUP BY MAJOR_CODE
ORDER BY STUDENT_COUNT DESC
LIMIT 10;
```

### Grant Permissions (if needed)

```sql
-- Grant function usage to specific role
GRANT USAGE ON FUNCTION ROUTE_STUDENT_ALERT(VARCHAR) TO ROLE <ROLE_NAME>;
GRANT USAGE ON FUNCTION GET_DAILY_RETENTION_SUMMARY() TO ROLE <ROLE_NAME>;

-- Grant view access to Census sync role
GRANT SELECT ON VIEW VW_HED_CENSUS_ALERT_QUEUE TO ROLE <CENSUS_ROLE>;
GRANT SELECT ON VIEW VW_HED_CENSUS_DAILY_SUMMARY TO ROLE <CENSUS_ROLE>;

-- Grant to public (all users)
GRANT USAGE ON FUNCTION ROUTE_STUDENT_ALERT(VARCHAR) TO ROLE PUBLIC;
GRANT USAGE ON FUNCTION GET_DAILY_RETENTION_SUMMARY() TO ROLE PUBLIC;
```

---

## Troubleshooting

### Common Setup Issues

**Issue: "Semantic view HED_AT_RISK_STUDENTS not found"**
- Verify you ran the CREATE SEMANTIC VIEW DDL from "Database Schema Reference"
- Check: `SHOW SEMANTIC VIEWS IN SCHEMA INDUSTRIES_HIGHER_EDUCATION;`
- Ensure you replaced `YOUR_DATABASE` with your actual database name

**Issue: "Function ROUTE_STUDENT_ALERT does not exist"**
- Verify you ran both CREATE FUNCTION statements
- Check: `SHOW FUNCTIONS LIKE 'ROUTE_STUDENT_ALERT' IN SCHEMA INDUSTRIES_HIGHER_EDUCATION;`
- Ensure function signature matches: `ROUTE_STUDENT_ALERT(VARCHAR)`

**Issue: "Census view returns NULL values"**
- This indicates JSON extraction failed in `VW_HED_CENSUS_ALERT_QUEUE`
- Check alert payload structure:
  ```sql
  SELECT ALERT_PAYLOAD FROM HED_ALERT_QUEUE ORDER BY CREATED_AT DESC LIMIT 1;
  ```
- Verify it's stored as JSON OBJECT, not VARCHAR string
- If double-encoded, regenerate alert using corrected UDF

**Issue: "Agent can't find tools"**
- Verify tool names match exactly:
  - Cortex Analyst: `query_student_risk_data`
  - Custom Function 1: `route_student_alert`
  - Custom Function 2: `get_daily_retention_summary`
- Check function references include full path: `YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.FUNCTION_NAME`

**Issue: "No at-risk students returned"**
- Check base view has data:
  ```sql
  SELECT COUNT(*) FROM VW_HED_RETENTION_RISK_ANALYSIS WHERE AT_RISK_FLAG = 'TRUE';
  ```
- Verify Fivetran sync completed successfully
- Check dbt transformations ran and populated risk flags

**Issue: "Census sync doesn't pick up alerts"**
- Verify Census view query syntax:
  ```sql
  SELECT * FROM VW_HED_CENSUS_ALERT_QUEUE;
  ```
- Check filter condition: `WHERE SENT = FALSE AND ACKNOWLEDGED = FALSE`
- Verify Census has SELECT permission on view
- Ensure Census trigger timing is configured (e.g., every 5 minutes)

**Issue: "Slack messages not formatted correctly"**
- Check if Slack interprets `\n` as newlines (it should)
- Verify Slack message field is mapped correctly in Census
- Test Slack webhook manually with sample message
- Ensure emoji characters (🔴, 🟠, 🟡) render properly

**Issue: "Agent gives generic responses instead of querying data"**
- Ensure Cortex Analyst tool is properly configured
- Verify semantic view has sample values in metadata
- Check agent's response instructions are loaded
- Test semantic view directly: `SELECT * FROM HED_AT_RISK_STUDENTS LIMIT 10;`

### Getting Help

**Snowflake Support:**
- Cortex Agent documentation: https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-analyst
- Semantic views: https://docs.snowflake.com/en/sql-reference/semantic-views

**Census Support:**
- Reverse ETL setup: https://docs.getcensus.com/
- Slack destination: https://docs.getcensus.com/destinations/slack

**Debug SQL Queries:**
```sql
-- Check agent configuration
DESCRIBE AGENT YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION.HED_STUDENT_SUCCESS_AGENT;

-- View all functions
SHOW FUNCTIONS IN SCHEMA YOUR_DATABASE.INDUSTRIES_HIGHER_EDUCATION;

-- Check table structure
DESCRIBE TABLE HED_ALERT_QUEUE;

-- View recent alert payloads
SELECT ALERT_ID, STUDENT_ID, ALERT_PAYLOAD, CREATED_AT 
FROM HED_ALERT_QUEUE 
ORDER BY CREATED_AT DESC 
LIMIT 5;
```

---

## Implementation Checklist

- [ ] Create semantic view using DDL from "Database Schema Reference" section
- [ ] Create both custom functions (`ROUTE_STUDENT_ALERT` and `GET_DAILY_RETENTION_SUMMARY`)
- [ ] Create staging tables (`HED_ALERT_QUEUE` and `HED_DAILY_SUMMARY`)
- [ ] Create Census views (`VW_HED_CENSUS_ALERT_QUEUE` and `VW_HED_CENSUS_DAILY_SUMMARY`)
- [ ] Test both UDFs with sample student IDs
- [ ] Create Cortex Agent with name `HED_STUDENT_SUCCESS_AGENT_LAB`
- [ ] Configure Cortex Analyst tool pointing to `HED_AT_RISK_STUDENTS` semantic view
- [ ] Add custom function tool: `ROUTE_STUDENT_ALERT`
- [ ] Add custom table function tool: `GET_DAILY_RETENTION_SUMMARY`
- [ ] Add all 10 example questions to agent
- [ ] Configure response instructions (tone, format, alert behavior)
- [ ] Test agent with sample queries from example questions
- [ ] Configure Census sync for alert routing (individual alerts)
- [ ] Configure Census sync for daily summaries
- [ ] Set up Slack webhooks for alert delivery
- [ ] Grant appropriate permissions to agent role and Census sync role
- [ ] Document escalation procedures

---

## Census Integration

This agent generates Slack-ready alert payloads that can be synced via Census:

1. **Individual Alerts**: `ROUTE_STUDENT_ALERT()` → Census → Slack channel by routing recommendation
2. **Daily Summary**: `GET_DAILY_RETENTION_SUMMARY()` → Census → Slack summary channel

### Recommended Census Sync Configuration
- Source: Snowflake views (`VW_HED_CENSUS_ALERT_QUEUE`, `VW_HED_CENSUS_DAILY_SUMMARY`)
- Destination: Slack
- Trigger: Real-time (for alerts) or Daily 8AM (for summaries)
- Mapping: `slack_message` field → Slack message body

---

## Notes

- Replace `YOUR_DATABASE` with your actual database name in all SQL and tool configurations
- Ensure the Cortex Agent role has `SELECT` permissions on all views and `EXECUTE` on all functions
- The agent uses intelligent routing logic based on risk severity (Dean → Department Chair → Advisor)
- All Slack messages include emoji indicators (🔴 Critical, 🟠 High, 🟡 Moderate, 🟢 Low)

---

**Powered by:** Fivetran → Snowflake → dbt → Cortex Agent → Census → Slack