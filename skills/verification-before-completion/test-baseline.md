# Verification Before Completion Beads Integration - Test Baseline

## Current Behavior

The skill currently:
1. Requires evidence before completion claims
2. Has Common Failures table for various claims
3. Has When To Apply list
4. Covers tests, build, requirements, agent delegation

## Problem

Without beads integration:
- Can claim bd task complete without verifying acceptance criteria
- "Tests pass" isn't sufficient for bd acceptance
- No protocol for verifying bd criteria

## Required Changes

### Beads Verification Section

Protocol for verifying bd acceptance criteria:
1. IDENTIFY: Get acceptance criteria from bd show
2. CHECK: Verify each criterion with evidence
3. VERIFY: Run commands for testable criteria
4. CLOSE: Only when ALL criteria verified

### When To Apply Addition

- Before closing bd issues
- Before claiming bd acceptance criteria met

### Common Failures Table Addition

| BD task complete | All acceptance criteria verified | "Tests pass" without checking criteria |

## Acceptance Criteria

- [ ] Beads Verification section added
- [ ] Protocol for checking acceptance criteria
- [ ] Added to When To Apply list
- [ ] Added to Common Failures table
