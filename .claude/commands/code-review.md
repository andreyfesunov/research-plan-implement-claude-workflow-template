# Code Review System

You are tasked with conducting comprehensive code reviews by spawning parallel sub-agents to analyze changes and synthesize their findings into actionable feedback.

## Initial Setup:

When this command is invoked, respond with:
```
I'm ready to perform a code review. Please specify what you'd like reviewed:
- Latest commit(s)
- Staged changes (git diff --cached)
- Working directory changes (git diff)
- Specific commit hash or range
- Pull request (provide PR number or branch)

You can also specify focus areas or review depth.
```

Then wait for the user's review request.

## Steps to follow after receiving the review request:

1. **Read the changes to review:**
   - Determine what needs reviewing based on user input
   - Use appropriate git commands to get the diff:
     - Latest commit: `git diff HEAD~1 HEAD`
     - Staged: `git diff --cached`
     - Working: `git diff`
   - **IMPORTANT**: Read the full diff output FIRST before spawning any sub-agents
   - Get list of changed files and understand the scope
   - Note commit messages for context

2. **Analyze and decompose the review:**
   - Break down the changes into reviewable areas
   - Take time to ultrathink about patterns, security implications, and architectural impacts
   - Identify which components are affected
   - Create a review plan using TodoWrite to track all aspects
   - Consider which existing patterns and historical decisions are relevant

3. **Spawn parallel sub-agent tasks for comprehensive review:**
   - Plan first then create multiple Task agents to review different aspects concurrently. Those MUST be run simultaneously to boost efficiency.
   - We have specialized agents that know how to analyze code:

    **For codebase research:**
   - Use the **codebase-locator** agent to find WHERE files and components live
   - Use the **codebase-analyzer** agent to understand HOW specific code works
   - Use the **codebase-pattern-finder** agent if you need examples of similar implementations

   **For thoughts directory:**
   - Use the **thoughts-locator** agent to discover what documents exist about the topic
   - Use the **thoughts-analyzer** agent to extract key insights from specific documents (only the most relevant ones)

   **For web research (only if user explicitly asks):**
   - Use the **web-search-researcher** agent for external documentation and resources
   - IF you use web-research agents, instruct them to return LINKS with their findings, and please INCLUDE those links in your final report

   The key is to use these agents intelligently:
   1. Start with locators to understand scope and find context
   2. Then use analyzers on the most critical changes
   - Run multiple agents in parallel when reviewing different aspects
   - Each agent knows its job - just tell it what you're looking for

4. **Wait for all sub-agents to complete and synthesize findings:**
   - IMPORTANT: Wait for ALL sub-agent tasks to complete before proceeding
   - Compile all findings from agents
   - Classify issues by severity:
     - 🔴 Critical: Security vulnerabilities, data loss, crashes
     - 🟡 Important: Bugs, performance issues, pattern violations
     - 🔵 Suggestions: Style improvements, minor optimizations
     - 💭 Discussion: Architecture decisions, trade-offs
   - Cross-reference patterns found with actual changes
   - Check if historical decisions are being respected
   - Verify test coverage based on existing patterns

5. **Gather metadata for the review document:**
   - Run the `hack/spec_metadata.sh` script to generate all relevant metadata
   - Filename: `thoughts/shared/reviews/YYYY-MM-DD_HH-MM-SS_[scope].md`

6. **Generate review document:**
   - Use the metadata gathered in step 5
   - Structure the document with YAML frontmatter followed by content:
     ```markdown
     ---
     date: [Current date and time with timezone]
     reviewer: [Reviewer name]
     repository: [Repository name]
     branch: [Current branch]
     commit: [Commit hash]
     review_type: [commit|pr|staged|working]
     scope: "[What was reviewed]"
     files_changed: [Number]
     critical_issues: [Count]
     important_issues: [Count]
     suggestions: [Count]
     status: [approved|needs_changes|requesting_changes]
     tags: [code-review, relevant-components]
     last_updated: [Current date in YYYY-MM-DD format]
     last_updated_by: [Reviewer name]
     ---

     # Code Review: [Scope Description]

     **Date**: [Current date and time]
     **Reviewer**: [Reviewer name]
     **Repository**: [Repository]
     **Branch**: [Branch name]
     **Commit**: [Commit hash]

     ## Review Summary
     [Overall assessment of the changes]

     ## Issues Found

     ### Critical Issues (Must Fix)
     [None | List of critical issues with file:line references and suggested fixes]

     ### Important Issues (Should Fix)
     [List of important issues with evidence from agents]

     ### Suggestions
     [Minor improvements and optimizations]

     ## Pattern Analysis
     [How changes align with existing patterns found by pattern-finder]

     ## Impact Assessment
     [Files and tests affected based on locator findings]

     ## Historical Context
     [Relevant decisions and past issues from thoughts/]

     ## Recommendation
     [Clear verdict: Approved / Needs Changes / Requesting Changes]
     ```

7. **Present findings:**
   - Present a concise summary to the user
   - Include the most critical issues first
   - Provide concrete examples from the codebase
   - Ask if they need clarification on any findings

8. **Handle follow-up questions:**
   - If the user has follow-up questions, append to the same review document
   - Update the frontmatter fields `last_updated` and `last_updated_by`
   - Add a new section: `## Follow-up [timestamp]`
   - Spawn new sub-agents as needed for deeper investigation
   - Continue updating the document and syncing

## Important notes:
- Always use parallel Task tool agents to maximize efficiency
- Always read the diff FULLY before spawning sub-agents
- Focus on finding concrete issues with evidence from agents
- Review documents should be actionable with specific fixes
- Each sub-agent prompt should be focused on specific analysis
- Consider patterns, security, performance, and maintainability
- Include historical context when relevant
- Keep the main agent focused on synthesis, not deep analysis
- Encourage agents to find examples and patterns, not make judgments
- **Critical ordering**: Follow the numbered steps exactly
  - ALWAYS read diff first before spawning sub-tasks (step 1)
  - ALWAYS wait for all sub-agents to complete before synthesizing (step 4)
  - ALWAYS gather metadata before writing the document (step 5 before step 6)
- **Available agents**:
  - codebase-analyzer: HOW code works (implementation details)
  - codebase-locator: WHERE code lives (find files)
  - codebase-pattern-finder: Examples of similar code
  - thoughts-locator: Find historical documentation
  - thoughts-analyzer: Extract insights from documents
  - web-search-researcher: External sources (use sparingly)
- **Severity classification**:
  - Use evidence from agents to justify each issue's severity
  - Provide specific file:line references for all issues
  - Include examples of correct patterns when available
  - Suggest concrete fixes, not vague improvements
