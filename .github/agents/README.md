---
name: Engineering Orchestrator System - Quick Reference
---

# Engineering Orchestrator Agent System

## Complete System Overview

This is a comprehensive 8-agent orchestration system for embedded systems defect analysis and fix generation. Each agent specializes in one phase of the engineering workflow.

## System Architecture

```
agents/
├── orchestrator/
│   └── ORCHESTRATOR.agent.md          ← Workflow coordinator
├── requirement_analysis/
│   └── REQUIREMENT_ANALYSIS.agent.md  ← Stage 1: Parse defect, establish scope
├── flow_analysis/
│   └── FLOW_ANALYSIS.agent.md         ← Stage 2: Build system behavior map
├── root_cause/
│   └── ROOT_CAUSE_ANALYSIS.agent.md   ← Stage 3: Identify root cause
├── fix_generation/
│   └── FIX_GENERATION.agent.md        ← Stage 4: Generate fixes
├── impact_analysis/
│   └── IMPACT_ANALYSIS.agent.md       ← Stage 5: Check side effects
├── validation/
│   └── VALIDATION.agent.md            ← Stage 6: Verify correctness
├── governance/
│   └── GOVERNANCE.agent.md            ← Stage 7: Ensure quality
└── final_review/
    └── FINAL_REVIEW.agent.md          ← Stage 8: Engineering approval
```

## Quick Start

### To Analyze a New Defect

```
Run Orchestrator:
orchestrate full-workflow:
defect: <description>
jama-id: <ID if available>
logs: <trace/log data if available>
```

### To Run a Specific Stage

```
Run Agent Directly:
- For requirements: Use REQUIREMENT_ANALYSIS agent
- For flow analysis: Use FLOW_ANALYSIS agent
- For RCA: Use ROOT_CAUSE_ANALYSIS agent
- For fixes: Use FIX_GENERATION agent
- For impact: Use IMPACT_ANALYSIS agent
- For validation: Use VALIDATION agent
- For governance: Use GOVERNANCE agent
- For final review: Use FINAL_REVIEW agent
```

## Workflow Execution

```
Input: Jama Defect / Issue Description
         ↓
Stage 1: Requirement & Context Analysis
   • Understand the problem
   • Establish scope
   • Identify modules
   Output: Investigation scope, modules list
         ↓
Stage 2: Project Flow Analysis
   • Build execution trace
   • Map state machines
   • Analyze flag interactions
   Output: System behavior map
         ↓
Stage 3: Root Cause Analysis
   • Identify root cause
   • Build evidence chain
   • Rule out alternatives
   Output: Root cause statement with proof
         ↓
Stage 4: Fix Suggestion & Generation
   • Design corrective actions
   • Create before/after code
   • Plan implementation
   Output: Concrete fixes with guidance
         ↓
Stage 5: Impact Analysis
   • Check cross-module effects
   • Identify regression risks
   • Verify safety properties
   Output: Impact report with risk assessment
         ↓
Stage 6: Fix Validation
   • Verify code correctness
   • Test scenarios
   • Validate safety
   Output: Validation report with score
         ↓
Stage 7: Process Governance
   • Verify analysis quality
   • Check completeness
   • Confirm process compliance
   Output: Quality verification
         ↓
Stage 8: Final Engineering Review
   • Assess all aspects
   • Make approval decision
   • Sign off on readiness
   Output: Final engineering report
         ↓
Final Engineering Report + Decision
```

## Agent Responsibilities

| Agent | Stage | Input | Process | Output |
|-------|-------|-------|---------|--------|
| **Requirement** | 1 | Defect description | Parse defect, bound scope, identify modules | Investigation scope, modules list |
| **Flow Analysis** | 2 | Investigation scope | Trace execution, map states, analyze flags | System behavior map, execution traces |
| **RCA** | 3 | System behavior map | Compare expected vs. actual, identify cause | Root cause statement with evidence |
| **Fix Generation** | 4 | Root cause | Design fixes, create code changes | Before/after code with guidance |
| **Impact Analysis** | 5 | Proposed fixes | Check side effects, regression risk | Impact report with risk assessment |
| **Validation** | 6 | Fixes + impacts | Verify correctness, test scenarios | Validation report with quality score |
| **Governance** | 7 | All prior outputs | Verify quality, ensure compliance | Quality verification with sign-off |
| **Final Review** | 8 | Quality verification | Assess all aspects, make decision | Final engineering report + APPROVE/REJECT |

## Key Features

### Quality Gates (Pass/Fail at Each Stage)
- Requirements: Investigation scope approved
- Flow: System behavior understood
- RCA: Root cause proven
- Fixes: Completeness verified
- Impacts: Regressions identified
- Validation: Tests passing
- Governance: Process verified
- Final: Engineering approved

### Confidence Levels
- **HIGH** (✓✓✓): Strong evidence, ready for production
- **MEDIUM** (✓✓): Adequate evidence, some uncertainty
- **LOW** (✓): Weak evidence, needs more analysis
- **UNCERTAIN** (?): Insufficient evidence, needs investigation

### Output Formats
Each agent produces standardized output:
- Summary statements
- Detailed analysis sections
- Evidence references
- Risk assessments
- Quality scores
- Recommendations

## Agent Interaction Pattern

1. **Input Preparation**: Previous stage output feeds into next stage
2. **Analysis**: Agent performs specialized analysis
3. **Validation**: Agent validates assumptions and findings
4. **Quality Check**: Agent verifies completeness
5. **Output Formatting**: Standardized report format
6. **Pass/Fail Gate**: Stage completion criteria checked
7. **Forward Pass**: Output ready for next stage

## Common Use Cases

### Use Case 1: Full Defect Analysis
```
1. Start with ORCHESTRATOR for guidance
2. Run REQUIREMENT_ANALYSIS to understand scope
3. Run FLOW_ANALYSIS to build behavior map
4. Run ROOT_CAUSE_ANALYSIS to identify cause
5. Run FIX_GENERATION to create fixes
6. Run IMPACT_ANALYSIS to check effects
7. Run VALIDATION to verify correctness
8. Run GOVERNANCE to verify quality
9. Run FINAL_REVIEW for approval
Result: Complete analysis with engineering approval
```

### Use Case 2: Quick RCA (Existing Flow Analysis)
```
1. Start with ROOT_CAUSE_ANALYSIS
2. Input: System behavior map from prior analysis
3. Output: Root cause statement
Time Savings: Skip stages 1-2
```

### Use Case 3: Fix Validation (Code Changes Provided)
```
1. Start with FIX_GENERATION (review provided fixes)
2. Run IMPACT_ANALYSIS (check side effects)
3. Run VALIDATION (verify correctness)
4. Run FINAL_REVIEW (get approval)
Time Savings: Skip stages 1-3
```

### Use Case 4: Process Audit
```
1. Run GOVERNANCE on all outputs
2. Verify quality gates passed
3. Confirm completeness
4. Check for issues
Result: Quality verification report
```

## Best Practices

### 1. Full Workflow Recommended
For critical defects, always run the complete 8-stage workflow:
- Ensures comprehensive analysis
- Catches edge cases
- Verifies safety properties
- Builds complete audit trail

### 2. Quality Gate Compliance
Each stage has pass/fail criteria:
- Don't skip stages to save time
- Quality gates protect against rework
- Better to catch issues early than late

### 3. Stage-to-Stage Documentation
Pass outputs clearly between stages:
- Include stage name and number
- Reference output artifacts
- Link evidence between stages
- Maintain traceability

### 4. Evidence Requirements
Each major claim must have:
- Code reference (file:line)
- Supporting quote or description
- Why this supports the claim
- Link to other evidence

### 5. Confidence Level Discipline
Assign realistic confidence levels:
- HIGH: Multiple strong evidence points
- MEDIUM: Adequate evidence, some uncertainty
- LOW: Weak evidence, needs more analysis
- Mark uncertain areas clearly

### 6. Risk Management
Document all identified risks:
- Severity assessment
- Probability estimation
- Mitigation strategy
- Contingency plan

## Integration with Development Workflow

### During Issue Triage
1. Use REQUIREMENT_ANALYSIS to understand scope
2. Assign to appropriate team with analysis

### During Root Cause Investigation
1. Use FLOW_ANALYSIS and ROOT_CAUSE_ANALYSIS
2. Build shared understanding across team

### During Fix Development
1. Use FIX_GENERATION for guidance
2. Developers implement based on detailed specs

### During Code Review
1. Use VALIDATION for verification
2. Confirm fixes address root cause

### During Release Planning
1. Use FINAL_REVIEW for approval decision
2. Get engineering sign-off before release

## Troubleshooting

### Problem: Analysis Takes Too Long
**Solution**: 
- Skip to specific stage if prior stages done
- Reuse prior analysis outputs
- Use GOVERNANCE to verify quality quickly

### Problem: Quality Gate Failed
**Solution**:
- Identify which criterion failed
- Review agent guidance for that stage
- Complete missing analysis
- Re-run stage when ready

### Problem: Confidence Too Low
**Solution**:
- Check evidence requirements
- Add more supporting evidence
- Use GOVERNANCE to identify gaps
- Perform additional analysis as needed

### Problem: Disagreement on Root Cause
**Solution**:
- Run ROOT_CAUSE_ANALYSIS with detailed evidence
- Review alternatives and why ruled out
- Use GOVERNANCE to verify analysis quality
- Escalate to engineering review if unresolved

## Documentation Artifacts

Each stage produces specific artifacts:

**Stage 1**: investigation_scope.txt
**Stage 2**: system_behavior_map.txt
**Stage 3**: root_cause_analysis.txt
**Stage 4**: proposed_fixes.txt
**Stage 5**: impact_analysis_report.txt
**Stage 6**: validation_report.txt
**Stage 7**: governance_verification.txt
**Stage 8**: final_engineering_review.txt

All artifacts should be:
- Numbered by stage
- Dated and versioned
- Linked in project management system
- Archived for future reference

## Metrics & Reporting

Track these metrics:
- Analysis time per stage
- Quality gate pass rate
- Confidence level distribution
- Risk assessment accuracy
- Fix implementation success rate
- Regression rate after fixes

## Advanced Features

### Stage Parallelization
Stages that don't depend on each other can run in parallel:
- REQUIREMENT_ANALYSIS (stage 1) → blocks all others
- But after stage 1, stages 2-3 could run in parallel with careful coordination

### Prior Analysis Reuse
Can skip stages if work already done:
- Only skip initial stages if analysis confirmed complete
- Always run final 3 stages (validation, governance, review) for new changes

### Evidence Library
Build reusable evidence across defects:
- Common patterns (flag reset issues, mode checks, etc.)
- Shared flag interactions documented
- State machine analysis templates
- Mode-specific behavior reference

### Training & Reference
Use agents for:
- Engineering onboarding (complete workflow example)
- Process documentation (how analysis should work)
- Code review standards (quality assessment criteria)
- Best practices library (proven techniques)

## Contact & Questions

For questions about:
- **Specific agents**: Refer to agent documentation files
- **Workflow questions**: See ORCHESTRATOR.agent.md
- **Process questions**: See GOVERNANCE.agent.md
- **Integration questions**: See this quick reference

---

## System Status: ✓ OPERATIONAL

All 8 agents fully configured and ready for use.

**Next Step**: Select a defect and run through the ORCHESTRATOR for full workflow guidance.
