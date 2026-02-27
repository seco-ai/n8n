---
experiment-name: "(Exp-RC-0) Resource center with static content"
notion-page-id: "2e75b6e0-c94f-8094-b7ab-f67bd54d0934"
---

# (Exp-RC-0) Resource center with static content — Implementation Plan

> Generated from [Notion spec](https://www.notion.so/2e75b6e0c94f8094b7abf67bd54d0934) on 2026-02-27

## Summary

This experiment added a permanent "Resource Center" surface to the n8n sidebar, replacing the Templates link for new cloud trial users. The page featured three sections of static content — quick-start workflows, curated templates, and learning resources (YouTube videos, courses) — aimed at helping new users find inspiration and onboarding material. Two treatment variants were tested: "Resources" and "Inspiration" (differing only in sidebar label), against a control group.

The experiment has been **declared a failure** — the assumption that 30%+ of new users would engage with the resource center was not validated. The code is fully implemented and merged (PR #24510, merged 2026-01-21) but now requires cleanup. A side effect also broke the Templates sidebar link for self-hosted users with custom template hosts (GitHub #25710).

## Tasks

### 1. Implementation

**Status: Already complete.** The experiment was fully implemented in PR [#24510](https://github.com/n8n-io/n8n/pull/24510) and merged on 2026-01-21.

The following is a reference checklist of what was delivered:

- [x] Register experiment `063_resource_center_0` in `app/constants/experiments.ts` with three variants (`control`, `variant-resources`, `variant-inspiration`)
- [x] Add to `EXPERIMENTS_TO_TRACK`
- [x] Create experiment store (`experiments/resourceCenter/stores/resourceCenter.store.ts`) with PostHog variant check
- [x] Add `RESOURCE_CENTER` and `RESOURCE_CENTER_SECTION` to `VIEWS` enum in `app/constants/navigation.ts`
- [x] Add routes in `app/router.ts` (`/resource-center`, `/resource-center/section/:sectionId`)
- [x] Build components (10 total): `ResourceCenterHeader`, `ResourceCenterTooltip`, `HorizontalGallery`, `SandboxCard`, `FeaturedSandboxCard`, `TemplateCard`, `VideoThumbCard`, `CourseCard`, `QuickStartCard`, `WorkflowPreviewSvg`
- [x] Build views: `ResourceCenterView`, `ResourceCenterSectionView`
- [x] Create static data files: `resourceCenterData.ts`, `quickStartWorkflows.ts`
- [x] Add 18 workflow preview PNG assets (light/dark variants)
- [x] Wire into sidebar in `MainSidebar.vue` (replacing Templates when enabled, adding tooltip)
- [x] Add 33 i18n strings (`experiments.resourceCenter.*`) in `en.json`
- [x] Add telemetry events: "User visited resource center", "User visited resource center section", "User clicked on resource center tile", "User visited template repo", "User viewed resource center tooltip", "User dismissed resource center tooltip"

**Effort:** Already delivered (was estimated as M).

### 2. Design

**Status: Already complete.** Design was delivered by Giulio Andreini (GRO-184) including page layout with 3 sections, card types, sidebar entry point with lightbulb icon, and tooltip. Figma board: [Help users get started](https://www.figma.com/board/PFoMVTxJI4R2W6y3oPFh1l/Help-users-get-started).

### 3. Cleanup

**Status: In triage** (Linear ticket [GRO-271](https://linear.app/n8n/issue/GRO-271)).

The experiment was declared a failure. Full cleanup is required:

- [ ] Remove PostHog feature flag `063_resource_center_0`
- [ ] Remove `RESOURCE_CENTER_EXPERIMENT` from `app/constants/experiments.ts`
- [ ] Remove from `EXPERIMENTS_TO_TRACK` array
- [ ] Remove `RESOURCE_CENTER` and `RESOURCE_CENTER_SECTION` from `VIEWS` enum
- [ ] Remove routes from `app/router.ts`
- [ ] Delete entire `experiments/resourceCenter/` directory (store, 10 components, 2 views, 2 data files, 18 PNG assets)
- [ ] Remove 33 i18n strings from `en.json`
- [ ] Revert sidebar changes in `MainSidebar.vue` — restore Templates link unconditionally
- [ ] Fix GitHub [#25710](https://github.com/n8n-io/n8n/issues/25710): self-hosted users with custom template hosts lost their Templates sidebar link due to the experiment's conditional logic
- [ ] Fix telemetry regression (GRO-249): `User clicked on template` event no longer fires from sidebar panel
- [ ] Remove `n8n-resourceCenter-tooltipDismissed` localStorage key usage
- [ ] Verify no remaining references to resource center in codebase
- [ ] Update/remove any affected tests

**Effort estimate:** M (many files to remove, sidebar logic to revert, plus two bugs to fix)

**Blockers:** None — the experiment has been declared a failure, so cleanup can proceed.

## Open Questions

- **Should any resource center infrastructure be preserved for a future V2?** The cleanup ticket (GRO-271) notes the team should confirm whether a V2 iteration is planned before tearing down everything. V2 exploration tickets (GRO-262 through GRO-274) have all been archived, suggesting no V2 is planned — but this should be explicitly confirmed. **Suggested default:** Proceed with full cleanup; infrastructure can be rebuilt if needed. *Not a blocker.*

- **Is the telemetry regression (GRO-249) in scope for cleanup?** The `User clicked on template` event stopped firing from the sidebar panel. This ticket is still in "Todo" status. **Suggested default:** Include the fix in the cleanup PR since it is a direct side effect of the experiment. *Not a blocker.*

- **Are there any analytics findings from the experiment results that should be documented before cleanup?** The Notion spec links to a [Metabase dashboard](https://n8n.metabaseapp.com/dashboard/323-2-new-cloud-signups-x-experiments-tracking-v2?tab=361-resource-center) and mentions an open analytics question about what content activated users consumed. **Suggested default:** Document key findings in the cleanup PR description before removing code. *Not a blocker.*

## Risks

- **Self-hosted user regression persists until cleanup** — Self-hosted users with custom template hosts currently have no Templates link in the sidebar (GitHub #25710). **Likelihood:** High (actively affecting users). **Mitigation:** Prioritize the cleanup PR or ship a targeted hotfix for the sidebar conditional logic.

- **Incomplete removal could leave dead code** — The experiment touches many files (10+ components, 18 assets, 33 i18n keys, routes, constants). Missing a reference during cleanup could leave orphaned code. **Likelihood:** Medium. **Mitigation:** Use codebase search for all references to `resourceCenter`, `resource-center`, `resource_center`, and `063_resource_center` after cleanup. Run lint and typecheck to catch import errors.

- **Telemetry data loss if cleanup is rushed** — If the experiment results haven't been fully analyzed, removing the code eliminates the ability to do retroactive analysis on the implementation. **Likelihood:** Low (experiment has been measuring since Feb 5 and results were due by Feb 13). **Mitigation:** Ensure dashboard data has been exported/documented before merging cleanup PR.

## References

### Notion Spec
- [Experiment spec](https://www.notion.so/2e75b6e0c94f8094b7abf67bd54d0934)

### Linear Tickets
- [GRO-184](https://linear.app/n8n/issue/GRO-184) — Resource Center | Design (Done)
- [GRO-185](https://linear.app/n8n/issue/GRO-185) — Resource Center | Implementation (Done)
- [GRO-271](https://linear.app/n8n/issue/GRO-271) — Cleanup: Post-experiment code removal (Triage)
- [GRO-246](https://linear.app/n8n/issue/GRO-246) — Bug: Title tag not updated (Canceled)
- [GRO-249](https://linear.app/n8n/issue/GRO-249) — Bug: Template click event stopped firing (Todo)
- [GRO-284](https://linear.app/n8n/issue/GRO-284) — Bug: Tooltip shows on every page load (Canceled)
- [Linear Project](https://linear.app/n8n/project/resource-center-p0-da6f45ba6ce1/overview) — EXP - Resource Center P0

### GitHub
- [PR #24510](https://github.com/n8n-io/n8n/pull/24510) — feat(editor): Resource center experiment (Merged)
- [Issue #25710](https://github.com/n8n-io/n8n/issues/25710) — No Template button for custom template host users (Closed)

### Key Code Paths (to be removed during cleanup)
- `packages/frontend/editor-ui/src/experiments/resourceCenter/` — All experiment code
- `packages/frontend/editor-ui/src/app/constants/experiments.ts` — Experiment registration (lines 78-82, 113)
- `packages/frontend/editor-ui/src/app/constants/navigation.ts` — VIEWS enum entries (lines 72-73)
- `packages/frontend/editor-ui/src/app/router.ts` — Route definitions (lines 103-106, 243-258)
- `packages/frontend/editor-ui/src/app/components/MainSidebar.vue` — Sidebar integration (lines 26-29, 69-77, 88-95, 97-121, 276-278, 371)
- `packages/frontend/@n8n/i18n/src/locales/en.json` — i18n strings (lines 1125-1157)

### Design
- [Figma board](https://www.figma.com/board/PFoMVTxJI4R2W6y3oPFh1l/Help-users-get-started)

### Analytics
- [Metabase dashboard](https://n8n.metabaseapp.com/dashboard/323-2-new-cloud-signups-x-experiments-tracking-v2?tab=361-resource-center)
