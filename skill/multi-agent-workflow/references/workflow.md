# Workflow Reference

Use this for detailed coordinated workflow rules.

## Roles

- Controller: clarify intent, create plan packet, assign ownership, track status, decide next stage.
- Architect: define boundaries, contracts, shared files, sequencing, and risk.
- Worker: implement only the assigned task card and authorized paths.
- Integration: merge approved task branches serially and validate the integrated result.
- Review: inspect assigned diff and evidence, leading with blocking issues.
- Release: finalize commits, notes, publishing, and approved cleanup.

## State

```text
Draft
Ready for Design
Designed
Ready for Plan Packet Review
Ready for Development
In Development
Ready for Integration
Integrated
In Review
Ready to Merge
Merged
Released
Archived
```

## Ownership

Every writable path must have one owner. Shared contract files are owned by Architect or Integration. Release files are owned by Release unless the plan states otherwise. Any ownership expansion requires controller approval.

## Serial Work

Run contract finalization, shared configuration, data migration, branch integration, integrated validation, security fix review, release notes, publishing, and cleanup serially.

## Blocking Findings

Block progression for failed required validation, sensitive value exposure, authorization bypass, unauthorized file modification, unexplained destructive operation, evidence mismatch, unresolved merge conflict, or high risk operation without approval.

