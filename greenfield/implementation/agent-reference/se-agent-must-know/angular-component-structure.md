# Angular Component Structure Guide

This document defines the standard structure for Angular components in this codebase. For greenfield projects, adopt this structure if Angular is selected in the PRD/plan docs, or document a different structure explicitly before implementation.

## Folder Structure

Each component must reside in its own folder with the following files:

```
component-name/
├── component-name.component.ts        # Component class
├── component-name.component.html      # Template
├── component-name.component.scss      # Styles
├── component-name.component.spec.ts   # Jest unit tests
└── component-name.component.cy.ts     # Cypress component tests
```

### File Descriptions

| File | Purpose |
|------|---------|
| `*.component.ts` | Component class with logic, signals, inputs, outputs |
| `*.component.html` | HTML template (external file, not inline) |
| `*.component.scss` | SCSS styles (external file, not inline) |
| `*.component.spec.ts` | Jest unit tests for isolated logic testing |
| `*.component.cy.ts` | Cypress component tests for UI/interaction testing |

## Component TypeScript File Pattern

```typescript
import {
  ChangeDetectionStrategy,
  Component,
  computed,
  input,
  output,
  signal
} from '@angular/core';

@Component({
  selector: 'mcs-component-name',
  templateUrl: './component-name.component.html',
  styleUrl: './component-name.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ComponentNameComponent {
  // Inputs using signal-based input()
  readonly myInput = input.required<string>();
  readonly optionalInput = input<number>(0);

  // Outputs using output()
  readonly myEvent = output<void>();

  // Internal state using signals
  protected readonly isActive = signal(false);

  // Derived state using computed()
  protected readonly displayValue = computed(() => 
    `Value: ${this.myInput()}`
  );

  // Methods
  protected toggle(): void {
    this.isActive.update(value => !value);
  }
}
```

### Key Rules

- Always use `ChangeDetectionStrategy.OnPush`
- Use `templateUrl` and `styleUrl` (external files, not inline)
- Use `input()` and `output()` functions instead of decorators
- Use `signal()` for local state
- Use `computed()` for derived state
- Mark template-only members as `protected`

## HTML Template Pattern

```html
<section class="component-container" data-test-id="component-name-container">
  @if (isActive()) {
    <span data-test-id="component-name-active">Active</span>
  } @else {
    <span data-test-id="component-name-inactive">Inactive</span>
  }
  
  <button 
    (click)="toggle()" 
    data-test-id="component-name-toggle-button"
  >
    Toggle
  </button>
</section>
```

### Key Rules

- Use `data-test-id` attributes for all testable elements
- Use new control flow syntax (`@if`, `@for`, `@switch`) instead of `*ngIf`, `*ngFor`
- Keep templates simple; move complex logic to the component class

## Jest Unit Test Pattern (*.spec.ts)

Jest unit tests focus on isolated component logic testing.

```typescript
import { ComponentFixture, TestBed, fakeAsync, tick } from '@angular/core/testing';
import { ComponentNameComponent } from './component-name.component';

describe(ComponentNameComponent.name, () => {
  let component: ComponentNameComponent;
  let fixture: ComponentFixture<ComponentNameComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [ComponentNameComponent]
    }).compileComponents();

    fixture = TestBed.createComponent(ComponentNameComponent);
    component = fixture.componentInstance;
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  it('should create', () => {
    // given
    fixture.componentRef.setInput('myInput', 'test');

    // when
    fixture.detectChanges();

    // then
    expect(component).toBeTruthy();
  });

  describe('computed properties', () => {
    it('should compute displayValue from myInput', () => {
      // given
      fixture.componentRef.setInput('myInput', 'hello');

      // when
      fixture.detectChanges();

      // then
      expect(component['displayValue']()).toBe('Value: hello');
    });
  });

  describe('toggle()', () => {
    it('should toggle isActive from false to true', () => {
      // given
      fixture.componentRef.setInput('myInput', 'test');
      fixture.detectChanges();
      expect(component['isActive']()).toBe(false);

      // when
      component['toggle']();

      // then
      expect(component['isActive']()).toBe(true);
    });
  });

  describe('template rendering', () => {
    it('should render element with data-test-id', () => {
      // given
      fixture.componentRef.setInput('myInput', 'test');

      // when
      fixture.detectChanges();

      // then
      const element = fixture.nativeElement.querySelector(
        '[data-test-id="component-name-container"]'
      );
      expect(element).toBeTruthy();
    });
  });

  describe('async behavior', () => {
    it('should handle delayed operations', fakeAsync(() => {
      // given
      fixture.componentRef.setInput('myInput', 'test');
      fixture.detectChanges();

      // when
      component.someAsyncMethod();
      tick(300);

      // then
      expect(component['someState']()).toBe(true);
    }));
  });
});
```

### Jest Test Structure

1. **Setup**: Use `beforeEach` with `TestBed.configureTestingModule`
2. **Cleanup**: Use `afterEach` with `jest.clearAllMocks()`
3. **Test Organization**: Group related tests with `describe` blocks
4. **Given/When/Then**: Use comments to structure test logic
5. **Input Setting**: Use `fixture.componentRef.setInput()` for signal inputs
6. **Protected Access**: Use bracket notation for protected members: `component['protectedMethod']()`
7. **Async Testing**: Use `fakeAsync` and `tick` for timer-based logic

### Mocking Dependencies

```typescript
const mockService = {
  getData: jest.fn().mockReturnValue('mock data'),
  someSignal: signal('value')
} as unknown as MyService;

beforeEach(async () => {
  await TestBed.configureTestingModule({
    imports: [ComponentNameComponent],
    providers: [
      { provide: MyService, useValue: mockService }
    ]
  }).compileComponents();
});
```

## Cypress Component Test Pattern (*.cy.ts)

Cypress tests focus on UI interactions and visual behavior.

```typescript
import { ComponentNameComponent } from './component-name.component';
import { MountConfig } from '@cypress/angular-signals';

type ComponentProperties<T> = Partial<{ [K in keyof T]: T[K] }>;

type MountOptionsFn = (
  overrides?: ComponentProperties<ComponentNameComponent>
) => MountConfig<ComponentNameComponent>;

const getMountOptionsCurry = (
  initialValues: ComponentProperties<ComponentNameComponent> = {}
): MountOptionsFn => {
  return (overrides = {}) => {
    return {
      componentProperties: {
        ...initialValues,
        ...overrides
      }
    };
  };
};

const selectByDataTestId = (id: string) => `[data-test-id="${id}"]`;
const container = selectByDataTestId('component-name-container');
const toggleButton = selectByDataTestId('component-name-toggle-button');
const activeState = selectByDataTestId('component-name-active');
const inactiveState = selectByDataTestId('component-name-inactive');

describe(ComponentNameComponent.name, () => {
  let getMountOptions: MountOptionsFn;

  beforeEach(() => {
    getMountOptions = getMountOptionsCurry({
      myInput: 'test value'
    });
  });

  describe('initial state', () => {
    it('should show inactive state by default', () => {
      // given
      cy.mount(ComponentNameComponent, getMountOptions());

      // when
      // then
      cy.get(inactiveState).should('exist');
      cy.get(activeState).should('not.exist');
    });
  });

  describe('toggle interaction', () => {
    it('should show active state after clicking toggle', () => {
      // given
      cy.mount(ComponentNameComponent, getMountOptions());

      // when
      cy.get(toggleButton).click();

      // then
      cy.get(activeState).should('exist');
      cy.get(inactiveState).should('not.exist');
    });
  });
});
```

### Cypress Test Structure

1. **Selectors**: Define `selectByDataTestId` helper and element constants at top
2. **Mount Options**: Use curried function pattern for flexible component mounting
3. **Given/When/Then**: Use comments to structure test logic
4. **Assertions**: Use Cypress chainable assertions (`.should()`)

## When to Use Jest vs Cypress

| Use Jest (*.spec.ts) | Use Cypress (*.cy.ts) |
|---------------------|----------------------|
| Testing computed signals | Testing click interactions |
| Testing method logic | Testing hover behaviors |
| Testing service integration | Testing visual states |
| Testing lifecycle hooks | Testing projected content |
| Mocking dependencies | Testing keyboard navigation |
| Testing error handling | Testing accessibility |

## Exports

Components must be exported from the library's `index.ts`:

```typescript
// src/index.ts
export * from './lib/component-name/component-name.component';
```
