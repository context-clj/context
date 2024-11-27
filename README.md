# Context

## Introduction

This framework is designed for building modular and dynamic backend platforms with plugin support. Its philosophy is based on simplicity, modularity, and native compatibility with Clojure.

Key features:
1. **Modules as the basic building blocks** of the system.
2. Works with **ordinary namespaces, functions, and `require`**, making the code intuitive and natural for Clojure developers.
3. A unified approach to state management and module interaction.
4. Minimal "magic" — you write "plain" Clojure code, avoiding complexities like in other frameworks (e.g., records, methods, custom maps).

---

## 1. Modules and Manifests

### Module
A **module** is an ordinary Clojure namespace containing standard functions. Modules serve as logical units of functionality and can interact with each other via service calls.

### Manifest
A **manifest** is an "ordinary" var containing a hashmap that describes the module's parameters and dependencies.

Example manifest:
```clj
(def manifest
  {:description "PostgreSQL Service"
   :deps [#'log/manifest]
   :config
   {:port      {:type "integer" :default 5432 :validator pos-int?}
    :host      {:type "string"  :required true}
    :database  {:type "string"  :required true}
    :password  {:type "string"  :sensitive true :required true}
    :pool-size {:type "integer" :default 5 :validator pos-int?}}})
```

#### Configuration in the Manifest
- **Types** (`:type`) — Define the expected type of the parameter (e.g., `"string"`, `"integer"`).
- **Required** (`:required`) — Indicates whether a parameter must be provided.
- **Default values** (`:default`) — Used if a parameter is not specified.
- **Sensitive data** (`:sensitive`) — Marks parameters that require secure handling.
- **Validation rules** (`:validator`) — A custom function for validating values.

---

## 2. Module Lifecycle

Modules have three phases: **initialization (start)**, **operation (services)**, and **shutdown (stop)**.

### `start`
The `start` function initializes a module:
- Accepts:
  - **Context**: The system-wide context.
  - **Parameters**: Validated values collected from the manifest.
- Returns the module state, which is stored in the system context.

Example:
```clj
(defn start [context config-params]
  (let [connection (connect-to-db config-params)]
    {:connection connection}))
```

### `stop`
The `stop` function shuts down a module:
- Accepts the state returned by `start`.
- Ensures resources are released, connections are closed, etc.

Example:
```clj
(defn stop [module-state]
  (let [connection (:connection module-state)]
    (close-connection connection)))
```

---

## 3. Module Services

Services are the interfaces provided by modules. They:
- Are implemented as ordinary functions.
- Accept the system context as the first parameter.
- Receive additional parameters for their operations.

Example:
```clj
(defn query-database [context params]
  (let [db-connection (get-in context [:system :database])]
    (execute-query db-connection params)))
```

Services can also invoke functions from other modules:
```clj
(defn process-request [context params]
  (let [response (other-module/service-fn context params)]
    (process-response response)))
```

---

## 4. Module Interaction

Modules interact with each other through standard Clojure tools:
- **`require`** for dependency inclusion.
- Service calls with context and parameter passing.

Example:
```clj
(ns my-module
  (:require [used-module :as um]))

(defn my-service [context params]
  (um/service-fn context params))
```

### Benefits of this Approach
1. Code looks like "ordinary" Clojure code.
2. Easy to read and maintain.
3. Uniform module composition.

---

## 5. System Loading Process

System startup is divided into two phases:

### Phase 1: Manifest Loading
1. A list of modules is provided as a list of namespaces.
2. Modules are dynamically required.
3. Manifests are read, and dependencies are resolved.
4. All dependencies are topologically sorted.

### Phase 2: Module Initialization
1. Each module's `start` function is called.
2. Validated parameters from the manifest are passed to the function.

---

## 6. Shutdown Process

The system tracks the states returned by the `start` functions of all initialized modules. During shutdown:
1. Each module's `stop` function is called.
2. The module's state is passed to `stop` for proper cleanup.

---

## 7. Extensibility and Plugins

The framework supports:
1. **Hooks**: Modules can declare hooks, and other modules can register handlers for them.
2. **Plugins**: Modules can be compiled into JAR files and dynamically loaded.

---

This documentation covers all key aspects of the framework's architecture and usage. Let me know if there's anything else you'd like to adjust or expand!
