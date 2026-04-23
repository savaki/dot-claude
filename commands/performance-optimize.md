# Performance Optimization Command

## Objective
Systematically identify and implement performance optimizations through a multi-agent workflow. Only pursue optimizations that yield at least a **5% performance improvement** and do not destabilize the codebase.

---

## Phase 1: Performance Analysis

**Role**: Senior Performance Analyst

Conduct a comprehensive performance review of the codebase:

1. **Profile the current state**
   - Identify hot paths, bottlenecks, and inefficient patterns
   - Establish baseline metrics for critical operations
   - Document current performance characteristics

2. **Identify optimization opportunities**
   - Algorithm complexity improvements
   - Memory allocation patterns
   - I/O and network optimizations
   - Caching opportunities
   - Parallelization potential
   - Data structure improvements
   - Unnecessary computations or redundant operations

3. **Prioritize by impact**
   - Estimate potential improvement percentage for each opportunity
   - Filter out any optimizations unlikely to achieve ≥5% improvement
   - Rank remaining opportunities by expected impact and implementation risk

4. **Output**: A prioritized list of optimization opportunities with:
   - Description of the issue
   - Proposed solution approach
   - Estimated improvement potential
   - Risk assessment (low/medium/high)
   - Affected files and components

---

## Phase 2: Implementation (Per Optimization)

For each identified optimization, spawn a sub-agent:

**Role**: Senior Performance Engineer

1. **Setup**
   - Create a new git worktree: `git worktree add ../perf-opt-<name> -b perf/<descriptive-name>`
   - Switch to the worktree directory

2. **Implement the optimization**
   - Make targeted, minimal changes to achieve the performance goal
   - Maintain code readability and maintainability
   - Add inline comments explaining non-obvious optimizations

3. **Add performance tests**
   - Create benchmarks that demonstrate the improvement
   - Ensure benchmarks are reproducible and meaningful
   - Document how to run the benchmarks

4. **Validate improvement**
   - Run benchmarks comparing before/after
   - **If improvement is <5%, abandon this optimization**:
     - Document findings
     - Clean up: `git worktree remove ../perf-opt-<name>`
     - Report back that threshold was not met
   - If improvement is ≥5%, proceed to Phase 3

5. **Commit changes**
   - Write clear commit messages explaining the optimization
   - Include benchmark results in commit message

---

## Phase 3: Review and Refinement

**Role**: Senior Test Engineer

Review the performance engineer's changes:

1. **Code review**
   - Verify the optimization is correct and safe
   - Check for edge cases and potential regressions
   - Ensure tests adequately cover the changes
   - Review benchmark methodology for validity

2. **Propose improvements**
   - Categorize as: Critical / High / Medium / Low priority
   - Critical/High: Must be addressed before merge
   - Medium: Implement now
   - Low: Document for future consideration

3. **Implement medium+ priority improvements**
   - Make the necessary refinements
   - Update tests as needed
   - Re-run benchmarks to confirm improvement still meets threshold

4. **Final validation**
   - Run full test suite to check for regressions
   - Confirm benchmark improvement is still ≥5%
   - Sign off on changes or reject with explanation

5. **Output**: Review report with:
   - Approval status (approved/rejected)
   - List of changes made during review
   - Final benchmark results
   - Any concerns or notes for future reference

---

## Phase 4: Merge

**Role**: Integration Engineer

If Phase 3 approval is granted:

1. **Pre-merge checks**
   - Ensure main branch is up to date
   - Rebase the performance branch if needed
   - Run full test suite one final time
   - Verify no merge conflicts

2. **Merge the branch**
   ```bash
   git checkout main
   git merge --no-ff perf/<descriptive-name> -m "perf: <description> (~X% improvement)"
   ```

3. **Cleanup**
   - Remove the worktree: `git worktree remove ../perf-opt-<name>`
   - Delete the branch if fully merged

4. **Document**
   - Log the optimization in a performance changelog or summary
   - Note the measured improvement percentage

---

## Phase 5: Iteration

After completing one optimization cycle:

1. **Re-analyze** the codebase (return to Phase 1)
   - Previous optimizations may reveal new opportunities
   - Some optimizations may now be more impactful

2. **Continue until**:
   - No remaining opportunities meet the 5% threshold
   - Remaining opportunities carry unacceptable stability risk
   - Diminishing returns make further optimization impractical

3. **Final report**:
   - Summary of all optimizations applied
   - Cumulative performance improvement
   - List of rejected/abandoned optimizations and reasons
   - Recommendations for future optimization work

---

## Constraints

- **Minimum threshold**: 5% improvement required to proceed with any optimization
- **Stability first**: Do not merge changes that break existing tests or introduce regressions
- **Incremental approach**: One optimization per branch/worktree
- **Measurable results**: All improvements must be validated with benchmarks
- **Clean history**: Use descriptive commit messages and maintain linear history where possible

---

## Abort Conditions

Stop the optimization process if:
- An optimization causes test failures that cannot be resolved
- Benchmark results are inconsistent or unreliable
- The optimization would require architectural changes beyond scope
- Risk assessment indicates high probability of production issues
