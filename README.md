# Clean Code Compendium

The Clean Code Compendium is an opinionated guide for keeping code clean, readable, and maintainable. It covers a mix of both best practices and code style.

Code style takes inspiration from the [Angular Style Guide](https://angular.io/guide/styleguide). Code styles that are not documented here should consider using the Angular Style Guide as a fallback.

Sections:
- ### [Angular](#angular)
  1. [OnPush Change Detection](#onpush-change-detection)
  1. [Single Responsibility Principle](#single-responsibility-principle)
  1. [Unused `@Input`s/`@Output`s](#unused-inputs-outputs)
  1. [Container vs Presentational Components](#container-vs-presentational-components)
  1. [Method calls in templates](#method-calls-in-templates)
  1. [Observables in templates](#observables-in-templates)
  1. [Complex expressions in templates](#complex-expressions-in-templates)
  1. [Aliases in templates](#aliases-in-templates)
  1. [Placement of `<ng-template>` in templates](#placement-of-ngtemplate-in-templates)
  1. [Naming event handler methods](#naming-event-handler-methods)
  1. [Formatting component properties](#formatting-component-properties)
  1. [Formatting modules](#formatting-modules)
  1. [Formatting properties on elements](#formatting-properties-on-elements)
  1. [Access modifiers](#access-modifiers)
  1. [Providers](#providers)
  1. [Inputs as observables](#inputs-as-observables)
- ### [Redux (NgRx)](#redux)
  1. [Folder structure](#folder-structure)
  1. [Reducers vs Effects](#reducers-vs-effects)
  1. [Redux action strings](#redux-action-strings)
  1. [Service-wrappers for store](#service-wrappers-for-store)
- ### [RxJS](#rxjs)
  1. [Avoid over-using `RxUtils.sync()`](#avoid-overusing-rxutilssync)
  1. [Compose RxJS](#compose-rxjs)
  1. [Long-running mergemap](#longrunning-mergemap)
  1. [Unsubscribing when needed](#unsubscribing-when-needed)
  1. [Unsubscribing when unneeded](#unsubscribing-when-unneeded)
- ### [General](#general)
  1. [Passing arguments](#passing-arguments)
  1. [Nested Ifs](#nested-ifs)
  1. [Comments](#comments)
  1. [Mutating args](#mutating-args)
  1. [Prefixing with underscore](#prefixing-with-underscore)
  1. [Consistent names](#consistent-names)
  1. [Simple methods and functions](#simple-methods-and-functions)
  1. [Formatting chained method calls](#formatting-chained-method-calls)
  1. [Standard methods over lodash](#standard-methods-over-lodash)
  1. [Dead/Unused code](#dead-unused-code)
  1. [Types](#types)
  1. [Unit test method names](#unit-test-method-names)
  1. [Unit test descriptions](#unit-test-descriptions)
  1. [Ordering of properties](#ordering-of-properties)
  1. [Spaces in Arrays/Square brackets](#spaces-in-arrays-square-brackets)
  1. [Spaces between curly braces](#spaces-between-curly-braces)
  1. [Trailing commas](#trailing-commas)
  1. [Structure by feature](#structure-by-feature)
  1. [Formatting constructors](#formatting-constructors)
  1. [Meaningful var and method names](#meaningful-var-and-method-names)
  1. [Naming of observable vars/methods](#naming-of-observable-vars-methods)
  1. [Lengthy if statements](#lengthy-if-statements)
  1. [Variable declarations](#variable-declarations)
  1. [Fat arrow function args](#fat-arrow-function-args)
  1. [Bracket pairs](#bracket-pairs)

---

## Angular <a name="angular"></a>

1. OnPush Change Detection <a name="onpush-change-detection"></a>
    - **Avoid** leaving `changeDetection` undefined in `@Component`
    - **Do** define `ChangeDetectionStrategy.OnPush` explicitly
    - **Why?** Improve performance, minimise CD cycles

    ```ts
    /* Avoid */
    @Component({
      selector: x,
      templateUrl: y,
    })

    /* Do this instead */
    @Component({
      selector: x,
      templateUrl: y,
      changeDetection: ChangeDetectionStrategy.OnPush,
    })
    ```

    

1. Single Responsibility Principle <a name="single-responsibility-principle"></a>
    - **Do** try to keep components focused only on the display-logic e.g. showing/hiding, etc
    - **Do** try to extract business logic out into services
    - **Why?** Keeps classes focused only on their 1 job

    ```ts
    /* Avoid */
    public class MyComponent {
      public isInternalDirectDebit() { }
    }

    <div
      *ngIf="isInternalDirectDebit()">
    </div>

    /* Do this instead */
    public class MyComponent {
      public showSection;

      ngOnInit() {
        this.showSection = this.getShowSection();
      }

      public getShowSection() {
        return this.service.isInternalDirectDebit();
      }
    }

    <div
      *ngIf="showSection">
    </div>
    ```  

    

1. Unused `@Input`s/`@Output`s <a name="unused-inputs-outputs"></a>
    - **Avoid** creating unused `@Input`s/`@Output`s "just in case they may be useful"
    - **Why?** You Ain't Gonna Need It (YAGNI)  
    

1. Container vs Presentational components <a name="container-vs-presentational-components"></a>
    - **Do** allow child/nested components to be container/smart components (e.g. inject services)
    - **Avoid** forcing `@Input`s to be passed down a whole chain of parent/child components
    - **Why?** Prevents duplicate `@Input`s on multiple parent/child components
    - **Why?** Prevents bloating up the 1 top-level container/smart component with vars just purely to pass them down  
    

1. Method calls in templates <a name="method-calls-in-templates"></a>
    - **Avoid** invoking complex methods in template
    - **Do** memoize the method if applicable
    - **Do** use a var to hold the value
    - **Do** initialise the var in `OnInit` and/or update in `OnChanges`
    - **Why?** Methods in template can be invoked as often as every CD cycle
    - **Why?** Prevents invoking the method repeatedly and executing the complex logic repeatedly

    ```ts
    /* Avoid */
    <div>{{ getMeaningOfLife() }}</div>

    /* Do this instead */
    public meaningOfLife

    ngOnChanges() {
      this.meaningOfLife = this.getMeaningOfLife();
    }

    <div>{{ meaningOfLife }}</div>
    ```

    

1. Observables in templates <a name="observables-in-templates"></a>
    - **Avoid** using observables with the async pipe in templates
    - **Avoid** calling methods returning observables in templates
    - **Do** extend `AbstractConnectableComponent`
      - **Do** create a var to hold the latest value of the observable
      - **Do** initialise it in `ngOnInit` using `this.connect()`
    - **Why?** Prevents repeated async pipe subscriptions to the same observable
    - **Why?** Prevents async pipe clutter in the template
    - **Why?** Prevents every CD cycle invoking the method and returning a new observable, and doing a new subscription, etc

    ```ts
    /* Avoid */
    <hello
      [world]="getWorld$()" | async>
    </hello>

    <foo
      [world]="world$" | async>
    </foo>

    /* Do this instead */
    class MyComponent extends AbstractConnectableComponent {
      public world;

      ngOnInit() {
        this.connect({ world: this.getWorld$() });
      }
    }

    <hello
      [world]="world">
    </hello>

    <foo
      [world]="world">
    </foo>
    ```

    

1. Complex expressions in templates <a name="complex-expressions-in-templates"></a>
    - **Avoid** complex expressions in template
    - **Do** use a var to hold the value
    - **Do** initialise the var in `OnInit` and/or update in `OnChanges`
    - **Why?** Improves readability

    ```ts
    /* Avoid */
    <div
      *ngIf="a.b.c === 'hello' || d.e.f === 'world' && g.h.k !== 'foobar'">
    </div>

    /* Do this instead */
    public showSection;

    ngOnChanges() {
      const isHello = a.b.c === 'hello';
      const isWorld = d.e.f === 'world';
      const isNotFoobar = g.h.k !== 'foobar';

      this.showSection = isHello || isWorld && isNotFoobar;
    }

    <div
      *ngIf="showSection">
    </div>
    ```

    

1. Aliases in templates <a name="aliases-in-templates"></a>
    - **Do** alias vars in templates to reference lengthy expressions
    - **Why?** Prevents repetition and improves readability

    ```html
    /* Avoid */
    <div *ngIf="x | async">
      {{ x | async }}
    </div>

    /* Do this instead */
    <div *ngIf="x | async as x">
      {{ x }}
    </div>
    ```

    

1. Placement of `<ng-template>` in templates <a name="placement-of-ngtemplate-in-templates"></a>
    - **Do** put the "real" html at the start of the template file
    - **Do** put `<ng-template>`s at the bottom of the template file
    - **Why?** Easier to understand what the component shows when glancing at start of file

    ```html
    /* Avoid */
    <ng-template></ng-template>

    <my-component></my-component>

    /* Do this instead */
    <my-component></my-component>

    <ng-template></ng-template>
    ```

    

1. Naming event handler methods <a name="naming-event-handler-methods"></a>
    - **Do** name events without the "on" prefix
    - **Do** name event handler methods with the "on" prefix
    - **Do** name event handlers methods after their event
    - **Why?** Makes it obvious which methods handle events

    ```ts
    /* Avoid */
    (onClick)="doThis()"

    /* Do this instead */
    (click)="onClick()"
    (click)="onClickAdd()"
    ```

    

1. Formatting component properties <a name="formatting-component-properties"></a>
    - **Do** group them by
      - public static
      - Inputs
      - Inputs (Setter+Getter pairs)
      - Inputs (multiple decorators)
      - Outputs
      - Other decorators
      - public readonly
      - public
      - protected
      - private
    - **Do** alphabetically order (case-sensitive) them within each group
    - **Why?** Improves readability, prevents duplicates

    ```ts
    // Public static vars
    public static HELLO: any;

    // Inputs: Standard ones first
    @Input public a: string;    // in-lined decorator with var declaration
    @Input public b: string;    // no empty line between declarations
  
    // Inputs: Setters + Getters
    @Input                     // decorator on its own line
    public set c(c: string) {
      this._c = c;
    }                          // no empty line between Setter + Getter pair
    public get c(): string {
      return this._c;
    }                          // empty line around the pair, just like other methods
  
    @Input
    public set d(d: string) {
      this._d = d;
    }
    public get d(): string {
      return this._d;
    }

    // Inputs: Multiple decorators
    @Input
    @ViewChild({ read: SomeDirective })   // each decorator on its own line
    public e: string;
                                          // empty line between declarations
    @Input
    @ContentChild(SomeOtherDirective)
    public f: string;
  
    // Outputs
    @Output public g: EventEmitter<any> = new EventEmitter();
    @Output public h: EventEmitter<any> = new EventEmitter();
  
    // Other decorators in their own groups
    @ContentChild(MyDirective) public i: any;
    @ContentChild(YourDirective) public j: any;
  
    @ViewChild('HisTemplate') public k: any;
    @ViewChild('HerTemplate') public l: any;

    // Public readonly vars
    public readonly WORLD: any;
  
    // Public vars
    public m: string;
    public n: string;

    // Protected static vars
    protected static foo: string;

    // Protected readonly vars
    protected readonly bar: string;

    // Protected vars
    protected o: string;
    protected p: string;

    // Private static vars
    private static cow: string;

    // Private readonly vars
    private readonly moo: string;
  
    // Private vars
    private _c: string;   // underscored vars first
    private _d: string;
    private q: string;
    private r: string;
    ```

    

1. Formatting modules <a name="formatting-modules"></a>
    - **Do** separate 3rd party modules from internal modules
    - **Do** alphabetically order (case-sensitive) modules in each group
    - **Do** alias Angular modules by prefixing with `Ng`
    - **Why?** Prevents duplicates

    ```ts
    /* Avoid */
    imports: [
      ZModule3rdParty,
      AModule,
      NgCommonModule,
      YModule3rdParty,
      BModule,
    ]

    /* Do this instead */
    imports: [
      NgCommonModule,
      YModule3rdParty,
      ZModule3rdParty,

      AModule,
      BModule,
    ],
    declarations: [
      AComponent,
      BComponent,
    ]
    ```

    

1. Formatting properties on elements <a name="formatting-properties-on-elements"></a>
    - **Do** group them into 3 groups:
      - Directives/Attributes
      - Inputs
      - Ouputs
    - **Do** alphabetically order (case-sensitive) them within each group
    - **Do** keep the 1st attribute on the same line as the element
    - **Do** use newlines for other additional attributes
    - **Why?** Improves readability, prevents duplicates

    ```html
    /* Avoid */
    <hello id="" class="" *ngIf="" [abc]="" [def]="" (abc)="" (def)=""></hello>

    /* Do this instead */
    <hello *ngIf=""
      abcDirective
      class=""
      defDirective
      id=""
      [abc]=""
      [def]=""
      (abc)=""
      (def)="">
    </hello>
    ```

1. Access modifiers <a name="access-modifiers"></a>
    - **Do** keep access modifiers to the smallest scope possible
      - `private` by default
      - `protected` if subclasses need access
      - `public` if external classes or template needs access
    - **Why?** Better encapsulation  

1. Providers <a name="providers"></a>
    - **Avoid** adding a class to the `providers` array if it is not being injected as a dependency
    - **Why?** Unnecessary
    - **Why?** Only classes that need to be injected as dependencies should be in `providers`

    ```ts
    /* Avoid */
    providers: [
      InjectableOne,
      InjectableTwo,
      NonInjectableClass
    ]

    /* Do this instead */
    providers: [
      InjectableOne,
      InjectableTwo
    ]
    ```

1. Inputs as observables <a name="inputs-as-observables"></a>
    - **Avoid** having complex `@Input` setters
    - **Avoid** having complex logic inside `ngOnChanges` to react to input value changes
    - **Do** create a private var to hold an observable version of the `@Input`
    - **Do** use `ngOnChanges` to push new values into the var
    - **Why?** Current value of `@Input` is accessible as normal
    - **Why?** Changes to the value of `@Input` can be handled via observables
    - **Why?** Simplifies code in `ngOnChanges`

    ```ts
    /* Avoid */
    @Input public set x(x) { ... }

    ngOnChanges(changes) {
      if (changes.x) { ... }
    }

    /* Do this instead */
    @Input public x;

    private x$: Subject<any>;

    ngOnChanges({ x: xChange }: SimpleChanges) {
      if (xChange) {
        this.x$.next(xChange.currentValue);
      }
    }
    ```

---

## Redux (NgRx) <a name="redux"></a>

1. Folder structure <a name="folder-structure"></a>
    - **Do** create separate files for state, actions, selectors, reducer, effects
    - **Why?** Separation of concerns, easier to understand

    ```ts
    /* Do this */
    x.state.ts     // state interface, initial state obj, entity adapter
    x.actions.ts   // action classes used by reducer, effects
    x.selectors.ts // selectors and pipeable operators for reading a slice of state
    x.reducer.ts   // pure functions for updating the state from a given action
    x.effects.ts   // side effects
    ```

1. Reducers vs Effects <a name="reducers-vs-effects"></a>
    - **Do** use reducer actions to handle logic where possible
    - **Do** use effects when dependency injection (services) or async calls are needed
    - **Why?** Reducer actions are pure functions, easier to test, etc  

1. Redux action strings <a name="redux-action-strings"></a>
    - **Do** define the action string as `[path to value][action][status]`
    - **Why?** Easier to identify what is being updated

    ```ts
    /* Avoid */
    const ADD_VALUATION_SUCCESS = '[valuation][add][valuerValuation][request][success]'

    /* Do this instead */
    const ADD_VALUATION_SUCCESS = '[valuation][valuerValuation][request][add][success]'
    ```

1. Service-wrappers for Store <a name="service-wrappers-for-store"></a>
    - **Avoid** creating services/methods to wrap the store
    - **Do** inject the `store` and use the selector, action directly
    - **Do** create selectors to access the exact slice of state needed
    - **Do** move logic from service methods into effects if appropriate
    - **Why?** Makes actions more self-contained, without relying on service methods performing logic beforehand
    - **Why?** Prevents problems with `combineLatest(service.x$, service.y$)` firing multiple times when the state updates a single time
    - **Why?** Prevents services from bloating up and becoming a dumping ground for methods
    - **Why?** More pure functions (selectors, utils)

    ```ts
    /* Avoid */
    service.getX$();
    service.patchX(x);

    /* Do this instead */
    this.store.select(XSelectors.x);
    this.store.dispatch(new PatchXAction(x));
    ```

---

## RxJS <a name="rxjs"></a>

1. Avoid over-using `RxUtils.sync()` <a name="avoid-overusing-rxutilssync"></a>
    - **Avoid** over-reliance on `RxUtils.sync()`
    - **Do** compose observables instead of using `RxUtils.sync()`
    - **Do** only use it when no longer working with async
    - **Why?** Prevents unnecessary subscriptions

    ```ts
    /* Avoid */
    const x = RxUtils.sync(x$);
    return y$.pipe(map(y => y && x));

    /* Do this instead */
    return y$.pipe(
      withLatestFrom(x$),
      map(([y, x]) => y && x),
    )
    ```

1. Compose RxJS <a name="compose-rxjs"></a>
    - **Avoid** complex logic in `map` or `mergeMap`
    - **Do** break up observables beforehand, and compose them later
    - **Why?** Improves readability

    ```ts
    /* Avoid */
    x$.pipe(
      pairwise(),
      map(([previousX, currentX]) => {
        const previousXProp = previousX.prop;
        const currentXProp = currentX.prop;
        return previousXProp !== currentXProp;
      })
    )

    /* Do this instead */
    const xProp$ = x$.pipe(map(x => x.prop));

    xProp$.pipe(
      pairwise(),
      map(([previousXProp, currentXProp]) => previousXProp !== currentXProp),
    )
    ```

1. Long-running `mergeMap` <a name="longrunning-mergemap"></a>
    - **Do** be careful when using `mergeMap`, as the inner observable will keep running
    - **Do** use `take(1)` or change to `withLatestFrom()` if the inner observable only needs to run once
    - **Why?** Prevents bugs from observables triggering even after initial event is over

    ```ts
    // Beware! Even if x$ only emits once, this will continue emitting when y$ emits
    x$.pipe(
      mergeMap(() => y$)
    )

    // This only takes 1 emit from y$
    x$.pipe(
      mergeMap(() => y$),
      take(1),
    )

    or

    // This only takes the current value in y$
    x$.pipe(
      withLatestFrom(y$),
    )
    ```

1. Unsubscribing when needed <a name="unsubscribing-when-needed"></a>
    - **Do** remember to unsubscribe when done (e.g. `ngOnDestroy`)
    - **Why?** Prevents subscriptions from continuously running

    ```ts
    /* Avoid */
    ngOnInit() {
      x$.subscribe(...)
    }

    /* Do this instead */
    private readonly destroy$: Subject<void> = new Subject();

    ngOnInit() {
      x$
        .pipe(takeUntil(this.destroy$))
        .subscribe(...)
    }

    ngOnDestroy() {
      this.destroy$.next();
    }
    ```

1. Unsubscribing when unneeded <a name="unsubscribing-when-unneeded"></a>
    - **Avoid** unsubscribing when unneeded  
      - e.g. `take(1)`  
      - e.g.  `http` (observables that complete after 1 emit)
    - **Why?** Prevents redundant code

    ```ts
    /* Avoid */
    this.xSubscription = x$.pipe(
      take(1)
    ).subscribe(...)

    this.xSubscription.unsubscribe();

    /* Do this instead */
    this.xSubscription = x$.pipe(
      take(1)
    ).subscribe(...)
    ```

    ```ts
    /* Avoid */
    this.httpSubscription = this.http.get().subscribe();

    this.httpSubscription.unsubscribe();

    /* Do this instead */
    this.http.get().subscribe();
    ```

---

## General <a name="general"></a>

1. Passing arguments <a name="passing-arguments"></a>
    - **Do** pass only what is needed
    - **Avoid** passing unnecessary data where practical
    - **Why?** Easier to read and understand
    - **Why?** Prevents unnecessary referencing

    ```html
    /* Avoid */
    <div (click)="onClick(application)"></div>

    public onClick(application: Application) { }

    /* Do this instead */
    <div (click)="onClick(application.id)"></div>

    public onClick(applicationId: Id) { }
    ```

1. Nested Ifs <a name="nested-ifs"></a>
    - **Avoid** nesting `if` blocks
    - **Do** move "exit" conditions to the start
    - **Why?** Improves readability

    ```ts
    /* Avoid */
    if (x) {
      if (y) {
        this.doWork();
      }
    }

    /* Do this instead */
    if (!x || !y) {
      return;
    }

    this.doWork();
    ```

1. Comments <a name="comments"></a>
    - **Avoid** adding code comments
    - **Do** come up with meaningful var/method names so that they are self-documenting
    - **Do** make an exception for code that may do something unexpected, e.g. code for performance optimisation
    - **Why?** Comments usually become outdated, causing confusion later

    ```ts
    /* Avoid */
    // adds one
    public doWork(x) {
      return x + 1;
    }

    /* Do this instead */
    public addOne(x) {
      return x + 1;
    }
    ```

1. Mutating args <a name="mutating-args"></a>
    - **Avoid** mutating input arguments/parameters
    - **Why?** Prevents consumer of method from having to deal with unexpected behaviour

    ```ts
    /* Avoid */
    public hello(world) {
      world.foobar = 'foobar';
    }

    /* Do this instead */
    public hello(world) {
      return {
        ...world,
        foobar: 'foobar',
      }
    }
    ```

1. Prefixing with underscore <a name="prefixing-with-underscore"></a>
    - **Do** prefix with underscore to prevent name conflicts or to indicate the var is intended to be of local scope
    - **Why** Improves consistency of var names

    ```ts
    /* Avoid */
    public findBunny(bunny) {
      return bunnies.find(rabbit => rabbit === bunny);
    }

    /* Do this instead */
    public findBunny(bunny) {
      return bunnies.find(_bunny => _bunny === bunny);
    }
    ```

1. Consistent names <a name="consistent-names"></a>
    - **Do** keep var names consistent with their type/collection
    - **Why** Improves clarity

    ```ts
    /* Avoid */
    public hello: World;

    /* Do this instead */
    public world: World;
    ```

    ```ts
    /* Avoid */
    rabbits.map(x => ...)

    /* Do this instead */
    rabbits.map(rabbit => ...)
    ```

1. Simple methods and functions <a name="simple-methods-and-functions"></a>
    - **Do** keep methods small in scope/purpose
    - **Avoid** bloating a method up with too much logic
    - **Why?** Improves readability and reuseability

    ```ts
    /* Avoid */
    public blah() {
      if (x < 5) {
        ...
        ...
        ...
      }

      if (x > 10) {
        ...
        ...
        ...
      }
    }

    /* Do this instead */
    public blah() {
      if (x < 5) {
        this.doSomething();
      }

      if (x > 10) {
        this.doSomethingElse();
      }
    }
    ```

1. Formatting chained method calls <a name="formatting-chained-method-calls"></a>
    - **Do** use a single line for single, simple method calls
    - **Do** use a newline each for multiple method calls
    - **Why?** Improves readability

    ```ts
    /* Avoid */
    x.filter(...).map(...).find(...)

    /* Do this instead */
    x
      .filter(...)
      .map(...)
      .find(...)
    ```

    ```ts
    /* Avoid */
    x
      .map(...)

    /* Do this instead */
    x.map(...)
    ```

1. Standard methods over lodash <a name="standard-methods-over-lodash"></a>
    - **Do** prefer standard ES methods instead of lodash unless lodash offers functional benefits
    - **Why?** Prevents inflating bundle size

    ```ts
    /* Avoid */
    find(things, (thing) => ...)

    /* Do this instead */
    things.find(thing => ...)
    ```

1. Dead/Unused code <a name="dead-unused-code"></a>
    - **Avoid** creating unused code "just in case it may be useful"
    - **Why?** You Ain't Gonna Need It (YAGNI)  
    - **Why?** Prevents unnecessary complications, extra unit tests, etc  

1. Types <a name="types"></a>
    - **Do** define the var type
    - **Do** define the return type
    - **Avoid** defining the type if TS can already infer it
    - **Why?** Because. this. is. SPARTAAAA!!!  
    ![](https://media3.giphy.com/media/anu53NkSNV3wI/giphy.gif)  
    (If you don't like using types in Typescript, go back to Javascript)

    ```ts
    /* Avoid */
    public blah(blah) { }

    /* Do this instead */
    public blah(blah: string): void { }
    ```

    ```ts
    public getX(): X { }

    /* Avoid */
    const x: X = this.getX();

    /* Do this instead */
    const x = this.getX();
    ```

1. Unit test method names <a name="unit-test-method-names"></a>
    - **Do** name method test suites with parenthesis
    - **Why?** Easier to identify the method being tested

    ```ts
    /* Avoid */
    describe('methodName', ...);

    /* Do this instead */
    describe('methodName()', ...);
    ```

1. Unit test descriptions <a name="unit-test-descriptions"></a>
    - **Do** use BDD style descriptions
    - **Why?** Easier to read and group related specs

    ```ts
    /* Avoid */
    it('should return x when y', ...);

    /* Do this instead */
    describe('when y', () => {
      it('should return x', ...);
    });
    ```

1. Ordering of properties <a name="ordering-of-properties"></a>
    - **Do** order properties alphabetically (case-sensitive) (per group)
    - **Why?** Prevents conflicts and avoid duplicates

    ```ts
    /* Avoid */
    public c;
    public a;
    public b;

    /* Do this instead */
    public a;
    public b;
    public c;
    ```

1. Spaces in arrays/square brackets <a name="spaces-in-arrays-square-brackets"></a>
    - **Avoid** using a space before/after square brackets
    - **Why?** Improves readability

    ```ts
    /* Avoid */
    [ 1, 2, 3 ]

    /* Do this instead */
    [1, 2, 3]
    ```

1. Spaces between curly braces <a name="spaces-between-curly-braces"></a>
    - **Do** use a separating space before/after curly braces
    - **Why?** Improves readability

    ```ts
    /* Avoid */
    {a: 'hello'}

    /* Do this instead */
    { a: 'hello' }
    ```

1. Trailing commas <a name="trailing-commas"></a>
    - **Do** use trailing commas for objects, arrays
    - **Why?** Prevents merge conflicts

    ```ts
    /* Avoid */
    ['hello', 'world']

    /* Do this instead */
    [
      'hello',
      'world',
    ]
    ```

1. Structure by feature <a name="structure-by-feature"></a>
    - **Do** group files by feature rather than by layer
    - **Why?** Promotes code cohesion

    ```ts
    /* Avoid */
    /reducers
      - abc.reducer
      - def.reducer
    /effects
      - abc.effects
      - def.effects
    /services

    /* Do this instead */
    /abc
      - abc.reducer
      - abc.effects
      - abc.service
    /def
    ```

1. Formatting constructors <a name="formatting-constructors"></a>
    - **Do** use a single-line for single-arg constructors
    - **Do** use newlines  for multi-arg constructors
    - **Do** alphabetically order (case-sensitive) them
    - **Why?** Improves clarity, and prevents duplicates

    ```ts
    /* Avoid */
    constructor(
      a: A,
    )

    /* Do this instead */
    constructor(a: A)
    ```

    ```ts
    /* Avoid */
    constructor(b: B, c: C, a: A)

    /* Do this instead */
    constructor(
      a: A,
      b: B,
      c: C,
    )
    ```

1. Meaningful var and method names <a name="meaningful-var-and-method-names"></a>
    - **Do** use vars and/or methods to provide more context
    - **Do** ensure var/arg names are meaningful
    - **Why?** Adds more meaning or explanation for the var/method

    ```ts
    /* Avoid */
    if (index === ownerships.length -1)

    /* Do this instead */
    const lastIndex = ownerships.length - 1;
    if (index === lastIndex)
    ```

    ```ts
    /* Avoid */
    public onIsValid(event: boolean) { }

    /* Do this instead */
    public onIsValid(isValid: boolean) { }
    ```

1. Naming of observable vars/methods <a name="naming-of-observable-vars-methods"></a>
    - **Do** name vars/methods that hold/return observables with a `$` suffix
    - **Why?** Makes it clear that we are working with async vars

    ```ts
    /* Avoid */
    public hello: Observable<any>;

    /* Do this instead */
    public hello$: Observable<any>;
    ```

    ```ts
    /* Avoid */
    public getWorld(): Observable<any>

    /* Do this instead */
    public getWorld$(): Observable<any>
    ```

1. Lengthy `if` statements <a name="lengthy-if-statements"></a>
    - **Do** use `const`s to break up lengthy expressions in `if` statements
    - **Why?** Improves readability

    ```ts
    /* Avoid */
    if ((this.desiredRepaymentMethodHasChanged(previousProduct, currentProduct) || this.selectedProductHasChanged(previousProduct, currentProduct))
      && (currentProduct.productDefinitionId && this.productsService.isDirectLoanRepayment(currentProduct)))

    /* Do this instead */
    const hasSelectedProductChanged = this.hasSelectedProductChanged(previousProduct, currentProduct);
    const hasDesiredRepaymentMethodChanged = this.hasDesiredRepaymentMethodChanged(previousProduct, currentProduct);
    const hasProductDefinition = !!currentProduct.productDefinitionId;
    const isDirectLoanRepayment = this.productsService.isDirectLoanRepayment(currentProduct);

    if ((hasDesiredRepaymentMethodChanged || hasSelectedProductChanged) && (hasProductDefinition && isDirectLoanRepayment))
    ```

1. Variable declarations <a name="variable-declarations"></a>
    - **Do** prefer to use `const`
    - **Avoid** using `let`
    - **Never** use `var`
    - **Why?** Improves safety and prevents unintended variable reassignment

    ```ts
    /* Avoid */
    var x = 5;
    let y = true;

    /* Do this instead */
    const x = 5;
    const y = true;
    ```

1. Fat arrow function args <a name="fat-arrow-function-args"></a>
    - **Avoid** using parenthesis for single, untyped arg
    - **Do** use parenthesis for single, typed arg (only if type cannot be inferred)
    - **Do** use parenthesis for multiple args
    - **Avoid** using return braces for simple/single-line statements
    - **Do** use return braces for complex/multi-line statements
    - **Why?** Improves readability

    ```ts
    /* Avoid */
    (x) => ...

    /* Do this instead */
    x => ...
    (x: string) => ...
    (x, y) => ...
    ```

    ```ts
    /* Avoid */
    x => {
      return doSomething(x);
    }

    /* Do this instead */
    x => doSomething(x)
    x => {
      const y = ...
      const z = ...

      return ...
    }
    ```

1. Bracket pairs <a name="bracket-pairs"></a>
    - **Do** keep both opening and closing brackets on same line for simple functions
    - **Do** put opening and closing brackets on new lines for complex functions
    - **Why?** Improves readability

    ```ts
    /* Avoid */
    do(x => {
      return 5; });

    /* Do this instead */
    do(x => 5);
    do(x => {
      return 5;
    });
    ```

