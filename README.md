# Dynamic Module Reform

ECMAScript Proposal specs for the reform to preserve the order of execution for dynamic modules.

Spec drafted by [@caridy](https://github.com/caridy).

This proposal is currently [stage 0](https://github.com/tc39/ecma262) of the [process](https://tc39.github.io/process-document/).

## Rationale

ES module spec requires all Module Records to know all the exports during `ModuleDeclarationInstantiation()`. This is needed to guarantee that the linking process (bindings from one module environment record to another) can be accomplish, and works very well for Source Text Module Records, but forces other dynamic modules (like Node CJS modules) to be evaluated during the `ModuleDeclarationInstantiation()` phase. As a result, it is impossible to preserve the order of evaluation.

This proposal introduces a new internal slot `[[PendingImportEntries]]` on Source Text Module Records that can be used to track down the import entries that are coming from Dynamic Module Records that hasn't been evaluated yet. As a result, a Dynamic Module Record can resolve to `"pending"` during the `ModuleDeclarationInstantiation()` phase to signal that the validation or assertion about the bindings should be deferred to the ModuleEvaluation() phase for the Source Text Module Record importing from a Dynamic Module Record.

## Additional Information

Most of the discussion around this topic is condenced in the meetings notes from TC39 Sept 2016 Meeting:

* [Meeting Notes](https://esdiscuss.org/notes/2016-09-28)
* [ES Modules Slides](https://esdiscuss.org/notes/2016-09/ES-Modules-Compat.pdf)

## Spec

You can view the spec rendered as [HTML](https://rawgit.com/caridy/proposal-dynamic-modules/master/index.html).
