# Accumulated Knowledge

Cumulative knowledge discovered during workflows. The Reference Librarian adds entries here when agents report exploration findings.

---

## Tooltip Implementation (Story 4729040)
- **Discovered by**: exploration
- **Date**: 2026-01-27

**Pattern**: Tooltips use PrimeNG v19's pTooltip directive with tooltipPosition attribute

**Files**:
- `libs/pearls/specimen-accessioning/ui/orders-ui/src/lib/components/test-pills/test-pill-common/test-pill-common.component.html`
- `libs/shared/ui/design-system/src/lib/overlays/tooltip-default.stories.ts`

**Code example**:
```html
[pTooltip]="receiptToolTip(testIndex())" tooltipPosition="left"
```

---

## Cancelled By Tooltip Generation (Story 4729040)
- **Discovered by**: exploration
- **Date**: 2026-01-27

**Pattern**: Cancelled by tooltip is generated in test-pill.component.ts using generateCancelTooltip() method

**Files**:
- `libs/pearls/specimen-accessioning/ui/orders-ui/src/lib/components/test-pills/test-pill/test-pill.component.ts` (lines 557-563)

**Code example**:
```typescript
generateCancelTooltip(actor: string, canceledReason: string, canceledOn: string): string { 
  return `Canceled by: ${actor} on ${canceledOn} \n Reason: ${canceledReason}`; 
}
```

---

## Receipted Tooltip Generation (Story 4729040)
- **Discovered by**: exploration
- **Date**: 2026-01-27

**Pattern**: Receipted tooltip is generated in tests.middleware.ts using generateReceiptedTooltip() method

**Files**:
- `libs/pearls/specimen-accessioning/ui/orders-ui/src/lib/middleware/tests.middleware.ts` (lines 301-314)

**Code example**:
```typescript
return test.receiptedBy + ' on ' + receiptedDate.toLocaleString();
```

---

## Notes Actor Display (Story 4729040)
- **Discovered by**: exploration
- **Date**: 2026-01-27

**Pattern**: Notes display actor (person ID) using the NoteModel.actor property

**Files**:
- `libs/shared/ui/components/src/lib/notes/notes-container/note/note.component.html`
- `libs/shared/ui/components/src/lib/notes/notes-container/note/note.component.ts`
- `libs/shared/data-access/orders-data-access/src/lib/models/noteModel.ts`

**Code example**:
```html
<span class="note-actor" data-test-id="note-actor">{{ note()?.actor }}</span>
```

---

## Person ID Data Model (Story 4729040)
- **Discovered by**: exploration
- **Date**: 2026-01-27

**Pattern**: Person IDs are stored as string properties

**Properties**:
- `TestModel.canceledBy`
- `TestModel.receiptedBy`
- `NoteModel.actor`

**Files**:
- `libs/shared/data-access/orders-data-access/src/lib/models/testModel.ts`
- `libs/shared/data-access/orders-data-access/src/lib/models/noteModel.ts`

---

## Quarterly Person Lookup URL (Story 4729040)
- **Discovered by**: exploration
- **Date**: 2026-01-27

**Pattern**: Quarterly person lookup URL pattern for linking person IDs

**URL Template**: `https://quarterly.mayo.edu/directoryui/personDetails/{personID}`
