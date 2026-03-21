# mobxjs/mobx[](https://github.com/mobxjs/mobx)

![Diagram 1][ref-1]

This software repository implements the MobX state management library, enabling applications to automatically react to changes in data through an observable programming model. Its primary purpose is to simplify state management by making data changes transparent and automatically propagating them to affected parts of an application. The repository also includes tooling for framework integration, code quality, and version migration.

- **Reactive Core**: The fundamental MobX library makes application state observable. It automatically tracks changes in this state, then triggers derived computations and side effects. This core defines how MobX processes various data structures for reactivity. See [Core MobX Library](#core-mobx-library).
- **UI Integration**: Specialized packages integrate MobX's reactivity with user interface frameworks. These solutions ensure that UI components efficiently re-render when their observed state changes. Dedicated tools support React applications for both class and functional components. Refer to [React Integration with MobX](#react-integration-with-mobx).
- **Development Tooling**: The repository provides tools that enhance developer workflow and enforce coding standards. This includes an ESLint plugin to identify and correct common MobX usage patterns. A migration tool assists in updating older MobX codebases. Explore [MobX ESLint Plugin](#mobx-eslint-plugin) and [MobX Version Migration Tool](#mobx-version-migration-tool).
- **Support Infrastructure**: This covers the internal processes for building and publishing MobX packages across different module formats. It also includes the comprehensive documentation website. This site offers detailed guides and interactive tutorials for users. Learn more in [Build and Publishing Infrastructure](#build-and-publishing-infrastructure) and [Documentation Website](#documentation-website).

## Core MobX Library

![Diagram 2][ref-2]

The MobX library provides a reactive programming model for state management in JavaScript applications. It focuses on making application state observable, allowing for automatic derivation of computed values and execution of side effects in response to state changes. The core functionality is implemented within the [`packages/mobx`][ref-3] directory.

The library's reactivity system is built upon fundamental building blocks that manage observable state, derived values, and side effects. [`Atom`][ref-4] instances ([`packages/mobx/src/core/atom.ts`][ref-5]) serve as the basic observable unit, tracking observers and notifying them of changes. [`ComputedValue`][ref-6] ([`packages/mobx/src/core/computedvalue.ts`][ref-7]) handles the lazy evaluation and caching of derived data, ensuring efficiency by recomputing only when dependencies change. [`Reaction`][ref-8] ([`packages/mobx/src/core/reaction.ts`][ref-9]) instances manage side-effectful computations that automatically re-execute upon changes in their tracked dependencies. The mechanisms for managing observers and propagating changes are centralized within [`packages/mobx/src/core/observable.ts`][ref-10], which defines interfaces for observables and methods for adding/removing observers and controlling update batches. Dependency tracking is managed by [`packages/mobx/src/core/derivation.ts`][ref-11], which also enforces rules for state modification during development.

MobX offers various observable data structures to handle different types of state. Scalar values are managed by [`ObservableValue`][ref-12] ([`packages/mobx/src/types/observablevalue.ts`][ref-13]). Reactive arrays are implemented using [`ObservableArrayAdministration`][ref-14] ([`packages/mobx/src/types/observablearray.ts`][ref-15]), which integrates with [`Proxy`][ref-16] traps for native-like array behavior. [`ObservableMap`][ref-17] ([`packages/mobx/src/types/observablemap.ts`][ref-18]) and [`ObservableSet`][ref-19] ([`packages/mobx/src/types/observableset.ts`][ref-20]) extend native [`Map`][ref-21] and [`Set`][ref-21] to track changes in their entries. For plain JavaScript objects, [`ObservableObjectAdministration`][ref-22] ([`packages/mobx/src/types/observableobject.ts`][ref-23]) handles property-level observability. The [`makeObservable`][ref-24] and [`makeAutoObservable`][ref-25] APIs allow explicit control over which properties become observable.

State modifications are managed through actions, which ensure atomic updates and proper transaction management. [`action`][ref-26] ([`packages/mobx/src/api/action.ts`][ref-27]) wraps functions that modify state, and [`runInAction`][ref-28] executes a function as a temporary action. The [`flow`][ref-29] API ([`packages/mobx/src/api/flow.ts`][ref-30]) enables the creation of cancellable asynchronous operations from generator functions, integrating structured concurrency within the reactive system. These actions utilize internal mechanisms defined in [`packages/mobx/src/core/action.ts`][ref-31] to manage batching, spy reporting, and state change permissions.

MobX provides an annotation and decorator system for declaratively marking class properties as observable, computed, or actions. The [`Annotation`][ref-32] type and related utilities ([`packages/mobx/src/api/annotation.ts`][ref-33], [`packages/mobx/src/api/decorators.ts`][ref-34]) support both legacy and modern Stage 3 decorator syntaxes. Specific annotation types are available for [`action`][ref-26] ([`packages/mobx/src/types/actionannotation.ts`][ref-35]), [`computed`][ref-36] ([`packages/mobx/src/types/computedannotation.ts`][ref-37]), [`flow`][ref-29] ([`packages/mobx/src/types/flowannotation.ts`][ref-38]), [`observable`][ref-39] ([`packages/mobx/src/types/observableannotation.ts`][ref-40]), and [`override`][ref-41] ([`packages/mobx/src/types/overrideannotation.ts`][ref-42]), with [`auto`][ref-43] ([`packages/mobx/src/types/autoannotation.ts`][ref-44]) providing automatic annotation based on property characteristics.

The library includes mechanisms for intercepting and observing state changes. [`intercept`][ref-45] ([`packages/mobx/src/api/intercept.ts`][ref-46]) allows for validation or transformation of proposed changes before they are applied, while [`observe`][ref-47] ([`packages/mobx/src/api/observe.ts`][ref-48]) provides hooks to react to changes after they occur. [`onBecomeObserved`][ref-49] and [`onBecomeUnobserved`][ref-50] ([`packages/mobx/src/api/become-observed.ts`][ref-51]) offer lifecycle hooks when observables gain or lose observers, which can be used for resource management.

For debugging and introspection, MobX offers several utilities. [`spy`][ref-52] ([`packages/mobx/src/core/spy.ts`][ref-53]) dispatches detailed events about all MobX operations, aiding in understanding the flow of reactivity. [`trace`][ref-54] ([`packages/mobx/src/api/trace.ts`][ref-55]) provides detailed logging or breakpoint triggers for derivations. [`getDependencyTree`][ref-56] and [`getObserverTree`][ref-57] ([`packages/mobx/src/api/extras.ts`][ref-58]) help visualize the reactive graph, showing what an observable depends on and what observes it. [`toJS`][ref-59] ([`packages/mobx/src/api/tojs.ts`][ref-60]) converts observable data structures into plain JavaScript objects for debugging or serialization.

MobX's internal global state is managed by [`MobXGlobals`][ref-61] ([`packages/mobx/src/core/globalstate.ts`][ref-62]), which centralizes runtime parameters and supports [`isolateGlobalState`][ref-63] for testing and complex application setups, as detailed in tests within [`packages/mobx/__tests__/mixed-versions`][ref-64]. Global behavior can be configured using the [`configure`][ref-65] API ([`packages/mobx/src/api/configure.ts`][ref-66]).

Utility functions within [`packages/mobx/src/utils`][ref-67] support core operations. This includes various equality [`comparer`][ref-68]s ([`packages/mobx/src/utils/comparer.ts`][ref-69]) like [`identity`][ref-70], [`structural`][ref-71], and [`shallow`][ref-72], as well as [`deepEqual`][ref-73] ([`packages/mobx/src/utils/eq.ts`][ref-74]) for robust deep equality checks, which are critical for optimizing re-computations and reactions. Error messages are managed in [`packages/mobx/src/errors.ts`][ref-75], providing detailed messages in development and minified codes in production. Flow type definitions for the library can be found in [`packages/mobx/flow-typed/mobx.js`][ref-76]. For server-side rendering specifics, refer to [Server-Side Rendering (SSR) Considerations](#react-integration-with-mobx-server-side-rendering-ssr-considerations).

### Core Reactive Primitives

![Diagram 3][ref-77]

The foundational building blocks of MobX's reactivity system are centered around three core components: [`Atom`][ref-4], [`ComputedValue`][ref-6], and [`Reaction`][ref-8]. These elements work in concert to track observable state, manage derived values, and execute side effects efficiently.

The [`Atom`][ref-4] in [`packages/mobx/src/core/atom.ts`][ref-5] serves as the most basic observable unit. It tracks observers and notifies them when its underlying state changes. This mechanism allows MobX to establish a dependency graph where an [`Atom`][ref-4] reports when it has been accessed (observed) and when its value has changed. Callbacks can be registered with an atom to execute when it becomes observed or unobserved, providing hooks for resource management.

Building upon atoms, the [`ComputedValue`][ref-6] in [`packages/mobx/src/core/computedvalue.ts`][ref-7] represents derived state. A computed value automatically re-evaluates when any of its dependencies (other observables, including atoms or other computed values) change. It employs lazy evaluation and caching, meaning its derivation function runs only when its value is requested and its dependencies have been identified as stale. This ensures that computations are performed only when necessary, optimizing performance. [`ComputedValue`][ref-6] also handles error propagation and supports custom equality comparers to prevent unnecessary updates if a newly computed value is structurally identical to its previous one.

[`Reaction`][ref-8] in [`packages/mobx/src/core/reaction.ts`][ref-9] provides a mechanism for side effects that need to respond to observable state changes. Unlike computed values, reactions do not produce a value themselves but instead execute a side-effecting function whenever their observed dependencies change. This is typically used for tasks like updating the UI, logging, or making network requests. Reactions manage their own lifecycle, including scheduling their execution, tracking dependencies, and disposing of themselves when no longer needed. MobX employs a batching system to ensure that multiple changes within a short period trigger only one reaction execution, further optimizing performance. The system also includes safeguards to detect and prevent infinite loops in reactive computations.

Collectively, these primitives define how MobX establishes and maintains the relationships between observable state, derived data, and side effects, forming the basis for its reactive programming model. Further details on how these primitives manage their dependencies and coordinate updates can be found in the configuration options discussed in [Global State Management and Configuration](#core-mobx-library-global-state-management-and-configuration). The [`packages/mobx/src/core/derivation.ts`][ref-11] file outlines the states of derivations, enabling optimizations such as [`UP_TO_DATE_`][ref-78], [`POSSIBLY_STALE_`][ref-79], and [`STALE_`][ref-80] to avoid redundant computations. The [`packages/mobx/src/core/observable.ts`][ref-10] file provides the core lifecycle and dependency tracking, including observer registration, batching updates, and propagation of changes across the system.

### Observable Data Structures

Data Structure Type

Description

File Path

`ObservableValue`

Represents a single observable value of type `T`.

`packages/mobx/src/types/observablevalue.ts`

`IObservableArray`

An observable array, providing methods to observe and react to changes in its elements and structure.

`packages/mobx/src/types/observablearray.ts`

`LegacyObservableArray`

An observable array implementation for environments that do not support proxies (older JavaScript engines).

`packages/mobx/src/types/legacyobservablearray.ts`

`ObservableMap`

An observable Map, allowing observation of key-value pairs.

`packages/mobx/src/types/observablemap.ts`

`ObservableSet`

An observable Set, allowing observation of its elements.

`packages/mobx/src/types/observableset.ts`

`ObservableObjectAdministration`

Manages the observable properties of an object, handling additions, updates, and deletions.

`packages/mobx/src/types/observableobject.ts`

MobX provides a set of observable data structures that are central to its reactive programming model. These structures wrap standard JavaScript data types, allowing MobX to automatically track their changes and trigger reactions.

For scalar values, MobX uses [`ObservableValue`][ref-12], defined in [`packages/mobx/src/types/observablevalue.ts`][ref-13]. This class encapsulates a single value and integrates it with MobX's core dependency tracking system, enabling observation, interception of mutations, and integration with developer tools. When a value within an [`ObservableValue`][ref-12] instance is accessed, MobX records the observer, and when the value changes, it notifies all registered observers, ensuring that computed values and reactions are re-evaluated.

Observable arrays, central to managing collections, are primarily handled by the [`ObservableArrayAdministration`][ref-14] class. This administrative class, detailed in [`packages/mobx/src/types/observablearray.ts`][ref-15], uses JavaScript [`Proxy`][ref-16] objects to intercept array accesses and mutations. It manages the array's internal state, tracks changes, and integrates with the MobX reaction system, ensuring that standard array methods like [`push`][ref-81], [`pop`][ref-82], and [`splice`][ref-83] correctly trigger reactivity. For environments with specific prototype inheritance bugs, MobX includes [`LegacyObservableArray`][ref-84] in [`packages/mobx/src/types/legacyobservablearray.ts`][ref-85], which extends [`Array.prototype`][ref-86] to provide similar reactive behavior while accommodating older browser quirks.

For key-value pair collections, MobX offers [`ObservableMap`][ref-17], described in [`packages/mobx/src/types/observablemap.ts`][ref-18]. This implementation extends the standard JavaScript [`Map`][ref-21] to track changes to its entries. Operations such as [`set`][ref-87], [`get`][ref-88], and [`delete`][ref-89] on an [`ObservableMap`][ref-17] automatically trigger reactions. It also includes mechanisms for intercepting changes before they occur and observing them after they are applied, as well as tracking observations of map keys, ensuring reactions when keys are added or removed.

Similarly, for unique value collections, MobX provides [`ObservableSet`][ref-19], found in [`packages/mobx/src/types/observableset.ts`][ref-20]. This class wraps a native JavaScript [`Set`][ref-21], augmenting it with MobX reactivity. It allows for observation and interception of additions, deletions, and clearing of elements, integrating these mutations into the MobX reactivity system.

Plain JavaScript objects and class instances are made observable through the [`ObservableObjectAdministration`][ref-22] class, defined in [`packages/mobx/src/types/observableobject.ts`][ref-23]. This class acts as an administrative layer that manages the reactivity of an object's properties. It handles property access, modification, and change notifications, tracking whether properties are observable, computed, or non-observable. It also supports dynamic definition of properties and manages the reactivity of object keys, ensuring observers react correctly when properties are added or deleted. Together, these observable data structures form the foundation of MobX's efficient and automatic reactivity.

### Annotations and Decorators System

![Diagram 4][ref-90]

MobX's annotation system allows developers to explicitly mark properties within classes or objects as observable state, computed derivations, or actions, thereby integrating them into the reactive programming paradigm. This system is crucial for [`makeObservable`][ref-24], MobX's primary API for converting plain JavaScript objects into observable ones, by providing the necessary metadata for each property.

The core of this system is the [`Annotation`][ref-32] type, defined in [`packages/mobx/src/api/annotation.ts`][ref-33], which outlines the behavior for different types of observable properties. Each annotation, such as those for [`action`][ref-26], [`computed`][ref-36], [`flow`][ref-29], [`observable`][ref-39], [`auto`][ref-43], and [`override`][ref-41], implements methods that dictate how MobX should process a property during its lifecycle, from initial definition to extensions.

MobX uses a flexible decorator system, managed in [`packages/mobx/src/api/decorators.ts`][ref-34], that supports both legacy and the newer TC39 Stage 3 decorator syntaxes. When a decorator is applied, a [`createDecoratorAnnotation`][ref-91] function captures the associated [`Annotation`][ref-32] and stores it on the class prototype. This metadata is then used by [`makeObservable`][ref-24] to properly configure the observable object during instantiation.

For example, an [`action`][ref-26] annotation, created by [`createActionAnnotation`][ref-92] in [`packages/mobx/src/types/actionannotation.ts`][ref-35], transforms a class method into a MobX action, optionally binding [`this`][ref-93] to the instance. Similarly, [`computed`][ref-36] annotations, defined by [`createComputedAnnotation`][ref-94] in [`packages/mobx/src/types/computedannotation.ts`][ref-37], mark a getter as a derived value that automatically reacts to changes in its dependencies. The [`flow`][ref-29] annotation, detailed in [`packages/mobx/src/types/flowannotation.ts`][ref-38], enables generator functions to be treated as cancellable asynchronous operations within the reactive system.

The [`observable`][ref-39] annotation, originating from [`createObservableAnnotation`][ref-95] in [`packages/mobx/src/types/observableannotation.ts`][ref-40], is responsible for converting properties into observable values. For advanced scenarios, the [`autoAnnotation`][ref-96], found in [`packages/mobx/src/types/autoannotation.ts`][ref-44], automatically infers the most appropriate MobX annotation based on the property's characteristics (e.g., a getter becomes computed, a function becomes an action, and a simple field becomes observable). This allows for a more concise syntax when defining observable classes. Finally, the [`override`][ref-41] annotation, from [`packages/mobx/src/types/overrideannotation.ts`][ref-42], acts as a safeguard, ensuring that a decorated member genuinely overrides an annotated member from a parent class, promoting robustness in class hierarchies.

### Asynchronous Flows

```typescript
function* fetchUser(userId: string) {
    console.log("Fetching user...");
    const user = yield fetch(`/api/users/${userId}`).then(response => response.json());
    console.log("User fetched:", user.name);
    const details = yield fetch(`/api/user-details/${user.id}`).then(response => response.json());
    console.log("User details fetched.");
    return { user, details };
}

// Convert the generator function into a MobX flow
const userFlow = flow(fetchUser);

async function runFlow() {
    console.log("Starting flow...");
    const promise = userFlow("123"); // Initiate the flow

    // To demonstrate cancellation (uncomment to test):
    // setTimeout(() => {
    //     promise.cancel(); // Cancel the flow after some time
    //     console.log("Flow cancelled!");
    // }, 100);

    try {
        const result = await promise;
        console.log("Flow completed:", result);
    } catch (error) {
        if (isFlowCancellationError(error)) {
            console.log("Flow caught cancellation error.");
        } else {
            console.error("Flow failed:", error);
        }
    }
}

// Stub out isFlowCancellationError for demonstration.
// In a real MobX application, this would be imported from MobX.
function isFlowCancellationError(error: any): boolean {
    return error && error.message === "FLOW_CANCELLED";
}

runFlow();
```

MobX provides a mechanism for managing asynchronous operations through its [`flow`][ref-29] API, which converts generator functions into cancellable promises. This allows for structured concurrency within reactive applications, where asynchronous tasks can be initiated, managed, and explicitly terminated.

The [`flow`][ref-29] function processes a generator function, returning a wrapper that, when called, executes the generator. During execution, each [`yield`][ref-97] expression within the generator is handled as a [`Promise`][ref-98] resolution, allowing MobX to track and react to state changes occurring throughout the asynchronous process. To ensure proper integration with MobX's reactive system, each step of the generator's execution (initialization, yielding, and error handling) is wrapped in a MobX [`action`][ref-26], guaranteeing transactional updates and preventing unexpected reactions during intermediate states.

A key feature of flows is their cancellability. The promise returned by a flow includes a [`cancel()`][ref-99] method. Invoking this method attempts to terminate the ongoing generator and rejects the flow's promise with a [`FlowCancellationError`][ref-100]. This provides a clear signal that the asynchronous operation was intentionally halted. The [`isFlowCancellationError`][ref-101] utility function helps in identifying this specific type of error for appropriate handling.

[`flow`][ref-29] can also be used as an annotation for class methods, particularly [`flow.bound`][ref-102], which automatically binds the method to the class instance, simplifying its use within class-based components. Type helpers like [`flowResult`][ref-103] assist in correctly inferring the return types of flows, enhancing type safety.

For debugging and understanding the reactive relationships within flows, MobX's introspection tools can be utilized. See [Debugging and Introspection](#core-mobx-library-debugging-and-introspection) for more details. The underlying reactive primitives, such as [`Atom`][ref-4] and [`Reaction`][ref-8], play a fundamental role in how flows integrate with MobX's core tracking mechanisms. See [Core Reactive Primitives](#core-mobx-library-core-reactive-primitives) for further explanation.

### Interception and Observation

![Diagram 5][ref-104]

The [`intercept`][ref-45] and [`observe`][ref-47] mechanisms provide a powerful set of hooks for interacting with MobX's reactive system, allowing for validation, transformation, and side effects before state changes are finalized or in response to them. These mechanisms allow for fine-grained control over how observable state evolves and how external systems react to those changes.

The [`intercept`][ref-45] function, defined in [`packages/mobx/src/api/intercept.ts`][ref-46], enables the registration of interceptors that run *before* a proposed change is applied to an observable. These interceptors can examine the change, modify it, or even prevent it from occurring by returning [`null`][ref-105]. This is useful for implementing validation rules, ensuring data integrity, or transforming data before it's stored in the observable state. The system supports intercepting changes to individual observable values, array mutations, map and set alterations, and property changes on observable objects. The core logic for handling interceptors is managed by [`interceptChange`][ref-106] in [`packages/mobx/src/types/intercept-utils.ts`][ref-107], which iterates through registered interceptors, allowing each to process the change.

Complementary to [`intercept`][ref-45] is [`interceptReads`][ref-108] from [`packages/mobx/src/api/intercept-read.ts`][ref-109], which allows for experimental modification or transformation of observable values *before* they are read. This can be used for purposes such as data anonymization or on-the-fly formatting of data.

In contrast, the [`observe`][ref-47] function, detailed in [`packages/mobx/src/api/observe.ts`][ref-48], registers listeners that react *after* a change has been applied to an observable. These listeners are invoked with details of the change, enabling side effects such as logging, synchronizing with external systems, or updating UI components. [`observe`][ref-47] is highly polymorphic, supporting various observable types including values, arrays, maps, sets, and properties of objects. The management of these change listeners is handled by utilities in [`packages/mobx/src/types/listen-utils.ts`][ref-110], which provide functions to register, deregister, and notify listeners efficiently, ensuring that listener execution does not inadvertently trigger further reactive dependencies.

Beyond direct observation of state changes, MobX also provides hooks to react to the observation status of observables themselves. The [`onBecomeObserved`][ref-49] and [`onBecomeUnobserved`][ref-50] functions, located in [`packages/mobx/src/api/become-observed.ts`][ref-51], allow callbacks to be registered that execute when an observable gains or loses active observers. This mechanism is crucial for resource optimization, as it enables lazy initialization or teardown of expensive computations or external connections, ensuring that resources are only consumed when genuinely needed by the reactive system.

### Debugging and Introspection

Utility Name

Primary Function

Key Features

Relevant File

`spy`

Intercepts all MobX events for debugging.

Provides a callback for every MobX event, including actions, reactions, and observable changes.

`mobx/src/core/spy.ts`

`toJS`

Converts observable data structures into plain JavaScript counterparts.

Deeply converts observable arrays, objects, maps, and sets into their non-observable equivalents; ignores non-observable properties.

`mobx/src/api/tojs.ts`

`getDependencyTree`

Visualizes the observable dependencies of a reactive component.

Returns a tree structure showing which observables a given reactive component (e.g., computed value, reaction) is observing.

`mobx/src/api/extras.ts`

`getObserverTree`

Visualizes which reactive components are observing a given observable.

Returns a tree structure showing all reactive components (e.g., reactions, computed values) that depend on a specific observable.

`mobx/src/api/extras.ts`

`trace`

Enables detailed logging or sets breakpoints for derivations.

Can log all MobX events related to a specific derivation or trigger a debugger breakpoint when the derivation re-runs.

`mobx/src/api/trace.ts`

MobX provides a suite of debugging and introspection utilities to understand and analyze the reactive behavior of applications. The [`spy`][ref-52] mechanism, defined in [`packages/mobx/src/core/spy.ts`][ref-53], allows for comprehensive observation of all MobX events, including actions, reactions, and changes to observable state. This is useful for integrating with developer tools, logging, or custom analytics. The [`spy`][ref-52] function enables registration of listeners that receive detailed event objects, providing insights into the MobX runtime.

For targeted debugging of reactive computations, the [`trace`][ref-54] function, implemented in [`packages/mobx/src/api/trace.ts`][ref-55], enables developers to monitor specific computed values or reactions. It can either log activity to the console or trigger a JavaScript debugger breakpoint whenever the derivation re-evaluates, offering a granular view into why and when reactive parts of the application are executing.

To visualize the reactive graph, MobX offers functions to inspect dependencies and observers. The [`packages/mobx/src/api/extras.ts`][ref-58] file defines [`getDependencyTree`][ref-56] and [`getObserverTree`][ref-57]. [`getDependencyTree`][ref-56] constructs a tree showing what other reactive values a specific observable or computed value relies on. Conversely, [`getObserverTree`][ref-57] reveals which other reactive components are observing a given observable, providing insight into the flow of data and reactive updates.

Finally, [`toJS`][ref-59], located in [`packages/mobx/src/api/tojs.ts`][ref-60], is a utility for converting observable data structures into plain JavaScript counterparts. This deep transformation unwraps observable arrays, sets, maps, and objects into their non-observable forms, which is particularly useful for debugging, serialization, or interoperability with libraries that do not inherently understand MobX observables. This function also handles circular references to prevent infinite loops during conversion.

### Global State Management and Configuration

![Diagram 6][ref-111]

MobX manages its internal global state through the [`MobXGlobals`][ref-61] class, which serves as a central repository for the reactive system's parameters, tracking information, and configuration in [`packages/mobx/src/core/globalstate.ts`][ref-62]. This class ensures a consistent environment for MobX operations, encompassing everything from tracking the currently active derivation to managing scheduled reactions and batching levels. The [`globalState`][ref-112] instance, initialized from [`MobXGlobals`][ref-61], attempts to integrate with any existing MobX global state if multiple MobX installations are present in the same environment. This allows for compatibility or graceful coexistence.

The [`configure`][ref-65] function in [`packages/mobx/src/api/configure.ts`][ref-66] provides a unified API for modifying MobX's global behavior. It allows developers to specify options such as [`useProxies`][ref-113] for observable creation, [`enforceActions`][ref-114] for strict state modification rules, and various flags for error handling and development-time checks. The [`configure`][ref-65] function interacts with the [`globalState`][ref-112] object, offering a controlled way to adjust MobX's runtime characteristics without directly manipulating internal state.

MobX supports scenarios where multiple instances of the library might be loaded within an application. By default, these instances share global state, allowing observables and reactions from different instances to interact. However, MobX also provides mechanisms to isolate global state for specific instances, preventing unintended cross-instance reactivity. This isolation is crucial for testing environments or complex applications where distinct MobX contexts are required. The tests in [`packages/mobx/__tests__/mixed-versions`][ref-64] demonstrate both the shared global state behavior and the effects of [`isolateGlobalState`][ref-63] on reactivity and warning suppression when multiple MobX instances are active.

### Utility Functions and Comparers

Function/Object Name

Purpose/Description

Source File

`comparer.identity`

Compares if two values are strictly identical (`a === b`).

`comparer.ts`

`comparer.structural`

Performs a deep structural comparison of two values.

`comparer.ts`

`comparer.shallow`

Performs a shallow structural comparison of two values (one level deep).

`comparer.ts`

`comparer.default`

Compares two values using `Object.is` if available, otherwise falls back to strict equality with special handling for `0` and `NaN`.

`comparer.ts`

`deepEqual`

A utility function for deep structural equality checking.

`eq.ts`

`assign`, `getDescriptor`, `defineProperty`, `objectPrototype`

Core JavaScript `Object` utilities for property manipulation.

`utils.ts`

`isFunction`, `isString`, `isObject`, `isPlainObject`, `isES6Map`, `isES6Set`

A collection of type-checking utility functions.

`utils.ts`

`addHiddenProp`, `addHiddenFinalProp`

Functions to add non-enumerable properties to objects.

`utils.ts`

`hasProp`

Checks if an object has a specified property directly on itself.

`utils.ts`

`getGlobal`

Returns the global object, handling different JavaScript environments.

`global.ts`

MobX incorporates a suite of utility functions to support its reactivity system, including various mechanisms for equality comparison, object manipulation, type checking, and robust global object access.

Core to MobX's operation are its equality comparers, which determine when a change to observable state should trigger reactions. The [`comparer`][ref-68] object, defined in [`packages/mobx/src/utils/comparer.ts`][ref-69], provides several comparison strategies. [`identityComparer`][ref-115] performs a strict equality check ([`===`][ref-116]). For more complex data structures, [`structuralComparer`][ref-117] conducts a deep equality comparison, while [`shallowComparer`][ref-118] performs a shallow comparison, checking only top-level properties. These deeper comparisons leverage the [`deepEqual`][ref-73] function, found in [`packages/mobx/src/utils/eq.ts`][ref-74], which can handle primitives, objects, arrays, and various MobX observable types and ES6 collections. This function is designed to prevent infinite loops by detecting cyclic references and supports a configurable depth limit for comparisons. The [`defaultComparer`][ref-119] offers a robust comparison using [`Object.is`][ref-120] with fallbacks for older environments.

Beyond comparison, MobX includes general utility functions in [`packages/mobx/src/utils/utils.ts`][ref-121] for internal operations. These include functions for object property manipulation, such as [`addHiddenProp`][ref-122] for adding non-enumerable properties, and [`getOwnPropertyDescriptors`][ref-123] for comprehensive property enumeration. The file also provides various type-checking predicates like [`isFunction`][ref-124], [`isObject`][ref-125], [`isES6Map`][ref-126], and [`isPlainObject`][ref-127], along with [`getNextId`][ref-128] for generating unique identifiers. Assertions and warnings related to the availability of [`Proxy`][ref-16] objects are also managed here, ensuring that MobX operates correctly in different JavaScript environments and configurations.

Accessing the global object uniformly across diverse JavaScript environments (browsers, Node.js, Web Workers) is facilitated by the [`getGlobal`][ref-129] function in [`packages/mobx/src/utils/global.ts`][ref-130]. This function prioritizes [`globalThis`][ref-131], falling back to [`window`][ref-132], [`global`][ref-133], or [`self`][ref-134], and finally to an empty mock object to ensure consistent global access.

## React Integration with MobX

![Diagram 7][ref-135]

MobX provides integration with React applications to enable reactive rendering, ensuring that the user interface automatically updates in response to changes in observable state. This integration includes utilities, Higher-Order Components (HOCs), and components tailored for both React DOM and React Native environments. A key aspect of this integration is the [`mobx-react-lite`][ref-136] package, which offers a lightweight solution optimized for functional React components, emphasizing performance and efficient observation cleanup. The integration facilitates the provisioning and injection of MobX stores, allowing components to consume state seamlessly. It also addresses considerations for server-side rendering (SSR) to prevent memory leaks and maintain predictable rendering behavior.

### Core MobX-React Integration (mobx-react)

![Diagram 8][ref-137]

The [`mobx-react`][ref-138] package facilitates the integration of MobX state management with React applications, encompassing both React DOM and React Native environments. It provides components, Higher-Order Components (HOCs), and utilities to enable reactive rendering and manage the lifecycle of reactive components.

Core to [`mobx-react`][ref-138] is the [`observer`][ref-139] function, defined in [`packages/mobx-react/src/observer.tsx`][ref-140]. This function transforms a React component into a reactive one, ensuring that it automatically re-renders when any observed MobX state changes. For class components, [`observer`][ref-139] modifies the component's prototype, such as overriding [`shouldComponentUpdate`][ref-141] to align with MobX's reaction system, and extending lifecycle methods like [`componentDidMount`][ref-142] and [`componentWillUnmount`][ref-143] for proper reactivity setup and cleanup. This management of class components occurs within [`packages/mobx-react/src/observerClass.ts`][ref-144]. The implementation ensures that React components respond efficiently to granular state changes, optimizing re-renders by only updating components whose observed data has changed.

For sharing MobX stores across the React component tree, [`mobx-react`][ref-138] provides the [`Provider`][ref-145] component and the [`inject`][ref-146] HOC. The [`Provider`][ref-145], defined in [`packages/mobx-react/src/Provider.tsx`][ref-147], utilizes React's Context API to make MobX stores available to all descendant components. It supports hierarchical store merging and includes development-only checks to ensure the immutability of the set of provided stores, preventing unexpected behavior from dynamic store changes. The [`inject`][ref-146] HOC, located in [`packages/mobx-react/src/inject.ts`][ref-148], then consumes these stores, making them available as props to the wrapped component. This decoupling allows components to receive necessary data without directly managing context consumption, promoting cleaner component interfaces. While [`Provider`][ref-145] and [`inject`][ref-146] are robust, [`React.createContext`][ref-149] is recommended for new projects.

The package also offers utilities for managing the lifecycle of MobX reactions within React components. The [`disposeOnUnmount`][ref-150] utility, found in [`packages/mobx-react/src/disposeOnUnmount.ts`][ref-151], allows developers to automatically execute disposer functions (e.g., from [`reaction`][ref-152] or [`autorun`][ref-153]) when a class component unmounts. This prevents memory leaks by cleaning up MobX subscriptions, though it is marked as deprecated for React versions 18 and higher due to compatibility issues. For Server-Side Rendering (SSR) scenarios, [`mobx-react`][ref-138] includes mechanisms like [`enableStaticRendering(true)`][ref-154] to prevent memory leaks by ensuring observer components do not attempt to react to future data changes when rendered on the server. This setting is crucial for maintaining predictable behavior and preventing resource exhaustion in SSR environments.

Additionally, [`mobx-react`][ref-138] provides custom [`PropTypes`][ref-155] in [`packages/mobx-react/src/propTypes.ts`][ref-156] for validating MobX observable data structures such as observable arrays, objects, and maps. These help ensure type correctness and provide clearer debugging information during development. The package also incorporates utility functions in [`packages/mobx-react/src/utils/utils.ts`][ref-157] for tasks such as shallow object comparison, property manipulation, and a method patching mechanism that supports mixins for React lifecycle methods. These utilities underpin the core functionalities by ensuring efficient updates and flexible component integration.

For a lightweight integration focused on functional components and hooks, refer to [Lightweight React Integration (mobx-react-lite)](#react-integration-with-mobx-lightweight-react-integration-mobx-react-lite).

#### Reactive Class Components and `disposeOnUnmount`

![Diagram 9][ref-158]

MobX provides mechanisms to integrate its reactivity system with React class components, ensuring that these components automatically re-render when the observable data they depend on changes. The [`observer`][ref-139] utility, found in [`packages/mobx-react/src/observer.tsx`][ref-140] and implemented for class components in [`packages/mobx-react/src/observerClass.ts`][ref-144], transforms a standard React class component into a reactive one. This transformation involves wrapping the component's [`render`][ref-159] method in a MobX [`Reaction`][ref-8] to track observable accesses and trigger updates.

A key aspect of this integration is how MobX handles React's lifecycle methods. When [`observer`][ref-139] is applied to a class component, it modifies the component's [`shouldComponentUpdate`][ref-141] method. For components that are not [`PureComponent`][ref-160], MobX ensures that updates are primarily driven by its reaction system. This effectively disallows custom [`shouldComponentUpdate`][ref-141] implementations in observable components to prevent conflicts with MobX's update logic, thereby centralizing rendering decisions.

The [`observer`][ref-139] also augments [`componentDidMount`][ref-142] and [`componentWillUnmount`][ref-143]. During mounting, it establishes the connection between the MobX reaction and the component's [`forceUpdate`][ref-161] method, allowing MobX to directly initiate re-renders. During unmounting, it disposes of the MobX reaction to prevent memory leaks and ensure proper cleanup. This lifecycle management is crucial for maintaining performance and stability in reactive applications.

For managing cleanup operations, particularly MobX reactions, the [`disposeOnUnmount`][ref-150] utility, located at [`packages/mobx-react/src/disposeOnUnmount.ts`][ref-151], was historically available. This utility patched the component's [`componentWillUnmount`][ref-143] method to automatically execute disposer functions or methods. However, due to incompatibilities with modern React versions, especially React 18+, [`disposeOnUnmount`][ref-150] has been deprecated. Its usage now triggers warnings or errors, signaling a shift towards alternative patterns for resource management in class components.

#### Store Provisioning with `Provider` and `inject`

![Diagram 10][ref-162]

The [`mobx-react`][ref-138] package provides mechanisms for making MobX stores accessible throughout a React component tree. The [`Provider`][ref-145] component and the [`inject`][ref-146] Higher-Order Component (HOC) facilitate this process by leveraging React's Context API.

The [`Provider`][ref-145] component, defined in [`packages/mobx-react/src/Provider.tsx`][ref-147], establishes a React Context ([`MobXProviderContext`][ref-163]) to hold MobX stores. When a [`Provider`][ref-145] is rendered, it makes the stores passed to it as props available to all its descendant components. This [`Provider`][ref-145] can also inherit stores from an ancestor [`Provider`][ref-145], allowing for a hierarchical merging of store sets. During development, the [`Provider`][ref-145] includes a check to ensure that the set of stores provided to it remains stable across renders, which helps prevent potential issues arising from dynamic store changes.

To consume these stores, components can utilize the [`inject`][ref-146] HOC, located in [`packages/mobx-react/src/inject.ts`][ref-148]. The [`inject`][ref-146] HOC wraps a React component and passes specified MobX stores as props. There are two primary ways to use [`inject`][ref-146]:

1. **By store names**: Developers can provide a list of string names corresponding to the desired stores. [`inject`][ref-146] will retrieve these stores from the [`MobXProviderContext`][ref-163] and pass them as props.
2. **With a custom mapping function**: A function can be provided to [`inject`][ref-146] that receives all available MobX stores and the component's current props. This function then returns an object of props to be merged into the component, offering greater flexibility in how stores are consumed.

The [`inject`][ref-146] HOC uses [`React.forwardRef`][ref-164] to ensure that refs are correctly passed through to the wrapped component, maintaining full React compatibility. It also offers optional reactivity: when a custom mapping function is used, the component is automatically made reactive via [`observer`][ref-139], assuming the function may derive observable data. When injecting by name, developers explicitly apply [`observer`][ref-139] if reactivity is needed. The [`inject`][ref-146] mechanism also copies static properties from the original component to the injected component, preserving [`propTypes`][ref-165] or other static members.

#### Custom PropTypes for MobX Observables

```tsx
// MyComponent.tsx
import React from 'react';
import { PropTypes as MobxPropTypes } from 'mobx-react'; // Assuming this import path

interface MyComponentProps {
  myObservableArray: any[];
  myRequiredObservableObject: object;
  myObservableMap?: Map;
  myObservableArrayOfNumbers: number[];
  myFlexibleArray: any[];
}

const MyComponent: React.FC = () => {
  return {/* Component content */};
};

MyComponent.propTypes = {
  // Checks if the prop is a MobX observable array
  myObservableArray: MobxPropTypes.observableArray,
  // Checks for a MobX observable object and makes it required
  myRequiredObservableObject: MobxPropTypes.observableObject.isRequired,
  // Checks if the prop is a MobX observable map (optional)
  myObservableMap: MobxPropTypes.observableMap,
  // Checks for a MobX observable array where all elements are numbers
  myObservableArrayOfNumbers: MobxPropTypes.observableArrayOf(MobxPropTypes.number),
  // Checks for a regular array OR a MobX observable array
  myFlexibleArray: MobxPropTypes.arrayOrObservableArray,
};

export default MyComponent;
```

The [`mobx-react`][ref-138] package offers custom React PropTypes for validating MobX observable data structures within React components. These PropTypes extend React's validation system to accommodate MobX-specific types, such as observable arrays, objects, and maps, aiding in the early detection of prop type mismatches during development.

The core of this functionality involves a set of type checkers designed to ascertain if a prop is an observable type. For instance, validators are provided for [`observableArray`][ref-166], [`observableObject`][ref-167], and [`observableMap`][ref-168]. The implementation largely mirrors React's internal [`PropTypes`][ref-155] structure, enabling the use of [`.isRequired`][ref-169] to designate mandatory observable props. To prevent unnecessary reactive updates during validation, the [`untracked`][ref-170] utility from MobX is employed, ensuring that prop type checks do not trigger MobX reactions.

In addition to basic observable type validation, these custom PropTypes support more specific scenarios. For example, [`observableArrayOf`][ref-171] validates observable arrays where all elements must conform to a specified type checker. Furthermore, combined validators such as [`arrayOrObservableArray`][ref-172] and [`objectOrObservableObject`][ref-173] allow props to accept either a native JavaScript type or its MobX observable equivalent. The design of these validators, including the [`createChainableTypeChecker`][ref-174] pattern, allows for flexibility and robust error reporting, as detailed in [`packages/mobx-react/src/propTypes.ts`][ref-156].

### Lightweight React Integration (mobx-react-lite)

![Diagram 11][ref-175]

The [`mobx-react-lite`][ref-136] package offers an optimized integration of MobX with React, specifically designed for functional components and the React Hooks API. It aims to provide efficient reactive rendering and streamlined local state management with a smaller bundle size compared to [`mobx-react`][ref-138]. The library ensures that React components automatically re-render when the MobX observable data they consume changes, facilitating a reactive programming model within the React ecosystem.

The core of [`mobx-react-lite`][ref-136] lies in its ability to make functional components reactive. The [`observer`][ref-139] higher-order component transforms a functional React component into one that automatically tracks MobX observables used during its render process. This ensures that the component re-renders precisely when those observed values change, optimizing performance by avoiding unnecessary updates. Similarly, the [](https://github.com/mobxjs/mobx/tree/main/docs/react-integration.md#L290)component, described in [`packages/mobx-react-lite/src/ObserverComponent.ts`][ref-176], allows for fine-grained reactivity within any component, including class components, by wrapping specific portions of the UI that depend on observable state. These mechanisms are underpinned by the [`useObserver`][ref-177] hook, detailed in [`packages/mobx-react-lite/src/useObserver.ts`][ref-178], which bridges MobX's reaction system with React's lifecycle, particularly leveraging [`useSyncExternalStore`][ref-179] for efficient subscription management and clean updates.

For managing local observable state within functional components, [`mobx-react-lite`][ref-136] provides the [`useLocalObservable`][ref-180] hook, as implemented in [`packages/mobx-react-lite/src/useLocalObservable.ts`][ref-181]. This hook enables the creation of stable, local MobX observable stores that persist across component re-renders. It simplifies the process of defining reactive properties, computed values, and actions directly within a component's scope, promoting a clean and encapsulated state management pattern. Older hooks such as [`useLocalStore`][ref-182] and [`useAsObservableSource`][ref-183], discussed in [`packages/mobx-react-lite/src/useLocalStore.ts`][ref-184] and [`packages/mobx-react-lite/src/useAsObservableSource.ts`][ref-185] respectively, are considered deprecated in favor of [`useLocalObservable`][ref-180] and standard React patterns using [`useEffect`][ref-186] for synchronizing non-observable values.

Performance and resource management are key considerations. [`mobx-react-lite`][ref-136] configures MobX's reaction scheduler to integrate with React's batching mechanism through [`observerBatching`][ref-187], found in [`packages/mobx-react-lite/src/utils/observerBatching.ts`][ref-188]. This ensures that multiple MobX updates are grouped together into a single React render cycle, minimizing UI thrashing. Furthermore, the library employs a sophisticated cleanup mechanism to prevent memory leaks, especially crucial in scenarios like React's [`StrictMode`][ref-189] or concurrent rendering. This mechanism utilizes a [`UniversalFinalizationRegistry`][ref-190], as detailed in [`packages/mobx-react-lite/src/utils/UniversalFinalizationRegistry.ts`][ref-191], or a timer-based fallback to automatically dispose of MobX [`Reaction`][ref-8] instances when observer components are garbage collected. The [`clearTimers`][ref-192] function, exported in [`packages/mobx-react-lite/src/index.ts`][ref-193], also provides a way to explicitly clean up internal timers, which is particularly useful in testing environments.

Server-Side Rendering (SSR) is supported with the [`enableStaticRendering`][ref-194] function, located in [`packages/mobx-react-lite/src/staticRendering.ts`][ref-195]. When activated, this mode ensures that [`observer`][ref-139] components render once and then clean up their MobX reactions, preventing memory leaks and maintaining predictable behavior in SSR environments. The overall architecture of [`mobx-react-lite`][ref-136] prioritizes a lightweight, hooks-centric approach to MobX integration, encouraging modern React practices and optimized performance.

#### Reactive Functional Components and Hooks

![Diagram 12][ref-196]

The primary mechanism for integrating MobX's reactive state with React functional components in [`mobx-react-lite`][ref-136] is the [`observer`][ref-139] higher-order component. The [`observer`][ref-139] function, found in [`packages/mobx-react-lite/src/observer.ts`][ref-197], transforms a standard functional React component into a reactive one. This means the component will automatically re-render whenever any MobX observable data, accessed during its rendering, undergoes a change. For performance optimization, [`observer`][ref-139] automatically applies [`memo`][ref-198] to the wrapped component, preventing unnecessary re-renders when its props remain unchanged. It also intelligently handles [`React.forwardRef`][ref-164] components, ensuring that ref forwarding is preserved while adding reactivity.

For more fine-grained control over reactivity within a component's render method, or when props cannot be passed directly, the [`Observer`][ref-199] component (defined in [`packages/mobx-react-lite/src/ObserverComponent.ts`][ref-176]) can be used. This component takes a render function as its [`children`][ref-200] or [`render`][ref-159] prop. It internally leverages the [`useObserver`][ref-177] hook to establish a reactive context, causing only the content within the [`Observer`][ref-199] component to re-render when its observed data changes, rather than the entire parent component.

At the core of this reactive integration for functional components is the [`useObserver`][ref-177] hook, located in [`packages/mobx-react-lite/src/useObserver.ts`][ref-178]. This hook bridges MobX's reactivity system with React's lifecycle by using [`useSyncExternalStore`][ref-179]. When a component utilizing [`useObserver`][ref-177] renders, a MobX [`Reaction`][ref-8] is established. This [`Reaction`][ref-8] monitors all MobX observables accessed during that render. If any of these observables change, the [`Reaction`][ref-8] signals [`useSyncExternalStore`][ref-179] that the external state has been updated, prompting React to potentially re-render the component. This design ensures efficient subscription management and alignment with React's concurrent rendering capabilities. To prevent memory leaks, particularly in strict mode or during concurrent rendering, [`useObserver`][ref-177] employs a [`FinalizationRegistry`][ref-201] (described in [Efficient Reaction Cleanup with FinalizationRegistry](#react-integration-with-mobx-lightweight-react-integration-mobx-react-lite-efficient-reaction-cleanup-with-finalizationregistry)) to automatically dispose of the MobX [`Reaction`][ref-8] instances when component instances are garbage collected.

#### Local Observable State with `useLocalObservable`

```tsx
// Example of a simple component using useLocalObservable
function Timer() {
    // Creates a local observable store that persists across re-renders.
    // The initializer function runs only once.
    const timer = useLocalObservable(() => ({
        secondsPassed: 0,
        increase() {
            this.secondsPassed++
        },
        reset() {
            this.secondsPassed = 0
        },
    }))

    // Renders the current state and provides actions to modify it.
    return (
        
            Seconds passed: {timer.secondsPassed}
            Increase
            Reset
        
    )
}
```

The [`useLocalObservable`][ref-180] hook, defined in [`packages/mobx-react-lite/src/useLocalObservable.ts`][ref-181], facilitates the creation and management of stable, local MobX observable state directly within functional React components. This hook is designed to ensure that an observable store persists across component re-renders, providing a consistent state management mechanism without the need for external store definitions.

The [`useLocalObservable`][ref-180] hook accepts an initializer function, which defines the initial state of the observable store, and optionally, MobX [`annotations`][ref-202] to specify observable behaviors for properties within that store. Internally, React's [`useState`][ref-203] hook is leveraged to manage the lifecycle of the MobX observable, ensuring it is initialized only once and retained throughout the component's existence. A key design choice within the hook's implementation is the use of [`autoBind: true`][ref-204] when creating the observable, which automatically binds methods defined within the store to its instance, simplifying their usage within React components.

While [`useLocalStore`][ref-182] in [`packages/mobx-react-lite/src/useLocalStore.ts`][ref-184] provides similar functionality, it is considered deprecated in favor of [`useLocalObservable`][ref-180]. Similarly, the [`useAsObservableSource`][ref-183] hook in [`packages/mobx-react-lite/src/useAsObservableSource.ts`][ref-185] is also deprecated, having previously allowed the conversion of a plain object into a MobX observable for local component state. The current recommendation for managing observable state is to use [`useLocalObservable`][ref-180] combined with [`useEffect`][ref-186] for syncing external updates, as it promotes a more efficient and modern approach to state management within functional components.

#### Efficient Reaction Cleanup with FinalizationRegistry

![Diagram 13][ref-205]

The [`mobx-react-lite`][ref-136] package ensures that MobX reactions are properly disposed of when a React component that observes them unmounts or is garbage collected, preventing memory leaks. This is achieved through an instance of [`UniversalFinalizationRegistry`][ref-190] located in [`packages/mobx-react-lite/src/utils/observerFinalizationRegistry.ts`][ref-206]. This registry automatically triggers a cleanup function when an object associated with a MobX reaction is no longer referenced. The cleanup function calls the [`dispose`][ref-207] method on the associated MobX [`Reaction`][ref-8] instance, effectively unsubscribing it and releasing its resources.

The [`UniversalFinalizationRegistry`][ref-190] itself provides a robust mechanism for tracking object finalization across different JavaScript environments. As detailed in [Efficient Reaction Cleanup with FinalizationRegistry](#react-integration-with-mobx-lightweight-react-integration-mobx-react-lite-efficient-reaction-cleanup-with-finalizationregistry), it leverages the native [`FinalizationRegistry`][ref-201] when available. For environments that lack native support, it gracefully falls back to a timer-based simulation to approximate object finalization. This dual approach, implemented in [`packages/mobx-react-lite/src/utils/UniversalFinalizationRegistry.ts`][ref-191], ensures consistent and efficient reaction cleanup regardless of the runtime environment, enhancing the stability and performance of MobX-React applications.

### Server-Side Rendering (SSR) Considerations

```typescript
// server.ts (or equivalent server-side rendering entry point)
import { enableStaticRendering } from 'mobx-react-lite';
import React from 'react';
import ReactDOMServer from 'react-dom/server';
// Assume your App component is defined elsewhere
import { App } from './App'; // Stub: Replace with your actual App component import

// Enable static rendering for SSR
enableStaticRendering(true);

const htmlContent = ReactDOMServer.renderToString();

// At this point, `isUsingStaticRendering()` would return true within components
// rendered by ReactDOMServer.

// Example of a simple observer component (for context)
// This component would not react to state changes if rendered after enableStaticRendering(true)
// and before a potential disableStaticRendering(false) on the client.

// import { observer } from 'mobx-react-lite';
// import { makeAutoObservable } from 'mobx';

// class Store {
//   count = 0;
//   constructor() {
//     makeAutoObservable(this);
//   }
// }

// const myStore = new Store();

// const MyObserverComponent = observer(() => {
//   return Count: {myStore.count};
// });

// Note: On the client, you would typically call enableStaticRendering(false)
// or ensure it's not called at all, to allow reactivity.
```

When performing Server-Side Rendering (SSR), [`mobx-react`][ref-138] and [`mobx-react-lite`][ref-136] include mechanisms to manage reactivity and prevent memory leaks. The function [`enableStaticRendering`][ref-194] from [`packages/mobx-react-lite/src/staticRendering.ts`][ref-195] is crucial in this context. Calling [`enableStaticRendering(true)`][ref-154] during SSR ensures that [`observer`][ref-139] components do not attempt to react to future data changes. This behavior is important because, in an SSR environment, components are rendered once to generate static HTML and are not expected to be interactive or undergo subsequent re-renders on the server. Disabling reactivity through this mechanism prevents the creation of unnecessary subscriptions and subsequent memory overhead, thereby ensuring predictable rendering and efficient resource utilization on the server. The [`README.md`][ref-208] for [`mobx-react`][ref-138] also highlights the importance of this utility for SSR.

## MobX ESLint Plugin

```javascript
// Before ESLint fix: Component 'MyComponent' is missing 'observer'.
function MyComponent() {
  // ... component logic using MobX observables
}

// After ESLint fix:
const MyComponent = observer(function MyComponent() {
  // ... component logic using MobX observables
});

// Another example: Arrow function component
const AnotherComponent = () => {
  // ...
};

// After ESLint fix:
const AnotherComponent = observer(() => {
  // ...
});

// Example with a class component
class ClassComponent extends React.Component {
  // ...
}

// After ESLint fix:
const ClassComponent = observer(class ClassComponent extends React.Component {
  // ...
});
```

The MobX ESLint Plugin ([`eslint-plugin-mobx`][ref-209]) provides a suite of linting rules designed to enforce best practices and identify common pitfalls when integrating MobX into applications. This plugin helps developers maintain code quality and consistency by guiding them in the correct usage of MobX's reactive features within JavaScript and TypeScript codebases. The plugin's implementation details, including its rules and configurations, are centralized in the [`packages/eslint-plugin-mobx`][ref-210] directory.

The plugin focuses on key areas of MobX usage, particularly around the [`makeObservable`][ref-24] function and the [`observer`][ref-139] higher-order component (HOC) in React integration. It ensures that MobX's core mechanisms for creating observable state and reacting to changes are applied correctly. For instance, rules prevent common mistakes such as neglecting to wrap React components that consume MobX observables with the [`observer`][ref-139] HOC or improperly configuring [`makeObservable`][ref-24] calls within classes. This systematic enforcement of patterns helps prevent subtle bugs related to reactivity and performance that might otherwise be difficult to detect.

The plugin provides flexible configuration options, including recommended rule sets for quick setup and granular control over individual rules. It supports both legacy ESLint configurations and the newer flat configuration format, accommodating various project setups. Detailed information on installation and configuration can be found in [`packages/eslint-plugin-mobx/README.md`][ref-211]. The plugin also includes extensive testing to ensure rule accuracy across different ESLint versions, as documented in its [`CHANGELOG.md`][ref-212] file, which tracks updates, bug fixes, and new features for the [`eslint-plugin-mobx`][ref-209] package.

### Core ESLint Rules

Filename

Description

Autofix

`exhaustive-make-observable.js`

Enforces all fields are listed in `makeObservable` calls within constructors or object expressions.

Yes

`missing-make-observable.js`

Prevents missing `makeObservable(this)` when using MobX decorators in class components.

Yes

`unconditional-make-observable.js`

Disallows conditional calls to `makeObservable` or `makeAutoObservable(this)` inside constructors.

No

`missing-observer.js`

Prevents missing `observer` wrapper on React components.

Yes

`no-anonymous-observer.js`

Forbids anonymous functions or classes as `observer` components.

Yes

The primary MobX ESLint rules are designed to enforce consistent and correct usage of MobX within JavaScript and TypeScript applications. These rules identify common pitfalls and provide autofixes to streamline development. The plugin's entry point is defined in [`packages/eslint-plugin-mobx/src/index.js`][ref-213], which aggregates and exposes the individual rules. A common utility module, [`packages/eslint-plugin-mobx/src/utils.js`][ref-214], provides helper functions for AST traversal and MobX decorator identification, which are leveraged by multiple rules.

One rule, implemented in [`packages/eslint-plugin-mobx/src/exhaustive-make-observable.js`][ref-215], ensures that all observable fields within a class or object are explicitly listed in [`makeObservable`][ref-24] calls. It reports fields that are part of the class or object structure but are not included in the [`makeObservable`][ref-24] configuration, and can automatically add these missing annotations. This helps prevent silent failures where properties intended to be observable are not tracked by MobX.

Another rule, defined in [`packages/eslint-plugin-mobx/src/missing-make-observable.js`][ref-216], checks for the presence of [`makeObservable(this)`][ref-217] in class constructors when MobX decorators are used. If a class utilizes MobX decorators but lacks this essential call, the rule flags it and can automatically insert [`makeObservable(this)`][ref-217] into the constructor, ensuring proper initialization of observability. It also validates that the [`makeObservable`][ref-24] call does not include a second argument, as this would be redundant when decorators are used.

To prevent issues arising from delayed or conditional observability setup, the rule in [`packages/eslint-plugin-mobx/src/unconditional-make-observable.js`][ref-218] enforces that [`makeObservable`][ref-24] or [`makeAutoObservable`][ref-25] calls with [`this`][ref-93] as the first argument must occur unconditionally within a class constructor. This prevents scenarios where observability might only be enabled under certain runtime conditions, leading to unpredictable behavior.

For React components, the rule in [`packages/eslint-plugin-mobx/src/missing-observer.js`][ref-219] identifies functional or class components that interact with MobX state but are not wrapped by the [`observer`][ref-139] higher-order component. This rule helps ensure that components re-render reactively when their observable dependencies change, and it provides an autofix to automatically apply the [`observer`][ref-139] wrapper.

Finally, [`packages/eslint-plugin-mobx/src/no-anonymous-observer.js`][ref-220] addresses debugging and maintainability concerns by disallowing anonymous functions or classes as arguments to the [`observer`][ref-139] HOC. It reports these anonymous components and offers an autofix to assign a name, typically derived from the variable identifier, improving component identification in debugging tools and error messages.

### Testing and Preview Environment

![Diagram 14][ref-221]

The MobX ESLint plugin leverages a comprehensive testing suite and a preview environment to validate its rules and showcase their application. The testing methodology primarily utilizes [`RuleTester`][ref-222] for ensuring each ESLint rule behaves as expected across different ESLint versions. This involves defining [`valid`][ref-223] and [`invalid`][ref-224] code examples for each rule, along with anticipated errors and potential auto-fixes. For instance, the [`exhaustive-make-observable`][ref-225] rule is tested to confirm that all observable properties, computed values, and actions within MobX classes are correctly registered via [`makeObservable`][ref-24]. Similarly, [`missing-make-observable`][ref-226] verifies that classes using MobX decorators include a [`makeObservable(this)`][ref-217] call in their constructors, while [`unconditional-make-observable`][ref-227] ensures these calls are not made conditionally. The [`missing-observer`][ref-228] rule identifies React components that use MobX observables but lack an [`observer`][ref-139] wrapper, and [`no-anonymous-observer`][ref-229] promotes better debugging by requiring [`observer`][ref-139]-wrapped components to have names.

A dedicated utility, [`getRuleTester`][ref-230], found in [`packages/eslint-plugin-mobx/__tests__/utils/get-rule-tester.js`][ref-231], is instrumental in configuring the [`RuleTester`][ref-222] for different ESLint versions, such as [`7`][ref-232] and [`9`][ref-233], by dynamically adjusting the parser configuration to use [`@typescript-eslint/parser`][ref-234].

Beyond automated testing, a preview environment located in [`packages/eslint-plugin-mobx/preview`][ref-235] serves as a demonstration and testbed. This environment includes an ESLint configuration file, [`packages/eslint-plugin-mobx/preview/.eslintrc.js`][ref-236], that extends the recommended MobX rules. It contains various code examples like [`packages/eslint-plugin-mobx/preview/make-observable.js`][ref-237], [`packages/eslint-plugin-mobx/preview/missing-observer.js`][ref-238], and [`packages/eslint-plugin-mobx/preview/no-anonymous-observer.js`][ref-239]. These files illustrate different usage patterns of [`makeObservable`][ref-24] and [`makeAutoObservable`][ref-25], component declarations with and without [`observer`][ref-139] wrappers, and examples of anonymous [`observer`][ref-139] components, thereby showcasing how the ESLint rules identify and address common MobX usage issues. A [`node_modules`][ref-240] workaround in [`packages/eslint-plugin-mobx/preview/node_modules/eslint-plugin-mobx`][ref-241] enables ESLint to properly load the plugin for these preview scenarios.

## MobX Version Migration Tool

```typescript
// Before: MobX 4/5 with decorators
class MyStore {
    @observable value = 0;
    @action increment() {
        this.value++;
    }
}

// After: MobX 6 with makeObservable in constructor
class MyStore {
    value = 0;

    constructor() {
        // mobx-undecorate transformed decorators to makeObservable
        makeObservable(this, {
            value: observable,
            increment: action
        });
    }

    increment() {
        this.value++;
    }
}
```

The [`mobx-undecorate`][ref-242] tool assists in migrating MobX 4/5 codebases to MobX 6 conformance, transforming deprecated decorator syntax and [`decorate`][ref-243] calls into the newer [`makeObservable`][ref-24] or functional wrapper patterns. This allows developers to update their projects to leverage MobX 6's API, which emphasizes explicit state declaration over implicit decorator usage.

The tool primarily focuses on converting decorator-based syntax, such as [`@observable`][ref-244], [`@action`][ref-245], and [`@computed`][ref-246], into explicit [`makeObservable`][ref-24] calls within class constructors. For [`mobx-react`][ref-138] specific decorators like [`@observer`][ref-247] and [`@inject`][ref-248], it refactors them into functional wrappers, aligning with modern React and MobX integration patterns. This refactoring helps to standardize the declaration of observable state and reactive entities, making code more predictable and easier to maintain.

The migration process is driven by [`jscodeshift`][ref-249], a tool for running codemods, which enables Abstract Syntax Tree (AST) manipulation to perform these transformations. This approach allows for automated and consistent updates across a codebase, reducing manual effort and potential errors during the migration. Users can execute the transformation via a command-line interface, which handles the execution of [`jscodeshift`][ref-249] with the necessary transformation scripts.

For more detailed information on the transformation logic and how it interacts with [`jscodeshift`][ref-249], refer to [Core Transformation Logic and jscodeshift](#mobx-version-migration-tool-core-transformation-logic-and-jscodeshift). The command-line options available for customizing the migration behavior are discussed in [Command-Line Interface and Options](#mobx-version-migration-tool-command-line-interface-and-options).

### Core Transformation Logic and jscodeshift

![Diagram 15][ref-250]

The [`mobx-undecorate`][ref-242] tool leverages [`jscodeshift`][ref-249] to perform Abstract Syntax Tree (AST) manipulation, enabling the automatic refactoring of MobX codebases from decorator-based syntax to the newer [`makeObservable`][ref-24] and functional wrapper patterns. This process is crucial for migrating MobX 4/5 projects to MobX 6 conformance, addressing changes in how observable state, computed values, and actions are defined.

The core of this transformation resides in the [`transform`][ref-251] function within [`packages/mobx-undecorate/src/undecorate.ts`][ref-252], which processes individual source files. It begins by configuring a robust parser, [`@babel/parser`][ref-253] (babylon), to handle a wide array of modern JavaScript and TypeScript features, including various decorator syntaxes, class properties, and JSX. This flexible parsing ensures that the tool can correctly interpret diverse code styles and project configurations.

A primary function of the tool is the conversion of MobX decorators. For class properties and methods, decorators like [`@action`][ref-245], [`@observable`][ref-244], and [`@computed`][ref-246] are identified and their information extracted. This extracted metadata is then used to construct an [`ObjectExpression`][ref-254] that defines the members for a [`makeObservable`][ref-24] call. If configured, the original decorators are removed from the source code. For class decorators such as [`@observer`][ref-247] and [`@inject`][ref-248], the tool doesn't inject them into [`makeObservable`][ref-24]. Instead, it wraps the entire class declaration or its default export with [`observer()`][ref-255] or [`inject()`][ref-256] function calls, maintaining the functional pattern favored in modern React.

A critical aspect of the transformation is the management of class constructors. The tool either creates a new constructor or modifies an existing one to insert the [`makeObservable`][ref-24] call. This call is strategically placed within the constructor body, typically after any [`super()`][ref-257] call if the class extends another. The [`ObjectExpression`][ref-254] formed from the extracted decorator information is passed as the second argument to [`makeObservable`][ref-24], precisely defining which class members should be made observable, acted upon, or computed. The system also handles [`private`][ref-258] and [`protected`][ref-259] class members, ensuring they are correctly typed within the [`makeObservable`][ref-24] call. Special considerations are made for React components ([`Component`][ref-260], [`PureComponent`][ref-160]) to ensure [`props`][ref-261] are correctly passed to [`super()`][ref-257].

Beyond decorator handling, the tool actively manages MobX imports. It identifies existing imports and updates them to include [`makeObservable`][ref-24] and [`override`][ref-41] where necessary, while also removing [`decorate`][ref-243] imports if they are no longer used after the transformation. This comprehensive approach ensures that the refactored codebase remains syntactically correct and aligns with MobX 6's API.

### Command-Line Interface and Options

![Diagram 16][ref-262]

The [`mobx-undecorate`][ref-242] package provides a command-line interface (CLI) to automate the migration of MobX 4/5 codebases to MobX 6 conformance. This CLI leverages the [`jscodeshift`][ref-249] toolkit for abstract syntax tree (AST) manipulation, allowing it to systematically transform deprecated decorator syntax and [`decorate`][ref-243] calls into the newer [`makeObservable`][ref-24] or functional wrapper patterns.

The primary entry point for the CLI is [`packages/mobx-undecorate/cli.js`][ref-263], which orchestrates the execution of the core transformation logic found in [`packages/mobx-undecorate/src/undecorate.ts`][ref-252]. The CLI is responsible for locating the [`jscodeshift`][ref-249] executable and passing the transformation script along with user-defined arguments to it.

Users can run the migration tool using [`npx mobx-undecorate`][ref-264]. The tool parses command-line arguments to determine the target directory for the transformation and to configure specific aspects of the migration process. Key options available through the CLI include:

- [`--ignoreImports`][ref-265]: This option instructs the tool to bypass checks for existing MobX import statements. When enabled, the tool proceeds to convert all instances of MobX-related decorators, such as [`@observable`][ref-244], [`@action`][ref-245], [`@computed`][ref-246], [`@observer`][ref-247], and [`@inject`][ref-248], without first verifying their import status.
- [`--keepDecorators`][ref-266]: By default, the tool replaces MobX decorators with [`makeObservable`][ref-24] calls and removes the original decorators. However, if a project intends to continue using decorators with MobX 6, this flag ensures that the decorators are retained while still injecting the necessary [`makeObservable`][ref-24] calls into the class constructors.
- [`--decoratorsAfterExport`][ref-267]: This option addresses potential [`SyntaxError`][ref-267] issues that can arise in Babel configurations where [`decoratorsBeforeExport`][ref-268] is set to [`false`][ref-269]. It ensures that decorators are correctly processed when they are positioned after the [`export`][ref-270] keyword in class definitions.
- [`--parseTsAsNonJsx`][ref-271]: For TypeScript files, this flag instructs the parser to treat [`.ts`][ref-272] files as non-JSX, which can be useful in scenarios where a [`.ts`][ref-272] file might contain syntax that could otherwise be misinterpreted as JSX, leading to parsing errors.

The CLI ensures that [`jscodeshift`][ref-249] is invoked with the appropriate arguments and the correct transformation script ([`packages/mobx-undecorate/src/undecorate.ts`][ref-252]) to perform the code modifications. This process includes converting decorator-based syntax to explicit [`makeObservable`][ref-24] calls within class constructors, and refactoring [`mobx-react`][ref-138]'s [`@observer`][ref-247] decorator into functional [`observer`][ref-139] wrappers. For a deeper understanding of the transformation logic, refer to [Core Transformation Logic and jscodeshift](#mobx-version-migration-tool-core-transformation-logic-and-jscodeshift).

### Testing Suite for Transformations

Category

Scenario

Description

**CLI Integration Tests**

Basic `@observable` transformation

Verifies CLI processes a file with `@observable` decorator, converting it to `makeObservable` setup.

`--parseTsAsNonJsx` option

Tests CLI's ability to correctly parse `.ts` files as non-JSX and `.tsx` as JSX, transforming decorators accordingly.

**Unit Tests: Decorator Transformations**

`@observable` (basic, shallow, computed name)

Covers transformation of `@observable` and `@observable.shallow` decorators on class fields, including those with computed names.

`@action` (field/method, bound/unbound, named/unnamed, computed name)

Details various `@action` decorator transformations for fields and methods, considering bound/unbound and named/unnamed variations, and computed names.

`@computed` (basic, with setter, options, struct)

Explores `@computed` decorator transformations for getters, including scenarios with setters, configuration options, and `.struct`.

`override` decorator

Ensures correct handling and transformation of the `override` decorator, especially when `keepDecorators` is used.

**Unit Tests: `decorate` Calls**

Basic `decorate` call transformation

Tests conversion of `decorate` calls into `makeObservable` within a class constructor.

Multiple `decorate` targets

Verifies handling of multiple `decorate` calls within the same scope.

Undecleared members in `decorate`

Checks that `makeObservable` correctly includes members from `decorate` calls even if not explicitly declared in the class.

Non-class objects with `decorate`

Confirms `decorate` calls on plain objects are transformed to `makeObservable` for those objects.

**Unit Tests: Private Fields**

`makeObservable` with private/protected fields

Tests the generation of generic arguments for `makeObservable` to include private and protected class fields.

**Unit Tests: `@observer` Transformations**

Class components with `@observer`

Covers transformation of `@observer` decorator on class components, including export statements and combinations with other decorators like `inject` and `withRouter`.

**Unit Tests: Syntax Preservation**

Various TypeScript/JSX syntaxes

Ensures `mobx-undecorate` preserves other language features like named tuples, async/generator functions, and JSX elements during transformation.

The [`mobx-undecorate`][ref-242] package includes a comprehensive testing suite to validate its transformation logic and command-line interface (CLI). This suite consists of both unit and integration tests, ensuring that the migration tool accurately converts MobX decorators and [`decorate`][ref-243] calls to the [`makeObservable`][ref-24] API, and transforms [`mobx-react`][ref-138]'s [`@observer`][ref-247] class decorators into functional [`observer`][ref-139] wrappers.

Unit tests, located in [`packages/mobx-undecorate/__tests__/undecorate.spec.ts`][ref-273], rigorously verify the core [`jscodeshift`][ref-249] transform. These tests cover a wide array of scenarios, including:

- **Decorator Transformations:** Validation of [`@observable`][ref-244], [`@action`][ref-245], and [`@computed`][ref-246] decorators for class fields, methods, getters, and setters. This includes handling various arguments (e.g., [`action.bound`][ref-274], [`observable.shallow`][ref-275], [`computed.struct`][ref-276]), inheritance, custom constructors, and edge cases like [`async`][ref-277] and generator methods.
- **[`decorate`][ref-243] Calls:** Verification that [`decorate`][ref-243] function calls are correctly converted into [`makeObservable`][ref-24] within the class constructor, addressing multiple calls and undeclared members.
- **Private and Protected Fields:** Ensuring that private and protected class fields decorated with [`@observable`][ref-244] are correctly handled and included in the [`makeObservable`][ref-24] configuration.
- **[`@observer`][ref-247] Transformation:** Extensive testing of the [`@observer`][ref-247] decorator from [`mobx-react`][ref-138] and [`mobx-react-lite`][ref-136], confirming its conversion from a class decorator to a functional wrapper. These tests also preserve class fields, constructor arguments, and generic types within the transformed components, and cover combinations with other decorators like [`@inject`][ref-248].
- **Syntax Preservation:** Checks that various TypeScript and JavaScript syntaxes, including named tuples, [`async`][ref-277] functions, generator functions, and JSX within methods, remain unaltered during the transformation process.

The test suite also includes CLI integration tests, found in [`packages/mobx-undecorate/__tests__/cli.spec.tsx`][ref-278]. These tests validate the end-to-end functionality of the [`mobx-undecorate`][ref-242] command-line tool, ensuring it correctly processes files and applies the transformations. They cover scenarios such as handling file paths with spaces and the [`--parseTsAsNonJsx`][ref-271] flag, which allows the CLI to correctly parse [`.ts`][ref-272] files as non-JSX, preventing parsing errors for specific syntaxes. Together, these tests ensure the reliability and consistency of the migration tool across various codebases and configurations.

## Build and Publishing Infrastructure

![Diagram 17][ref-279]

The processes and tools for building JavaScript packages, managing their versioning, and publishing them are primarily handled by TSDX for compilation and [`@changesets/cli`][ref-280] for automated version bumps and publication workflows.

TSDX, configured through the build script at [`scripts/build.js`][ref-281], orchestrates the compilation of JavaScript packages into various module formats, including ESM, CommonJS, and UMD. It supports different build targets such as "publish" for comprehensive builds encompassing development and production ESM bundles, and "test" for CommonJS bundles suitable for testing environments. This script manages temporary file movements during "publish" builds to ensure specific bundles are preserved and correctly located, working around potential [`tsdx`][ref-282] purging behaviors.

Versioning and publication workflows are automated using [`@changesets/cli`][ref-280]. This tool, whose configuration is managed within the [`.changeset`][ref-283] directory, supports both multi-package (monorepo) and single-package repositories. It facilitates the creation of changesets, which are then used to automatically determine version bumps and generate changelogs during the release process.

### Package Compilation with TSDX

![Diagram 18][ref-284]

JavaScript packages are compiled using [`tsdx`][ref-282], orchestrated by the script in [`scripts/build.js`][ref-281]. This script supports various formats (ESM, CJS, UMD) and environments (development, production), adapting the build process based on a specified target. For instance, a "publish" target triggers a comprehensive build that includes separate development and production ESM bundles, while a "test" target focuses solely on CommonJS bundles. The script also manages temporary file movements for specific build scenarios to prevent conflicts with [`tsdx`][ref-282]'s output purging behavior. For more information on how package versions are managed and published, see [Automated Versioning and Publishing with Changesets](#build-and-publishing-infrastructure-automated-versioning-and-publishing-with-changesets).

### Automated Versioning and Publishing with Changesets

![Diagram 19][ref-285]

The [`@changesets/cli`][ref-280] tool automates the versioning and publishing processes for both multi-package (monorepo) and single-package repositories. This system manages the release workflow by tracking changes and generating appropriate version bumps, thereby streamlining the publication of packages. The configuration and management of these changesets are handled within the [`.changeset`][ref-283] directory.

## Documentation Website

![Diagram 20][ref-286]

The MobX documentation website, built with Docusaurus, serves as a comprehensive resource for users. Its configuration, defined in [`website/siteConfig.js`][ref-287], manages metadata such as the site title, tagline, URL, and base URL, alongside external service integrations like Algolia search and Google Analytics. This configuration also dictates the website's navigation structure, including header links to API references, localized versions, and sponsor pages, as well as branding elements like icons and thematic colors.

The site incorporates various static assets to enhance user experience. Custom styling is applied via [`website/static/css/custom.css`][ref-288], which overrides Docusaurus's default appearance to provide responsive adjustments, unique layouts for elements like collapsible content and tabbed sections, and specialized styling for advertisements. For interactive learning, the website hosts a "Ten minute introduction to MobX and React" tutorial at [`website/static/getting-started.html`][ref-289]. This tutorial leverages editable code snippets with real-time execution and visual feedback to illustrate MobX's core concepts such as observable state, computed values, actions, and their integration with React components. Client-side JavaScript, found in [`website/static/js/scripts.js`][ref-290], further enriches the user interface by enabling dynamic content display, such as expanding[](https://github.com/mobxjs/mobx/tree/main/docs/subclassing.md#L93)

elements based on URL hash changes, and adding informative tooltips to specific UI elements. A reusable React [`Footer`][ref-291] component, located in [`website/core/Footer.js`][ref-292], dynamically generates navigation links to documentation, community resources, and integrates social media sharing options.

### Interactive Tutorials and Live Code Execution

```javascript
class ObservableTodoStore {
  todos = [];
  pendingRequests = 0;

  constructor() {
    makeObservable(this, {
      todos: observable,
      pendingRequests: observable,
      completedTodosCount: computed,
      report: computed,
      addTodo: action,
    });
    // This autorun will log the report whenever relevant observable data changes.
    autorun(() => console.log(this.report));
  }

  get completedTodosCount() {
    return this.todos.filter(
      todo => todo.completed === true
    ).length;
  }

  get report() {
    if (this.todos.length === 0)
      return "";
    const nextTodo = this.todos.find(todo => todo.completed === false);
    return `Next todo: "${nextTodo ? nextTodo.task : ""}". ` +
      `Progress: ${this.completedTodosCount}/${this.todos.length}`;
  }

  addTodo(task) {
    this.todos.push({
      task: task,
      completed: false,
      assignee: null
    });
  }
}

const observableTodoStore = new ObservableTodoStore();

// Example usage demonstrating reactivity
observableTodoStore.addTodo("read MobX tutorial");
observableTodoStore.addTodo("try MobX");
observableTodoStore.todos[0].completed = true;
observableTodoStore.todos[1].task = "try MobX in own project";
observableTodoStore.todos[0].task = "grok MobX tutorial";
```

The MobX website includes an interactive "Ten minute introduction to MobX and React" tutorial, provided by [`website/static/getting-started.html`][ref-289]. This tutorial demonstrates MobX's state management principles and its integration with React components through editable code snippets. The user can modify these snippets, and the system executes the changes in real-time, providing immediate visual feedback on the effects of MobX concepts.

The tutorial introduces concepts sequentially, starting with a basic, non-reactive [`TodoStore`][ref-293] and then progressively enhancing it with MobX features. This includes illustrating how [`makeObservable`][ref-24] is used to define observable properties, computed values, and actions. It also shows how [`autorun`][ref-153] establishes reactions that automatically track and respond to changes in observable data. The integration with React is demonstrated by wrapping components with the [`observer`][ref-139] higher-order component from [`mobx-react-lite`][ref-136], showcasing how React components can efficiently re-render in response to MobX state changes. The tutorial also covers handling references between observable objects and managing asynchronous operations within MobX actions. The interactive nature of the tutorial, supported by JavaScript code in [`website/static/js/scripts.js`][ref-290], allows for direct experimentation and observation of MobX's dynamic dependency tracking.

### Custom Styling and Theming

```css
/* custom.css */

/* Responsive design for images */
article p img {
  display: inline-block; /* Override Docusaurus default to inline-block */
}

/* Example of a media query for smaller screens */
@media only screen and (max-width: 735px) {
  .nav-footer .sitemap .nav-home {
    margin-left: -10px;
  }
}

/* Styling for  and  elements */
details {
    background-color: aliceblue;
    font-size: 0.8em;
    padding: 4px 8px;
    border-radius: 4px;
}

details > summary {
    color: navy;
    cursor: pointer;
}

/* Styling for tabbed content */
.tabs {
    background: #efefef
}

.tab-content {
    padding: 1.25rem 1.5rem;
}

/* Custom styling for Carbon Ads integration */
#carbonads {
    display: block;
    max-width: 728px;
    /* ... other Carbon Ads styling ... */
    margin-left: auto;
    margin-right: auto;
    margin-bottom: 20px;
}

.carbon-text {
    /* ... Carbon Ads text styling ... */
}

/* Media query specific to Carbon Ads for smaller screens */
@media only screen and (min-width: 320px) and (max-width: 759px) {
    .carbon-text {
        font-size: 14px;
    }
}
```

The documentation website utilizes custom CSS to extend and override Docusaurus's default styling, ensuring a consistent visual identity and responsive behavior. The primary stylesheet, located at [`website/static/css/custom.css`][ref-288], manages these modifications.

Key styling adjustments include:

- **Responsive Layout**: Media queries within [`website/static/css/custom.css`][ref-288] are extensively used to adapt the layout and appearance of various elements, such as the footer navigation, header container, and documentation navigation, across different screen sizes. This ensures the site remains functional and aesthetically pleasing on both large desktop displays and smaller mobile devices.
- **Component-Specific Theming**: Styles are applied to specific HTML elements and components to create unique visual presentations. This includes custom styling for collapsible content sections defined by [](https://github.com/mobxjs/mobx/tree/main/docs/subclassing.md#L93)[](https://github.com/mobxjs/mobx/tree/main/docs/subclassing.md#L93)and [](https://github.com/mobxjs/mobx/tree/main/docs/actions.md#L171)[](https://github.com/mobxjs/mobx/tree/main/docs/actions.md#L171)tags, as well as distinct appearances for tabbed content areas. The stylesheet also defines a flexible layout for "benefits" sections, often used to highlight key features or advantages, and provides specific styling for call-to-action buttons.
- **External Integrations**: The styling also addresses the integration of external elements, such as Carbon Ads. Dedicated CSS rules ensure that advertisements are displayed correctly, are responsive, and align with the website's overall branding.
- **Visual Enhancements**: Subtle enhancements are implemented, such as replacing text-based GitHub links with SVG icons for a cleaner navigation experience and adding specific unicode characters after certain elements for visual cues.

### Dynamic UI Enhancements and Navigation

```javascript
function openTarget() {
    var hash = location.hash.substring(1);
    if (hash) {
        var details = document.getElementById(hash);
    }

    // Opens  tags based on the URL hash.
    if (details && details.tagName.toLowerCase() === 'details') {
        details.open = true;
        // Scroll into view after a short delay to avoid conflicts.
        setTimeout(function() {
            details.scrollIntoView();
        }, 150);
    }
}

function addTooltipToRockets() {
    // Selects elements that might contain rocket emojis.
    var classNames = ['navGroups', 'onPageNav', 'post']; // ... and other relevant class names
    var rocketRegex = /🚀/g;

    for (var className of classNames) {
        var els = document.getElementsByClassName(className);
        for (var el of els) {
            // Replaces rocket emojis with a styled span that includes a tooltip.
            el.innerHTML = el.innerHTML.replace(rocketRegex, '🚀');
        }
    }
}

// Event listeners to trigger these functions.
window.addEventListener('hashchange', openTarget);
window.addEventListener('DOMContentLoaded', function() {
    addTooltipToRockets();
    openTarget();
});
```

Client-side JavaScript functions enhance the user experience by enabling hash-based navigation for deep linking and providing informative tooltips. The [`openTarget`][ref-294] function, defined in [`website/static/js/scripts.js`][ref-290], manages URL hash changes. When a hash is present in the URL, this function locates the corresponding HTML element by its ID. If the element is a[](https://github.com/mobxjs/mobx/tree/main/docs/subclassing.md#L93)

tag, it programmatically opens it and scrolls it into view. This ensures that deep links to collapsible content are properly displayed. Additionally, the [`addTooltipToRockets`][ref-295] function in the same file enriches the UI by adding "Advanced feature" tooltips to rocket emojis ([`🚀`][ref-296]) found within various navigation and content sections of the website. These functionalities are activated upon page load and whenever the URL hash changes, providing dynamic and interactive elements to the documentation.

---

## References

[ref-1]: ./images/diagram-1.svg
[ref-2]: ./images/diagram-2.svg
[ref-3]: https://github.com/mobxjs/mobx/tree/main/packages/mobx
[ref-4]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/atom.ts#L26
[ref-5]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/atom.ts
[ref-6]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/computedvalue.ts#L80
[ref-7]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/computedvalue.ts
[ref-8]: https://github.com/mobxjs/mobx/tree/main/docs/assets/getting-started-assets/javascripts/mobx.umd.js#L948
[ref-9]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/reaction.ts
[ref-10]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/observable.ts
[ref-11]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/derivation.ts
[ref-12]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observablevalue.ts#L60
[ref-13]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observablevalue.ts
[ref-14]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observablearray.ts#L118
[ref-15]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observablearray.ts
[ref-16]: https://github.com/mobxjs/mobx/tree/main/docs/configuration.md#L14
[ref-17]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observablemap.ts#L93
[ref-18]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observablemap.ts
[ref-19]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observableset.ts#L69
[ref-20]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observableset.ts
[ref-21]: https://github.com/mobxjs/mobx/tree/main/docs/api.md#L63
[ref-22]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observableobject.ts#L91
[ref-23]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observableobject.ts
[ref-24]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/makeObservable.ts#L27
[ref-25]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/makeObservable.ts#L51
[ref-26]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/__tests__/v5/base/observables.js#L2167
[ref-27]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/action.ts
[ref-28]: https://github.com/mobxjs/mobx/tree/main/docs/api.md#L180
[ref-29]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/flow.ts#L31
[ref-30]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/flow.ts
[ref-31]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/action.ts
[ref-32]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/annotation.ts#L9
[ref-33]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/annotation.ts
[ref-34]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/decorators.ts
[ref-35]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/actionannotation.ts
[ref-36]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/__tests__/v5/base/proxies.js#L329
[ref-37]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/computedannotation.ts
[ref-38]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/flowannotation.ts
[ref-39]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/observable.ts#L42
[ref-40]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observableannotation.ts
[ref-41]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/__tests__/v5/base/make-observable.ts#L1613
[ref-42]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/overrideannotation.ts
[ref-43]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/__tests__/v5/base/make-observable.ts#L1612
[ref-44]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/autoannotation.ts
[ref-45]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/intercept.ts#L18
[ref-46]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/intercept.ts
[ref-47]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/observe.ts#L17
[ref-48]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/observe.ts
[ref-49]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/become-observed.ts#L16
[ref-50]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/become-observed.ts#L35
[ref-51]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/become-observed.ts
[ref-52]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/spy.ts#L61
[ref-53]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/spy.ts
[ref-54]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/trace.ts#L3
[ref-55]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/trace.ts
[ref-56]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/extras.ts#L13
[ref-57]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/extras.ts#L27
[ref-58]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/extras.ts
[ref-59]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/tojs.ts#L72
[ref-60]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/tojs.ts
[ref-61]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/globalstate.ts#L23
[ref-62]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/globalstate.ts
[ref-63]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/globalstate.ts#L188
[ref-64]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/__tests__/mixed-versions
[ref-65]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/configure.ts#L8
[ref-66]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/configure.ts
[ref-67]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils
[ref-68]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/comparer.ts#L27
[ref-69]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/comparer.ts
[ref-70]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/__tests__/v5/base/observables.js#L1185
[ref-71]: https://github.com/mobxjs/mobx/tree/main/docs/computeds.md#L156
[ref-72]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/observable.ts#L210
[ref-73]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/eq.ts#L15
[ref-74]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/eq.ts
[ref-75]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/errors.ts
[ref-76]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/flow-typed/mobx.js
[ref-77]: ./images/diagram-3.svg
[ref-78]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/derivation.ts#L18
[ref-79]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/derivation.ts#L25
[ref-80]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/derivation.ts#L28
[ref-81]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observablearray.ts#L463
[ref-82]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observablearray.ts#L469
[ref-83]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observablearray.ts#L442
[ref-84]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/legacyobservablearray.ts#L55
[ref-85]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/legacyobservablearray.ts
[ref-86]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/__tests__/v5/base/array.js#L604
[ref-87]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/object-api.ts#L78
[ref-88]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/object-api.ts#L159
[ref-89]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observablemap.ts#L168
[ref-90]: ./images/diagram-4.svg
[ref-91]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/decorators.ts#L12
[ref-92]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/actionannotation.ts#L14
[ref-93]: https://github.com/mobxjs/mobx/tree/main/LICENSE#L6
[ref-94]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/computedannotation.ts#L12
[ref-95]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/observableannotation.ts#L13
[ref-96]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/autoannotation.ts#L19
[ref-97]: https://github.com/mobxjs/mobx/tree/main/.eslintrc.js#L26
[ref-98]: https://github.com/mobxjs/mobx/tree/main/docs/actions.md#L278
[ref-99]: https://github.com/mobxjs/mobx/tree/main/docs/actions.md#L495
[ref-100]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/flow.ts#L20
[ref-101]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/flow.ts#L25
[ref-102]: https://github.com/mobxjs/mobx/tree/main/docs/actions.md#L482
[ref-103]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/flow.ts#L149
[ref-104]: ./images/diagram-5.svg
[ref-105]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/CHANGELOG.md#L716
[ref-106]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/intercept-utils.ts#L27
[ref-107]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/intercept-utils.ts
[ref-108]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/intercept-read.ts#L19
[ref-109]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/intercept-read.ts
[ref-110]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/types/listen-utils.ts
[ref-111]: ./images/diagram-6.svg
[ref-112]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/globalstate.ts#L158
[ref-113]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/globalstate.ts#L141
[ref-114]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/globalstate.ts#L100
[ref-115]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/comparer.ts#L7
[ref-116]: https://github.com/mobxjs/mobx/tree/main/scripts/build.js#L15
[ref-117]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/comparer.ts#L11
[ref-118]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/comparer.ts#L15
[ref-119]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/comparer.ts#L19
[ref-120]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/CHANGELOG.md#L303
[ref-121]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/utils.ts
[ref-122]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/utils.ts#L114
[ref-123]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/utils.ts#L215
[ref-124]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/utils.ts#L62
[ref-125]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/utils.ts#L81
[ref-126]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/utils.ts#L146
[ref-127]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/utils.ts#L85
[ref-128]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/utils.ts#L42
[ref-129]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/global.ts#L6
[ref-130]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/global.ts
[ref-131]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/CHANGELOG.md#L391
[ref-132]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/global.ts#L1
[ref-133]: https://github.com/mobxjs/mobx/tree/main/docs/api.md#L279
[ref-134]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/utils/global.ts#L2
[ref-135]: ./images/diagram-7.svg
[ref-136]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/README.md#L1
[ref-137]: ./images/diagram-8.svg
[ref-138]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/README.md#L1
[ref-139]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/observer.tsx#L10
[ref-140]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/observer.tsx
[ref-141]: https://github.com/mobxjs/mobx/tree/main/docs/assets/getting-started-assets/javascripts/mobx-react.js#L97
[ref-142]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/__tests__/finalizationRegistry.tsx#L34
[ref-143]: https://github.com/mobxjs/mobx/tree/main/docs/assets/getting-started-assets/javascripts/mobx-react.js#L74
[ref-144]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/observerClass.ts
[ref-145]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/__tests__/Provider.test.tsx#L7
[ref-146]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/inject.ts#L78
[ref-147]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/Provider.tsx
[ref-148]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/inject.ts
[ref-149]: https://github.com/mobxjs/mobx/tree/main/docs/react-integration.md#L319
[ref-150]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/disposeOnUnmount.ts#L26
[ref-151]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/disposeOnUnmount.ts
[ref-152]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/autorun.ts#L114
[ref-153]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/api/autorun.ts#L38
[ref-154]: https://github.com/mobxjs/mobx/tree/main/docs/react-integration.md#L310
[ref-155]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/propTypes.ts#L202
[ref-156]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/propTypes.ts
[ref-157]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/utils/utils.ts
[ref-158]: ./images/diagram-9.svg
[ref-159]: https://github.com/mobxjs/mobx/tree/main/website/core/Footer.js#L24
[ref-160]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/README.md#L61
[ref-161]: https://github.com/mobxjs/mobx/tree/main/docs/assets/getting-started-assets/javascripts/react-with-addons.js#L6876
[ref-162]: ./images/diagram-10.svg
[ref-163]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/Provider.tsx#L5
[ref-164]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/CHANGELOG.md#L277
[ref-165]: https://github.com/mobxjs/mobx/tree/main/docs/assets/getting-started-assets/javascripts/react-with-addons.js#L6136
[ref-166]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/propTypes.ts#L194
[ref-167]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/propTypes.ts#L197
[ref-168]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/propTypes.ts#L196
[ref-169]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/__tests__/propTypes.test.ts#L91
[ref-170]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/src/core/derivation.ts#L299
[ref-171]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/propTypes.ts#L195
[ref-172]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/propTypes.ts#L198
[ref-173]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/propTypes.ts#L200
[ref-174]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/src/propTypes.ts#L4
[ref-175]: ./images/diagram-11.svg
[ref-176]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/ObserverComponent.ts
[ref-177]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/index.ts#L22
[ref-178]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/useObserver.ts
[ref-179]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/CHANGELOG.md#L73
[ref-180]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/useLocalObservable.ts#L4
[ref-181]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/useLocalObservable.ts
[ref-182]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/useLocalStore.ts#L7
[ref-183]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/useAsObservableSource.ts#L5
[ref-184]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/useLocalStore.ts
[ref-185]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/useAsObservableSource.ts
[ref-186]: https://github.com/mobxjs/mobx/tree/main/docs/react-integration.md#L160
[ref-187]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/utils/observerBatching.ts#L7
[ref-188]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/utils/observerBatching.ts
[ref-189]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/README.md#L95
[ref-190]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/utils/UniversalFinalizationRegistry.ts#L62
[ref-191]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/utils/UniversalFinalizationRegistry.ts
[ref-192]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/index.ts#L20
[ref-193]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/index.ts
[ref-194]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/staticRendering.ts#L3
[ref-195]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/staticRendering.ts
[ref-196]: ./images/diagram-12.svg
[ref-197]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/observer.ts
[ref-198]: https://github.com/mobxjs/mobx/tree/main/docs/react-integration.md#L327
[ref-199]: https://github.com/mobxjs/mobx/tree/main/docs/api.md#L228
[ref-200]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/ObserverComponent.ts#L6
[ref-201]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/utils/UniversalFinalizationRegistry.ts#L7
[ref-202]: https://github.com/mobxjs/mobx/tree/main/docs/api.md#L28
[ref-203]: https://github.com/mobxjs/mobx/tree/main/docs/react-integration.md#L142
[ref-204]: https://github.com/mobxjs/mobx/tree/main/docs/actions.md#L4
[ref-205]: ./images/diagram-13.svg
[ref-206]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/src/utils/observerFinalizationRegistry.ts
[ref-207]: https://github.com/mobxjs/mobx/tree/main/docs/assets/getting-started-assets/javascripts/mobx.umd.js#L985
[ref-208]: https://github.com/mobxjs/mobx/tree/main/docs/configuration.md#L75
[ref-209]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/README.md#L1
[ref-210]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx
[ref-211]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/README.md
[ref-212]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/package.json#L19
[ref-213]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/src/index.js
[ref-214]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/src/utils.js
[ref-215]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/src/exhaustive-make-observable.js
[ref-216]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/src/missing-make-observable.js
[ref-217]: https://github.com/mobxjs/mobx/tree/main/docs/observable-state.md#L152
[ref-218]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/src/unconditional-make-observable.js
[ref-219]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/src/missing-observer.js
[ref-220]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/src/no-anonymous-observer.js
[ref-221]: ./images/diagram-14.svg
[ref-222]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/__tests__/utils/get-rule-tester.js#L3
[ref-223]: https://github.com/mobxjs/mobx/tree/main/docs/enabling-decorators.md#L155
[ref-224]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/CHANGELOG.md#L1936
[ref-225]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/README.md#L62
[ref-226]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/README.md#L82
[ref-227]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/README.md#L87
[ref-228]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/README.md#L91
[ref-229]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/README.md#L124
[ref-230]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/__tests__/utils/get-rule-tester.js#L25
[ref-231]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/__tests__/utils/get-rule-tester.js
[ref-232]: https://github.com/mobxjs/mobx/tree/main/package.json#L34
[ref-233]: https://github.com/mobxjs/mobx/tree/main/package.json#L9
[ref-234]: https://github.com/mobxjs/mobx/tree/main/package.json#L45
[ref-235]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/preview
[ref-236]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/preview/.eslintrc.js
[ref-237]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/preview/make-observable.js
[ref-238]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/preview/missing-observer.js
[ref-239]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/preview/no-anonymous-observer.js
[ref-240]: https://github.com/mobxjs/mobx/tree/main/.watchmanconfig#L2
[ref-241]: https://github.com/mobxjs/mobx/tree/main/packages/eslint-plugin-mobx/preview/node_modules/eslint-plugin-mobx
[ref-242]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/README.md#L1
[ref-243]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/__tests__/undecorate.spec.ts#L763
[ref-244]: https://github.com/mobxjs/mobx/tree/main/docs/api.md#L60
[ref-245]: https://github.com/mobxjs/mobx/tree/main/docs/api.md#L175
[ref-246]: https://github.com/mobxjs/mobx/tree/main/docs/api.md#L210
[ref-247]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/__tests__/undecorate.spec.ts#L1027
[ref-248]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/README.md#L316
[ref-249]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/cli.js#L7
[ref-250]: ./images/diagram-15.svg
[ref-251]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/src/undecorate.ts#L87
[ref-252]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/src/undecorate.ts
[ref-253]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/package.json#L28
[ref-254]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/src/undecorate.ts#L10
[ref-255]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite/__tests__/useLocalObservable.test.tsx#L377
[ref-256]: https://github.com/mobxjs/mobx/tree/main/docs/assets/getting-started-assets/javascripts/react-with-addons.js#L10424
[ref-257]: https://github.com/mobxjs/mobx/tree/main/docs/actions.md#L253
[ref-258]: https://github.com/mobxjs/mobx/tree/main/package.json#L3
[ref-259]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/CHANGELOG.md#L1187
[ref-260]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-react/__tests__/observer.test.tsx#L892
[ref-261]: https://github.com/mobxjs/mobx/tree/main/packages/mobx/__tests__/v4/base/cycles.js#L91
[ref-262]: ./images/diagram-16.svg
[ref-263]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/cli.js
[ref-264]: https://github.com/mobxjs/mobx/tree/main/docs/migrating-from-4-or-5.md#L47
[ref-265]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/README.md#L23
[ref-266]: https://github.com/mobxjs/mobx/tree/main/docs/migrating-from-4-or-5.md#L60
[ref-267]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/README.md#L25
[ref-268]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/src/undecorate.ts#L61
[ref-269]: https://github.com/mobxjs/mobx/tree/main/README.md#L170
[ref-270]: https://github.com/mobxjs/mobx/tree/main/docs/lazy-observables.md#L21
[ref-271]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/__tests__/cli.spec.tsx#L32
[ref-272]: https://github.com/mobxjs/mobx/tree/main/package.json#L19
[ref-273]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/__tests__/undecorate.spec.ts
[ref-274]: https://github.com/mobxjs/mobx/tree/main/docs/actions.md#L180
[ref-275]: https://github.com/mobxjs/mobx/tree/main/docs/api.md#L119
[ref-276]: https://github.com/mobxjs/mobx/tree/main/docs/computeds.md#L154
[ref-277]: https://github.com/mobxjs/mobx/tree/main/docs/api.md#L7
[ref-278]: https://github.com/mobxjs/mobx/tree/main/packages/mobx-undecorate/__tests__/cli.spec.tsx
[ref-279]: ./images/diagram-17.svg
[ref-280]: https://github.com/mobxjs/mobx/tree/main/package.json#L35
[ref-281]: https://github.com/mobxjs/mobx/tree/main/scripts/build.js
[ref-282]: https://github.com/mobxjs/mobx/tree/main/package.json#L70
[ref-283]: https://github.com/mobxjs/mobx/tree/main/.changeset
[ref-284]: ./images/diagram-18.svg
[ref-285]: ./images/diagram-19.svg
[ref-286]: ./images/diagram-20.svg
[ref-287]: https://github.com/mobxjs/mobx/tree/main/website/siteConfig.js
[ref-288]: https://github.com/mobxjs/mobx/tree/main/website/static/css/custom.css
[ref-289]: https://github.com/mobxjs/mobx/tree/main/website/static/getting-started.html
[ref-290]: https://github.com/mobxjs/mobx/tree/main/website/static/js/scripts.js
[ref-291]: https://github.com/mobxjs/mobx/tree/main/website/core/Footer.js#L10
[ref-292]: https://github.com/mobxjs/mobx/tree/main/website/core/Footer.js
[ref-293]: https://github.com/mobxjs/mobx/tree/main/docs/defining-data-stores.md#L79
[ref-294]: https://github.com/mobxjs/mobx/tree/main/website/static/js/scripts.js#L1
[ref-295]: https://github.com/mobxjs/mobx/tree/main/website/static/js/scripts.js#L12
[ref-296]: https://github.com/mobxjs/mobx/tree/main/docs/api.md#L346