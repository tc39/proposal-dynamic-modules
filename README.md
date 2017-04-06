# Dynamic Module Reform (REJECTED)

ECMAScript Proposal specs for the reform to preserve the order of execution for dynamic modules.

## Champions

* [@caridy](https://github.com/caridy).
* [@dherman](https://github.com/dherman).

This proposal was rejected in favor of preserving the current semantics. You can read more about this decision in the [tc39-notes](https://github.com/rwaldron/tc39-notes/blob/master/es8/2017-03/mar-23.md#10iia-dynamic-module-reform-for-stage-2). The last slide from [Update 2 - March 2017](https://docs.google.com/presentation/d/1EYOysPhgjXtgmuNoZ_wUCMElZ8GKLxJmCLeF0EvUXkc/edit#slide=id.g1f1f22a6f5_1_31) shows why this proposal is faulty.

## Slides

* [Update 1 - Nov 2016](https://docs.google.com/presentation/d/1EYOysPhgjXtgmuNoZ_wUCMElZ8GKLxJmCLeF0EvUXkc/edit#slide=id.p)
* [Update 2 - March 2017](https://docs.google.com/presentation/d/1EYOysPhgjXtgmuNoZ_wUCMElZ8GKLxJmCLeF0EvUXkc/edit#slide=id.g1f19d52d1f_0_0)

## Rationale

ES module spec requires all Module Records to know all the exports during `ModuleDeclarationInstantiation()`. This is needed to guarantee that the linking process (bindings from one module environment record to another) can be accomplish, and works very well for Source Text Module Records, but forces other dynamic modules (like Node CJS modules) to be evaluated during the `ModuleDeclarationInstantiation()` phase. As a result, it is impossible to preserve the order of evaluation.

## Preserving the order of evaluation for Dynamic Module Records

This proposal introduces the `"pending"` resolution value when calling `ResolveExport()` for a Dynamic Module Record. Additionally, this proposal introduces a new internal slot `[[PendingImportEntries]]` on Source Text Module Records that is used to track the import entries that are coming from Dynamic Module Records that hasn't been evaluated yet. As a result, calling `ResolveExport()` on the Dynamic Module Record during the `ModuleDeclarationInstantiation()` phase can resolve to `"pending"` to signal that the validation or assertion about the bindings should be deferred to the `ModuleEvaluation()` phase for the Source Text Module Record importing from a Dynamic Module Record.

These changes allow us to identify, during the linking phase, an import binding that cannot be created yet and does not require explicit assertion during this phase. As a result, we can defer the evaluation of a Dynamic Module Record to preserve the execution order, and we do so under the assumption that the imported bindings from the Dynamic Module Record will gets populated after it is evaluated.

## Enabling Circular Dependencies with Dynamic Module Records

This change also enable us to match the semantics of NCJS when it comes to circular dependencies. The following example illustrates this:

```js
// even.js
module.exports = function even(n) {
    ...
    require('odd')(n - 1)
    ...
}

// odd.js
module.exports = function odd(n) {
    ...
    require('even')(n - 1)
    ...
}
```

These CJS modules will work in Node independently of which one is imported first, the same applies if both are written as ESM. But if one of them is ESM and the other is CJS, based on the currenct spec, we might get a static error depending on which one is imported first. E.g.:

```js
// even.js (ESM):
import odd from "odd";
export default function even() {
    ...
    odd(n - 1)
    ...
}

// odd.js (DM)
module.exports = function odd(n) {
    ...
    require('even')(n - 1)
    ...
}
```

If the binding for `odd` in `even.js` is not created until after `odd.js` is evaluated, the NCJS semantics are preserved, and this example works independently of which one is imported first.

## SyntaxError in Source Text Module Records

* Throw during `ModuleDeclarationInstantiation` if the indirect import entry resolves to *null* or *ambiguous*.
* Throw during `ModuleEvaluation` if a pending import entry resolves to *null* or *ambiguous* or *pending* after evaluating all dependencies, and before evaluating the source text.

## Compromises

With this proposal, the statically verifiable mechanism introduced by ESM can only be enforced in a ES Module that is importing from another ES Module, and any interaction with Dynamic Module Records will be deferred to the `ModuleEvaluation()` phase for Dynamic Module Records that were not previously evaluated.

## Additional Information

Most of the discussion around this topic is condenced in the meetings notes from TC39 Sept 2016 Meeting:

* [Meeting Notes](https://esdiscuss.org/notes/2016-09-28)
* [ES Modules Slides](https://esdiscuss.org/notes/2016-09/ES-Modules-Compat.pdf)

## Spec

You can view the spec rendered as [HTML](https://rawgit.com/tc39/proposal-dynamic-modules/master/index.html).
