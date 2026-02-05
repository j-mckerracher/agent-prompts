  
## Critical Rule: Follow Existing Conventions  
  
**All new code MUST follow the existing conventions in the codebase.**  
  
Before writing any code:  
1. **Examine nearby files** in the same module/feature  
2. **Identify the patterns** used by existing code  
3. **Match those patterns exactly** in your new code  

# IMPORTANT
- **Never introduce new patterns, libraries, or conventions** when existing code already establishes how things should be done.  
- ALWAYS check official documentation to see if the libraries already in use can provide the functionality required by your task.  
  - PrimeNG: https://v19.primeng.org (use search)  
  - PrimeNG Themes: https://primeng.org/theming  
  - PrimeIcons: https://primeng.org/icons  
  - Angular: https://angular.dev  
  - Angular Signals: https://angular.dev/guide/signals  
  - Angular Components: https://angular.dev/essentials/components  
  - Angular Templates: https://angular.dev/essentials/templates  
  - Angular Dependency Injection: https://angular.dev/essentials/dependency-injection  
  - Angular Router: https://angular.dev/guide/routing  
  - Angular Forms (Reactive): https://angular.dev/guide/forms/reactive-forms  
  - Nx: https://nx.dev  
  - Nx Angular Plugin: https://nx.dev/nx-api/angular  
  - NgRx Signals: https://ngrx.io/guide/signals  
  - NgRx Operators: https://ngrx.io/guide/operators  
  - NgRx Store: https://ngrx.io/guide/store  
  - NgRx Store DevTools: https://ngrx.io/guide/store-devtools  
  - RxJS: https://rxjs.dev  
  - Cypress: https://docs.cypress.io  
  - Cypress Component Testing: https://docs.cypress.io/guides/component-testing/overview  
  - Cypress Angular: https://docs.cypress.io/app/component-testing/angular/overview  
  - Jest: https://jestjs.io/docs/getting-started  
  - Jest Preset Angular: https://thymikee.github.io/jest-preset-angular/  
  - TypeScript: https://www.typescriptlang.org/docs/  
  - Storybook: https://storybook.js.org/docs  
  - Storybook for Angular: https://storybook.js.org/docs/get-started/frameworks/angular  
  - date-fns: https://date-fns.org/docs/Getting-Started  
  - date-fns-tz: https://github.com/marnusw/date-fns-tz  
  - Ramda: https://ramdajs.com/docs/  
  - angular-oauth2-oidc: https://github.com/manfredsteyer/angular-oauth2-oidc  
  - angular-svg-icon: https://github.com/czeckd/angular-svg-icon  
  - Quagga2 (Barcode Scanner): https://github.com/ericblade/quagga2  
  - ngx-webcam: https://github.com/basst314/ngx-webcam  
  - xlsx-js-style (Excel Export): https://github.com/gitbrent/xlsx-js-style  
  - Faker.js (Test Data): https://fakerjs.dev/guide/  
  - testdouble (Mocking): https://github.com/testdouble/testdouble.js  
  - ESLint: https://eslint.org/docs/latest/  
  - Angular ESLint: https://github.com/angular-eslint/angular-eslint  
  - Prettier: https://prettier.io/docs/en/index.html  
  - Husky (Git Hooks): https://typicode.github.io/husky/  
  - Mayo Icons: Internal Mayo Clinic library  
  - Mayo ng-config: Internal Mayo Clinic library
 
   
---  
  
## Import Conventions  
  
### Follow Existing Import Order and Style  
  
Before adding imports, check how existing files in the same directory organize them:  
  
```typescript  
// ❌ WRONG: Inventing your own import orderimport { MyService } from './my.service';  
import { Component } from '@angular/core';  
import * as _ from 'lodash';  
  
// ✅ CORRECT: Match the existing pattern in the codebaseimport { Component } from '@angular/core';           // Framework imports first  
import { Observable } from 'rxjs';                   // Third-party next  
import { MyService } from './my.service';            // Local imports last  
```  
  
### Check for:  
- **Import grouping**: Are imports grouped by type (framework, third-party, local)?  
- **Blank lines**: Are there blank lines between import groups?  
- **Path aliases**: Does the project use `@app/`, `@libs/`, or relative paths?  
- **Barrel imports**: Does the project use index.ts barrel files?  
- **Named vs default imports**: Which style is preferred?  
  
---  
  
## Variable Naming Conventions  
  
### Match Existing Naming Patterns  
  
```typescript  
// ❌ WRONG: Using camelCase when codebase uses snake_case (or vice versa)const userData = fetchUser();  
const user_name = userData.name;  // Inconsistent!  
  
// ✅ CORRECT: Use whatever the existing code usesconst userData = fetchUser();     // If codebase uses camelCase  
const userName = userData.name;   // Stay consistent  
```  
  
### Check for:  
- **Variables**: camelCase vs snake_case  
- **Constants**: UPPER_SNAKE_CASE vs camelCase  
- **Private members**: `_prefix` vs `#private` vs no prefix  
- **Observables**: `user$` suffix convention  
- **Boolean variables**: `is`, `has`, `can` prefixes  
- **Event handlers**: `onClick` vs `handleClick` vs `onClickHandler`  
  
---  
  
## Library and Utility Usage  
  
### Use What's Already There  
  
**Never add a new library when an existing one serves the same purpose.**  
  
```typescript  
// ❌ WRONG: Adding lodash when project uses ramdaimport _ from 'lodash';  
const result = _.map(items, fn);  
  
// ✅ CORRECT: Use the library already in the projectimport * as R from 'ramda';  
const result = R.map(fn, items);  
```  
  
### Before Adding Any Dependency:  
1. **Search the codebase** for existing utilities  
2. **Check package.json** for already-installed libraries  
3. **Look for internal utility functions** in `utils/`, `helpers/`, `shared/`  
  
### Common Substitutions to Check:  
| Need | Check if project already uses |  
|------|------------------------------|  
| Date handling | `date-fns`, `dayjs`, `moment`, `luxon` |  
| HTTP requests | `axios`, `fetch`, `HttpClient` |  
| State management | `NgRx`, `Akita`, `Redux`, `MobX` |  
| Form handling | `Reactive Forms`, `Template Forms`, `Formik` |  
| Utility functions | `lodash`, `ramda`, internal utils |  
  
---  
  
## Coding Strategies and Patterns  
  
### Follow Established Architectural Patterns  
  
```typescript  
// ❌ WRONG: Using a different pattern than the codebase// If the codebase uses services for data fetching...  
@Component({...})  
export class MyComponent {  
  async ngOnInit() {    const data = await fetch('/api/data').then(r => r.json());  // Direct fetch!  }}  
  
// ✅ CORRECT: Use the established pattern@Component({...})  
export class MyComponent {  
  constructor(private dataService: DataService) {}    ngOnInit() {  
    this.dataService.getData().subscribe(data => {...});  // Use the service  }}  
```  
  
### Check for These Patterns:  
- **Data fetching**: Services vs direct HTTP vs hooks  
- **Error handling**: Try/catch vs .catch() vs error boundaries  
- **Async patterns**: Promises vs Observables vs async/await  
- **Component structure**: Smart/dumb components, container pattern  
- **State updates**: Immutable vs mutable patterns  
- **Dependency injection**: Constructor injection vs property injection  
  
---  
  
## TypeScript Conventions  
  
### Match Type Definition Styles  
  
```typescript  
// ❌ WRONG: Using interfaces when codebase uses types (or vice versa)interface UserData {  
  name: string;}  
  
// ✅ CORRECT: Match what the codebase usestype UserData = {  
  name: string;};  
```  
  
### Check for:  
- **Interface vs Type**: Which is preferred for object shapes?  
- **Explicit return types**: Are they required on functions?  
- **Type assertions**: `as Type` vs `<Type>` syntax  
- **Null handling**: Optional chaining, nullish coalescing usage  
- **Generics**: Naming conventions (`T`, `TData`, `TProps`)  
- **Enums**: String enums vs const objects vs union types  
  
---  
  
## Testing Conventions  
  
### Match Test Structure and Style  
  
```typescript  
// ❌ WRONG: Using different test structure than existing teststest('should work', () => {...});  
  
// ✅ CORRECT: Match existing test file patternsdescribe('MyComponent', () => {  
  it('should work', () => {...});});  
```  
  
### Check for:  
- **Test file location**: `__tests__/` folder vs `.spec.ts` co-located  
- **Test structure**: `describe/it` vs `test` blocks  
- **Setup patterns**: `beforeEach` vs test-specific setup  
- **Mocking approach**: Jest mocks, testing-library, custom mocks  
- **Assertion style**: `expect().toBe()` vs custom matchers  
  
---  
  
## How to Investigate Existing Conventions  
  
Before writing code, run these checks:  
  
### 1. Find Similar Files  
```bash  
# Find files with similar purpose  
find . -name "*.service.ts" | head -5  
find . -name "*.component.ts" | head -5  
```  
  
### 2. Check Import Patterns  
```bash  
# See how imports are organized  
head -30 src/app/some-existing-file.ts  
```  
  
### 3. Search for Library Usage  
```bash  
# Check which utility library is used  
grep -r "from 'lodash'" src/ | head -3  
grep -r "from 'ramda'" src/ | head -3  
```  
  
### 4. Check Package.json  
```bash  
# See installed dependencies  
cat package.json | grep -A 50 '"dependencies"'  
```  
  
---  
  
## Summary Checklist  
  
Before committing any code, verify:  
  
- [ ] Import order matches existing files  
- [ ] Variable naming matches codebase conventions  
- [ ] No new libraries added when existing ones work  
- [ ] Architectural patterns match (services, state management, etc.)  
- [ ] TypeScript style matches (interfaces vs types, etc.)  
- [ ] Test structure matches existing tests  
- [ ] Error handling follows established patterns  
- [ ] Async patterns match (Observables vs Promises)  
  
**When in doubt, find a similar existing file and mirror its patterns exactly.**