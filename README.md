# Dynamic Module Reform

ECMAScript Proposal specs for the reform to preserve the order of execution for dynamic modules.

Spec drafted by [@caridy](https://github.com/caridy).

This proposal is currently [stage 0](https://github.com/tc39/ecma262) of the [process](https://tc39.github.io/process-document/).

## Rationale

ES module spec requires all Module Records to know all the exports during `ModuleDeclarationInstantiation()`. This is needed to guarantee that the linking process (bindings from one module environment record to another) can be accomplish, and works very well for Source Text Module Records, but forces other dynamic modules (like Node CJS modules) to be evaluated during the `ModuleDeclarationInstantiation()` phase. As a result, it is impossible to preserve the order of evaluation.

## Preserving the order of evaluation for Dynamic Module Records

This proposal introduces the `"pending"` resolution value when calling `ResolveExport()` for a Dynamic Module Record. This simple change allow us to identify, during the linking phase, a binding that could potentially be in TDZ and does not require explicit assertion during this phase. As a result, we can defer the evaluation of a Dynamic Module Record to preserve the execution order, and we do so under the assumption that __eventually__ the imported bindings from the corresponding environment record is in TDZ until it gets populated by the Dynamic Module Record.

## Enabling Circular Dependencies with Dynamic Module Records

This change also enable us to match the semantics of NCJS when it comes to circular dependencies. The following example illustrates this:

```js
// even.js
module.exports = function even(n) { ... require('odd')(n - 1) ... }

// odd.js
module.exports = function odd(n) { ... require('even')(n - 1) ... }
```

These CJS modules will work in Node independently of which one is imported first, the same if both are written as ESM. But if one of them is ESM and the other is CJS, based on the currenct spec, we might get a static error depending on which one is imported first. E.g.:

```js
// even.js (ESM):
import odd from "odd";
export default function even() {  ... odd(n - 1) ... }

// odd.js (DM)
module.exports = function odd(n) { ... require('even')(n - 1) ... }
```

If the binding for `odd` in `even.js` is not validated until it is accessed, the NCJS semantics are preserved, and this example works independently of which one is imported first.

## Compromises

With this proposal, the statically verifiable mechanism introduced by ESM can only be enforced in a ES Module that is importing from another ES Module, and any interaction with Dynamic Module Records will rely on a runtime errors (TDZ when accessing named exports).

## Additional Information

Most of the discussion around this topic is condenced in the meetings notes from TC39 Sept 2016 Meeting:

* [Meeting Notes](https://esdiscuss.org/notes/2016-09-28)
* [ES Modules Slides](https://esdiscuss.org/notes/2016-09/ES-Modules-Compat.pdf)

## Spec

You can view the spec rendered as [HTML](https://rawgit.com/caridy/proposal-dynamic-modules/master/index.html).
